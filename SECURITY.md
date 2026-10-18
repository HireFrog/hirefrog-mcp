# Security

## Reporting a vulnerability

Email **support@hirefrog.co** with "Security" in the subject. Please include
what you found, how to reproduce it, and what you think the impact is. Don't
open a public issue for a vulnerability.

We'll acknowledge your report within three working days and keep you
updated. We won't take legal action against good-faith research that
respects these rules:

- test only with your own HireFrog account;
- don't access, change or delete anyone else's data, and stop as soon as you
  see data that isn't yours;
- no denial-of-service or load testing, and no social engineering;
- give us reasonable time to fix an issue before you disclose it.

## Scope

- The hosted connector at `https://mcp.hirefrog.co` (the MCP endpoint, its
  discovery documents and the page at the bare domain).
- HireFrog's approval page (`https://hirefrog.co/oauth/consent`), the
  confirmation page for requests (`https://hirefrog.co/app/confirm/…`) and
  Settings → Connected apps.
- How a connector token is treated by HireFrog's database and services.

The retired self-hosted server (v0.1) is out of scope.

## How the connector protects your account

The design in short. Each point is enforced in code or in the database, and
covered by tests.

1. **Only Claude and ChatGPT.** An app is approved only if every redirect
   address it registered is exactly on `claude.ai` or `chatgpt.com` (https,
   no port, no subdomain, no lookalikes). Any other app is refused by the
   server, and the database returns nothing to it.
2. **Never a HireFrog browser session.** The connector accepts only tokens
   issued to an approved connector app, checked with HireFrog's Auth server
   on every request (issuer, audience, role, expiry, client).
3. **Everything runs as you.** Each read and write runs on your own token
   under HireFrog's database access rules. The server holds no administrator
   key, and a build check fails if its code ever names one.
4. **The database is the boundary.** A connector token can reach only your
   pipeline (roles, agents, stage history) and two curated summaries, whether
   it comes through the connector or straight to the database. Every other
   table denies it. Your profile's contact details, date of birth,
   self-identification answers, pay expectations and work authorization are
   never selected.
5. **Changes are limited.** Directly, a connector can only save a role or move
   it forward; the database refuses any other change to a role. Everything
   else is a request the database stores with its exact arguments, and only
   your click on HireFrog's own page, under your normal sign-in, runs it. A
   connector token can't confirm, change or delete a request.
6. **Untrusted text is marked.** Job-posting text and similar third-party
   text is fenced as untrusted data in every result, and markers inside it
   are neutralised.
7. **Limits and counts, no content.** Per-user rate limits and a cap on open
   requests apply across all your apps. HireFrog stores daily counts only:
   never arguments, results or text. Logs never contain tokens or content.
8. **You can disconnect at any time.** Settings → Connected apps →
   Disconnect revokes the app and stops tokens it already holds, at once.
9. **No shared caching.** Every response is marked `private, no-store`.
10. **Kill switch.** HireFrog can turn the connector off for everyone within
    seconds without affecting the rest of HireFrog.

### Known limitation

HireFrog signs users in with Supabase, which includes the account's sign-in
email and phone number (if any) in every access token it issues, connector
tokens included, and returns the account to any valid token. HireFrog's tools
never return these details; the approval page tells you which ones your
account has before you approve.

## Independent review

Before opening to other users, the connector was reviewed by a reviewer that
did not build it, against a written attack list (other users' data, forged
redirect hosts, direct database use, self-confirmation, prompt injection,
excluded fields, rate-limit bypasses, the kill switch, secrets and caching).
Findings were fixed or accepted before launch.
