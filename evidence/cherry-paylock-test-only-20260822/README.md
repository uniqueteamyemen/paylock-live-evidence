# Cherry PayLock Test-Only Payment-Simulation Evidence — 22 August 2026

## Claim and execution boundary

This package records a narrow runtime exercise against the existing PayLock service using synthetic test fixtures. The tested lifecycle sequence was:

```text
test payment intent → simulated receipt → optional informational test webhook/provider acknowledgement
→ PayLock verification → test user unlock → exactly one H1-bearing response
```

The exercise demonstrated the **Core lifecycle semantics** listed below. In the clarified PayLock model, the webhook is optional informational intake only: it is not a transfer mechanism, a payment-authorization mechanism, or a core-unlock gate. The exercise did **not** invoke a payment processor, prove a real payment, prove delivery of a digital resource, prove client consumption, or change any deployment or configuration. Raw H0/H1 values, service address, headers, and request/response bodies remain outside this repository.

| Chain element | Classification | Direct retained evidence | Boundary |
| --- | --- | --- | --- |
| Test intent / H0 creation | **Verified** | `success-summary.tsv`: `session_success` returned 200; the raw retained response contained an H0. | Synthetic fixture only; no commercial purchase. |
| Simulated receipt acceptance | **Verified (test receipt only)** | `payment_attestation` returned 200. | This does not validate a payment processor, payment authenticity, or funds settlement. |
| Optional `provider_ack` / webhook intake | **Observed (informational only)** | `provider_ack` and its replay both returned 200. | The request was a harness-generated optional information call, not a signed external provider delivery. It is neither payment transfer nor payment authorization nor a prerequisite for the core unlock path. |
| Test user unlock | **Verified (Core semantics)** | `user_unlock` returned 200; the redacted raw response included status `UNLOCKED`, an H1 field, and `EXECUTION_PROVEN`. | This is not proof that a customer received or consumed bytes. |
| One effective H1 path | **Verified (Core semantics)** | First unlock returned 200; `unlock_replay` and duplicate receipt returned 409. | No concurrent-race test or external access-gate delivery proof was in this run. |
| Cancellation closure | **Verified (Core semantics)** | Cancellation receipt returned 200; later unlock and resolve attempts both returned 409. | Test fixture only. |
| Missing receipt rejection | **Verified (schema-level)** | Missing `receipt_id` returned 400. | Does not prove cryptographic receipt validation. |
| Receipt mismatch rejection | **Verified (frozen-intent binding)** | Different test receipt for the frozen test intent returned 409. | This is a mismatch control, not signed-webhook tamper verification. |
| Modified optional-webhook signature field | **Signature validation not demonstrated** | A matching synthetic receipt with the test-only value `TEST_ONLY_INVALID_SIGNATURE` in a `provider_signature` field returned 200. | The endpoint accepted this synthetic informational request. This run therefore does not support a claim that this optional endpoint validates webhook origin or signature; it does not block the independently exercised core payment-simulation and unlock lifecycle. |
| Unknown reference rejection | **Verified** | A test webhook for a nonexistent reference returned 404. | Does not prove origin authentication. |

> **Do not interpret this run as payment processing, proof of a real provider integration, or proof of digital-resource delivery.** It proves bounded Core behaviour under synthetic inputs. The optional information webhook control is a separate observation and does not determine the core payment-simulation or unlock result.

## Result tables

The first run exercised the success sequence, replay controls, and cancellation closure. The separate negative-controls run added a duplicate payment-intent control and malformed/mismatched receipt controls. Both remote runners recorded exit status `0` and passed their local assertions.

| Package | Source pin | Run exit | Remote manifest SHA-256 | Local re-check |
| --- | --- | ---: | --- | --- |
| Success, replay, cancellation | [`e52d19864cf3b386ba3d27e6ba52d971a435a8db`](https://github.com/uniqueteamyemen/paylock-core/commit/e52d19864cf3b386ba3d27e6ba52d971a435a8db) | 0 | `4f7892522fd8c656a5426169251dd2cf7e55e6c4a42cd4b011d54be63f57306b` | Passed; manifest digest matched byte-for-byte after secure collection. |
| Negative receipt controls | Same service contract; runner separately hashed | 0 | `a30884c6d6068ace41ec444ce765b0bb1786e79c300e956a0d476b0af677e718` | Passed; manifest digest matched byte-for-byte after secure collection. |
| Modified-webhook control | Same live service contract; test-only runner | 0 recorded for the request lifecycle; shell completed `123` after the requests because its SHA-256 inventory used a relative-path defect | `ac8dd7cac097df29235794312e83b544534618680144886f38adac4824841f4e` (recovered inventory) | Passed after one read-only recovery; remote/local recovered-manifest digest matched. No request was replayed. |

The normalized request outcomes are in [`success-summary.tsv`](success-summary.tsv), [`negative-controls-summary.tsv`](negative-controls-summary.tsv), and [`tampered-webhook-control.tsv`](tampered-webhook-control.tsv). A deterministic, redacted terminal capture of the modified-webhook result is retained at [`captures/tampered-webhook-control.png`](captures/tampered-webhook-control.png), while the matching test-only success-completion capture is retained at [`captures/success-completion-capture.txt`](captures/success-completion-capture.txt). Both are companions to raw retained records and do not introduce new evidence. The raw-response locator and SHA-256 for the modified-webhook control are published in [`tampered-webhook-response-reference.txt`](tampered-webhook-response-reference.txt). The raw manifests and raw files are not committed because they contain operational identifiers and unredacted evidence.

## Evidence provenance and integrity

The source and environment records retained privately contain the exact runner command, host identity, OS/kernel, tool versions, source clone record where applicable, original input templates, HTTP headers/bodies, runner stdout/stderr, exit record, and SHA-256 inventory. The collection protocol rebuilt a fresh SHA-256 inventory on Cherry from inside each evidence directory, verified every listed file remotely, securely copied the directory, verified every listed file locally, and compared the manifest digest returned by Cherry with the locally calculated digest. The modified-webhook runner had a post-request manifest-path defect; its raw initial inventory is retained, and its stable completed artifacts were then re-inventoried once without replaying the test request.

The manifest SHA-256 values above identify the exact raw inventory used in that comparison. They are integrity locators, not a substitute for the original files and do not expose live protocol identifiers.

## Smallest evidence-producing next action

The next meaningful step, if an external signed-provider adapter is later introduced, is an **Adapter-only experimental branch** with a signed, resource-ready provider event, opaque private state, an atomic one-time resource ticket, and a first-use/replay/concurrency test. That is a separate integration-hardening campaign, not a condition for the already exercised core lifecycle or for the optional informational webhook.
