# Case Study: Multi-Layered API Access Control Analysis

**Target:** Global Content Management & Cloud Hosting Provider [Redacted]

**Vulnerability Class:** Broken Access Control (BAC) / IDOR

**Severity:** Medium (Research & Logic Verification)

## Executive Summary
This assessment focused on the authorization logic of a major CMS provider, specifically targeting the **REST API infrastructure** that powers the modern "Block Editor" and centralized "Reader" services. 

The objective was to determine if state-changing operations or private data retrieval could be performed by bypassing the primary Web UI through direct API manipulation. While the target demonstrated high resilience, the methodology identified and tested multiple "logic-heavy" endpoints that are often overlooked in standard vulnerability scans.

## Key Findings & Methodology

### 1. Cross-Service IDOR Testing (Editor vs. Reader)
The application handles "Posts" through two distinct API layers: the **Core Post API** (for editing/management) and the **Reader API** (for social discovery). 
* **The Theory:** A private post restricted on the Core API might be "leaked" through the Reader API due to inconsistent synchronization of "Private" status flags across microservices.
* **The Test:** Identified a private Post ID in a "Victim" session and attempted to fetch the JSON object via the Reader endpoint (`/rest/v1.1/read/sites/[SITE_ID]/posts/[POST_ID]`) using an unrelated "Attacker" session.
* **Outcome:** `403 Forbidden` / `{"error":"unauthorized"}`. The backend correctly enforces object-level permissions across disparate microservices.

### 2. Context-Based Authorization Bypass Attempts
I investigated whether "lowering" the requested data context could trick the API into returning restricted content by triggering a weaker security check.
* **The Logic:** Changing `context=edit` (stricter check) to `context=view` (standard check) to see if the server fails to verify the "Private" status of the object for "view-only" requests.
* **The Test:** Manipulated query parameters in Burp Repeater to request restricted objects under various contexts, including `view`, `embed`, and the omission of the context parameter entirely.
* **Outcome:** Secure. The server utilizes a unified permission-middleware that validates ownership regardless of the requested data context.

### 3. API "Batch" Request Analysis
Investigated the **Batch API** (`/rest/v1/batch`), which allows multiple REST requests to be bundled into a single HTTP envelope.
* **The Strategy:** Batch processors sometimes fail to apply the same security filters as individual endpoint controllers, potentially allowing "nested" requests to bypass WAF or API gateway restrictions.
* **The Test:** Crafted a batch request containing a "private" URL path inside the JSON envelope to see if the batch handler would execute the request without checking the attacker session's relationship to the target Site ID.
* **Outcome:** Secure. The batch handler correctly inherited the user's session restrictions for all nested URLs.

## Skills Demonstrated
* **Advanced Session Management:** Utilized isolated browser containers to maintain distinct "Victim" and "Attacker" environments, eliminating session-leakage false positives.
* **API Mapping:** Identified background API calls to `public-api.wordpress.com` that operate independently of the primary dashboard UI.
* **Defense-in-Depth Analysis:** Verified that security controls were applied at the database level, the API gateway, and the individual service controllers.

## Conclusion
The assessment confirmed a mature security posture. The primary strength of the target lies in its **Centralized Authorization Service**, which ensures that regardless of which API layer (Reader, Core, or Batch) is used to access data, the user's relationship to the object is verified. This methodology remains a high-value approach for identifying leaks in complex, multi-service SaaS environments.