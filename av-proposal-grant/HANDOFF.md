# AV Proposal: Grant Handoff for Chelsea

This document gives you (Chelsea) everything you need to continue working on the Grant AV proposal with Claude Code. Open this file and `av-proposal-grant.html` together before starting a session.

---

## How to use this with Claude Code

1. Open Claude Code on your machine (terminal or VS Code extension)
2. At the start of your session, say: **"Read this handoff doc and the HTML file before we do anything else"** and paste the file paths below
3. Claude Code will read both files and have full context to make any changes you need

**File paths (update if you move them):**
- Handoff doc: `~/Desktop/Baton/drafts/planning/av-proposal-grant-HANDOFF.md`
- HTML proposal: `~/Desktop/Baton/drafts/planning/av-proposal-grant.html`

---

## What this is

A 5-tab HTML decision brief for Grant (CEO), asking him to approve the all-hands space AV upgrade. Ashley built this with Claude Code to replace the original 10-tab draft. The goal: Grant reads Tab 1, knows what he needs to decide, and can click into details only if he wants to.

This is **Phase 3** of Baton's AV refresh. Conference rooms are done (Phase 1/2). This covers the all-hands/TGIT space only.

**Where it lives:** `~/Desktop/Baton/drafts/planning/av-proposal-grant.html`
**Do not overwrite:** `~/Desktop/Draft html in claude.html` (your original draft, kept for reference)

---

## Project context

**Two vendor quotes are in hand:**

| | CTI (Conference Technologies Inc.) | Young Electric |
|---|---|---|
| Total | $46,788.85 | $44,021.49 |
| Architecture | BYOD, IP-based, platform-agnostic | Zoom Room (hardwired) |
| Deposit | 60% upfront (~$28,073) | None |
| Support | 1-yr CTI Complete support included | None |
| Platform | Works with Teams and Google Meet | Zoom-only (conflicts with Teams requirement) |
| Cameras | 2x QSC NC-12x80 PTZ | 1x Logitech Rally PTZ |
| Display | Newline 98" TV included | Samsung 98" display |
| Local presence | Remote / national | Local SF contractor |

**Recommendation in the HTML: CTI.** Young Electric is named as a fallback only if the 60% deposit becomes a cash flow issue.

**CTI updated quote is pending.** They are revising scope to add a Teams or Google Meet touchpad (one-tap room joining, no personal laptop needed). This quote arrives after Grant decides on platform. Current quote total: $46,788.85.

---

## The 5 decisions Grant needs to make

These are listed in Tab 1 of the HTML. Grant needs to confirm or decide each one:

1. **Approve CTI as vendor** (Confirm)
2. **Choose platform: Teams or Google Meet** (Decide, no recommendation given, Grant's call)
3. **Choose display type: 98" TV or projector** (Decide, TV is recommended)
4. **Align on July 6 deadline and understand the risks** (Confirm)
5. **Gaming area: where does it go?** (Decide, front-door placement recommended pending measurement)

---

## Key accuracy corrections (important, do not revert)

Ashley corrected these from your original draft:

| Item | Wrong (original) | Correct (current HTML) |
|---|---|---|
| Deadline | July 10 | **July 6** (before OKR week July 13) |
| CTI total | $45,767 | **$46,788.85** (actual quote total) |
| Platform status | "Teams recommended" | **TBD, Grant's call** (platform is undecided) |
| Network risk | Monitor | **Low risk / FYI** (network recently upgraded) |

---

## HTML structure: 5 tabs

### Stats bar (always visible)
Budget ~$56K | Deadline July 6 | Vendor CTI (Recommended) | Platform: Your call (amber) | Quote status: Update pending

### Header pills
"CTI recommended" (green) | "Platform TBD" (amber) | "5 decisions needed" (amber)

### Tab 1: The Ask (default tab)
- 3 "yes yes" agreement questions (Dale Carnegie framing to anchor Grant in agreement mode)
- 5 numbered decisions with Confirm/Decide tags
- Blue info callout: updated CTI quote coming
- Post-approval action checklist (all Chelsea-owned, nothing left for Grant after he approves)

### Tab 2: Why CTI
- Green verdict block with recommendation and price
- 7 feature bullets mapped to Grant's original requirements (citations like "f-grant" in the HTML map to his original requirements doc)
- Amber warning: Young Electric is Zoom-centric and hardwired, conflicts with Teams requirement
- Collapsible comparison table (click "Show full comparison" to expand)

### Tab 3: Open Decisions
Three decision cards:
- **Platform:** Option A (Teams) vs Option B (Google Meet), no recommendation, Grant decides
- **Display:** TV recommended (included in quote, portable, no construction); projector = +$8,500, electrical work, landlord involvement, not portable
- **Gaming area:** Three placement options (near front door recommended, pending measurement)

### Tab 4: Timeline + Budget
- 6-milestone timeline working backward from July 6
- Fallback plan: if install slips, keep current setup for OKR week July 13, cut over after
- Budget breakdown: CTI $46,788.85 + TBD add-ons = all-in ~$56K
- 60% deposit warning (~$28,073 due on award)
- 10% contingency callout

### Tab 5: Risks + Next Steps
Three risk tiers:
- **Resolve before signing:** Hidden costs (shipping, wiring, labor beyond quote), handheld mic (adds cost if added), storage room outlet (outlet run required, cost unconfirmed)
- **Monitor:** Install timeline confirmation with CTI, product delivery lead times
- **Low risk / FYI:** Network compatibility (recently upgraded, Martin to confirm as precaution)

Next steps are all Chelsea-owned with Martin (Jones IT) on network check.

---

## Design system (if you're making visual changes)

The HTML uses CSS custom properties. Key ones:

```
--bg: #F5F4F1        (page background)
--surface: #FDFCFA   (card background)
--ink: #1A1916       (body text)
--green: #1A7A4A     (positive / recommended)
--gbg: #EAF6EE       (green background)
--amber: #B45309     (warning / TBD)
--abg: #FEF9EC       (amber background)
--border: #E5E3DC    (dividers)
```

Fonts: **Fraunces** (serif, headers) and **Geist** (sans-serif, body). Both loaded from Google Fonts.

---

## Things that are still pending / may need updates

- **Updated CTI quote:** When CTI sends the revised quote with the Teams/Google Meet touchpad included, update the price throughout the HTML (stats bar, verdict block, budget table). Search for `46,788.85` to find all instances.
- **Gaming area measurement:** If the front-door area is confirmed (or ruled out) after measuring, update the gaming area decision card in Tab 3 to reflect the confirmed option.
- **Platform decision:** Once Grant decides Teams vs Google Meet, update Tab 3 to show the chosen option, and update the info callout on Tab 1 to remove the "pending your platform decision" note.
- **Award timing:** If the award date is confirmed, update the timeline milestones in Tab 4.

---

## What Grant cares about (use this to sharpen any edits)

1. **Ryder/Teams alignment:** He originally required Teams explicitly ("Zoom and Google Meet are out of scope"). He's asked multiple times about Teams Room licensing, Outlook calendar integration, and bridging Ryder and Baton systems. Any change that affects the Teams angle needs careful framing.
2. **Not getting locked in:** He asked if equipment was returnable before approving a $4K test purchase. He ran Phase 2.5 to test before committing to full rollout. He needs to know he has a named fallback (Young Electric is that fallback in the HTML).
3. **Moving fast on clear asks:** He approved Ashley's previous proposals the same day when the recommendation was clear and the risk was named. What slows him down is ambiguity. Keep the ask clean and the next steps Ashley-owned.

---

## Questions? Ask Ashley before sending to Grant.

Ashley reviewed all corrections and signed off on this draft. If you make material changes (price, deadline, platform framing, risk labels), loop her in before it goes out.
