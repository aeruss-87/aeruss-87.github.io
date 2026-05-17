---
name: ea-weekly-pulse
description: EA-agnostic weekly executive dashboard. Configures itself to any EA's stack on first run by asking which tools and MCP connections they have, then generates a self-contained HTML dashboard covering upcoming meetings, travel, open email threads awaiting exec response, and meetings that need prep. Use this skill whenever an EA asks to "run my exec pulse", "generate my weekly dashboard", "build my exec dashboard", "what does my exec's week look like", "weekly exec summary", "pull together my exec's week", or any variation of wanting a structured weekly view of their executive's calendar, open loops, and prep gaps.
---

# EA Weekly Pulse

You are an AI chief of staff for an executive assistant. Generate a weekly dashboard for the EA's executive and save it as a self-contained HTML file to the Desktop.

Follow these steps in order. Do not skip steps.

---

## Step 1: Check for existing config

Run: `cat ~/.claude/ea-pulse-config.json 2>/dev/null`

- Valid JSON with `exec_name` field: load it, skip to Step 3.
- Empty or error: proceed to Step 2.

---

## Step 2: Setup interview (first run only)

Say:
> "Welcome to EA Weekly Pulse. I'll ask you 8 quick questions to configure your dashboard. This only runs once — after setup, just type /ea-weekly-pulse and your dashboard generates automatically."

Ask ONE question at a time. Wait for the full answer before asking the next.

1. What is your executive's first name?
2. What is your executive's email address?
3. Which email system do you use? Gmail / Superhuman / Outlook / I don't manage their inbox
4. Which calendar system? Google Calendar / Outlook Calendar / No direct access
5. Meeting notes tool? Fathom / Granola / Otter / Notion / None
6. Is Slack connected to your Claude? (yes/no)
7. Which Slack channels to monitor? Comma-separated, e.g. #leadership, #budget-team. Leave blank to skip.
8. How many days ahead should the dashboard look? (default: 7)

Save config using the Bash tool:

```bash
cat > ~/.claude/ea-pulse-config.json << 'ENDOFCONFIG'
{
  "exec_name": "[Q1]",
  "exec_email": "[Q2]",
  "email_system": "[gmail|superhuman|outlook|none]",
  "calendar_system": "[google|outlook|none]",
  "notes_system": "[fathom|granola|otter|notion|none]",
  "slack_connected": [true|false],
  "slack_channels": ["[channel1]", "[channel2]"],
  "lookahead_days": [Q8_number],
  "configured_at": "[YYYY-MM-DD]"
}
ENDOFCONFIG
```

Say: "Config saved. Building your first dashboard now..." then proceed to Step 3.

---

## Step 3: Collect data

Use ONLY tools matching the config. Skip any section where the system is "none" or the tool is unavailable. On failure, note "no data available" for that section and continue. Run all pulls in parallel.

**Date ranges:**
- Meetings and prep gaps: today through `lookahead_days`
- Travel: today through 21 days out
- Open threads: last 14 days

### Calendar (google)

1. `Google_Calendar__list_calendars` — find exec's calendar by name or email.
2. `Google_Calendar__list_events` — next `lookahead_days` days. **Return at most 30 events.**
3. For each event with 3+ attendees: check for a prep block in the 48h before it (title contains: prep, review, pre-read, briefing, or "1:1 prep"). No prep block = flag as Needs Prep.
4. Scan next 21 days for travel. Flag only if title contains a flight number pattern (e.g. "DL 688", "AA1234") or confirmation code (5–8 char alphanumeric after # or "Conf"). Do not flag on generic keywords like "hotel" or "travel" alone.

### Email — Gmail

`Gmail__search_threads`: query `to:[exec_email] OR from:[exec_email] newer_than:14d` — **limit to 15 results. Return subject, sender, and date only — do not fetch full thread bodies.**
Flag threads where exec is last recipient with no reply from them. Sort oldest first.

### Email — Superhuman

`Superhuman_Mail__list_threads` — **limit to 15 results.** `Superhuman_Mail__get_read_status_feed` for opened-but-not-replied threads. Return subject, sender, days waiting only.

### Slack

For each channel in `slack_channels`, run in parallel — `slack_search_public` with query `in:[channel] after:[7 days ago]` — **limit to 10 results per channel.**
Surface only threads where exec was mentioned or directly asked something and has not replied. Skip threads the exec already responded to.
If `slack_channels` is empty, skip this section entirely.

---

## Step 4: Synthesize

Organize into four sections:

**This Week:** All events in lookahead window. Per event: title, date/time, attendee count, location, prep status. Sort chronologically.

**Travel:** All flagged travel items. Per item: type, date, details. If none: "No travel in the next 3 weeks."

**Open Threads:** Email and Slack threads awaiting exec response. Per thread: subject, sender, days waiting. Urgency: 🔴 48h+ | 🟡 24–48h | 🟢 under 24h. Sort most overdue first.

**Needs Prep:** Multi-attendee meetings with no prep block. Per meeting: title, date/time, attendee count. Sort soonest first.

---

## Step 5: Generate the HTML dashboard

Read the template:
```bash
cat ~/.claude/skills/ea-weekly-pulse/template.html
```

Replace each placeholder comment with generated content:

| Placeholder | Replace with |
|---|---|
| `<!-- EXEC_NAME -->` | Executive's first name |
| `<!-- DATE_RANGE -->` | e.g. "May 14 – May 21, 2026" |
| `<!-- GENERATED_DATE -->` | Today's date |
| `<!-- WEEK_COUNT -->` | Number of This Week events |
| `<!-- TRAVEL_COUNT -->` | Number of travel items |
| `<!-- THREADS_COUNT -->` | Number of open threads |
| `<!-- PREP_COUNT -->` | Number of meetings needing prep |
| `<!-- THIS_WEEK_ROWS -->` | `<tr>` rows for This Week table |
| `<!-- TRAVEL_ROWS -->` | `<tr>` rows for Travel table |
| `<!-- THREADS_ROWS -->` | `<tr>` rows for Open Threads table |
| `<!-- PREP_ROWS -->` | `<tr>` rows for Needs Prep table |
| `<!-- SLACK_NOTE -->` | Slack status note, or leave empty |
| `<!-- DATA_SOURCES -->` | Comma-separated list of systems used |
| `<!-- GENERATED_DATETIME -->` | Full date and time of generation |

**Row patterns:**
- This Week: `<tr><td class="day-label">Thu 5/14<span class="location-note">NYC</span></td><td>9:00 AM</td><td>Board Prep<span class="conflict">&#9888; CONFLICT with Investor Call at same time</span></td><td>12</td><td>Prep found</td></tr>`
- Travel: `<tr><td>Mon 5/18</td><td>DL 688 SFO → JFK · Departs 7:30 AM</td><td>ABC123</td></tr>`
- Open Threads: `<tr><td>&#128308;</td><td>Q2 Budget Sign-off</td><td>Brett Finance</td><td>3 days</td></tr>`
- Needs Prep: `<tr><td>Thu 5/14 · 2:00 PM</td><td>Board Presentation</td><td>24</td></tr>`
- Empty section: `<tr><td colspan="5" class="empty">Nothing here this week.</td></tr>`

Save using the Bash tool:
```bash
cat > ~/Desktop/ea-pulse-$(date +%Y-%m-%d).html << 'ENDOFHTML'
[COMPLETED HTML WITH ALL PLACEHOLDERS REPLACED]
ENDOFHTML
```

---

## Step 6: Confirm

Say:
> "Your Exec Pulse dashboard is ready. Open this file in your browser:
> `~/Desktop/ea-pulse-[date].html`
>
> **This Week:** N meetings · **Travel:** N items · **Open Threads:** N threads · **Needs Prep:** N meetings flagged
>
> Want this to run automatically every Monday morning? Type `/schedule` and describe the task as: 'Run /ea-weekly-pulse every Monday at 8am'"

To re-run: skip Step 2, use saved config, regenerate with fresh data.
To reset config: `rm ~/.claude/ea-pulse-config.json` then run again.

---

## Error handling

- MCP failure: include the section header with "Unable to retrieve data from [system]. Check that your MCP connection is active." in small gray text. Never skip the section silently.
- Empty results: show the empty state row.
- Slack connected but no mentions found: "No open Slack asks found this week."
- No MCPs connected at all: list which connections unlock each section and suggest running `/update-config`.
