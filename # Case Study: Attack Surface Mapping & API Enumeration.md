# Case Study: Attack Surface Mapping & API Enumeration

**Target:** 
European Pet Supply E-Commerce Platform [Redacted]

**Vulnerability Class:** Reconnaissance / Broken Access Control (BAC)

**Severity:** 
Medium (Discovery & Roadmapping)

## Executive Summary
This research phase focused on the initial reconnaissance and mapping of a large-scale e-commerce platform. The objective was to move beyond the main UI and identify hidden API endpoints, unauthenticated administrative paths, and regional variations in security controls across the `.com`, `.co.uk`, and `.de` domains.

By utilizing a systematic enumeration strategy, I successfully mapped the application's core functionality—from authentication to wishlist management—and identified several "Quick Win" targets for deep-dive logic testing.

## Key Findings & Recon Milestones

1.  **API Schema Discovery:** Identified a standardized API versioning pattern (`/api/v1/`, `/api/v2/`) across all regional domains, allowing for predictable endpoint guessing.

2.  **Unauthenticated Endpoint Identification:** Discovered that certain account-related endpoints (like `/wishlist` and `/orders`) were accessible via direct API calls, requiring manual verification for Broken Access Control (BAC).

3.  **Regional Security Parity:** Confirmed that while the UI is localized, the backend API infrastructure is centralized, meaning a vulnerability found on one regional domain likely exists on all others.

## Skills Demonstrated
* **Methodical Enumeration:** Transitioning from "Black Box" discovery to a structured "Site Map."

* **Traffic Analysis:** Using Burp Suite to intercept and catalog background XHR requests that are invisible to standard browser users.

* **Attack Surface Prioritization:** Identifying which endpoints (e.g., `/checkout`, `/settings`) carry the highest risk and merit the most testing time.

---

*Note: Specific target identifiers and user data have been redacted for confidentiality.*

---


**Target:** [Redacted European E-Commerce]

**Testing Period:** October 2025

**Tools Used:** Burp Suite Professional (Target Site Map), Wappalyzer, Ffuf (Directory Fuzzing)

## 1. Asset & Scope Verification
The program scope was restricted to four main domains. My first step was to verify the technology stack using Wappalyzer and initial header analysis.

* **Primary Domains:** 
`target-pets.com`, `target-pets.co.uk`, `target-pets.de`.

* **Constraints:** 
No subdomains allowed; limit of 5 requests per second to avoid WAF triggering.

## 2. Technical Process: Endpoint Mapping
I performed a manual "walkthrough" of the site while proxying traffic through Burp Suite. This allowed me to build a comprehensive map of the backend API.


### 2.1: Authentication Flow
* `/login` / `/logout`

* `/reset-password`

* **Observation:** The site uses a centralized authentication service. I prioritized testing the `reset-password` flow for account enumeration vulnerabilities.



### 2.2: Account Management (XHR/API)
While the URL in the browser remained `/account/overview`, Burp Suite revealed underlying API calls:

* `GET /myaccount/api/order-details/v2/[USER_ID]/lastOrders`

* `GET /myaccount/api/wishlist/v1/[USER_ID]/wishlists`



**Test Performed:** 
I attempted to access these endpoints by stripping the session cookies to see if the backend enforced authentication at the API layer.

* **Result:** `401 Unauthorized` (Secure implementation).




### 2.3: "Quick Win" Target Identification
Based on the mapping, I generated a list of high-priority paths for fuzzing and parameter injection:

| Category | High-Priority Path | Strategy |

| --- | --- | --- |

| **Admin** | `/admin`, `/dashboard` | Fuzzing common directory names |

| **Logic** | `/api/v1/cart/add` | Negative quantity / Price manipulation |

| **Privilege** | `/api/v1/users` | Parameter injection (`isAdmin=true`) |

| **IDOR** | `/api/order-details/v2/[ID]` | Numerical ID incrementing |





## 3. Methodology: Parameter Injection Testing
I targeted the `/wishlist` API to check for vertical privilege escalation.

**Request:**
```
    HTTP
    GET /myaccount/api/wishlist/v1/[USER_ID]/wishlists?isAdmin=true HTTP/2
    Host: target-pets.com
    Cookie: [Valid_User_Session]

```
**Result:** 200 OK (Standard User View). Analysis: The application ignores unrecognized parameters like isAdmin=true rather than crashing or escalating, which indicates a robust input handling logic.


## 4. Conclusion
The reconnaissance phase successfully transformed a broad web application into a structured list of actionable API targets. By mapping the /api/v1/ and /api/v2/ structures, I established a foundation for systematic Broken Access Control testing. The application demonstrates a consistent security posture across all regional domains.