# Documentation verification: recoverable fulfillment in LiliShop

This note accompanies [0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md). It describes documentation verification on 23 October 2026. Application test runs were inspected as retained evidence; no application tests, SQL fixtures, migrations, workers, provider calls or configuration changes were executed by this documentation task.

## Documentation choice and output scope

The existing local article checkout was found through the preceding documentation scripts in the known workspace. Its numbered articles and README index support a standalone companion. Filename 0069 was unused, while the README display sequence ended at 69 for filename 0068; the new entry therefore uses display number 70 without renumbering earlier content. Payment verification and downstream recovery have different records, transactions and operational limits, so extending the already large 0068 article would obscure their boundary.

Outputs are `0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md`, `0069_Documentation_Verification.md` and `0069_Source_Manifest.json` in the documentation repository, plus a minimal README entry and a short historical-scope/companion paragraph in 0068. Prior README edits and untracked articles 0066–0068 are preserved. No unrelated article is overwritten. Working evidence/check scripts remain outside the application and documentation checkouts.

## Source provenance

| Repository | Branch / HEAD at capture | Working tree at capture | Source identification |
| --- | --- | --- | --- |
| Backend | main / `214077e34636c2f4ccdc491ac1120dd5f7ffe72e` | Clean | All 612 retained final-validation file fingerprints matched. |
| Frontend | main / `66273c85a6363d7a06cc8c42b638473c209f1fd6` | Clean | All 434 retained final-validation fingerprints matched, including the earlier status-route correction. |
| Article repository | main / `f5880ab6ac0daf96d68564d1902fdfc4e6193528` | README already modified; 0066, 0067 and 0068 already untracked | Prior files were fingerprinted before documentation edits. No reset/stash/commit was used. |

The owner committed application changes after the implementation run. Backend commit subject is `feat(fulfillment): add recoverable inventory, invoice and email processing`; frontend subject is `feat(fulfillment): add admin recovery queue and customer processing status`. Both commits were confirmed available through the GitHub connector. No application commit was made by this task.

The exact change set was checked using the retained implementation baseline and current commits: backend baseline `5d463095031b0164710e8e0167a238d070993fca`, frontend baseline `52630fc20a4484ddbf0e0042dfe518476fd5ab99`. Current changed paths match the retained A5-only source inventory: 23 backend files and 14 frontend files. Added/modified classification uses that baseline; supporting earlier files are marked reused. Generated migration Designer, model snapshot and translation keys are not omitted, but repetitive generated property/union boilerplate is grouped with its schema/data coverage.

The sanitized [0069_Source_Manifest.json](0069_Source_Manifest.json) lists 53 precise repository-relative source references, pinned URLs and SHA-256 values for both working-tree bytes and committed-blob bytes. Those hashes can differ because Git may normalize line endings; both identify their byte representation explicitly. There are no relevant uncommitted application source files at this capture. Future uncommitted versions require new file hashes and path references until the owner commits them; an old commit link must not be used to imply it contains new content.

Article 0068 was fetched read-only through GitHub and compared with local Git blob identity: both were `aa3595f5845346eb853a9b641aebfa0076ec563d`. Its pre-edit SHA-256 was `f2381d70ff6f73cae9e380ec17558eb3531aa4160be38588adcf38138f5c2ebc`. The article used that matching local/published version. It preserves the earlier design/evidence and adds only a top scope/companion paragraph. The old local documentation HEAD is not falsely used as a permalink for its untracked article.

Applicable instruction files were checked in known ancestors and source/documentation trees; no AGENTS.md was found. Existing project/test documentation, the full A5 design/report, final source inventory/diffs and relevant earlier checkout guide were inspected. No unrelated drive search or application-tree copy was used. Machine paths, credentials, private account data and synthetic database identities are excluded from the public outputs.

## Evidence inspected, without rerunning it

| Retained evidence label | Inspected content / status |
| --- | --- |
| `A5_artifacts/logs/A5-final2-full.trx` | Parsed 445 UnitTestResult records, all Passed; counters executed=445, failed=0, notExecuted=0. Includes 36 earlier checkout SQL cases and 30 fulfillment cases. |
| `backend-final2-release-build.log` and `.result.json` | Exit 0, Release solution build, zero errors; existing six package warnings retained. |
| `backend-final2-full-tests.log` and `.result.json` | Exit 0, full Release run with explicit dedicated SQL fixture environment as recorded. No skipped SQL cases in TRX. |
| `frontend-full-tests.log` and `.result.json` | Exit 0, 67 files / 298 passed, with existing diagnostics retained. |
| `frontend-production-build.log` and `.result.json` | Exit 0; production bundle generated outside source trees. |
| `ef-model-drift.log` and `.result.json` | Exit 0, no changes since migration. This documentation task compares files; it does not run EF. |
| `newman-safe-summary.json` and command metadata | 26 requests, 105 assertions, no failures, all mutation/external flags false. Conditional safe coverage is separate from new fulfillment contracts. |
| `sql-migrate-Fresh.log` and `.result.json` | Successful actual empty-to-current path; 46 migrations. No EnsureCreated substitution. |
| `sql/A5_migration.sql`, migration, Designer and snapshot | Additive schema, exact fields/indexes/FKs/checks and disabled seed compared. |
| Successful retained upgrade JSON snapshots and upgrade test | Real preceding-to-current schema migration, preserved serialized financial/historical-invoice snapshots, repeat initialization, post-expansion partial resume and separate review case. Local DB names/record IDs are not republished. |
| `A5-red.trx`, `regression-before.log`, `A5-upgrade-release-red.trx`, `sql-upgrade-and-release-red.log` | Four initial behavioral failures plus one later stale-legacy-expiry failure before fixes; final corresponding methods passed. Setup/compile failures are not called behavioral red evidence. |
| `sql/final-test-databases.json`, `environment/services.json`, delivery verification | Recorded controls closed and dedicated services stopped after implementation. These are historical inspected records, not a current live-database observation. |

Selected evidence fingerprints are included in the manifest. Full sanitized command arguments, times and working directories remain in the retained local artifacts; public text uses portable labels rather than exposing the author's machine layout. Recorded core commands include Release `dotnet build` / `dotnet test`, Angular `ng test --watch=false` with one-worker runner config, Angular production build, EF model comparison and safe Newman. No command was rerun here. Destructive reset fixtures require new dedicated synthetic databases and must never target ordinary data. The existing local implementation report is the runbook; it is not an invented file in the article repository.

The available source fingerprint comparison ties the documented commits to that evidence. The report's original “uncommitted” statement was true at its own stopping point; the current clean committed source is a later owner action. This note records both rather than treating HEAD alone as proof of an earlier run.

## Source/report qualifications and discrepancies

| Topic | What was qualified in the guide | Effect on interpretation |
| --- | --- | --- |
| Initialization | The report/design sometimes describe initialization as handling eligible work. The actual selection inserts an execution for any missing handoff in its batch; full eligibility is checked by StepAsync. | Pending execution is not permission to apply an effect. Contradictory associations still enter review before stock/invoice effects. |
| Per-item isolation | Maintenance catches inside item loops, but initialization runs outside those catches. | A shared initialization/database error may stop one invocation. This is not a claim of unconditional progress during a database outage. |
| Current processing status | Immutable handoff label remains BlockedAwaitingA5 while execution progresses. | Current status comes from FulfillmentExecution, not mutation of historical creation evidence. |
| Refresh and polling | Customer clears data signals and cancels previous progress reads; it does not reset its message to Checking on every refresh. Admin/customer additions have no new timer, and admin load does not switchMap-cancel older overlapping reads. | Guide describes actual manual refresh and stale-data clearing rather than claiming automatic live polling or a stronger loading-state guarantee. |
| Operator queue visibility | Special mail filter is UnknownDelivery; other values filter execution State. Handoff detail returns only linked audits; control audits have null HandoffId. | No fictional filter for every mail state or dedicated control-audit history endpoint is described. |
| Owner-safe views | Several reads compose a view without a snapshot transaction. | Local effect atomicity is not misrepresented as atomic multi-query UI snapshot consistency. |
| Legacy paths | Expiry double-release is fixed; legacy sale hold-status boundary and best-effort legacy invoice-email enqueue remain. | Guarantees are scoped to the new version-1 consumer. No automatic historical repair or complete legacy rewrite is claimed. |
| Mail behavior | Preparation shares existing PDF/composer/localization; transport uses classified SendGrid without legacy fallback. Correlation ID is not provider idempotency. | No exactly-once delivery, automatic uncertainty resend, delivery webhook, SMTP fallback or provider status lookup is invented. |
| Invoice/PDF details | Builder uses saved order snapshots and existing seller/tax/country behavior; PDF preparation returns rendered bytes. | Earlier money work is not attributed to this change, and no new Cloudinary pipeline or full tax/address feature is claimed. |
| Test coverage | Existing-invoice test resets execution progress, not every missing-outbox/corruption combination. New mail tests inject outcomes rather than exercising actual SendGrid status responses. | These narrower boundaries are explicit; source classification is not mislabeled as external-provider evidence. |

No discrepancy was “fixed” by modifying the application. The guide follows source and explains its effect.

## Complete changed-file coverage

Each path in the exact implementation diff appears below and in the guide. Production methods/properties are expanded in Appendix C/A; generated boilerplate is explicitly grouped. Tests are reference/evidence coverage, not new application APIs.

| Repository / path | Change | Article section and covered behavior |
| --- | --- | --- |
| `backend` / `Main/LiliShop.Domain/Entities/Checkout/FulfillmentRecords.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#appendix-a-schema-fields-and-constraints): New entities and every property; historical handoff is reused, not weakened. |
| `backend` / `Main/LiliShop.Infrastructure/Checkout/FulfillmentService.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#a-lease-is-temporary-permission-to-work): New options/claim/request records and consumer; eligibility, per-line and invoice orchestration, retry/control. |
| `backend` / `Main/LiliShop.Infrastructure/Checkout/FulfillmentEvidence.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#the-worker-proves-eligibility-before-effects): New conflict/grouping types and all four validator methods. |
| `backend` / `Main/LiliShop.Infrastructure/Checkout/FulfillmentMaintenance.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#scheduling-and-controls): New independent poll, scope/batch limits and item catches; initialization exception boundary. |
| `backend` / `Main/LiliShop.Infrastructure/Checkout/InvoiceEmailWorker.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#email-has-its-own-state-machine): New ownership/recovery/preparation/send/result worker and private helpers. |
| `backend` / `Main/LiliShop.Infrastructure/Checkout/InvoiceMailTransport.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#transport-classification-is-deliberately-conservative): New message/result records, both interfaces and classified SendGrid adapter. |
| `backend` / `Main/LiliShop.API/Controllers/FulfillmentController.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#http-contract): New owner/role endpoint actions, wrapper, actor and safe projections. |
| `backend` / `Main/LiliShop.Infrastructure/Data/Config/FulfillmentConfiguration.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#appendix-a-schema-fields-and-constraints): All five configuration overloads; complete keys/FKs/indexes/checks/concurrency/seed. |
| `backend` / `Main/LiliShop.Infrastructure/Data/Migrations/20261023100434_A5_RecoverableFulfillment.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#migration-and-model-artifacts): Up/Down and every changed table/constraint, including existing ledger extension. |
| `backend` / `Main/LiliShop.Infrastructure/Data/Migrations/20261023100434_A5_RecoverableFulfillment.Designer.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#migration-and-model-artifacts): Generated target model checked structurally; repeated property boilerplate grouped with schema, not copied. |
| `backend` / `Main/LiliShop.Infrastructure/Data/Migrations/ShopDbContextModelSnapshot.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#migration-and-model-artifacts): Generated current model compared with migration/SQL; repeated schema boilerplate grouped. |
| `backend` / `Main/LiliShop.Infrastructure/Data/ShopDbContext.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-shopdbcontextcs): New DbSets, changed tracked guards, original/current ledger identity protection; existing save/timestamp behavior distinguished. |
| `backend` / `Main/LiliShop.Domain/Entities/InventoryTransaction.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#inventorytransactions): New nullable OrderItemId; all existing ledger columns/context documented. |
| `backend` / `Main/LiliShop.Infrastructure/Data/Config/InventoryTransactionConfiguration.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#inventorytransactions): Filtered unique line/type index and restrictive line FK; earlier variant/order mapping retained. |
| `backend` / `Main/LiliShop.Infrastructure/Services/InventoryService.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#legacy-expiry-and-the-remaining-legacy-boundary): Changed legacy DeductForOrderAsync and ApplyReleaseAsync; old caller and sale boundary explained. |
| `backend` / `Main/LiliShop.Infrastructure/Services/InvoiceService.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#one-invoice-transaction-also-records-email-work): New constrained issuance plus reused legacy public guard, number allocator and snapshot builder. |
| `backend` / `Main/LiliShop.Infrastructure/Services/InvoiceEmailService.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#email-has-its-own-state-machine): New shared PrepareAsync/interface; guarded legacy send and false-gateway failure; reused culture resolution. |
| `backend` / `Main/LiliShop.API/Extensions/ApplicationServiceExtensions.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#scheduling-and-controls): Concrete scoped service reuse, options/workers/transport bindings. |
| `backend` / `Main/LiliShop.API/Program.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#scheduling-and-controls): Independent minutely job registration; old inbox job remains separate. |
| `backend` / `Main/LiliShop.Infrastructure/Data/SeedData/localization-entries.json` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#localization-and-presentation-limits): 73 English additions and all 6,382 existing entries preserved; no full-language-translation claim. |
| `frontend` / `src/app/shared/models/fulfillment.ts` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#angular-dto-fields): Every new interface/property/union/composition, including narrower projections. |
| `frontend` / `src/app/core/services/fulfillment.service.ts` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-fulfillmentservicets): All six service methods, HttpClient and environment base URL. |
| `frontend` / `src/app/core/i18n/fulfillment-text.ts` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#localization-and-presentation-limits): Central fallback dictionary and fulfillmentText lookup; keys covered as data. |
| `frontend` / `src/app/core/i18n/translation-keys.ts` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#localization-and-presentation-limits): Generated key inventory refreshed; repetitive union entries grouped rather than treated as methods. |
| `frontend` / `src/app/features/admin-area/admin/fulfillment/fulfillment.component.ts` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#an-admin-retry-from-button-to-sql): Injected dependencies, state and all lifecycle/load/action/helper methods. |
| `frontend` / `src/app/features/admin-area/admin/fulfillment/fulfillment.component.html` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#localization-and-presentation-limits): Reason/control/list/detail/audit/progress/invoice widgets and action disabling. |
| `frontend` / `src/app/features/admin-area/admin/fulfillment/fulfillment.routes.ts` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#an-admin-retry-from-button-to-sql): Empty/detail child routes use same standalone component. |
| `frontend` / `src/app/features/admin-area/admin/admin.routes.ts` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#an-admin-retry-from-button-to-sql): New lazy child policy under retained existing admin parent authentication. |
| `frontend` / `src/app/features/admin-area/admin/admin.component.ts` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#localization-and-presentation-limits): New queue nav item, with visibility distinguished from authorization. |
| `frontend` / `src/app/features/user-area/checkout/checkout-success/checkout-success.component.ts` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#a-customer-read-from-route-to-safe-status): New progress request/mapping and translation/subscription handling; actual refresh limits. |
| `frontend` / `src/app/features/user-area/checkout/checkout-success/checkout-success.component.html` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#a-customer-read-from-route-to-safe-status): Distinct processing/email display and preserved order link, no shipment claims. |
| `backend` / `Main/Applications/Lili.Shop.Tests/Services/FulfillmentSqlA5Tests.cs` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#guarantee-to-evidence-matrix): All 30 SQL/HTTP/upgrade cases and fixture boundaries; fake provider/mail distinguished. |
| `backend` / `Main/Applications/Lili.Shop.Tests/Services/InventoryServiceTests.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#guarantee-to-evidence-matrix): Three historical legacy regression additions and relevant changed expectations. |
| `backend` / `Main/Applications/Lili.Shop.Tests/Services/InvoiceEmailServiceTests.cs` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#guarantee-to-evidence-matrix): False-gateway regression and preserved rendering/composition unit context. |
| `frontend` / `src/app/features/admin-area/admin/fulfillment/fulfillment.routes.spec.ts` | Added | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#guarantee-to-evidence-matrix): Nine new Router/service/template/auth/reason/revision/unknown-outcome/control/status cases. |
| `frontend` / `src/app/features/user-area/checkout/checkout-success/checkout-success.component.spec.ts` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#guarantee-to-evidence-matrix): Existing six component cases gain service/translation dependencies; not counted as six new cases. |
| `frontend` / `src/app/features/user-area/checkout/checkout.routes.spec.ts` | Modified | [Section](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#a-customer-read-from-route-to-safe-status): Existing eight Router cases retained; success adds fulfillment GET flush, does not recreate earlier route fix. |

## Field-by-field coverage

Every column of the five new tables and the full changed InventoryTransactions table is listed in the guide with SQL type, nullability, default/generation, writer/reader and purpose. The ledger includes earlier BaseEntity fields; other unchanged stock/order/invoice tables are supporting context, not misidentified new schema.

| Table | Complete field set | Article section |
| --- | --- | --- |
| `FulfillmentExecutions` | `HandoffId`, `State`, `Stage`, `Revision`, `LeaseToken`, `LeaseUntil`, `Attempts`, `CycleAttempts`, `NextAttemptAt`, `Reason`, `CreatedAt`, `UpdatedAt`, `CompletedAt`, `InvoiceId` | [Field reference](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#fulfillmentexecutions) |
| `FulfillmentLines` | `OrderItemId`, `HandoffId`, `ProductVariantId`, `Quantity`, `ReservationId`, `DeductionId`, `CompletedAt` | [Field reference](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#fulfillmentlines) |
| `InvoiceEmailWork` | `Id`, `HandoffId`, `InvoiceId`, `State`, `Revision`, `LeaseToken`, `LeaseUntil`, `Attempts`, `CycleAttempts`, `NextAttemptAt`, `Reason`, `CreatedAt`, `UpdatedAt`, `SendStartedAt`, `AcceptedAt` | [Field reference](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#invoiceemailwork) |
| `FulfillmentControls` | `Id`, `Enabled`, `Revision` | [Field reference](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#fulfillmentcontrols) |
| `FulfillmentAudits` | `Id`, `HandoffId`, `EmailWorkId`, `Action`, `Actor`, `Reason`, `PreviousRevision`, `Revision`, `CreatedAt` | [Field reference](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#fulfillmentaudits) |
| `InventoryTransactions` | `Id`, `ProductVariantId`, `Delta`, `Type`, `Reason`, `OrderId`, `OrderItemId`, `PerformedBy`, `CreatedDate`, `ModifiedDate` | [Field reference](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#inventorytransactions) |

All generated-SQL PK/FK/index/check names are included. All new FKs use Restrict/SQL NO ACTION; the unchanged variant-to-ledger Cascade is labeled separately. Audit HandoffId/EmailWorkId/Actor and ledger OrderId are explicitly application-only references. New state checks do not prove transition order or external delivery. No SQL DEFAULT constraints are invented from CLR values or seed data.

## Method, endpoint and frontend coverage

| Source reference | Significant methods / properties covered |
| --- | --- |
| `Main/LiliShop.Infrastructure/Checkout/FulfillmentService.cs` | `Now`, `IsIsolated`, `EnabledAsync`, `RetryAt`, `Reason`, `InitializeAsync`, `ClaimAsync`, `Touch`, `Own`, `StepAsync`, `RecordFailureAsync`, `RetryAsync`, `ControlAsync`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-fulfillmentservicecs) |
| `Main/LiliShop.Infrastructure/Checkout/FulfillmentEvidence.cs` | `Require`, `LoadAsync`, `CompletedAsync`, `ValidateInvoice`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-fulfillmentevidencecs) |
| `Main/LiliShop.Infrastructure/Checkout/FulfillmentMaintenance.cs` | `RunAsync`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-fulfillmentmaintenancecs) |
| `Main/LiliShop.Infrastructure/Checkout/InvoiceEmailWorker.cs` | `Now`, `Touch`, `Own`, `ClaimAsync`, `ProcessAsync`, `FinishAsync`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-invoiceemailworkercs) |
| `Main/LiliShop.Infrastructure/Checkout/InvoiceMailTransport.cs` | `PrepareAsync`, `SendAsync`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-invoicemailtransportcs) |
| `Main/LiliShop.Infrastructure/Services/InvoiceService.cs` | `IssueForFulfillmentAsync`, `CreateInvoiceForOrderAsync`, `AllocateNextSequenceValueAsync`, `BuildInvoice`, `GetInvoicePdfForAdminAsync`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-invoiceservicecs) |
| `Main/LiliShop.Infrastructure/Services/InvoiceEmailService.cs` | `SendInvoiceEmailAsync`, `PrepareAsync`, `ResolveBuyerCultureAsync`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-invoiceemailservicecs) |
| `Main/LiliShop.Infrastructure/Services/InventoryService.cs` | `DeductForOrderAsync`, `ApplyReleaseAsync`, `ReleaseIntentReservationsAsync`, `ReleaseExpiredReservationsAsync`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-inventoryservicecs) |
| `Main/LiliShop.Infrastructure/Data/ShopDbContext.cs` | `ValidateCheckoutSnapshots`, `SaveChangesAsync`, `SaveChanges`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-shopdbcontextcs) |
| `Main/LiliShop.API/Controllers/FulfillmentController.cs` | `Actor`, `Run`, `Customer`, `List`, `Detail`, `Retry`, `Control`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-fulfillmentcontrollercs) |
| `src/app/core/services/fulfillment.service.ts` | `status`, `list`, `detail`, `retry`, `control`, `updateControl`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-fulfillmentservicets) |
| `src/app/features/admin-area/admin/fulfillment/fulfillment.component.ts` | `t`, `ngOnInit`, `load`, `filter`, `move`, `canRetry`, `retry`, `toggle`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-fulfillmentcomponentts) |
| `src/app/features/user-area/checkout/checkout-success/checkout-success.component.ts` | `t`, `ngOnInit`, `refresh`, `ngOnDestroy`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-checkout-successcomponentts) |
| `src/app/core/i18n/fulfillment-text.ts` | `fulfillmentText`; [contracts](0069_Recoverable_Fulfillment_Inventory_Invoices_and_Email_in_LiliShop.md#methods-in-fulfillment-textts) |

All new entity accessors are grouped with their 58-column field reference; constructor dependencies and value-record properties are in the member appendix. All six Angular HTTP methods, component signals/inputs/lifecycle/actions, DTO fields/unions, route guards and template widgets are covered. Trivial generated union/property boilerplate is the only grouped repetition; no meaningful effect/error boundary is omitted. The six endpoint routes are individually listed in the guide, with two overloaded Control actions described together in the method appendix.

## Critical behavior coverage

All 30 expanded SQL case names below were parsed from the retained final TRX. Every distinct method is mapped to a guarantee in the article; theory rows identify expanded cases rather than double-counting methods.

- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.AmbiguousSendOrLostAcceptedPersistenceNeverAutoResends(persistenceFailure: True)`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.LostEmailIntentSaveRollsBackInvoiceAndNumberThenRecovers`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.InvalidEvidenceBecomesPaidReviewWithoutInvoice(damage: "unverified")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.LocalFailureRollsBackCountersHoldLedgerAndMarker(table: "FulfillmentLines")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.LocalFailureRollsBackCountersHoldLedgerAndMarker(table: "InventoryTransactions")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.StalePreparationCannotSendAfterAnotherWorkerReclaims`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.InvalidEvidenceBecomesPaidReviewWithoutInvoice(damage: "missing_hold")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.ExistingInvoiceProgressIsReconciledWithoutReissue`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.RealHttpOperationsEnforceOwnerRoleReasonRevisionAndSafeCustomerStatus`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.CorruptCompletedEvidenceNeverDeductsAgain`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.InvalidEvidenceBecomesPaidReviewWithoutInvoice(damage: "counter")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.SqlUniquenessAndLegacyEntryPointsProtectEffects`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.VerifiedExpiryPauseAndDuplicateSuccessNeverReleaseConsumedStock`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.AmbiguousSendOrLostAcceptedPersistenceNeverAutoResends(persistenceFailure: False)`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.ThreeLineTerminationResumesMissingLinesAndFencesOldWorker`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.InvalidEvidenceBecomesPaidReviewWithoutInvoice(damage: "owner")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.InvalidEvidenceBecomesPaidReviewWithoutInvoice(damage: "version")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.ConcurrentManualRetryRequiresRevisionAndAppendOnlyAudit`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.RenderFailureAndKnownRejectionRetryOneLogicalMailWithoutOtherEffects`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.InvoiceFailureRetriesWithoutInventoryAndCommitsOneMailInstruction`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.ConcurrentClaimsHaveOneOwnerAndOneCompleteEffect`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.LateTransportAcceptanceCannotOverwriteUnknownAfterLeaseExpiry`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.InvalidEvidenceBecomesPaidReviewWithoutInvoice(damage: "shortage")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.TwoOrdersAdjustmentAndExpirySerializeWithoutOversell`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.LeaseExpiryInsideLocalTransactionRollsBackEveryLineEffect`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.RepeatedInitializationWorkerControlsAndPoisonBackoffPreserveHealthyWork`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.ExhaustedItemsDoNotHideHealthyProcessingOrEmail`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.A5RegressionStaleLegacyExpiryCannotReleaseVerifiedReservedUnitsTwice`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.InvalidEvidenceBecomesPaidReviewWithoutInvoice(damage: "snapshot")`.
- `Lili.Shop.Tests.Services.FulfillmentSqlA5Tests.ActualA4UpgradePreservesVerifiedBlockedMoneyAndHistoricalInvoice`.

The separate four initial unit/in-memory regressions plus the SQL legacy release case are the five historical regressions. The guide identifies nine new Angular Router cases and eight retained status-route cases, with mocks and real route/component boundaries stated. Safe Newman and untested provider paths are separately labeled.

## Diagram edge and relationship review

| Diagram | Source-aligned review |
| --- | --- |
| Top-down overview | Payment boundary produces immutable handoff; repeatable owned line loop precedes invoice/intent and external mail; conflicts can enter review. Arrows do not claim one global transaction. |
| Execution states | Pending/Retry become Processing only when due with budget; owned line/reclaim self-transition; failure/backoff/review; manual eligible review/retry to Pending; only invoice transaction reaches Complete. |
| Per-line flow | Exact evidence can skip a completed line; no prior effect proceeds to locked stock check; contradictions review; write/lease failure rolls back current line transaction. |
| Invoice sequence | Existing control lock and invoice service operate inside one SQL transaction; invoice save precedes final intent/completion commit, without external send inside it. |
| Email sequence | Preparing claim commits before render, Sending commits before external transport, result persists only for current owner. Four participants. |
| Email states | Expired preparation can be reclaimed; expired Sending becomes UnknownDelivery; known rejection retries/reviews; Accepted/UnknownDelivery have no automatic outgoing retry. |
| Execution/invoice/mail ER | Handoff-to-execution PK/FK, optional unique execution invoice, unique required mail execution/invoice FKs. No equality-of-two-invoice-fields claim at SQL level. |
| Line ER | Required execution/item/reservation/deduction FKs with unique saved-item/reservation/deduction identities; variant FK listed separately to keep diagram narrow. |
| Ledger/variant ER | Nullable saved-line FK versus existing required variant FK; ledger OrderId excluded as application-only. Earlier Cascade is explained, not relabeled restrictive. |
| Admin retry interaction | Server role/revision/evidence/idle-state check precedes retry/audit commit; same revision racing request conflicts. No force-complete or uncertain resend. |

Initial Mermaid review was structural/source-only because no compatible local Mermaid parser/renderer or mmdc executable was installed. That review missed six sequence-message labels containing semicolons, which Mermaid interprets as statement separators. The user subsequently reported the resulting GitHub parse errors.

Follow-up rendering check: the original invoice-sequence error was reproduced in Mermaid Live Editor, and all three corrected sequence diagrams rendered successfully with Mermaid 12.2.0. Message-label semicolons were replaced by commas or conjunctions without changing participants, arrows, transaction boundaries or operation order. Screenshots and exact rendered-source hashes are retained in the documentation working evidence directory under diagram-fix. No tools or project dependencies were installed. The online editor was used for validation only; the repository was not committed or published by this correction.

The seven other diagrams retain structural/source review only, so their visual rendering remains unverified. The documentation check now rejects semicolons inside sequence blocks to catch this specific failure before regeneration. Diagrams use narrow top-down flows or at most four sequence participants; ER graphs are split into small groups. No HTML labels, click directives or per-diagram configuration directives are used.

## Documentation checks and preservation

The machine-readable `documentation-checks.json` in the working evidence directory records final Markdown/code-fence/anchor/reference checks, exact source-excerpt comparisons, JSON parse checks, complete changed-file/field/method/test coverage, source-model/SQL comparison and secret/machine-path scans. Local relative links are checked against final intended repository destinations; source links are commit-pinned and the two containing commits were verified through GitHub. External framework links were opened read-only in primary EF/Hangfire/SendGrid documentation. This does not assert that every remote URL will remain reachable forever.

Final verification compares both application repository heads, branches, working-tree states and every captured file fingerprint with the before-state. The delivery check requires exact equality. Original documentation files are also checked unchanged except README and the declared 0068 companion insertion, whose prior content is preserved exactly around the additions. The retained final-verification record confirms the comparisons passed. No application, migration, dependency, configuration or database change was introduced; no commit/push/publish was performed.

## Remaining limits

Visual rendering of the seven non-sequence diagrams remains unverified; the three corrected sequences passed the follow-up online rendering check. Real provider acceptance/delivery, Stripe sandbox/SCA, remote webhooks, external upload paths, identity-provider login and production load/cutover are not established by the retained isolated evidence. The earlier inbox fairness concern and legacy sale transaction boundary remain release dependencies. New UI strings have English fallback rather than complete translations in every language. Source/read-view qualifications above remain documented behavior, not silently repaired application code.

The article and note are ready for the owner to review and commit. Publication, release fixes and operational activation are separate tasks.
