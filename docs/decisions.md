# Decisions: On the Lees

This page covers the six decisions in the README's table, in date order, from the project's decision log and operations runbook. The project does not number its decisions; the numbers here are for this page only.

---

### 1. No Over-Engineering, No Feature Additions

**Date**: 2026-06-14 (my standing rule)
**Context**: A small, unproven, non-commercial goodwill project for a peer community.
**Decision**: Ship the defined product properly and fix only real problems that members hit.
**Consequences**:
- Later changes are argued against this rule in the record, including the two that were adopted in decision 2. An `npm audit` step in CI was kept out under it; Dependabot alerts at the repository give the same signal with no CI weight.
- The record says the rule is specific to this project and does not generalize.

### 2. Reverse Two Launch-Time Cuts: A CI Gate and Error Monitoring

**Date**: 2026-06-16
**Context**: Before the app was shared, a seven-dimension audit with adversarial verification fixed what a member would feel (among other things, the private app was indexable, and the three `noindex` layers date from that audit) and cut what would not make the app better for a member. Among the cuts were list pagination, disposable-email blocking, a captcha, error monitoring and a CI gate. Once the app was live with a real member, the question shifted from "ship it" to "make it diagnosable and safe to change."
**Decision**:
- **CI and branch protection.** Every pull request runs build, lint, typecheck, the Vitest suite (including RLS proofs against an ephemeral Supabase) and Playwright. `main` requires the build and test checks and, since 2026-07-06, that every review thread is resolved.
- **Sentry, errors first.** Light tracing, no session replay (a real-name network), `sendDefaultPii: false`, and console capture, so the existing `console.error` sites became searchable events with no call-site changes. A `beforeSend` scrubber trims exception messages so a database error cannot carry a member's email or phone into an event.

**Consequences**:
- Sentry has reported from production since 2026-09-20.
- Other cuts were later reversed too: the captcha, which is now live and enforced, and per-field privacy for contact fields, now enforced in the database.
- Still cut: list pagination and disposable-email blocking, among others.

### 3. Database Hardening: Scope the Writes the Way the Reads Were Scoped

**Date**: 2026-09-20 (from a read-only audit of the migrations on 2026-09-19)
**Context**: The audit found write-side gaps the read policies hid. A row could change the column its read policy scopes on: a private group's thread moved to the lobby, a reply repointed at another group's thread. The parties to a connection could be swapped to forge an accepted one and reveal a contact card. Storage reads were scoped to the bucket rather than the post, and listing and signing both consult the read policy. Each was reproduced on a local stack before it was fixed, and each fix ships with an integration RLS proof.
**Decision**:
1. **Freeze scope columns with a trigger, not a `WITH CHECK`.** A policy sees only the new row, so it cannot stop a row leaving its group, and mirroring the insert policy would take an author's edit rights away the day they left. Authorship, membership records, a report's reporter and target, a reply's parent and every primary key are frozen.
2. **Grant the predicates a policy names.** A policy is permission-checked as the calling role, so revoking execute across the schema broke ordinary reads. Policy helpers keep an explicit grant to `authenticated`; `anon` keeps none. The two predicates that answer about another member moved to a private schema the API does not expose.
3. **Hide applicants and suspended members.** Applicants and suspended members are no longer readable by ordinary members, and the settings copy now says what the app actually does.
4. **Broadcast an invite only when its minter says so.** The Invite page used to show whichever live code was newest, so a code minted for one guest went to every member. A `shared` flag now decides, and only moderators mint.
5. **Keep the weekly summary's watermark in its own server-only table.** A member can update their own preference row, so a watermark there could be rewritten to repeat or suppress their own summary.

**Consequences**: Storage reads now follow the post that references the object. A new network-wide invite must be minted with `shared` on, or the Invite page goes empty.

### 4. Sign-In Links by Token Hash, With a Typed Code

**Date**: 2026-09-20 (recorded in the runbook)
**Context**: The default magic link works only in the browser that asked for it, but a phone opens the link inside the mail app's own browser, so links tapped within seconds still failed.
**Decision**: The email templates link by token hash, redeemed by `/auth/confirm` with no cookie needed, and carry an 8-digit code that works in any browser. The confirm GET does not spend the token, because mail scanners and link previews issue it; a button press on the next step does. The token moves into a 10-minute `httpOnly` cookie so it never reaches an access log, a referrer or Sentry.
**Consequences**: The code length stays at 8, since anyone can request a code for any address and 6 digits would cut the guessing work a hundredfold. Old-style links in inboxes keep working.

### 5. The Human Check Runs Before Anything About an Invite

**Date**: 2026-09-21
**Context**: The request-access action looked up the invite code, checked its owner could vouch, and wrote a binding, all before calling the Supabase sign-in, which is the only place the Turnstile token is checked. Anyone could guess codes at any rate with no token, and a hit let them sign in, finish onboarding, and be vouched past moderator review.
**Decision**: Send the sign-in email first; read the invite only after Supabase accepted the send. A failed send looks up and writes nothing. The answer is identical for a valid code, an invalid one and none.
**Consequences**:
- **The app never calls Cloudflare's verification endpoint.** The token is single use and Supabase Auth redeems it; a check in the app would spend it first and Auth would reject every real sign-in. Order is the fix, not a second check.
- **The signed-in door is capped, not closed.** Applying a code after sign-in had no limit of its own. It is now limited per account by the hour, counted in a server-only table before any code is read, and it fails closed.

### 6. One Member Cannot Drain the Mail Quota Everyone Signs In With

**Date**: 2026-09-21
**Context**: Sign-in and notification email share one daily sending quota. Cancel-then-request and remove-then-add loops let one member mail the same person without limit, and once the quota is gone nobody can sign in: a denial of service on the front door, open to any member.
**Decision**:
- **Cap per actor, not globally.** A global cap stops the flood and hands the flooder the outage. Per actor, one member can use a known fraction (under half the daily quota) and the rest of the network is untouched.
- **Push has its own, wider numbers that ignore the subject.** Making a group is free, so a window keyed on the group is "make another group" away from no window at all.
- **The decision and the row are one locked statement.** Read-then-write bounded nothing: most of a burst of parallel sends to one pair went out, and every reservation in a parallel burst was allowed past the hourly ceiling. One `security definer` function under a per-actor advisory lock now decides and records, and its prune, the only part that crosses actors, is bounded.
- **It fails closed** and never fails the member's action.
- **Same actor, recipient, type and subject: spaced by minutes, capped by the day.** A whole-day rule was tried first and took away the second reply of an afternoon, which is the traffic the site exists for.

**Consequences**: The counter is server-only and prunes itself at 30 days, matching the privacy page. Bulk adds are capped at the hour's allowance, so an owner is never told members were added when the cap then refused to tell them.
