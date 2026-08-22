# Cherry PayLock Clean Independent Repeat — 2026-08-22

## Purpose and scope

This private, redacted record preserves a **clean independent repeat** of the PayLock live test-only lifecycle. The run used a new synthetic fixture and the current test-key route, from the Cherry host, against the already deployed service. It did **not** create a payment, use a card, contact a payment processor or real provider, involve a real user, change configuration, or deploy code.

The evidence demonstrates only the observed behavior of this test-only interface and fixture:

```text
synthetic payment intent → synthetic receipt webhook → provider acknowledgement
→ unlock → EXECUTION_PROVEN / H1 resolution → replay and duplicate-intent rejection
```

It does not demonstrate payment-processor authorization, authenticated provider-webhook origin or signature validation, fulfillment, customer identity, money movement, or production commercial readiness.

## Source and execution provenance

| Field | Retained value |
|---|---|
| Execution host | Cherry bare-metal host; host/IP details retained in the private raw bundle |
| Source repository | `uniqueteamyemen/paylock-core` |
| Source commit | `e52d19864cf3b386ba3d27e6ba52d971a435a8db` |
| Runner SHA-256 | `b6698cf75b60e4ac3395829f0abb45ea3f622741cf9f51513b4973e2da02414e` |
| Raw-bundle manifest SHA-256 | `ac3dfc9778c0c8baa34e0b42a538469a70b36d4eee5ed254c06d8431fbf37e59` |
| Raw manifest file count | 1,020 |
| Runner exit status | 0 |
| Remote SHA-256 verification | 1,020/1,020 `OK` |
| Local SHA-256 verification | 1,020/1,020 `OK` |

The full raw bundle remains retained locally and on the controlled Cherry path. It contains exact commands, stdout/stderr, request/response bodies, headers, environment/provenance, and `SHA256SUMS`; it is intentionally not copied into this redacted repository because it contains operational identifiers and endpoint material.

## Result

All eight expected statuses were observed. The response-derived H1 from `resolve` matched the `user_unlock` H1 in the retained raw bundle. The raw H1 value is not published; its verification is recorded as a 64-character value with a retained SHA-256 in the raw evidence.

See [`summary.tsv`](summary.tsv) and [`integrity-record.txt`](integrity-record.txt). Verify this redacted publication with [`PUBLISHED_SHA256SUMS`](PUBLISHED_SHA256SUMS).

## Relationship to the initial PayLock run

This is an independent clean repeat using a new test fixture. It complements the separate `cherry-paylock-test-only-20260822` record, which contains the first successful test-only flow and negative payment-simulation controls. In that earlier control, a deliberately supplied fake signature field was accepted by the tested route; that is recorded there as **lack of demonstrated signature validation**, not as tamper rejection. This clean repeat does not alter that finding.
