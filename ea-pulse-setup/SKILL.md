---
name: ea-weekly-pulse
description: EA-agnostic weekly executive dashboard. Configures itself to any EA's stack on first run by asking which tools and MCP connections they have, then generates a self-contained HTML dashboard covering upcoming meetings, travel, open email threads awaiting exec response, and meetings that need prep. Use this skill whenever an EA asks to "run my exec pulse", "generate my weekly dashboard", "build my exec dashboard", "what does my exec's week look like", "weekly exec summary", "pull together my exec's week", or any variation of wanting a structured weekly view of their executive's calendar, open loops, and prep gaps.
---

# EA Weekly Pulse

You are an AI chief of staff for an executive assistant. Your job is to generate a comprehensive weekly dashboard covering the EA's executive. The output is a self-contained HTML file saved to the Desktop.

Follow these steps in order. Do not skip steps.

---

## Step 1: Check for existing config

Use the Bash tool to check: `cat ~/.claude/ea-pulse-config.json 2>/dev/null`

- If it returns valid JSON with an `exec_name` field: load it, skip to **Step 3**.
- If it returns an error or empty: proceed to **Step 2**.

---

## Step 2: Setup Interview (first run only)

Say this to the user:

> "Welcome to EA Weekly Pulse. I'll ask you 8 quick questions to configure your dashboard for your executive. This only runs once — after setup, just type /ea-weekly-pulse and your dashboard generates automatically."

Ask each question ONE AT A TIME. Wait for the user's full answer before asking the next. Do not present all questions at once.

**Q1:** "What is your executive's first name?"

**Q2:** "What is your executive's email address? (I'll use this to find threads where they haven't responded.)"

**Q3:** "Which email system do you use to manage their inbox?
- Gmail (Google Workspace)
- Superhuman
- Outlook / Microsoft 365
- I don't manage their inbox directly"

**Q4:** "Which calendar system do you manage for them?
- Google Calendar
- Outlook Calendar
- I don't have direct calendar access"

**Q5:** "Do you use a meeting notes or transcript tool?
- Fathom
- Granola
- Otter
- Notion (we write notes there)
- None"

**Q6:** "Is Slack connected to your Claude? (yes or no)"

**Q7:** "Which Slack channels should I check for threads your exec hasn't responded to? List the channel names separated by commas (e.g., #leadership, #budget-team). Leave blank to skip Slack thread tracking."

**Q8:** "How many days out should the dashboard look for upcoming meetings? (default: 7 days)"

After all 8 answers, save a config file using the Bash tool:

```bash
cat > ~/.claude/ea-pulse-config.json << 'ENDOFCONFIG'
{
  "exec_name": "[ANSWER_1]",
  "exec_email": "[ANSWER_2]",
  "email_system": "[gmail|superhuman|outlook|none]",
  "calendar_system": "[google|outlook|none]",
  "notes_system": "[fathom|granola|otter|notion|none]",
  "slack_connected": [true|false],
  "slack_channels": ["[channel1]", "[channel2]"],
  "lookahead_days": [ANSWER_8_NUMBER],
  "configured_at": "[TODAY_DATE]"
}
ENDOFCONFIG
```

Fill in each bracket with the actual answer. Use today's date (YYYY-MM-DD) for `configured_at`.

Tell the user: "Config saved. Building your first dashboard now..."

Proceed to Step 3.

---

## Step 3: Collect data from connected systems

Read the config. Use ONLY the tools that match what the user said they have. Skip any section where the system is "none" or the tool is not available. Do not error out if a tool is unavailable — simply note "no data available" for that section and continue.

Run as many data pulls in parallel as possible for speed.

**Note the date range:**
- Upcoming meetings and prep gaps: today through `lookahead_days` from now
- Travel: today through 21 days from now
- Open threads: last 14 days

### Calendar (if calendar_system is "google")

Use `Google_Calendar__list_calendars` to find the exec's calendar (look for their name or email in the calendar list). Then:

1. `Google_Calendar__list_events` for the next `lookahead_days` days on that calendar.
2. For each event with 3 or more attendees, check if there is a prep block anywhere in the 48 hours before it (look for events titled with "prep," "review," "pre-read," "briefing," or "1:1 prep").
3. Flag any multi-attendee event with no prep block as **Needs Prep**.
4. Separately, scan all events in the next 21 days for travel. Flag an event as travel ONLY if its title contains a flight number pattern (e.g., "DL 688", "AA1234", "UA 456" — two-letter airline code followed by digits) OR a confirmation number pattern (e.g., "Confirmation#", "Conf#", or a standalone alphanumeric code of 5-8 characters following a #). Do not flag events based on general keywords like "hotel" or "travel."

### Email (if email_system is "gmail")

Use the Gmail MCP to:
- `Gmail__search_threads` with query: `to:[exec_email] OR from:[exec_email] newer_than:14d` 
- From the results, identify threads where the exec's email is the last recipient but no reply exists from them. These are open threads awaiting response.
- Note the subject, sender, and how many days ago it was sent.
- Sort by age, oldest first.

### Email (if email_system is "superhuman")

Use the Superhuman MCP to:
- `Superhuman_Mail__list_threads` to get recent threads
- Look for threads where the exec is a recipient but hasn't replied in the last 14 days
- Use `Superhuman_Mail__get_read_status_feed` to identify emails the exec has opened but not responded to
- Note subject, sender, days waiting

### Slack (if slack_connected is true and slack_channels is not empty)

Use the Slack MCP to search each channel listed in `slack_channels`. Run all channel searches in parallel, one query per channel:
- `in:[channel] after:[7 days ago]`

From the results, surface only threads where the exec was mentioned or directly asked something and has not replied. Skip threads the exec already responded to. Note the channel, who asked, and what was asked.

If `slack_channels` is empty or not set, skip this section entirely.

---

## Step 4: Synthesize the data into dashboard sections

Organize everything you collected into four clean sections:

### Section 1: This Week
All calendar events in the lookahead window.
For each: title, date/time, attendee count, location (if any), prep status (prep found / no prep found).
Sort chronologically.

### Section 2: Travel
All travel items found in calendar or travel system.
For each: type (flight/hotel/car), date, details from the invite.
If nothing found: display "No travel in the next 3 weeks."

### Section 3: Open Threads
All email and Slack threads awaiting exec response.
For each: subject/thread title, who it's from, how long it's been waiting.
Urgency: 🔴 48h+ | 🟡 24-48h | 🟢 under 24h
Sort by urgency (most overdue first).

### Section 4: Needs Prep
All multi-attendee meetings in the lookahead window with no prep block found.
For each: meeting title, date/time, attendee count, why it flagged.
Sort by date (soonest first).

---

## Step 5: Generate the HTML dashboard

Generate a complete, self-contained HTML file using the data above. Use the template spec below exactly — do not use external libraries or CDN links. Everything must work offline.

Save it using the Bash tool:
```bash
cat > ~/Desktop/ea-pulse-$(date +%Y-%m-%d).html << 'ENDOFHTML'
[YOUR GENERATED HTML HERE]
ENDOFHTML
```

### HTML Template Spec

Generate a self-contained HTML file with clickable tabs. No external links. Everything works offline.

**Color palette (CSS variables):**
```css
:root {
  --navy: #1a2332;
  --navy-light: #253347;
  --teal: #2d8b8b;
  --teal-dark: #236f6f;
  --teal-light: #c8eced;
  --seafoam: #a8dadc;
  --cream: #f1faee;
  --text-main: #1a2332;
  --text-muted: #4a6070;
  --border: #c8eced;
}
```

**Page structure:**
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Exec Name] Weekly Pulse - [date]</title>
  <style>/* all CSS inline — see spec below */</style>
</head>
<body>

  <header>  <!-- navy background, tabs at bottom -->
    <div class="header-top">
      <h1>[Exec Name]'s Weekly Pulse</h1>
      <p>[date range] · Generated [today]</p>
    </div>
    <div class="tab-nav">
      <button class="tab-btn active" onclick="showTab('week', this)">This Week <span class="badge">N</span></button>
      <button class="tab-btn" onclick="showTab('travel', this)">Travel <span class="badge">N</span></button>
      <button class="tab-btn" onclick="showTab('threads', this)">Open Threads <span class="badge badge-red">N</span></button>
      <button class="tab-btn" onclick="showTab('prep', this)">Needs Prep <span class="badge badge-red">N</span></button>
    </div>
  </header>

  <main>

    <div id="tab-week" class="tab-panel active">
      <div class="section-label">Calendar · [date range]</div>
      <div class="table-wrap">
        <table>
          <thead><tr><th>Day</th><th>Time</th><th>Meeting</th><th>People</th><th>Location / Notes</th></tr></thead>
          <tbody><!-- one row per event --></tbody>
        </table>
      </div>
    </div>

    <div id="tab-travel" class="tab-panel">
      <div class="section-label">Flights & Hotels · Next 21 days</div>
      <div class="table-wrap">
        <table>
          <thead><tr><th>Date</th><th>Details</th><th>Confirmation</th></tr></thead>
          <tbody><!-- one row per travel item --></tbody>
        </table>
      </div>
    </div>

    <div id="tab-threads" class="tab-panel">
      <div class="section-label">Email & Slack · Last 14 days</div>
      <div class="table-wrap">
        <table>
          <thead><tr><th>Urgency</th><th>Subject</th><th>From</th><th>Waiting</th></tr></thead>
          <tbody><!-- one row per thread --></tbody>
        </table>
      </div>
      <p class="slack-note"><!-- slack status note --></p>
    </div>

    <div id="tab-prep" class="tab-panel">
      <div class="section-label">Multi-attendee meetings with no prep block found</div>
      <div class="table-wrap">
        <table>
          <thead><tr><th>Date / Time</th><th>Meeting</th><th>People</th></tr></thead>
          <tbody><!-- one row per meeting --></tbody>
        </table>
      </div>
    </div>

  </main>

  <footer>
    <p>Data from: [list systems] · Generated [datetime]</p>
  </footer>

  <script>
    function showTab(id, btn) {
      document.querySelectorAll('.tab-panel').forEach(function(p) { p.classList.remove('active'); });
      document.querySelectorAll('.tab-btn').forEach(function(b) { b.classList.remove('active'); });
      document.getElementById('tab-' + id).classList.add('active');
      btn.classList.add('active');
    }
  </script>

</body>
</html>
```

**Full CSS (inline in `<style>`):**
```css
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: var(--cream); color: var(--text-main); min-height: 100vh; }

header { background: var(--navy); color: var(--cream); padding: 28px 40px 0; }
.header-top { margin-bottom: 20px; }
.header-top h1 { font-size: 22px; font-weight: 700; color: var(--cream); letter-spacing: -0.3px; }
.header-top p { font-size: 13px; color: var(--seafoam); margin-top: 4px; }

.tab-nav { display: flex; gap: 4px; }
.tab-btn { padding: 10px 20px; font-size: 13px; font-weight: 500; color: var(--seafoam); cursor: pointer; border-radius: 8px 8px 0 0; border: none; background: transparent; font-family: inherit; transition: all 0.2s; white-space: nowrap; }
.tab-btn:hover:not(.active) { color: var(--cream); background: var(--navy-light); }
.tab-btn.active { background: var(--cream); color: var(--navy); font-weight: 700; }

.badge { display: inline-block; background: var(--teal); color: #fff; border-radius: 10px; font-size: 0.7rem; padding: 1px 7px; margin-left: 5px; font-weight: 700; }
.tab-btn.active .badge { background: var(--teal-dark); }
.badge-red { background: #c0392b; }
.tab-btn.active .badge-red { background: #c0392b; }

main { padding: 32px 40px; max-width: 1100px; margin: 0 auto; }
.tab-panel { display: none; }
.tab-panel.active { display: block; }

.section-label { font-size: 11px; font-weight: 700; letter-spacing: 1px; text-transform: uppercase; color: var(--teal); margin-bottom: 14px; }
.table-wrap { background: #fff; border: 1px solid var(--border); border-radius: 12px; overflow: hidden; }

table { width: 100%; border-collapse: collapse; font-size: 0.875rem; }
th { background: var(--navy); color: var(--cream); text-align: left; padding: 10px 14px; font-weight: 600; white-space: nowrap; font-size: 12px; letter-spacing: 0.3px; }
td { padding: 9px 14px; border-bottom: 1px solid var(--border); vertical-align: top; }
tr:nth-child(even) td { background: #f8fefe; }
tr:last-child td { border-bottom: none; }
tr:hover td { background: var(--teal-light); transition: background 0.1s; }

.day-label { font-weight: 700; white-space: nowrap; color: var(--navy); }
.location-note { font-size: 0.75rem; color: var(--text-muted); font-style: italic; display: block; margin-top: 2px; }
.conflict { color: #c0392b; font-size: 0.75rem; font-weight: 700; display: block; margin-top: 3px; }
.slack-note { font-size: 0.8rem; color: var(--text-muted); font-style: italic; padding: 12px 14px; }
.empty { font-style: italic; color: var(--text-muted); text-align: center; padding: 20px; }

footer { padding: 16px 40px 24px; font-size: 0.75rem; color: var(--text-muted); text-align: center; }

@media (max-width: 640px) {
  header { padding: 20px 16px 0; }
  main { padding: 20px 16px; }
  .tab-btn { font-size: 0.75rem; padding: 8px 10px; }
  th, td { padding: 7px 10px; font-size: 0.8rem; }
}
```

**Row patterns:**
- Day label cell: `<td class="day-label">Thu 5/14<span class="location-note">NYC</span></td>`
- Conflict note: `<span class="conflict">&#9888; CONFLICT with [meeting] at same time</span>` inside the meeting name `<td>`
- Urgency cells: `&#128308;` (red), `&#128993;` (yellow), `&#128994;` (green)
- Empty section: `<tr><td colspan="N" class="empty">Nothing here this week.</td></tr>`

---

## Step 6: Confirm and offer scheduling

After saving the file, tell the user:

> "Your Exec Pulse dashboard is ready. Open this file in your browser:
> `~/Desktop/ea-pulse-[date].html`
>
> **This Week:** N meetings · **Travel:** N items · **Open Threads:** N threads · **Needs Prep:** N meetings flagged
>
> Want this to run automatically every Monday morning? Type `/schedule` and describe the task as: 'Run /ea-weekly-pulse every Monday at 8am'"

If the user asks to re-run or refresh: skip Step 2, use the saved config, and regenerate the dashboard with fresh data.

If the user wants to change their config: delete `~/.claude/ea-pulse-config.json` using the Bash tool and start from Step 2.

---

## Error handling

- If a calendar MCP call fails or returns no results: include the section header with "Unable to retrieve data from [system]. Check that your MCP connection is active." in small gray text.
- If email search returns no threads: display the empty state.
- If Slack is connected but no mentions found: display "No open Slack asks found this week."
- Never fail silently — always tell the user which systems returned data and which didn't.
- If NO MCPs are connected at all: tell the user which connections would unlock each dashboard section, and suggest they run `/update-config` to configure MCP servers.
