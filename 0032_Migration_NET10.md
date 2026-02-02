

# Goodbye Newtonsoft, Hello Speed: Migrating to .NET 10 & System.Text.Json

I recently started the exciting journey of upgrading my project, **LiliShop**, from .NET 9 to the newly released **.NET 10**. While there are many new features to talk about, the biggest "under the hood" change I made was completely removing `Newtonsoft.Json` in favor of the native **System.Text.Json**.

For a long time, `Newtonsoft` was the king of JSON in .NET. But with .NET 10, `System.Text.Json` has become so powerful and fast that I decided it was time to say goodbye to the old king.

Here is why I made the switch, what is new in .NET 10, and how it improved my code.

## Why I used Newtonsoft in the past?

In previous versions of my project, I stuck with `Newtonsoft.Json` for two main reasons:

1. **JSON Patch Support:** I needed to support `HTTP PATCH` requests. For a long time, the only easy way to handle `JsonPatchDocument` was using Newtonsoft.
2. **Flexibility:** Handling things like circular references in Entity Framework was easier with Newtonsoft's `ReferenceLoopHandling.Ignore`.

However, this convenience came at a cost. Newtonsoft is slower and allocates more memory compared to the modern `System.Text.Json`.

## What is new in .NET 10?

.NET 10 brings massive improvements that finally allowed me to migrate:

* **Native JSON Patch Support:** Microsoft introduced `Microsoft.AspNetCore.JsonPatch.SystemTextJson`. This means I can now use `JsonPatchDocument` directly with `System.Text.Json` without needing complex custom formatters.
* **HybridCache Stability:** The new `HybridCache` (which was experimental in .NET 9) is now stable and works seamlessly with `System.Text.Json`.
* **Performance:** .NET 10 includes AVX10.2 optimizations, making text processing and serialization significantly faster.

## The Migration: 3 Big Improvements

### 1. Zero-Allocation Caching (The Performance Win)

The biggest performance bottleneck in my code was how I handled caching. When retrieving data from Redis, the old `Newtonsoft` serializer had to convert bytes into a massive string before it could create an object. This created a lot of "garbage" memory.

**The Old Code (Inefficient):**

```csharp
public T Deserialize(ReadOnlySequence<byte> source)
{
    // ❌ BAD: Creates a huge String object just to parse JSON
    var jsonString = Encoding.UTF8.GetString(source.ToArray()); 
    return JsonConvert.DeserializeObject<T>(jsonString);
}

```

**The New Code (.NET 10):**
With `System.Text.Json`, I can read directly from the memory bytes using `Utf8JsonReader`. This is called **"Zero-Allocation"** deserialization. It is much faster and lighter on memory.

```csharp
public T Deserialize(ReadOnlySequence<byte> source)
{
    // ✅ GOOD: Reads directly from memory bytes. No String creation!
    
    // Optimization: If data is in one piece, read it instantly
    if (source.IsSingleSegment)
    {
        return JsonSerializer.Deserialize<T>(source.FirstSpan, _options);
    }

    // Optimization: If data is fragmented, read it via a Reader without copying
    var reader = new Utf8JsonReader(source);
    return JsonSerializer.Deserialize<T>(ref reader, _options);
}

```

### 2. Cleaning up Program.cs

In .NET 9, I had to configure `Newtonsoft` and add a custom "Input Formatter" hack just to make JSON Patch work. It made my `Program.cs` file messy.

**Before (Messy):**

```csharp
builder.Services.AddControllers(options =>
{
    // Hack: Manually inserting a custom Newtonsoft formatter
    options.InputFormatters.Insert(0, MyJPIF.GetJsonPatchInputFormatter());
})
.AddNewtonsoftJson(options => ... );

```

**After (Clean .NET 10):**
In .NET 10, everything is native. I removed the hacky formatter and the `AddNewtonsoftJson` call.

```csharp
builder.Services.AddControllers(options =>
{
    options.Filters.Add(new ValidateModelAttribute());
})
// Native .NET 10 Configuration
.AddJsonOptions(options =>
{
    options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
    options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
    options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
});

```

### 3. Simplifying Dependencies

By switching to `System.Text.Json`, I was able to **delete** several files that only existed to wrap Newtonsoft:

* ❌ `NewtonsoftJsonSerializer.cs`
* ❌ `NewtonsoftJsonSerializerFactory.cs`
* ❌ `MyJPIF.cs` (My custom Patch formatter)

My project is now lighter, and I have fewer dependencies to manage.

## Summary of Benefits

1. **Speed:** `System.Text.Json` in .NET 10 is up to 2x-4x faster for high-performance scenarios.
2. **Memory Efficiency:** By using `ReadOnlySpan<byte>`, I reduced the pressure on the Garbage Collector (GC), which keeps the application responsive under high load.
3. **Future Proofing:** Newtonsoft is in maintenance mode. All new performance innovations (like AI integration and Vector Search support) are happening in `System.Text.Json`.

Migrating to .NET 10 wasn't just about changing the version number; it was about embracing the modern, high-performance standards of the .NET ecosystem.

---
## Level Up: Batch Operations with EF Core 10

After optimizing the JSON serialization, I realized there was another silent performance killer in my code: **Database Round-Trips**.

In traditional Entity Framework (and in my old code), if I wanted to update or delete something, I had to follow a slow 3-step process:

1. **Fetch:** Load the data from the database into memory (RAM).
2. **Modify:** Change the object in C#.
3. **Save:** Send the changes back to the database.

This is fine for a single item, but it is terrible for bulk operations or high-performance endpoints. In .NET 10 (EF Core), I refactored my critical services to use **Batch Operations** (`ExecuteUpdateAsync` and `ExecuteDeleteAsync`).

### The Problem: "The Round-Trip"

In my `OrderService`, if a user retried a failed payment, I had to delete their previous "pending" order. My old code looked like this:

```csharp
// 🐢 Old Way: Load it just to kill it
var existingOrder = await _repo.GetByPaymentIntentIdAsync(basket.PaymentIntentId);

if (existingOrder != null) 
{
    // We paid the "network cost" to bring this order into memory...
    // ...only to immediately tell the DB to delete it.
    _repo.Delete(existingOrder);
    await _unitOfWork.CompleteAsync();
}

```

This waste of memory and bandwidth is exactly what `ExecuteDeleteAsync` solves.

### The Solution: ExecuteDeleteAsync

I refactored the code to send a **single command** to the database. No data ever enters my application memory.

```csharp
// 🚀 New Way: Instant Deletion
// This sends one SQL command: "DELETE FROM Orders WHERE PaymentIntentId = '...'"
var rowsDeleted = await _unitOfWork.Repository<Order>()
    .GetByCriteria(o => o.PaymentIntentId == basket.PaymentIntentId)
    .ExecuteDeleteAsync();

```

### The Solution: ExecuteUpdateAsync

I also applied this to my `NotificationService`. I used to fetch thousands of subscription records just to mark them as "Notified". Now, I update them all in one go:

```csharp
// 🚀 New Way: Batch Update
// Updates thousands of rows instantly without loading a single object into RAM.
await _unitOfWork.Repository<DiscountGroupCondition>()
    .GetByCriteria(c => c.ShouldNotify == true && ...)
    .ExecuteUpdateAsync(setters => setters
        .SetProperty(c => c.ShouldNotify, false));

```

### Why does this have "Huge Performance"?

1. **Zero Memory Allocation:** Since we don't load entities, the Garbage Collector (GC) has nothing to clean up.
2. **Network Efficiency:** * **Old:** DB sends 1,000 rows  App  App sends 1,000 updates  DB.
* **New:** App sends 1 command  DB.


3. **Atomic Speed:** The database engine can optimize a single bulk `UPDATE` or `DELETE` command much better than 1,000 individual statements.

### When should you use them?

* ✅ **Bulk Deletions:** "Delete all logs older than 30 days."
* ✅ **Status Updates:** "Mark all selected notifications as read."
* ✅ **Simple Reverts:** "Reset the price for this product ID." (like I used in `DeactivateDiscountAsync`).
* ✅ **Conflict Resolution:** "If this record exists, wipe it out."

### When should you NOT use them?

It is important to know that `ExecuteUpdate` bypasses the EF Core **Change Tracker**. This means you should avoid it in these cases:

* ❌ **Complex Business Logic:** If you need to compare the *old* value with the *new* value (e.g., "Only notify the user if the price dropped by more than 10%"). You need to load the data to know the old price.
* ❌ **Domain Events:** If your architecture relies on `SavedChanges` events (like automatically updating an "Audit Log" whenever an entity changes), batch updates will skip this logic because the entity is never tracked.
* ❌ **Concurrency Checks:** If you rely on `RowVersion` for optimistic concurrency, standard updates are safer.

### Conclusion

By combining **System.Text.Json** for serialization and **Batch Operations** for database calls, I have removed the two biggest bottlenecks in the application. My API now processes requests with significantly less memory and far fewer database round-trips.
