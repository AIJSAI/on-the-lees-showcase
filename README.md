# On the Lees: A Private Network for Sommeliers

> A members-only, real-name network for sommeliers and wine professionals that I designed, built and maintain: a member directory, a forum, study groups with events, and Reconnect, for finding someone you met but never swapped contact details with. Live with members since June 2026. Built on Next.js 16 and Supabase, with Postgres row-level security as the final gate on every read and write a member's session makes.

---

**This repository documents the architecture and design decisions for On the Lees. The implementation is private, and so is the network: there is no public demo and no member data appears here.**

[Portfolio case study](https://jamesshehan.dev/projects/on-the-lees)

Other showcases: [Connect](https://github.com/AIJSAI/connect-voice-ai-showcase) · [Ratify](https://github.com/AIJSAI/ratify-showcase) · [Vinny](https://github.com/AIJSAI/vinny-showcase) · [Hive](https://github.com/AIJSAI/hive-showcase) · [Backyard](https://github.com/AIJSAI/backyard-showcase) · [The Chief](https://github.com/AIJSAI/operating-partner-showcase)

---

## Problem

It began with wine professionals who had trained together and ended up scattered across group chats, with no single members-only place to find each other, talk, or study. Members need to be able to find each other, so they use real names, but a real-name directory of working sommeliers must never be public or indexable.

The requirements that followed:

- **Private from the start.** Every member-facing page is behind sign-in. `noindex` is set in the page metadata and in an `X-Robots-Tag` response header, and `robots.ts` disallows every crawler.
- **Real names, gated by trust.** A new member gets in with an invite code a member shares, or by asking for access and waiting for a moderator's approval. Either path records a vouch, and a vouch is what verifies a member.
- **The network is the product.** The directory, forum and study groups are its core; Reconnect is one feature among them.
- **Phones first.** Controls a thumb has to hit are at least 44 by 44 pixels on a phone, measured on the main pages by a Playwright test in WebKit with iPhone emulation, and the app installs to a home screen.

## Architecture

Three checks gate every member page: `proxy.ts`, the `(authed)` layout, and Postgres row-level security, which binds every read and write a member's session makes. Server code that has to act beyond one member's rights, such as invite redemption and the storage sweep, runs as the service role, which RLS does not bind by design; the send counters sit behind a `security definer` function that only the service role may call.

```mermaid
flowchart TB
    subgraph Client["Phone or desktop browser"]
        UI["Installable web app"]
    end

    subgraph Vercel["Vercel: Next.js 16 App Router"]
        PX["proxy.ts<br/>refresh session,<br/>signed-out visitor to /join"]
        LAY["(authed) layout<br/>verified profile required,<br/>else /onboarding or /pending"]
        RSC["Server Components<br/>+ Server Actions<br/>(Zod at the boundary)"]
        FILES["/files/[id]<br/>access check, then 303<br/>to a 60-second signed URL"]
        DEL["deliver()<br/>the notification path"]
        CRON["Weekly summary cron<br/>bearer secret; built,<br/>not yet switched on"]
    end

    subgraph Supabase["Supabase"]
        AUTH["Auth<br/>magic link + 8-digit code"]
        PG[("Postgres<br/>RLS on every table,<br/>anon holds no grant")]
        ST[("Storage<br/>private buckets,<br/>signed URLs only")]
    end

    TS["Cloudflare Turnstile"]
    RS["Resend<br/>sign-in + notification email"]
    WP["Web Push services"]
    SE["Sentry<br/>errors first, PII scrubbed"]

    UI --> PX --> LAY --> RSC
    UI -. "widget token" .-> TS
    RSC -->|"getUser(), RLS-bound queries"| PG
    RSC -->|"signInWithOtp + token"| AUTH
    AUTH -. "redeems the token" .-> TS
    AUTH -->|SMTP| RS
    UI -->|"upload to own folder"| ST
    PX --> FILES
    FILES --> ST
    RSC --> DEL
    CRON --> DEL
    DEL -->|"reserve the send<br/>in one locked statement"| PG
    DEL --> RS
    DEL --> WP
    RSC -.-> SE
```

| Component | Function |
|-----------|----------|
| **`proxy.ts`** | Next 16's replacement for middleware. Refreshes the Supabase session and sends any signed-out request for a non-public route to `/join`. Public routes: `/`, `/join`, `/auth/*`, `/legal/*`, the install page, the web manifest, the push service worker, the unsubscribe link, and the weekly cron route, which checks its own bearer secret |
| **`(authed)` layout** | A signed-in visitor with no profile goes to onboarding, and one not yet verified waits on `/pending` |
| **Row-level security** | RLS is on for every table in the `public` schema. Member-facing tables carry members-only policies for the `authenticated` role; three server-only tables carry no policy and no grant. `anon` is denied everywhere and holds no table grant |
| **Private Storage** | Private buckets, own-folder uploads, short-lived signed URLs, read policies that follow the post; photos uploaded through the app are re-encoded in the browser first, so their EXIF metadata, GPS location included, is not uploaded |
| **`deliver()`** | The path for every notification except the transactional access-approved email: the suspended check, preferences, per-member send limits, and a signed one-click unsubscribe link |

Detail: [docs/architecture.md](docs/architecture.md).

## Tech Stack

| Technology | Role | Why This Choice |
|-----------|------|-----------------|
| Next.js 16 (App Router), React 19, TypeScript strict, Zod 4 | Web app | Server Components by default; client code only for forms, uploads and filters |
| Supabase Postgres + RLS | Data and authorization | The database decides who reads and writes what, so a missed UI check cannot leak a row through a member's session |
| Supabase Auth | Sign-in | Passwordless link or 8-digit code; Turnstile verified inside Auth |
| Resend + Web Push (VAPID) | Email and phone notifications | The service worker handles push only, with no cache, because phones get shared |
| Sentry | Error monitoring | No session replay, PII scrubbed before send |
| Vercel | Hosting and scheduled jobs | Manual production deploys; rollback by promoting the previous deployment |
| Vitest + Playwright (Chromium and WebKit) | Tests | RLS tests against a local Supabase Postgres; end-to-end tests against a production build |

## Technical Challenges & Solutions

### 1. Freezing the Columns a Policy Cannot Guard

**Challenge**: A Postgres UPDATE policy checks the existing row with `USING` and the new row with `WITH CHECK`, but it cannot compare the two. So a policy alone cannot stop an edit that moves a row out of its group, and copying the insert rule into `WITH CHECK` would take an author's edit rights away the day they left a group.

**Solution**: A read-only audit of the migrations checked every write path for this, and each finding was reproduced on a local stack before it was fixed. A trigger now freezes every column that names an owner, a parent or a scope, and an integration test that signs in as different members pins each case.

### 2. The Human Check Comes Before the Invite Code

**Challenge**: The single-use Turnstile token is verified in one place only, inside Supabase Auth's sign-in call; verifying it in the app as well would spend it first, and Auth would then reject every real sign-in. Anything the join action does before that call runs without a human check, and the join action used to read the invite code there.

**Solution**: The join action now sends the sign-in email first and reads the invite code only after Supabase accepts. A failed send looks up and writes nothing, and the page shows the same answer for a valid code, an invalid one and none. Applying a code after sign-in is limited per account by the hour, counted in a server-only table before any code is read, and it fails closed.

### 3. Mail Limits That Hold Under a Parallel Burst

**Challenge**: Several member actions email another member, and some can be repeated on demand, such as cancelling and re-sending a connection request. A network-wide cap would stop one member's flood by silencing everyone else.

**Solution**: Per-member hourly and daily limits in `deliver()`, plus a short spacing rule for the same sender, recipient and subject. A first version that read the count and then wrote it let a parallel burst through, so one function call now decides and records under a per-member advisory lock: one member's sends queue behind each other, and nobody else waits.

## Key Decisions

| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-06-14 | Build the defined product, and change it only for real problems that affect members | Every later change is argued against this rule in the decision log |
| 2026-06-16 | Adopt two items the launch audit had cut: a CI gate and error monitoring | Once the app was live, I wanted it diagnosable and safe to change. The CI gate took effect then; Sentry went live in production on 2026-09-20 |
| 2026-09-20 | Freeze ownership and scope columns by trigger | An UPDATE policy cannot compare a column's old and new values, so it cannot stop a row leaving its group |
| 2026-09-20 | Sign-in links by token hash, plus an 8-digit code | The default link works only in the browser that asked for it, and a phone opens it inside the mail app's own browser, so links tapped within seconds still failed |
| 2026-09-21 | The human check runs before anything about an invite | Changing the order needs no second verification, which would spend the single-use token |
| 2026-09-21 | Per-member email limits, decided under one lock | A network-wide cap punishes everyone for one member's flood, and a count taken outside the lock loses the race |

[docs/decisions.md](docs/decisions.md) gives each one's context and consequences.

## What I Learned

- **Write policies need the same scrutiny as read policies.** A row could change the very column its read policy scopes on.
- **A limit needs its check and its record under one lock.** Counting in one query and recording in the next let a parallel burst through; a per-member advisory lock around one `security definer` function now serializes them.
- **A local stack is not evidence about production.** Default privileges and auth rate limits differ between the two, so grants are now stated explicitly in a migration rather than trusted to defaults, and production's own configuration is read before a number about it is quoted.
- **A phone opens a sign-in link in the mail app's own browser.** The default magic link only works in the browser that asked for it, so sign-in now uses a token-hash link that needs no PKCE cookie from that browser, plus an 8-digit code that works anywhere.

## Results

From the source repository at the head of `main` (2026-09-25):

- **Live since June 2026**, with member moderators
- **46 database migrations; 25 tables, all in the `public` schema and each with row-level security enabled**
- **38 integration, 74 unit and 28 end-to-end test files**; the integration tests sign in as different members against a Supabase stack started for the run, on a laptop and in CI, to check the RLS rules
- **82 pull requests merged** to `main`
- **Eight notification email types plus web push**; every type except the access-approved email has its own switch and unsubscribe link
- **Self-serve data export and account deletion**, with group ownership handed off and every stored file swept

## Project Status

As the repository records it at the head of `main` (2026-09-25):

| Area | Status | Description |
|------|--------|-------------|
| Production | Live | Serving on its own domain since 2026-06-15 |
| Directory, Reconnect, forum and study groups | Live | Per-field contact privacy enforced in the database, by a function that nulls each field the viewer may not see; mutual opt-in connections; public, private and hidden groups with events |
| Trust and safety | Live | Invite and vouch, moderator review queue, report, block, suspend, action log |
| Human check, error monitoring, notifications | Live | Turnstile enforced on join; Sentry and web push live in production since 2026-09-20 |
| Weekly summary email | Built | Not switched on in production: its cron route refuses every run until its secret is set |
| Photo and file attachments on posts (up to four of each), Markdown subset, long-post folding, forum index layout | Merged | Merged to `main` on 2026-09-24 and 2026-09-25; deploys are manual, and the repository does not yet record one for these |

---

**Built by [James Shehan](https://jamesshehan.dev)**
