
# 🚀 My Recent Updates: Moving Forward with SmartSupervisorBot

Hi everyone!  
I’ve been working recently to upgrade and improve my project, **SmartSupervisorBot**, a Telegram robot that helps groups by translating and correcting messages. I want to share some updates and changes I’ve made to make the bot faster, smarter, and more cost-effective.

## What Did I Do?  
Here are the main updates I’ve worked on:

1. **Upgraded to .NET 9**  
   I updated the project from **.NET 8** to **.NET 9**, which brings better performance and new features. It’s always good to stay up-to-date with the latest tools.

2. **Upgraded OpenAI SDK**  
   The OpenAI SDK was updated from **version 1.11.0** to **2.1.0**. This allows the bot to use the latest AI models and APIs for better accuracy and features.

3. **Upgraded Telegram.Bot Library**  
   To keep up with Telegram’s changes, I upgraded the library from **version 19.0.0** to **22.3.0**. Now the bot works perfectly with Telegram’s latest features and updates.

4. **Switched to GPT-4o-mini**  
   I switched the bot’s AI model from **gpt-3.5-turbo-instruct** to **gpt-4o-mini**. Why? Because it’s smarter, faster, and much cheaper to use for tasks like translating and correcting messages. This change makes the bot more efficient for public groups with many users.

5. **Refactored the Code**  
   I spent time cleaning up and improving the code. It’s now easier to read, maintain, and extend in the future.

[SmartSupervisorBot GitHub Repository](https://github.com/jahanalem/SmartSupervisorBot)

#OpenAI #TelegramBot

![smartsupervisorbot02](https://github.com/user-attachments/assets/bc5264ee-d295-49e5-800f-4ee06d551dd0)

![smartsupervisorbot01](https://github.com/user-attachments/assets/99ea11b5-6b1c-438b-be8b-3c701e326db2)

---
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
---


# Enhancing Security and User Control: New Features in LiliShop

I'm excited to share some new features I've added to enhance security, user control, and maintain clean code. Here's a quick overview:

## 🔐 1. Refresh Token
- When a user's login session expires, they no longer need to log in again.
- A **Refresh Token** (stored in a secure HTTP-only cookie) automatically creates a new session.

## 📲 2. Logout from All Devices
- Users can now log out from all their devices with a single click.
- This feature clears login sessions and tokens across all active devices (e.g., mobile, desktop).
- It's especially useful for securing accounts or logging out from old sessions.

## 🔄 3. Role Change Update
- When a user's role changes (e.g., from **User** to **Admin**), their permissions are updated immediately.
- Old tokens are invalidated to ensure users only have access to what they should.

## 📘 4. RFC 9457 - Problem Details for HTTP APIs
- Implemented **RFC 9457** for providing clear and standardized error messages in the API.

---

### 🔗 Live Project
[LiliShop Live Project](https://lnkd.in/eAK8B3Ka)

### 📂 Source Code
- **Frontend:** [GitHub Repository](https://lnkd.in/ePhuYGhP)

---

### Tags
`#webdevelopment` `#security` `#dotnet` `#angular` `#cleancode` `#softwaredevelopment` `#programming`

![LogoutFromAllDevices](https://github.com/user-attachments/assets/83d9eb74-40b4-4e95-a72b-db29158115d5)


# Performance Boost: Upgrading LiliShop to .NET 9 and Angular 19

I’m excited to share the results of a recent update to my project, **LiliShop**, deployed on Azure!  
The application was upgraded from **.NET 8** and **Angular 18** to **.NET 9** and **Angular 19**, and I introduced **MapStaticAssets()** for optimized static file handling.

## 🚀 Highlights of the Improvements:
- **DOMContentLoaded Time:** Reduced from **5.95 seconds** to **700 milliseconds** (88.24% improvement).
- **Total Load Time:** Reduced from **6.46 seconds** to **1.09 seconds** (83.13% improvement).
- **Finish Time:** Reduced from **12.39 seconds** to **6.88 seconds** (44.47% improvement).
- **API Response Times:** Improved by **44.88%** (e.g., products endpoint).
- **Transferred Data:** Reduced from **6.9 MB** to **5.9 MB**.

🎯 These updates have significantly improved the application’s speed and efficiency!

---

### 🔗 Explore the Project
- **Live Application:** [LiliShop Store](https://lilishop-bwdfb5azanh0cfa8.germanywestcentral-01.azurewebsites.net/shop)  
- **Frontend Source Code:** [GitHub Repository](https://github.com/jahanalem/LiliShop-frontend-angular)


![Lilishop-NET9-Angular19](https://github.com/user-attachments/assets/a5e42cd3-bac9-4824-81ef-e7f9c8b1b678)

![LiliShop-NET8-Angular18](https://github.com/user-attachments/assets/8d59dc14-6ff2-4444-b05f-2b099477476c)


# Optimizing Rectangle Detection: Performance and Memory Improvements

A while ago, I developed a program to detect rectangles from a set of points on a two-dimensional surface. Recently, I worked on improving its performance and memory usage.

## 🔧 What I Improved:
- Replaced `OrderBy` operations with `Array.Sort` for better memory efficiency.
- Added caching to avoid unnecessary computations.
- Used `CollectionsMarshal.AsSpan` for faster data access.

## 📈 Results:
- **55%** reduction in total memory allocations.
- **26%** decrease in peak live objects.
- **75%** less memory usage in the `GetOrderedPoints` method.
- **8.5%** faster execution time.

## 🛠️ Test Environment:
- **Visual Studio Version:** 17.12.0
- **.NET Version:** .NET 9
- **Build Configuration:** Release mode

These changes demonstrate how optimizing algorithms and data handling can significantly improve performance, especially in memory-intensive applications.

---

### 🔗 Explore the Project:
- [Project Link 1](https://github.com/jahanalem/RectanglesCalculator/tree/performance-improvements)  
- [Project Link 2](https://github.com/jahanalem/RectanglesCalculator/tree/main)

If you have any suggestions to further improve the performance, I’d love to hear them! 😊


![1](https://github.com/user-attachments/assets/2853e170-c1dc-4800-837c-d16f2515b7b8)

![2](https://github.com/user-attachments/assets/0cc37a9e-4d55-4b9a-aca1-e7029a24a6e0)

![3](https://github.com/user-attachments/assets/4c872a0a-9af1-4d31-923a-4a64a0f53eee)

![4](https://github.com/user-attachments/assets/895c5b19-0caa-446d-b35d-d4b075a2ff4b)

![5](https://github.com/user-attachments/assets/21566796-a69f-4579-97e4-017a5555cdda)

![6](https://github.com/user-attachments/assets/7c028cf5-6ec8-42fe-8e8d-3050aa06ed54)

![7](https://github.com/user-attachments/assets/049a4990-f02c-43bb-b95b-bcc6e254c04e)


# Revamping My Blog Application: From .NET 5 to .NET 8

🚀 I'm excited to share an update on one of my old projects!

A few years ago, I created a blog application using **ASP.NET MVC**. Recently, I decided to give it a major upgrade. 🔄  
I’ve updated the project, **Alpha** ([GitHub Repository](https://github.com/jahanalem/Alpha)), from **.NET 5** to **.NET 8**, and improved both the **Data Access** and **Service layers**. The project is now cleaner, faster, and takes advantage of the latest .NET features.

While there's still room for improvement, this update is a significant step forward. 💪  

---

### 🔗 Explore the Project:
- [Alpha GitHub Repository](https://github.com/jahanalem/Alpha)

Below are some screenshots showing the updated interface and features.  
Feel free to check out the repository and explore the changes! 🤓  

---

### Tags
`#GitHub` `#DotNet` `#MVC` `#CSharp` `#OpenSource` `#Refactoring`


![001](https://github.com/user-attachments/assets/6abb2c24-9880-44f8-b23b-27efd1008299)

![002](https://github.com/user-attachments/assets/08d752b9-c9bd-4a1d-abc3-8a59a4a9333e)


# Certification Achievement: Postman REST API Testing

🎉 I’m excited to share that I’ve completed the **'Postman: The Complete Guide - REST API Testing'** course on Udemy!  

This course has enhanced my skills in testing REST APIs, and I’m already applying these techniques in my current project. 🚀  

---

### Certification:
- **Course Title:** Postman: The Complete Guide - REST API Testing  
- **Platform:** Udemy  

---

![001](https://github.com/user-attachments/assets/00f41ba8-fb65-4ac2-8d63-8dd6e0b92f55)

![002](https://github.com/user-attachments/assets/3d8103af-d315-46c4-955e-e5bb5b239e67)


# 🚀 Improving API Error Handling: My Approach

In my recent project, I implemented a method called **HandleOperationResult** in the **BaseApiController**. This method converts messages from the service layer into appropriate HTTP status codes along with any data.

I also integrated **MyApiConventions** to improve Swagger documentation. By defining common status codes such as `200 OK`, `400 BadRequest`, `404 NotFound`, and `500 InternalServerError`, the API documentation is now clearer and more accurate.

## ✨ Key Improvements:
- Using **MyApiConventions** eliminates the need to write `[ProducesResponseType()]` for every method in controllers, following the **DRY (Don't Repeat Yourself)** principle. This keeps the code clean and easier to maintain.
- The **service layer** remains independent of the API layer, ensuring a clean architecture.
- Users receive clear and consistent error messages from the API.
- **Swagger documentation** is more precise and easier to understand.
- Custom error messages help identify problems quickly during development.

---

### 🔗 Explore the Project:
[Live Project](https://lnkd.in/eAK8B3Ka)

---

### Tags
`#WebDevelopment` `#API` `#ErrorHandling` `#Swagger` `#DotNet` `#CSharp` `#BackendDevelopment`


![005](https://github.com/user-attachments/assets/96f33983-6067-4a4d-8254-b98c0fc5fbff)

![006](https://github.com/user-attachments/assets/471be81a-5eeb-4f86-8a21-95711b59038d)

![007](https://github.com/user-attachments/assets/ac07fe40-d337-41d5-be05-37f99b785be8)


# 🧪 Developing Unit Tests for LiliShop

In parallel with improving application performance and adding new features, I’m working on developing unit tests using **xUnit.net**. While I haven’t written many tests yet, my goal is to achieve comprehensive coverage of all controllers in the **API layer** and the **service layer**.

---

### 🔗 Test Project:
The test project is publicly available on my GitHub. Feel free to check it out and provide feedback:  
[LiliShop Backend Test Project](https://github.com/jahanalem/LiliShop-backend-dotnet-test)

---

This initiative aims to ensure higher code quality and reliability for the LiliShop project. 🚀


# 🚀 Automating Deployments with GitHub Actions: LiliShop Update

I recently completed the **GitHub Actions** course on Udemy and immediately applied what I learned to my project, **LiliShop**.

## 🌟 What I Implemented:
- Created a workflow that automatically builds my **.NET** and **Angular** project.
- Set up automated deployment to **Azure** whenever changes are pushed to the main branch.

This automation ensures a smoother and more efficient deployment process, saving time and reducing manual effort. 🚀  

---

### 🔗 Explore the Project:
- [LiliShop Frontend Repository](https://github.com/jahanalem/LiliShop-frontend-angular)

![008](https://github.com/user-attachments/assets/84af0ca5-7b94-4d95-9d5b-756b456c14b8)


# 🎉 LiliShop: Deployed on Microsoft Azure!

I’m thrilled to announce that my project, **LiliShop**, is now live on **Microsoft Azure**!  
You can visit it here:  
👉 [LiliShop Live Project](https://lnkd.in/eAK8B3Ka)

---

## 💻 Tech Stack:

### 🔧 Backend:
- **Framework:** .NET 8  
- **Language:** C#  
- **ORM:** Entity Framework  
- **Database:** MS SQL Server, Redis  
- **Image Storage:** Cloudinary  
- **Services:** Stripe (Payments), SendGrid (Emails)

### 🎨 Frontend:
- **Framework:** Angular  
- **Language:** TypeScript  

---

## 🚀 Performance Optimizations:

### 🔙 Backend:
- ⚙️ Created indexes on searchable properties in the product table.  
- 🗃️ Implemented caching using Redis.  
- 📧 Integrated SendGrid for email services.  
- ☁️ Utilized Cloudinary for cloud-based image storage.  
- 🛠️ Implemented `BackgroundService` for email dispatch.

### 🔝 Frontend:
- ⚡ Implemented caching strategies.  
- 🔄 Used `ChangeDetectionStrategy.OnPush` and Signals for efficient change detection.  
- 🖼️ Leveraged Cloudinary for image rendering on the shop page.  
- 🚦 Created modules for lazy loading of components.  

---

## 🔧 Ongoing Work:
I’m continually working on performance improvements and planning many more features for future implementation.  
If you find a bug, please report it. 🙂

![010](https://github.com/user-attachments/assets/1c6fb81b-8b46-4183-8023-0d61c0253cfa)

![011](https://github.com/user-attachments/assets/193b0286-ce54-4ae7-8eae-fda20ede32d4)

![012](https://github.com/user-attachments/assets/f208ad36-52c7-4395-b14d-2c2b6f5feb86)

![013](https://github.com/user-attachments/assets/8edfa677-39b5-4f58-a80c-f9fcbafa4f39)

![014](https://github.com/user-attachments/assets/982ea2d5-bc4e-4a90-a30f-abdc2a12e924)

![015](https://github.com/user-attachments/assets/486d8d56-a3fe-4130-9b6d-a544e073bdc6)

# 🚀 Email Notification Feature for LiliShop

I’ve developed an **email notification feature** for registered and subscribed users in my project, **LiliShop**, implemented in both the backend and frontend.

---

## 🔧 Implementation Details:
- **Backend:**  
  - Used **Entity Framework** for generating tables and SQL queries.
- **Frontend:**  
  - Users need to register with a confirmed email.
  - A simple click on **'Notify me of price drops'** enables notifications.

---

## 🔔 How It Works:
- When the **admin reduces the price** of an item, subscribed users automatically receive an email notification.
- Users can unsubscribe from notifications either via the **website** or directly from the **email**.

---

## 🔮 Future Plans:
I plan to further enhance this notification system with more advanced features. For now, this marks a great first step!

---

### 🔗 Explore the Project:
[Frontend Repository](https://github.com/jahanalem/LiliShop-frontend-angular)

---

### Tags
`#Angular` `#Csharp` `#NETCore` `#EntityFramework`

![016](https://github.com/user-attachments/assets/6a754267-04ba-4b1f-b74a-5078b0c67027)

![017](https://github.com/user-attachments/assets/69fabbe1-bc0d-4b92-84c5-5170266d4b62)

![018](https://github.com/user-attachments/assets/b24c941d-25f8-46fb-b868-5c3f43be1015)

![019](https://github.com/user-attachments/assets/f3fc3caf-681b-4e02-8577-2049c91ab172)

![020](https://github.com/user-attachments/assets/0f7886a1-0657-4167-87f3-5f407dc03011)


# 🚀 Angular 18 Upgrade: Enhancing LiliShop

I’m excited to share that I’ve upgraded my **Angular knowledge** to **Version 18** and applied these advancements to my project, **LiliShop**.  
You can check out the updated project here:  
👉 [LiliShop Frontend Repository](https://lnkd.in/ePhuYGhP)

---

## 📈 Upgrade Journey:
I started this project with **Angular 14.2** and have now upgraded it to **Angular 18.1**. Along the way, I implemented the latest features and syntax improvements, including:

### 🔧 New Features and Changes:
- **Signals** for more reactive programming.
- **Function-based interceptors** replacing class-based ones.
- **Function-based guards** instead of class-based ones.
- Used `@for` and `@if` as alternatives to `ngFor` and `ngIf`, where applicable.
- Eliminated **zone.js** by using `provideExperimentalZonelessChangeDetection()` and adopted the **OnPush Change Detection Strategy**.
- Used the `inject()` function for service calls, moving away from traditional constructor injection in components.
- Replaced `@ViewChild` decorator with `viewChild()`.
- Adopted `@Output` decorator as `output()`.
- Employed `@Input` decorator as `input()`.

---

## 🎓 Learning and Inspiration:
A big thank you to the course on **Udemy** that helped me get up to speed with Angular 18. These updates are a significant step forward in enhancing the performance and maintainability of my project.

---

### Tags
`#Angular` `#Angular18` `#WebDevelopment` `#Udemy` `#Programming` `#TechUpdate`


![021](https://github.com/user-attachments/assets/362d4170-b95a-450f-9bde-ef7ae7b5c8f8)

# 🚀 Angular 18.1 Upgrade: Enhancing LiliShop

I’m excited to share that I’ve updated my **LiliShop** project to **Angular 18.1.0**! This update brings some great improvements to performance and state management.

---

## 🌟 Key Improvements:

### 🚀 Better Performance:
- Removed **zone.js**, allowing the app to run faster.
- Without zone.js, Angular avoids unnecessary change detection, making the app smoother and quicker.

### 🔄 OnPush Change Detection:
- Implemented **OnPush Change Detection**, so Angular checks for changes only when necessary.
- This reduces checks and significantly improves performance.

### 📶 Signals:
- Leveraged **Signals** for easier and cleaner state management.
- Signals reduce extra code and simplify handling state changes within the app.

---

### 🔗 Explore the Project:
Check out the frontend part of my project on GitHub:  
👉 [LiliShop Frontend Repository](https://lnkd.in/ePhuYGhP)

---

Take a look and let me know your thoughts! 😊

![022](https://github.com/user-attachments/assets/82b977a7-25e0-45b9-8d7d-f0aa35f7f3fe)

![023](https://github.com/user-attachments/assets/b221d812-51bb-403c-92e0-9878746023d7)

# 🎉 Neues Zertifikat: Deutsch-Test für den Beruf C1 von Telc

Ich freue mich, mein neues Zertifikat präsentieren zu können: **Deutsch-Test für den Beruf C1 von Telc**!  

Dieses Zertifikat bestätigt meine fortgeschrittenen Sprachkenntnisse und stärkt meine beruflichen Kommunikationsfähigkeiten. 🚀  

---


# 🚀 SmartSupervisorBot: Verbesserte Gruppeninteraktionen auf Telegram

Ich habe mein Projekt **SmartSupervisorBot** auf GitHub veröffentlicht!  
Dieser Bot verbessert Gruppeninteraktionen durch **angepasste Sprachübersetzungen** und **Textkorrekturen** direkt in Telegram-Gruppen.

---

## 🤓 Projektstruktur:
- **🔖 SmartSupervisorBot.Core:**  
  Steuert die Hauptfunktionen, inklusive Verbindung zur **Telegram Bot API** und **OpenAI**.
- **🔖 SmartSupervisorBot.DataAccess:**  
  Verwaltet Daten, speziell mit **Redis** für Gruppeneinstellungen.
- **🔖 SmartSupervisorBot.ConsoleApp:**  
  Startpunkt der App, der den Serviceaufbau und die Verarbeitung von Nachrichten koordiniert.
- **🔖 SmartSupervisorBot.Test.Core:**  
  Enthält Tests, die durch **xUnit** und **Moq** die Zuverlässigkeit sicherstellen.

---

## 📌 Zukunftspläne:
Ich plane, das Projekt um folgende Funktionen zu erweitern:
- **WebAPI** für eine einfachere Integration.
- **Angular-Frontend** zur Verbesserung der Benutzerfreundlichkeit und Verwaltung.

---

### 🔗 Projekt ansehen:
[SmartSupervisorBot auf GitHub](https://github.com/jahanalem/SmartSupervisorBot)

⭐ Schauen Sie gerne vorbei, geben Sie einen Stern, wenn es Ihnen gefällt, und teilen Sie Ihr Feedback oder Ihre Beiträge! 😊

![024](https://github.com/user-attachments/assets/a11cccc9-d57c-44c2-8ca6-ccc46bd049d6)

![025](https://github.com/user-attachments/assets/994ce369-4ef5-463b-916b-da1442c053f5)



