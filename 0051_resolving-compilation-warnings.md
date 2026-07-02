# Resolving Compilation Warnings in a .NET Project

This article explains why we spent real engineering time cleaning up compiler warnings in the LiliShop backend, what kinds of warnings we found, and exactly how we fixed each kind. All examples are real code from this project.

We started with **468 warnings** across the solution. After this cleanup, the build produces **0 warnings and 0 errors**, and all 65 existing tests still pass.

## Why bother? The app "works" anyway

It's tempting to ignore warnings. The project compiles, the app runs, customers can buy products. So why spend hours on yellow squiggly lines?

Because a warning is the compiler telling you: *"I found a spot where this code could crash, and I can't prove it won't."* Most of our warnings were nullable reference warnings — the compiler noticing a value **could** be `null` at runtime, even though the code treats it as if it never is.

A few concrete reasons this matters:

- **Warnings hide real bugs.** When a project has 468 warnings, nobody reads them anymore. A genuinely new and dangerous warning — say, a real null-reference bug in a payment flow — gets lost in the noise. Zero warnings means any new warning stands out immediately.
- **They are often silent time bombs.** A property like `string ProductName { get; set; }` with no safety net can be `null` at runtime even though the compiler treats it as "always has a value." The first time that assumption is wrong, you get a `NullReferenceException` in production, usually in the code path nobody tested.
- **They erode trust in tooling.** If your team is used to seeing hundreds of warnings, real signals (like a security-relevant nullable vulnerability, or an outdated package with a known CVE) blend into the background.
- **They make refactoring riskier.** Nullable warnings exist specifically to catch the case where you change a method's behavior (e.g., it now can return `null`) and forget to update every caller. Ignoring them removes a safety net you already paid for (the compiler check is free — you just have to listen to it).

In short: warnings are the compiler's way of doing code review for you, for free, every single build. Ignoring them is like ignoring a smoke detector because "the house hasn't burned down yet."

## The categories of warnings we fixed

| Category | Warning codes | What it means |
|---|---|---|
| SDK & package setup | `NETSDK1080`, `NU1510`, `NU1903` | Wrong project setup, unnecessary package, package with a known security vulnerability |
| Domain entity nullability | `CS8618` | A non-nullable property was never guaranteed to be set |
| Identity framework mismatches | `CS8766` | Our code disagreed with ASP.NET Core Identity about whether a property can be `null` |
| Null conversions | `CS8625`, `CS8600`, `CS8601` | Assigning or converting `null` into a spot that claims to never be `null` |
| Possible null dereference / argument / return | `CS8602`, `CS8603`, `CS8604` | Using, passing, or returning a value that might be `null`, without checking first |
| Nullable value types | `CS8629` | Reading `.Value` on a `Nullable<T>` (like `int?`) without checking `.HasValue` first |

Let's go through each one with real before/after code.

### 1. SDK and package warnings

**`NETSDK1080`** — our `Infrastructure` project referenced `Microsoft.AspNetCore.App` as a NuGet package, which is redundant on modern .NET. The fix depends on which SDK the project uses:

```xml
<!-- Before -->
<PackageReference Include="Microsoft.AspNetCore.App" />

<!-- After (project uses Microsoft.NET.Sdk, not Microsoft.NET.Sdk.Web) -->
<FrameworkReference Include="Microsoft.AspNetCore.App" />
```

**`NU1510`** — a package was listed but never actually used because something else pulled it in transitively. We just deleted the redundant line:

```xml
<!-- Removed entirely -->
<PackageReference Include="Microsoft.Extensions.Configuration.Abstractions" Version="10.0.2" />
```

**`NU1903`** — `Microsoft.OpenApi` 3.3.1 had a known high-severity security advisory (GHSA-v5pm-xwqc-g5wc). We bumped the version:

```xml
<PackageReference Include="Microsoft.OpenApi" Version="3.7.0" />
```

### 2. Domain entities that never checked if they were "ready"

Take `BasketItem`, a plain class with no constructor logic:

```csharp
// Before
public class BasketItem
{
    public int Id { get; set; }
    public string ProductName { get; set; }
    public string PictureUrl { get; set; }
    public string Brand { get; set; }
    public string Type { get; set; }
}
```

The compiler flagged every `string` property: nothing forces `ProductName` to be set before the object is used. Since every place in the code that creates a `BasketItem` always supplies all of these values, we used the `required` keyword — it makes the compiler itself enforce "you must set this at creation time":

```csharp
// After
public class BasketItem
{
    public int Id { get; set; }
    public required string ProductName { get; set; }
    public required string PictureUrl { get; set; }
    public required string Brand { get; set; }
    public required string Type { get; set; }
}
```

Now if anyone writes `new BasketItem { Id = 1 }` and forgets `ProductName`, it's a **compile error**, not a runtime surprise months later.

Not everything should be `required` though. Look at `CustomerBasket`:

```csharp
// Payment fields — genuinely absent until a Stripe payment intent is created
public string? ClientSecret { get; set; }
public string? PaymentIntentId { get; set; }
```

Here we made the properties **nullable** (`string?`) instead of `required`, because they really can be empty — a basket exists before any payment attempt happens. The rule we followed throughout:

> If a value is always present when the object is created → make it `required`.
> If a value is legitimately optional or fills in later → make it nullable (`string?`).
> If the value is populated later by a framework (like Entity Framework) rather than by our own code → use `= null!;` with a short comment explaining why.

Example of the third case, from `ProductPhoto`:

```csharp
public string PublicId { get; set; } = null!;
public virtual Product Product { get; set; } = null!;
```

Entity Framework fills in `Product` when it loads data from the database — our code never constructs it directly. `= null!` tells the compiler "trust me, this will be set," without lying about the property's real type.

### 3. Identity framework mismatches

ASP.NET Core's built-in `IdentityUser<TKey>` declares `Email` and `UserName` as nullable (`string?`), because a user account technically doesn't need one at creation. Our own `IUser` interface declared them as non-nullable `string`. When `ApplicationUser : IdentityUser<int>, IUser` tried to satisfy both, the compiler complained the two didn't match (`CS8766`):

```csharp
// Before
public interface IUser : IBaseEntity
{
    string UserName { get; set; }
    string Email { get; set; }
}

// After
public interface IUser : IBaseEntity
{
    // Nullable to exactly match IdentityUser<TKey>, which ApplicationUser implicitly implements.
    string? UserName { get; set; }
    string? Email { get; set; }
}
```

The fix is simple once you understand the cause: match the nullability of the class you inherit from, don't fight it.

### 4. Repositories that lied about "always found"

`GenericRepository.GetByIdAsync` looked like this:

```csharp
// Before
public virtual async Task<T> GetByIdAsync(int id)
{
    return await _context.Set<T>().FindAsync(id);
}
```

`FindAsync` returns `null` when nothing matches that ID. But the method signature promised `Task<T>` — "I will always return something." That's a lie the compiler catches. The fix: be honest about it.

```csharp
// After
public virtual async Task<T?> GetByIdAsync(int id)
{
    return await _context.Set<T>().FindAsync(id);
}
```

This one change ripples outward — every caller now has to handle "maybe nothing was found," which is exactly what should have been happening all along. For example, in `OrderService`:

```csharp
var deliveryMethod = await _unitOfWork.Repository<DeliveryMethod>().GetByIdAsync(orderDto.DeliveryMethodId);
if (deliveryMethod is null)
{
    return OperationResult.Failure<Order>(ErrorCode.ResourceNotFound, "Delivery method not found");
}
```

Before this fix, a bad ID could silently crash deep inside order creation. Now it returns a clean, expected failure result.

### 5. Mapper safety without spraying `!` everywhere

Mappers convert between database entities and DTOs (data transfer objects). Many of ours looked like this:

```csharp
public static ProductPhotoDto ToProductPhotoDto(this ProductPhoto photo)
{
    if (photo == null) return null;
    return new ProductPhotoDto { /* ... */ };
}
```

The method already checks for `null` and returns `null` safely — but its signature (`ProductPhotoDto`, not `ProductPhotoDto?`) says that never happens. Rather than just tacking on `!` everywhere callers use it (which would silence the warning without describing the real behavior), we used an attribute that tells the compiler the exact rule: *"the output is null only when the input is null."*

```csharp
using System.Diagnostics.CodeAnalysis;

[return: NotNullIfNotNull(nameof(photo))]
public static ProductPhotoDto? ToProductPhotoDto(this ProductPhoto? photo)
{
    if (photo == null) return null;
    return new ProductPhotoDto { /* ... */ };
}
```

Now, if a caller already knows their `photo` isn't null, the compiler knows the result isn't null either — no manual `!` needed on their end. If they pass something that might be null, the compiler correctly asks them to check the result.

### 6. Query and sorting code — nullable value types in expressions

Database queries often sort by columns that can be `null`, like `Product.PreviousPrice` (only set when a discount is active). Our sorting helper was declared as:

```csharp
// Before
Expression<Func<T, object>> OrderBy { get; }
```

Sorting by a nullable field boxes it into an `object`, and that boxed value can be `null` — which the type `object` (non-nullable) doesn't allow. The fix was a one-line change to the shared base class that fixed this warning everywhere it appeared, instead of patching each individual query:

```csharp
// After
Expression<Func<T, object?>> OrderBy { get; }
```

This is a good example of finding the *root* of a repeated warning instead of silencing it dozens of times.

### 7. Guard clauses instead of "trust me" operators

The task instructions were explicit about this, and it's good practice generally: **don't use `!` (the null-forgiving operator) to make a business-logic warning disappear.** Use it only for framework quirks you can't otherwise express. For real logic, write an actual check.

In `ShopContextSeed`, the code loaded JSON seed files from disk:

```csharp
// Before
var brands = JsonSerializer.Deserialize<List<ProductBrand>>(brandsData);
foreach (var item in brands) // could crash if brands is null
{
    await context.ProductBrands.AddAsync(item);
}
```

`Deserialize` can return `null` if the JSON is empty or malformed. Instead of asserting it away, we gave it a safe, sensible fallback:

```csharp
// After
var brands = JsonSerializer.Deserialize<List<ProductBrand>>(brandsData) ?? new List<ProductBrand>();
foreach (var item in brands)
{
    await context.ProductBrands.AddAsync(item);
}
```

Similarly, in `DiscountCrudService`, we found code was reaching for `?.` (the "do nothing if null" operator) on a variable that was already checked for null a few lines earlier — which actually confused the compiler into still flagging later uses as unsafe:

```csharp
// Before — existingDiscount was already null-checked above, but this
// redundant "?." kept the compiler unsure about it further down
existingDiscount?.DiscountGroup?.ConditionGroups.Add(newGroup);
```

```csharp
// After — since we already returned early if existingDiscount was null,
// we can just use it directly
existingDiscount.DiscountGroup?.ConditionGroups.Add(newGroup);
```

The lesson: when you've already handled the null case with an `if (x is null) return;` earlier in the method, don't keep adding `?.` afterward. It doesn't add safety — the compiler already knows `x` is safe there — and it can actually make the compiler *less* confident in code further down.

## The overall decision process we followed

For every nullable warning, we asked the same three questions, in order:

1. **Is this value always present by the time the object is used?**
   → Add the `required` keyword.
2. **Can this value genuinely be missing sometimes?**
   → Make the type nullable (`string?`, `Product?`, etc.) and add a real `if (x is null)` check wherever it's used.
3. **Is this only "unsafe" because a framework (EF Core, model binding, JSON deserialization) fills it in later, in a way the compiler can't see?**
   → Use `= null!;` or a single `!`, with a short comment explaining *why* it's actually safe.

We used option 3 sparingly and always with a reason attached, because a null-forgiving operator with no explanation is just a warning wearing a disguise.

## Result

- **Before:** 468 warnings, spread across SDK config, domain models, repositories, services, DTOs, and mappers.
- **After:** 0 warnings, 0 errors, 65/65 tests still passing.
- **No behavior changed.** Every fix either made an existing implicit assumption explicit (`required`, `null!`), or added a real check where one was missing (guard clauses, `?? fallback`), or corrected a type to match reality (`Task<T?>`, nullable Identity properties).

Cleaning up warnings isn't glamorous work, but it converts hundreds of "the compiler is a little worried" signals into either compiler-enforced guarantees or intentional, documented decisions. The next time someone touches this code and introduces a real null bug, it will show up as the *only* warning in the build — impossible to miss.
