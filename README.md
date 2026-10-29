# crAPI Local API Security Lab

Broken object-level authorization matters because an API that checks only whether a user is signed in can still expose another customer's records when the caller changes an object ID. This project records a verified cross-account vehicle-location access issue in a locally hosted OWASP crAPI lab and provides a small checker to repeat that ownership test with two test accounts.

## Contents

- [Assessment report](report.pdf) and editable [report source](report.md)
- [Reconnaissance notes](recon.md)
- `bola_tester/` - Python httpx checker for vehicle-location BOLA
- `evidence/` - sanitized evidence summary

## Lab scope

Use only your own local crAPI instance. The tested URL was `http://127.0.0.1:8888`; the observed Docker port bindings were loopback-only. The checker makes requests only to its configured base URL. It does not enumerate IDs or scan networks.

## Run the checker

From this directory, create a virtual environment and install dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install httpx rich pyyaml
python bola_tester/main.py
```

The program prompts for User A and User B email/passwords without writing the passwords to disk. User A must own at least one vehicle. It fetches User A's vehicle list, selects the first vehicle UUID, and requests that vehicle's location using User B's token. A 200 response containing location data is reported as `FAIL (BOLA)`; a 401/403/404 is reported as `PASS`; other outcomes are `INCONCLUSIVE` or `ERROR`. Results are written to `results.json`. Treat that file as lab evidence and review it before sharing.

## Limitations

- The checker currently covers one read-only endpoint: vehicle location.
- It assumes the login response contains a JWT field named `token`, `access_token`, `accessToken`, or `jwt`.
- The script has not yet been run from this Windows workspace against the user's Kali VM. Run it in Kali and record the actual output before claiming automated reproduction.
- A 200 alone is not counted as confirmation unless the response contains location data or matches User A's vehicle identifier/email.
- The assessment report records one manually verified finding. Other planned vulnerability classes were not tested.

## Portfolio/resume wording

Use the measured scope accurately: inventoried 43 OpenAPI operations and seven localhost TCP listeners; manually verified one cross-account vehicle-location BOLA using two lab identities. Add an automated-script result only after running the checker and recording its real output.

