# SOURCES (review only: delete this file before publishing)

This file is for review. It maps every factual claim in this showcase to the place in the private source repository that supports it, and it must be deleted before the showcase is published. **Never publish a git history that contains this file:** before any push, remove it from the commit (`git rm --cached SOURCES.md`, then `git commit --amend` keeping the message), or publish from a fresh `git init` of the public files (README.md, LICENSE, docs/).

Paths are relative to the source checkout (`~/Projects/somm-cohort`, repository `AIJSAI/on-the-lees`, read at commit `4f26bfc`, 2026-09-25). Line numbers are as read on 2026-10-07.

---

## 1. Status Facts (Tagline, Results, Project Status)

| Claim | Source |
|-------|--------|
| A personal project James built and runs | The source repository is his own; `docs/DECISIONS.md:19` (his standing rule) |
| Live with real members since June 2026 | `docs/RUNBOOK.md:23-24` |
| Serving on its own domain since 2026-06-15 | `docs/RUNBOOK.md:85`, `:92-94` |
| First commit 2026-06-13 | `git log --reverse` (first commit "Initial commit from Create Next App", 2026-06-13) |
| Live with a real member by 2026-06-16 | `docs/DECISIONS.md:58` |
| 22 members by 2026-09-24 | `docs/DECISIONS.md:410` |
| Three moderators as of 2026-09-20 | `docs/RUNBOOK.md:118-119` |
| Goodwill, non-commercial, unproven | `docs/DECISIONS.md:19`, `docs/RUNBOOK.md:17` |
| Sentry live in production 2026-09-20 | `docs/RUNBOOK.md:788` |
| Push keys set in production 2026-09-20 | `docs/RUNBOOK.md:67-72`, `:948-953` |
| Weekly summary refuses every run until its secret is set (as of 2026-09-23) | `docs/RUNBOOK.md:74-76`, `:8` |

**Conflict to know about:** `CLAUDE.md:12` still says "Code is feature-complete and merged; what remains is deploy wiring." That line is stale. `docs/RUNBOOK.md:23-24` says so explicitly. The showcase follows the runbook.

**Deploy status of the late-September features:** photos and files (#86), the Markdown subset (#85), long-post folding (#84) and the forum index layout (#91) merged on 2026-09-24 and 2026-09-25 (`git log`). The repository does not record their production deploy (`docs/RUNBOOK.md:689-695`, `:1063-1064`), so the README marks them "Merged", not "Live". Re-read the deploy record before publishing.

---

## 2. README.md

| Claim | Source |
|-------|--------|
| Product: directory, forum, study groups with events, Reconnect; Reconnect is one feature | `CLAUDE.md:3-7`; `docs/DECISIONS.md:7-11` |
| Origin: wine professionals who trained together, scattered across private chats | `docs/DECISIONS.md:12-15` (generalized; see section 6) |
| Private, never indexable, real-name, vouch by invite | `docs/DECISIONS.md:16-18` |
| `noindex` at three layers | `docs/DECISIONS.md:40`; `src/app/layout.tsx:23-25`; `src/app/robots.ts:5-9`; `next.config.ts:21` |
| Invite code shared by members; moderators approve requests | `supabase/migrations/20260616140000_vouch_is_moderator_only.sql:1-14`; `docs/RUNBOOK.md:188-200` |
| 44 by 44 pixels for controls a thumb has to hit, checked on the iPhone engine; exemptions exist | `REVIEW.md:20`; `docs/RUNBOOK.md:1104-1111` (exemption list); `docs/DECISIONS.md:518-520` (member links are 17px, exempt) |
| Installable app | `src/app/manifest.ts`; `src/app/app/page.tsx` |
| Three gates (proxy, authed layout, RLS) | `CLAUDE.md:65-73`; `src/proxy.ts:7-26`; `src/app/(authed)/layout.tsx:37-39` |
| Public routes, including the install page, manifest, service worker, unsubscribe and cron route | `src/lib/public-paths.ts:16-43` (`PUBLIC_PREFIXES`, `PUBLIC_EXACT`); `CLAUDE.md`'s shorter list is stale |
| RLS on every public table; member-facing tables have members-only policies; three server-only tables have no policy and no grant | `docs/DECISIONS.md:29-33`, `:121-125`, `:153-155`, `:214-217` |
| RLS gates a member's session; the service role bypasses it by design | `CLAUDE.md:74-75` |
| `anon` holds no table grant | `supabase/migrations/20260921100000_anon_holds_no_table_grants.sql`; `docs/RUNBOOK.md:677-687` |
| Private Storage, own-folder uploads, signed URLs, read policies follow the post, EXIF stripped on device | `CLAUDE.md:84-88`; `docs/RUNBOOK.md:315-321`; `supabase/migrations/20260920100200_storage_read_scope.sql:1-3` |
| `deliver()` carries every notification except the access-approved email | `src/lib/notify.ts:285-305` ("the ONLY app notification that does not pass through `deliver`"); `src/lib/notify-delivery.ts:110-250` |
| Diagram: `/files/[id]` reached through the proxy, 303 to a 60-second signed URL | `docs/RUNBOOK.md:321`; `src/app/(authed)/files/[id]/route.ts` |
| Diagram: weekly cron with bearer secret | `vercel.json`; `docs/RUNBOOK.md:857-863`; `src/lib/public-paths.ts:29-32` |
| Diagram: Turnstile redeemed by Supabase Auth; Auth mail through Resend SMTP | `docs/RUNBOOK.md:748-765`, `:100-104` |
| Diagram: Sentry errors first, PII scrubbed (tracing at 0.1) | `docs/DECISIONS.md:66-71`; `docs/RUNBOOK.md:788-805`, `:802` |
| Stack versions (Next 16, React 19, Zod 4, web-push, Sentry) | `package.json` |
| Server Components by default; client only for forms, uploads, filters | `CLAUDE.md:91` |
| Lora and Inter, one garnet accent | `CLAUDE.md:94-96` |
| Service worker handles push only, no cache | `docs/RUNBOOK.md:1034-1039` |
| Manual deploys; rollback by promotion | `docs/RUNBOOK.md:288-293` |
| e2e against a production build | `docs/DECISIONS.md:285-315` |
| Challenge 1 (writes not scoped; freeze triggers) | `docs/DECISIONS.md:80-105` |
| Challenge 2 (invite oracle; order fix; 878 guesses a minute on the local stack; five an hour) | `docs/DECISIONS.md:127-156` (`:150-151`: "Measured on the local stack"); `docs/RUNBOOK.md:758-776` |
| Challenge 3 (mail quota; per-actor caps; 40 of 40 measurement; advisory lock) | `docs/DECISIONS.md:164-228`; `docs/RUNBOOK.md:589-644` |
| Key decision: token-hash links; 11 of 19 links tapped within seconds and still failed | `docs/RUNBOOK.md:406-411` (the cookie jar is the explanation, the 11 of 19 is the measurement) |
| Key decision: Sentry adopted 06-16, live in production 2026-09-20 | `docs/DECISIONS.md:56-71`; `docs/RUNBOOK.md:788` |
| Learned: 18 original tables, anon grants for three months | `docs/RUNBOOK.md:677-687` |
| Learned: audit cut Sentry and CI on 06-14, adopted 06-16 | `docs/DECISIONS.md:45-49`, `:56-65` |
| 46 migrations; 25 tables, RLS enabled on all 25 | `ls supabase/migrations`; `grep 'create table'` and `grep 'enable row level security'` (25 distinct each) |
| 707 Vitest and 118 Playwright green locally on 2026-09-21 | `.github/workflows/ci.yml:65-66` ("Proven locally 2026-09-21") |
| 38 integration test files | `ls tests/integration/*.test.ts` (38) |
| 82 pull requests merged by 2026-09-25 | `git log --format=%s` on `main`: 82 distinct `(#N)` references in 89 commits |
| Eight notification emails with per-type switches | `docs/RUNBOOK.md:696-702`, `:854-855` |
| Self-serve export and deletion; group handoff; storage sweep | `docs/RUNBOOK.md:232-236`; `src/lib/account.ts:158-204` |
| Status: contact fields masked in the database | `docs/DECISIONS.md:76-78`; `docs/RUNBOOK.md:723-726` |

---

## 3. docs/architecture.md

| Section | Source |
|---------|--------|
| 1. Three gates, `getUser()` rule, public routes | `src/proxy.ts:7-26`; `src/lib/public-paths.ts:16-43`; `src/app/(authed)/layout.tsx:37-39`; `CLAUDE.md:65-76`; `REVIEW.md:29-30` |
| 1. `noindex` header | `next.config.ts:5-31` |
| 2. Join order of operations | `docs/DECISIONS.md:127-147`; `src/app/join/actions.ts:136-150` |
| 2. Reply text "An email was sent" | `src/app/join/actions.ts:171` |
| 2. `/auth/confirm` and `/auth/continue` are app routes; continue calls `verifyOtp` | `src/app/auth/confirm/route.ts:1-10`; `src/app/auth/continue/actions.ts:33`; `docs/RUNBOOK.md:468-473` |
| 2. A verified member lands on the forum | `src/app/onboarding/actions.ts:88` (`redirect("/forum")`) |
| 2. Vouch on redeem; moderator-only vouching; invites capped | `src/app/onboarding/actions.ts:77`; `src/lib/invites.ts:108-113`; `supabase/migrations/20260616140000_vouch_is_moderator_only.sql:1-20`; `docs/RUNBOOK.md:177-200` |
| 3. Tables and relationships | `supabase/migrations/*.sql` (`create table` and `references` clauses) |
| 3. Optional ends in the ER diagram | `supabase/migrations/20260613160000_init_schema.sql:162`, `:294-297`, `:429`, `:462`, `:476` (nullable `created_by`, `reporter_id`; votes target exactly one parent); `20260924190000_post_files.sql:49-50`; `20260920110000_mod_actions.sql:29` |
| 3. Server-only tables | `docs/DECISIONS.md:121-125`, `:153-155`, `:214-217` |
| 4. Techniques table | `docs/DECISIONS.md:29-33`, `:94-110` |
| 4. Contact visibility levels | `src/lib/validation.ts:375-389`; `docs/RUNBOOK.md:723-726` |
| 4. Moderator action log | `docs/RUNBOOK.md:261-271` |
| 5. Photos and files | `docs/RUNBOOK.md:311-336`; `docs/DECISIONS.md:333-391` |
| 6. `deliver()` exception for the access-approved email | `src/lib/notify.ts:285-305` |
| 6. Flow order: suspended read first (fail closed), then preferences, then email cap only with an actor, then push only with a device | `src/lib/notify-delivery.ts:117-133`, `:154-171`, `:217`, `:227-242`; `docs/RUNBOOK.md:625-627` ("System mail has no actor and is never capped") |
| 6. Limits table and bullets | `docs/RUNBOOK.md:597-671`; `docs/DECISIONS.md:164-228` |
| 7. Sentry posture, 10% tracing, live 2026-09-20; service worker | `docs/RUNBOOK.md:788-805`; `docs/DECISIONS.md:66-71`; `docs/RUNBOOK.md:1034-1039` |
| 8. CI jobs; build and test required; e2e for signal; `enforce_admins` off | `.github/workflows/ci.yml`; `docs/RUNBOOK.md:298-305` |
| 8. RLS-coverage structural test | `tests/unit/rls-coverage.test.ts:1-10` |
| 8. Sign-in in tests; WebKit surfaces | `docs/RUNBOOK.md:1078-1083`; `playwright.config.ts:108-154` |
| 8. Tap-target guard and its exemptions | `docs/RUNBOOK.md:1104-1111` |
| 8. Definition of Done | `CLAUDE.md:103-123`; `docs/DECISIONS.md:278-283` |
| 9. Operations | `docs/RUNBOOK.md:285-296`, `:495-564`, `:731-732` |
| 9. Test walks never write production as the owner | `REVIEW.md:31-32` |
| 9. Known limits kept | `docs/RUNBOOK.md:675-742` |

---

## 4. docs/decisions.md

| # | Decision | Source |
|---|----------|--------|
| 1 | No over-engineering (and Dependabot over a CI audit step) | `docs/DECISIONS.md:19-21`; `CLAUDE.md:14-16`; `docs/RUNBOOK.md:711-722` |
| 2 | CI gate and Sentry, 2026-06-16; the 06-14 audit's fixes and cuts; `noindex` layers from that audit; Sentry live 09-20 | `docs/DECISIONS.md:37-78` (`:39-40` for `noindex`); `docs/RUNBOOK.md:788` |
| 3 | Database hardening, 2026-09-20 | `docs/DECISIONS.md:80-125`; `supabase/migrations/20260920100200_storage_read_scope.sql:1-3` |
| 4 | Token-hash sign-in links | `docs/RUNBOOK.md:398-493`; `src/app/auth/confirm/route.ts:1-10` |
| 5 | Human check first, 2026-09-21 | `docs/DECISIONS.md:127-162` |
| 6 | Per-actor mail cap, 2026-09-21 | `docs/DECISIONS.md:164-228` |

---

## 5. Repository Facts Not Taken From Prose

Counts made directly against the source checkout on 2026-10-07:

- 46 migration files (`ls supabase/migrations | wc -l`).
- 25 distinct `create table public.*` statements, none dropped, and 25 distinct `enable row level security` statements.
- 38 integration, 74 unit and 28 end-to-end test files.
- 89 commits on `main`, 82 of them carrying a `(#N)` pull-request reference.
- The portfolio site (`~/Projects/portfolio`) has no On the Lees page, so the README carries no portfolio link yet.

---

## 6. Deliberately Left Out

Kept out of the showcase because they are private, identifying, internal, or would help someone get around a live setting:

- The production domain, the hosting and database project URLs and identifiers, DNS records and IP addresses.
- Every email address in the source.
- The name of the certifying body and course where the project began, its size, and the third-party forum where invites were shared (`docs/DECISIONS.md:12-13`, `docs/RUNBOOK.md:163`).
- Verbatim quotes in `docs/DECISIONS.md:7-8` and `:257-258`.
- Anything a member did, including the moderator incident behind the attachments work (`docs/DECISIONS.md:335-339`) and a member's first name in the runbook (`docs/RUNBOOK.md:500`).
- Where per-field profile privacy and moderator edit logging stand (`docs/DECISIONS.md:111-114`; `docs/RUNBOOK.md:268-271`, `:725-726`): the network has real members, and those details describe live limits.
- Credential locations, 1Password item names, the error-monitoring organization name, and every secret.
