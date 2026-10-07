# SOURCES (review only: delete this file before publishing)

This file is for review. It maps every factual claim in this showcase to the place in the private source repository that supports it, and it must be deleted before the showcase is published. Paths are relative to the source checkout (`~/Projects/somm-cohort`, repository `AIJSAI/on-the-lees`, read at commit `4f26bfc`, 2026-09-25). Line numbers are as read on 2026-10-07.

---

## 1. The Status Statement, and a Conflict Between Two Files

**Stated in the README:** live in production since June 2026 with real members; first commit 2026-06-13; serving on its own domain 2026-06-15; 14 members on 2026-09-21 and 22 by 2026-09-24; three moderators; non-commercial goodwill project.

| Claim | Source |
|-------|--------|
| Live since June 2026 with real members; deploy wiring long done | `docs/RUNBOOK.md:23-24` ("The app has been live since June 2026 with real members; the deploy wiring this section used to call the remaining work is long done.") |
| Domain live 2026-06-15 | `docs/RUNBOOK.md:85` (section heading "Current status (2026-06-15): LIVE"), `:92-94` |
| First commit 2026-06-13 | `git log --reverse` (first commit "Initial commit from Create Next App", 2026-06-13) |
| Live with a real member by 2026-06-16 | `docs/DECISIONS.md:58` |
| 14 members on 2026-09-21 | `docs/RUNBOOK.md:617` (push ration table, section dated 2026-09-21: "at 14 members") |
| 22 members by 2026-09-24 | `docs/DECISIONS.md:410` ("with 22 members by 2026-09-24") |
| Three moderators (owner plus two co-moderators) as of 2026-09-20 | `docs/RUNBOOK.md:118-119` |
| Goodwill, non-commercial, unproven | `docs/DECISIONS.md:19`, `docs/RUNBOOK.md:17` |

**Conflict to know about:** `CLAUDE.md:12` still says "Code is feature-complete and merged; what remains is deploy wiring." That line is stale. `docs/RUNBOOK.md:23-24` says so explicitly, and `docs/DECISIONS.md` records production measurements from June through September. The showcase follows the runbook.

**Deploy status of the late-September features:** photos and files (#86), the Markdown subset (#85), long-post folding (#84), the 50,000-character cap (#87), Start Here (#88) and the forum index layout (#91) merged on 2026-09-24 and 2026-09-25 (`git log`). The repository does not record their production deploy (`docs/RUNBOOK.md:689-695` treats the attachment walk as "the first thing to check after the deploy"; `:1063-1064` says to post formatted text only once the renderer is deployed). The README therefore marks them "Merged", not "Live".

**Weekly summary:** `docs/RUNBOOK.md:74-76` (last updated 2026-09-23, `:8`) lists `CRON_SECRET` as still to set; until then the route refuses every request. The README marks it "Built".

---

## 2. README.md

| Claim | Source |
|-------|--------|
| Product: directory, forum, study groups with events, Reconnect; Reconnect is one feature | `CLAUDE.md:3-7`; `docs/DECISIONS.md:7-11` |
| Origin: people who trained together, scattered across private chats | `docs/DECISIONS.md:12-15` (generalized; see section 6) |
| Private, never indexable, real-name, vouch by invite | `docs/DECISIONS.md:16-18` |
| `noindex` at three layers | `docs/DECISIONS.md:40`; `src/app/layout.tsx:23-25`; `src/app/robots.ts:5-9`; `next.config.ts:21` |
| Invite code shared by members auto-vouches; moderators approve requests | `supabase/migrations/20260616140000_vouch_is_moderator_only.sql:1-14`; `docs/RUNBOOK.md:188-200` |
| 44px controls on a phone; installable app | `REVIEW.md:20`; `src/app/manifest.ts`; `src/app/app/page.tsx` |
| Three gates (proxy, authed layout, RLS) and public routes | `CLAUDE.md:65-73`; `src/proxy.ts:7-26`; `src/app/(authed)/layout.tsx:37-39` |
| `anon` holds no table grant | `supabase/migrations/20260921100000_anon_holds_no_table_grants.sql`; `docs/RUNBOOK.md:677-687` |
| `can_read_group`, `visible_contacts()`, private schema for helpers about another member | `CLAUDE.md:78-82`; `docs/RUNBOOK.md:723-726`; `docs/DECISIONS.md:106-110` |
| Private Storage, own-folder uploads, signed URLs, EXIF stripped on device | `CLAUDE.md:84-88`; `docs/RUNBOOK.md:315-321` |
| `deliver()` is the one send path with preference, suspended check, cap, unsubscribe | `docs/DECISIONS.md:201-203`; `docs/RUNBOOK.md:606-609`, `:836-837`; `src/lib/notify-delivery.ts:110-129` |
| Diagram: `/files/[id]` answers 303 to a 60-second signed URL | `docs/RUNBOOK.md:321`; `src/app/(authed)/files/[id]/route.ts` |
| Diagram: weekly cron with bearer secret | `vercel.json`; `docs/RUNBOOK.md:857-863` |
| Diagram: Turnstile redeemed by Supabase Auth; Auth mail through Resend SMTP | `docs/RUNBOOK.md:748-765`, `:100-104` |
| Diagram: browser uploads straight to Storage | `src/lib/upload-attachment.ts:8-10`, `:73-78` |
| Stack versions (Next 16.3.5, React 19.2.4, Zod 4, Tailwind 4, web-push, Sentry) | `package.json` |
| Server Components by default; client only for forms, uploads, filters | `CLAUDE.md:91` |
| Zod schemas in one module | `CLAUDE.md:92` |
| Design "Editorial Utilitarian", Lora and Inter, one garnet accent | `CLAUDE.md:94-96`; `docs/design-brief/README.md` (section 2) |
| Passwordless link or 8-digit code | `docs/RUNBOOK.md:406-411`, `:422-426` |
| Eight notification emails; access-approved has no switch or unsubscribe | `docs/RUNBOOK.md:696-702`, `:854-855` |
| Service worker handles push and notification clicks only, no cache | `docs/RUNBOOK.md:1034-1039` |
| Sentry posture | `docs/DECISIONS.md:66-71`; `docs/RUNBOOK.md:788-805` |
| Manual deploys from the CLI; rollback by promotion | `docs/RUNBOOK.md:288-293` |
| e2e on Chromium; twelve surfaces again on iPhone WebKit | `playwright.config.ts:108-154` |
| CI jobs; `main` protection | `.github/workflows/ci.yml`; `docs/RUNBOOK.md:298-305` |
| Challenge 1 (reads scoped, writes not; freeze triggers; storage read scope) | `docs/DECISIONS.md:80-105`; `supabase/migrations/20260920100200_storage_read_scope.sql` |
| Challenge 2 (invite oracle; order fix; never call siteverify; 878 guesses a minute; five an hour) | `docs/DECISIONS.md:127-156`; `docs/RUNBOOK.md:758-776` |
| Challenge 3 (mail quota DoS; per-actor caps; 8 of 10 and 40 of 40 measurements; advisory lock) | `docs/DECISIONS.md:164-228`; `docs/RUNBOOK.md:589-644` |
| Key decision: token-hash sign-in links; 11 of 19 | `docs/RUNBOOK.md:406-411` |
| Other key decisions | see the decisions table in section 4 below |
| Learned: 18 original tables, anon grants for three months | `docs/RUNBOOK.md:677-687` |
| Learned: production-build differences (Secure cookie, tab title) | `docs/DECISIONS.md:297-309` |
| Learned: audit cut Sentry and CI on 06-14, adopted 06-16 | `docs/DECISIONS.md:45-49`, `:56-65` |
| Learned: 229 of 463 controls under 44px | `docs/RUNBOOK.md:1105-1112`; `playwright.config.ts:133` |
| Learned: pixel-identical render on three engines | `docs/DECISIONS.md:499-502` |
| 46 migrations | `ls supabase/migrations` (46 files) |
| 25 tables, RLS enabled on all 25 | `grep 'create table' supabase/migrations/*.sql` (25 distinct tables); `grep 'enable row level security'` (25 distinct tables) |
| 707 Vitest and 118 Playwright (117 passed, 1 skipped by design) green 2026-09-21 | `.github/workflows/ci.yml:65-66` |
| 38 integration test files | `ls tests/integration/*.test.ts` (38) |
| 82 pull requests merged, 2026-06-13 to 2026-09-25 | `git log --format=%s` on `main`: 82 distinct `(#N)` squash-merge references in 89 commits |
| Self-serve export and deletion; group handoff; storage sweep | `docs/RUNBOOK.md:232-236`; `src/lib/account.ts:158-204` |
| Append-only moderator action log | `docs/RUNBOOK.md:261-271` |
| Copy rules enforced by a test over the whole source tree | `docs/DECISIONS.md:264-269`; `tests/unit/voice-lint.test.ts` |
| Status table: Turnstile live and enforced | `docs/DECISIONS.md:76-77`; `docs/RUNBOOK.md:748` |
| Status table: Sentry live since 2026-09-20; inbound mail live since 2026-09-20 | `docs/RUNBOOK.md:788`, `:809` |
| Status table: push keys set in production 2026-09-20 | `docs/RUNBOOK.md:67-72`, `:948-953` |
| Status table: contact fields masked per member's setting | `src/lib/validation.ts:375-389`; `docs/RUNBOOK.md:723-726` |

---

## 3. docs/architecture.md

| Section | Source |
|---------|--------|
| 1. Three gates, `next` parameter, `getUser()` rule | `src/proxy.ts:7-26`; `src/app/(authed)/layout.tsx:37-39`; `CLAUDE.md:65-76`; `REVIEW.md:29-30` |
| 1. Response headers | `next.config.ts:5-31` |
| 2. Join sequence and order of operations | `docs/DECISIONS.md:127-147`; `src/app/join/actions.ts:136-150`; `src/app/onboarding/actions.ts:77`; `src/lib/invites.ts:108-113` |
| 2. Confirm GET does not spend the token; continue step does | `src/app/auth/confirm/route.ts:1-10`; `src/app/auth/continue/actions.ts:33`; `docs/RUNBOOK.md:468-473` |
| 2. Identical answer for any code | `docs/DECISIONS.md:140-142` |
| 2. Invites: moderator or service role mints, capped, `active_invite()`, `shared` | `docs/RUNBOOK.md:177-200`; `docs/DECISIONS.md:115-120` |
| 2. Vouching is a moderator action | `supabase/migrations/20260616140000_vouch_is_moderator_only.sql:1-20` |
| 2. Code length stays at 8 | `docs/RUNBOOK.md:422-426` |
| 3. Tables and relationships | `supabase/migrations/*.sql` (`create table` and `references` clauses); `supabase/migrations/20260613160200_group_visibility.sql:22-26` |
| 3. Server-only tables | `docs/DECISIONS.md:121-125`, `:153-155`, `:214-217` |
| 3. Former member; heir is the longest-standing member | `docs/RUNBOOK.md:241-243`, `:258-259`; `src/lib/account.ts:176-184` |
| 4. Techniques table | `docs/DECISIONS.md:29-33`, `:94-110` |
| 4. Contact visibility levels | `src/lib/validation.ts:375-389`; `src/lib/reconnect.ts:1-10` |
| 4. Moderator action log and last-active function | `docs/RUNBOOK.md:261-281` |
| 5. Photos and files table and bullets | `docs/RUNBOOK.md:311-336`; `docs/DECISIONS.md:333-391` |
| 6. Notification flow, limits table, bullets | `docs/RUNBOOK.md:597-671`, `:834-946`, `:962-969`; `docs/DECISIONS.md:164-228` |
| 7. Sentry, console capture, service worker | `docs/DECISIONS.md:66-71`; `docs/RUNBOOK.md:788-805`, `:1034-1039` |
| 8. CI jobs, branch protection, e2e signal only | `.github/workflows/ci.yml`; `docs/RUNBOOK.md:298-305`; `docs/DECISIONS.md:61-65` |
| 8. RLS-coverage structural test | `tests/unit/rls-coverage.test.ts:1-10` |
| 8. Sign-in in tests; WebKit surfaces | `docs/RUNBOOK.md:1078-1083`; `playwright.config.ts:108-154` |
| 8. Tap-target guard; typed-before-ready spec | `docs/RUNBOOK.md:1105-1112`; `docs/DECISIONS.md:241-243` |
| 8. Definition of Done | `CLAUDE.md:103-123`; `docs/DECISIONS.md:278-283` |
| 8. Trimmed Supabase services in CI; concurrency | `.github/workflows/ci.yml:12-14`, `:59-66` |
| 8. Dependabot instead of an audit step | `docs/RUNBOOK.md:711-722` |
| 9. Operations | `docs/RUNBOOK.md:285-296`, `:338-342`, `:495-564`, `:731-732`; `REVIEW.md:27`, `:31-32` |
| 10. Known limits | `docs/RUNBOOK.md:675-742`; `docs/DECISIONS.md:74-75`, `:111-114` |

---

## 4. docs/decisions.md

Every section in Part 1 maps to a heading or bullet in `docs/DECISIONS.md`. Part 2 maps to `docs/RUNBOOK.md`.

| # | Decision | Source |
|---|----------|--------|
| 1 | Private from day one | `docs/DECISIONS.md:16-18` |
| 2 | No over-engineering | `docs/DECISIONS.md:19-21`; `CLAUDE.md:14-16` |
| 3 | RLS is the final backstop | `docs/DECISIONS.md:29-33` |
| 4 | One forum table | `docs/DECISIONS.md:34-35`; `supabase/migrations/20260613160200_group_visibility.sql:1-26` |
| 5 | Production audit, 2026-06-14 | `docs/DECISIONS.md:37-54`, `:76-78` |
| 6 | Connection-request email | `docs/DECISIONS.md:271-276`; `docs/RUNBOOK.md:64-65` |
| 7 | CI gate and Sentry, 2026-06-16 | `docs/DECISIONS.md:56-78` |
| 8 | Definition of Done | `docs/DECISIONS.md:278-283`; `CLAUDE.md:103-123` |
| 9 | Copy voice | `docs/DECISIONS.md:245-269` |
| 10 | A network for sommeliers | `docs/DECISIONS.md:7-15` |
| 11 | Database hardening, 2026-09-20 | `docs/DECISIONS.md:80-125` |
| 12 | Text typed before ready | `docs/DECISIONS.md:230-243` |
| 13 | Human check first, 2026-09-21 | `docs/DECISIONS.md:127-162` |
| 14 | Per-actor mail cap, 2026-09-21 | `docs/DECISIONS.md:164-228` |
| 15 | e2e on a production build | `docs/DECISIONS.md:285-315` |
| 16 | Long posts fold | `docs/DECISIONS.md:317-331` |
| 17 | Markdown subset | `docs/DECISIONS.md:461-528` |
| 18 | Photos and files | `docs/DECISIONS.md:333-391`; `playwright.config.ts:137-139` (the moderator's attempt was from an iPhone) |
| 19 | 50,000-character bodies | `docs/DECISIONS.md:393-403` |
| 20 | Start Here directions | `docs/DECISIONS.md:405-420` |
| 21 | Forum index layout | `docs/DECISIONS.md:422-459` |
| 22 | Token-hash sign-in links | `docs/RUNBOOK.md:398-493`; `src/app/auth/confirm/route.ts:1-10` |
| 23 | Service worker with no cache | `docs/RUNBOOK.md:1034-1039`; `docs/DECISIONS.md:312-313` |
| 24 | Dependabot over a CI audit step | `docs/RUNBOOK.md:711-722` |

---

## 5. Repository Facts Not Taken From Prose

These are counts made directly against the source checkout on 2026-10-07, not quoted from a document:

- 46 migration files (`ls supabase/migrations | wc -l`).
- 25 distinct `create table public.*` statements, none dropped, and 25 distinct `enable row level security` statements.
- 38 integration, 74 unit and 28 end-to-end test files.
- 89 commits on `main`, 82 of them carrying a `(#N)` pull-request reference.
- The portfolio site (`~/Projects/portfolio`) has no On the Lees page, so the README links only to the site root.

---

## 6. Deliberately Left Out

Kept out of the showcase because they are private, identifying, or internal:

- The production domain, the hosting and database project URLs and identifiers, DNS records and IP addresses.
- Every email address in the source (the owner's, the seeded dev account, the inbound aliases).
- The name of the certifying body and course where the project began, its size, and the third-party forum where invites were shared (`docs/DECISIONS.md:12-13`, `docs/RUNBOOK.md:163`). The README says only that it began with "a large group of wine professionals who had trained together."
- The owner's verbatim quotes in `docs/DECISIONS.md:7-8` and `:257-258`, which name the course or carry informal spelling.
- The moderator incident's time stamp and thread subject (`docs/DECISIONS.md:335-337`), and a member's first name in the runbook's stuck-sign-up note (`docs/RUNBOOK.md:500`).
- Credential locations, 1Password item names, the error-monitoring organization name, and every secret.
