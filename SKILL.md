---
name: security-checks
description: Use when the user asks to run security checks, security audit, security review, vulnerability scan, or harden their app. Works for any project type — web, mobile, API, etc. Covers secrets, data privacy, production readiness, auth/payments, and attack-surface review.
---

# Security Checks

Run a full security audit on the codebase. This skill performs 5 checks in order. After each check, report what was found, what was fixed, and what passed.

---

## Check 1 — Secret Leak Prevention

Find and fix hardcoded secrets, API keys, tokens, and credentials.

1. Search the entire codebase for string literals that look like secrets: API keys, passwords, tokens, database URLs, connection strings, private keys.
2. Move every secret found to environment variables. No secret should exist as a string literal in source code.
3. Check framework-specific env var exposure (`NEXT_PUBLIC_`, `REACT_APP_`, `EXPO_PUBLIC_`, `VITE_`) — ensure no sensitive key uses these prefixes.
4. Verify `.env` is in `.gitignore`. Create `.env.example` with placeholder values if missing.
5. Check `console.log`, error handlers, and API responses for accidental secret leaks.
6. If secrets were previously hardcoded, warn that git history still contains them and recommend rotation.

**Report:** Table of every secret found, its location, and what it was moved to.

---

## Check 2 — Personal Data Flow Audit

Track how user data (emails, phones, passwords, PII) moves through the app.

1. Map every data collection point — forms, API endpoints, SDKs, analytics. Trace where each piece of data goes.
2. Clean all logs — remove any `console.log`, logger, or error handler that outputs user PII. Replace with "[REDACTED]" or remove entirely.
3. Audit every third-party integration — list what user data each service receives. Strip unnecessary fields.
4. Verify password handling — must use bcrypt, argon2, or scrypt. Never MD5/SHA256 alone. Plaintext must never be stored, logged, or returned in responses.
5. Check cookies for `httpOnly`, `secure`, `sameSite` flags. Ensure no PII is in `localStorage` or `sessionStorage`.
6. Verify API responses only return what the client needs — no password hashes, no internal IDs, no other users' data.
7. Check if a data deletion mechanism exists for users.

**Report:** Data flow map — what is collected, where stored, where sent externally, and what was fixed.

---

## Check 3 — Pre-Deploy Production Audit

Verify the app is production-ready.

1. **Env vars** — verify every required env var is referenced and the app refuses to start if critical vars (DB URL, API keys, auth secret) are missing.
2. **Debug code** — find and remove: `console.log` used for debugging, commented-out code, TODO/FIXME referencing incomplete security, hardcoded test credentials, test-only endpoints (`/test`, `/debug`, `/admin-backdoor`, `/seed-data`).
3. **Error handling** — no client-facing error should include stack traces, DB queries, file paths, or internal info. Return generic messages with a correlation ID. Detailed errors go to server logs only.
4. **Security headers** — ensure these are set: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security`, `Content-Security-Policy`. Use helmet or equivalent if available.
5. **Rate limiting** — auth endpoints (login, signup, password reset, OTP) must have rate limiting. Minimum: 5 attempts/min per IP on login, 3/hour on password reset.
6. **CORS** — must not be `*` unless the API is genuinely public. Restrict to specific frontend domain(s).
7. **Database** — production DB must use TLS/SSL, no default credentials, no open port without auth.

**Report:** Pass/fail for each check with fixes applied.

---

## Check 4 — Deep Security Audit (Auth, Payments, Input)

Professional-grade audit for critical paths.

### Authentication & Authorization
- Every protected route/endoint must have proper auth middleware.
- Check for IDOR — no endpoint should accept a user ID from the client and return data without verifying ownership.
- Password reset tokens must be random, single-use, time-limited (15 min max), tied to a specific user.
- JWT: strong signing secret, expiry set, token blacklist on logout.

### Payment Logic
- Server must independently calculate totals, taxes, discounts — never trust client-side calculations.
- Verify webhook signatures from payment providers (Stripe, Razorpay, etc.).
- Payment status must be verified server-side before granting access to paid features.

### Input Handling
- SQL injection — replace any raw SQL with parameterized queries/ORM.
- XSS — ensure no user input is rendered in HTML without sanitization.
- File uploads — validate type server-side, limit size, don't serve from same domain with executable permissions.

**Report:** For each vulnerability: what it is, where in code, how an attacker would exploit it, and the exact fix.

---

## Check 5 — Attacker's Perspective Review

Think like a hacker. Test these specific attack paths:

1. **ID manipulation** — can I access another user's data by changing an ID in the URL or request body? Test every endpoint that takes a user/order/document ID.
2. **Login bypass** — do any API endpoints work without an auth token? Are expired/malformed tokens rejected? Are there default admin accounts?
3. **Privilege escalation** — can a regular user access admin endpoints by guessing URLs or modifying their role? Role checks must be server-side, not just hidden UI.
4. **Feature abuse** — check rate limits on: signup (mass account creation?), messaging (spam?), file uploads (storage fill?), API calls (DDoS?), promo codes/referrals (infinite use?).
5. **Content injection** — try putting JavaScript in every text field (usernames, bios, comments, search, file names). Test SQL injection through search, filters, login forms.
6. **Internal exposure** — check if any of these are publicly accessible: DB admin panel, env vars via error messages, `.env` file via direct URL, `.git` directory, internal Swagger/OpenAPI docs, health check endpoints leaking system info.
7. **Business logic** — for payments: can I pay negative amounts? Stack discounts? Restart free trials? Refer myself? These are logic flaws, not code bugs.

**Report:** For each vulnerability: attacker action, damage potential, and immediate fix. Prioritize data theft and unauthorized access first, then abuse and logic flaws.

---

## Final Summary

After all 5 checks, produce a single summary:

| Check | Issues Found | Fixed | Remaining |
|-------|-------------|-------|-----------|
| 1. Secrets | | | |
| 2. Personal Data | | | |
| 3. Pre-Deploy | | | |
| 4. Deep Audit | | | |
| 5. Attacker Review | | | |

List any items that require manual review or cannot be auto-fixed.
