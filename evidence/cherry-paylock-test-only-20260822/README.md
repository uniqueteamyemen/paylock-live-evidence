# Cherry PayLock Test-Only Payment-Simulation Evidence — 22 August 2026

## Claim and execution boundary

This package records a narrow runtime exercise against the existing PayLock service using synthetic test fixtures. The tested core sequence was:

```text
test payment intent → simulated receipt → test webhook/provider acknowledgement
→ PayLock verification → test user unlock → exactly one H1-bearing response
```

The exercise demonstrated the **Core lifecycle semantics** listed below. It did **not** invoke a payment processor, prove a real payment, prove a signed external provider event, prove delivery of a digital resource, prove client consumption, or change any deployment or configuration. Raw H0/H1 values, service address, headers, and request/response bodies remain outside this repository.

| Chain element | Classification | Direct retained evidence | Boundary |
| --- | --- | --- | --- |
| Test intent / H0 creation | **Verified** | `success-summary.tsv`: `session_success` returned 200; the raw retained response contained an H0. | Synthetic fixture only; no commercial purchase. |
| Simulated receipt acceptance | **Verified (test receipt only)** | `payment_attestation` returned 200. | This does not validate a payment processor, payment authenticity, or funds settlement. |
| `provider_ack` input | **Partial** | `provider_ack` and its replay both returned 200. | The request was a harness-generated test call, not a signed external provider delivery. No provider identity, event semantics, or signature validation is demonstrated. |
| Test user unlock | **Verified (Core semantics)** | `user_unlock` returned 200; the redacted raw response included status `UNLOCKED`, an H1 field, and `EXECUTION_PROVEN`. | This is not proof that a customer received or consumed bytes. |
| One effective H1 path | **Verified (Core semantics)** | First unlock returned 200; `unlock_replay` and duplicate receipt returned 409. | No concurrent-race test or external access-gate delivery proof was in this run. |
| Cancellation closure | **Verified (Core semantics)** | Cancellation receipt returned 200; later unlock and resolve attempts both returned 409. | Test fixture only. |
| Missing receipt rejection | **Verified (schema-level)** | Missing `receipt_id` returned 400. | Does not prove cryptographic receipt validation. |
| Receipt mismatch rejection | **Verified (frozen-intent binding)** | Different test receipt for the frozen test intent returned 409. | This is a mismatch control, not signed-webhook tamper verification. |
| Modified webhook signature field | **Contradicted for signature-validation claim** | A matching synthetic receipt with the test-only value `TEST_ONLY_INVALID_SIGNATURE` in a `provider_signature` field returned 200. | The endpoint accepted this synthetic request; this run therefore does not support any claim that webhook origin or signature is validated here. |
| Unknown reference rejection | **Verified** | A test webhook for a nonexistent reference returned 404. | Does not prove origin authentication. |

> **Do not interpret this run as payment processing, provider integration, or proof of digital-resource delivery.** It proves bounded Core behaviour under synthetic inputs and shows which production-facing arrows remain unproven.

## Result tables

The first run exercised the success sequence, replay controls, and cancellation closure. The separate negative-controls run added a duplicate payment-intent control and malformed/mismatched receipt controls. Both remote runners recorded exit status `0` and passed their local assertions.

| Package | Source pin | Run exit | Remote manifest SHA-256 | Local re-check |
| --- | --- | ---: | --- | --- |
| Success, replay, cancellation | [`e52d19864cf3b386ba3d27e6ba52d971a435a8db`](https://github.com/uniqueteamyemen/paylock-core/commit/e52d19864cf3b386ba3d27e6ba52d971a435a8db) | 0 | `4f7892522fd8c656a5426169251dd2cf7e55e6c4a42cd4b011d54be63f57306b` | Passed; manifest digest matched byte-for-byte after secure collection. |
| Negative receipt controls | Same service contract; runner separately hashed | 0 | `a30884c6d6068ace41ec444ce765b0bb1786e79c300e956a0d476b0af677e718` | Passed; manifest digest matched byte-for-byte after secure collection. |
| Modified-webhook control | Same live service contract; test-only runner | 0 recorded for the request lifecycle; shell completed `123` after the requests because its SHA-256 inventory used a relative-path defect | `ac8dd7cac097df29235794312e83b544534618680144886f38adac4824841f4e` (recovered inventory) | Passed after one read-only recovery; remote/local recovered-manifest digest matched. No request was replayed. |

The normalized request outcomes are in [`success-summary.tsv`](success-summary.tsv), [`negative-controls-summary.tsv`](negative-controls-summary.tsv), and [`tampered-webhook-control.tsv`](tampered-webhook-control.tsv). A deterministic, redacted terminal capture of the modified-webhook result is retained at [`captures/tampered-webhook-control.png`](captures/tampered-webhook-control.png); it is a companion to the raw retained record and does not introduce new evidence. The raw manifests and raw files are not committed because they contain operational identifiers and unredacted evidence.

## Evidence provenance and integrity

The source and environment records retained privately contain the exact runner command, host identity, OS/kernel, tool versions, source clone record where applicable, original input templates, HTTP headers/bodies, runner stdout/stderr, exit record, and SHA-256 inventory. The collection protocol rebuilt a fresh SHA-256 inventory on Cherry from inside each evidence directory, verified every listed file remotely, securely copied the directory, verified every listed file locally, and compared the manifest digest returned by Cherry with the locally calculated digest. The modified-webhook runner had a post-request manifest-path defect; its raw initial inventory is retained, and its stable completed artifacts were then re-inventoried once without replaying the test request.

The manifest SHA-256 values above identify the exact raw inventory used in that comparison. They are integrity locators, not a substitute for the original files and do not expose live protocol identifiers.

## Smallest evidence-producing next action

The next meaningful step is an **Adapter-only experimental branch** with a signed, resource-ready provider event, opaque private state, an atomic one-time resource ticket, and a first-use/replay/concurrency test. A payment event alone must not be treated as proof that a named digital resource is ready.
