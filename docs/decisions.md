# Decisions: On the Lees

This page summarizes every decision the project records in its own decision log, in date order, followed by three decisions recorded in its operations runbook. The project does not number its decisions, so each section is named by what was decided and dated where the record gives a date.

---

## Part 1: The Decision Log

### 1. Private From Day One, Never Indexable

**Date**: From the first commit, 2026-06-13
**Context**: Findability is the point of a member directory, so members use their real names. Research on real-name harm is about forced or open exposure; a closed, consented, vouched membership is a different case, but only if it stays closed.
**Decision**: The network is private and never publicly indexable. Real-name identity is visible only to verified members, and membership is by web-of-trust vouch through an invite.
**Consequences**:
- `noindex` at three layers (page metadata, `robots.ts`, an `X-Robots-Tag` header), plus a sign-in wall on every member route.
- Later privacy choices (no session replay, no offline cache, scrubbed error events) cite it directly.

### 2. No Over-Engineering, No Feature Additions

**Date**: 2026-06-14 (the owner's standing rule)
**Context**: A small, unproven, non-commercial goodwill project for a peer community.
**Decision**: Ship the defined product properly and fix only real problems that members hit.
**Consequences**:
- Later changes are argued against this rule in the record, including ones that were adopted (decisions 6, 7 and 24).
- The record says the rule is specific to this project and does not generalize.

### 3. Row-Level Security Is the Final Backstop

**Date**: 2026-06-13, restated through every later audit
**Context**: Three layers gate access (the proxy, the verified-member layout, the database), and only the database cannot be bypassed by a forgotten check.
**Decision**: Every public table has members-only policies for the `authenticated` role, and `anon` is denied everywhere. RLS is never weakened to make a feature easier.
**Consequences**:
- A grants migration grants table privileges explicitly, because a database built from scratch otherwise answers "permission denied" before RLS is even consulted.
- `security definer` helpers avoid policy recursion.
- The invite-redeem function is revoked from both client roles and runs only as the service role, so the public key cannot call it.

### 4. One Forum Table, Scoped by Group

**Date**: 2026-06-13
**Context**: Each study group needs its own forum, and the network needs a shared lobby.
**Decision**: One `threads` table with a nullable `group_id` (NULL is the lobby), with reads gated by one helper, `can_read_group`. A group is public, private or hidden; private and hidden read identically and differ only in discovery. Storage buckets are private, served by signed URLs, with EXIF stripped on the device.
**Consequences**:
- Any new group-scoped table must gate its reads through the same helper.
- Visibility and join policy started as separate settings: who can find and read a group is a different question from who can join it. A later fix made private groups request-only (decision 11).

### 5. A Production Audit That Fixed Some Things and Cut Others on Purpose

**Date**: 2026-06-14
**Context**: Before sharing the app, a seven-dimension audit with adversarial verification ran against `main`.
**Decision**: Fix what a member would feel; cut what would not make the app better for a member.
- **Fixed**: the private app was indexable; error and not-found pages; an invite function the public key could call; blocks enforced in RLS on contact data; injection guards on query filters; `server-only` guards; foreign-key indexes; a structural RLS-coverage test; event time zones; honest landing copy; a select control that showed a raw database value; a visual audit of every page at desktop and phone width.
- **Cut**: a per-field privacy rewrite in RLS, list pagination, disposable-email blocking, a captcha, error monitoring and a CI gate.
**Consequences**:
- The audit also recorded two findings as overstated: a "stored XSS" that React 19 already blocks, and an "account takeover" that still required a signed-in session (the real risk, invite griefing, was closed by the revoke in decision 3).
- Several cuts were later reversed: the CI gate and error monitoring (decision 7), the captcha, which is now live and enforced, and per-field privacy for contact fields, now enforced in the database (profile fields stay in app code).

### 6. The Connection-Request Email Completes the Core Loop

**Date**: June 2026 (live in production from 2026-06-15)
**Context**: Reconnect is mutual opt-in, and at the time there were no other notifications. A request nobody hears about never gets accepted.
**Decision**: Send one best-effort transactional email when a connection request arrives, through Resend, env-gated (no key, no send), and server-only where it touches the service role.
**Consequences**:
- Judged in scope under decision 2: it makes the defined product work rather than adding a feature.
- It was the first of what are now eight notification emails, all sent through one path (decision 14).

### 7. Reverse Two Launch-Time Cuts: A CI Gate and Error Monitoring

**Date**: 2026-06-16
**Context**: Once the app was live with a real member, the question shifted from "ship it" to "make it diagnosable and safe to change."
**Decision**:
- **CI and branch protection.** Every pull request runs build, lint, typecheck, the Vitest suite (including RLS proofs against an ephemeral Supabase) and Playwright. `main` requires the build and test checks and, since 2026-07-06, that every review thread is resolved.
- **Sentry, errors first.** Light tracing, no session replay (a real-name network), `sendDefaultPii: false`, and console capture so the existing `console.error` sites become searchable events with no call-site changes. A `beforeSend` scrubber trims exception messages so a database error cannot carry a member's email or phone into an event. No DSN means a silent no-op.
**Consequences**:
- Deleting a thread or post now removes its stored images, which had been orphaned forever.
- Still cut: pagination, disposable-email blocking, a data-request portal, image scanning, uptime monitoring.

### 8. A Definition of Done for Every Pull Request

**Date**: Enforced per pull request
**Decision**: A pull request merges only with a green build, lint, typecheck, Vitest and Playwright; unit tests for new logic; an RLS test proving a non-member cannot read anything newly gated; an end-to-end happy path; a real look at screenshots at 1280 and 390 pixels; the copy lint; and a multi-agent review (code, security, UX and voice) with every critical and high finding cleared.
**Consequences**: The checklist lives in the project guide and the review guide, so a reviewer and an author read the same bar.

### 9. Copy Voice: Title Case Every Word, Peer to Peer

**Date**: 2026-09-19 and 2026-09-20 (owner's rulings)
**Context**: The app was written in sentence case with a "warm, knowledgeable, understated" voice. The owner wanted short labels in his house style and longer text that talks to working adults as peers.
**Decision**:
- Headings, buttons, tabs, labels, navigation and email subjects are Title Case Every Word, short words included ("Add To Home Screen"). The brand keeps its own casing.
- Anything longer than a button is short, direct, said once, in "you". No "we", "us" or "our" (the product is not a person), no reassurance, no idioms, no rhetorical questions, no explaining how email or sign-in works, no em dashes, none of the AI-tell banlist.
- One name per thing: a control has the same label on every screen, and a raw database value is mapped to a word before a person sees it.
**Consequences**:
- This reversed the original convention; the whole source tree was converted on 2026-09-20, and a sentence-case heading is now a defect.
- A unit test lints the whole source tree for the em dash, the banlist and first-person plural, and checks Title Case on every h1 to h3 heading, button label and email subject. The old per-file opt-in list was deleted when the check widened.

### 10. A Network for Sommeliers; Reconnect Is One Feature

**Date**: 2026-09-20 (owner's ruling)
**Context**: The project began among people who had trained together, and "find the people you could not reach" was the launch hook. Until this ruling, reconnecting was how the product described itself.
**Decision**: On the Lees is a private network for sommeliers and wine professionals: directory, forum, study groups, and Reconnect. Reconnect is never the headline, and no screen frames the membership as a course, a class or a cohort.
**Consequences**: The origin stays in the record as history, not positioning.

### 11. Database Hardening: Scope the Writes the Way the Reads Were Scoped

**Date**: 2026-09-20 (from a read-only audit of the migrations on 2026-09-19)
**Context**: The audit found write-side gaps the read policies hid. A row could change the column its read policy scopes on: a private group's thread moved to the lobby, a reply repointed at another group's thread. The parties to a connection could be swapped to forge an accepted one and unlock a contact card. A private group left on "anyone can join" was self-joinable. Any member could mint an unlimited invite that vouches whoever redeems it. Storage released every object in the photo bucket to every member, because listing and signing both consult the read policy. Each was reproduced on a local stack before it was fixed, and each fix ships with an integration RLS proof.
**Decision**, in five judgment calls:
1. **Freeze scope columns with a trigger, not a `WITH CHECK`.** A policy sees only the new row, so it cannot stop a row leaving its group, and mirroring the insert policy would take an author's edit rights away the day they left. Authorship, membership records, a report's reporter and target, a reply's parent and every primary key are frozen.
2. **Grant the predicates a policy names.** A policy is permission-checked as the calling role, so revoking execute across the schema broke ordinary reads. Policy helpers keep an explicit grant to `authenticated`; `anon` keeps none. The two predicates that answer about another member moved to a private schema the API does not expose.
3. **Keep per-field profile privacy in app code for now.** Enforcing it in the database means moving the maskable columns out of `profiles`, which breaks every author embed in the API. What changed is the row level: applicants and suspended members are no longer readable by ordinary members, and the settings copy now says what the app actually does.
4. **Broadcast an invite only when its minter says so.** The Invite page used to show whichever live code was newest, so a small code minted for one guest went to every member. A `shared` flag now decides, and only moderators mint.
5. **Keep the weekly summary's watermark in its own server-only table.** A member can update their own preference row, so a watermark there could be rewritten to repeat or suppress their own summary, and it doubles as the guard that stops a retried cron mailing everyone twice.
**Consequences**: A new network-wide invite must be minted with `shared` on, or the Invite page goes empty.

### 12. Accept Text Typed Before the Page Is Ready

**Date**: 2026-09-20
**Context**: A server-rendered form is visible and typeable seconds before React takes it over. On a slow phone, typed words stayed in the box but never reached the form's state, so Post Thread sat disabled under a filled-in title and the next render wiped it. Five end-to-end tests went red this way in CI.
**Decision**: A small hook reads each controlled field once on mount and adopts any value that render did not put there. Measured on the installed React version, hydration keeps the typed value in the DOM for inputs, text areas and selects. It is on every controlled field of the three forms that appear in a page's first HTML.
**Consequences**: An end-to-end spec holds back the page's JavaScript and types before React arrives; remove the hook and it fails, which keeps it from being mistaken for dead code.

### 13. The Human Check Runs Before Anything About an Invite

**Date**: 2026-09-21
**Context**: The request-access action looked up the invite code, checked its owner could vouch, and wrote a binding, all before calling the Supabase sign-in, which is the only place the Turnstile token is checked. Anyone could guess codes at any rate with no token, and a hit let them sign in, finish onboarding, and be vouched past moderator review into a real-name network.
**Decision**: Send the sign-in email first; read the invite only after Supabase accepted the send. A failed send returns what it returned before, having looked up and written nothing. The answer is identical for a valid code, an invalid one and none.
**Consequences**:
- **The app never calls Cloudflare's verification endpoint.** The token is single use and Supabase Auth redeems it; a check in the app would spend it first and Auth would reject every real sign-in. Order is the fix, not a second check.
- **The signed-in door is capped, not closed.** Applying a code after sign-in measured 878 guesses a minute from one account on a local stack. It now allows five an hour, counted in a server-only table before any code is read, and fails closed.
- The typed 8-digit sign-in code is bounded by Supabase Auth's own per-IP limit and a one-hour expiry, not by the app, and the record notes that the local stack does not reproduce production's limit.

### 14. One Member Cannot Drain the Mail Quota Everyone Signs In With

**Date**: 2026-09-21
**Context**: Sign-in and notification email share one free-tier quota of about 100 a day. Cancel-then-request and remove-then-add loops let one member mail the same person without limit, and once the quota is gone nobody can sign in: a denial of service on the front door, open to any member.
**Decision**:
- **Cap per actor, not globally.** A global cap stops the flood and hands the flooder the outage. Per actor, one member can use a known fraction (40 a day, under half the quota) and the rest of the network is untouched.
- **Push has its own, wider numbers that ignore the subject.** Making a group is free, so a window keyed on the group is "make another group" away from no window at all. A push over the ration is dropped silently with one log line, and a recipient with no registered device claims nothing.
- **The decision and the row are one locked statement.** Read-then-write bounded nothing: ten parallel sends for one pair mailed eight times, and forty parallel reservations were all allowed against a ceiling of twenty. One `security definer` function under a per-actor advisory lock now decides and records. Its prune, the only part that crosses actors, is bounded so it can never become the statement everyone waits on.
- **It lives in the one send path**, which already held the preference, the suspended check and the unsubscribe link.
- **It fails closed** and never fails the member's action.
- **Same actor, recipient, type and subject: once in 15 minutes, five a day.** A whole-day rule was tried first and took away the second reply of an afternoon, which is the traffic the site exists for.
- **The counter is server-only** and prunes itself at 30 days, matching the privacy page, with no cron to schedule or forget.
- **Bulk adds are capped at the hour's allowance**, so an owner never reads "Added 25 members" over people the cap then refused to tell.

### 15. End-to-End Tests Run Against a Production Build

**Date**: 2026-09-21
**Context**: The suite signed in with a dev-only button, so it could only run against the dev server, and CI tested a server nobody deploys. The dev server on a runner lost two routes for a whole run three times, about 29 runner minutes each.
**Decision**: Specs sign in through the real door: the service role mints the one-time token the email would carry, and the test walks the confirm step with it. The dev button keeps two tests of its own: it works under the dev server, and a production build does not offer it.
**Consequences**: Two production-build differences surfaced, both measured before anything was edited:
- `Secure` cookies keyed on the environment were never sent back by WebKit over `http://localhost`, so the phone engine was testing neither sign-in cookie. The flag now follows the configured site URL and fails safe to `Secure`.
- The two sign-in doors share one path with a different query, and prefetching carries no query, so the tab title showed the other door. The fix is Next's documented one: `prefetch` on the four links between the doors.

### 16. Long Posts Fold Behind Read More

**Date**: 2026-09-23
**Context**: Measured on production at 390 pixels, a 241-line schedule thread was 9,320 pixels tall: eleven screens before its first reply.
**Decision**: A body over 15 phone-width lines shows a 12-line preview with Read More (and Show Less), and a folded opening post with replies offers Jump To Replies. The rule reads the text alone, so the server renders the fold in the first HTML; the browser undoes a fold that fits the width it actually has, and measures again when the width changes.
**Consequences**:
- The forum list stays titles only (owner's ruling).
- The fade is a CSS mask, not a colored overlay, because the overlay drew a gray band in WebKit.

### 17. Posts Understand a Small Piece of Markdown

**Date**: 2026-09-23
**Context**: A long study schedule posted as plain text read as a wall on a phone: its week headings looked like every other line. The owner asked for bold headings and an easy-to-read schedule.
**Decision**: Thread bodies and replies render headings, bullet and numbered lists, quotes, bold, italic, and http(s) links. No HTML, and no Markdown library: the parser returns data and React builds every element, so typed markup reaches the page as escaped text, and there is nothing to sanitize and no dependency to patch.
**Consequences**:
- Every construct is found in one pass per line with bounded look-ahead. Hostile bodies at the full length cap parsed in under 6 ms against a 50 ms ceiling in the test, and emphasis nests at most 12 deep, because one crafted body overflowed the render stack.
- A single newline stays a line break, unlike standard Markdown, because every existing post depends on its line breaks. A plain post rendered pixel-identical to the old renderer on three engines.
- Member headings render as h3 to h5 so they never compete with the page's own title.

### 18. Photos and Files on a Forum Post

**Date**: 2026-09-24
**Context**: A moderator tried to share material from a phone and gave up. Walked on production in both phone engines: there was no way to attach a document, a second pick discarded the first, five photos posted four with no word, every failure read "Try again", and a posted photo could not be opened.
**Decision**:
- Up to 4 photos and 4 files per post. Files are PDF, plain text or CSV up to 25 MB, stored byte for byte in a private bucket, one row each. Word and Excel files, HTML, SVG and archives are refused when picked, with "Save it as a PDF". Every file downloads; nothing a member uploads is rendered by a browser as a page.
- A table, not an array column, so the storage read policy is a unique-index lookup, and the four-file cap is a trigger behind an advisory lock.
- Two buttons, Add Photos and Add Files, because the input's `accept` decides which sheet a phone opens; one classifier sits behind both.
- HEIC is converted where the browser can decode it and refused by name where it cannot, decided by trying, never by the user agent. A decoder library (about a megabyte) was cut.
- Progress through `XMLHttpRequest`, one retry, and a named failure reported to Sentry without the file name.
- Edit is one transaction, and deletes sweep their objects as the service role, only inside the deleted post's own author's folder.
**Consequences**: Cut as more than the problem needs: a lightbox library, drag and drop, image editing, video, resumable uploads, per-attachment moderator removal. Open: an iPhone uploads JPEG at about three times the bytes of other engines' WebP, and any change waits on a measurement with real photos.

### 19. Post Bodies May Run to 50,000 Characters

**Date**: 2026-09-24
**Context**: A study schedule with a link for every reading ran to 27,000 characters and had to be split across a post and a reply. The 20,000 cap lived in seven places.
**Decision**: Raise both database check constraints to 50,000 and make one constant the single number the schemas and the composers read, pinned to the migration by a unit test.
**Consequences**: Long bodies fold behind Read More (decision 16), and notification emails carry titles and links, never bodies, so nothing mailed grows.

### 20. Directions at the Top of the Forum

**Date**: 2026-09-24, superseded in part by decision 21
**Context**: A new member's directions lived in a welcome thread, which sank under newer posts and still carried an expired access code.
**Decision**: A server-rendered Start Here block at the top of the forum, each line a link to the place it names, with no dismiss control. On a phone it is one row that opens in place, because the forum is the installed app's start page. The install line hides inside the installed app; the notifications line does not, because on an iPhone that is the only place notifications can be turned on.
**Consequences**: The record's plan is to delete the welcome thread once the block is live.

### 21. Forum Index, Desktop Layout

**Date**: 2026-09-25
**Context**: The owner called the desktop forum sloppy. Measured at 1280 pixels, Start Here was 222 pixels tall, the first thread began 435 pixels down, and each row clustered its details on the left with the right 60% empty.
**Decision**:
- Start Here becomes one line of links in the subtitle's place (two lines below 1024 pixels, broken by hand so one entry never wraps alone).
- One toolbar row: category filter on the left, New Thread on the right. Auto-submitting the filter on change was built and dropped in review: an arrow key on a closed select fires a change event on Windows and Linux, which fails WCAG 3.2.2, and it only worked with JavaScript.
- Columns from 640 pixels: vote, thread, replies, activity, on one grid shared with a header row. Activity shows the exact time on hover, in UTC and labeled so, because the server does not know the reader's zone. The header row is hidden from screen readers and each number carries its own name instead.
**Consequences**: The first thread now starts at 231 pixels on a desktop (from 435) and 236 on an iPhone (from 343).

---

## Part 2: Decisions Recorded in the Runbook

### 22. Sign-In Links by Token Hash, With a Typed Code

**Date**: 2026-09-20
**Context**: The default magic link only works in the browser that asked for it, but a phone opens the link inside the mail app's own browser. In production, 11 of 19 sign-in links that never completed had been tapped within seconds.
**Decision**: The email templates link by token hash, redeemed by `/auth/confirm` with no cookie needed, and carry an 8-digit code that works in any browser. The confirm GET does not spend the token, because mail scanners and link previews issue it; a button press on the next step does. The token moves into a 10-minute `httpOnly` cookie so it never reaches an access log, a referrer or Sentry.
**Consequences**: The code length stays at 8, since anyone can request a code for any address and 6 digits would cut the guessing work a hundredfold. Old-style links in inboxes keep working.

### 23. A Service Worker With No Cache

**Context**: Push notifications need a service worker. Phones get shared, and this is a private, real-name network.
**Decision**: The service worker handles push and notification clicks and nothing else: no fetch handler, no cache. It registers only when a member turns push on.
**Consequences**: Member pages never stay in a browser cache after sign-out. The runbook records that adding caching later is a privacy decision, not a performance one.

### 24. Dependency Alerts at the Repository, Not an Audit Step in CI

**Context**: Dependency vulnerabilities need watching, and the obvious move is an `npm audit` step in CI.
**Decision**: The step was kept out as over-engineering under decision 2. Dependabot alerts and automated security fixes were switched on for the repository instead: the same signal with no CI weight.
**Consequences**: No workflow mentions `npm audit`, so the runbook records the decision explicitly; reading the workflows alone would wrongly suggest there is no mechanism at all.
