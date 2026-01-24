# Vulnerability Report: Host Header Injection leading to OIDC Discovery Poisoning

**Target:** [REDACTED_TARGET] (Financial Services Gateway)
**Vulnerability Type:** Server-Side Injection > Content Spoofing > External Authentication Injection
**Severity:** Medium (P3) / Low (P4)
**Status:** Submitted

## Summary
I identified a Host Header Injection vulnerability on the OpenID Connect (OIDC) discovery endpoint. The application dynamically generates critical authentication metadata—including the `issuer` and `authorization_endpoint`—using the user-supplied Host header. This allows an attacker to "poison" the discovery document.

## Technical Description
The target server hosts an OIDC configuration file at `/.well-known/openid-configuration`. This endpoint fails to validate the HTTP `Host` header. When a request is made with a malicious Host header (e.g., `Host: evil-attacker.com`), the server reflects this domain into the JSON response.

This effectively changes the trusted "Issuer" and "Authorization Endpoint" to an attacker-controlled domain.

## Steps to Reproduce
1.  **Identify the Endpoint:**
    Target URL: `https://[REDACTED_TARGET]/oauth/idp/.well-known/openid-configuration`

2.  **Send Malicious Request:**
    Send a curl request with a modified Host header:
    ```bash
    curl -v -k https://[REDACTED_TARGET]/oauth/idp/.well-known/openid-configuration \
    -H "Host: evil-attacker-domain.com"
    ```

3.  **Observe Response:**
    The server responds with `200 OK`, and the JSON body contains the injected domain:
    ```json
    {
      "issuer": "[https://evil-attacker-domain.com/oauth/idp](https://evil-attacker-domain.com/oauth/idp)",
      "authorization_endpoint": "[https://evil-attacker-domain.com/oauth/idp/login](https://evil-attacker-domain.com/oauth/idp/login)",
      "token_endpoint": "[https://evil-attacker-domain.com/oauth/idp/token](https://evil-attacker-domain.com/oauth/idp/token)",
      "jwks_uri": "[https://evil-attacker-domain.com/oauth/idp/certs](https://evil-attacker-domain.com/oauth/idp/certs)"
    }
    ```

## Impact (Attack Scenario)
This vulnerability leads to **OIDC Discovery Poisoning**:

* **OAuth Hijacking:** If a client application (e.g., a mobile banking app) relies on this discovery document to locate the authorization server, it will be directed to send the user's credentials or OAuth tokens to the attacker's server instead of the legitimate Identity Provider.
* **Web Cache Poisoning (Potential):** If this response is cached by a CDN or intermediate proxy, legitimate users requesting the configuration will be served the poisoned metadata, redirecting all traffic to the attacker.
* **Phishing Credibility:** An attacker can use the legitimate domain to serve a configuration that points to a malicious login page, bypassing standard phishing filters that check the initial URL.

## Remediation
The application should be configured to ignore the HTTP `Host` header for generating links. Instead, it should use a static, trusted configuration or a whitelist of allowed domains (e.g., `[REDACTED_TARGET]`) to construct the OIDC metadata.