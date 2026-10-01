# Trinity — a governed agent that keeps a ledger of its own errors

*By Mike Umesumbu, who designed Trinity and operates it. Contact: Umesumbumike@gmail.com*

*Published 2026-10-01 from the ledger as it stood at 02:00 UTC on 2026-09-28. Every number below names where it was
read. Nothing here is invented, and where a number is small, it says so. No code, schema internals, secrets, or
contact details from the private system appear here.*

---

## What Trinity is

Trinity is a small, private orchestration system I designed and operate over AI tools. It runs on a schedule, does a
narrow commercial job, and does one unusual thing: **every consequential action is a recorded bet, graded later
against what actually happened, and its mistakes — including the mistakes of the people and models that built it —
are numbered rows on a ledger it cannot edit after the fact.**

Its job today is to measure public data and public code, find real defects, and give the measurement away to a
named person who could use it. Nothing it produces reaches the world without a human hand: sends, spends, publishing,
credential provisioning, migrations, and changes to its own constitution are on a written never-automated list, and
most of that list is enforced by validators, not by prose.

## The real stack

- **Backend:** Node.js / Express (CommonJS) on Railway. Deploys automatically from `main`; a `/health` endpoint
  reports the running commit, and a deploy is confirmed by reading it, never by a green merge.
- **Database:** Supabase Postgres. A governance schema isolated from the orchestration tables; sixty migration files,
  every one idempotent and reversible by a commented block; row-level security on every governance table; triggers
  that make bets immutable once minted, belief updates append-only, and the two-seat memo channel unforgeable (below).
- **Frontend:** Next.js (App Router) on Vercel — an operator console with a Rulings Room, a Decisions Room, and a
  daily Line the operator reads on his phone.
- **Model runtime:** Anthropic's API for the agent turns, behind per-run token ceilings and a daily spend envelope
  that stop a session before the spend, not after it.

## How it was built

I designed the system and operate it. The code was implemented with AI coding agents under my direction — a
**Builder** seat that writes branches, tests first, and reports what it built, and a **Lead** seat that reads every
pushed hash, rules on it, and applies the migration files by hand. The two seats talk through a machine channel where
every memo is a whole row with its own hash, so neither seat can act on a truncated paste. I hold every irreversible
act: I register capabilities under my own name, I apply nothing, and I post by hand.

The build is small, reviewable diffs: one task per branch, red-green tests, a full suite (about 2,000 tests) green
before any merge, and a written handoff updated at every session close so a fresh session starts from the record, not
from memory.

## The canon: failures that became rules

Trinity keeps a **canon index** — one line per rule, each pointing at the memo that recorded the failure it came
from. The memo is the authority; the line is the reminder. A sample, with the failure first:

| What broke | The rule it earned |
|---|---|
| A module was written, tested, and never called; it read as "built" for a week (B-6, B-7, B-9). | **"Built" includes "called by X at Y."** Nothing reports as built without naming its live invocation site; otherwise the words are *written, not yet called*. |
| A public finding was drafted from one code-reading method and was wrong (B-29, below). | **One method is a claim; two independent methods agreeing is a finding.** |
| A budget ceiling stopped the session *after* the spend it was meant to prevent (B-21). | **A ceiling that stops the session after the spend is a receipt, not a control.** |
| A feature commit shipped its own navigation link, so the surface appeared with the merge, before the operator's word (B-13). | **A nav link is a separate, gated commit.** |
| A revocation flipped a flag but could not prove it had revoked (B-24). | **A revocation that cannot prove it revoked has not revoked.** |
| Three seeded model instances agreed with each other and were all wrong the same way. | **Agreement is not a second method; a private answer key is.** |
| A record of a file outlived the file it described and blocked a verified apply for a day. | **A record that outlives its referent is a phantom too; retire it the moment the second method lands.** |
| A checkpoint reported a ruling the Lead had not yet made (intercept #6). | **A ruling exists when the authority issues it, never when a checkpoint anticipates it.** Proposals are labelled proposals. |

The index carries 41 such lines. None of them was written in advance; each one has a dated failure behind it.

## The dual-method catch

Two findings, one shape.

**The one it overturned itself.** On 2026-08-26 the system measured a public federal awards feed and recorded a
first-person finding: *12% duplicate award keys* in a 100-record sample. It drafted a public issue and queued it for a
human's approval. It did not post. On 2026-09-05, before anyone saw the issue, a re-run with the request recorded showed
the 12% was keyed on a field that legitimately recurs; on the source's own unique id the duplicate rate was **0 of
100**. The finding was overturned on the record, the queued issue was withdrawn, and the overturn is stamped on the
original row. The rule: *reproducible is derived, never asserted* — a number rides into a public artifact only when the
same request, re-run at draft time, returns it again.

**The one the machinery caught.** In September the system's second mission — reading public agent repositories for
*phantom mechanisms*, code that exists and is called by nothing — produced its first model-drafted gift: an issue
saying a guard function in a public repository was invoked by no source file. It was validated, queued, and waited for
the operator's tap. The Lead held it for a second read. The re-verification at source (an identifier graph and a
separate read of the runner surfaces — CI workflows, package scripts — both live through the API) came back **wired**:
the guard was called from its own file's main block, and that file was run by CI. A maintainer never saw the wrong
issue. The publish door now refuses any finding that does not carry two independent read paths whose verdicts agree,
both recorded on the row. As of 2026-09-28, 83 findings across 5 repositories are on the ledger and **0 of them have
passed that door** — which the system reports as zero, not as progress.

## The refused forgeries

Some prohibitions live in prose; the ones that matter live in the database, where they can be probed.

- **A memo in the other seat's name.** Every row in the Lead↔Builder channel is stamped by a trigger with the
  database role that wrote it; a Lead memo may be written only from the Lead's own role, and the Builder's seat
  verifies authority only against rows carrying that stamp — never against a memo's claim about itself. A standing
  probe has the Builder's seat *try* to write a Lead memo and passes only when the insert is refused.
- **An edited draft riding an old approval.** An outbound send needs a single-use token bound to the hash of the
  exact payload the human approved. Editing the draft invalidates the token; the send is refused until the edited
  draft is approved again.
- **A bet edited after minting.** Claim, confidence, criteria, and disconfirming signals are immutable by trigger. A
  review date may move only when the same write records from, to, reason, and by whom.
- **A count that pretends to be a receipt.** Every write is read back; a put without a read-back is recorded as a
  counter, not a receipt. A table the system cannot read is reported as *unreadable*, never as empty.

## The scoreboard, honestly

Calibration is computed only on **mechanically graded** bets (an automated judge, no human in the loop), trailing 90
days, read 2026-09-28:

| Stated confidence | n | Hit rate | Verdict |
|---|---|---|---|
| 0–20% | 42 | 64% | systematically under-confident |
| 20–40% | 37 | 73% | systematically under-confident |
| 40–60% | 0 | — | insufficient n |
| 60–80% | 2 | 100% | insufficient n (2 < 20) |
| 80–100% | 0 | — | insufficient n |

Brier score 0.47 over n = 81. The plain reading: the system's low-confidence bets hit far more often than it says they
will. That is a calibration defect, it is on the board, and no bucket is called "improved" against a prior window
until its n is at least 20 in both windows and the change exceeds binomial noise. The scoreboard is allowed to
embarrass the system; that is what it is for.

## The shape of the error ledgers

| Ledger | What it holds | Read 2026-09-28 |
|---|---|---|
| Predictions | every bet, with a claim, a confidence, and a review date fixed before the action | 134 rows: 107 reviewed, 25 cancelled (voided on the record, never deleted), 2 open |
| Actuals | what happened, graded against the bet: matched or not, the delta, the lesson | 103 rows |
| Assumptions and belief updates | seven written priors, set before evidence, and every move of a belief naming its evidence row | 6 updates; one belief moved 0.35 → 0.15 across graded misses, each step weak by rule (n = 1) |
| Governance findings | phantom mechanisms found by reading public agent repositories through the API, each with file, line, a re-run command, and a proposed fix | 83 findings, 5 repositories, 0 through the dual-method door |
| Public artifacts | everything drafted for the outside world, with its status | 8 queued, 8 withdrawn, 0 published |
| Runs | every scheduled or manual run, with tokens spent and how it ended | 1,016 rows |
| Defect rows | the Builder's and the Lead's own errors, numbered (B-29 and L-30 are the latest), each with what broke and what the fix was | on the known-issues file and in the memos |
| Friction | every run that ended refused, over ceiling, or short of variety, in fourteen named classes; a recurring class raises a card the Lead must rule on | on the day-close ledger |

The Lead's errors are numbered in the same series as the Builder's. The month's most important finding — the wrong
gift the second method caught — sits at the top of the known-issues file on the Lead's order, above every fix.

## What a human never delegates

Sends, spends, publishing, capability registration, migrations, the constitution, and the kill switch stay a human's
hand — by a written list, with validators behind most of it. This page is an instance: it was prepared from the
ledger by the Builder seat, reviewed by the Lead seat, and reaches the world only because the operator posted it.

---

*Sources, for anyone who wants to check the shape rather than take my word: the overturn is a stamped metadata field on
the original finding row; the scoreboard is a calibration function over the predictions and actuals tables; the belief
updates are an append-only table by trigger; the governance findings are task rows typed as findings; the run count is
the runs table. The private repository is not public; this page is what it can honestly show.*
