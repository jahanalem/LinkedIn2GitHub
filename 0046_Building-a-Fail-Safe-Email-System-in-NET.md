# Building a Fail-Safe Email System in .NET: A Practical Guide to Fallback and Resilience

In any production-grade backend, relying on a single third-party service for critical features like user registration is a risk. Services can go down, API limits can be reached, or trial tiers can expire.

In my project, **[LiliShop](https://lilishop-bwdfb5azanh0cfa8.germanywestcentral-01.azurewebsites.net/)**, I faced a common issue: my primary email provider (SendGrid) reached its limit. To ensure the application remained functional, I implemented a resilient email delivery system using two main strategies: the **Fallback Pattern** and the **[Polly library](https://github.com/App-vNext/Polly)**.

This article explains how to build a system that automatically switches to a backup provider and handles temporary network failures gracefully.

---

## 1. The Strategy: The Fallback Pattern

The goal was to keep the core business logic (like the `AccountController`) unaware of which email service is actually being used.

First, I defined a standard interface:

```csharp
public interface IEmailService
{
    Task<bool> SendEmailAsync(string to, string subject, string htmlContent);
}

```

Then, I created two implementations:

1. **SendGridEmailService**: The primary provider.
2. **SmtpEmailService**: The backup provider using native .NET libraries and Gmail's SMTP server.

Finally, I created a **FallbackEmailService**. This class acts as a "wrapper." It tries SendGrid first; if that fails, it catches the error and immediately tries the SMTP service.

---

## 2. Adding Resilience with Polly

Even with a backup service, network calls can fail momentarily due to a "glitch." Instead of giving up immediately, we should "retry." Writing manual `for` loops with `Thread.Sleep` is messy and hard to maintain. This is why I used **Polly**.

### What is Polly?

Polly is an open-source .NET resilience library. It allows you to define policies like "Retry" or "Circuit Breaker" in a clean, fluent way.

### How the Code Works

In the email delivery logic, I implemented a **Wait and Retry** policy with **Exponential Backoff**:

```csharp
var retryPolicy = Policy
    .Handle<Exception>()
    .WaitAndRetryAsync(
        retryCount: 3,
        sleepDurationProvider: attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)),
        onRetry: (exception, timeSpan, attempt, context) =>
        {
            _logger.LogWarning(exception, "Retry {Attempt} for email to {Email} after {Delay}ms", 
                attempt, userEmail, timeSpan.TotalMilliseconds);
        });

try
{
    // Execute the network call within the policy
    await retryPolicy.ExecuteAsync(async () =>
    {
        await _emailService.SendEmailAsync(user.Email, "Email Confirmation", htmlTemplate);
    });

    return true;
}
catch (Exception ex)
{
    // If all retries fail, Polly re-throws the final exception.
    _logger.LogError(ex, "Critical error: Failed to send confirmation email for user {Email} after all retry attempts.", user.Email);

    return false;
}

```

### Breakdown of Parameters:

* **Handle**: This tells Polly to trigger the retry for any type of error. In a more specific setup, you could target only `SmtpException`.
* **retryCount (3)**: The system will try a maximum of three times before finally failing.
* **sleepDurationProvider**: This implements "Exponential Backoff."
* Attempt 1: waits $2^1 = 2$ seconds.
* Attempt 2: waits $2^2 = 4$ seconds.
* Attempt 3: waits $2^3 = 8$ seconds.
This is better than a constant delay because it gives the failing server more time to recover if it is under heavy load.


* **onRetry**: A callback used for logging each failed attempt, which is crucial for monitoring the health of the system.

---

## 3. Configuring Gmail as a Backup SMTP

Using Gmail's SMTP server (`smtp.gmail.com`) is a reliable and free backup option. However, you cannot use your regular Gmail password in the code because Google blocked "Less Secure Apps" for security reasons.

### Step 1: Generate an App Password

To allow your .NET app to send emails, you must:

1. Enable **2-Step Verification** on your Google Account.
2. Go to [https://myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords).
3. Generate a new password (e.g., call it "LiliShop Backend").
4. Google will give you a **16-character code**. This is your new SMTP password.

### Step 2: Secure Storage

Never hardcode this 16-character password in your `appsettings.json`. For production environments like **Azure**, you should store it in the **Environment Variables**.

In ASP.NET Core, use the double underscore convention to map nested JSON keys. For example:

* Variable Name: `EmailSenderProvider__Smtp__Password`
* Value: `your-16-char-code`

---

```csharp
using LiliShop.Application.Interfaces.Services.External;
using Microsoft.Extensions.Logging;

namespace LiliShop.Infrastructure.Services.External.Email
{
    public class FallbackEmailService : IEmailService
    {
        private readonly SendGridEmailService _primaryService;
        private readonly SmtpEmailService _fallbackService;
        private readonly ILogger<FallbackEmailService> _logger;

        public FallbackEmailService(
            SendGridEmailService primaryService,
            SmtpEmailService fallbackService,
            ILogger<FallbackEmailService> logger)
        {
            _primaryService = primaryService;
            _fallbackService = fallbackService;
            _logger = logger;
        }

        public async Task<bool> SendEmailAsync(string to, string subject, string htmlContent)
        {
            // 1. Try SendGrid
            bool isSuccess = await _primaryService.SendEmailAsync(to, subject, htmlContent);

            if (isSuccess)
            {
                return true;
            }

            // 2. Fallback to native SMTP
            _logger.LogWarning("Primary email service failed. Executing SMTP fallback.");
            bool fallbackSuccess = await _fallbackService.SendEmailAsync(to, subject, htmlContent);

            if (!fallbackSuccess)
            {
                _logger.LogCritical("Critical failure: Both primary and fallback email services failed to send email to {To}", to);
                throw new InvalidOperationException("The email delivery system is currently unavailable.");
            }

            return true;
        }
    }
}
```

## Conclusion

By combining the **Fallback Pattern** and **Polly**, the email system is now "Fault-Tolerant." If SendGrid fails, the system retries. If it still fails after 3 retries, it switches to Gmail. This architecture ensures that users always receive their confirmation emails, providing a smooth and professional experience.

---
