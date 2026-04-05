# Role-Based and Policy-Based Authorization (RBAC)

> **Note:** This document describes the security and authorization system in Lilishop. This project is designed and maintained by a single developer. However, the word "we" is used throughout the document for consistency with standard technical writing.

In our [previous documents](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0035_global-logout.md), we discussed **Authentication** (proving *who* you are by logging in with a password or refresh token). Now, we will explore **Authorization** (proving *what* you are allowed to do). 

Lilishop uses a highly scalable security model called **Role-Based Access Control (RBAC)** combined with **Policy-Based Authorization** to protect sensitive areas of the API.

---

## 1. The Idea: Roles vs. Policies

To understand our system, we must understand the difference between a Role and a Policy:

* **Role (The Label):** A role is a simple tag attached to a user's account in the database. For example, a user might have the `Standard` role or the `SuperAdmin` role.
* **Policy (The Rulebook):** A policy is a flexible rule that an endpoint enforces. Instead of hardcoding an endpoint to say "only allow the Administrator role," we tell it to "enforce the `RequireAtLeastAdministratorRole` policy." 

**Why use Policies?** Policies allow us to create a hierarchy! By using a policy like `RequireAtLeastAdministratorRole`, we can automatically allow both `Administrator` and `SuperAdmin` to use the endpoint without writing complex logic on every single controller.

---

## 2. Defining the Constants

To keep our code perfectly clean and prevent spelling mistakes, we define our Roles and Policies as constants in the `Domain` layer. These files act as our single source of truth.

Our `Role` class clearly defines the hierarchy of users in the system:

```csharp
// LiliShop.Domain/Constants/Role.cs
namespace LiliShop.Domain.Constants
{
    public static class Role
    {
        /// <summary>
        /// somebody who has all privileges.
        /// </summary>
        public const string SuperAdmin = "SuperAdmin";

        /// <summary>
        /// somebody who has access to most of the administration features.
        /// </summary>
        public const string Administrator = "Administrator";

        /// <summary>
        /// somebody who can manage the discount system.
        /// </summary>
        public const string DiscountManager = "DiscountManager";

        /// <summary>
        /// somebody who can log in and buy something
        /// </summary>
        public const string Standard = "Standard";

        public const string AdminPanelViewer = "AdminPanelViewer";
    }
}
```

Our `PolicyType` class holds the names of the rules we will enforce on our API endpoints:

```csharp
// LiliShop.Domain/Constants/PolicyType.cs
namespace LiliShop.Domain.Constants
{
    public static class PolicyType
    {
        public const string RequireSuperAdminRole = "RequireSuperAdminRole";
        public const string RequireAtLeastAdministratorRole = "RequireAtLeastAdministratorRole";
        public const string RequireAtLeastDiscountManagerRole = "RequireAtLeastDiscountManagerRole";
        public const string RequireAtLeastStandardRole = "RequireAtLeastStandardRole";
        public const string RequireAtLeastAdminPanelViewerRole = "RequireAtLeastAdminPanelViewerRole";
    }
}
```

---

## 3. The Setup: Wiring the Hierarchy Together

When our application starts, we need to teach the ASP.NET Core framework what our policies actually mean. We do this inside our `IdentityServiceExtensions.cs` file.

This is where the true power of the system shines. We map a single Policy to **multiple acceptable Roles**, creating a perfect security hierarchy:

```csharp
// LiliShop.API/Extensions/IdentityServiceExtensions.cs

services.AddAuthorization(opts =>
{
    // Only the SuperAdmin can pass this policy
    opts.AddPolicy(PolicyType.RequireSuperAdminRole, policy => 
        policy.RequireRole(Role.SuperAdmin));

    // Both Administrators AND SuperAdmins can pass this policy
    opts.AddPolicy(PolicyType.RequireAtLeastAdministratorRole, policy =>
        policy.RequireRole(Role.Administrator, Role.SuperAdmin));

    // Both Administrators AND SuperAdmins can act as Discount Managers
    opts.AddPolicy(PolicyType.RequireAtLeastDiscountManagerRole, policy =>
        policy.RequireRole(Role.Administrator, Role.SuperAdmin));

    // Anyone with viewing rights or higher can pass this policy
    opts.AddPolicy(PolicyType.RequireAtLeastAdminPanelViewerRole, policy =>
        policy.RequireRole(Role.AdminPanelViewer, Role.Administrator, Role.SuperAdmin));

    // Almost all authenticated users pass this policy
    opts.AddPolicy(PolicyType.RequireAtLeastStandardRole, policy =>
        policy.RequireRole(Role.Standard, Role.AdminPanelViewer, Role.Administrator, Role.SuperAdmin));
});
```

---

## 4. The Execution: Protecting Endpoints

Applying these rules is incredibly easy. Whenever we have a sensitive action, we simply place the `[Authorize]` attribute at the top of the controller or the specific method, and pass in our constant.

```csharp
// Example: ProductsController.cs

[HttpPost("create-product")]
[Authorize(Policy = PolicyType.RequireAtLeastAdministratorRole)] // 🔒 The Shield!
public async Task<ActionResult<ProductToReturnDto>> CreateProduct(ProductToCreateDto productDto)
{
    // Only users who possess the "Administrator" or "SuperAdmin" roles will ever reach this code!
    var result = await _productService.CreateProductAsync(productDto);
    return HandleOperationResult(result);
}
```

Because we use constants, if we ever rename a policy, Visual Studio will automatically update it everywhere.

---

## 5. Edge Cases and Error Handling

What happens when users try to break the rules? The ASP.NET Core framework handles the rejections automatically before the controller code is ever executed.

Here is exactly how the system responds to different edge cases:

| Edge Case | What Happens | HTTP Status Code |
|-----------|---------------|-------------------|
| **A user is completely logged out and tries to access a protected endpoint.** | The system sees they have no Access Token. It rejects them immediately. | `401 Unauthorized` |
| **A `Standard` user tries to access a `RequireAtLeastAdministratorRole` endpoint.** | The user *does* have a valid token, so they pass the 401 check. However, the policy sees they lack the required roles. The system blocks them. | `403 Forbidden` |
| **A `SuperAdmin` accesses the same endpoint.** | The token is valid, and the policy accepts the `SuperAdmin` role. The controller runs normally. | `200 OK` (or `201 Created`) |
| **An Administrator's access token expires during the request.** | The system rejects the request. The Angular frontend catches the error and silently triggers the Refresh Token process to get a new token! | `401 Unauthorized` |

### Final Note
By separating **Roles** (who the user is) from **Policies** (the rules of the application), Lilishop achieves an enterprise-level security architecture. This hierarchical design allows us to easily add new roles or adjust permissions in the future without modifying hundreds of API endpoints!

***
