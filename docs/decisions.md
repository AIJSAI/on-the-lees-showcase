# Decisions: On the Lees

This page covers the six decisions in the README's table, in date order, from the project's decision log and operations runbook. The project does not number its decisions; the numbers here are for this page only.

---

### 1. Build the Defined Product, and Change It Only for Real Problems

**Date**: 2026-06-14 (my standing rule)
**Context**: A private network for a peer community, which I build and maintain.
**Decision**: Ship the defined product and fix only real problems that affect members.
**Consequences**:
- Later changes are argued against this rule in the record, including the two that were adopted in decision 2. An `npm audit` step in CI was kept out under it; Dependabot alerts at the repository give the same signal with no CI weight.
- The record says the rule is specific to this project and does not generalize.

### 2. Reverse Two Launch-Time Cuts: A CI Gate and Error Monitoring

**Date**: 2026-06-16
**Context**: Before the app was shared, a seven-dimension audit with adversarial verification fixed what a member would feel (among other things, the private app was indexable, and its `noindex` settings date from that audit) and cut what would not make the app better for a member. Among the cuts were list pagination, disposable-email blocking, a captcha, error monitoring and a CI gate. Once the app was live, the question shifted from "ship it" to "make it diagnosable and safe to change."
**Decision**:
- **CI and branch protection.** Every pull request runs build, lint, typecheck, the Vitest suite (including RLS tests against an ephemeral Supabase) and Playwright. `main` requires the build and test checks and, since 2026-07-06, that every review thread is resolved.
- **Sentry, errors first.** Light tracing, no session replay (a real-name network), `sendDefaultPii: false`, and console capture, so the existing `console.error` sites became searchable events with no call-site changes. A `beforeSend` scrubber drops the console argument echo and keeps only the first line of each exception message, because Supabase errors carry the offending value, such as a member's email or phone, in their details.

**Consequences**:
- The Sentry code shipped on 2026-06-16 and stays silent until its key is set. The key went into production, and Sentry went live, on 2026-09-20.
- Other cuts were later reversed too: the captcha, which is now live and enforced, and per-field privacy for contact fields, now enforced in the database.
- Still cut: list pagination and disposable-email blocking, among others.

### 3. Database Hardening: Scope the Writes the Way the Reads Were Scoped

**Date**: 2026-09-20 (from a read-only audit of the migrations on 2026-09-19)
**Context**: The audit read the write side, which the read policies did not cover. Most of its findings came down to one thing: reads were scoped, writes were not, so a row could change the very column its read policy scopes on. Each finding was reproduced on a local stack before it was fixed, and each fix ships with an integration RLS test.
**Decision**:
1. **Freeze scope columns with a trigger, not a `WITH CHECK`.** A `WITH CHECK` sees only the new row and cannot compare it with the old one, so it cannot stop a row leaving its group, and mirroring the insert policy would take an author's edit rights away the day they left. Authorship, membership records, a report's reporter and target, a reply's parent and every primary key are frozen.
2. **Grant the predicates a policy names.** A policy is permission-checked as the calling role, so revoking execute across the schema broke ordinary reads. Policy helpers keep an explicit grant to `authenticated`; `anon` keeps none. The two predicates that answer about another member moved to a private schema the API does not expose.
3. **Applicants and suspended members are not readable by ordinary members**, and the settings copy says exactly what the app does.
4. **Broadcast an invite only when its minter says so.** A code minted for one guest should reach that guest, not every member, so the Invite page shows only a code its minter marked `shared`, and only moderators mint.
5. **Keep the weekly summary's watermark in its own server-only table.** A member can update their own preference row, so a watermark there could be rewritten to repeat or suppress their own summary.

**Consequences**: Storage reads now follow the post that references the object. A new network-wide invite must be minted with `shared` on, or the Invite page goes empty.

### 4. Sign-In Links by Token Hash, With a Typed Code

**Date**: 2026-09-20 (recorded in the runbook)
**Context**: The default magic link works only in the browser that asked for it, but a phone opens the link inside the mail app's own browser, so links tapped within seconds still failed.
**Decision**: The email templates link by token hash, redeemed by `/auth/confirm` with no cookie needed, and carry an 8-digit code that works in any browser. The confirm GET does not spend the token, because mail scanners and link previews issue it; a button press on the next step does. On that first request the token moves out of the URL into a 10-minute `httpOnly` cookie, so no later request's log line, the next page's referrer or a Sentry event carries it.
**Consequences**: The code length stays at 8, since anyone can request a code for any address and 6 digits would cut the guessing work a hundredfold. Old-style links in inboxes keep working.

### 5. The Human Check Runs Before Anything About an Invite

**Date**: 2026-09-21
**Context**: The request-access action used to look up the invite code, check that the code's creator could still vouch, and record a binding, all before calling Supabase's sign-in, which is the only place the Turnstile token is checked. Invite lookups therefore ran without a human check.
**Decision**: Send the sign-in email first; read the invite only after Supabase accepted the send. A failed send looks up and writes nothing. The answer is identical for a valid code, an invalid one and none.
**Consequences**:
- **The app never calls Cloudflare's verification endpoint.** The token is single use and Supabase Auth redeems it; a check in the app would spend it first and Auth would reject every real sign-in. Changing the order closes the gap, so no second check is needed.
- **Applying a code after sign-in is limited, not removed.** It is limited per account by the hour, counted in a server-only table before any code is read, and it fails closed.

### 6. Per-Member Email Limits, Decided in One Locked Statement

**Date**: 2026-09-21
**Context**: Several member actions email another member, and two of them can be repeated on demand: cancelling and re-sending a connection request, and removing and re-adding someone to a group. Each repeat sent another email to the same person.
**Decision**:
- **Limit per member, not globally.** A network-wide cap would stop one member's flood by silencing everyone else; a per-member limit leaves the rest of the network untouched.
- **Push has its own, wider limits that ignore the subject.** Making a group is free, so a window keyed on the group is "make another group" away from no window at all.
- **The decision and the record are one locked statement.** The first version read the count and then wrote it, and a parallel burst got past the hourly ceiling. One `security definer` function under a per-member advisory lock now decides and records, and its prune, the only part that crosses members, is bounded.
- **It fails closed:** a counter that cannot be read skips the email, and the member's own action still succeeds.
- **Same sender, recipient, type and subject: spaced by minutes, capped by the day.** A whole-day rule was tried first and took away the second reply of an afternoon, which is the traffic the site exists for.

**Consequences**: The counter is server-only and prunes itself at 30 days, matching the privacy page. Bulk adds are capped at the hour's allowance, so an owner is never told members were added when the cap then refused to tell them.
