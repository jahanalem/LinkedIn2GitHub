# LinkedIn2GitHub

**LinkedIn2GitHub** is a repository to share [my LinkedIn](https://www.linkedin.com/in/said-roohullah-allem/) articles and posts.  
Topics include **web development**, **clean code**, and **project updates**. Feedback is always welcome! 😊

---

## 📚 Articles:

### 61. 🌍 LiliShop Security Series — Part 8: CORS Misconfiguration & Security Headers
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0060_LiliShop-Security-08-CORS-SecurityHeaders.md)

---

### 60. 🔍 LiliShop Security Series — Part 7: Information Disclosure — Logs, Error Messages, and User Enumeration
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0059_LiliShop-Security-07-InformationDisclosure.md)

---

### 59. 💳 LiliShop Security Series — Part 6: Payment & Business Logic Integrity — Defending Against Price and Quantity Tampering
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0058_LiliShop-Security-06-PaymentIntegrity-Tampering.md)

---

### 58. 🔓 LiliShop Security Series — Part 5: Broken Access Control — IDOR and Function-Level Authorization
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0057_LiliShop-Security-05-BrokenAccessControl-IDOR.md)

---

### 57. 🌐 LiliShop Security Series — Part 4: Server-Side Request Forgery (SSRF) — The Printess Callback
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0056_LiliShop-Security-04-SSRF-PrintessCallback.md)

---

### 56. 🔑 LiliShop Security Series — Part 3: The JWT Forgery Vulnerability in Google Sign-In
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0055_LiliShop-Security-03-GoogleSignIn-JWTForgery.md)

---

### 55. 🔐 LiliShop Security Series — Part 2: Multi-Factor Authentication (MFA) Implementation
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0054_LiliShop-Security-02-MFA.md)

---

### 54. 🛡️ LiliShop Security Series — Part 1: Brute-Force Attack Protection
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0053_LiliShop-Security-01-BruteForce-Protection.md)

---

### 53. How I Deployed LiliShop to Azure
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0052_Deploy-LiliShop-to-Azure.md)

---

### 52.1 Resolving Compilation Warnings in a .NET Project
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0051_resolving-compilation-warnings.md)

---

### 52. Migrating LiliShop to Angular 22: A Practical Report
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0050_lilishop-angular-22-migration.md)

---

### 51. Cypress End‑to‑End Testing: A Complete Beginner’s Guide with LiliShop 🔬🧪🔎🕵️
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0049_Cypress-Ent-to-End-Testing.md)

---

### 50. **LiliShop – Complete Technical Architecture Manual** (PDF, 160+ pages)  
Full documentation covering Clean Architecture, design patterns, backend (.NET 10), frontend (Angular 21), and all subsystems.  
[📄 Read the full manual](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/LiliShop_Technical_Architecture_Manual.pdf)

---

### 49. LiliShop: Comprehensive Technical Architecture, Design Patterns, and Structure Manual
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0048_LiliShop-Technical-Architecture-Manual.md)

---

### 48. Discount System Unit Tests – Technical Documentation (Lilishop) 🔬🧪🔎🕵️
This document explains how I built the unit tests for the LiliShop discount system. It focuses on the two most important parts: the price calculation logic and the workflow orchestrator. By using tools like xUnit and Moq to fake the database and background jobs, all 42 tests run in less than 5 seconds. The article shows how the tests prove that the system always calculates the correct price and performs actions in the right order.

[Read More](https://github.com/jahanalem/LiliShop-backend-dotnet-test/blob/main/README.md)

---

### 47. Refactored Multi-Discount Architecture In Lilishop
To support complex promotional strategies, I completely refactored the discount engine in LiliShop. This updated architecture allows multiple discount campaigns to overlap seamlessly, automatically calculating the best possible price for the customer. The article explores the "Best-Price Evaluation Engine," the "Clean-Slate" update pattern, and how the system uses optimistic concurrency to safely manage overlapping database updates.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0047_refactored-multi-discount-architecture.md)

---

### 46. Building a Fail-Safe Email System in .NET: A Practical Guide to Fallback and Resilience
Relying on a single third-party service for critical features like user registration is a risk. In this article, I explain how I architected a highly resilient, fault-tolerant email delivery system for LiliShop using advanced .NET design patterns and tools.

**Key Topics Covered:**
* **The Fallback Pattern:** Creating a wrapper to seamlessly switch from a primary email provider (SendGrid) to a backup provider (Gmail SMTP) without impacting core business logic.
* **Polly Library Integration:** Implementing an Exponential Backoff retry policy to gracefully handle transient network failures.
* **Secure Azure Configuration:** Managing sensitive SMTP credentials in production using Azure Environment Variables instead of hardcoded strings.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0046_Building-a-Fail-Safe-Email-System-in-NET.md)

---

### 45. Discount System Documentation (Lilishop)
A detailed look at the discount and pricing system behind Lilishop.  
To avoid runtime price calculations, the system uses a **Pre-calculated Read Strategy**, which keeps product reads fast with $O(1)$ performance on the storefront and moves large discount updates to asynchronous **Hangfire** background jobs.

The documentation explains how multi-level discount rules work for store-wide campaigns, how database consistency is protected using the **Unit of Work** pattern, and how important edge cases are handled, such as overlapping discounts, product price updates during active sales, and safely changing discount rules while a campaign is already running.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0045_discount-system-lilishop.md)

---

### 44. Stripe Payment Process in Lilishop
This article outlines the secure payment workflow implemented in the LiliShop project. It details the architecture connecting the Angular frontend, the .NET backend API, and the Stripe payment gateway. The core focus is on a "zero-trust" security model, where final payment confirmation is handled exclusively via secure asynchronous webhooks from Stripe to prevent client-side manipulation. The document includes a step-by-step process flow, a visual sequence diagram, and configuration instructions for local development using the Stripe CLI.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0044_stripe-payment-process.md)

---

### 43. Orchestrating the Offline-First Healthcare Data Lifeline (System Architecture Case Study) 
In this case study, You will read about a resilient, offline-first system architecture designed for a mobile caregiver application. The system tackles the challenge of recording and processing patient admission talks in rural areas with zero or unstable internet connectivity.

The article details the implementation of advanced architectural patterns to ensure zero data loss and GDPR compliance, including:
* The **Outbox Pattern** (WatermelonDB/RxDB) for local data storage under storage pressure.
* The **TUS Protocol** for resumable, chunked file uploads over 4G/3G drops.
* The **Claim-Check Pattern** to handle heavy audio files without overloading message brokers (RabbitMQ).
* An asynchronous AI pipeline utilizing **OpenAI Whisper** for transcription and **GPT-4o** (JSON Mode) for generating structured medical reports.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0043_offline-first-system-architecture-case-study.md)

---

### 42. System Security & Authentication Architecture in Lilishop 🛡️
I have custom-built an enterprise-grade security and authentication system for Lilishop. This includes token rotation, Google SSO, fault-tolerant email verification, secure password recovery, and Role-Based Access Control (RBAC). 

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0040_system-security-and-authentication-architecture.md)

---

### 41. Secure Password Recovery (Forgot / Reset Password) Flow in Lilishop
This document details the highly secure, multi-step process for account recovery. It highlights the protective measures taken to prevent database scraping, token forging, and replay attacks during the password reset cycle.

**Key Takeaways:**
* **Anti-Enumeration Security:** The system never reveals whether an email exists during the recovery request, neutralizing brute-force enumeration attacks.
* **Cryptographic Expiration:** The system utilizes one-time-use, time-sensitive tokens that are instantly destroyed the moment a password is changed, neutralizing replay attacks.
* **Branded Communication:** The backend dynamically loads and injects the secure recovery URL into a custom HTML template (`PasswordResetTemplate.html`), ensuring all communication maintains high-quality brand standards.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0039_secure-password-recovery.md?)

---

### 40. Registration and Email Confirmation Flow in Lilishop
This document outlines the complete lifecycle of a new user joining the Lilishop platform. It details the robust security and fault-tolerance measures taken to ensure every account belongs to a real, verified person.

**Key Takeaways:**
* **Fault-Tolerant Delivery:** The system utilizes a custom "exponential backoff" retry mechanism to guarantee the delivery of branded HTML emails even if the external email server experiences temporary hiccups.
* **Backend Redirection:** Instead of linking the email directly to the frontend, Lilishop safely routes the user back through the ASP.NET Core API for verification, before issuing a smooth `302 Redirect` to the Angular interface.
* **Strict Expirations:** The backend encodes the precise token issue time into the link, allowing the service to enforce a strict custom 24-hour expiration policy before querying the database.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0038_registration-and-email-confirmation.md)

---

### 39. Google External Login (Single Sign-On / SSO) in Lilishop
This document details the implementation of Google Single Sign-On (SSO). It explains the "Translation" strategy, where a third-party Google token is processed and seamlessly exchanged for a local, role-aware access token and a secure refresh token.

**Key Takeaways:**
* **Token Extraction:** The backend uses `JwtSecurityTokenHandler` to parse the Google ID Token and securely extract vital user claims like Email and Name.
* **Frictionless Registration:** If a user logs in with Google for the first time, the system silently creates an account and assigns them the `Standard` role, bypassing traditional registration forms entirely.
* **Local Authority & Refresh Continuity:** By discarding the Google token and issuing custom Lilishop Access and Refresh tokens, the system ensures SSO users benefit from the same silent background refresh process and strict Role-Based Access Control (RBAC) as standard users.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0037_google-external-login-sso.md)

---

### 38. Role-Based and Policy-Based Authorization (RBAC) in Lilishop
This document explains how Lilishop protects its API using a highly scalable Role-Based Access Control (RBAC) system combined with Policy-Based Authorization. It details the separation of user roles from endpoint policies to create a clean, hierarchical security structure.

**Key Takeaways:**
* **Roles vs. Policies:** Roles act as static labels for users (e.g., `SuperAdmin`, `Standard`), while Policies act as flexible rules for endpoints (e.g., `RequireAtLeastAdministratorRole`).
* **Hierarchical Security:** A single policy can accept multiple roles. This automatically creates a permission hierarchy without cluttering the controller logic.
* **Single Source of Truth:** Defining roles and policies as strict constants in the Domain layer prevents typos and makes global updates effortless.
* **Built-in Protection:** The ASP.NET Core framework automatically intercepts invalid requests, returning accurate `401 Unauthorized` or `403 Forbidden` statuses before they ever reach the business logic.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0036_role-and-policy-based-authorization.md)

---

### 37. Global Logout: Logging Out From All Devices in Lilishop
This document explains the "Log Out from All Devices" feature. It allows users to instantly terminate every active session across all their devices. This is a vital security tool for protecting accounts if a device is lost or stolen.

**Key Takeaways:**
* **Security Stamp:** The system uses a "Security Stamp" as a master kill-switch to invalidate all current tokens.
* **Database Wipe:** Every refresh token associated with the user is deleted from the database, preventing any device from staying logged in.
* **Automatic Kick:** Other devices are automatically logged out as soon as their short-lived access tokens expire and they fail to refresh.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0035_global-logout.md)

---

### 36. Refresh Token and Login Process Documentation in Lilishop
This document explains the secure authentication system built for the Lilishop application. It details how the system uses short-lived access tokens alongside long-lived refresh tokens to keep users securely logged in without interrupting their experience. 

**Key Takeaways:**
* **Secure Storage:** Refresh tokens are safely stored inside `HttpOnly` cookies. This prevents hackers from stealing the tokens using malicious JavaScript (XSS attacks).
* **Silent Refresh:** When a user's access token expires, the Angular frontend automatically catches the error, gets a new token in the background, and retries the original request silently. 
* **Token Rotation:** Every time the system refreshes a token, it generates a brand new refresh token and deletes the old one. This is a highly recommended security practice.
* **Smooth User Experience:** Users never see an error message or get asked to log in again, as long as their refresh token remains valid.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0034_refresh-token-system.md)

---

### 35. Trends in Testing 2026: Meine Insights von der imbus Roadshow 🔬🧪🔎🕵️
This article explores the future of software quality assurance, highlighting the shift from traditional test automation to autonomous **Agentic AI**. Based on insights from the imbus roadshow, the core theme is the **"Licence to Execute"**—giving AI the autonomy to perform complex testing tasks while humans keep control using strict safety boundaries (Guardrails). 

**Key Takeaways:**
* **Multi-Agent Systems:** Instead of one large AI, a team of specialized AI agents (Analyst, Designer, Coder) works together to test software.
* **Self-Healing Tests:** AI agents can understand web interfaces semantically. If a UI element (like an Angular button) changes, the AI fixes the test script in real-time without breaking.
* **LLM-as-a-Judge:** AI is used to test other AI models to ensure correctness and reduce hallucinations.
* **The New Role of Developers/Testers:** The job is changing. Developers and testers will write fewer manual scripts and instead become **Quality Architects** or **Evaluation Engineers** who design, orchestrate, and supervise AI systems. 
* **The Value of Coding Skills:** Deep technical knowledge (like C#/.NET and Angular) is a huge advantage for writing powerful prompts and controlling AI agents effectively.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0033_imbus-trends-in-testing-2026.md)

---

### 34. Lilishop Update: Migration to .NET 10 and Angular 21
This release documents the comprehensive migration of the **Lilishop** platform from .NET 9/Angular 20 to **.NET 10** and **Angular 21**. The update prioritizes high-performance backend architecture, significant memory reduction, and code modernization.
Key Technical Improvements:
* **High-Performance Serialization:** Complete replacement of `Newtonsoft.Json` with native **System.Text.Json**. This leverages "Zero-Allocation" deserialization (reading directly from memory bytes) to achieve up to 4x faster performance and reduced GC pressure.
* **EF Core 10 Batch Operations:** Implementation of `ExecuteUpdateAsync` and `ExecuteDeleteAsync` to perform bulk database modifications in a single round-trip, bypassing the need to load entities into memory.
* **Tag-Based Hybrid Caching:** A robust caching system using **HybridCache** and `SerializeToUtf8Bytes`. It stores raw bytes to save memory and utilizes tag-based invalidation for instant, O(1) cache clearing, replacing slow Redis pattern scans.
* **Codebase Cleanup:** Removal of legacy formatters and wrapper classes, resulting in a cleaner, dependency-light architecture.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0032_Lilishop_Migration_NET10_and_Angular21.md)

---

### 33. Neues Projekt: Malteser Tandem App (Microsoft Power Apps und SharePoint)
This project documents the development of the **Malteser Tandem App**, a low-code solution built with **Microsoft Power Apps** and **SharePoint** for the Malteser Hilfsdienst.The application digitizes the management of volunteer-based "Tandem" sponsorships, connecting volunteers with individuals seeking support.

### Key Features

* **Volunteer & Case Management:** Centralized tracking of volunteers and support-seekers including skills, contact details, and matchmaking status.
* **Automated Data Import:** A backend **Power Automate** flow that extracts data from Excel uploads and populates the SharePoint database automatically.
* **Robust Backend:** Utilizes SharePoint Online for data storage with a "Soft Delete" mechanism to prevent accidental data loss.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0031_Malteser_PowerApp_Project.md)

---

### 32. Catch Eleven: A C# Game Engine with Clean Architecture
A C#/.NET 9 project for the classic Persian card game "Pasoor" (Catch Eleven). The project was refactored to use Clean Architecture, separating the core game logic (Domain/Application) from the UI (Infrastructure/ConsoleUI). This creates a testable (with XUnit) and maintainable "game engine" ready for any future UI, like WPF or Angular.

[Read More](https://github.com/jahanalem/CatchEleven)

---

### 31. Go (Golang) Rectangles: A Performance Comparison with C#
A case study comparing Go and C# performance on a concurrent, computational task. This Go program finds all possible rectangles from 2D points, running up to 20x faster than the original C# implementation.

[Read More](https://github.com/jahanalem/go-rectangles?tab=readme-ov-file#go-rectangles-a-performance-comparison-with-c)

---

### 30. New Skill Acquired: Foundations of Go (Golang)
I’ve completed a foundational Go (Golang) course as a first step toward learning microservices architecture for my e-commerce project LiliShop.
While the core stays in .NET, I plan to use Go for high-concurrency services like notifications, thanks to its efficiency and lightweight performance.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0030_golang-foundations-and-microservices-plan.md)

---

### **29. Wachstum über den Code hinaus: Ein Blick auf meine Projekte außerhalb der Entwicklung 🌱 💪**
This article is about personal growth outside of work. It covers two of my projects: a fitness journey with systematic progress tracking, and my volunteer work with Malteser. My roles there include helping seniors with tech at a "Café Digital" and serving as an IT-Referent.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0027_growth-beyond-code.md)

---

### **28. RectanglesCalculator: Rechtecke aus Punkten finden**
Dieses Projekt ist eine C#-Anwendung. Es löst eine interessante Aufgabe: Finde alle möglichen Rechtecke aus einer gegebenen Liste von Punkten. 
**Was macht das Projekt besonders?**

* **Intelligenter Algorithmus:** Es benutzt einen klaren und cleveren Algorithmus, um die Rechtecke zu finden.
* **Schnelle Verarbeitung:** Das Programm ist sehr schnell. Es nutzt **Parallel-Processing**, um die Arbeit auf mehrere CPU-Kerne zu verteilen.
* **Sauberer Code:** Der Code ist gut strukturiert und leicht verständlich. Er benutzt moderne C#-Techniken.
* **Effizient:** Durch smarte Tricks wie Caching und optimierte Datenstrukturen ist das Programm sehr effizient.

Dieses Projekt ist ein super Beispiel dafür, wie man ein komplexes Problem mit gutem Softwaredesign und modernen Programmiertechniken lösen kann.

[Read More](https://github.com/jahanalem/RectanglesCalculator?tab=readme-ov-file#dokumentation-wie-man-rechtecke-aus-punkten-findet)

---

### **27. From Monolith to Masterpiece: Refactoring LiliShop with Clean Architecture**
This article documents the refactoring of the LiliShop e-commerce platform's backend from a traditional N-Layer system to Clean Architecture. The change was made to improve maintainability, testability, and technology independence by separating the software into four distinct layers: Domain (core business logic), Application (use cases), Infrastructure (external details like databases and services), and API (the presentation layer). By enforcing the Dependency Rule, where all dependencies point inward toward the Domain, the project achieves a more robust, scalable, and flexible foundation, as illustrated by the successful decoupling of framework-specific components like ASP.NET Identity from the core business rules.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0026_Refactoring-LiliShop-with-Clean-Architecture.md)

---

### **26. Homeoffice Zeiterfassung – Ein Full-Stack-Projekt (.NET 9 und Angular 20)**
Eine moderne Webanwendung zur Zeiterfassung im Homeoffice, realisiert als Full-Stack-Projekt mit .NET 9 und Angular 20. Mitarbeiter können sich einloggen, um ihre Arbeitszeit zu starten und zu stoppen. Nach Abschluss eines Eintrags versendet das System automatisch eine Benachrichtigung per E-Mail an die Personalabteilung.

[Read More](https://github.com/jahanalem/HomeofficeApp/tree/client?tab=readme-ov-file#homeoffice-zeiterfassung--ein-full-stack-projekt)

---

### **25. How to Upgrade Angular v19 to v20 and Deploy to Azure Using GitHub Actions**
I recently upgraded my project LiliShop from Angular 19 to Angular 20. In this article, I have explained the key steps I took, focusing on important changes like the new browser folder in the build output. I also cover how I updated my ASP.NET Core backend and the GitHub Actions workflow to ensure a smooth deployment to Azure.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0025_upgrade-angular19-to-20-azure-gh-actions.md)

---

### **24. LiliShop's Group Discount System**
This article dives into the flexible group discount feature I developed for my e-commerce platform, LiliShop 🛍️. The system allows for creating discounts for **groups of products** based on their **brand** 🏷️ and/or **type** 👕. Admins can select a specific brand or type, or even apply the discount to **ALL** brands or types! 🌐

Each discount rule can be set with a specific **start** 🗓️ and **end date** 🔚, giving you full control over promotional periods. For example, you can easily set a discount for all products of a certain brand for a week! 🗓️➡️🔚

It's also important to note the **priority**: if a product has both a specific, single-product discount and a group discount, the **single-product discount will always be applied** 🥇. This ensures clarity for customers.

LiliShop is built with modern technologies like **Angular 19** 💻 for the frontend and **.NET 9** ⚙️ for the backend, running smoothly on **Microsoft Azure** ☁️. This group discount system is designed to be both powerful and easy for administrators to manage 💪.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0024_group-discount-system.md#lilishops-group-discount-system-)

---

### **23. Discount System Algorithm Documentation**
LiliShop's discount feature was redesigned from a simple model with fields inside the `Product` table to a more flexible and scalable structure using dedicated `Discount` and `ProductDiscount` tables.

This change brings key benefits:
- Reusable discounts across products  
- Support for future features like group discounts  
- Cleaner and more maintainable database structure  

To handle the new complexity, a custom `ProductMapper` was implemented instead of AutoMapper, offering better control, testability, and performance for mapping logic.

Because the article is quite long, I won’t include all the technical details here.  

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0023_LiliShop_discount-system-algorithm-documentation.md)

---

### **22. Printess Editor Integration with .NET 9 & Angular 19**
This guide explains how to integrate **Printess Editor**, a powerful online design editor for print products, into a modern **.NET 9 backend** and **Angular 19 frontend** application.

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0022_LiliShop_PrintessEditor_Integration_Guide.md)

---

### **21. New Feature: Sale Status Filter and Admin Sale Column**

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0021_sale-feature-update.md)

---

### **20. Discount Feature with Time Scheduling Implemented!**

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0020_discount-feature-scheduling.md)

---

### **19. More Speed and Cleaner Code with async/await and Promise.all (LiliShop)**

[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/improving-product-loading-in-admin-page.md)

---

### **18. New Price Drop Subscription Feature in LiliShop**
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/price-drop-subscription.md)

---

### **17. Migrating SmartSupervisorBot to a Web API**
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/Migrating-SmartSupervisorBot-to-a-Web-API.md)

---

### **16. My Recent Updates: Moving Forward with SmartSupervisorBot**
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/smart-supervisorbot-updates-net9.md)

---

### **15. Boosting Performance with HybridCache in .NET 9: New Feature in LiliShop**
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/hybrid-cache.md)

---

### **14. Enhancing Security and User Control: New Features in LiliShop**
- **Refresh Token**
- **Logout from All Devices**
- **Role Change Update**
- **RFC 9457 - Problem Details for HTTP APIs**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/enhancing-security-and-user-control.md)

---

### **13. Performance Boost: Upgrading LiliShop to .NET 9 and Angular 19**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/performance-boost-dotnet9-angular19.md)

---

### **12. Optimizing Rectangle Detection: Performance and Memory Improvements**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/rectangle-detection-optimization.md)

---

### **11. Revamping My Blog Application: From .NET 5 to .NET 8**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/blog-app-net8-upgrade.md)

---

### **10. Certification Achievement: Postman REST API Testing 🔬🧪🔎🕵️**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/postman-certification-completion.md)

---

### **9. Improving API Error Handling: My Approach 🔬🧪🔎🕵️**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/api-error-handling-improvements.md)

---

### **8. Developing Unit Tests for LiliShop 🔬🧪🔎🕵️**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/lilishop-unit-tests-xunit.md)

---

### **7. Automating Deployments with GitHub Actions: LiliShop Update**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/github-actions-automation.md)

---

### **6. LiliShop: Deployed on Microsoft Azure!**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/lilishop-azure-deployment.md)

---

### **5. Email Notification Feature for LiliShop**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/lilishop-email-notifications.md)

---

### **4. Angular 18 Upgrade: Enhancing LiliShop**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/lilishop-angular18-upgrade.md)

---

### **3. Angular 18.1 Upgrade: Enhancing LiliShop**
- **Removed zone.js**, allowing the app to run faster. Without zone.js, Angular avoids unnecessary change detection, making the app smoother and quicker.
- **Implemented OnPush Change Detection**, so Angular checks for changes only when necessary.
- **Leveraged Signals** for easier and cleaner state management, reducing extra code and simplifying handling of state changes.  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/lilishop-angular18-zone-less-onpush.md)

---

### **2. Neues Zertifikat: Deutsch-Test für den Beruf C1 von Telc**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/deutsch-c1-telc-zertifikat.md)

---

### **1. SmartSupervisorBot: Verbesserte Gruppeninteraktionen auf Telegram**  
[Read More](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/smart-supervisor-bot-telegram.md)

---

Feel free to explore, star the repository, and share your feedback or contributions! 🌟
