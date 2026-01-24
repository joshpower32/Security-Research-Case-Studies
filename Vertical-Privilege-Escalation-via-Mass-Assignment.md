# Vulnerability Research Note: Vertical Privilege Escalation via Mass Assignment

**Date:** January 23, 2026
**Target Type:** Financial Services / Developer API Portal
**Vulnerability Class:** Broken Access Control (BAC) / Mass Assignment
**Severity:** Critical (P1)
**Status:** Submitted & Sanitized for Records

---

## 1. Executive Summary
A critical **Vertical Privilege Escalation** was identified in the project creation workflow. By exploiting a **Mass Assignment** vulnerability, an authenticated user can inject administrative parameters into a `POST` request to enable restricted services and self-approve them. By impersonating an internal administrator handle discovered via information disclosure, the attacker bypasses the manual review cycle and gains an **Admin** role with high-privilege permissions.

---

## 2. Technical Chain of Events

### Phase 1: Reconnaissance (Metadata Leak)
An audit of the `/services` metadata endpoint revealed internal-only service definitions.
* **Target Service ID:** `7688` (Internal Account Administration).
* **Leak:** The metadata included the handle of the internal authorizer, `[REDACTED_ADMIN_HANDLE]`, which is required for the approval logic.

### Phase 2: The Exploit (Mass Assignment)
The backend project creation endpoint fails to filter or validate administrative fields in the incoming JSON body. An attacker can manually define the project status and the approving entity.

**Vulnerable Endpoint:** `POST /devzone/developers/projects`

**Payload Used:**
```json
{
  "project_name": "Exploit_PoC",
  "service": {
    "serviceId": 7688,
    "status": "APPROVED",
    "requestedBy": "[REDACTED_ADMIN_HANDLE]",
    "approvedBy": "[REDACTED_ADMIN_HANDLE]"
  }
}

Here is the full, sanitized content for your notes in Markdown format. You can copy and paste this directly into any .md editor or your personal knowledge base.

Markdown
# Vulnerability Research Note: Vertical Privilege Escalation via Mass Assignment

**Date:** January 23, 2026
**Target Type:** Financial Services / Developer API Portal
**Vulnerability Class:** Broken Access Control (BAC) / Mass Assignment
**Severity:** Critical (P1)
**Status:** Submitted & Sanitized for Records

---

## 1. Executive Summary
A critical **Vertical Privilege Escalation** was identified in the project creation workflow. By exploiting a **Mass Assignment** vulnerability, an authenticated user can inject administrative parameters into a `POST` request to enable restricted services and self-approve them. By impersonating an internal administrator handle discovered via information disclosure, the attacker bypasses the manual review cycle and gains an **Admin** role with high-privilege permissions.

---

## 2. Technical Chain of Events

### Phase 1: Reconnaissance (Metadata Leak)
An audit of the `/services` metadata endpoint revealed internal-only service definitions.
* **Target Service ID:** `7688` (Internal Account Administration).
* **Leak:** The metadata included the handle of the internal authorizer, `[REDACTED_ADMIN_HANDLE]`, which is required for the approval logic.

### Phase 2: The Exploit (Mass Assignment)
The backend project creation endpoint fails to filter or validate administrative fields in the incoming JSON body. An attacker can manually define the project status and the approving entity.

**Vulnerable Endpoint:** `POST /devzone/developers/projects`

**Payload Used:**
```json
{
  "project_name": "Exploit_PoC",
  "service": {
    "serviceId": 7688,
    "status": "APPROVED",
    "requestedBy": "[REDACTED_ADMIN_HANDLE]",
    "approvedBy": "[REDACTED_ADMIN_HANDLE]"
  }
}

### Phase 3: Impact (Privilege Escalation)
The server trusts the injected values, returning a 200 OK. The attacker’s session is immediately upgraded to the admin role. Confirmed permissions gained include:

User Management: user_add, user_delete (Ability to create persistence or lock out admins).

Infrastructure Control: production_update, production_delete (Direct risk to live services).

Project Hijacking: ownership_transfer (Ability to seize existing partner projects).

## 3. Business Impact Assessment
Audit Integrity: By impersonating a real employee, the attacker poisons the system logs, making the unauthorized actions appear legitimate.

Availability Risk: Unauthorized access to production deletion permissions poses a multi-million dollar risk to transaction volume.

Regulatory Risk: Failure of "Four-Eyes" (dual-authorization) protocols required for financial data handling.

## 4. Remediation Recommendations
Field Filtering: Implement a strict allow-list for the project creation API. Fields like status and approvedBy must never be accepted from the client-side.

Session Validation: The backend should only assign statuses based on server-side logic and verified administrative sessions.

Metadata Hardening: Ensure internal-only service IDs and admin handles are stripped from public-facing API responses.

## 5. Metadata for Tracking
VRT Category: Broken Access Control (BAC) > Insecure Direct Object References (IDOR)

Submission ID: 4f424476-fe3a-4484-810a-9a403d54fb77

Key Indicator: Look for hidden: true flags in JSON responses as a sign of restricted but reachable functionality.