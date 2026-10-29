# OWASP crAPI Local API Security Assessment

**Prepared:** 2026-10-29  
**Assessment type:** Time-boxed educational lab VAPT  
**Status:** Limited assessment; one finding verified

## 1. Cover page and executive summary

This assessment tested the locally hosted OWASP crAPI training application using two test accounts. The testing confirmed that one signed-in account could retrieve another account's vehicle location and owner details by using that vehicle's identifier. This could expose a person's location information and identifying details in a real deployment. The assessment was limited to the local lab, and only this access-control issue was confirmed; the other vulnerability categories listed in the original plan remain untested.

## 2. Scope and methodology

**Scope note:** Assessment performed against a locally hosted instance of OWASP crAPI, an intentionally vulnerable application, for educational purposes. No production or third-party systems were tested.

**Target:** `http://127.0.0.1:8888`, running in Docker Compose within the Kali VM. MailHog was available at `http://127.0.0.1:8025`. The observed Compose port bindings were on loopback. Nmap was run against `127.0.0.1` only.

**Methodology:** Reviewed the provided OpenAPI specification and Postman collection, identified listening TCP ports with Nmap service detection, and performed manual authenticated cross-account testing. User A (`roll`) owned the vehicle; User B (`rock`) issued the request with its own bearer token. No production or third-party systems were involved.

**Limitations:** The work was time-boxed. JWT forgery, mass assignment, excessive data exposure beyond the observed response, OTP/rate limiting, SSRF, and other BOLA/BFLA scenarios were not assessed. The 44221 loopback listener was attributed by `ss` to `containerd`, but its purpose was not established. Results describe the observed crAPI build and lab state only.

## 3. Findings summary

| # | Title | Severity | CVSS | Status |
| --- | --- | --- | --- | --- |
| 1 | Cross-account access to vehicle location (BOLA) | Medium | 4.3 | Verified |

## 4. Detailed findings

### 1. Cross-account access to vehicle location (BOLA)

**Severity:** Medium  
**CVSS v3.1:** 4.3 - `AV:N/AC:L/PR:L/UI:N/S:U/C:L/I:N/A:N` ([FIRST calculator](https://www.first.org/cvss/calculator/3.1#CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:L/I:N/A:N))  
**Affected endpoint:** `GET /identity/api/v2/vehicle/{vehicleId}/location`  
**OWASP:** API1:2023 Broken Object Level Authorization  
**CWE:** CWE-639, Authorization Bypass Through User-Controlled Key

**Description:** The endpoint returned a vehicle-location object when called by an authenticated account that did not own the referenced vehicle. The response also included the vehicle owner's name and email. The tester authenticated as `rock` and requested a vehicle UUID belonging to `roll`; the API returned HTTP 200 with the vehicle location and `roll` owner fields.

**Proof of concept:**

1. Authenticate as User A and add a vehicle in the local crAPI lab.
2. Capture the vehicle UUID used by User A's location request.
3. Authenticate as User B and send `GET /identity/api/v2/vehicle/{User-A-vehicle-UUID}/location` with User B's bearer token.
4. Observe HTTP 200 and a response containing the target vehicle's location plus User A's owner details.

**Evidence reference:** PDF Appendix, Exhibits 1-3. Exhibit 1 shows the terminal HTTP 200 with the response body excluded; Exhibit 2 shows the location request URL and status; Exhibit 3 shows the User B session header and is cropped to exclude the bearer token.

**Business impact:** In a real deployment, an ordinary authenticated user could access another customer's vehicle location and identifying details. Repeated access to vehicle identifiers could enable unauthorized tracking or disclosure of personal information.

**Remediation:** Enforce object-level authorization on every vehicle-location request. Resolve the authenticated principal from the validated token and query the vehicle using both its identifier and owner identity. Return `403 Forbidden` or a non-enumerable `404 Not Found` for vehicles not owned by the caller. Add regression tests that verify cross-account reads and writes are denied for every vehicle endpoint.

## 5. Appendix

### Tools and data sources

- Kali Linux VM, Docker Engine, Docker Compose
- Nmap 7.99 (`-sV -p- -T4`, target `127.0.0.1`)
- Browser Developer Tools and `curl`
- OWASP crAPI OpenAPI specification and Postman collection
- FIRST CVSS v3.1 calculator

### Timeline

| Phase | Activity | Result |
| --- | --- | --- |
| Setup | Installed Docker and started crAPI Compose stack | Services running; web app reachable |
| Reconnaissance | Full TCP scan of loopback; OpenAPI route inventory | Seven listening ports and 43 API operations recorded |
| Manual testing | Cross-account vehicle-location check | One BOLA finding verified |
| Reporting/tooling | Report and BOLA checker prepared | Script requires local execution with the tester's own account passwords |

### BOLA checker output

The checker is configured to fetch User A's vehicle list, extract a vehicle UUID, then request its location using User B's token. Record its actual run output here after executing it in the Kali VM. No automated run result is claimed in this report.

