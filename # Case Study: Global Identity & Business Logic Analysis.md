# Case Study: Global Identity & Business Logic Analysis

## Target: Global Home Technology Retailer [Redacted] 

## Vulnerability Class: 
Broken Access Control (BAC) / Business Logic Severity: Medium (Informative/Research)

## Executive Summary
This research project involved a comprehensive security assessment of a global retail platform’s identity and commerce logic across 18+ regional domains (e.g., .at, .ca, .de, .it). The focus was on identifying cross-border data leaks, horizontal privilege escalation in the "My Account" section, and race conditions within the promotional/coupon logic.

While the core assets demonstrated high resilience, the methodology verified the effectiveness of the platform's regional data isolation and session management protocols.

## Key Findings & Logic Tests
1. Cross-Domain Identity Isolation: Verified that sessions and PII (Personally Identifiable Information) are strictly segregated by region. A valid session token for the Canadian domain (.ca) was rejected by the European gateways (.de, .at), preventing cross-border account takeovers.

2. Password Reset Integrity: Tested for email-based IDORs where an attacker might trigger a password reset for a victim in a different region. The backend correctly validated the user's regional context before processing the request.

3. Promotional Logic Resilience: Investigated the cart's vulnerability to race conditions. Attempts to apply single-use coupons across multiple concurrent sessions were successfully mitigated by server-side locking mechanisms.

## Skills Demonstrated
* Global Infrastructure Mapping: Systematic testing of identical logic across geographically dispersed assets.

* Session Integrity Analysis: Evaluating how JWTs and session cookies are scoped to specific TLDs (Top-Level Domains).

* PII Protection Verification: Ensuring that user data reflected in XHR/API responses is strictly limited to the authenticated user.

**Note:** Specific target identifiers and user data have been redacted for confidentiality.

--- 

## Target: [Redacted Global Retailer] Testing Period: November 2025 Tools Used: Burp Suite Professional, PwnFox (Container-based testing), Turbo Intruder

### 1. Scope & Asset Mapping
The target operates a vast network of regional e-commerce sites. My methodology involved mapping the API structure of one region (e.g., api.target.at) and then using Burp Suite's "Find and Replace" to replay those same tests against all other regional assets.

Core Assets Tested: 18+ Regional Domains including Austria, Canada, Italy, Germany, and Norway.

### 2. Technical Process & Logic Tests
#### Test 2.1: Horizontal IDOR in Profile Management
I analyzed the /your-target-overview endpoint which returns a JSON object containing the user's uid, firstName, and email.

**The Attack:**

Log in as Attacker in Container A.

Capture the GET /api/user/details request.

Attempt to swap the ACCESS_TOKEN and CSRF_TOKEN with those belonging to a Victim account.

Result: 200 OK (when using valid tokens). Analysis: The application correctly maps tokens to the specific user session. I attempted to modify the uid in the response locally to see if the UI would pull data for a different user, but the server-side validation remained firm.

#### Test 2.2: Cross-TLD Session Injection
Since the target uses similar subdomains (e.g., www.dyson.at and www.dyson.de), I tested if a cookie from one could be "carried over" to the other to bypass login.

Request:
```
HTTP 
    GET /your-account/overview HTTP/2
    Host: www.target.de
    Cookie: ACCESS_TOKEN=[Token_From_Target_AT]

```
Result: 302 Found (Redirect to Login)

Conclusion: The application uses strict domain scoping for cookies, preventing session hijacking across regional assets.


#### Test 2.3: Password Reset Logic (Email IDOR)
I tested the password reset flow to see if a user's region could be bypassed to reset an account in another country.

The Test:

Submit a reset request on the Austrian site using an email registered on the Canadian site.

Outcome: The Austrian site returned a "Success" message, but no email was sent, and the Canadian account remained untouched.

Analysis: This is a secure implementation. The system prevents "Account Enumeration" by returning a success message regardless of whether the email exists in that specific regional database.

#### 3. Race Condition Analysis (Promotions)
Using Burp Suite Turbo Intruder, I tested the /checkout/v2/apply-coupon endpoint.

Methodology: I attempted to apply a single-use 10% discount code to three different carts simultaneously using a race.py script to ensure all requests hit the server within the same millisecond.

Result:

Request 1: 200 OK (Coupon Applied)

Request 2: 409 Conflict (Coupon Already in Use)

Request 3: 409 Conflict (Coupon Already in Use)

Conclusion: The business logic handles concurrency correctly, preventing multiple-use exploits of single-use vouchers.

#### 4. Final Assessment
The target demonstrates a mature security posture regarding Identity and Access Management (IAM). The primary defense mechanism is the strict regional isolation of user databases and session tokens. No high-severity vulnerabilities were identified, but the assessment confirmed the integrity of the global e-commerce logic.