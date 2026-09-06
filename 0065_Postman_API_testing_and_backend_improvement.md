# From Postman Tests to a Stronger Backend: The LiliShop API Testing Story

## Introduction

API testing is most useful when it does more than confirm that an endpoint returns a response. A good API test suite checks the complete behavior that a real client depends on: routing, authentication, authorization, validation, database behavior, response formats, security headers, cookies, file downloads, redirects, external integrations, and cleanup.

That is what happened in the LiliShop project. Building a comprehensive Postman suite did not only produce a collection of HTTP requests. It exposed real weaknesses in the backend. Some endpoints returned the wrong status code. Some error responses lost their security headers. One paginated query could not correctly search and sort user data. Several delete operations depended on database errors instead of checking business rules. Cleanup logic also taught an important lesson about the difference between a disposable resource and a resource that must be retained.

The result was valuable in two directions:

- The Postman suite became safer, more deterministic, and easier to run repeatedly.
- The backend became more correct, more explicit, and easier for clients to understand.

This article explains how the suite is organized, how its requests work together, what its environments and feature flags do, which backend problems it discovered, and why the resulting code changes matter.

The main commits discussed here are:

- [`e8c4b3f` — fix(api): address defects uncovered by Postman integration tests](https://github.com/jahanalem/LiliShop-backend-dotnet/commit/e8c4b3f0a177fe958f52c23737e7a7bb8686b3db)
- [`4d5e34d` — update Postman tests](https://github.com/jahanalem/LiliShop-backend-dotnet/commit/4d5e34d175687398088dcc318d9db60bd2104ac3)

## Why API testing with Postman matters

Unit tests are excellent for testing a class or method in isolation. They can verify business logic quickly and precisely. However, a unit test usually does not send a real HTTP request through the full ASP.NET Core pipeline.

A Postman integration test sees the API from the outside. It can detect problems across several layers at once:

1. The route must match the request.
2. Authentication must read the token or cookie correctly.
3. Authorization must apply the expected role or policy.
4. Model binding and validation must accept or reject the payload.
5. The controller must call the correct service.
6. The service and repository must execute a valid database operation.
7. Error handling must convert failures into the correct HTTP response.
8. Middleware must add the expected headers.
9. Serialization must produce the response shape promised to the client.

This end-to-end view is important because many defects exist between components rather than inside one component. A middleware may work for successful responses but fail for exception responses. A repository method may work when called directly, while the service still calls an older repository path. A delete method may pass its unit tests with mocks but fail against a real relational database because another table references the resource.

Postman is also useful because it is both interactive and automatable. A developer can inspect one request manually, then run the same collection in the Collection Runner or Newman. This allows the suite to serve as:

- executable API documentation;
- a regression suite;
- a local integration test tool;
- a CI-friendly command-line test suite;
- a way to reproduce defects with exact requests and responses.

## From the old collection to a comprehensive suite

The old LiliShop collection was useful as historical documentation, but it was not a reliable regression suite. Its verified inventory was:

- 60 requests;
- 8 top-level folders;
- 44 requests with tests;
- 16 requests without tests;
- 29 requests that resolved authentication through a hardcoded, expired JWT.

It also used fixed IDs, fixed email addresses, fixed basket keys, and routes that no longer matched the source code. Nine current API controllers were missing entirely, including Inventory, Invoices, Languages, Localization, Product Attributes, Product Variants, Roles, Authorization, and Contact Messages.

The new source-derived collection covers all 112 attributed HTTP actions found in the API controllers. Its canonical inventory is:

- 182 total Postman requests;
- 112 endpoint-positive requests;
- 40 explicit negative requests;
- 20 conditional external-integration requests.

The total request count is larger than the endpoint count because one endpoint may need several tests. For example, an endpoint may have a successful request, an invalid-model request, an unauthenticated request, and a cleanup request.

The endpoint coverage matrix is stored in `postman/API_ENDPOINT_COVERAGE.md`. It records the method, full route, authorization rule, request model, success and error statuses, preconditions, test data, and corresponding Postman requests. This matrix prevents silent coverage gaps.

## Structure of the Postman test suite

The collection is ordered as an executable workflow. Earlier folders create the data required by later folders.

| Folder | Main responsibility |
|---|---|
| `00 - Preflight and Configuration` | Validate the environment, generate the run ID, and read basic API configuration |
| `01 - Authentication and Accounts` | Login, registration, refresh cookies, account data, MFA, email-related flows, and user administration |
| `02 - Authorization and Roles` | Verify policy boundaries and role behavior |
| `03 - Product Brands` | Create, read, update, list, and delete brands |
| `04 - Product Types` | Create, read, update, list, and delete product types |
| `05 - Product Attributes` | Manage attributes and values, including validation and delete behavior |
| `06 - Products and Photos` | Create products, read and search products, manage product data, and prepare photo workflows |
| `07 - Product Variants` | Generate, upsert, read, and safely delete variants |
| `08 - Inventory` | Adjust stock and verify inventory transactions |
| `09 - Discounts` | Create and manage discount rules and their catalog dependencies |
| `10 - Basket` | Create baskets, validate stock behavior, and read delivery methods |
| `11 - Payments` | Test payment intents and Stripe webhook rejection/acceptance paths |
| `12 - Orders and Payment Confirmation` | Create and read orders and verify payment state transitions |
| `13 - Invoices` | Test invoice JSON responses and PDF downloads |
| `14 - Languages and Localization` | Test language configuration, localized dictionaries, ETags, and translation administration |
| `15 - Notification Subscriptions` | Test price-drop subscriptions, authorization boundaries, search, sorting, and pagination |
| `16 - Contact Messages` | Test public submission and privileged management workflows |
| `17 - Printess` | Test the public security boundary of the Printess callback |
| `90 - Optional External Integrations` | Hold tests that require Cloudinary, Stripe, email, Google, or Printess infrastructure |
| `99 - Cleanup` | Remove run-created data in reverse dependency order and verify that cleanup succeeded |

This order is part of the test design. A product cannot be created until its brand and type exist. A variant cannot be created until the product and defining attribute values exist. A basket needs a variant. An order needs a basket and, in the Stripe flow, a valid payment state. Cleanup must reverse this order.

## Postman environments in this project

### The canonical local environment template

The main environment file is:

```text
postman/LiliShop.Local.postman_environment.json
```

It targets the Development URL defined by `launchSettings.json`:

```text
http://localhost:6001
```

The file contains variable names and safe default values. Real passwords, MFA secrets, provider tokens, and API keys are deliberately empty. This makes the file safe to commit and share.

### The later exported local environment

Commit `4d5e34d` added another Postman export:

```text
postman/LiliShop Local.postman_environment.json
```

Both files are named **LiliShop Local** inside Postman and both target the same local API. They are not separate Development and Staging environments. The second file is a later export that added the bounded tracking arrays `runAttributeValueIds`, `runVariantIds`, and `runDiscountIds`.

For normal work, one environment should be treated as the canonical template. Importing two environments with the same display name can be confusing because a developer may edit or run the wrong copy. The dot-named canonical file is the one referenced by the repository's Postman workspace configuration and README.

### Active and runtime environments

It is helpful to think about three environment layers:

1. **Committed template** — contains safe defaults and empty secrets.
2. **Active local Postman environment** — contains credentials and runtime values on the developer's machine.
3. **Exported Newman runtime environment** — optionally carries cookies, dynamically captured IDs, or a newly created MFA secret between separate Newman processes.

The runtime export can contain secrets. It must not be committed.

A future Staging environment can reuse the same variable names with a different `baseUrl` and different credentials. Destructive tests should only be enabled against an isolated test database. Production must never be used for this suite's mutation and cleanup lifecycle.

## Environment variables and why they are necessary

The suite uses variables for configuration, secrets, feature flags, runtime identity, and captured resource IDs.

### Configuration variables

| Variable | Purpose |
|---|---|
| `baseUrl` | Base address of the API |
| `authRequestSpacingMs` | Minimum spacing between rate-limited authentication requests |
| `minInterRunDelayMs` | Optional delay between complete runs |
| `allowDestructiveTests` | Explicit non-local safety override for mutation requests |

### Credential and secret variables

Examples include:

- `superAdminEmail`;
- `superAdminPassword`;
- `superAdminMfaSecret`;
- `administratorEmail`;
- `administratorPassword`;
- `administratorMfaSecret`;
- `stripeWebhookSecret`;
- `googleIdToken`;
- email confirmation and password-reset tokens;
- Printess callback and save tokens.

These values are empty in committed files. Access tokens are not copied from an old collection. They are obtained from login responses during the run.

### Runtime variables

The suite captures values such as:

- user IDs and access tokens;
- brand, type, attribute, value, product, and variant IDs;
- discount and discount-tier IDs;
- basket and payment-intent IDs;
- order and invoice IDs;
- localization and contact-message IDs;
- ETags and localization versions.

These variables connect requests without depending on static database IDs.

## Feature flags and conditional tests

Not every test can or should run by default. Some requests mutate data, require a privileged identity, reset MFA, upload a real file, or call an external service. Feature flags make these requirements explicit.

| Flag | Enables |
|---|---|
| `runDestructiveCoreTests` | The main create/update/delete lifecycle |
| `runPrivilegedTests` | Requests that require Administrator or SuperAdmin authorization |
| `runAdministratorIdentityTests` | Tests that specifically require the Administrator account |
| `runMfaTests` | MFA setup, enable, login, and reset workflows |
| `runCloudinaryTests` | Real Cloudinary upload and deletion tests |
| `runStripeTests` | Payment intent, signed webhook, order, and invoice workflow |
| `runEmailTests` | Confirmation, reset, and signed unsubscribe workflows that need captured mail tokens |
| `runGoogleLoginTests` | Login using a valid Google ID token |
| `runPrintessTests` | Printess templates, rendering, and callback workflows |

A guarded request checks both its flags and required variables:

```javascript
const disabledFlags = [
  "runDestructiveCoreTests",
  "runPrivilegedTests",
  "runCloudinaryTests"
].filter(key => String(pm.environment.get(key)).toLowerCase() !== "true");

if (disabledFlags.length) {
  console.warn("Skipped; enable: " + disabledFlags.join(", "));
  pm.execution.skipRequest();
}

const missingVars = ["superAdminToken", "productId"]
  .filter(key => !String(pm.environment.get(key) || "").trim());

if (missingVars.length) {
  console.warn("Skipped; missing: " + missingVars.join(", "));
  pm.execution.skipRequest();
}
```

Skipping is appropriate only when the test is explicitly conditional. If its flag is enabled and its configuration is present, a real API failure must remain visible.

## The non-production safety guard

The collection contains POST, PUT, PATCH, and DELETE requests. A collection-level pre-request script protects non-local environments:

```javascript
const method = String(pm.request.method || "").toUpperCase();
const mutating = ["POST", "PUT", "PATCH", "DELETE"].includes(method);
const rawBaseUrl = pm.variables.replaceIn("{{baseUrl}}").trim();
const hostMatch = rawBaseUrl.match(/^https?:\/\/([^\/:?#]+)/i);
const host = hostMatch ? hostMatch[1].toLowerCase() : "";
const local = host === "localhost" || host === "127.0.0.1" || host === "::1";

if (
  mutating &&
  !local &&
  String(pm.environment.get("allowDestructiveTests")).toLowerCase() !== "true"
) {
  console.error("Safety guard skipped a non-local mutation request");
  pm.execution.skipRequest();
}
```

Local hosts are allowed. A non-local host requires an explicit opt-in. This prevents an accidental environment switch from turning an integration test into a production data-deletion script.

## How the requests work together

### 1. Every run receives a unique identity

The first request creates a timestamp-and-GUID `runId`. It derives a shorter `testDataPrefix` from that value:

```javascript
const id =
  new Date().toISOString().replace(/[-:.TZ]/g, "").slice(0, 14) +
  "-" +
  pm.variables.replaceIn("{{$guid}}");

const shortId = id.replace(/-/g, "").slice(-20).toLowerCase();
const prefix = "pm-" + shortId;

pm.environment.set("runId", id);
pm.environment.set("testDataPrefix", prefix);
pm.environment.set("primaryUserEmail", `${prefix}.primary@example.test`);
pm.environment.set("managedUserEmail", `${prefix}.managed@example.test`);
pm.environment.set("basketId", `${prefix}-basket`);
```

Names, emails, SKUs, discount names, basket keys, and other unique values use this prefix. Therefore, an interrupted previous run cannot cause duplicate-data failures in the next run.

### 2. Authentication is centralized

The collection authenticates each identity once and reuses its token. It does not log in before every protected request.

The main identities are:

- anonymous;
- dynamically registered Standard users;
- Administrator;
- SuperAdmin.

JWT access tokens are captured from login responses. Refresh tokens and device IDs remain in the Postman/Newman cookie jar because the API stores them in HttpOnly cookies.

For MFA-enabled accounts, the collection can generate an RFC 6238 TOTP code from a locally supplied Base32 secret. The secret is never committed. Authentication requests are paced using `authRequestSpacingMs` because the API applies a fixed-window rate limit to authentication and contact endpoints.

### 3. Resources are created in dependency order

The simplified dependency chain looks like this:

```text
users and roles
    ↓
brand + product type + product attributes
    ↓
product
    ↓
product variants + inventory
    ↓
discounts and subscriptions
    ↓
basket
    ↓
payment intent
    ↓
order
    ↓
invoice and PDF
```

Each successful create or lookup request captures the ID needed by later requests. When an endpoint does not return the created representation, the suite resolves the record using an exact, run-unique name rather than guessing its database ID.

### 4. Assertions validate the real contract

The suite avoids weak checks such as “status is below 500.” Each request expects one exact result for its current scenario.

Examples include:

- exact status codes;
- JSON Content-Type before parsing JSON;
- required object properties and data types;
- pagination metadata and result count;
- expected role and identity;
- Problem Details status, title, and actionable detail;
- empty response bodies for `204 No Content`;
- `Location` headers and query parameters for redirects;
- `application/pdf` and a non-empty body for invoice PDFs;
- security headers on successful and error responses;
- correct state changes after update, payment, and webhook requests.

A negative response is a successful test when it is the correct contract. For example, an unauthenticated request should return exactly `401`, and deleting a resource that is still in use should return exactly `409`.

### 5. Cleanup is part of the test, not an afterthought

Every DELETE endpoint receives a dedicated disposable resource. The suite does not delete seed data or arbitrary records.

Cleanup runs in reverse dependency order:

```text
subscriptions
    → basket
    → discounts
    → products and variants
    → attributes and values
    → product type and brand
    → contact/localization data
    → sessions and users
```

An ID is cleared only after the expected success response. Unexpected results are stored in `cleanupFailures`. The final request scans all run-owned ID variables and fails if an unexpected resource remains.

This is important because blindly clearing variables can make a failed cleanup look successful. Retaining the failed ID makes investigation possible.

Orders and invoices are different. They represent historical business records and have no general delete endpoint. If the optional Stripe flow creates an order, its product hierarchy may need to remain because the order references a product variant. The suite records that intentional retention in `retainedHistoricalResources` instead of pretending that cleanup was complete.

## Backend problem 1: security headers disappeared from error responses

One of the most important discoveries involved the security-header middleware in `ConfigureMiddleware()`.

### Before the change

The middleware set the headers before calling the next component:

```csharp
app.Use(async (context, next) =>
{
    var headers = context.Response.Headers;
    headers["X-Content-Type-Options"] = "nosniff";
    headers["X-Frame-Options"] = "DENY";
    headers["Referrer-Policy"] = "strict-origin-when-cross-origin";
    headers["Content-Security-Policy"] = "frame-ancestors 'none'";

    await next();
});
```

This looked reasonable. It also worked for many successful responses. The weakness appeared when a later component threw an exception.

ASP.NET Core's exception handler can clear the downstream response before writing a new Problem Details response. When that happened, headers written directly to the old response could be removed. The client then received a JSON error response without the expected security headers.

### How Postman made the problem visible

The collection asserted the same security headers on every response, including negative responses and server errors:

```javascript
pm.test("Security headers are present", () => {
  pm.expect(pm.response.headers.get("X-Content-Type-Options"))
    .to.eql("nosniff");
  pm.expect(pm.response.headers.get("X-Frame-Options"))
    .to.eql("DENY");
  pm.expect(pm.response.headers.get("Referrer-Policy"))
    .to.eql("strict-origin-when-cross-origin");
});
```

This test failed on error paths such as the failed paginated subscription request and failed delete requests. A browser-based happy-path check might never have noticed the inconsistency.

### After the change

The middleware now registers a callback with `Response.OnStarting()`:

```csharp
app.Use(async (context, next) =>
{
    context.Response.OnStarting(() =>
    {
        var headers = context.Response.Headers;
        headers["X-Content-Type-Options"] = "nosniff";
        headers["X-Frame-Options"] = "DENY";
        headers["Referrer-Policy"] = "strict-origin-when-cross-origin";
        headers["Content-Security-Policy"] = "frame-ancestors 'none'";
        return Task.CompletedTask;
    });

    await next();
});
```

`OnStarting()` runs immediately before ASP.NET Core sends the final response headers. By that time, the exception handler has already replaced the response if necessary. The callback therefore adds the headers to the response that will actually be sent.

This change does **not** hide a `500` error or turn it into success. It only makes the security policy consistent across successful, validation, authorization, and exception responses.

The improvement matters for:

- **security**, because clickjacking and content-sniffing protections remain present;
- **correctness**, because all response paths follow one policy;
- **maintainability**, because the behavior is centralized instead of repeated in controllers.

## Backend problem 2: price-drop subscription pagination used the wrong query path

The request `Get paginated price-drop subscriptions` failed these important assertions:

- `Status is exactly 200`;
- `Subscription page is structured`;
- `Security headers are present`.

The security-header failure had the middleware cause described above. The status and response-structure failures came from the subscription query.

### The earlier service flow

The service created generic specifications over `NotificationSubscription`:

```csharp
var spec = new PriceDropSubscriptionSpecification(specParams);
var countSpec = new PriceDropSubscriptionSpecification(
    specParams,
    returnOnlyCount: true);

var totalItems = await _unitOfWork
    .Repository<NotificationSubscription>()
    .CountAsync(countSpec);

var subscriptions = await _unitOfWork
    .Repository<NotificationSubscription>()
    .ListAsync(spec);

var userIds = subscriptions.Select(s => s.UserId).Distinct().ToList();
var userInfoLookup = await _applicationUserService
    .GetUserInfoLookupAsync(userIds);
```

The problem was that user display names and email addresses live in `AspNetUsers`. The similarly named properties on `NotificationSubscription` are `[NotMapped]`. EF Core cannot translate filtering or sorting on those entity properties into SQL.

Fetching user data after the subscription query was too late. Search, sorting, counting, and paging had already been decided by the wrong query.

### The corrected flow

The service now delegates the complete operation to the specialized repository:

```csharp
var result = await _unitOfWork
    .NotificationSubscriptions
    .GetSubscriptions(specParams);

var data = new Pagination<PriceDropSubscriptionDetailsDto>(
    specParams.PageIndex,
    specParams.PageSize,
    result.TotalCount,
    result.Subscriptions);
```

The repository joins users before applying user-facing behavior:

```csharp
var query = _context.NotificationSubscriptions
    .AsNoTracking()
    .Where(ns =>
        ns.AlertType == AlertType.PriceDrop &&
        ns.ProductId != null)
    .Join(
        _context.Users.AsNoTracking(),
        ns => ns.UserId,
        user => user.Id,
        (ns, user) => new PriceDropSubscriptionDetailsDto
        {
            UserId = ns.UserId,
            DisplayName = user.DisplayName ?? user.UserName ?? string.Empty,
            Email = user.Email ?? string.Empty,
            ProductId = ns.ProductId ?? 0,
            ProductName = ns.Product != null ? ns.Product.Name : string.Empty,
            ProductPrice = ns.Product != null ? ns.Product.Price : 0m,
            PreviousPrice = ns.Product != null ? ns.Product.PreviousPrice : null,
            SubscriptionDate = ns.CreatedDate ?? DateTimeOffset.MinValue
        });
```

Search and sorting now operate on translatable projected fields. The total count is calculated after filtering but before paging. Stable secondary ordering prevents rows with equal display names from moving between pages:

```csharp
var subscriptions = await ordered
    .ThenBy(s => s.ProductId)
    .ThenBy(s => s.UserId)
    .Skip(specParams.PageSize * (specParams.PageIndex - 1))
    .Take(specParams.PageSize)
    .ToListAsync();
```

The code also validates that `PageIndex` and `PageSize` are greater than zero.

These changes improved:

- **correctness**, because the returned count matches the filtered data;
- **reliability**, because EF Core can translate the query;
- **determinism**, because equal sort keys have a stable order;
- **performance**, because read-only queries use `AsNoTracking()` and paging stays in SQL.

SQLite-backed repository tests were added to verify joined-user search, sorting, alert-type filtering, count-before-page behavior, and stable results.

## Backend problem 3: expected delete conflicts appeared as server errors

Several Postman cleanup requests expected `204 No Content` but received failures. The first temptation might be to loosen the Postman assertion. That would have been the wrong fix.

The failures revealed two different situations:

1. The collection selected a resource in the wrong state or cleanup order.
2. The backend represented an expected business conflict as a generic `500` error.

An existing resource that cannot be deleted because another resource uses it is not necessarily a server malfunction. It is normally a conflict between the requested operation and the current state of the system.

### Introducing `ResourceInUse`

A new application error code was added:

```csharp
/// <summary>
/// Indicates the resource exists but cannot be changed or removed
/// while another resource uses it.
/// </summary>
ResourceInUse,
```

The HTTP mapper converts it to `409 Conflict`:

```csharp
{ ErrorCode.ResourceInUse, StatusCodes.Status409Conflict },
```

The response is a structured Problem Details document with an actionable message. For example, a product referenced by an order can tell the client to deactivate it instead of deleting it.

This is better than `500` because:

- clients can distinguish a correct business rejection from an unexpected server failure;
- the response tells an administrator what must be changed first;
- tests can assert a precise contract;
- logs are not filled with avoidable foreign-key exceptions.

## Backend problem 4: delete methods did not check all dependencies

The Postman lifecycle creates real relationships between catalog entities. This revealed missing delete guards.

### Products

Before deletion, the product service now checks:

- whether the product exists;
- whether a discount condition references the product;
- whether any product variant appears on an order.

These checks occur before starting the delete transaction. This avoids unnecessary transaction work and returns a clear `409` response.

### Brands and product types

Brand and type deletion now checks both products and discount conditions. Successful deletion also invalidates the relevant cache tags:

```csharp
await _cacheManagerService
    .InvalidateCacheByTagsAsync(new[] { "brands" });
```

and:

```csharp
await _cacheManagerService
    .InvalidateCacheByTagsAsync(new[] { "types" });
```

The cache invalidation is an important secondary discovery. A database delete can succeed while clients still see stale lookup data. End-to-end testing encourages verification of the state after mutation, which makes stale-cache behavior easier to notice.

### Product attributes and attribute values

Attribute deletion now checks references from:

- discount conditions;
- product variant attribute-value links;
- product photo attribute-value links.

Attribute-value deletion performs the same kind of checks at the value level.

Without these guards, SQL Server could reject the operation with a foreign-key exception. With the guards, the API returns an intentional `409 ResourceInUse` before attempting the delete.

### Product variants

Variant deletion has two important domain rules:

- a product must keep at least one variant;
- a variant that appeared on an order must be retained for order and invoice history.

The earlier implementation returned `DeletionFailed`, which mapped these expected states to `500`.

The corrected code uses `ResourceInUse`:

```csharp
if (siblingCount == 0)
{
    return OperationResult.Failure(
        ErrorCode.ResourceInUse,
        "A product must keep at least one variant.");
}

if (wasOrdered)
{
    return OperationResult.Failure(
        ErrorCode.ResourceInUse,
        "This variant has been ordered and cannot be deleted. " +
        "Deactivate it instead.");
}
```

A disposable sibling still returns exactly `204`.

## Backend problem 5: not every `DbUpdateException` means “resource in use”

Adding a broad `catch (DbUpdateException)` would prevent raw database failures from escaping, but it could create a new defect. A database timeout, deadlock, connection failure, or duplicate-key error is not the same as a foreign-key conflict.

The final implementation uses a narrow exception filter:

```csharp
catch (DbUpdateException ex)
    when (DatabaseExceptionClassifier.IsForeignKeyConstraintViolation(ex))
{
    return OperationResult.Failure(
        ErrorCode.ResourceInUse,
        "The resource cannot be deleted because another resource " +
        "still references it.");
}
```

`DatabaseExceptionClassifier` examines the nested SQL Server exception and recognizes error number `547`, the relevant constraint error for these guarded delete operations.

Tests verify that other important SQL Server error numbers are not classified as resource conflicts:

- `2601` — duplicate key;
- `2627` — unique constraint;
- `1205` — deadlock victim;
- `-2` — command timeout.

Unexpected database failures therefore continue through the normal server-error path. This prevents a serious operational problem from being mislabeled as a harmless client conflict.

## What the follow-up Postman commit added

Commit `4d5e34d` continued the cleanup work in a separately exported collection. It added arrays for tracking all run-created IDs:

- `runDiscountIds`;
- `runVariantIds`;
- `runAttributeValueIds`.

It also added bounded cleanup requests. Bounds such as 20 discount IDs or 100 variant IDs are a useful safety measure. They prevent a malformed environment variable from causing an unbounded delete loop.

The follow-up also captured IDs with validation:

```javascript
const runVariantIds = [
  ...new Set(
    variants
      .map(item => Number(item && item.id))
      .filter(Number.isSafeInteger)
      .filter(id => id > 0)
  )
].sort((a, b) => a - b);
```

This is valuable for interrupted-run recovery because the suite knows exactly which IDs came from its own responses.

However, this iteration exposed another important lesson: **captured does not always mean deletable**. A list of all product variant IDs includes the variant that must remain on the product. It may also include a variant already deleted by its dedicated DELETE test. A cleanup loop that treats every captured ID equally can therefore:

- retry an already deleted variant;
- attempt to delete the product's final required variant;
- encounter an ordered variant that must be preserved.

The canonical cleanup strategy is safer:

- use a dedicated `deleteVariantId` for the DELETE endpoint test;
- preserve `variantId` as the keeper;
- remove a successfully deleted ID from optional tracking arrays;
- let product deletion cascade the remaining unreferenced variants;
- retain ordered catalog history intentionally when an order exists.

This is a good example of a test suite improving through its own failures. A failed cleanup test was not noise. It revealed both an API status-code problem and a flaw in the test-data model.

## External integrations and special response types

A comprehensive suite should not silently omit difficult endpoints. LiliShop includes several workflows that need external infrastructure.

### Stripe

The Stripe branch requires test credentials and a matching webhook secret. The suite can:

- create a payment intent;
- sign a synthetic webhook payload;
- test an invalid signature separately;
- verify an idempotent payment state transition;
- read the resulting order and invoice.

### Cloudinary

The positive upload test uses a small committed PNG fixture. An invalid text fixture is used for a negative upload test. The current backend maps the invalid extension to `500`; the test records this implemented behavior as a known defect instead of pretending that it is a successful `400` contract.

### Email workflows

Confirmation and password-reset tests require access to tokens sent by email. The repository does not expose a local mail-capture API, so these tests remain conditional unless a controlled mail sink is available.

### Google login and Printess

Valid workflows require short-lived or sandbox credentials. Invalid-token and unauthenticated security boundaries can still run without real credentials.

### Redirects and PDFs

Redirect tests intentionally disable automatic redirect following so they can assert the original `302` status and `Location` header. Invoice PDF tests check `application/pdf` and a non-empty body without attempting JSON parsing.

These details matter because a generic JSON test helper is not correct for every endpoint.

## Why unit tests alone did not reveal everything

Unit tests and Postman tests solve different problems.

| Unit test strength | Postman integration test strength |
|---|---|
| Fast and isolated | Exercises the real HTTP pipeline |
| Precise business-logic checks | Verifies routing, middleware, auth, serialization, and status mapping together |
| Easy simulation of rare inputs | Uses real request bodies, cookies, headers, and database relationships |
| Clear failure location | Reveals cross-layer contract failures |

For example:

- A middleware unit test may verify that headers are assigned, but it may not reproduce an exception handler clearing the response.
- A mocked repository can return a list successfully, but it may not reveal that EF Core cannot translate a `[NotMapped]` property into SQL.
- A service test can simulate a successful delete, but it may not create the real foreign-key graph produced by products, variants, discounts, photos, orders, and invoices.
- A controller test can verify `OperationResult`, while a Postman test confirms the final Problem Details JSON and headers seen by a client.

The best result is not to replace unit tests with Postman. It is to use the Postman failure to find the backend weakness, fix the code, and then add a focused unit or repository regression test. That is exactly what the commits did for subscription queries, resource conflicts, variant deletion, and database error classification.

## Running the suite

### Start the API

```powershell
cd Main/LiliShop.API
dotnet run --launch-profile LiliShop.API
```

### Import into Postman

Import:

```text
postman/LiliShop.API.postman_collection.json
postman/LiliShop.Local.postman_environment.json
```

Select **LiliShop Local**. The safe default runs public, read-only, and deterministic negative tests. To run the full local lifecycle, provide the required SuperAdmin values locally and enable:

```text
runPrivilegedTests=true
runDestructiveCoreTests=true
```

Enable external flags only when their test infrastructure is available.

### Run with Newman

From the repository root:

```powershell
npx newman run postman/LiliShop.API.postman_collection.json `
  -e postman/LiliShop.Local.postman_environment.json `
  --working-dir . `
  --reporters cli,junit `
  --reporter-junit-export postman/newman-results.xml
```

Run two consecutive iterations in one process:

```powershell
npx newman run postman/LiliShop.API.postman_collection.json `
  -e postman/LiliShop.Local.postman_environment.json `
  --working-dir . `
  -n 2 `
  --export-environment postman/LiliShop.Local.runtime.postman_environment.json
```

The runtime export may contain secrets and must remain local.

## Validation results and honest limits

The generated artifacts were validated as follows:

- both canonical JSON files parsed successfully;
- the collection conformed to Postman Collection Schema v2.1;
- all 346 embedded scripts compiled as JavaScript;
- Newman imported and executed the collection;
- two consecutive safe-default iterations passed with 52 requests and 210 assertions in total;
- the complete .NET solution passed 337 automated tests after the backend corrections;
- secret scans found no committed JWT, private key, Stripe key, or Google API key.

The fully privileged destructive lifecycle was not claimed as executed without credentials. The committed environment intentionally contains no SuperAdmin password or MFA secret. This distinction is important: “covered but conditional” is not the same as “executed successfully.”

## Known limitations that remain visible

The suite documents problems instead of weakening tests to hide them:

- The invalid photo extension currently becomes `PhotoUploadFailed` and maps to `500`; invalid client input would normally be expected to return `400`.
- The local Swagger document returned `500` during one validation session, although several API endpoints remained healthy.
- A `DiscountManager` role constant exists, but the implemented policy and seed data do not currently provide a separate working DiscountManager identity.
- The repository's compose file includes PostgreSQL, while the API uses EF Core SQL Server; it is not a complete local database setup for this API.
- Email, Google, Stripe, Cloudinary, and Printess success paths still require controlled external infrastructure.

Keeping these limitations visible is more valuable than allowing many unrelated statuses or skipping assertions when fields are missing.

## Lessons learned

### 1. Test the full response, not only the status code

A response can have the correct status while missing required headers or containing the wrong structure. Status, Content-Type, headers, body shape, and important values should be tested together.

### 2. Exact negative tests are valuable

A `409` can be a correct result. A negative test should pass only when the exact expected status and error structure are returned.

### 3. A cleanup failure may reveal a backend design problem

Cleanup is real API usage. Foreign-key failures, wrong status codes, stale caches, and missing domain rules often become visible during cleanup.

### 4. Unique test data is simpler than searching for old data

A new `runId` avoids duplicate emails, names, SKUs, and basket keys. It also makes ownership checks possible.

### 5. Captured data still needs semantic meaning

An ID returned during the run is run-owned, but it may not be safe to delete. The suite must distinguish disposable, keeper, ordered, and intentionally retained resources.

### 6. Feature flags should express real prerequisites

Conditional tests are honest when their flags clearly describe missing credentials or infrastructure. A flag should not be used to hide a deterministic core failure.

### 7. Error classification must be precise

Not every database exception is a resource conflict. Only known constraint failures should become `409`; unexpected database errors must remain visible as server failures.

### 8. Stable sorting is part of deterministic pagination

Sorting by a non-unique field is not enough. Secondary keys prevent records from moving between pages across runs.

### 9. API tests and unit tests should reinforce each other

Postman finds the cross-layer problem. A focused regression test then protects the corrected implementation close to the code.

### 10. Do not change assertions only to make a collection green

The goal is zero false failures, not zero information. A green suite that accepts `200`, `400`, `404`, `409`, or `500` for the same request is not a reliable contract test.

## Conclusion

The LiliShop Postman project became much more than a set of saved requests. It became a structured integration test system with endpoint coverage, role-aware authentication, unique test data, external-integration gates, exact assertions, safe cleanup, and Newman support.

More importantly, it changed the backend for the better. It revealed inconsistent security headers, an invalid EF Core query path, incorrect pagination semantics, missing delete guards, poor conflict status mapping, stale-cache risks, and overly broad database-exception handling. Each useful discovery was turned into a code fix and, where practical, a focused automated regression test.

That is the strongest reason to build comprehensive API tests: they do not only confirm that the application works. They show where the application's real public behavior differs from its intended contract, and they provide a repeatable path for improving it.
