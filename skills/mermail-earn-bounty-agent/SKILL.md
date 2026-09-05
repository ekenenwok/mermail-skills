---
name: mermail-earn-bounty-agent
description: Turns Earn.xyz bounty emails sitting in a Mermail inbox into a ready-to-submit skill scaffold. Use when a user says things like "check my bounty email", "what bounties came in", or "help me start this bounty" and their inbox is connected via Mermail. Extracts prize, deadline, and submission requirements, then scaffolds the files and a submission-tracking checklist.
---

# Mermail Earn Bounty Agent

## What this skill enables

Freelancers and builders get bounty/gig notifications by email (Earn.xyz, Superteam, Gitcoin, etc.) and lose time manually re-reading each one, figuring out the deadline, and remembering what still needs submitting. This skill turns that triage into one workflow: **inbox → parsed brief → scaffolded submission → tracked status.**

## How it interacts with Mermail

This skill calls the Mermail MCP tools already configured in this repo's `.mcp.json` to:

1. **List/search the inbox** for unread mail matching bounty-platform senders (e.g. `noreply@earn.xyz`, `notifications@superteam.fun`) or subject keywords (`bounty`, `submission`, `deadline`, `prize`).
2. **Read the full email body** of each match to get the raw bounty text.
3. Optionally **label or archive** the email once its submission is scaffolded, so it doesn't get re-processed next run.

No Mermail write access beyond labeling is required — this skill never sends mail on the user's behalf.

## Workflow (start to finish)

1. **Trigger** — user asks the agent to check bounty mail (or this runs on a schedule).
2. **Fetch** — query Mermail for matching messages; if none, report "no new bounty mail" and stop.
3. **Parse** — from each email body, extract:
   - Bounty/task title
   - Prize amount and structure (e.g. "500 USDC — 1st: 250, 2nd: 100")
   - Deadline
   - Submission requirements (what files/links are required — PR, video, tweet, etc.)
   - Judging criteria, if listed
4. **Scaffold** — for each new bounty, create a folder `bounties/<slug>/` containing:
   - `BRIEF.md` — the parsed summary from step 3
   - `SUBMISSION.md` — a checklist template matching the exact requirements found (see `templates/SUBMISSION.md` in this skill)
   - `STATUS.md` — one line: `status: not_started | in_progress | submitted`, plus the deadline
5. **Track** — maintain `bounties/INDEX.md`, one row per bounty: title, deadline, status, prize. Update this file every run instead of recreating it.
6. **Report back** — tell the user in chat: how many new bounties found, their deadlines sorted soonest-first, and what's still incomplete from prior runs.

## Example prompts and expected results

**Prompt:** "Check my bounty email"
**Expected result:** Agent lists 1–3 new bounty emails found in Mermail, shows title/prize/deadline for each, and confirms it scaffolded a `bounties/<slug>/` folder for each with `BRIEF.md` and `SUBMISSION.md` ready to fill in.

**Prompt:** "What bounties do I still need to submit before Friday?"
**Expected result:** Agent reads `bounties/INDEX.md`, filters by deadline ≤ Friday and status ≠ submitted, and returns that filtered list — no new Mermail fetch needed.

**Prompt:** "Mark the Mermail skill bounty as submitted"
**Expected result:** Agent updates that bounty's `STATUS.md` to `submitted` and reflects it in `INDEX.md`.

## Notes

- This is intentionally **platform-agnostic within email**: it works for any bounty platform that emails its notifications (Earn.xyz, Superteam, Gitcoin, Layer3), not just Mermail-specific senders — widen or narrow the sender/subject filter in step 2 to taste.
- Keep parsing conservative: if a required field (deadline, submission link type) isn't clearly stated in the email, leave it blank in `BRIEF.md` rather than guessing, and flag it to the user.
