# 🔐 LiliShop — Multi-Factor Authentication (MFA) Implementation

> A complete, tutorial-style guide to how LiliShop adds a second layer of login security for administrators — using Time-based One-Time Passwords (TOTP), the same technology behind Google Authenticator and Authy.

This document assumes **no prior knowledge of MFA**. Every concept is explained the first time it appears, in plain English, and illustrated with LiliShop's real backend (ASP.NET Core) and frontend (Angular) code. By the end, you'll understand not just *what* the code does, but *why* each design decision was made — enough to build a similar system yourself.

> [!NOTE]
> This is **Part 2** of the LiliShop security series. Part 1 covered brute-force protection (rate limiting and account lockout). MFA is the third and strongest layer of the login defense described there — this document zooms in on it in full.

---

## 📑 Table of Contents

1. [Introduction: Why MFA?](#1-introduction-why-mfa)
2. [Core Concepts](#2-core-concepts)
   - [2.1 Authentication vs. Authorization](#21-authentication-vs-authorization)
   - [2.2 Factors: Something You Know, Have, Are](#22-factors-something-you-know-have-are)
   - [2.3 OTP and TOTP](#23-otp-and-totp)
   - [2.4 The Shared Secret](#24-the-shared-secret)
   - [2.5 Authenticator Apps & QR Codes](#25-authenticator-apps--qr-codes)
   - [2.6 Recovery Codes](#26-recovery-codes)
3. [System Architecture](#3-system-architecture)
4. [How TOTP Actually Works](#4-how-totp-actually-works)
5. [Backend Implementation](#5-backend-implementation)
   - [5.1 The DTOs](#51-the-dtos-the-shapes-of-data)
   - [5.2 Enrollment: Two Steps](#52-enrollment-getting-set-up-two-steps)
   - [5.3 The Login Gate](#53-the-login-gate-enforcing-mfa)
   - [5.4 Recovery Code Fallback](#54-the-recovery-code-fallback)
6. [The Login State Machine](#6-the-login-state-machine)
7. [Frontend Implementation](#7-frontend-implementation)
   - [7.1 The Login Component](#71-the-login-component-the-orchestrator)
   - [7.2 The Setup Component](#72-the-setup-component)
   - [7.3 The Verify Component](#73-the-verify-component)
   - [7.4 Rendering the QR Code](#74-rendering-the-qr-code)
8. [The JWT Interceptor: Managing Tokens and Protecting the Login Flow](#8-the-jwt-interceptor-managing-tokens-and-protecting-the-login-flow)
9. [Closing the Google Bypass](#9-closing-the-google-bypass)
10. [The Complete End-to-End Flow](#10-the-complete-end-to-end-flow)
11. [API Reference](#11-api-reference)
12. [📖 Glossary](#12--glossary)

---

## 1. Introduction: Why MFA?

For decades, logging in meant one thing: type your password. If it's correct, you're in. This worked when passwords were the only realistic barrier — but it has a fundamental weakness. **A password is a single secret, and secrets leak.**

Here are the real, everyday ways a password gets into the wrong hands, none of which involve "guessing" it:

- **Data breaches on other sites.** People reuse passwords. If someone's password leaks from an unrelated website's breach, attackers try that same password everywhere else — including your shop. (This is called *credential stuffing*.)
- **Phishing.** A fake login page tricks the user into typing their real password, handing it straight to the attacker.
- **Malware.** A keylogger on the user's computer records everything they type, password included.

In every one of these cases, the attacker ends up holding the *correct* password. From the system's point of view, their login looks completely legitimate — because it *is* the right password. No amount of password-strength rules or brute-force protection helps here, because nothing is being brute-forced.

**This is the exact gap MFA closes.** MFA (Multi-Factor Authentication) requires a *second, independent* proof of identity — something the attacker almost certainly does **not** have, even if they stole the password. In LiliShop, that second proof is a rotating 6-digit code from an app on the user's phone.

### Why only administrators?

LiliShop enforces MFA specifically for `Administrator` and `SuperAdmin` accounts. The reasoning is a straightforward risk trade-off:

| Account type | What's at risk if compromised | MFA required? |
|---|---|---|
| `Standard` (customer) | That one customer's orders and saved details | No (optional) |
| `Administrator` / `SuperAdmin` | Discounts, **all** customer orders, store control | **Yes, mandatory** |

An admin account is a master key to the whole shop. The small extra friction of entering a phone code at login is a very cheap price for protecting that level of access. A regular customer, by contrast, would find mandatory MFA more annoying than the contained risk justifies — so for them it's not forced.

> [!TIP]
> **The core idea in one sentence:** a password proves you *know* a secret; MFA additionally proves you *physically have* a specific device. Stealing a secret is easy and remote; stealing a physical phone is hard and local. Requiring both makes remote attacks dramatically harder.

---

## 2. Core Concepts

Before diving into code, let's define every term you'll need. Each is explained in plain language first.

### 2.1 Authentication vs. Authorization

These two words sound alike and are constantly confused, but they mean different things:

- **Authentication** = *"Who are you?"* — proving your identity. Logging in is authentication.
- **Authorization** = *"What are you allowed to do?"* — checking your permissions. Being blocked from an admin page because you're a regular customer is authorization.

MFA is part of **authentication** — it's an extra step in proving *who you are*. It happens during login, before any permission checks.

### 2.2 Factors: Something You Know, Have, Are

A "factor" is a *category* of evidence for proving identity. There are three classic categories:

| Factor | Meaning | Example |
|---|---|---|
| Something you **know** | A secret in your memory | Password, PIN |
| Something you **have** | A physical object you possess | Your phone, a hardware key |
| Something you **are** | A physical trait of your body | Fingerprint, face scan |

"**Multi**-factor" means using **two or more different categories** together. The key word is *different*. A password plus a security question is **not** true MFA — both are "something you know," so a single leak (a phishing page) could capture both. LiliShop combines *something you know* (password) with *something you have* (phone) — two genuinely different categories, so one leak doesn't compromise both.

- The **first factor** in LiliShop is the password.
- The **second factor** is the phone-generated code.

### 2.3 OTP and TOTP

- **OTP** = **One-Time Password**. A code that's valid only once (or only briefly), then becomes useless. Unlike your regular password, which stays the same for months, an OTP is disposable.
- **TOTP** = **Time-based One-Time Password**. A specific kind of OTP that automatically changes based on the *current time* — typically every 30 seconds. This is the 6-digit number you see counting down in an authenticator app.

Why time-based? Because it means the code is self-expiring. Even if an attacker somehow glimpsed a valid code, it would already be dead within 30 seconds — long before they could realistically use it.

### 2.4 The Shared Secret

This is the heart of how TOTP works, so it's worth getting crystal clear.

When you set up MFA, the server generates a random **secret** — a string like `JBSWY3DPEHPK3PXP` — and shares it with your phone exactly once (via a QR code). From that moment, **both the server and your phone hold the same secret.**

Here's the crucial part that surprises most people: **after that one-time sharing, your phone and the server never communicate again.** There is no live "give me a code" request over the internet. Instead, both sides independently run the same math — combining the shared secret with the current time — and both arrive at the *same* 6-digit number, separately.

> [!IMPORTANT]
> **Analogy: two twins with synchronized watches.** Imagine two twins who, just once, agree on a secret math formula and set their watches to the exact same time. From then on, without ever speaking again, each can look at their own watch, plug the time into the shared formula, and both produce the identical number. TOTP works exactly like this — the "formula input" is the shared secret, the "watch" is each side's clock, and the "number" is the 6-digit code. No communication needed after setup.

### 2.5 Authenticator Apps & QR Codes

An **authenticator app** is a free phone app that stores TOTP secrets and displays the current codes. Any of these work with LiliShop, because TOTP is an open standard not owned by any one company:

- Google Authenticator
- Microsoft Authenticator
- Authy
- 1Password (and most password managers)

A **QR code** is just a convenient way to transfer the secret to the phone. Rather than making the user carefully type a long secret like `JBSWY3DPEHPK3PXP` by hand, the server encodes it in a QR image; the user points their phone camera at it, and the app reads the secret instantly. (Manual typing is still offered as a fallback for anyone who can't scan.)

### 2.6 Recovery Codes

MFA introduces a new fear: *"What if I lose my phone? Am I locked out forever?"*

**Recovery codes** are the answer — a set of 10 single-use backup codes, generated during setup. Each one can substitute for a phone code exactly once. If your phone is lost, you use a recovery code to get in. They're the emergency spare key for your MFA.

---

## 3. System Architecture

Before the details, here's the big picture: which pieces exist, and how they relate. MFA in LiliShop spans the backend, the frontend, and the user's phone.

```mermaid
flowchart TB
    subgraph Phone["📱 User's Phone"]
        App["Authenticator App<br/>(stores the secret,<br/>shows the 6-digit code)"]
    end

    subgraph Frontend["🖥️ Angular Frontend"]
        Login["login.component<br/>(orchestrates the flow)"]
        Setup["mfa-setup.component<br/>(enrollment + QR)"]
        Verify["mfa-verify.component<br/>(enter the code)"]
        Interceptor["jwtInterceptor<br/>(protects the flow)"]
    end

    subgraph Backend["⚙️ ASP.NET Core Backend"]
        Controller["AccountController<br/>(/login, /mfa/setup, /mfa/enable)"]
        Service["ApplicationUserService<br/>(all MFA logic)"]
        Identity["ASP.NET Core Identity<br/>(UserManager / SignInManager)"]
    end

    DB[("🗄️ Database<br/>AspNetUsers +<br/>AspNetUserTokens")]

    Login --> Setup
    Login --> Verify
    Login -.through.-> Interceptor
    Interceptor --> Controller
    Controller --> Service
    Service --> Identity
    Identity --> DB
    App -.QR scanned once<br/>at setup.-> Setup
```

The division of responsibility is clean:

| Layer | Responsibility |
|---|---|
| **Phone (authenticator app)** | Holds the secret; generates the current 6-digit code |
| **`login.component`** | The "traffic controller" — decides which screen to show based on the backend's response |
| **`mfa-setup.component`** | Handles first-time enrollment: shows the QR, confirms the first code, displays recovery codes |
| **`mfa-verify.component`** | Handles entering the code during a normal login |
| **`jwtInterceptor`** | Attaches tokens to requests and — critically — keeps login errors from breaking the flow |
| **`AccountController`** | The HTTP entry points the frontend calls |
| **`ApplicationUserService`** | Where all the actual MFA decisions live |
| **ASP.NET Core Identity** | The built-in engine that stores secrets, verifies codes, manages recovery codes |
| **Database** | Persists the secret, the "MFA enabled" flag, and the hashed recovery codes |

> [!NOTE]
> A key thing to appreciate: LiliShop did **not** hand-write the cryptography. TOTP generation, verification, secret storage, and recovery codes are all provided by **ASP.NET Core Identity**, which you already use for passwords. The custom code is only the *policy* — "admins must use MFA" and "here's how the screens flow" — wrapped around Identity's proven primitives. This is exactly how you *should* build MFA: never roll your own crypto.

---

## 4. How TOTP Actually Works

This section makes the "two twins with synchronized watches" idea concrete, because understanding it removes all the mystery from the code later.

### The one-time setup

1. The server generates a random secret, e.g. `JBSWY3DPEHPK3PXP`.
2. It saves that secret in its database.
3. It shows the secret to the phone (via QR code) exactly once.
4. The phone saves the secret too.

Now **both sides have the same secret**, permanently. This never needs to happen again.

### Generating a code (both sides, independently)

Every 30 seconds, both the phone and the server can do this same calculation, *without talking to each other*:

```
current_time ÷ 30   →   a "time-slice number" (e.g. 58212345)

secret + time-slice number   →   [ cryptographic formula ]   →   truncate to 6 digits   →   483920
```

Both the phone and the server:
- use the **same secret** (`JBSWY3DPEHPK3PXP`),
- read the **same current time**, producing the **same time-slice number**,
- run the **same formula**,
- and therefore land on the **same 6-digit code** (`483920`).

The phone *shows* you that code. The server *recalculates* it when you log in, and checks whether what you typed matches what it computed. **No network call between them — ever — after setup.**

```mermaid
sequenceDiagram
    participant P as 📱 Phone
    participant S as ⚙️ Server

    Note over P,S: One-time setup (via QR)
    S->>P: Share secret JBSWY3DPEHPK3PXP
    Note over P,S: Both now hold the same secret. They NEVER talk again.

    Note over P: Every 30 seconds...
    P->>P: secret + current time → 483920
    Note over P: Displays 483920

    Note over S: When user logs in...
    S->>S: secret + current time → 483920
    Note over S: Compares to what the user typed
```

### The clock-drift safety margin

Phones and servers rarely have *perfectly* synchronized clocks — one might be a few seconds off. If the check were exact-to-the-second, a slightly-off phone would produce a code the server rejects, locking out a legitimate user.

To prevent this, the verification also accepts the code from the *previous* and *next* 30-second time-slice, not only the exact current one. So a phone that's up to ~30 seconds off still works. This tolerance is built into ASP.NET Core Identity — LiliShop gets it for free, no code required.

> [!TIP]
> This is why you'll never see LiliShop's server "ask" the phone for anything. When you read `VerifyTwoFactorTokenAsync` in the backend code later, remember: it's the server *recalculating the code itself* from the stored secret and its own clock — the "look at my own watch and do the math" step from the twins analogy.


---

## 5. Backend Implementation

Now we get into LiliShop's real code. All the MFA logic lives in one service, `ApplicationUserService`, which leans on ASP.NET Core Identity's `UserManager` and `SignInManager` for the heavy lifting.

### 5.1 The DTOs (the "shapes" of data)

A **DTO** (Data Transfer Object) is a simple class that defines the *shape* of data moving between the frontend and backend. It carries only what's needed — never the full internal database entity. Here are the MFA-related DTOs.

**`LoginDto`** — what the client sends to log in:

```csharp
public class LoginDto
{
    public required string Email { get; set; }
    public required string Password { get; set; }

    /// Optional TOTP authenticator code (or a recovery code). Required only for accounts that must use
    /// multi-factor authentication (administrators). Ignored for accounts without MFA.
    public string? TwoFactorCode { get; set; }
}
```

The important field is `TwoFactorCode`, and notice it's **optional** (`string?`). A regular customer never fills it in. An admin fills it in on their second login attempt (with the code from their phone). This one optional field is what lets a *single* login endpoint handle both MFA and non-MFA users — a design we'll explore fully in Section 6.

**`UserDto`** — what the backend sends back after a login attempt:

```csharp
public class UserDto
{
    public int? Id { get; set; }
    public string? Email { get; set; }
    public required string DisplayName { get; set; }
    public required string Token { get; set; }
    public required string Role { get; set; }
    public bool EmailConfirmed { get; set; }

    /// True when the account must enrol an authenticator before it can obtain a token (admin without MFA).
    /// When either two-factor flag is true, Token is empty — no session was issued.
    public bool RequiresTwoFactorSetup { get; set; }

    /// True when the account has MFA enabled and must supply a valid authenticator/recovery code to log in.
    public bool RequiresTwoFactorCode { get; set; }
}
```
These two boolean properties represent **two different stages** of the MFA flow:

- **`RequiresTwoFactorSetup`** is `true` when the user is required to enable MFA but has **not enrolled an authenticator app yet**. The frontend should redirect the user to the MFA setup page, where they can scan a QR code and complete enrollment. No JWT token is issued at this stage.

- **`RequiresTwoFactorCode`** is `true` when the user has **already enabled MFA** and must provide a valid authenticator code (or a recovery code) to complete sign-in. Again, no JWT token is issued until the code is successfully verified.

These two flags are **mutually exclusive**. Only one of them can be `true` during a login attempt, and if either flag is `true`, the `Token` property is empty because authentication has not yet been completed.
The two boolean flags — `RequiresTwoFactorSetup` and `RequiresTwoFactorCode` — are the signals that tell the frontend what to do next. We'll see exactly how in Section 6.

**`AuthenticatorSetupDto`** — the data needed to show a QR code during enrollment:

```csharp
public class AuthenticatorSetupDto
{
    public required string SharedKey { get; set; }        // for manual entry
    public required string AuthenticatorUri { get; set; } // otpauth:// URI, rendered as a QR code
}
```

**`EnableAuthenticatorDto`** — what the client sends to *finish* enrollment:

```csharp
public class EnableAuthenticatorDto
{
    public required string Email { get; set; }
    public required string Password { get; set; }
    public required string Code { get; set; }
}
```

**`EnableAuthenticatorResultDto`** — the recovery codes, returned once after enabling MFA:

```csharp
public class EnableAuthenticatorResultDto
{
    /// Result of enabling MFA: one-time recovery codes the user must store securely to regain access if the
    /// authenticator device is lost. Shown exactly once.
    public string[] RecoveryCodes { get; set; } = Array.Empty<string>();
}
```

| DTO | Direction | Purpose |
|---|---|---|
| `LoginDto` | Client → Server | Email, password, and optional 2FA code |
| `UserDto` | Server → Client | Login result + the two "pending" flags |
| `AuthenticatorSetupDto` | Server → Client | The secret + QR URI for enrollment |
| `EnableAuthenticatorDto` | Client → Server | Credentials + first code to confirm enrollment |
| `EnableAuthenticatorResultDto` | Server → Client | The one-time recovery codes |

### 5.2 Enrollment: Getting Set Up (Two Steps)

**Enrollment** is the one-time process of registering an authenticator app (such as Google Authenticator or Microsoft Authenticator) with a user's account. Once enrollment is complete, both the server and the phone hold the same secret and can independently generate matching TOTP codes, exactly as described in Section 4.

Enrollment faces a subtle problem:

> "To log in, you need MFA. To set up MFA, you need to be logged in."

This is a chicken-and-egg problem: admins must use MFA before they can receive a token, but they can't configure MFA without a token to prove who they are in the first place.

LiliShop breaks this loop by having the enrollment endpoints **re-verify the password directly**, instead of requiring a token. Enrollment happens across two endpoints.

#### Step 1 — `GetAuthenticatorSetupAsync` (get the QR)

```csharp
public virtual async Task<OperationResult<AuthenticatorSetupDto>> GetAuthenticatorSetupAsync(LoginDto loginDto)
{
    var user = await AuthenticateForMfaEnrolmentAsync(loginDto.Email, loginDto.Password);
    if (user is null)
    {
        return OperationResult.Failure<AuthenticatorSetupDto>(ErrorCode.InvalidPassword, "Invalid email or password.");
    }

    // Provision an authenticator key if the user does not already have one.
    var key = await _userManager.GetAuthenticatorKeyAsync(user);
    if (string.IsNullOrEmpty(key))
    {
        await _userManager.ResetAuthenticatorKeyAsync(user);
        key = await _userManager.GetAuthenticatorKeyAsync(user);
    }

    if (string.IsNullOrEmpty(key) || string.IsNullOrEmpty(user.Email))
    {
        return OperationResult.Failure<AuthenticatorSetupDto>(ErrorCode.GeneralException, "Unable to generate an authenticator key.");
    }

    return OperationResult.Success<AuthenticatorSetupDto>(new AuthenticatorSetupDto
    {
        SharedKey = key,
        AuthenticatorUri = BuildAuthenticatorUri(user.Email, key)
    });
}
```

Step by step:

1. **Re-verify the password** (via the helper, explained below). No token required.
2. **`GetAuthenticatorKeyAsync`** asks: does this user already have a secret? If not...
3. **`ResetAuthenticatorKeyAsync`** generates a brand-new random secret and saves it to the database, then fetches it back. **This is the moment the secret is born — at this point it exists only on the server, not yet on the phone.**
4. **`BuildAuthenticatorUri`** wraps the secret in the special `otpauth://` format the phone understands (see below).
5. Return the secret (`SharedKey`) and the URI (`AuthenticatorUri`).

**Why does the response include both `SharedKey` and `AuthenticatorUri`?** They look different, but they hold **exactly the same secret** — just in two formats:

| Field | Format | Purpose |
|---|---|---|
| `SharedKey` | Raw Base32 text | For manual typing, if the user can't scan a QR |
| `AuthenticatorUri` | An `otpauth://` link wrapping the same secret | For rendering as a QR code |

**Where is the secret stored?** In a table ASP.NET Core Identity manages automatically, `AspNetUserTokens` — deliberately *not* in the `AspNetUsers` table itself, since `AspNetUserTokens` is Identity's general-purpose place for storing different kinds of per-user tokens in a consistent way. A row looks like:

| UserId | LoginProvider | Name | Value |
|---|---|---|---|
| `42` | `[AspNetUserStore]` | `AuthenticatorKey` | `JBSWY3DPEHPK3PXP` |

That `Value` is the secret. It stays there **permanently** — it does not expire — until MFA is reset or disabled.

**Building the QR URI** — `BuildAuthenticatorUri`:

```csharp
private static string BuildAuthenticatorUri(string email, string unformattedKey)
{
    const string issuer = "LiliShop";
    return $"otpauth://totp/{Uri.EscapeDataString(issuer)}:{Uri.EscapeDataString(email)}"
         + $"?secret={unformattedKey}&issuer={Uri.EscapeDataString(issuer)}&digits=6";
}
```

This produces a string like:

```
otpauth://totp/LiliShop:admin@lilishop.com?secret=JBSWY3DPEHPK3PXP&issuer=LiliShop&digits=6
```

Every part has a job:

| Part | Meaning |
|---|---|
| `otpauth://totp/` | "This is a TOTP setup link" — every authenticator app recognizes this |
| `LiliShop:admin@lilishop.com` | The label the app displays, so the user knows which account it's for |
| `secret=JBSWY3DPEHPK3PXP` | **The actual secret** the phone stores |
| `issuer=LiliShop` | Who issued it (for display) |
| `digits=6` | The code should be 6 digits |

> [!WARNING]
> This URI contains the secret **in plain text**. It must never be written to application logs, error messages, or analytics — and it should only ever be sent to the one authenticated user completing enrollment, never displayed or transmitted anywhere else.

When the phone scans the QR made from this string, it extracts `secret=` and stores it. **This is the one and only moment the secret travels to the phone.** After that, neither side ever sends the secret again — the phone keeps its copy, the server keeps its copy, and (as covered in Section 4) both independently generate matching 6-digit codes from that shared secret and the current time.

Here's the secret's full journey, from birth to first use:

```mermaid
flowchart TD
    A["Admin submits email and password"] --> B["Backend re-verifies credentials"]
    B --> C["No existing secret found"]
    C --> D["A new random secret is generated"]
    D --> E["Secret saved in AspNetUserTokens"]
    E --> F["Secret wrapped in an otpauth link"]
    F --> G["SharedKey and AuthenticatorUri returned"]
    G --> H["Frontend renders the QR code"]
    H --> I["User scans the QR with their phone"]
    I --> J["Authenticator app stores its own copy of the secret"]
    J --> K["Both sides can now generate matching six-digit codes"]
```

#### Step 2 — `EnableAuthenticatorAsync` (confirm & turn on)

At this point the user already has the QR code from Step 1 and has scanned it — their authenticator app is now generating live 6-digit codes. This step asks them to submit one of those codes back to the server, proving the setup actually worked, before MFA is switched on for real.

```csharp
public virtual async Task<OperationResult<EnableAuthenticatorResultDto>> EnableAuthenticatorAsync(EnableAuthenticatorDto dto)
{
    var user = await AuthenticateForMfaEnrolmentAsync(dto.Email, dto.Password);
    if (user is null)
    {
        return OperationResult.Failure<EnableAuthenticatorResultDto>(ErrorCode.InvalidPassword, "Invalid email or password.");
    }

    var isValid = await _userManager.VerifyTwoFactorTokenAsync(
        user, _userManager.Options.Tokens.AuthenticatorTokenProvider, NormalizeCode(dto.Code));

    if (!isValid)
    {
        return OperationResult.Failure<EnableAuthenticatorResultDto>(ErrorCode.InvalidData, "Verification code is invalid. Please try again.");
    }

    var enableResult = await _userManager.SetTwoFactorEnabledAsync(user, true);
    if (!enableResult.Succeeded)
    {
        return OperationResult.Failure<EnableAuthenticatorResultDto>(ErrorCode.UpdateOperationFailed, "Failed to enable two-factor authentication.");
    }

    _logger.LogInformation("Two-factor authentication enabled. UserId={UserId}", user.Id);

    var recoveryCodes = await _userManager.GenerateNewTwoFactorRecoveryCodesAsync(user, 10);

    return OperationResult.Success<EnableAuthenticatorResultDto>(new EnableAuthenticatorResultDto
    {
        RecoveryCodes = recoveryCodes?.ToArray() ?? Array.Empty<string>()
    });
}
```

Step by step:

1. **Re-verify the password** *again* (yes, again — see the callout below for why).
2. **`VerifyTwoFactorTokenAsync`** checks the 6-digit code the user typed. This isn't the server blindly trusting whatever the user submits — it's the server independently recalculating what the correct code *should* be right now, using the secret it already has stored (the exact same TOTP math from Section 4), and checking whether that matches what was typed. If they match, it proves the phone really was set up with the right secret — a wrong or fabricated code simply won't compute to the same number.
3. **`SetTwoFactorEnabledAsync(user, true)`** flips the `TwoFactorEnabled` column on the user's row in `AspNetUsers` from `false` to `true`. This single value is what changes everything about future logins — from this moment on, the MFA gate in `EnforceAdminMfaAsync` (Section 5.3) will demand a valid code from this admin every time they log in.
4. **`GenerateNewTwoFactorRecoveryCodesAsync(user, 10)`** creates 10 one-time emergency backup codes — each one can stand in for a phone code exactly once, if the device is ever lost. They're returned here and shown to the user **only once**; once this screen is left, the same codes can never be retrieved again. (The full mechanics — why they're stored hashed, how the login fallback uses them — are covered in Section 5.4.)

> [!IMPORTANT]
> **Why re-check the password in *both* steps (and again at final login)?** Look at the helper's comment:
> ```csharp
> /// Re-authenticates a user by email + password for the MFA enrolment endpoints (which are reached
> /// before a token exists). Honours account lockout. Returns null on any failure.
> private async Task<ApplicationUser?> AuthenticateForMfaEnrolmentAsync(string email, string password)
> {
>     var user = await _userManager.FindByEmailAsync(email);
>     if (user is null) return null;
>     var passwordCheck = await _signInManager.CheckPasswordSignInAsync(user, password, lockoutOnFailure: true);
>     return passwordCheck.Succeeded ? user : null;
> }
> ```
> The key phrase is **"reached before a token exists."** Normally the server knows who you are from your login token — but enrollment happens *before* any token exists. So the only way the server can know who's calling is to re-verify the password on each request. This is the **secure, stateless** choice: the server never has to *remember* "this person passed step 1," which would mean storing exploitable pending state. Each step proves itself independently. Notice too that `lockoutOnFailure: true` means even these enrollment endpoints respect the account-lockout protection — an attacker can't hammer them with password guesses.

Here's the full enrollment flow, showing how the two endpoints fit together as one HTTP conversation:

```mermaid
sequenceDiagram
    participant U as 👤 Admin
    participant FE as 🖥️ mfa-setup.component
    participant BE as ⚙️ Backend
    participant DB as 🗄️ Database

    Note over U,DB: STEP 1, get the QR code
    FE->>BE: POST account/mfa/setup with email and password
    BE->>BE: Re-verify password
    BE->>DB: Create and save secret, if none exists
    BE-->>FE: sharedKey and authenticatorUri
    FE->>FE: Render QR from authenticatorUri
    U->>U: Scan QR, phone now shows a six-digit code

    Note over U,DB: STEP 2, confirm and enable
    U->>FE: Type the six-digit code
    FE->>BE: POST account/mfa/enable with email, password, and code
    BE->>BE: Re-verify password again
    BE->>DB: Verify code against the secret
    BE->>DB: Set TwoFactorEnabled to true
    BE->>DB: Generate ten recovery codes
    BE-->>FE: recoveryCodes
    FE->>U: Show recovery codes and a saved confirmation gate
```

> [!NOTE]
> **Enrollment ≠ login.** At the end of enrollment, the admin still has **no token** — enabling MFA and logging in are deliberately separate. After saving recovery codes, they're sent to the verify screen to log in with a *fresh* code. This ensures that even right after setup, they prove they can produce a live code.

### 5.3 The Login Gate: Enforcing MFA

Now the login side. This is where an admin's password-correct login gets checked for MFA *before* any token is issued. It lives inside `LoginAsync`, the same method that handles every login.

Before looking at the code, here's the shape of the decision LiliShop makes on every login attempt, once the password and lockout checks (covered in the brute-force protection document) have already passed:

```mermaid
flowchart TD
    A[Password correct] --> B{Does this role require MFA}
    B -->|No| C[Issue the JWT token]
    B -->|Yes| D{Verify the MFA code}
    D -->|Valid code| C
    D -->|Missing or invalid code| E[Stop here, no token issued]
```

Here's the relevant portion of the real code:

```csharp
public virtual async Task<OperationResult<UserDto>> LoginAsync(LoginDto loginDto)
{
    var user = await _userManager.FindByEmailAsync(loginDto.Email);
    if (user is null)
    {
        return OperationResult.Failure<UserDto>(ErrorCode.InvalidPassword, "Invalid email or password.");
    }

    var isPasswordCorrect = await _signInManager.CheckPasswordSignInAsync(user, loginDto.Password, lockoutOnFailure: true);

    if (isPasswordCorrect.IsLockedOut) { /* ... locked-out response ... */ }
    if (!isPasswordCorrect.Succeeded)  { /* ... invalid credentials ... */ }

    var role = await user.GetRoleAsync(_userManager);

    // F14.2 — administrators MUST authenticate with a second factor. This gate runs BEFORE any token is
    // issued: an admin without an enrolled authenticator receives no token (only a "setup required"
    // signal), and an enrolled admin must supply a valid TOTP / recovery code.
    if (IsMfaRequiredForRole(role))
    {
        var mfaOutcome = await EnforceAdminMfaAsync(user, role, loginDto.TwoFactorCode);
        if (mfaOutcome is not null)
        {
            return mfaOutcome;   // stop here — either "pending" or "invalid code"
        }
    }

    // ... only reached if MFA is satisfied (or not required): issue tokens ...
    var accessToken = await _tokenService.CreateAccessTokenAsync(user);
    // ... build and return the authenticated UserDto ...
}
```

> [!IMPORTANT]
> Notice the ordering above: **no JWT token is ever created until every required authentication step — password, lockout, and (for admins) MFA — has completed successfully.** There is no code path where a partially-authenticated admin can slip through with a token. This is the single most important security guarantee this section makes.

**Which roles need MFA** — a tiny helper:

```csharp
private static bool IsMfaRequiredForRole(string role)
    => role == Role.Administrator || role == Role.SuperAdmin;
```

**The gate itself** — `EnforceAdminMfaAsync`. This method doesn't behave like a typical method that always returns a clear success or failure — its return value carries a special meaning of its own:

- A **non-null** `OperationResult` means "stop immediately — login cannot continue."
- **`null`** is the *only* value that means "MFA has been fully satisfied — `LoginAsync`, go ahead and issue the token."

Read the summary comment on the method itself; it states this contract directly:

```csharp
/// <summary>
/// Enforces the administrator MFA policy during login.
///
/// This method acts as a security gate immediately before JWT token creation.
/// It does not complete the login or issue a token. Instead, it decides whether
/// login may continue.
///
/// Returns:
/// - a non-null <see cref="OperationResult{T}"/> when login must stop immediately
///   (MFA setup required, MFA code required, or an invalid authentication code);
/// - null only when all MFA requirements have been satisfied and LoginAsync may
///   safely continue issuing the JWT token.
/// </summary>
private async Task<OperationResult<UserDto>?> EnforceAdminMfaAsync(ApplicationUser user, string role, string? twoFactorCode)
{
    // MFA has not been enrolled for this administrator yet.
    // Do not issue a token. Instead, tell the frontend to start the
    // MFA enrollment flow (display the QR code setup screen).
    if (!await _userManager.GetTwoFactorEnabledAsync(user))
    {
        _logger.LogWarning(
            "Admin login blocked: two-factor authentication is not enrolled. UserId={UserId}",
            user.Id);

        return OperationResult.Success<UserDto>(
            BuildMfaPendingDto(user, role, setupRequired: true));
    }

    // MFA is already enabled, but no authenticator (or recovery) code was supplied.
    // The frontend should prompt the user for their six-digit authenticator code
    // before login can continue.
    if (string.IsNullOrWhiteSpace(twoFactorCode))
    {
        return OperationResult.Success<UserDto>(
            BuildMfaPendingDto(user, role, setupRequired: false));
    }

    // Remove spaces or dashes so formats like "123 456" or "123-456"
    // are accepted before verification.
    var normalizedCode = NormalizeCode(twoFactorCode);

    // Verify the TOTP code generated by the user's authenticator app.
    // ASP.NET Core Identity recreates the expected code from the stored
    // shared secret and the current time, then compares it with the
    // code supplied by the user.
    var isTotpValid = await _userManager.VerifyTwoFactorTokenAsync(
        user,
        _userManager.Options.Tokens.AuthenticatorTokenProvider,
        normalizedCode);

    if (!isTotpValid)
    {
        // If the authenticator code is invalid, allow a one-time recovery code
        // as a fallback. This prevents users who have lost their authenticator
        // device from being permanently locked out of their account.
        var redeemed = await _userManager.RedeemTwoFactorRecoveryCodeAsync(
            user,
            normalizedCode);

        if (!redeemed.Succeeded)
        {
            _logger.LogWarning(
                "Admin login failed: invalid two-factor / recovery code. UserId={UserId}",
                user.Id);

            return OperationResult.Failure<UserDto>(
                ErrorCode.InvalidPassword,
                "Invalid authentication code.");
        }

        _logger.LogWarning(
            "Admin login completed using a recovery code. UserId={UserId}",
            user.Id);
    }

    // MFA has been successfully satisfied.
    // Returning null tells LoginAsync that it may now continue
    // and issue the JWT token.
    return null;
}
```

Here's every path through this method, and exactly what each one returns:

| Situation | What's returned | Effect on `LoginAsync` |
|---|---|---|
| MFA not enrolled yet | `OperationResult` with `RequiresTwoFactorSetup = true` | Stops here — no token issued |
| Enrolled, but no code was sent | `OperationResult` with `RequiresTwoFactorCode = true` | Stops here — no token issued |
| A valid TOTP or recovery code was sent | `null` | `LoginAsync` continues and issues the token |
| An invalid code was sent | `OperationResult` failure | Stops here — no token issued |

**Understanding `VerifyTwoFactorTokenAsync`** — this is the line that checks the code:

```csharp
var isTotpValid = await _userManager.VerifyTwoFactorTokenAsync(
    user, _userManager.Options.Tokens.AuthenticatorTokenProvider, normalizedCode);
```

Internally, `VerifyTwoFactorTokenAsync` performs the same TOTP calculation described in Section 4:

1. It retrieves the user's secret key from `AspNetUserTokens`. This is the same secret that was shared with the user's authenticator app during MFA setup.

2. It reads the current server time. TOTP codes are not stored anywhere; they are calculated dynamically from the secret key and the current time.

3. It calculates the expected 6-digit code. Because a user's phone clock and the server clock may differ slightly (for example, the phone clock may be a few seconds ahead or behind), Identity also accepts codes from the immediately previous or next time period. This small tolerance prevents valid users from being rejected because of minor clock differences.

4. It compares the calculated code with the code entered by the user (`normalizedCode`).

5. It returns `true` if the codes match. The server never contacts the user's phone and the phone never sends the code to the server. Both sides independently calculate the same code using the shared secret and the current time. If the calculations produce the same result, the code is considered valid.

If this check fails, the code gives the user one last chance to authenticate by accepting a recovery code instead (using `RedeemTwoFactorRecoveryCodeAsync`; the full mechanics are covered in Section 5.4). A valid recovery code is treated as an alternative second factor, allowing login to continue. However, it is immediately consumed and permanently removed from the account, ensuring that each recovery code can be used only once.

**A small thoughtful touch — `NormalizeCode`:**

```csharp
private static string NormalizeCode(string code)
    => code.Replace(" ", string.Empty).Replace("-", string.Empty);
```

This means all of these inputs:

```
483920
483 920
483-920
```

normalize to the exact same value before being checked:

```
483920
```

Small, but it prevents needless "invalid code" frustration from a stray space or dash.

---

You can think of `EnforceAdminMfaAsync` as a **security gate** placed immediately before token issuance. Passing the password check gets an administrator to the gate; passing MFA is what opens it.

### 5.4 The Recovery Code Fallback

Look again at the fallback inside `EnforceAdminMfaAsync`:

```csharp
if (!isTotpValid)
{
    // Fall back to a one-time recovery code so a lost authenticator does not cause a permanent lockout.
    var redeemed = await _userManager.RedeemTwoFactorRecoveryCodeAsync(user, normalizedCode);
    if (!redeemed.Succeeded) { /* both failed → reject */ }
    _logger.LogWarning("Admin login completed using a recovery code. UserId={UserId}", user.Id);
}
```

This is elegant: the user types a code into the *same* box, whether it's a phone code or a recovery code. The server tries it as a TOTP first; if that fails, it tries it as a recovery code. Only if *both* fail is the login rejected.

```mermaid
flowchart TD
    Start["User types a code"] --> TOTP{"Valid TOTP code?"}
    TOTP -->|"Yes"| Success["✅ Logged in"]
    TOTP -->|"No"| Recovery{"Valid recovery code?"}
    Recovery -->|"Yes"| Consume["✅ Logged in<br/>(this code is now permanently used up)"]
    Recovery -->|"No"| Reject["❌ Invalid authentication code"]
```

Two important properties:

**Recovery codes are stored hashed.** When `GenerateNewTwoFactorRecoveryCodesAsync` saves them, it stores only a scrambled, one-way *hash* of each code — not the code itself. This is why they're **"shown exactly once"**: the server literally cannot display them again, because it doesn't keep the originals, only their fingerprints. (Same reason a well-built site can't email you your existing password.) This is a security feature, not a limitation.

**"Redeem" means consumed.** The word `Redeem` in the method name is deliberate — when a recovery code is used, it's permanently marked as used and can never work again. Ten codes means ten emergency logins, total, until regenerated.

> [!TIP]
> Notice the recovery-code login is logged at **Warning** level even though the login *succeeded*. Why warn on a success? Because using a recovery code is *unusual* — a legitimate admin normally uses their phone. Falling back to a recovery code means either they genuinely lost their device, or someone who shouldn't have the codes is using them. Either way, it's a security-relevant event worth an administrator's attention. That's a security-conscious detail.


---

## 6. The Login State Machine

This is the architectural heart of the whole design, so it deserves its own section. It explains *why* the code is shaped the way it is.

### The problem: one login, three outcomes

Before MFA, login had two outcomes: success (here's your token) or failure. With MFA, an admin's login can now end in **three** places:

1. **Straight success** — normal customer, or an admin who sent a valid code → here's your token.
2. **"Set up MFA first"** — an admin who's never enrolled → no token; go to the setup screen.
3. **"Enter your code"** — an enrolled admin who didn't send a code → no token; go to the verify screen.

The question: **how does the frontend know which of the three happened?**

### The solution: one endpoint, flags in the response

A naive design would use three separate endpoints. LiliShop instead uses **one login endpoint**, and the *response* carries flags telling the frontend what to do. Every response is a `UserDto`, but *which fields are set* tells the story:

| Outcome | `Token` | `RequiresTwoFactorSetup` | `RequiresTwoFactorCode` |
|---|---|---|---|
| **Success** | *(a real token)* | `false` | `false` |
| **Needs setup** | `""` (empty) | `true` | `false` |
| **Needs code** | `""` (empty) | `false` | `true` |

The backend builds the "pending" responses with this helper:

```csharp
private static UserDto BuildMfaPendingDto(ApplicationUser user, string role, bool setupRequired) => new()
{
    Id = user.Id,
    Email = user.Email,
    DisplayName = user.DisplayName,
    Role = role,
    Token = string.Empty,          // No session is issued until MFA is satisfied.
    EmailConfirmed = user.EmailConfirmed,
    RequiresTwoFactorSetup = setupRequired,
    RequiresTwoFactorCode = !setupRequired
};
```

Note the two flags are mirror images, and **`Token` is always empty** in a pending response.

### Why there's NO "pending login" state on the server

This is the most important design decision. From the frontend's `mfa-verify.component`:

```typescript
/**
 * Second factor for administrators. Re-posts the login with the TOTP (or recovery) code — the backend
 * holds no pending-login state, so the credentials plus the code are submitted together.
 */
```

A **stateful** design would have the server *remember* "this admin passed the password check and is waiting for a code." LiliShop deliberately does the opposite — the server remembers **nothing**. When the admin enters their code, the frontend re-sends *everything* — email, password, **and** the code — together in a fresh login call:

```typescript
this.accountService
  .login({ email: this.email(), password: this.password(), twoFactorCode: code })
  .subscribe(...)
```

**Why stateless is better here:**

| Concern | Stateful (remembering) | Stateless (LiliShop) |
|---|---|---|
| Expiry management | Must decide how long a half-login lives, and clean up abandoned ones | No half-logins exist; nothing to expire |
| Multiple servers | Pending state on server A doesn't exist on server B | Each attempt carries everything; works on any server |
| Attack surface | A half-authenticated session could be hijacked | Nothing half-authenticated exists to hijack |

The trade-off: the password briefly lives in the browser's memory during the flow. LiliShop handles this correctly — the credentials live *only* in component memory (Angular signals), never in `localStorage` or `sessionStorage`.

### The one rule that must never be broken

Every one of the three outcomes returns **HTTP 200 OK**. The pending responses are *successful* responses that simply mean "not done yet." This creates a trap: if the frontend assumed "200 = logged in," an admin in a pending state would appear authenticated *without a token* — a serious hole.

The rule that closes it, stated in the frontend `IUser` model:

```typescript
export interface IUser {
  // ...
  token: string;   // '' whenever an MFA step is pending — never treat '' as authenticated
  requiresTwoFactorSetup?: boolean;
  requiresTwoFactorCode?: boolean;
}
```

**An empty token is NEVER "logged in."** The verify component enforces this explicitly before declaring success:

```typescript
next: (user) => {
  this.submitting.set(false);
  if (this.accountService.isAuthenticatedUser(user)) {   // checks the token is real & non-empty
    this.authenticated.emit(user);
  } else {
    this.serverError.set('Verification did not complete. Please try again.');
  }
}
```

### The complete state machine

```mermaid
stateDiagram-v2
    [*] --> Credentials: User opens login page

    Credentials --> Deciding: POST /account/login<br/>{email, password}

    Deciding --> Authenticated: Token is non-empty<br/>(normal user, or admin who sent a valid code)
    Deciding --> SetupScreen: RequiresTwoFactorSetup = true<br/>(admin, never enrolled)
    Deciding --> VerifyScreen: RequiresTwoFactorCode = true<br/>(admin, enrolled, no code sent)

    SetupScreen --> VerifyScreen: Finished enrolment<br/>+ saved recovery codes
    VerifyScreen --> Deciding: POST /account/login<br/>{email, password, twoFactorCode}

    Authenticated --> [*]

    note right of Deciding
        All three outcomes are HTTP 200.
        The difference is the flags +
        whether Token is empty.
    end note
```

Trace a brand-new admin's first login: enter credentials → backend says "setup required" → land on setup screen → enroll & save recovery codes → move to verify screen → enter a fresh code → re-post login *with* the code → backend satisfied → token issued → authenticated.

---

## 7. Frontend Implementation

The Angular side has three components working together, all built with modern Angular **signals** (reactive state holders) and the `@if`/`@else` template syntax. Before looking at each one individually, here's how they relate as a whole:

```mermaid
flowchart TD
    A[LoginComponent] --> B[Credentials form, the default screen]
    A --> C[mfa-setup component]
    A --> D[mfa-verify component]
    C --> E[Handles enrollment, the one time setup]
    D --> F[Handles verification, every login after that]
```

`LoginComponent` never navigates to separate routes for these steps — instead it behaves like a small state machine of its own: changing its `stage` signal simply swaps which one of these three is currently rendered inside the same page.

### 7.1 The Login Component (the orchestrator)

`login.component` is the traffic controller. It holds a `stage()` signal that decides which of three screens to show, and the template branches on it:

```html
@if (stage() === 'credentials') {
    <!-- the email/password form -->
} @else if (stage() === 'verify') {
    <app-mfa-verify
      [email]="credentials().email"
      [password]="credentials().password"
      (authenticated)="onVerified($event)"
      (cancelled)="onCancelMfa()">
    </app-mfa-verify>
} @else {
    <app-mfa-setup
      [email]="credentials().email"
      [password]="credentials().password"
      (enrolled)="onEnrolled()"
      (cancelled)="onCancelMfa()">
    </app-mfa-setup>
}
```

So the login page is really *three screens sharing one component*, and `stage()` is the switch. When a login response comes back with `RequiresTwoFactorSetup = true`, the component sets `stage` to show `mfa-setup`; when `RequiresTwoFactorCode = true`, it shows `mfa-verify`.

Notice it passes `email` and `password` down to the child components. This connects directly to the stateless design from Section 6: **the first login attempt never creates a session at all** — the backend only confirms the password is correct and replies with a "next step required" signal, with an empty token. Since the backend keeps no memory of that first attempt, the *second* request (the one carrying the MFA code) has to resubmit the original credentials from scratch, not the code alone. That's exactly why these two child components need `email` and `password` as inputs — they're not decorative, they're required for the re-post to work at all.

### 7.2 The Setup Component

`mfa-setup.component` handles **enrollment** — the one-time process, introduced in Section 5.2, of registering an authenticator app for the first time. This is different from *verification* (Section 7.3, next), which happens on every ordinary login once enrollment is already done. The component has two internal stages of its own, tracked by a `stage` signal (`'scan'` → `'recovery'`).

On load, it fetches the setup data and renders the QR:

```typescript
ngOnInit(): void {
  this.loadSetup();
}

private loadSetup(): void {
  this.loading.set(true);
  this.accountService.getAuthenticatorSetup(this.email(), this.password()).subscribe({
    next: async (setup) => {
      this.sharedKey.set(setup.sharedKey);
      try {
        this.qrDataUrl.set(await QRCode.toDataURL(setup.authenticatorUri, { margin: 1, width: 220 }));
      } catch {
        this.qrDataUrl.set('');   // manual key entry still works if QR rendering fails
      }
      this.loading.set(false);
    },
    error: (err) => {
      this.loading.set(false);
      this.serverError.set(this.extractError(err, 'Could not start MFA setup. Please try again.'));
    },
  });
}
```

When the user submits the first code, it calls enable, then switches to the recovery-codes stage:

```typescript
enable(): void {
  const code = this.code().trim();
  if (!code || this.loading()) return;

  this.loading.set(true);
  this.accountService.enableAuthenticator(this.email(), this.password(), code).subscribe({
    next: (result) => {
      this.loading.set(false);
      this.recoveryCodes.set(result.recoveryCodes ?? []);
      this.stage.set('recovery');   // switch from 'scan' screen to 'recovery' screen
    },
    error: (err) => {
      this.loading.set(false);
      this.serverError.set(this.extractError(err, 'Verification code is invalid. Please try again.'));
    },
  });
}
```

**The recovery-code safety gate.** This is the most safety-critical UI in the whole flow, because the codes are shown *once*. Three deliberate features enforce careful handling:

```html
<!-- 1. A copy button for easy saving -->
<button mat-stroked-button (click)="copyRecoveryCodes()">
  <mat-icon>content_copy</mat-icon> Copy codes
</button>

<!-- 2. A mandatory confirmation checkbox -->
<mat-checkbox [ngModel]="savedConfirmed()" (ngModelChange)="savedConfirmed.set($event)">
  I have saved these recovery codes.
</mat-checkbox>

<!-- 3. "Continue" is disabled until the box is ticked -->
<button [disabled]="!savedConfirmed()" (click)="finish()">
  Continue to sign in
</button>
```

This is a "speed bump" by design — a user rushing through setup *cannot* skip past the one screen they'll never see again. The `finish()` method only proceeds if the box is ticked:

```typescript
finish(): void {
  if (this.savedConfirmed()) {
    this.enrolled.emit();
  }
}
```

After the user leaves this screen, the backend will never return these exact codes again — the value is only ever included in this one API response. If they're lost before being saved, the account would need a fresh set generated. That's technically possible through the same backend method described in Section 5.4, though there's currently no self-service "regenerate" screen in the UI for it.

### 7.3 The Verify Component

`mfa-verify.component` handles entering the code during a normal login. Its job is simple: take the code, re-post the *whole* login, and check the result.

```typescript
submit(): void {
  const code = this.code().trim();
  if (!code || this.submitting()) return;

  this.submitting.set(true);
  this.serverError.set(null);

  this.accountService
    .login({ email: this.email(), password: this.password(), twoFactorCode: code })
    .subscribe({
      next: (user) => {
        this.submitting.set(false);
        if (this.accountService.isAuthenticatedUser(user)) {
          this.authenticated.emit(user);
        } else {
          this.serverError.set('Verification did not complete. Please try again.');
        }
      },
      error: (err) => {
        this.submitting.set(false);
        this.serverError.set(this.extractError(err));
      },
    });
}
```

It's worth being precise about what this component does and does not do. **It never verifies the code itself.** All it does is collect what the user typed and send it to the backend — the actual TOTP calculation and comparison (Section 5.3) happens entirely server-side. The backend stays the single source of truth for every authentication decision; the frontend's only job here is to display whatever the backend decides.

This is exactly why the `isAuthenticatedUser(user)` check matters so much. Remember from Section 6: **every one of the three login outcomes returns HTTP 200** — even the "please set up MFA" and "please enter your code" responses. So a 200 status alone says nothing about whether the user is actually logged in. The only reliable signal is whether `token` is non-empty, and that's precisely what this check enforces before the component fires its `authenticated` event.

This is the "stateless re-post" in action: it sends `email`, `password`, and `twoFactorCode` all together. And the template even tells the user they can use a recovery code here: *"Enter the 6-digit code from your authenticator app. You can also enter a one-time recovery code."* — matching the backend's recovery-code fallback.

### 7.4 Rendering the QR Code

The backend sends the `otpauth://` URI as plain text — it doesn't send an image. The frontend turns that text into an actual QR picture using the `qrcode` npm package:

```typescript
import * as QRCode from 'qrcode';
// ...
this.qrDataUrl.set(await QRCode.toDataURL(setup.authenticatorUri, { margin: 1, width: 220 }));
```

`toDataURL` converts the URI string into a data-URL image that's dropped straight into an `<img>` tag. Two supporting configuration details make this work cleanly:

```jsonc
// angular.json — qrcode is a CommonJS module; this silences the build warning
"allowedCommonJsDependencies": ["qrcode"]
```

A quick note on why the next file exists at all: TypeScript normally expects type information for every package it imports, so the compiler knows what functions exist and what arguments they take. The official types for `qrcode` caused a conflict in this project, so this file is a small, hand-written substitute — just enough type information for the one function actually used:

```typescript
// src/app/shared/types/qrcode.d.ts — a minimal hand-written type declaration
// We deliberately avoid @types/qrcode because it pulls in @types/node, whose global
// declarations (e.g. AbortSignal) conflict with the DOM lib in this project.
declare module 'qrcode' {
  export function toDataURL(text: string, options?: QRCodeToDataURLOptions): Promise<string>;
}
```

> [!NOTE]
> That hand-written `.d.ts` file is a small but real engineering decision: the "official" `@types/qrcode` package would have dragged in Node.js type definitions that clash with the browser's DOM types, causing build errors. Writing a minimal declaration for just the one function actually used avoids the whole conflict.

---

## 8. The JWT Interceptor: Managing Tokens and Protecting the Login Flow

### 8.1 What Is an HTTP Interceptor?

An **HTTP interceptor** is code that sits between the Angular app and the network, automatically inspecting and modifying every request going out and every response coming back — like a mailroom that stamps all outgoing mail and sorts all incoming mail centrally, so individual components don't each have to handle it themselves.

`jwtInterceptor` does two jobs: attaching tokens to outgoing requests, and deciding what to do when a response comes back `401`. Neither job involves the interceptor "deciding" anything in a human sense — it applies the same fixed rule to every request that passes through it. That distinction matters for what follows.

### 8.2 Job 1 — Attaching the Token

```typescript
const isApiRequest = request.url.startsWith(environment.apiUrl);
const token = storageService.get<string>(LOCAL_STORAGE_KEYS.AUTH_TOKEN);

if (token && isApiRequest) {
  request = request.clone({
    setHeaders: { Authorization: `Bearer ${token}` }
  });
}
```

If we have a token *and* the request goes to our own API, stamp the token on it. The `isApiRequest` check is a quiet security detail: it stops your login token from leaking to any *third-party* host the app might call.

### 8.3 Job 2 — Refreshing Expired Tokens

Access tokens are deliberately short-lived (your notes mention ~15 minutes). This is a security trade-off: if one is ever stolen, the window in which it's useful to an attacker is small. But short-lived tokens create an obvious usability problem on their own — nobody wants to re-enter their password every 15 minutes. A **refresh token** (a longer-lived secret stored in a secure cookie) solves that trade-off: it lets the app quietly obtain a fresh access token in the background, so the short expiry buys security without costing the user constant re-logins.

```typescript
return next(request).pipe(
  catchError((error: any) => {
    if (error.status === 401 && isApiRequest && !isAuthEndpoint) {
      return accountService.refreshToken().pipe(
        switchMap((newToken) => {
          storageService.set(LOCAL_STORAGE_KEYS.AUTH_TOKEN, newToken);
          request = request.clone({ setHeaders: { Authorization: `Bearer ${newToken}` } });
          return next(request);   // retry the original request with the fresh token
        }),
        catchError(refreshError => {
          accountService.logout();   // refresh also failed → give up, log out
          return throwError(() => refreshError);
        })
      );
    }
    return throwError(() => error);
  })
);
```

Concretely, here's what "retry the original request" means: say a `GET /api/orders` call goes out with an expired token and comes back `401`. The interceptor holds onto that original request, calls the refresh endpoint, gets a new access token back, and resends that *exact same* `GET /api/orders` request — this time with the new token in the header. The code that originally asked for the orders never sees the intermediate `401` at all; it just receives the eventual successful response, slightly later than usual.

> [!NOTE]
> **Guarding against a "refresh stampede."** If several requests all hit a `401` around the same moment, you wouldn't want each one triggering its own separate refresh call. An `isRefreshing()` flag ensures only the *first* 401 starts a refresh; any others that arrive while it's in progress wait for that same refresh to finish and then retry using its result, instead of firing off duplicate refresh requests of their own.

### 8.4 The Critical Guard: `isAuthEndpoint`

Look at the condition: `if (error.status === 401 && isApiRequest && !isAuthEndpoint)`. That `!isAuthEndpoint` is the guard, checked against this list:

```typescript
const isAuthEndpoint = [
  'account/login',
  'account/mfa/',
  'account/google-login',
  'account/register',
  'account/forgot-password',
  'account/reset-password',
  'account/refresh-token',
].some((path) => request.url.includes(path));
```

Its comment explains exactly why:

```typescript
// Pre-authentication endpoints. A 401 from these means "bad credentials / wrong
// MFA code / admin blocked" — NOT "session expired", so we must not attempt a
// silent token refresh + retry (that would swallow the error and force a logout
// redirect, breaking the MFA and Google flows). Let their 401 reach the caller.
```

One entry on that list is worth calling out specifically: `account/refresh-token` is in there too. This isn't an oversight — if a refresh attempt itself came back `401` and the interceptor tried to "refresh the refresh," it would be trying to fix a broken session by calling the very endpoint that just failed, with nothing to recover with. Excluding it stops that recursive dead end before it can happen; a failed refresh just means the session is over, and the user needs to log in again.

### 8.5 Why This Guard Protects the MFA Flow

Before walking through what goes wrong without this guard, it's worth being precise about what a `401` means here. Normally, `401` signals "your access token is missing or has expired." But ASP.NET Core also reuses the same status code for a completely different situation: a login or MFA attempt that was simply *wrong* — a bad password, a bad code. At this pre-authentication stage, there is no session and no access token yet to have expired at all — the user hasn't logged in yet. The interceptor has to tell these two situations apart, and the guard is exactly how it does that.

Now picture what would happen **without** the guard, when an admin types a wrong MFA code:

1. Admin types a wrong code into `mfa-verify.component`.
2. The re-posted login comes back `401`.
3. Without the guard, the interceptor's normal rule fires anyway: *"401 from a non-auth endpoint means the token probably expired — refresh and retry."* It has no way to know this particular `401` actually came from a wrong code, not an expired session.
4. It refreshes and retries the login... which fails again (still the same wrong code)...
5. The real error ("Invalid authentication code") is **swallowed** by the refresh attempt.
6. When the refresh eventually fails too, the interceptor calls `logout()` and redirects to login.

The result: **one wrong digit throws the admin out of the login flow entirely.** The same thing would happen with a wrong password, or with the Google admin-block from Section 9. The guard prevents all of this by letting these specific 401s pass straight through to the components that display them inline, instead of running the refresh logic on them at all.

```mermaid
flowchart TD
    Wrong["Admin submits a wrong MFA code, wrong password, or blocked Google login"] --> Resp["Backend responds 401"]
    Resp --> Check{"Is this an auth endpoint"}

    Check -->|"WITHOUT the guard"| Bad1["Interceptor assumes the token expired"]
    Bad1 --> Bad2["Tries to refresh and retry"]
    Bad2 --> Bad3["Swallows the real error"]
    Bad3 --> Bad4["Refresh fails, logout and redirect"]
    Bad4 --> BadEnd["User kicked out over a typo"]

    Check -->|"WITH the guard"| Good1["Interceptor leaves the 401 alone"]
    Good1 --> Good2["Error reaches the component"]
    Good2 --> Good3["Shows an inline invalid code message"]
    Good3 --> GoodEnd["User just retries"]
```

### 8.6 Two Meanings of HTTP 401

> [!IMPORTANT]
> **A `401` status code alone is not enough to know what happened.** The same status is used for two completely different situations, and the interceptor has to tell them apart using something other than the status code itself:
>
> | A 401 from... | Actually means | Correct response |
> |---|---|---|
> | A **protected** endpoint (`/orders`, `/basket`) | "Your session's access token expired" | Refresh silently and retry |
> | An **auth** endpoint (`/login`, `/mfa/*`, `/google-login`) | "Your credentials or code were wrong — you were never logged in this session to begin with" | Show the error inline; never refresh or log out |
>
> The *URL* is the only clue available. The `isAuthEndpoint` list is the interceptor asking, on every single 401, "which of these two situations is this?" — and only applying the refresh-and-retry logic to the first one.

---

## 9. Closing the Google Bypass

LiliShop also supports "Sign in with Google." This is convenient and safe **for regular customers** — but it's inherently **single-factor**: it proves only *"this person controls this Google account,"* with no concept of TOTP or codes.

If admins could use it, it would be a silent bypass around all the MFA work:

1. You carefully force admins through password + MFA on the normal login.
2. An attacker phishes the admin's *Google* account (a separate attack).
3. The attacker clicks "Sign in with Google"... and walks in as an admin, **with no second factor ever requested.**

It's like fitting a vault door on the front of a building but leaving a side door with an ordinary lock. So the fix is absolute: **admins are blocked from Google sign-in entirely** and redirected to the password + MFA flow.

The backend returns a `401` with a clear message (e.g. *"Administrator accounts must sign in with email, password, and an authenticator code."*). And this connects directly to the interceptor: notice `account/google-login` is in the `isAuthEndpoint` list. That's deliberate — its `401` is *not* treated as "session expired," so the message passes straight through to the form's error display instead of triggering a refresh-and-logout.

```mermaid
flowchart TD
    Start["User clicks 'Sign in with Google'"] --> Verify["Backend verifies the Google token"]
    Verify --> Role{"Is this an admin account?"}
    Role -->|"No — regular customer"| OK["✅ Log in normally (token issued)"]
    Role -->|"Yes — Administrator/SuperAdmin"| Block["❌ 401 — blocked"]
    Block --> Passthrough["isAuthEndpoint guard lets the 401 reach the component"]
    Passthrough --> Message["Clean message shown; admin directed to password + MFA login"]
```

> [!IMPORTANT]
> **The principle: a security control is only as strong as the weakest way around it.** Protecting the *main* login path isn't enough if an *alternate* path skips the protection. Adding MFA to the password login while leaving Google login open for admins wouldn't be "mostly secure" — it would be effectively unprotected, because an attacker just picks the open door. Closing the Google door for admins is what makes the MFA guarantee actually hold.


---

## 10. The Complete End-to-End Flow

Here's everything tied together — the full journey of a brand-new admin, from their very first login through to accessing a protected resource. This diagram combines every piece covered above.

```mermaid
sequenceDiagram
    participant U as 👤 Admin
    participant FE as 🖥️ Frontend
    participant BE as ⚙️ Backend
    participant DB as 🗄️ Database
    participant Ph as 📱 Phone
    Note over U,Ph: FIRST LOGIN (not yet enrolled)
    U->>FE: Enter email and password
    FE->>BE: POST /account/login (email, password)
    BE->>BE: Password OK, role is admin, MFA not enrolled
    BE-->>FE: 200, empty token, requiresTwoFactorSetup true
    FE->>FE: Token empty, so show SETUP screen
    Note over U,Ph: ENROLLMENT
    FE->>BE: POST /account/mfa/setup (email, password)
    BE->>DB: Create and save secret
    BE-->>FE: sharedKey and authenticatorUri
    FE->>FE: Render QR
    U->>Ph: Scan QR (phone stores the secret)
    Ph-->>U: Shows a 6-digit code
    U->>FE: Enter that code
    FE->>BE: POST /account/mfa/enable (email, password, code)
    BE->>DB: Verify code, enable MFA, make 10 recovery codes
    BE-->>FE: recoveryCodes
    FE->>U: Show recovery codes plus a saved-confirmation gate
    U->>FE: Tick box and continue
    Note over U,Ph: VERIFY (finish the login)
    FE->>FE: Show VERIFY screen
    Ph-->>U: Shows current 6-digit code
    U->>FE: Enter code
    FE->>BE: POST /account/login (email, password, twoFactorCode)
    BE->>DB: Verify code against secret
    BE->>BE: MFA satisfied, issue tokens
    BE-->>FE: 200, real token, role Administrator
    FE->>FE: Token present, so AUTHENTICATED
    Note over U,Ph: USING THE APP
    U->>FE: Open an admin page
    FE->>BE: GET /orders with Bearer token
    BE-->>FE: 200, orders returned
```

And every **subsequent** login (already enrolled) is much shorter: enter email + password → backend says `requiresTwoFactorCode:true` → verify screen → enter phone code → re-post login with the code → authenticated. Three screens collapse to two.

---

## 11. API Reference

### Endpoints

| Method | Endpoint | Purpose | Auth required? |
|---|---|---|---|
| `POST` | `/account/login` | Log in; also submits the TOTP code (MFA step) | No (pre-auth) |
| `POST` | `/account/mfa/setup` | Get the secret + QR URI to begin enrollment | No — re-verifies password |
| `POST` | `/account/mfa/enable` | Confirm the first code, enable MFA, get recovery codes | No — re-verifies password |
| `POST` | `/account/google-login` | Google sign-in (blocked for admins) | No (pre-auth) |

### Sample requests & responses

**Login — admin, first attempt (no code yet):**

```jsonc
// Request
POST /account/login
{ "email": "admin@lilishop.com", "password": "CorrectPassword1!" }

// Response — 200 OK (but NOT authenticated: token is empty)
{
  "id": 42,
  "email": "admin@lilishop.com",
  "displayName": "Site Admin",
  "token": "",
  "role": "Administrator",
  "emailConfirmed": true,
  "requiresTwoFactorSetup": false,
  "requiresTwoFactorCode": true
}
```

**Login — admin, second attempt (with code):**

```jsonc
// Request
POST /account/login
{ "email": "admin@lilishop.com", "password": "CorrectPassword1!", "twoFactorCode": "483920" }

// Response — 200 OK, now authenticated (token present)
{
  "id": 42,
  "email": "admin@lilishop.com",
  "displayName": "Site Admin",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6...",
  "role": "Administrator",
  "emailConfirmed": true,
  "requiresTwoFactorSetup": false,
  "requiresTwoFactorCode": false
}
```

**Setup — begin enrollment:**

```jsonc
// Request
POST /account/mfa/setup
{ "email": "admin@lilishop.com", "password": "CorrectPassword1!" }

// Response — 200 OK
{
  "sharedKey": "JBSWY3DPEHPK3PXP",
  "authenticatorUri": "otpauth://totp/LiliShop:admin@lilishop.com?secret=JBSWY3DPEHPK3PXP&issuer=LiliShop&digits=6"
}
```

**Enable — confirm & get recovery codes:**

```jsonc
// Request
POST /account/mfa/enable
{ "email": "admin@lilishop.com", "password": "CorrectPassword1!", "code": "483920" }

// Response — 200 OK (recovery codes shown ONCE)
{
  "recoveryCodes": [
    "4f8a2-91bc7", "d0e11-77a3f", "b62c9-40de8", "1a5f3-c8b90", "9e7d4-2f16a",
    "c3b80-6ea51", "72f9d-13c40", "a41e6-9db27", "5c0af-3e819", "e8d72-04b6c"
  ]
}
```

### Common error responses

| Status | Meaning | Example message |
|---|---|---|
| `401` | Wrong password / wrong code / admin Google-blocked | `"Invalid email or password."` / `"Invalid authentication code."` |
| `400` | Bad code during enrollment | `"Verification code is invalid. Please try again."` |
| `429` | Rate limited (too many attempts) | *(empty body — detected by status)* |

> [!TIP]
> Remember from the state-machine section: a `401` from these auth endpoints is an *expected credential failure* shown inline — it does **not** trigger the token-refresh/logout path, thanks to the interceptor's `isAuthEndpoint` guard.

---

## 12. 📖 Glossary

| Term | Meaning |
|---|---|
| **Authentication** | Proving *who you are* (logging in) |
| **Authorization** | Checking *what you're allowed to do* (permissions) |
| **Factor** | A category of identity evidence: something you know / have / are |
| **MFA** | Multi-Factor Authentication — requiring two or more *different* factors |
| **OTP** | One-Time Password — a code valid only once/briefly |
| **TOTP** | Time-based OTP — a code derived from a shared secret + the current time, changing every 30s |
| **Shared secret** | The random value both the server and phone hold; the basis for generating codes |
| **Authenticator app** | A phone app (Google Authenticator, Authy…) that stores secrets and shows current codes |
| **QR code** | An image that transfers the secret to the phone by scanning |
| **Recovery codes** | 10 one-time backup codes for logging in if the phone is lost |
| **DTO** | Data Transfer Object — a simple class defining the shape of data sent between client and server |
| **JWT** | JSON Web Token — the signed token proving a user is authenticated after login |
| **Access token** | The short-lived (~15 min) JWT sent with each request |
| **Refresh token** | A longer-lived secret (in a cookie) used to silently get a new access token |
| **Interceptor** | Angular code that inspects/modifies every HTTP request and response centrally |
| **Stateless** | A design where the server keeps no memory between requests; each request carries everything it needs |
| **`AspNetUserTokens`** | The ASP.NET Core Identity database table where the TOTP secret is stored |

---

<div align="center">

*Part 2 of the LiliShop security series. This document uses LiliShop's real backend and frontend code as its running example throughout.*

</div>
