## 🛡️ System Security & Authentication Architecture in Lilishop

This section documents the enterprise-grade security, authentication, and authorization systems custom-built for Lilishop. It covers everything from silent token rotation and external SSO integration to fault-tolerant email verifications and anti-enumeration protections.

---

### 1. Refresh Token and Login Process Documentation in Lilishop
This document explains the secure authentication system built for the Lilishop application. It details how the system uses short-lived access tokens alongside long-lived refresh tokens to keep users securely logged in without interrupting their experience. 

**Key Takeaways:**
* **Secure Storage:** Refresh tokens are safely stored inside `HttpOnly` cookies. This prevents hackers from stealing the tokens using malicious JavaScript (XSS attacks).
* **Silent Refresh:** When a user's access token expires, the Angular frontend automatically catches the error, gets a new token in the background, and retries the original request silently. 
* **Token Rotation:** Every time the system refreshes a token, it generates a brand new refresh token and deletes the old one. This is a highly recommended security practice.
* **Smooth User Experience:** Users never see an error message or get asked to log in again, as long as their refresh token remains valid.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0034_refresh-token-system.md)

---

### 2. Registration and Email Confirmation Flow in Lilishop
This document outlines the complete lifecycle of a new user joining the Lilishop platform. It details the robust security and fault-tolerance measures taken to ensure every account belongs to a real, verified person.

**Key Takeaways:**
* **Fault-Tolerant Delivery:** The system utilizes a custom "exponential backoff" retry mechanism to guarantee the delivery of branded HTML emails even if the external email server experiences temporary hiccups.
* **Backend Redirection:** Instead of linking the email directly to the frontend, Lilishop safely routes the user back through the ASP.NET Core API for verification, before issuing a smooth `302 Redirect` to the Angular interface.
* **Strict Expirations:** The backend encodes the precise token issue time into the link, allowing the service to enforce a strict custom 24-hour expiration policy before querying the database.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0038_registration-and-email-confirmation.md)

---

### 3. Role-Based and Policy-Based Authorization (RBAC) in Lilishop
This document explains how Lilishop protects its API using a highly scalable Role-Based Access Control (RBAC) system combined with Policy-Based Authorization. It details the separation of user roles from endpoint policies to create a clean, hierarchical security structure.

**Key Takeaways:**
* **Roles vs. Policies:** Roles act as static labels for users (e.g., `SuperAdmin`, `Standard`), while Policies act as flexible rules for endpoints (e.g., `RequireAtLeastAdministratorRole`).
* **Hierarchical Security:** A single policy can accept multiple roles. This automatically creates a permission hierarchy without cluttering the controller logic.
* **Single Source of Truth:** Defining roles and policies as strict constants in the Domain layer prevents typos and makes global updates effortless.
* **Built-in Protection:** The ASP.NET Core framework automatically intercepts invalid requests, returning accurate `401 Unauthorized` or `403 Forbidden` statuses before they ever reach the business logic.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0036_role-and-policy-based-authorization.md)

---

### 4. Secure Password Recovery (Forgot / Reset Password) Flow in Lilishop
This document details the highly secure, multi-step process for account recovery. It highlights the protective measures taken to prevent database scraping, token forging, and replay attacks during the password reset cycle.

**Key Takeaways:**
* **Anti-Enumeration Security:** The system never reveals whether an email exists during the recovery request, neutralizing brute-force enumeration attacks.
* **Cryptographic Expiration:** The system utilizes one-time-use, time-sensitive tokens that are instantly destroyed the moment a password is changed, neutralizing replay attacks.
* **Branded Communication:** The backend dynamically loads and injects the secure recovery URL into a custom HTML template (`PasswordResetTemplate.html`), ensuring all communication maintains high-quality brand standards.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0039_secure-password-recovery.md)

---

### 5. Google External Login (Single Sign-On / SSO) in Lilishop
This document details the implementation of Google Single Sign-On (SSO). It explains the "Translation" strategy, where a third-party Google token is processed and seamlessly exchanged for a local, role-aware access token and a secure refresh token.

**Key Takeaways:**
* **Token Extraction:** The backend uses `JwtSecurityTokenHandler` to parse the Google ID Token and securely extract vital user claims like Email and Name.
* **Frictionless Registration:** If a user logs in with Google for the first time, the system silently creates an account and assigns them the `Standard` role, bypassing traditional registration forms entirely.
* **Local Authority & Refresh Continuity:** By discarding the Google token and issuing custom Lilishop Access and Refresh tokens, the system ensures SSO users benefit from the same silent background refresh process and strict Role-Based Access Control (RBAC) as standard users.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0037_google-external-login-sso.md)

---

### 6. Global Logout: Logging Out From All Devices in Lilishop
This document explains the "Log Out from All Devices" feature. It allows users to instantly terminate every active session across all their devices. This is a vital security tool for protecting accounts if a device is lost or stolen.

**Key Takeaways:**
* **Security Stamp:** The system uses a "Security Stamp" as a master kill-switch to invalidate all current tokens.
* **Database Wipe:** Every refresh token associated with the user is deleted from the database, preventing any device from staying logged in.
* **Automatic Kick:** Other devices are automatically logged out as soon as their short-lived access tokens expire and they fail to refresh.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0035_global-logout.md)

---

### 7. Security Architecture: Questions & Answers (English)
This document provides a comprehensive list of technical questions and detailed answers regarding the Lilishop security system. It serves as an excellent review guide for understanding the "how" and "why" behind the project's architectural decisions. 

**Key Takeaways:**
* **Concept Breakdown:** Features clear, easy-to-understand explanations of complex security workflows (such as silent token rotation, SSO translation, and Global Logout).
* **Architectural Insights:** Explores the reasoning behind specific security choices, such as using `HttpOnly` cookies and preventing email enumeration attacks.
* **Quick Reference:** Acts as a fast, accessible summary of all the major authentication and authorization concepts implemented across the API.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0041_security-architecture-questions-and-answers.md)

---

### 8. Sicherheitsarchitektur: Fragen & Antworten (Deutsch)
Dieses Dokument bietet eine umfassende Liste technischer Fragen und detaillierter Antworten zum Sicherheitssystem von Lilishop. Es dient als hervorragender Leitfaden, um das „Wie“ und „Warum“ hinter den architektonischen Entscheidungen des Projekts zu verstehen. 

**Wichtige Erkenntnisse:**
* **Konzeptübersicht:** Bietet klare, leicht verständliche Erklärungen komplexer Sicherheitsabläufe (wie lautlose Token-Rotation, SSO-Übersetzung und globaler Logout).
* **Architektonische Einblicke:** Erläutert die Gründe für spezifische Sicherheitsentscheidungen, wie die Verwendung von `HttpOnly`-Cookies und die Verhinderung von E-Mail-Enumeration-Angriffen.
* **Schnellreferenz:** Dient als schnelle, leicht zugängliche Zusammenfassung aller wichtigen Authentifizierungs- und Autorisierungskonzepte, die in der API implementiert wurden.

[Weiterlesen](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0042_sicherheitsarchitektur-fragen-und-antworten.md)

