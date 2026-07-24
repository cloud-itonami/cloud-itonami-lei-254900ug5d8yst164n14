# cloud-itonami-lei-254900ug5d8yst164n14

**独立した第三者による分析/アーカイブであり、Xero Limited と提携・後援関係にありません。**
This is an independent third-party archive/analysis. Not affiliated with, endorsed by, or sponsored by Xero Limited.

The archive itself (`blueprint.edn`, `NOTICE`, `80-data/public/tos.journal.edn`) is
unchanged by anything below — it stays a read-only reference archive; this repo does
not act, propose, or execute anything on the company's behalf using the archive data
itself.

**As of 2026-07-24 this repo ALSO carries a governed actor layered on top of that
archive** (`src/tosmonitor/*`, ToSMonitor-LLM ⊣ ToSArchiveGovernor), added as part of a
third 10-repo validation batch extending the pilot on
`cloud-itonami-lei-2572ibtt8cczw6au4141` (P&G)
([ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn),
[ADR-2607242400](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607242400-cloud-itonami-lei-tos-monitor-actor-batch10-round3.edn),
repo-local record: [`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md)).
This does NOT change the archive's own read-only nature, and it does NOT extend to
every other `cloud-itonami-lei-*` repository — most repos in this family remain plain
archives exactly as ADR-2607110300 describes. See `## Governed actor` below.

## Company

- Legal name: Xero Limited
- LEI: 254900UG5D8YST164N14 ([GLEIF record](https://search.gleif.org/#/record/254900UG5D8YST164N14))
- Jurisdiction: New Zealand
- Website: https://www.xero.com
- Ticker: XRO (NZX, ASX)

## Purpose

This repository archives Xero Limited's publicly published Terms of Use with source-url/retrieved-at/sha256 provenance, per the design established in [ADR-2607110300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.edn) (cloud-itonami-lei-corporate-tos-catalog). See `80-data/public/tos.journal.edn` for the archived text and provenance.

## Governed actor

**ToSMonitor-LLM ⊣ ToSArchiveGovernor**, built on this workspace's
[`langgraph-clj`](https://github.com/com-junkawasaki/langgraph-clj) StateGraph runtime,
modeled directly on
[`cloud-itonami-commitment-ledger`](https://github.com/cloud-itonami/cloud-itonami-commitment-ledger)'s
Store/Registry/Advisor/Governor/Phase/Operation/Sim shape (byte-identical code to the
pilot and every other repo in this batch except `tosmonitor.store`'s company/baseline
data).

Given a candidate ToS/legal-document snapshot, the actor proposes whether it materially
diverges from the archived baseline above, and drafts a change summary — but **it never
writes to the archive itself.** `tosmonitor.store`'s `commit-record!` only writes to
this actor's own Store (an in-process ledger), never to `80-data/public/
tos.journal.edn`. A human archivist decides whether to fold a proposal into the real
archive.

**Single actuation, `:tos/change-proposal`, never autonomous.** Every run — whether the
candidate matches the baseline or diverges from it — carries `:stake :actuation/
archive-update` and is permanently excluded from every rollout phase's auto-commit set.
Six HARD governor checks, each independently re-verified rather than taken on the
advisor's self-report:

| Check | Guards against |
|---|---|
| `grounding-violations` | a cited excerpt not actually present in the candidate's own text (hallucinated quotes) |
| `provenance-incomplete-violations` | a candidate missing full-text/source-url/retrieved-at/sha256 |
| `sha256-mismatch-violations` | a candidate's self-reported sha256 not matching its own content (ground-truth recompute, `:clj`-only in V1) |
| `retrieved-at-not-advancing-violations` | proposing a change from input staler than what is already archived |
| `doc-type-unknown-violations` | a doc-type outside this archive family's own observed vocabulary |
| `source-domain-mismatch-violations` | this actor's own distinctive check — a source-url that doesn't belong to the archived company (misattribution risk specific to an LEI-keyed independent archive) |

```bash
clojure -M:dev:run     # clean lifecycle + all six HARD-hold checks + a phase-0 hold + a MemStore->DatomicStore swap
clojure -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
clojure -M:lint        # clj-kondo (errors fail; CI mirrors this)
```

`clojure -M:dev:run` and the test suite always use the deterministic mock-advisor — no
live fetch of the company's current ToS page and no live `kotoba-server`/CACAO publish
happen anywhere in this actor. `tosmonitor.advisor/llm-advisor` exists as a written,
swappable seam but is not invoked. See
[`docs/adr/0001-tos-monitor-actor.md`](docs/adr/0001-tos-monitor-actor.md) for the full
design record.

## License

Repository structure: AGPL-3.0-or-later (see LICENSE). Archived third-party text remains the copyright of the source company — see NOTICE.
