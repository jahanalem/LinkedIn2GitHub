
# 🚀 Boosting Performance with HybridCache in .NET 9

HybridCache is a new feature in .NET 9 that combines the speed of in-memory caching with the scalability of Redis. However, this feature is still evolving, and some parts, like the tags functionality, aren’t fully implemented yet. Before switching to HybridCache, I only used Redis for caching. It worked well, but I wanted to take advantage of the benefits of having both in-memory and distributed caching layers.

To set it up, I configured Redis as the backend for the distributed cache and used `HybridCacheEntryOptions` to control how long data stays in memory and in Redis. This ensures that frequently accessed data is served quickly from memory, while Redis provides a reliable backup for scaling across systems.

I also created a custom `NewtonsoftJsonSerializer` to ensure all cached data is handled consistently throughout the app. For caching API responses, I added a `[Cached]` attribute that automatically saves responses based on the request path and query parameters. This keeps the caching logic simple and reusable.

To keep the cache up-to-date, I implemented a way to remove old entries when the data changes. By matching patterns in Redis keys, I ensure users always get the latest data. I also centralized cache durations in a `CacheDurations` class, making it easy to adjust how long data stays in the cache (e.g., short, default, or long durations).

---

## Links

- **Live Project**:  
  [LiliShop](https://lilishop-bwdfb5azanh0cfa8.germanywestcentral-01.azurewebsites.net/shop)

- **Backend Repository**:  
  [LiliShop-backend-dotnet](https://github.com/jahanalem/LiliShop-backend-dotnet)

- **HybridCache Library in ASP.NET Core**:  
  [Microsoft Documentation](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/hybrid?view=aspnetcore-9.0#serialization)

---

### #dotnet #caching #Redis #HybridCache #PerformanceOptimization #Development


## Example API Endpoint

```csharp
[Cached(CacheDurations.Long)]
[HttpGet]
public async Task<ActionResult<Pagination<ProductToReturnDto>>> GetProducts([FromQuery] ProductSpecParams productParams)
{
    var operationResult = await _productService.GetPaginatedProductsAsync(productParams);
    return HandleOperationResult(operationResult);
}
```

---

## HybridCache Service Configuration

```csharp
// Add HybridCache service
#pragma warning disable EXTEXP0018  // Suppress warning for evaluation features
builder.Services.AddHybridCache(options =>
{
    options.DefaultEntryOptions = new HybridCacheEntryOptions
    {
        Expiration = TimeSpan.FromSeconds(CacheDurations.Default),
        LocalCacheExpiration = TimeSpan.FromSeconds(CacheDurations.Short)
    };
}).AddSerializerFactory<NewtonsoftJsonSerializerFactory>();
#pragma warning restore EXTEXP0018
```

---

## CachedAttribute Implementation

```csharp
namespace Lili.Shop.API.Helpers
{
    public class CachedAttribute : Attribute, IAsyncActionFilter
    {
        private readonly int _timeToLiveSeconds;
        private const int OkStatusCode = 200;

        public CachedAttribute(int timeToLiveSeconds)
        {
            _timeToLiveSeconds = timeToLiveSeconds;
        }

        public async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
        {
            var hybridCache = context.HttpContext.RequestServices.GetRequiredService<HybridCache>();

            var cacheKey = GenerateCacheKeyFromRequest(context.HttpContext.Request);
            var cacheResponse = await hybridCache.GetOrCreateAsync(
                cacheKey,
                async cancelToken =>
                {
                    var executedContext = await next();

                    if (executedContext.Result is OkObjectResult okObjectResult)
                    {
                        return JsonConvert.SerializeObject(okObjectResult.Value, JsonSerializerSettingsProvider.GetDefaultSettings());
                    }

                    return null;
                },
                new HybridCacheEntryOptions
                {
                    Expiration = TimeSpan.FromSeconds(_timeToLiveSeconds),
                    LocalCacheExpiration = TimeSpan.FromSeconds(_timeToLiveSeconds / 2),
                },
                cancellationToken: context.HttpContext.RequestAborted
            );

            if (cacheResponse != null)
            {
                var serializedContent = cacheResponse as string;
                context.Result = new ContentResult
                {
                    Content = serializedContent,
                    ContentType = "application/json",
                    StatusCode = OkStatusCode
                };
            }
        }

        private string GenerateCacheKeyFromRequest(HttpRequest request)
        {
            return CacheKeyHelper.GenerateCacheKey(request);
        }
    }
}
```

---

## Redis Configuration

```csharp
void ConfigureRedis(WebApplicationBuilder builder)
{
    builder.Services.AddSingleton<IConnectionMultiplexer>(c =>
    {
        var logger = c.GetRequiredService<ILogger<Program>>();
        var connectionString = builder.Configuration.GetConnectionString("Redis");

        if (string.IsNullOrEmpty(connectionString))
        {
            logger.LogError("Redis connection string is missing. Ensure that it is set in the configuration.");
            throw new InvalidOperationException("Redis connection string is required.");
        }

        var configuration = ConfigurationOptions.Parse(connectionString, true);

        try
        {
            var connection = ConnectionMultiplexer.Connect(configuration);
            logger.LogInformation("Connected to Redis successfully.");
            return connection;
        }
        catch (RedisConnectionException ex)
        {
            logger.LogError(ex, "Failed to connect to Redis.");
            throw;
        }
    });
    builder.Services.AddStackExchangeRedisCache(options =>
    {
        options.Configuration = builder.Configuration.GetConnectionString("Redis");
    });
}
```

---

## NewtonsoftJsonSerializer for HybridCache

```csharp
namespace Lili.Shop.API.Helpers
{
    public class NewtonsoftJsonSerializerFactory : IHybridCacheSerializerFactory
    {
        public bool TryCreateSerializer<T>([NotNullWhen(true)] out IHybridCacheSerializer<T>? serializer)
        {
            serializer = new NewtonsoftJsonSerializer<T>();
            return true;
        }
    }

    public class NewtonsoftJsonSerializer<T> : IHybridCacheSerializer<T>
    {
        public T Deserialize(ReadOnlySequence<byte> source)
        {
            var jsonString = Encoding.UTF8.GetString(source.ToArray());
            return JsonConvert.DeserializeObject<T>(jsonString);
        }

        public void Serialize(T value, IBufferWriter<byte> target)
        {
            var jsonString = JsonConvert.SerializeObject(value);
            var jsonBytes = Encoding.UTF8.GetBytes(jsonString);
            target.Write(jsonBytes);
        }
    }
}
```

---

### NewtonsoftJsonPatchInputFormatter

```csharp
namespace Lili.Shop.API.Helpers
{
    public static class MyJPIF
    {
        public static NewtonsoftJsonPatchInputFormatter GetJsonPatchInputFormatter()
        {
            var builder = new ServiceCollection()
                .AddLogging()
                .AddMvc()
                .AddNewtonsoftJson(options =>
                {
                    var settings = JsonSerializerSettingsProvider.GetDefaultSettings();
                    options.SerializerSettings.ContractResolver = settings.ContractResolver;
                    options.SerializerSettings.Formatting = settings.Formatting;
                    options.SerializerSettings.ReferenceLoopHandling = settings.ReferenceLoopHandling;
                })
                .Services.BuildServiceProvider();

            return builder
                .GetRequiredService<IOptions<MvcOptions>>()
                .Value
                .InputFormatters
                .OfType<NewtonsoftJsonPatchInputFormatter>()
                .First();
        }
    }
}
```

### Cache Durations

```csharp
namespace Lili.Shop.API.Helpers
{
    public static class CacheDurations
    {
        public const int Default = 10 * 60;    // 10 minutes
        public const int Short = 5 * 60;      // 5 minutes
        public const int Long = 24 * 60 * 60; // 24 hours
    }
}
```

---

```markdown

## Middleware: MonitorRequestMiddleware

This middleware is responsible for invalidating cache entries when certain HTTP methods (`PUT`, `POST`, `DELETE`) are used.

```csharp
namespace Lili.Shop.API.Middlewares
{
    public class MonitorRequestMiddleware : IMiddleware
    {
        private readonly HybridCache _hybridCache;
        private readonly IConnectionMultiplexer _redisConnection;
        private static readonly HashSet<string> ModifyingMethods = new HashSet<string> { "PUT", "POST", "DELETE" };

        public MonitorRequestMiddleware(HybridCache hybridCache, IConnectionMultiplexer redisConnection)
        {
            _hybridCache = hybridCache;
            _redisConnection = redisConnection;
        }

        public async Task InvokeAsync(HttpContext context, RequestDelegate next)
        {
            if (ShouldInvalidateCache(context.Request))
            {
                var cacheKeyPattern = GetCacheKeyPattern(context.Request);
                if (!string.IsNullOrEmpty(cacheKeyPattern))
                {
                    // Invalidate the cache by pattern
                    await InvalidateCacheByPattern(cacheKeyPattern);
                }
            }
            await next(context);
        }

        private bool ShouldInvalidateCache(HttpRequest request)
        {
            return ModifyingMethods.Contains(request.Method);
        }

        private string GetCacheKeyPattern(HttpRequest request)
        {
            var segments = request.Path.ToString().Split('/');
            // Assuming the meaningful segment for cache keys is always the third one (/api/[controller]/[action]/[id])
            return segments.Length > 2 ? segments[2] : string.Empty;
        }

        private async Task InvalidateCacheByPattern(string pattern)
        {
            var endpoints = _redisConnection.GetEndPoints();
            var server = _redisConnection.GetServer(endpoints.First());
            var keys = server.Keys(pattern: $"*{pattern}*").Select(k => k.ToString()).ToArray();

            if (!keys.Any())
            {
                return;
            }

            await _hybridCache.RemoveAsync(keys);
        }
    }
}
```

---

## Helper Class: CacheKeyHelper

```csharp
namespace Lili.Shop.API.Helpers
{
    public static class CacheKeyHelper
    {
        public static string GenerateCacheKey(HttpRequest request)
        {
            var keyBuilder = new StringBuilder();

            keyBuilder.Append($"{request.Path}");

            // Add query parameters to key
            foreach (var (key, value) in request.Query.OrderBy(x => x.Key))
            {
                keyBuilder.Append($"|{key}-{value}");
            }

            return keyBuilder.ToString();
        }
    }
}
```

---
