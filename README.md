# OWASP crAPI Local API Security Assessment

This repository contains the sanitized assessment report and editable report source for a time-boxed test of OWASP crAPI running in a local Kali/Docker lab.

- [Assessment report PDF with sanitized screenshot exhibits](report.pdf)
- [Editable report source](report.md)

The assessment confirmed one cross-account vehicle-location BOLA issue (CVSS 3.1: 4.3 Medium). A follow-up checker run in Kali reproduced the issue: User B received HTTP 200 and User A's vehicle data. Credentials, bearer tokens, vehicle IDs, and precise location data are omitted.

Scope was limited to the local training application. Other planned vulnerability categories were not tested. The checker and reconnaissance notes are included in the local project package, not in this repository.

