# How I Hardened LiliShop: An End-to-End Application Security Series

LiliShop is a full-stack e-commerce project I built, deployed, and maintain myself — a .NET backend, an Angular frontend, real integrations with Stripe, Google, and a few other services. Building it was one project. Making it genuinely secure turned out to be a much bigger one.

I ran a full white-box security review of the whole application — meaning I went through the actual source code myself, end to end, rather than just testing it from the outside. I found a number of real security issues, ranging from small oversights to a couple that could have had serious consequences. Then I fixed them, one at a time, and wrote a detailed article about each fix, using the real code from the project as the example throughout.

This post is the short version: what the problems were, why they mattered, and where to go if you want the full technical story behind any of them. Each linked article is written to actually teach the topic, not just document a code change — it explains why the vulnerability existed, how the attack would have worked, and why the fix was built the way it was, using real code and diagrams from the project throughout.

## A Quick Word on OWASP

You'll see the word **OWASP** a few times below, so it's worth explaining before diving in.

**OWASP** (the Open Worldwide Application Security Project) is a nonprofit that publishes the closest thing the security industry has to a shared checklist of "the most common ways web applications get broken into." Their best-known list is the **OWASP Top 10** — ten broad categories of vulnerability that show up, over and over, across real applications. When security professionals talk about a bug, they'll often reference which OWASP category it falls under, the same way a doctor might reference a diagnosis code. It's a shared vocabulary, not a rulebook — but it's the one almost everyone in the field uses.

I checked every fix in this series against these categories, partly to make sure I wasn't missing anything obvious, and partly because it's a useful way to show that this wasn't eight random tweaks — it was a structured pass through the categories that actually matter most.

## The Series: Eight Real Vulnerabilities, Eight Real Fixes

Each part below follows the same real code, in the same real project, and walks through exactly what was wrong and exactly how it was fixed. The articles are in the order I actually worked through them — starting with how someone logs in, moving through what they're allowed to do once they're in, and ending with how the browser itself is protected — so the sequence isn't random if you want to read them in order.

### 1. 🛡️ Brute-Force Attack Protection
Before this fix, someone could try unlimited passwords against any account, as fast as a script could send them — no slowdown, no lockout, nothing. This post covers the two defenses that now stop that: one that slows down anyone rapidly trying one password after another from one location, and one that locks an account after too many wrong attempts, no matter where the attempts come from.
[Read the full article](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0053_LiliShop-Security-01-BruteForce-Protection.md)

### 2. 🔐 Multi-Factor Authentication (MFA) for Admins
Admin accounts control the whole store, so a stolen password alone shouldn't be enough to get in. This post covers how LiliShop added a second login step for admins — a rotating code from a phone app — along with backup codes for a lost phone, and a fix so a simple typo doesn't throw you out of the login process entirely.
[Read the full article](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0054_LiliShop-Security-02-MFA.md)

### 3. 🔑 The JWT Forgery Vulnerability in Google Sign-In
"Sign in with Google" hands your browser a signed proof of identity — like an ID card with a seal on it. LiliShop's code was reading that ID card without checking whether the seal was genuine, meaning anyone could build a fake one claiming to be any user at all, including an admin. This post covers the fix, and what a "signed token" actually means under the hood.
[Read the full article](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0055_LiliShop-Security-03-GoogleSignIn-JWTForgery.md)

### 4. 🌐 Server-Side Request Forgery (SSRF) — The Printess Callback
One feature was designed to receive a simple "your file is ready" message from a partner service. The problem: nothing checked whether that message really came from the partner, or whether the address it pointed to was actually safe — meaning it could have been tricked into fetching private, internal data instead. This post covers the two independent checks that now close that door.
[Read the full article](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0056_LiliShop-Security-04-SSRF-PrintessCallback.md)

### 5. 🔓 Broken Access Control — IDOR and Admin Access
Could one logged-in customer see another customer's private data just by changing a number in a request? This post covers three different ways LiliShop makes sure a request only ever touches data that actually belongs to the person making it — plus an admin-only dashboard that used to have no real protection guarding it at all.
[Read the full article](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0057_LiliShop-Security-05-BrokenAccessControl-IDOR.md)

### 6. 💳 Payment & Business Logic Integrity
What happens if someone sends a negative quantity, or a different price than what's actually in the catalog? This post covers how every number that affects a charge gets re-checked against the real database before any money moves — including a real rounding bug, found along the way, that had been quietly undercharging customers for shipping.
[Read the full article](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0058_LiliShop-Security-06-PaymentIntegrity-Tampering.md)

### 7. 🔍 Information Disclosure — Logs, Errors, and Enumeration
Not every problem is about someone breaking in — sometimes a system just says too much on its own. This post covers three places that could happen: log files holding more detail than they should, error messages describing exactly what went wrong internally, and a login page that could be used to check which email addresses have an account.
[Read the full article](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0059_LiliShop-Security-07-InformationDisclosure.md)

### 8. 🌍 CORS Misconfiguration & Security Headers
A single mistake in how LiliShop checked which websites were allowed to talk to it meant a fake website with a name deliberately chosen to fool that check could pass it anyway — and, combined with how login sessions worked, that could have let a malicious site quietly read a logged-in visitor's private data. This post covers the fix, plus four extra browser-level protections against similar tricks.
[Read the full article](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0060_LiliShop-Security-08-CORS-SecurityHeaders.md)

## How These Map to OWASP

| # | Topic | Main OWASP Category |
|---|---|---|
| 1 | Brute-Force Protection | Identification and Authentication Failures |
| 2 | Multi-Factor Authentication | Identification and Authentication Failures |
| 3 | Google Sign-In (JWT Forgery) | Identification and Authentication Failures |
| 4 | SSRF (Printess Callback) | Server-Side Request Forgery |
| 5 | Broken Access Control (IDOR) | Broken Access Control |
| 6 | Payment & Business Logic Integrity | Insecure Design |
| 7 | Information Disclosure | Security Logging & Monitoring Failures / Sensitive Data Exposure |
| 8 | CORS & Security Headers | Security Misconfiguration |

Between them, these eight parts touch most of the categories that show up most often in real-world security reviews — which is exactly the point of checking against a list like this. It's less about chasing a perfect score, and more about making sure nothing obvious got skipped.

## Two More Things Worth Mentioning

These two aren't part of the security series itself, but they came out of the same overall effort, and each has a direct security angle worth knowing about.

**Secrets never live in the code.** None of LiliShop's real API keys, database passwords, or signing secrets are stored in the project's configuration files. They're kept in Azure's own settings and pulled in as environment variables at deploy time — meaning even if the entire codebase were public, none of the real credentials would be sitting in it.
[Read how the deployment is set up](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0052_Deploy-LiliShop-to-Azure.md)

**A clean build is a security tool, not just tidiness.** I went through the project and fixed 468 compiler warnings down to zero. The reasoning is simple: if a project has hundreds of warnings, nobody actually reads them anymore — which means if one of them is ever a real, dangerous signal, it's invisible in the noise. With zero warnings, any new one immediately stands out and gets investigated.
[Read the full write-up](https://github.com/jahanalem/LinkedIn2GitHub/blob/main/0051_resolving-compilation-warnings.md)

## Closing Thoughts

Security isn't something I treat as a one-time checklist — it's an ongoing part of how I build and maintain any web application. I keep learning, keep reviewing, and keep looking for ways to make LiliShop more secure over time.

---

*If you want to go deeper on any of the eight parts above, each one includes the real code, a full explanation of the vulnerability, and diagrams walking through exactly how the fix works.*
