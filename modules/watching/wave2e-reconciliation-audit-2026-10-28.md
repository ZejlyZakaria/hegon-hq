# Watching Wave 2E - read-only reconciliation audit, 2026-10-28

Follow-up2026-10-29: remote-only migration finding is resolved by exact local restoration,31/31registry alignment and watchdog watermark update; no production changes. Full fresh replay remains blocked/unvalidated as detailed in migration-alignment-2026-10-29.md. Counts and mismatch statements below describe the original2026-10-28audit snapshot.

Status: AUDIT COMPLETE FOR THE CAPTURED PERSONAL WATCHING DATA; OWNER REVIEW BEFORE ANY CORRECTION. This is not Wave2E implementation or final lock. No application code, migration, production data, Git index or deployment changed.

## Authorization and evidence boundary

Owner approved the scope and explicitly requested an initial complete read-only audit followed by discussion before corrections. Cases use PROVEN (a stored fact/discrepancy is demonstrable), UNKNOWN (no sufficient evidence), INFERRED (an estimate retained as such). PROVEN discrepancy does not automatically mean the personal viewing assertion is false or authorize a repair. Bleach part-year reconciliation is excluded; detecting an already-applied remote migration is an operational finding, not a new reconciliation attempt.

Production access used only SELECT via the dedicated Management API database/query/read-only endpoint, using the previously authorized saved CLI credential in memory. No token output, provider refresh, robot invocation or authenticated Stats navigation/receipt observation. Stored provider facts were read, not externally refreshed. No other person's rows or frozen-module business data audited.

Personal snapshot captured2026-10-28T22:36:26.691057Z (23:36:26 Paris). Provider evidence captured separately shortly afterwards, so these are not an atomic cross-request snapshot. Snapshot hash and provider timestamp are recorded in ignored wave2e-audit-results.json. Results are bounded to that capture; no claim that no concurrent external change is possible.

## Inventory

- 515 personal library rows:372 movies,74 series,69 anime;392 titles contribute first-view consumption.
- 392 accepted legacy consumption snapshots, including frozen runtime/coverage estimates.
- 21 canonical assertions:6 FIRST_WATCH,10 REWATCH,5 COMPLETION;20 active and1 retracted. Precision:15 DAY,1 YEAR,5 UNKNOWN. Date confidence:16 CONFIRMED,5 UNKNOWN.
- 10 compatibility rewatches, all represented by canonical IDs and correctly shadowed: no extra compatibility event counted.
- 4 scoped rating assertions;187 other saved season/cour opinions across52 titles have no corresponding frozen scoped assertion (outside Bleach).
- 143 personal TV/anime TMDB heads;143 accepted/historical referenced snapshots. No missing accepted catalogue for a consumed TV/anime title in this capture. Current AniList grouping cache also included; no new provider lookup performed.
- 3 existing permanent achievement receipts;0 history-only display stubs needed.
- 84 retained synthetic-date provenance markers outside Bleach. The earlier85-target estimate batch included Bleach; its later completion correction changed the current projection. This audit does not propose restoring or reclassifying that excluded row.

## Findings and dispositions

Counts below are observations, not disjoint numbers of erroneous media. One title can occur in multiple categories; season contributions are not titles.

| Finding | Class | Count outside Bleach | Before / evidence | Proposed after this audit |
|---|---|---:|---|---|
| Remote-only migration | PROVEN |1 version | Remote registry has20261028010000, name watching_bleach_part4_viewing_year; local deployable folder/reference stop at20261028000000 | Unchanged; owner reviews repository/registry alignment separately |
| Scope years beyond current position | PROVEN discrepancy; true personal scope UNKNOWN |4 entries /3 titles | Tokyo Ghoul S3=2019 while cursor S2E12; The Seven Deadly Sins S3=2024/S4=2021 while cursor S2E24; Fire Force S2=2025 while cursor S1E18 | Keep all originals; do not delete years or advance position without personal evidence |
| Calendar year differs from season-hour year | PROVEN difference, accepted compatibility |84 contributions /29 titles | A title finish/calendar date can coexist with older season years. E.g. Demon Slayer calendar2024-09-24 and season-hour years2019/2021/2022/2023 | Retain both facts; no redistribution of hours or fabrication of season months |
| Year-only contributions | UNKNOWN month |116 contributions /45 titles | Annual attribution exists but contribution month is absent; some title calendar facts exist independently | Retain partial precision; no filling from release dates or invented months |
| Historical synthetic dates | INFERRED |84 titles | Explicit USER_REQUESTED_ESTIMATE_V1, personal_evidence=NONE, original_watched_at retained | Keep estimates and provenance; never upgrade to confirmed viewing evidence |
| December31 without estimate provenance | UNKNOWN authenticity |29 titles | Stored dates include Silo2024-12-31 and Tokyo Ghoul2019-12-31; shape alone proves nothing about authenticity | Existing dates stay usable under the locked policy; no automatic substitution |
| Legacy opinions without frozen rating assertions | UNKNOWN frozen membership |187 opinions /52 titles | Saved season/cour scores remain readable; historical catalogue membership was not frozen with the opinion | Keep scores; no mass confirmation, copying, erasure or scope expansion |
| Accepted storage sizes differ from current aired counts | PROVEN difference, intentional distinct meanings |3 titles | Silo storage_sizes[10,10,10,1] vs aired[10,10,10,0]; Black Clover[170,26] vs[170,4]; Kuroko[26,26,26] vs[26,26,25] | Keep frozen offsets/counts; storage size or announced capacity is not an aired/viewed count |

### Remote-only migration: concrete evidence and limitation

Read-only registry SELECT returns20261028010000 watching_bleach_part4_viewing_year. Its DO repair body matches the archived scripts/.shots/deferred-bleach-part4-viewing-year.sql after whitespace normalization. Current stored Bleach revision9 and FIRST_WATCH S2E41-48, YEAR2026,192frozen minutes confirm this repair is already present. Therefore the previous documentation saying it had never been applied is superseded by this fresh evidence. This audit cannot identify who applied it or establish its application time from this registry row.

Current actual Bleach first-view allocation is2020=146.4h,2023=16h,2026=4h; all-time166.4h. These are observations, not a new proposal or action. No rollback, reversal, mutation or history repair is proposed automatically. Owner's exclusion remains in force.

Schema counts still match81tables/857columns/225policies/49functions/277indexes/20activecrons because the additional migration is data-only. The local schema-reference checker passes but cannot detect a remote-only version: there are31remote versions versus30local. This is the one concrete operational reconciliation issue requiring an owner decision before declaring2E complete. A later approved repository alignment would preserve the real applied version and update the reference watermark, with no database reapplication or registry repair; this audit deliberately does not perform it.

### No safe personal-data correction proven yet

For the three position/year conflicts, stored evidence does not establish whether the cursor is outdated, the year map is stale, or non-contiguous viewing occurred. The existing position filter already excludes these future stamps. Diagnostic copies removing those four entries produced identical hours, title/episode counts, monthly/daily activity, rating distribution and Top Picks for all-time and every available year. Keep originals pending owner clarification; a cosmetic cleanup would discard evidence without fixing an observed metric.

The other categories are uncertainty or accepted compatibility, not proof of corrupt history. No replacement personal dates, runtime backfill, rating migration or automatic normalization is justified by these results.

## Replayed invariants and baseline

Calculations execute the current application TypeScript, not an independent implementation. The ignored harness exposes the actual computeStats ledger in memory only. Snapshot and provider exports do not modify product files.

- All-time3730.7h = movies653.45 + series1158.2666666666667 + anime1520.1166666666666 + rewatches398.8666666666667.
- Every known minute partitions across2013-2026 annual buckets; no unattributed known hours. Episode annual partition also matches all-time.
- Sum of actual Hours breakdown rows equals the card total, without intermediate rounding.
- All10 compatibility rewatches are shadowed by canonical records. No overlapping active FIRST_WATCH coordinate found, no broken history/rating references, and no duplicate personal work identity found.
- No invalid canonical date precision/range/episode-ID cardinality, future watch date, conflicting simultaneous status flags, out-of-range saved rating, frozen episode ID missing from the accepted current catalogue, or stored watch/release/rewatch chronology conflict detected by the specified checks outside Bleach. No historical truth is inferred merely from passing these checks.
- No annual Top Pick lacks first-view consumption in its year; rewatch-only entries remain excluded. Hannibal2026=8.75 with S1-S2 rated scope.
- Server receipt metrics equal UI achievement live metrics on the same snapshot. Existing receipt evidence retained; no observer or receipt writer invoked, no historical unlock date reconstructed.
- Doubling current runtime metadata on in-memory copies of accepted legacy rows leaves all-time hour slices unchanged: frozen accepted estimates remain authoritative.
- One all-time unknown-duration/scope contribution remains in the excluded Bleach history; none in any dated annual bucket. It was not repaired or hidden.
- Remote schema counts match local watchdog counts. Migration registry equality FAILS only for the remote-only20261028010000 described above; local schema-reference script PASS must not be reported as remote registry synchronization.

## Validation and artifacts

174 existing targeted tests across7files PASS: computeStats, history-consumption, accepted-consumption, season-history, milestone-receipts, provider-ratings and stats-org. Actual snapshot replay and offline-only probes PASS for the domain invariants above. No new product tests were added because no product code changed. TypeScript/lint/build/browser UI checks were not rerun for a documentation/ignored-diagnostic-only audit. No broad suite or SQL write rehearsal necessary before an approved correction exists.

Ignored local artifacts: scripts/.shots/wave2e-current.json; wave2e-provider.json; wave2e-remote-extra-migration.json; wave2e-registry-evidence.json; wave2e-audit-results.json; wave2e-findings.csv; audit-wave2e.cjs; read-wave2e.ps1; wave2e-provider.sql. CSV lists individual evidence, classification and UNCHANGED proposed-after values. These contain personal data and must not be staged. Product repo remains unchanged apart from pre-existing debug.log.

## Next owner discussion

Review the remote-only migration finding first. Then clarify the three anime cursor/year conflicts only if desired; UNKNOWN is a valid final disposition. Do not automatically restore/delete metadata or repair the registry. Any approved correction requires a separate concrete before/reason/after manifest and targeted validation. No production change is authorized by accepting this audit report. Bleach personal part-year reconciliation remains excluded.
