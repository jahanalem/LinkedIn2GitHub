


# Discount System – Service Architecture Class Diagram

**Version:** 1.0  
**Project:** LiliShop – Discount System  
**Date:** 2026-05-16

---

## 1. Introduction

After refactoring the original “God Class” `DiscountService`, the discount logic of LiliShop was split into four focused services, each defined by a clear interface. The class diagram below captures this architecture: it shows the interfaces (abstractions), their concrete implementations, and the dependency relationships between them.

This document explains the diagram, decodes the visual notation, and provides a detailed description of every service and interface. Use it as the authoritative reference for the discount system's service layer.

---

## 2. Class Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'background': '#ffffff',
    'primaryTextColor': '#2C3E50',
    'lineColor': '#7F8C8D',
    'fontFamily': 'Arial, Helvetica, sans-serif'
  }
}}%%
classDiagram
    direction TB

    %% INTERFACES
    class IDiscountCrudService {
        <<interface>>
        +GetPaginatedDiscountsAsync(discountParams) IOperationResult
        +GetDiscountByIdAsync(discountId) IOperationResult
        +CreateDiscountAsync(dto) IOperationResult
        +UpdateDiscountAsync(dto) IOperationResult
        +DeleteDiscountAsync(id) IOperationResult
    }

    class IDiscountQueryService {
        <<interface>>
        +BuildEffectiveDiscountsQuery(prod, asOf, discountId) IQueryable
        +GetProductsByDiscountIdAsync(discountId) IReadOnlyList
        +GetProductsCountByDiscountIdAsync(discountId) int
        +GetProductsByDiscountIdWithPagingAsync(...) Tuple
        +GetSingleDiscountForProduct(productId) IOperationResult
    }

    class IDiscountPriceService {
        <<interface>>
        +ResolveTierForProduct(discount, product) DiscountTier
        +CalculateEffectivePrice(product, tier) decimal
        +ApplyBestDiscountsToRestoredProductsAsync(products, excludeId) Task
        +RestorePricesForAffectedProductsAsync(discountId) List~Product~
        +GetProductsAffectedByDiscountGroupAsync(group) List~Product~
    }

    class IDiscountLifecycleService {
        <<interface>>
        +CreateDiscountAndNotifySubscribers...(dto) IOperationResult
        +ActivateDiscountByIdAsync(discountId) IOperationResult
        +DeactivateDiscountByIdAsync(discountId) IOperationResult
        +UpdateDiscountAndNotifyAsync(dto) IOperationResult
        +SweepExpiredDiscountsAsync() IOperationResult
        +DeleteDiscountAndCleanUpAsync(id) IOperationResult
    }

    %% CONCRETE IMPLEMENTATIONS
    class DiscountCrudService {
        -_unitOfWork: IUnitOfWork
        -_httpContextAccessor: IHttpContextAccessor
        -_logger: ILogger
        -CreateAuditLog() DiscountAuditLog
    }

    class DiscountQueryService {
        -_unitOfWork: IUnitOfWork
        -BuildDiscountedProductsQuery() IQueryable
    }

    class DiscountPriceService {
        -_unitOfWork: IUnitOfWork
        -_logger: ILogger
        -DoesConditionMatchProduct() bool
    }

    class DiscountLifecycleService {
        -_crudService: IDiscountCrudService
        -_queryService: IDiscountQueryService
        -_priceService: IDiscountPriceService
        -_unitOfWork: IUnitOfWork
        -_cacheManager: ICacheManagerService
        -_notificationService: Lazy~INotificationService~
        -InvalidateDiscountCacheAsync() Task
    }

    %% RELATIONSHIPS
    DiscountCrudService ..|> IDiscountCrudService
    DiscountQueryService ..|> IDiscountQueryService
    DiscountPriceService ..|> IDiscountPriceService
    DiscountLifecycleService ..|> IDiscountLifecycleService

    DiscountLifecycleService --> IDiscountCrudService : Injects
    DiscountLifecycleService --> IDiscountQueryService : Injects
    DiscountLifecycleService --> IDiscountPriceService : Injects

    %% STYLING
    style IDiscountCrudService fill:#EAFAF1,stroke:#27AE60,stroke-width:2px,color:#2C3E50
    style IDiscountQueryService fill:#EBF5FB,stroke:#2980B9,stroke-width:2px,color:#2C3E50
    style IDiscountPriceService fill:#F5EEF8,stroke:#8E44AD,stroke-width:2px,color:#2C3E50
    style IDiscountLifecycleService fill:#FEF5E7,stroke:#E67E22,stroke-width:2px,color:#2C3E50

    style DiscountCrudService fill:#ffffff,stroke:#27AE60,stroke-width:2px,stroke-dasharray: 5 5,color:#2C3E50
    style DiscountQueryService fill:#ffffff,stroke:#2980B9,stroke-width:2px,stroke-dasharray: 5 5,color:#2C3E50
    style DiscountPriceService fill:#ffffff,stroke:#8E44AD,stroke-width:2px,stroke-dasharray: 5 5,color:#2C3E50
    style DiscountLifecycleService fill:#ffffff,stroke:#E67E22,stroke-width:3px,color:#2C3E50
```

---

## 3. Understanding the Diagram Notation

### 3.1 Boxes – Classes and Interfaces

- **Solid boxes with solid borders** = **Interfaces** (e.g., `IDiscountCrudService`).  
  They are tinted with a distinct colour to visually group them. The `<<interface>>` stereotype confirms their role.
- **Solid boxes with dashed borders** = **Concrete service classes** (e.g., `DiscountCrudService`).  
  They have a white background for clarity. The dashed stroke indicates “implementation detail” – the outside world should depend on the interface, not these classes.
- **The Lifecycle orchestrator** (`DiscountLifecycleService`) uses a **thicker solid border** to emphasize its special role as the central coordinator.

### 3.2 Arrows and Lines

The diagram uses two kinds of relationships:

| Relationship | Mermaid Syntax | Appearance | Meaning |
|--------------|----------------|------------|---------|
| **Interface Implementation** | `..|>` | Dashed line with a hollow triangle arrow | The concrete class implements the interface. E.g., `DiscountCrudService` realises `IDiscountCrudService`. |
| **Dependency / Injection** | `-->` | Solid line with an open arrowhead | One class depends on another (typically via constructor injection). The arrow points from the dependent class to the dependency. E.g., `DiscountLifecycleService` depends on `IDiscountCrudService`. |

In the Mermaid code, the implementation arrows are the four lines:
```
DiscountCrudService ..|> IDiscountCrudService
...
```
The injection arrows are:
```
DiscountLifecycleService --> IDiscountCrudService : Injects
...
```

The label `Injects` reminds us that the dependency is provided through Dependency Injection.

### 3.3 Direction and Layout

The keyword `direction TB` (top‑to‑bottom) instructs the renderer to arrange classes vertically. Interfaces appear at the top, concrete implementations below, and the orchestrator at the bottom. This layout naturally shows the abstraction stack.

### 3.4 Color Scheme

Each interface and its matching implementation share a colour family, making it easy to trace which class belongs to which contract:

- **Green** (`#27AE60`) → CRUD operations.
- **Blue** (`#2980B9`) → Queries and rule engine.
- **Purple** (`#8E44AD`) → Price calculations and tier logic.
- **Orange** (`#E67E22`) → Lifecycle orchestration and scheduling.

---

## 4. Interface and Class Descriptions

### 4.1 `IDiscountCrudService` / `DiscountCrudService`

**Purpose:** Pure persistence layer. Handles creation, reading, updating, and deleting of discount entities, along with audit logging.

**Key Members:**
- `GetPaginatedDiscountsAsync(DiscountSpecParams)` – Returns a paginated list of discounts using specifications.
- `GetDiscountByIdAsync(int discountId)` – Loads a discount with all its details (tiers, groups, conditions).
- `CreateDiscountAsync(CreateDiscountDto)` – Persists a new discount, including its tier, group, and condition hierarchy.
- `UpdateDiscountAsync(UpdateDiscountDto)` – Updates discount properties, condition groups, tiers; manages audit log creation. Runs within a database transaction.
- `DeleteDiscountAsync(int id)` – Removes a discount and its child entities, cancels background jobs, restores prices (if needed), and cleans up the discount group.

**Key Dependencies:**
- `IUnitOfWork` – For database access.
- `IHttpContextAccessor` – To read the current user for audit logs.

**Note:** This service does **not** schedule Hangfire jobs, touch caches, or send notifications. Those are the Lifecycle service’s responsibilities.

---

### 4.2 `IDiscountQueryService` / `DiscountQueryService`

**Purpose:** All read‑only queries needed for the discount system. It builds complex queries that determine which products are covered by a discount and which discounts apply to a product.

**Key Members:**
- `BuildEffectiveDiscountsQuery(Product, DateTimeOffset, int?)` – Returns an `IQueryable<Discount>` that finds all active discounts matching a given product at a specific time. Used extensively by the price service and the rule engine.
- `GetProductsByDiscountIdAsync(int)` – Retrieves the list of products affected by a specific discount (either directly or via condition groups).
- `GetProductsCountByDiscountIdAsync(int)` – Returns the count of such products.
- `GetProductsByDiscountIdWithPagingAsync(...)` – Paginated version of the above.
- `GetSingleDiscountForProduct(int productId)` – Fetches the single, direct discount (if any) assigned to a product.

**Key Dependencies:**
- `IUnitOfWork` – For access to `Product`, `Discount`, `ProductDiscount`, `ConditionGroup`, `DiscountGroupCondition`, and `ProductCharacteristic` tables.

**Note:** All query methods use `AsNoTracking()` to avoid side‑effects on the change tracker. This service is side‑effect‑free.

---

### 4.3 `IDiscountPriceService` / `DiscountPriceService`

**Purpose:** Pure domain logic. It knows how to match a discount to a product (tier resolution), calculate the final price, and apply the best price across multiple overlapping discounts. It also contains the price restoration and re‑application logic with concurrency retries.

**Key Members:**
- `ResolveTierForProduct(Discount, Product)` – Determines which `DiscountTier` (if any) applies to the given product by evaluating all conditions in the discount’s group.
- `CalculateEffectivePrice(Product, DiscountTier)` – Applies the tier’s amount/percentage to the product’s base price (`PreviousPrice ?? Price`).
- `ApplyBestDiscountsToRestoredProductsAsync(List<Product>, int? excludeDiscountId)` – The core “best price” loop. For a batch of products, it loads all active discounts, resolves tiers, calculates prices, and updates each product with the lowest found price. It also handles `DbUpdateConcurrencyException` with a retry loop (using the `RowVersion` column).
- `RestorePricesForAffectedProductsAsync(int discountId)` – Reverts products back to their original price (`Price = PreviousPrice`, `PreviousPrice = null`) for a given discount, with concurrency retries.
- `GetProductsAffectedByDiscountGroupAsync(DiscountGroup)` – Queries products that match a set of condition groups.

**Key Dependencies:**
- `IUnitOfWork` – To load products, discounts, tiers, conditions, and to persist price changes.
- `ILogger` – For logging concurrency conflicts.

**Note:** This service contains the critical “safety net” logic that guarantees a product always shows the best available price and never a stale one.

---

### 4.4 `IDiscountLifecycleService` / `DiscountLifecycleService`

**Purpose:** The orchestrator. It coordinates the other three services and handles all external concerns: Hangfire job scheduling, cache invalidation, and notification triggering. It implements the complete workflows for activation, deactivation, and live updates.

**Key Members:**
- `CreateDiscountAndNotifySubscribersAboutNewDiscountAsync(CreateDiscountDto)` – Creates a discount, schedules activation/deactivation jobs, and, if the discount is already active, immediately applies prices and notifies subscribers.
- `ActivateDiscountByIdAsync(int discountId)` – Activates a discount (called by Hangfire or admin). It fetches affected products and delegates price application to `DiscountPriceService`.
- `DeactivateDiscountByIdAsync(int discountId)` – Deactivates a discount (called by Hangfire or sweeper). It restores prices and applies fallback discounts.
- `UpdateDiscountAndNotifyAsync(UpdateDiscountDto)` – The “clean slate” update flow: restores old prices, updates the discount entity, then reapplies the new rules or fallback discounts, schedules jobs, and triggers notifications.
- `SweepExpiredDiscountsAsync()` – Finds all active discounts whose `EndDate` has passed and deactivates them.
- `DeleteDiscountAndCleanUpAsync(int id)` – A convenience method that wraps deletion with proper price restoration and job cancellation (orchestrates `DiscountCrudService.DeleteDiscountAsync` plus lifecycle steps).

**Key Dependencies:**
- `IDiscountCrudService`, `IDiscountQueryService`, `IDiscountPriceService` – The three sibling services, injected via constructor.
- `IUnitOfWork` – For direct data access when needed (e.g., fetching products).
- `ICacheManagerService` – To invalidate product caches after price changes.
- `Lazy<INotificationService>` – To send notifications about discounted products (avoids circular dependency).

**Note:** This service is the only one that references Hangfire or the cache manager. It acts as a **Facade** for external consumers (controllers, Hangfire jobs). All time‑based logic (scheduling, rescheduling, sweeping) resides here.

---

## 5. Relationships and Dependency Flow

The diagram illustrates a clean dependency hierarchy:

1. **Interfaces define contracts** – The rest of the application depends only on `IDiscountCrudService`, `IDiscountQueryService`, etc. Controllers or Hangfire jobs never reference concrete classes.
2. **Concrete classes implement contracts** – Each concrete class fulfils exactly one interface.
3. **The Lifecycle service depends on the other three interfaces** – This makes it the orchestrator. It does **not** inherit from them; it uses them as injected services. This is a classic **Facade with Mediator** pattern.
4. **No circular dependencies** – The query, price, and CRUD services have no knowledge of the lifecycle service or each other. This keeps the system modular and testable.

The dependency graph can be summarised as:

```
Controller / Hangfire
      |
      v
IDiscountLifecycleService (orchestrator)
      |            |            |
      v            v            v
IDiscountCrudService  IDiscountQueryService  IDiscountPriceService
```

Each “leaf” service talks directly to the database via `IUnitOfWork`. The orchestrator may also use `IUnitOfWork` for operations that don’t fit neatly into one of the leaf services (e.g., updating the `Discount` entity’s `StartJobId`).

---

## 6. Design Patterns in Play

- **Single Responsibility Principle** – Each service has exactly one reason to change.
- **Dependency Inversion** – High‑level modules (orchestrator, controllers) depend on abstractions, not concretions.
- **Facade Pattern** – `DiscountLifecycleService` provides a simplified interface that hides the complexity of coordinating three services, caching, and background jobs.
- **Strategy Pattern (implicit)** – The price calculation logic inside `DiscountPriceService` can be seen as a family of algorithms (resolve tier, calculate price) that can be swapped via the interface.
- **Unit of Work** – The `IUnitOfWork` manages database transactions across services.

---

## 7. Conclusion

This class diagram and the accompanying documentation form the architectural blueprint of LiliShop's discount engine. By separating concerns into four well‑defined services and expressing their relationships through interfaces, the system achieves high maintainability, testability, and scalability.

For any future development, please refer to this document to understand where new functionality should be placed:
- New database operations → `IDiscountCrudService`.
- New read‑only queries → `IDiscountQueryService`.
- New pricing rules → `IDiscountPriceService`.
- New scheduling or notification logic → `IDiscountLifecycleService`.

The refactored architecture ensures that extending the discount system remains straightforward and safe.

***
***

# Sequence Diagram – Admin Updates an Active Campaign

**Version:** 1.0  
**Project:** LiliShop – Discount System  
**Date:** 2026-05-16

---

## 1. Overview

When an administrator modifies the rules, dates, or tiers of an already active discount campaign, the system must update the discount safely without leaving any product with an incorrect price. This sequence diagram models the entire interaction between the controller, the refactored services, the database, Hangfire, and the cache.

The flow implements the **Clean Slate** pattern: first, all effects of the old discount are removed (prices restored), then the new rules are persisted, then new prices are calculated using the updated rules, and finally side effects (background jobs, cache) are updated.

The diagram corresponds to **Scenario 3: Admin Updates an Active Campaign** from the Multi‑Discount Price Management Workflow and to the **Active → Active** self‑transition in the Discount Lifecycle State Diagram.

---

## 2. Sequence Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'background': '#ffffff',
    'primaryColor': '#ffffff',
    'primaryTextColor': '#2C3E50',
    'primaryBorderColor': '#34495E',
    'lineColor': '#34495E',
    'fontFamily': 'Arial, sans-serif'
  }
}}%%
sequenceDiagram
    autonumber
     
    actor Admin
    participant API as DiscountsController
    participant Life as LifecycleService
    participant Price as PriceService
    participant Crud as CrudService
    participant DB as SQL Database
    participant HF as Hangfire Worker
    participant Cache as CacheManager

    Admin->>API: PUT /api/discounts/update/{id} (New Rules)
    activate API
    API->>Life: UpdateDiscountAndNotifyAsync(dto)
    activate Life

    %% PHASE 1: CLEAN SLATE
    rect rgb(253, 237, 236)
        Note over Life,DB: Phase 1: Clean Slate (Restore Baseline)
        Life->>Price: RestorePricesForAffectedProductsAsync(id)
        activate Price
        Price->>DB: Fetch old affected products
        Price->>DB: Revert Price = PreviousPrice (Optimistic Concurrency)
        DB-->>Price: Commit successful
        Price-->>Life: Return List<Product> restoredProducts
        deactivate Price
    end

    %% PHASE 2: APPLY RULE CHANGES
    rect rgb(234, 250, 241)
        Note over Life,DB: Phase 2: Persist New Campaign Rules
        Life->>Crud: UpdateDiscountAsync(dto)
        activate Crud
        Crud->>DB: Delete old ConditionGroups
        Crud->>DB: Insert new Tiers & ConditionGroups
        DB-->>Crud: Commit successful (Audit Log generated)
        Crud-->>Life: Return UpdateDiscountDto (New State)
        deactivate Crud
    end

    %% PHASE 3: RE-EVALUATION
    rect rgb(235, 245, 251)
        Note over Life,DB: Phase 3: Recalculate New Reality
         
        alt Campaign is currently Active
            Life->>Life: ActivateDiscountByIdAsync(id)
            Life->>Price: GetProductsAffectedByDiscountGroupAsync()
            Price->>DB: Query products matching new rules
            DB-->>Price: Return new targeted products
            Life->>Price: ApplyBestDiscountsToRestoredProducts()
            Price->>DB: Save new lowest prices (Handling Conflicts)
        end
         
        Note over Life,Price: Ensure products that "lost" the discount<br/>get the next-best active campaign fallback.
        Life->>Price: ApplyBestDiscountsToRestoredProducts(restoredProducts)
        activate Price
        Price->>DB: Calculate & Commit fallbacks
        DB-->>Price: Commit successful
        Price-->>Life: Done
        deactivate Price
    end

    %% PHASE 4: SIDE EFFECTS
    rect rgb(244, 236, 247)
        Note over Life,Cache: Phase 4: Handle Side Effects (Jobs & Cache)
         
        Life->>HF: Delete old Start/End Jobs
         
        opt StartDate is in the future
            Life->>HF: Schedule ActivateDiscountByIdAsync
        end
        opt EndDate is in the future
            Life->>HF: Schedule DeactivateDiscountByIdAsync
        end
         
        Life->>Cache: InvalidateDiscountCacheAsync()
        Cache-->>Life: Cache Cleared
    end

    Life-->>API: Success Result
    deactivate Life
    API-->>Admin: HTTP 200 OK
    deactivate API
```

---

## 3. Participants

| Participant | Abbreviation | Role |
|-------------|--------------|------|
| **Admin** | – | The human user updating the discount from the admin panel. |
| **DiscountsController** | API | The ASP.NET Core controller that receives the HTTP request. |
| **DiscountLifecycleService** | Life | The orchestrator that coordinates the entire update flow. |
| **DiscountPriceService** | Price | Domain service for price restoration, tier resolution, and best‑price calculation. |
| **DiscountCrudService** | Crud | Persistence service that updates discount entities and generates audit logs. |
| **SQL Database** | DB | The relational database storing discounts, products, conditions, and prices. |
| **Hangfire Worker** | HF | Background job scheduler for future activation/deactivation. |
| **CacheManager** | Cache | In‑memory or distributed cache for product data. |

---

## 4. The Four Phases

The diagram is split into four coloured regions, each representing a distinct phase of the update process.

### Phase 1: Clean Slate – Restore Baseline (Red)

**Goal:** Remove all price effects of the old discount before applying new rules.

| Step | Caller → Callee | Method | Description |
|------|-----------------|--------|-------------|
| 1–2 | Admin → API → Life | `UpdateDiscountAndNotifyAsync(dto)` | Request enters the system. |
| 3 | Life → Price | `RestorePricesForAffectedProductsAsync(id)` | Asks the price service to undo the old discount. |
| 4–5 | Price → DB | Fetch & Update | Retrieves products that were affected by the old discount. For each product, sets `Price = PreviousPrice` (original price) and clears `PreviousPrice`. Uses optimistic concurrency (`RowVersion`) to prevent overwrites. |
| 6 | Price → Life | Return | Returns the list of restored products (`restoredProducts`). |

**Outcome:** All products are back to their original prices. No trace of the old discount remains.

---

### Phase 2: Persist New Campaign Rules (Green)

**Goal:** Save the administrator’s updated discount definition to the database.

| Step | Caller → Callee | Method | Description |
|------|-----------------|--------|-------------|
| 7 | Life → Crud | `UpdateDiscountAsync(dto)` | Hands the updated DTO to the CRUD service. |
| 8 | Crud → DB | Delete + Insert | Deletes all old `ConditionGroup` entities, inserts new tiers and condition groups, updates the discount’s fields. An audit log is created with a before/after JSON snapshot. Everything runs in a single database transaction. |
| 9 | Crud → Life | Return | Returns the updated `UpdateDiscountDto` reflecting the persisted state. |

**Outcome:** The database now holds exactly the new rules, but no product prices have changed yet.

---

### Phase 3: Recalculate New Reality (Blue)

**Goal:** Apply the new discount rules to products and ensure products that no longer match get the next‑best active discount.

#### Sub‑flow A: Campaign is currently Active

| Step | Caller → Callee | Method | Description |
|------|-----------------|--------|-------------|
| 10 | Life → Life | `ActivateDiscountByIdAsync(id)` | Triggers the full activation flow for the updated discount. |
| 11 | Life → Price | `GetProductsAffectedByDiscountGroupAsync()` | Finds all products that match the **new** condition groups. |
| 12 | Price → DB | Query | Rule engine query to fetch matching products. |
| 13 | Life → Price | `ApplyBestDiscountsToRestoredProducts(…)` | For these products, loads all active discounts, resolves tiers, calculates the lowest price, and updates `Price`/`PreviousPrice`. Concurrency conflicts are handled with a retry loop. |

#### Sub‑flow B: Fallback for products that “lost” the discount

| Step | Caller → Callee | Method | Description |
|------|-----------------|--------|-------------|
| 14 | Life → Price | `ApplyBestDiscountsToRestoredProducts(restoredProducts)` | The products that were restored in Phase 1 but do **not** match the new rules still need the next‑best active discount. The current discount is excluded from this evaluation. |
| 15 | Price → DB | Calculate & Commit | For each product, the system finds the best alternative discount and updates the price. Committed with concurrency protection. |

**Outcome:**  
- Products that still match the campaign get the new (or best) discounted price.  
- Products that no longer match get the next‑best price from any other active discount.  
- No product is left without a valid discount if one exists.

---

### Phase 4: Handle Side Effects – Jobs & Cache (Purple)

**Goal:** Update Hangfire background jobs and clear the product cache so that the frontend reflects the latest prices.

| Step | Caller → Callee | Method | Description |
|------|-----------------|--------|-------------|
| 16 | Life → HF | Delete | Removes the old activation and deactivation Hangfire jobs (identified by `StartJobId` and `EndJobId`). |
| 17 | Life → HF | Schedule (optional) | If the new `StartDate` is in the future, schedules a new `ActivateDiscountByIdAsync` job. If the new `EndDate` is in the future, schedules a `DeactivateDiscountByIdAsync` job. |
| 18 | Life → Cache | `InvalidateDiscountCacheAsync()` | Clears the product cache so that the next page load fetches fresh prices. |
| 19–20 | Life → API → Admin | Return | Returns a success result to the controller, and the controller responds with HTTP 200 OK. |

---

## 5. Key Design Points

- **Clean Slate First:** Prices are always restored before any new rules are applied. This prevents double‑discounting or orphaned sale prices.
- **Transactional Integrity:** Phase 2 (the database update) is wrapped in a transaction; if anything fails, the whole discount update is rolled back.
- **Best‑Price Guarantee:** Phase 3 doesn’t just apply the updated discount – it evaluates **all** active discounts to ensure the customer always sees the lowest price.
- **Concurrency Safety:** The price service uses a `DbUpdateConcurrencyException` retry loop with `RowVersion` to handle simultaneous updates from other jobs or admin actions.
- **Orchestrator Pattern:** The `LifecycleService` coordinates everything but doesn’t do the heavy lifting itself. This keeps the code modular and testable.
- **Separation of Concerns:** The diagram directly maps to the refactored services: `PriceService` for pricing logic, `CrudService` for persistence, `LifecycleService` for orchestration and side effects.

---

## 6. Mapping to Code

| Phase | Primary Method(s) |
|-------|-------------------|
| **Phase 1** | `DiscountLifecycleService.UpdateDiscountAndNotifyAsync` → `DiscountPriceService.RestorePricesForAffectedProductsAsync` |
| **Phase 2** | `DiscountCrudService.UpdateDiscountAsync` |
| **Phase 3** | `DiscountLifecycleService.ActivateDiscountByIdAsync` + `DiscountPriceService.GetProductsAffectedByDiscountGroupAsync` + `DiscountPriceService.ApplyBestDiscountsToRestoredProductsAsync` |
| **Phase 4** | Hangfire job deletion/rescheduling calls inside `UpdateDiscountAndNotifyAsync`, `InvalidateDiscountCacheAsync` |

---

## 7. Conclusion

This sequence diagram captures one of the most intricate operations in LiliShop’s discount engine. By following the clean‑slate, four‑phase approach, the system guarantees that product prices are always accurate, even when an active campaign is modified on the fly.

****************************************************************************
****************************************************************************

# Multi-Discount Price Management Workflow

**Version:** 1.0  
**Project:** LiliShop – Discount System  
**Architecture:** Refactored (Separation of Concerns)  
**Date:** 2026-05-16

---

## 1. Introduction

The discount system in LiliShop must handle overlapping promotions, scheduled activations, manual updates, and safe deactivation—all while guaranteeing that each product always displays the best valid price. After refactoring the original *God Class* `DiscountService`, the logic now lives in four focused services:

- **`DiscountLifecycleService`** – orchestrates state transitions (activate, deactivate, update) and coordinates Hangfire background jobs, cache invalidation, and notifications.
- **`DiscountPriceService`** – contains pure domain logic for tier resolution, price calculation, and applying the best discount across multiple campaigns.
- **`DiscountCrudService`** – handles persistence, audit logs, and transaction management.
- **`DiscountQueryService`** – serves all read‑only queries (rule engine, paginated product retrieval).

This document describes the three main operational workflows using a flowchart and maps every step to the responsible method(s). The diagram and text assume you have the refactored code from the previous step.

---

## 2. Workflow Overview

The following Mermaid flowchart visualises the three scenarios covered by the system. Each scenario is a sub‑graph with a distinct trigger.

```mermaid
---
title: Multi-Discount Price Management Workflow (Refactored Architecture)
---
flowchart TD
    classDef trigger fill:#2E86C1,stroke:#fff,stroke-width:2px,color:#fff
    classDef decision fill:#D68910,stroke:#fff,stroke-width:2px,color:#fff
    classDef action fill:#8E44AD,stroke:#fff,stroke-width:2px,color:#fff
    classDef database fill:#27AE60,stroke:#fff,stroke-width:2px,color:#fff
    classDef service fill:#34495E,stroke:#fff,stroke-width:1px,color:#fff

    %% SCENARIO 1: BACKGROUND ACTIVATION
    subgraph Scenario1 ["Scenario 1: Activation (Lifecycle & Price)"]
        direction TB
        A1([Worker: StartDate Reached]):::trigger
        A2[LifecycleService: Set IsActive = true]:::service
        A3{Is Single or Group?}:::decision
        
        A4_S[Fetch Direct Products]:::action
        A4_G[Execute Rule Engine Query]:::action
        
        A5[PriceService: ApplyBestDiscountsToRestoredProducts]:::service
        A6[(Fetch ALL Currently Active Discounts)]:::database
        A7[Set Base Price = PreviousPrice ?? Price]:::action
        A8[Calculate all matching tiers & find lowest price]:::action
        
        A9{Is Best Price < Base Price?}:::decision
        A10[Set PreviousPrice = Base Price <br/> Set Price = Best Price]:::action
        A11[Keep existing Price]:::action

        A12[(Commit with DbUpdateConcurrency Retry Loop)]:::database

        A1 --> A2
        A2 --> A3
        A3 -- Single --> A4_S
        A3 -- Group --> A4_G
        A4_S --> A5
        A4_G --> A5
        A5 --> A6
        A6 --> A7
        A7 --> A8
        A8 --> A9
        A9 -- YES --> A10
        A9 -- NO --> A11
        A10 --> A12
        A11 --> A12
    end

    %% SCENARIO 2: BACKGROUND DEACTIVATION
    subgraph Scenario2 ["Scenario 2: Deactivation (Lifecycle & Price)"]
        direction TB
        B1([Worker: EndDate Reached]):::trigger
        B2[LifecycleService: Set IsActive = false]:::service
        
        B3[PriceService: RestorePricesForAffectedProducts]:::service
        B4[Fetch products tied to expired campaign]:::action
        B5[Revert Price = PreviousPrice <br/> Clear PreviousPrice]:::action
        B6[(Commit Restored Base Prices)]:::database
        
        B7[PriceService: ApplyBestDiscountsToRestoredProducts]:::service
        B8[(Fetch REMAINING Active Discounts)]:::database
        B9[Calculate next-best fallback price]:::action
        
        B10[(Commit Fallback Prices with Concurrency Retries)]:::database

        B1 --> B2
        B2 --> B3
        B3 --> B4
        B4 --> B5
        B5 --> B6
        B6 --> B7
        B7 --> B8
        B8 --> B9
        B9 --> B10
    end

    %% SCENARIO 3: LIVE UPDATE (CLEAN SLATE)
    subgraph Scenario3 ["Scenario 3: Admin Updates an Active Campaign"]
        direction TB
        C1([Admin Submits New Rules/Tiers]):::trigger
        
        C2[PriceService: RestorePricesForAffectedProducts]:::service
        C3[Clear old campaign prices & restore base prices]:::action
        
        C4[CrudService: Update Discount Entities in DB]:::service
        
        C5{Is Campaign Still Active Now?}:::decision
        
        C6["Trigger Activation Flow (Scenario 1)"]:::action
        C7[Evaluate restored products against OTHER active campaigns]:::action
        
        C8[(Commit & Schedule Hangfire Jobs)]:::database

        C1 --> C2
        C2 --> C3
        C3 --> C4
        C4 --> C5
        C5 -- YES --> C6
        C5 -- NO --> C7
        C6 --> C8
        C7 --> C8
    end
```

The next three sections walk through each scenario in detail.

---

## 3. Scenario 1: Campaign Activation (Background Worker)

### Trigger
A Hangfire job scheduled at the discount’s `StartDate` fires and invokes:

```
DiscountLifecycleService.ActivateDiscountByIdAsync(discountId)
```

### Step-by-Step Execution

| Step | Description | Responsible Method |
|------|-------------|-------------------|
| 1 | Set `IsActive = true` on the discount entity and save it. | `DiscountLifecycleService.ActivateDiscountByIdAsync` |
| 2 | Determine if the discount is a **single** direct assignment or a **group** rule‑based one. | `if (discount.DiscountGroup == null)` inside `ActivateDiscountByIdAsync` |
| 3a | **Single** discount → fetch affected products from `ProductDiscount`. | Query `productDiscountsSet.Where(pd => pd.DiscountId == discountId).Select(pd => pd.Product)` |
| 3b | **Group** discount → execute the rule engine query. | `DiscountPriceService.GetProductsAffectedByDiscountGroupAsync(discount.DiscountGroup)` |
| 4 | Collect all affected products (distinct) and hand them to the price service. | Call to `DiscountPriceService.ApplyBestDiscountsToRestoredProductsAsync(allAffectedProducts)` |
| 5 | Inside the price service: load **all currently active discounts** (not just the activated one). | `_unitOfWork.Context.Set<Discount>()...` with filters on `IsActive`, dates, and exclusion of a specific discount (if any) |
| 6 | For each product, set **base price** = `PreviousPrice ?? Price` (original price before any discount). | `ApplyBestDiscountsToRestoredProductsAsync` – the loop over products |
| 7 | For each active discount, resolve the matching tier and compute the discounted price. Keep the lowest price found. | `DiscountPriceService.ResolveTierForProduct(discount, product)` and `CalculateEffectivePrice(product, tier)` |
| 8 | If the best discounted price is lower than the base price: set `PreviousPrice = basePrice`, `Price = bestPrice`. Otherwise, leave the product unchanged. | Conditional assignment inside the loop |
| 9 | Commit all price changes with a **concurrency retry loop** (up to 3 retries using `DbUpdateConcurrencyException` handling and `RowVersion` reload). | `ApplyBestDiscountsToRestoredProductsAsync` – `try/catch` block |
| 10 | Schedule the deactivation Hangfire job (if `EndDate` is set) and invalidate the product cache. | `ActivateDiscountByIdAsync` – `BackgroundJob.Schedule` and `InvalidateDiscountCacheAsync` |

**Outcome:** All products covered by the discount now show the best possible price considering all overlapping promotions.

---

## 4. Scenario 2: Campaign Deactivation / Expiration

### Trigger
A Hangfire job scheduled at the discount’s `EndDate` (or a sweeper job for missed discounts) calls:

```
DiscountLifecycleService.DeactivateDiscountByIdAsync(discountId)
```

### Step-by-Step Execution

| Step | Description | Responsible Method |
|------|-------------|-------------------|
| 1 | Set `IsActive = false` on the discount. Cancel any associated Hangfire jobs (`StartJobId`, `EndJobId`). | `DeactivateDiscountByIdAsync` |
| 2 | Call the price service to restore original prices for all products that were affected by this discount. | `DiscountPriceService.RestorePricesForAffectedProductsAsync(discountId)` |
| 3 | Inside that method: fetch the affected products. For **single** discounts, query `ProductDiscount`; for **group** discounts, use `GetProductsAffectedByDiscountGroupAsync`. | `RestorePricesForAffectedProductsAsync` |
| 4 | For each product where `PreviousPrice` is set, revert `Price = PreviousPrice` and clear `PreviousPrice`. | Loop inside `RestorePricesForAffectedProductsAsync` |
| 5 | Commit the restored base prices with concurrency retries. | `RestorePricesForAffectedProductsAsync` – retry loop |
| 6 | Now that the expired discount’s prices are gone, apply any **other active discounts** to the same restored products. | `DiscountPriceService.ApplyBestDiscountsToRestoredProductsAsync(restoredProducts, excludeDiscountId: discountId)` |
| 7 | The method loads all remaining active discounts, resolves tiers, computes prices, and updates products exactly like Scenario 1, but with the expired discount excluded. | `ApplyBestDiscountsToRestoredProductsAsync` |
| 8 | Commit the fallback prices (again with concurrency retries). | Same method |
| 9 | Invalidate the product cache to reflect the new prices. | `InvalidateDiscountCacheAsync` inside `DeactivateDiscountByIdAsync` |

**Outcome:** Products revert to their original prices or the next best active discount, never showing a stale sale price.

---

## 5. Scenario 3: Admin Updates an Active Campaign (Clean Slate)

### Trigger
An administrator modifies the rules, dates, or tiers of a discount that is currently active. The controller calls:

```
DiscountLifecycleService.UpdateDiscountAndNotifyAsync(dto)
```

### Step-by-Step Execution

| Step | Description | Responsible Method |
|------|-------------|-------------------|
| 1 | **Undo the old discount:** Restore prices for all products that were affected by the current (pre‑update) version of the discount. | `DiscountPriceService.RestorePricesForAffectedProductsAsync(dto.Id)` |
| 2 | All affected products get their original base price back (`Price = PreviousPrice`), and `PreviousPrice` is cleared. | Inside `RestorePricesForAffectedProductsAsync` |
| 3 | **Update the discount entity** (rules, tiers, conditions, dates) in the database, including audit logging. This is a transactional operation. | `DiscountCrudService.UpdateDiscountAsync(dto)` |
| 4 | After the update, check whether the discount is still active **right now** (`IsActive == true` and `StartDate <= now <= EndDate`). | Decision inside `UpdateDiscountAndNotifyAsync` |
| 5a | **YES – still active:** Trigger the full activation flow (Scenario 1) to apply the *new* rules to all matching products. | Calls `ActivateDiscountByIdAsync(dto.Id)` |
| 5b | **NO – no longer active:** Apply only the other active discounts to the restored products, excluding this discount. | Calls `ApplyBestDiscountsToRestoredProductsAsync(restoredProducts, excludeDiscountId: dto.Id)` |
| 6 | Schedule or re‑schedule Hangfire background jobs for activation/deactivation based on the new `StartDate` and `EndDate`. | `BackgroundJob.Schedule` calls inside `UpdateDiscountAndNotifyAsync` |
| 7 | Invalidate the product cache so that the frontend reflects the latest prices. | `InvalidateDiscountCacheAsync` |

**Outcome:** The discount’s updated rules take effect seamlessly. Products that no longer match receive the next best available price; products that still match are recalculated. No price is left in an inconsistent state.

---

## 6. Key Design Principles

- **Best‑price guarantee:** Every price recalculation considers *all* currently active discounts, not just the one being toggled.  
- **Clean slate approach:** Before applying new rules, the system completely removes the old discount’s prices, eliminating any risk of double‑counting or orphaned prices.  
- **Concurrency safety:** All database commits that modify product prices are wrapped in a `DbUpdateConcurrencyException` retry loop, using the `RowVersion` column to detect conflicts.  
- **Separation of concerns:** The `LifecycleService` orchestrates, the `PriceService` does math, the `CrudService` handles persistence, and the `QueryService` handles reads. This makes each piece independently testable and maintainable.  
- **Hangfire integration:** All time‑based transitions rely on persisted job IDs, with proper cancellation and rescheduling when dates change.

---

## 7. Conclusion

The refactored Multi-Discount Price Management Workflow guarantees that every product in LiliShop always reflects the most beneficial active promotion, even when discounts overlap, are updated live, or expire. By following the three scenarios described above—and using the exact methods referenced—any developer can understand, debug, or extend the system with confidence.

For further questions, refer to the source code of `DiscountLifecycleService`, `DiscountPriceService`, `DiscountCrudService`, and `DiscountQueryService` inside `LiliShop.Infrastructure.Services`.

***
***

# Discount Lifecycle State Diagram – Technical Documentation

**Version:** 1.0  
**Project:** LiliShop – Discount System  
**Date:** 2026-05-16

---

## 1. Introduction

Every discount in LiliShop follows a strict lifecycle governed by four distinct states. The state diagram below visualises all possible transitions—17 in total—triggered by admin actions, background jobs, or system sweeps. This document defines each state, explains every transition, and references the responsible service methods from the refactored codebase.

**Key Services Involved**

- **`DiscountLifecycleService`** – orchestrates state changes, Hangfire jobs, cache invalidation, and notifications.
- **`DiscountPriceService`** – handles price restoration, best‑price recalculation, and concurrency‑safe commits.
- **`DiscountCrudService`** – manages persistence of discount entities and audit logs.
- **`DiscountQueryService`** – provides read‑only queries used during transitions (e.g., fetching affected products).

---

## 2. State Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'background': '#ffffff',
    'primaryColor': '#ffffff',
    'secondaryColor': '#ffffff',
    'tertiaryColor': '#ffffff',
    'lineColor': '#333333',
    'fontFamily': 'Arial',
    'edgeLabelBackground':'#ffffff'
  }
}}%%

flowchart TD

    classDef draft fill:#F8F9F9,stroke:#7F8C8D,stroke-width:3px,color:#2C3E50,font-weight:bold,rx:15,ry:15
    classDef scheduled fill:#FEF5E7,stroke:#D68910,stroke-width:3px,color:#B9770E,font-weight:bold,rx:15,ry:15
    classDef active fill:#E9F7EF,stroke:#27AE60,stroke-width:3px,color:#196F3D,font-weight:bold,rx:15,ry:15
    classDef expired fill:#FDEDEC,stroke:#C0392B,stroke-width:3px,color:#922B21,font-weight:bold,rx:15,ry:15
    classDef endpoint fill:#2C3E50,stroke:#ffffff,stroke-width:2px,color:#ffffff,font-weight:bold,rx:30,ry:30

    Start((Start)):::endpoint
    End((Deleted)):::endpoint

    Draft["Draft<br/>(Inactive)"]:::draft
    Scheduled["Scheduled<br/>(Pending Start)"]:::scheduled
    Active["Active<br/>(Live Sale)"]:::active
    Expired["Expired<br/>(Deactivated)"]:::expired

    Start -->|Create Draft| Draft
    Start -->|Create Scheduled| Scheduled
    Start -->|Create Active| Active

    Draft -->|Activate Future| Scheduled
    Draft -->|Activate Now| Active
    Draft -->|Delete| End

    Scheduled -->|Deactivate| Draft
    Scheduled -->|Worker: StartDate Reached| Active
    Scheduled -->|Modify Rules| Scheduled
    Scheduled -->|Delete| End

    Active -->|Deactivate| Draft
    Active -->|Update: Clean Slate Re-evaluation| Active
    Active -->|Worker: EndDate Reached| Expired
    Active -->|Delete & Restore Prices| End

    Expired -->|Extend Future| Scheduled
    Expired -->|Reactivate| Active
    Expired -->|Delete| End

    linkStyle 0 stroke:#1ABC9C,stroke-width:3px
    linkStyle 1 stroke:#3498DB,stroke-width:3px
    linkStyle 2 stroke:#9B59B6,stroke-width:3px
    linkStyle 3 stroke:#F1C40F,stroke-width:3px
    linkStyle 4 stroke:#E67E22,stroke-width:3px
    linkStyle 5 stroke:#E74C3C,stroke-width:3px
    linkStyle 6 stroke:#95A5A6,stroke-width:3px
    linkStyle 7 stroke:#2ECC71,stroke-width:3px
    linkStyle 8 stroke:#F39C12,stroke-width:3px
    linkStyle 9 stroke:#C0392B,stroke-width:3px
    linkStyle 10 stroke:#BDC3C7,stroke-width:3px
    linkStyle 11 stroke:#8E44AD,stroke-width:3px
    linkStyle 12 stroke:#16A085,stroke-width:3px
    linkStyle 13 stroke:#641E16,stroke-width:3px
    linkStyle 14 stroke:#2980B9,stroke-width:3px
    linkStyle 15 stroke:#27AE60,stroke-width:3px
    linkStyle 16 stroke:#34495E,stroke-width:3px
```

---

## 3. State Descriptions

### 3.1 Draft (Inactive)

**Colour:** Grey  
**Meaning:** The discount exists in the database but has absolutely **no effect** on product prices. `IsActive` is `false`. No Hangfire jobs are scheduled.

**Characteristics:**
- Saved as a template; all fields editable by admin.
- No price changes, no cache invalidation.
- Can be safely deleted without any price restoration.

**Typical use:** A marketer prepares a discount for later review, or an active campaign is manually paused.

---

### 3.2 Scheduled (Pending Start)

**Colour:** Gold/Amber  
**Meaning:** The discount is **intended to become active**, but the current time is before its `StartDate`. `IsActive` is `true`, but the discount is **not yet** applied to any product. A Hangfire job is scheduled to activate it at the exact start time.

**Characteristics:**
- A `StartJobId` is stored in the `Discount` entity.
- No product prices are touched.
- The discount can still be modified; rules and dates can change, causing a self‑loop transition (jobs are rescheduled).
- Deletion cancels the pending job.

**Typical use:** A promotion set to start next Monday.

---

### 3.3 Active (Live Sale)

**Colour:** Green  
**Meaning:** The discount is **currently affecting product prices**. `IsActive` is `true`, the current time is between `StartDate` and `EndDate`, and all eligible products have their prices recalculated.

**Characteristics:**
- A deactivation Hangfire job (`EndJobId`) is scheduled for the `EndDate`.
- Products that match the discount rules have `PreviousPrice` set to the original price and `Price` updated to the discounted amount (or the best available price from overlapping discounts).
- Any update triggers a “clean slate” re‑evaluation to avoid stale prices.
- Deactivation or deletion restores base prices and applies fallback discounts.

**Typical use:** A 20% off summer sale that is currently running.

---

### 3.4 Expired (Deactivated)

**Colour:** Red  
**Meaning:** The discount’s `EndDate` has passed. It is **no longer active** and does not affect product prices. It remains in the database for historical/audit purposes.

**Characteristics:**
- `IsActive` is `false`.
- No Hangfire jobs associated.
- Products previously discounted by this campaign have already been restored (via Scenario 2).
- Can be extended or reactivated, moving it back to Scheduled or Active.

**Typical use:** A past Black Friday deal that the team may want to reuse next year.

---

## 4. Transition Catalogue

The diagram defines **17 transitions**. Below, each is listed with its trigger, outcome, and the primary method(s) responsible.

### 4.1 Creation Flows

| # | Transition | Trigger | Outcome | Method(s) |
|---|-----------|---------|---------|-----------|
| 1 | **Create Draft** | Admin creates discount with `IsActive = false`. | Discount saved in **Draft**, no jobs. | `DiscountCrudService.CreateDiscountAsync` |
| 2 | **Create Scheduled** | Admin creates with `IsActive = true` and `StartDate > Now`. | Discount moves to **Scheduled**, activation job scheduled. | `DiscountLifecycleService.CreateDiscountAndNotifySubscribersAboutNewDiscountAsync` (orchestrates) |
| 3 | **Create Active** | Admin creates with `IsActive = true` and `StartDate <= Now`. | Discount is immediately **Active**; prices are applied. | `DiscountLifecycleService.CreateDiscountAndNotifySubscribersAboutNewDiscountAsync` |

---

### 4.2 Draft Transitions

| # | Transition | Trigger | Outcome | Method(s) |
|---|-----------|---------|---------|-----------|
| 4 | **Activate Future** | Admin sets `IsActive = true` and a future `StartDate`. | Moves to **Scheduled**, schedules activation job. | Handled inside `DiscountCrudService.UpdateDiscountAsync` and/or `UpdateDiscountAndNotifyAsync` |
| 5 | **Activate Now** | Admin sets `IsActive = true` and `StartDate <= Now`. | Moves to **Active**, prices recalculated (Scenario 1). | `DiscountLifecycleService.UpdateDiscountAndNotifyAsync` → triggers `ActivateDiscountByIdAsync` |
| 6 | **Delete** | Admin removes the discount. | Permanently deleted; no price impact. | `DiscountCrudService.DeleteDiscountAsync` |

---

### 4.3 Scheduled Transitions

| # | Transition | Trigger | Outcome | Method(s) |
|---|-----------|---------|---------|-----------|
| 7 | **Deactivate** | Admin sets `IsActive = false`. | Moves back to **Draft**, cancels pending job. | `DiscountCrudService.UpdateDiscountAsync` + job cancellation |
| 8 | **Worker: StartDate Reached** | Hangfire job fires. | Discount becomes **Active**, prices applied (Scenario 1). | `DiscountLifecycleService.ActivateDiscountByIdAsync` |
| 9 | **Modify Rules** | Admin changes dates or conditions while still Scheduled. | Stays in **Scheduled**, reschedules the activation job. | `UpdateDiscountAndNotifyAsync` (handles job rescheduling) |
| 10 | **Delete** | Admin removes the discount. | Cancels the scheduled job and deletes the discount. | `DiscountCrudService.DeleteDiscountAsync` |

---

### 4.4 Active Transitions

| # | Transition | Trigger | Outcome | Method(s) |
|---|-----------|---------|---------|-----------|
| 11 | **Deactivate** | Admin sets `IsActive = false`. | Moves to **Draft**, restores prices, applies fallback discounts (Scenario 2). | `DiscountLifecycleService.DeactivateDiscountByIdAsync` or `UpdateDiscountAndNotifyAsync` |
| 12 | **Update: Clean Slate Re‑evaluation** | Admin modifies rules/tiers of an already active discount. | Stays **Active**, but prices are restored and then reapplied with new rules (Scenario 3). | `DiscountLifecycleService.UpdateDiscountAndNotifyAsync` → `RestorePricesForAffectedProductsAsync` → `UpdateDiscountAsync` → `ActivateDiscountByIdAsync` |
| 13 | **Worker: EndDate Reached** | Hangfire job (or sweeper) fires. | Moves to **Expired**, prices restored, fallback discounts applied (Scenario 2). | `DeactivateDiscountByIdAsync` or `SweepExpiredDiscountsAsync` |
| 14 | **Delete & Restore Prices** | Admin deletes an active discount. | Permanently deleted; prices restored, fallback discounts applied, jobs cancelled. | `DiscountCrudService.DeleteDiscountAsync` (orchestrates restoration) |

---

### 4.5 Expired Transitions

| # | Transition | Trigger | Outcome | Method(s) |
|---|-----------|---------|---------|-----------|
| 15 | **Extend Future** | Admin sets a new future `StartDate`. | Moves back to **Scheduled**, schedules new activation job. | `UpdateDiscountAndNotifyAsync` (reschedules based on new dates) |
| 16 | **Reactivate** | Admin sets `IsActive = true` and a current or past `StartDate`. | Moves directly to **Active**, prices recalculated. | `UpdateDiscountAndNotifyAsync` → triggers `ActivateDiscountByIdAsync` |
| 17 | **Delete** | Admin removes the expired discount. | Permanently deleted; no price impact (already restored). | `DiscountCrudService.DeleteDiscountAsync` |

---

## 5. System‑Level Guarantees

- **No orphaned sale prices:** Every transition that leaves the Active state triggers a price restoration and fallback evaluation, using `DiscountPriceService.RestorePricesForAffectedProductsAsync` and `ApplyBestDiscountsToRestoredProductsAsync`.
- **Atomic transactions:** Updates that involve both entity changes and product price modifications are wrapped in database transactions and concurrency retry loops (`DbUpdateConcurrencyException` + `RowVersion`).
- **Traceability:** Every modification to a discount’s core fields is captured as a `DiscountAuditLog` entry by `DiscountCrudService`.
- **Time‑bound automation:** All future state changes rely on Hangfire jobs, with job IDs stored on the discount entity. If dates change, jobs are cancelled and re‑created.

---

## 6. Conclusion

The discount lifecycle state diagram provides a complete, deterministic map of how any discount moves from creation to deletion. It complements the multi‑discount price management workflow by showing *when* price calculations occur and *what triggers* them.

***
***
