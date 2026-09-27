# Duva conformance fixtures

Machine-readable fixtures every Duva client library must satisfy. Generated from the real Duva
service (webhook signer, request models, retry policy) — never a hand-written copy — by
`scripts/gen_conformance.py` in the `duva` repository (private; this repository mirrors its output).

- **`webhooks.json`** — HMAC-signed webhook payloads. Each case's `expect` is what `verify(secret,
  headers, body)` must return. ⚠️ **Mock your clock to `reference_now`** (a signature expires
  `tolerance_seconds` after it was signed, by design, to prevent replay): using your real current
  time will make every case fail once `reference_now` is in the past. `signed_with` is
  informational only (which secret produced the signature); always verify with the top-level
  `secret`, except `category: "rotation"` cases, which need a verifier that tries several active
  secrets at once (`other_valid_secret`).
- **`requests.json`** — for each API operation with a JSON body, the exact HTTP request (method,
  path, headers, body) a correct client must send for the given `input`. Bodies were validated
  against Duva's real pydantic request models at generation time.
- **`retries.json`** — retry scenarios (`docs/bibliotheques-clientes.md` §3.5 of the `duva` repo):
  a sequence of HTTP responses (or `status: null` for a network failure with no response) and how
  many attempts, in total, a conforming client should make before giving up or succeeding.

## Using these fixtures

Load the JSON, drive your library against each case, and assert the outcome matches `expect` /
`expected_request` / `expected_attempts` and `expected_outcome`. No network access is required:
`requests.json` and `retries.json` describe requests and responses as data; `webhooks.json` is
fully self-contained (pre-signed) once your test clock is mocked to `reference_now`.

## Regenerating

These files are generated, not hand-edited. From the `duva` repository:

```bash
uv run python scripts/gen_conformance.py          # writes conformance/*.json
uv run python scripts/gen_conformance.py --check  # fails if this mirror is stale
```

Copy the three JSON files here after regenerating and open a pull request.
