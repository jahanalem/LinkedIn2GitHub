# Documentation verification: checkout hardening and recovery

Verified on 24 October 2026. This packet is documentation only and remains uncommitted. The article number/filename is `0070`; the README entry is 71, following the existing list where article `0069` is entry 70.

## Source provenance

| Repository | Chosen source | Earlier fulfillment baseline | Working tree before documentation |
| --- | --- | --- | --- |
| Backend | `9b0aca800d363720c4a3c2dc3af263e468a2b19c` | `214077e34636c2f4ccdc491ac1120dd5f7ffe72e` | Clean |
| Frontend | `d9990375008d817797bfbd887fbde9a016413b3a` | `66273c85a6363d7a06cc8c42b638473c209f1fd6` | Clean |
| Documentation | Initial local HEAD `f5880ab6ac0daf96d68564d1902fdfc4e6193528`, later observed `62870a283d866111ca4e9cddf257dbcaba065698` | Existing local checkout | Initially README modified and articles 0066–0069 untracked; repository advanced during drafting, before final review |

No AGENTS.md was found in the known ancestors or these repository trees. No new clone/worktree, reset, stash, commit, push, publishing or deployment was used. Published companion articles 0068/0069 and the two chosen application commits were checked read-only through GitHub. Existing earlier documentation edits were preserved. Public source links are pinned to the application commits, not to the old documentation HEAD, which does not contain these uncommitted articles.

The pre-write preservation check found a change in 0068 relative to the initial capture: its earlier companion/scope paragraph was no longer present. The minimal new companion link was prepared against the latest local bytes; that paragraph was not restored over the changed file. Initial and pre-write hashes are retained in the manifest. README and 0069 matched their initial bytes. The 0069 pre-write raw Git blob matches the published blob; 0068's pre-write body agrees with the fetched published text after line-ending/trailing-blank normalization. These checks preserve the latest local history without treating local documentation HEAD as its publication version.

The documentation repository advanced while drafting to the observed merge commit, and the initial fingerprints of 0066/0067 also differed at final review. This task issued no commit/pull/reset and wrote neither file. Their latest versions were preserved; the manifest records that drift rather than claiming every old documentation byte or documentation HEAD remained fixed. Application HEADs and all tracked application bytes did remain fixed. New companion lines use LF while preserving the existing surrounding article bytes, avoiding Git's trailing-CR warning without rewriting the historical body.

The [source manifest](0070_Source_Manifest.json) inventories 115 changed/reused source files with repository-relative paths, pinned URLs and SHA-256 hashes for working-tree bytes and committed-blob bytes. Their line-ending representations can differ. The comparison covers 85 backend and 10 frontend changed files, including formatting/import changes and generated migration code; no relevant uncommitted application source needed a substitute hash-only reference.

The retained follow-up source inventory matches 623/624 current backend files and all 439 frontend files. Only QuestPdfInvoiceRenderer differs, by the later glyph-setting rename. All current direct backend package versions match the retained full-run dependency metadata. No later package update is falsely covered by an older result.

## Coverage and evidence inspected

All eight named design/report/runbook documents were inspected, including later SMTP addenda. The manifest records their hashes. Those addenda resolve the earlier one-message authorization/SendGrid-only test prerequisite for the successful SMTP validation; they did not change persistent registration.

Retained TRX files were parsed: the two demonstrated initial regressions failed in the red record; the final initial hardening run has 461 passed, zero skipped; the later full run has 481 passed, zero failed/skipped. Frontend final logs record 298 then 302 passes across 67 files. Browser logs distinguish 13 application cases (four real HTTP/SQL plus nine mocked auth cases), three actual Elements passes and one separate retained-attempt continuation. Safe Newman records 26 requests and 105 assertions with all destructive/external flags false. These are inspected execution records, not tests rerun by this writing task.

Provider read/effect/ledger and restart records support ten nonlive objects, four captured purchases of 6,500 EUR minor units each and six unpaid objects. Each capture has one protected handoff, stock-line effect, deduction, invoice and mail intent; unpaid cases have none. Deliberate remote duplicate/delay injection and literal OS browser crash remain untested. Corrected fresh abandonment was not rerun end to end as one case; prior abandonment and separate continuation are kept distinct. Original capture remains unresolved and is taught under a neutral alias.

The SMTP record supports one specifically authorized synthetic message, one Accepted durable work item with Attempts=1/CycleAttempts=1, no accepted-work reclamation and independent PDF receipt in Spam. Its fixture used synthetic financial evidence and a scoped SMTP transport override. No recipient address, message URL/ID, raw provider reference, local database identity or credential is included here. This is not a new send or a claim of Inbox placement.

Security evidence includes before/after resolved graph/audit, patched Pkcs product version and crypto consumer/round-trip checks. Migration evidence includes actual preceding schema and fresh 47-migration expansion, historical equality, COPY_ONLY/CHECKSUM backup, real restore, DBCC and resumed partial work. SQL checksums are not represented as an unread backup-file SHA.

Retained packet checksum manifests were newly verified while writing: A6_artifacts (285 files), A6_followup_artifacts (270 files), A6_smtp_artifacts (30 files). Only the documented manifest members are covered; this does not hash an inaccessible SQL-owned backup file or prove the truth of every prior execution by itself.

## Documentation checks

- Nine selected current code excerpts match their exact source ranges after common indentation removal. The obsolete-property example is explicitly historical. The legacy query excerpt explicitly marks its omitted remainder.
- The complete source inventory, schema field/index/FK distinction, package changes, named regressions and coverage matrix are connected to the narrative and appendices. Generated property repetitions and formatting-only changes have stated exclusions; inherited tables are not attributed to the new migration.
- Every new Mermaid block was rendered using Mermaid Live Editor 12.2.0 and visually inspected. Exact rendered source was compared with the article source hash. No project dependency was installed and no application service was started for rendering. Diagrams were reviewed for transaction/lock order and SQL versus logical associations.
- All article/note relative links and heading fragments, source reference definitions, companion links and image files are checked locally. Pinned source paths resolve in the corresponding exact local commit; the two chosen commits were also read through GitHub. This does not assert a fresh HTTP request to every external URL.
- Synthetic arithmetic reconciles: the three-line tutorial purchase is 2 × 20 + 10 + 10 + 5 = 65 EUR = 6,500 cents; shirt/cap/mug stock becomes 98/11/7 with zero reserved. The separate actual-provider fixture is 2 × 30 + 5 = 65 EUR. The stock guard 5 − 2 < 5 − 1 rejects taking another hold; 100 − 2 − 2 − 95 = 1; eight two-unit sales leave 84. Restore history preserves USD 7 + 3 = 10 and separately unknown money. The tutorial's three-line legacy rollback follows the source boundary; its particular retained hold-save fault fixture had one line.
- Public text/manifest are scanned for private paths, addresses, live secrets and raw operational/provider identifiers. Raw private reports/log bodies are not copied into the public packet. Screenshot previews contain only public synthetic diagrams.

| Diagram | Responsibility | Check | Retained preview |
| --- | --- | --- | --- |
| 1 | overview | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/01-overview.png) |
| 2 | inbox states | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/02-inbox-states.png) |
| 3 | financial fence | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/03-financial-fence.png) |
| 4 | legacy transaction | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/04-legacy-transaction.png) |
| 5 | browser readiness | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/05-browser-readiness.png) |
| 6 | browser verification | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/06-browser-verification.png) |
| 7 | version boundary | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/07-version-boundary.png) |
| 8 | schema relations | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/08-schema-relations.png) |
| 9 | cutover | Exact source parsed/rendered and visually inspected | [preview](0070_Diagrams/09-cutover.png) |

GitHub's published rendering is not asserted: this task does not publish the article, and Mermaid Live Editor's version is the checked renderer. The fitted editor preview is retained for the state graph because its full-screen viewer placed an outer edge beyond the viewport; the underlying Mermaid source parsed/rendered correctly. A successful parse alone was not treated as visual review.

## Current discrepancies and limits

General/legacy mail already uses SendGrid-first with SMTP after a boolean false. The durable invoice worker instead injects IInvoiceMailTransport registered to SendGridInvoiceMailTransport. Its preparation reuses the invoice composer but does not enter IEmailService fallback. The one-message SMTP override succeeded without changing that registration. The unresolved scope is integration of the durable path with the owner's intended primary/fallback policy while preserving ambiguous-outcome safety; it is not absence of fallback throughout the application, a failed SMTP test, or a recommendation to remove SendGrid.

The boolean general adapter catches network exceptions into false, so SMTP fallback can duplicate mail after ambiguous primary acceptance. This is a source-derived risk, not a demonstrated duplicate incident. Future hardening is clearly marked as future review. No transport priority or code was changed while writing.

Original captured expired/reviewed payment reconciliation, general admission implementation and owner deployment/cutover/capacity decisions remain open. General/live release remains blocked; current provider rejects live mode. Earlier resolved provider/SCA/mail/security gates are not repeated as wholly untested blockers.

The full suite preceded the final one-line QuestPDF rename. A subsequent Release build in the preceding maintenance conversation succeeded with zero warnings/errors; no standalone retained build log for that separate check is added to the old sealed execution packet. The documentation does not present it as a new full run or claim all frontend diagnostics vanished. Current glyph semantics were also checked against the installed QuestPDF 2026.9.1 XML documentation; official web settings pages were unavailable during lookup, so they are not cited as fetched evidence.

No executable source/configuration/DI/dependency/migration changes, database connections/writes, payments, email sends, workers, live checkout activation or external state repair were performed. Final preservation checks compare all tracked application bytes and Git states to the initial capture. Documentation remains a local reviewable packet with minimal README/companion additions.
