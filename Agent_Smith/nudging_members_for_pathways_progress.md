---
name: nudging_members_for_pathways_progress
description: "Use this skill when asked to send Pathways encouragement emails, nudge members about logging their speeches, follow up on educational goal progress, or run the weekly/monthly member outreach cycle. Triggers: 'nudge members', 'send Pathways reminders', 'check educational goals', 'who should I reach out to this week'. Do NOT use for one-off email drafting, role assignments, or member roster updates."
compatibility: "Claude Code with Flask MCP server — requires read_agent_smith_sheet, write_agent_smith_sheet, and reply_to_vpe_personal_assistant_email tools"
---

# Nudging Members for Pathways Progress

## Why this skill exists

The club has strong attendance but weak Pathways logging. Members are delivering speeches
but not recording them in Basecamp, and the club is falling short of its educational goals
as a result. This skill gives you a reliable, consistent way to reach out to members with
personalized encouragement — without spamming them or sending generic blasts.

The core loop is: **read the roster → identify who needs a nudge → draft personalized
emails → send → log that you sent them.**

---

## Sheet structure assumptions

This skill assumes the following columns exist in your Google Sheet. If they don't yet,
add them before running this skill for the first time.

### `Roster` tab — required columns

| Column | Description |
|--------|-------------|
| `Name` | Member's full name |
| `Email` | Member's email address |
| `Pathways Path` | The path they enrolled in (e.g., "Presentation Mastery"). Leave blank if unknown. |
| `Current Level` | Level they are working on (1–5). Leave blank if unknown. |
| `Last Speech Date` | Date of their most recent speech at club (any speech — you update this manually after meetings). Format: YYYY-MM-DD |
| `Last Nudge Date` | Date you last sent them a Pathways nudge email. Format: YYYY-MM-DD. Leave blank if never nudged. |
| `Nudge Notes` | Free-text notes about this member's situation (e.g., "Said they're working on Level 2 project", "Prefers not to be contacted about Pathways"). |
| `Do Not Nudge` | TRUE or FALSE. Set to TRUE if member has asked not to be contacted or is inactive. |

### `Meetings` tab — used for context

Read the last 4–6 rows to understand which members spoke recently. This helps personalize
emails ("Great job on your Table Topics last week!") and avoids nudging someone who just
spoke.

---

## Step-by-step execution protocol

### Step 1: Read the sheet

Call `read_agent_smith_sheet` twice:

```
range: "Roster!A:I"        # all roster columns
range: "Meetings!A:F"      # recent meeting schedule and role assignments
```

Parse both results into a working data structure before proceeding. Do not skip this step
or rely on cached data from a previous run.

### Step 2: Build the candidate list

Identify members who meet ALL of the following criteria:

- `Do Not Nudge` is not TRUE
- Email address is present
- Has not been nudged in the last **28 days** (compare `Last Nudge Date` to today's date)
- Has not given a speech in the last **14 days** (compare `Last Speech Date` to today)

Then **prioritize** within that list using this order:

1. **Never nudged** — `Last Nudge Date` is blank. These members come first.
2. **No speech logged in 60+ days** — longest gap at the top.
3. **Pathways Path is blank** — they may not have enrolled yet; a light-touch onboarding
   nudge is appropriate.
4. **Everyone else** — sorted by longest time since last nudge.

Cap the outreach list at **8 members per run** to keep volume manageable and emails
feeling personal, not automated.

### Step 3: Draft emails

Draft one email per member. Do not batch members into a single email.

**Follow these rules for every email:**

- Address the member by first name only
- Write in a warm, collegial tone — peer-to-peer, not top-down
- Keep it short: 3–5 sentences maximum for the body
- Never use the word "remind" or "reminder" — it sounds like nagging
- Never mention the club's educational goals or metrics — this is about the member's
  growth, not club performance
- Never say "I noticed you haven't..." — frame positively around what they *can* do
- Always include one specific, actionable next step (see personalization rules below)
- Sign off as: "— [Your Name], VP of Education"

**Personalization rules — apply the first one that fits:**

| Member situation | Personalization hook |
|-----------------|----------------------|
| Spoke at a recent meeting | Reference the speech or role: "Your [role] last week was great." |
| Has a Pathways path but no level logged | Ask what project they're working on and offer to help |
| Has a level but no recent speech | Encourage them to sign up for an upcoming meeting slot |
| Pathways path is blank | Mention Pathways briefly and offer a 5-minute chat to get them set up |
| Never nudged, no data | Warm intro — you're here to support their journey, here's how to reach you |

**Subject line formula:** Keep it conversational. Examples:
- "Quick check-in on your Pathways journey"
- "How's [Path Name] going?"
- "Wanted to reach out — [First Name]"

Do NOT use: "Reminder:", "Action Required", "Following up on Pathways", or any subject
that reads like a mass email.

### Step 4: Present drafts for review

Before sending anything, output all drafted emails in this format:

```
=== NUDGE BATCH — [today's date] ===
[X] emails drafted | [Y] members skipped (Do Not Nudge or recently contacted)

---
DRAFT 1 of X
To: [Name] <[email]>
Subject: [subject line]
Last nudge: [date or "never"] | Last speech: [date or "unknown"]
Personalization hook used: [one line explaining why you wrote what you wrote]

[email body]

---
DRAFT 2 of X
...
```

Then ask: **"Send all, send some, or edit before sending?"**

Do not proceed to Step 5 until you receive explicit approval.

### Step 5: Send approved emails

For each approved email, call `reply_to_vpe_personal_assistant_email`. 

> ⚠️ Important: This tool sends replies within existing Gmail threads. If a member
> email address does not correspond to an existing thread in the VPE_Personal_Assistant
> label, flag it and skip — do not attempt to fabricate a message_id. Note these
> in your post-send summary so a new thread can be initiated manually if needed.

### Step 6: Log the sends

After sending, call `write_agent_smith_sheet` to update the `Roster` tab:

- Set `Last Nudge Date` to today's date (YYYY-MM-DD) for every member you successfully
  emailed
- Append any relevant context to `Nudge Notes` (e.g., "Sent Level 2 encouragement
  email 2025-06-01")

Do not overwrite existing `Nudge Notes` — append with a date prefix.

### Step 7: Produce a run summary

Output a clean summary for your records:

```
=== NUDGE RUN COMPLETE — [date] ===

Sent:    [N] emails
Skipped: [N] members (already nudged recently)
Flagged: [N] members (no Gmail thread found — need manual outreach)

Next recommended run: [date 28 days from today]

Members flagged for manual outreach:
- [Name] ([email]) — reason
```

---

## What this skill must never do

- **Never send an email without explicit approval** from you in Step 4
- **Never nudge a member more than once every 28 days** regardless of what you're asked
- **Never mention club metrics, goals, or "we need you to complete your path"** framing
- **Never fabricate a message_id** to use with the reply tool — skip and flag instead
- **Never overwrite the `Do Not Nudge` column** — only you set that manually
- **Never contact more than 8 members per run** without being explicitly asked to override

---

## References

- Club tone guide: `references/club_voice_and_tone.md` *(create this — see note below)*
- Pathways path names and level structure: `references/pathways_reference.md`

### Note on references

Before your first nudge run, create `references/club_voice_and_tone.md` with 2–3
example emails you've sent that felt right, plus a short description of your club's
culture. Claude will use these as tone calibration. Even 3 sentences of guidance ("We're
a professional club but not stiff — members go by first names, humor is welcome, nobody
likes being lectured") will meaningfully improve output quality.

---

## Troubleshooting

**"I don't have Pathways path info for most members"**
That's expected at first. Run the skill anyway — the personalization rules handle blank
path data gracefully. Over time, as members reply to your nudges or you gather info at
meetings, fill in the sheet. The skill gets better as data improves.

**"The reply tool says there's no existing thread for this member"**
This means they've never emailed your VPE label. Flag them in the run summary and send
the first email to them manually from Gmail — once they reply, future nudges can go
through the tool.

**"A member asked to not be contacted"**
Set `Do Not Nudge` to TRUE in the Roster tab immediately. This skill checks that column
on every run.