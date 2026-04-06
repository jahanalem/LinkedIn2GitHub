### 1. Refresh Token and Login Process
**Question:** Can you explain how your login system keeps users secure while providing a smooth experience?
**Answer:** When a user logs in, my backend creates two tokens: a short-lived Access Token and a long-lived Refresh Token. The Access Token is returned normally, but the Refresh Token is securely stored inside an `HttpOnly` cookie. This prevents hackers from stealing it using JavaScript. When the Access Token expires, my Angular interceptor catches the 401 error, pauses the request, and silently sends the `HttpOnly` cookie to the backend. The backend rotates the tokens by issuing a brand new Refresh Token and Access Token, and the original request continues without the user ever noticing. 

### 2. Registration and Email Confirmation
**Question:** How do you handle user registration to ensure email addresses are valid and secure?
**Answer:** When a user registers, their account is created, but their `EmailConfirmed` status is false. My system generates a secure token and records the exact issue date. I wrap the email sending process in a `try-catch` loop with exponential backoff. This means if the SendGrid email service is temporarily down, the system waits and retries automatically. The email contains a link that points directly back to my ASP.NET Core API. When the user clicks it, the backend verifies that the token is valid and less than 24 hours old, confirms the email in the database, and safely redirects the user back to the Angular frontend.

### 3. Role-Based and Policy-Based Authorization (RBAC)
**Question:** What is the difference between Roles and Policies in your authorization system, and why did you separate them?
**Answer:** A Role is just a static label attached to a user, like "Standard" or "SuperAdmin." A Policy is a flexible rule that an endpoint enforces. I separated them to create a clean hierarchy. For example, my `RequireAtLeastAdministratorRole` policy accepts both the "Administrator" and "SuperAdmin" roles. This means I can protect my controllers with a single policy name instead of writing complex logic everywhere. It makes the code incredibly clean and easy to scale.

### 4. Secure Password Recovery
**Question:** How did you implement the "Forgot Password" feature to prevent security vulnerabilities like email enumeration?
**Answer:** Security is critical here. When someone submits an email to the "Forgot Password" form, my backend checks if the user exists. However, it always returns a generic success message to the frontend, regardless of whether the email is real or not. This completely stops hackers from guessing valid email addresses. For real users, the system generates a secure, time-sensitive token. Once the user submits their new password, that token is used exactly once and immediately destroyed to prevent replay attacks.

### 5. Google Single Sign-On (SSO)
**Question:** How do you handle Google Single Sign-On, and why don't you use the Google token for API access?
**Answer:** When a user clicks "Sign in with Google," the frontend receives a Google ID token and sends it to the backend. I use the `JwtSecurityTokenHandler` to extract their email and name. If they are a new user, I silently create an account for them and assign them a "Standard" role. Then, I discard the Google token and issue my own local Access Token and Refresh Token. I do this because Google doesn't know about my internal roles or my background refresh logic. Issuing my own tokens gives me complete control over their session security.

### 6. Global Logout
**Question:** How does your "Log out from all devices" feature work behind the scenes?
**Answer:** Whenever I create a Refresh Token, I generate a unique Device ID and attach it to the token's name in the database. This allows one user to have multiple active sessions. When a user triggers a global logout, my backend updates their ASP.NET Core `SecurityStamp` to act as a kill-switch. Then, it searches the database and deletes every single Refresh Token belonging to that user. When their other devices try to silently refresh their expired Access Tokens, the backend rejects the request, and the Angular app automatically kicks them back to the login screen.

***
***
***

# 🔐 1. Refresh Token & Login Process

### ❓ Q1: How does your login system work?

**Answer:**
In my system, when the user logs in, the backend creates two tokens:
an access token and a refresh token.

The access token is short-lived and used for API requests.
The refresh token is long-lived and stored in an HttpOnly cookie.

This way, the user stays logged in without entering credentials again.

---

### ❓ Q2: What is the difference between access token and refresh token?

**Answer:**
The access token is used to access APIs and expires quickly.
The refresh token is used to get a new access token when it expires.

The refresh token is more sensitive, so I store it securely in an HttpOnly cookie.

---

### ❓ Q3: What happens when the access token expires?

**Answer:**
When the access token expires, the backend returns a 401 error.

The Angular interceptor catches this error and automatically sends a request to refresh the token.

If the refresh token is valid, the backend sends a new access token and the request is retried.

---

### ❓ Q4: What is token rotation?

**Answer:**
Token rotation means that every time we use a refresh token,
the backend creates a new refresh token and invalidates the old one.

This improves security and reduces the risk of token theft.

---

### ❓ Q5: Why do you use HttpOnly cookies?

**Answer:**
HttpOnly cookies cannot be accessed by JavaScript.

This protects the refresh token from XSS attacks and makes the system more secure.

---

# 📧 2. Registration & Email Confirmation

### ❓ Q6: Why do you confirm user emails?

**Answer:**
We confirm emails to make sure the email is real and belongs to the user.

It also helps with password recovery and prevents fake accounts.

---

### ❓ Q7: How does the email confirmation process work?

**Answer:**
After registration, the backend generates a secure token and a confirmation link.

This link is sent to the user by email.

When the user clicks the link, the backend validates the token and confirms the email.

---

### ❓ Q8: Why does your confirmation link point to the backend?

**Answer:**
Because the backend is responsible for validating the token.

After validation, the backend redirects the user to the frontend with a success or failure message.

---

### ❓ Q9: How do you handle email sending failures?

**Answer:**
I use a retry mechanism with exponential backoff.

If sending fails, the system retries after 2, 4, and 8 seconds before stopping.

---

### ❓ Q10: How do you handle token expiration?

**Answer:**
I store the token creation time and allow only 24 hours.

If the token is older than 24 hours, it is rejected.

---

# 🔑 3. RBAC (Authorization)

### ❓ Q11: What is the difference between authentication and authorization?

**Answer:**
Authentication means verifying who the user is.

Authorization means checking what the user is allowed to do.

---

### ❓ Q12: What is RBAC?

**Answer:**
RBAC stands for Role-Based Access Control.

Each user has a role like Standard or Administrator,
and access to endpoints depends on these roles.

---

### ❓ Q13: What is the difference between role and policy?

**Answer:**
A role is a label assigned to a user.

A policy is a rule that defines which roles can access an endpoint.

---

### ❓ Q14: Why do you use policy-based authorization?

**Answer:**
Policies make the system more flexible and scalable.

We can define rules once and reuse them across many endpoints.

---

### ❓ Q15: What happens if a user does not have permission?

**Answer:**
If the user is not logged in, they get 401 Unauthorized.

If they are logged in but lack permission, they get 403 Forbidden.

---

# 🔁 4. Password Recovery

### ❓ Q16: How does your forgot password flow work?

**Answer:**
The user enters their email.

The backend generates a secure reset token and sends a reset link by email.

The user clicks the link and sets a new password.

---

### ❓ Q17: How do you prevent email enumeration attacks?

**Answer:**
The backend always returns the same response,
even if the email does not exist.

This way, attackers cannot know which emails are valid.

---

### ❓ Q18: What makes your reset token secure?

**Answer:**
The token is cryptographically generated and tied to the user.

It is also time-limited and can only be used once.

---

### ❓ Q19: What happens if the token is invalid or expired?

**Answer:**
The backend rejects the request and returns an error.

The user must request a new reset link.

---

# 🔓 5. Google Login (SSO)

### ❓ Q20: How does Google login work in your system?

**Answer:**
The frontend gets an ID token from Google.

This token is sent to the backend.

The backend extracts the user information and logs the user in.

---

### ❓ Q21: Why do you not use the Google token directly?

**Answer:**
Because Google does not know our roles and permissions.

We create our own access token and refresh token
to control security and authorization.

---

### ❓ Q22: What happens if the user is new?

**Answer:**
If the user does not exist, we create a new account automatically
and assign a default role.

---

# 🚪 6. Global Logout

### ❓ Q23: What is global logout?

**Answer:**
Global logout logs the user out from all devices at the same time.

---

### ❓ Q24: How do you implement global logout?

**Answer:**
We update the user's security stamp
and delete all refresh tokens from the database.

We also clear cookies on the current device.

---

### ❓ Q25: What happens on other devices?

**Answer:**
Other devices still have an access token for a short time.

When it expires, they try to refresh it.

But since the refresh token is deleted, the request fails
and the user is logged out.

---

### ❓ Q26: Why is global logout important?

**Answer:**
It protects users in case of stolen devices or hacked accounts.

It allows users to immediately terminate all active sessions.

***
***
***

## 1. Refresh Token & Login Process

### Q1: Why do you use both an access token and a refresh token? Why not just one long‑lived token?
**Answer:**  
The access token is short‑lived (e.g., 15 minutes). If it gets stolen, the attacker can use it only for a short time.  
The refresh token is long‑lived but is stored in an `HttpOnly` cookie – JavaScript cannot read it, so it is safe from XSS attacks.  
Together, they give security (short access token) and a smooth user experience (silent refresh without asking the user to log in again).

### Q2: What happens when the access token expires and the user makes an API request?
**Answer:**  
The backend returns `401 Unauthorized`.  
The Angular **error interceptor** catches this error, pauses the request, and automatically calls the `/refresh-token` endpoint. The browser sends the `HttpOnly` cookie with the refresh token.  
If the refresh token is valid, the backend issues a new access token and a **new** refresh token (token rotation). Then the interceptor retries the original request – the user sees no error.

### Q3: What is token rotation and why is it important?
**Answer:**  
Token rotation means every time we refresh, we create a **brand new refresh token** and invalidate the old one.  
If a refresh token is stolen, it can only be used once. The attacker cannot use it again because the server already replaced it. This greatly reduces the damage of a stolen refresh token.

---

## 2. Registration & Email Confirmation

### Q1: How does your system prevent fake or inactive email addresses?
**Answer:**  
After registration, we send a confirmation email with a unique, cryptographically secure token. The user must click the link inside that email.  
Only after clicking the link does the account become fully active. This ensures the email address really belongs to the user.

### Q2: What happens if the user waits more than 24 hours before clicking the confirmation link?
**Answer:**  
Our code checks the **token issue date**. If the difference between `DateTime.UtcNow` and the issue date is more than 24 hours, we reject the confirmation.  
The user sees a message that the token has expired and must request a new confirmation email.

### Q3: How do you handle email delivery failures (e.g., the email server is down)?
**Answer:**  
We use a **retry mechanism with exponential backoff**.  
We try to send the email up to 3 times. The first retry after 2 seconds, then 4 seconds, then 8 seconds. If all attempts fail, we log the error and the user is informed to try again later. This makes the system fault‑tolerant.

---

## 3. Role‑Based & Policy‑Based Authorization (RBAC)

### Q1: What is the difference between a Role and a Policy in your system?
**Answer:**  
A **Role** is a simple label attached to a user, like `Standard`, `Administrator`, or `SuperAdmin`.  
A **Policy** is a rule that an API endpoint enforces. For example, the policy `RequireAtLeastAdministratorRole` allows both `Administrator` and `SuperAdmin` to access the endpoint.  
Policies are more flexible because we can define a **hierarchy** once, and then use the policy name everywhere – we don’t have to write complex `if` checks on each controller.

### Q2: If a user with the `Standard` role tries to access an endpoint protected by `RequireAtLeastAdministratorRole`, what HTTP status code do they get and why?
**Answer:**  
They get `403 Forbidden`.  
They are authenticated (they have a valid token), so it’s not `401 Unauthorized`. But their role does not meet the policy requirement, so the server correctly denies access.

### Q3: How do you keep role and policy names consistent across the whole project?
**Answer:**  
We define all role names and policy names as **constants** in a shared `Domain` project (e.g., `Role.SuperAdmin`, `PolicyType.RequireAtLeastAdministratorRole`).  
We use these constants in controllers and in the `Startup` configuration. This prevents typos and makes renaming easy – change in one place, and the whole project updates.

---

## 4. Secure Password Recovery (Forgot / Reset Password)

### Q1: How does your system prevent an attacker from finding out which email addresses are registered in your database?
**Answer:**  
We use a technique called **“no email enumeration”**.  
When a user submits the “Forgot Password” form, we always return the same generic success message – even if the email does not exist in our database.  
The attacker cannot distinguish between a real email and a fake one. This hides our user list.

### Q2: What happens if a user tries to use a password reset token that is very old (e.g., from a week ago)?
**Answer:**  
The token is rejected.  
Our `ResetPasswordAsync` method uses ASP.NET Core Identity’s built‑in validation, which checks the token expiration.  
Even if the token string is correct, the system will return an “invalid or expired token” error. The user must request a new reset email.

### Q3: Why do you build the reset link pointing to the Angular frontend instead of directly to the backend?
**Answer:**  
The backend only handles the token validation and password change. The frontend is responsible for showing the “Enter new password” form.  
We pass the token and email as URL parameters to the Angular route. Angular then extracts them and sends them back to the backend in a `POST /reset-password` request. This keeps the user experience clean and separates concerns.

---

## 5. Google External Login (SSO)

### Q1: When a user logs in with Google, why does your system issue its own access token and refresh token instead of using Google’s token?
**Answer:**  
Google’s token only tells us who the user is (their email and name). It does **not** know about our internal roles like `Administrator` or `Standard`. Also, Google’s token does not work with our silent refresh mechanism.  
So we “translate” the Google token into our own token pair. This way, we keep full control over session lifetime and permissions.

### Q2: What happens if a user logs in with Google for the first time? Do they need to register separately?
**Answer:**  
No. Our `GoogleLoginAsync` method checks if the email already exists in our database.  
If not, it **automatically creates** a new user account, assigns the `Standard` role, and logs them in – all in one step. The user does not have to fill out any registration form.

### Q3: How do you keep the Google login user hooked into your refresh token system?
**Answer:**  
After we receive the Google ID token, we generate our own refresh token exactly like in the normal login flow. That refresh token is stored in an `HttpOnly` cookie.  
So the Google user benefits from the same silent refresh process – when the access token expires, the browser automatically sends the refresh token cookie to get a new access token.

---

## 6. Global Logout (Log Out From All Devices)

### Q1: How does your system track different devices so that it can log out from all of them?
**Answer:**  
When a user logs in, we generate a unique **Device ID** (a GUID). We store the refresh token in the `AspNetUserTokens` table with a name like `RefreshToken_<deviceId>`.  
If the same user logs in from three different browsers, we have three separate refresh token records, each with a different device ID.

### Q2: What exactly happens when a user clicks “Log out from all devices”?
**Answer:**  
Three things happen:
1. We generate a **new SecurityStamp** for the user. This immediately invalidates all existing access tokens.
2. We delete **every** refresh token from the database that belongs to this user (all device IDs).
3. We clear the `refreshToken` and `deviceId` cookies from the current browser.  
The user is logged out on the current device, and any other device will be forced to log in again as soon as its access token expires or it tries to refresh.

### Q3: Imagine you press “Log out from all devices” on your laptop. Your phone is still logged in – how does the phone get logged out automatically?
**Answer:**  
The phone does not know instantly. But the phone’s access token is short‑lived (e.g., 15 minutes).  
When that token expires, the phone tries to silently refresh using its refresh token. The backend looks for that refresh token in the database – but we already deleted all refresh tokens during global logout.  
The refresh fails, and the phone’s error interceptor receives a `401`. The phone then clears its local state and redirects the user to the login screen. So the phone gets logged out within minutes.

***
