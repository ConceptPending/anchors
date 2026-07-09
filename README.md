# anchors

Public, append-only anchors for the **Baseplate Cloud audit trail**
(`baseplate-control`, the approval/gating control plane).

Every approval and audit event in Baseplate Cloud is hash-chained: each row
carries `row_hash = sha256(prev_hash + canonical(content))`, so editing,
deleting, or reordering any historical record breaks every hash after it.
This repo publishes the **chain heads** on a schedule. A published head is a
commitment to the entire history behind it — once a head is anchored here,
not even the operator of the service can rewrite the records it covers
without the mismatch being detectable.

## Why you can trust this (and what you're trusting)

Honesty first: ConceptPending operates both the service *and* this repo.
The design compensates with witnesses the operator does not control:

1. **Public commits** — anyone can clone this repo; every clone is an
   independent copy of every anchor.
2. **GitHub Actions run logs** — each anchor is taken by a scheduled
   workflow whose logs are retained by GitHub, independent of the repo's
   git history.
3. **OpenTimestamps** — each anchor file is stamped (`.ots` proof alongside
   it), committing its hash to the Bitcoin blockchain. This one is
   non-retractable by anyone, including us. Verify with
   [`ots verify`](https://opentimestamps.org).

## Layout

- `anchors/<timestamp>.json` — one file per observed head change (append-only)
- `anchors/<timestamp>.json.ots` — its OpenTimestamps proof
- `latest.json` — the most recent heads (convenience copy)

## Verifying

- **A stamp:** `ots verify anchors/<ts>.json.ots -f anchors/<ts>.json`
- **The live heads:** `GET https://web-production-8fffa.up.railway.app/api/chain-heads`
  and compare with `latest.json`.
- **The chains behind a head:** full verification recomputes every row hash
  from record content (`scripts/verify_chain.py` in `baseplate-control`);
  clients and auditors are given that output — this repo lets them check it
  wasn't rewritten after the fact.

Anchor `#0` (2026-07-09) was taken manually at the moment the hash chain
first went live in production and is recorded here retroactively as the
chain's starting witness; scheduled anchors begin after it.
