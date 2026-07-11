# FIELD — Session Log: 2026-07-11

## Overview

This session focused on MARK audit improvements that directly affect data flowing into FIELD.
No FIELD HTML code changed this session. All changes are upstream in MARK (the review desktop app).
FIELD reads MARK's output from Google Sheets — so any change to what MARK writes affects what FIELD displays.

---

## What Changed Upstream (MARK → FIELD pipeline)

### 1. Scoring is now correct (critical for FIELD dashboards)

**Previous MARK behavior (wrong):**
- MARK counted cross-collector corrections and self-corrections as errors
- The specialist collector (players+location person) was misidentified as a reviewer
- This inflated the error count, deflating quality scores by ~15 percentage points
- Example: a session that should score 92% was showing 77% in MARK and therefore in FIELD

**New MARK behavior (correct):**
- Error = amendment by the QA reviewer only
- Reviewer = person with amendments only, zero collection work (`diagnostics.work`: base=0, refinement=0, amendment>0)
- Specialist collector (heavy refinements, zero base) correctly excluded from reviewer detection
- Cross-collector corrections, self-corrections, system amendments all excluded
- Multiple reviewers per half supported

**Impact on FIELD:**
- All future audit scores flowing into the Collector Results Sheet will be correct
- Historical sessions (before v7.5.20) have inflated error counts and deflated scores
- If FIELD surfaces per-collector quality scores, the pre-v7.5.20 data should be treated as approximate

---

### 2. Error table now has structured per-type data (new columns in export)

MARK's CSV/Sheet export now has 15 columns with structured Before/After per error type:

```
Match ID | Match Name | Half | Timestamp | Event Name | Team |
Error Type | Module | Before | After |
Collector HR | Collector Name | Reviewer HR | Reviewer Name | Captured At
```

**Before** (old export): `Aspect` + `Edit Type` columns (Base/Extra/Location/Players + Added/Deleted/Wrong)
**After** (new export): `Error Type` (12 specific types) + `Module` + human-readable Before/After strings

**12 error types now tracked:**
`Deleted`, `Wrong event`, `Replaced`, `Wrong timestamp`, `Wrong extras`, `Wrong location`,
`Wrong player`, `Freeze frame`, `Goal location`, `Squad`, `Missed event`, `Wrong event`

**If FIELD ever reads the Events Sheet:** the column schema changed. `colParseEvent` in FIELD will need updating if it reads Error Type or Before/After columns.

---

### 3. Reviewer identity now correctly attributed

**Previous:** `reviewerName` and `reviewerHrCode` in the Sheet were always the MARK operator's login (whoever's machine ran MARK), not the actual QA reviewer.

**Now:** `reviewerName` and `reviewerHrCode` resolved from:
1. `identityMap` built from Tag Once's `EventHistory.authorInfo` (real reviewer's hrcode + name)
2. Firestore roster fallback
3. MARK operator's profile as last resort

**Impact on FIELD:** The Reviewer columns in the Collector Results Sheet now contain real reviewer identities, not the machine operator. FIELD's reviewer dashboard and QTL assignment views will show correct reviewer names going forward.

---

### 4. Half labels now correct in all exports

**Previous:** Half stored as `'1H'` / `'2H'` internally, but `fmtHalf()` had no mapping for these — the folder name, sheet name, and CSV `Half` column would show `"1H"` / `"2H"`.

**Now:** `fmtHalf()` correctly maps:
- `'1H'` / `'1'` → `'1st Half'`
- `'2H'` / `'2'` → `'2nd Half'`
- `'ET1'` → `'ET 1'`
- `'ET2'` → `'ET 2'`

**Impact on FIELD:** The `Half` column in the Collector Results Sheet now shows `'1st Half'` / `'2nd Half'` consistently. The `colParseEvent` and `colFilterEvents` functions already handle the `'1st Half'` / `'1H'` normalization — this aligns MARK's output to the expected format.

---

### 5. Session folder structure in Drive

Each MARK export now creates a folder in the master Drive folder:
```
{matchId}_{matchName}_{half}_{collectorHr}/
  ├── {matchId}_{matchName}_{half}_{collectorHr}.csv   ← native Google Sheet (converted)
  ├── {eventName}_{MM-SS.mmm}_{errorType}.mp4         ← video clips per error
  └── ...
```

**Clip naming format changed:** `ball-recovery_12-37.450_Wrong event.mp4` (was `ball-recovery_12m37s_Wrong event.mp4`)

---

## FIELD-Specific Status (Unchanged)

### Roles
| Role | Status |
|---|---|
| Batch Coordinator | ✅ complete |
| Batch Supervisor | ✅ complete |
| Batch Trainer | ✅ complete |
| Training Manager (role switcher) | ✅ complete |
| Quality Team Leader (QTL) | ✅ complete (added prior sessions) |
| Training Supervisor | ⏳ pending |
| Trainee | ⏳ pending |

### QTL Gate (added in prior sessions, not this one)
- Team Overview, Match Assignments, Feedback Tasks, Team Tasks pages
- My Assignments page for reviewer gate
- Collector/reviewer search from CSV lists (1,445 collectors, 199 reviewers)
- Instant-fresh Firestore catch-up for all dashboards
- Multi-collector session support via `colHrCodeMatches()` helper
- Event Analysis page with correct 17-col schema (joined via match_id+half+hr_code)
- Reviewer identity from Sheet columns AA/AB (real QA reviewer, not MARK operator)

---

## Pending Work for Next FIELD Session

1. **Training Supervisor role** — still pending, as noted in system prompt
2. **Trainee role** — still pending
3. **Score data refresh** — consider whether FIELD should display a "pre-v7.5.20" warning on historical audit sessions (scores may be understated by ~15pp)
4. **Error Type filter in Event Analysis** — now that MARK exports structured error types, FIELD could filter events by the 12 specific error categories instead of just Aspect+EditType

---

## Rules Reminder (unchanged)

- Single HTML file (`index.html` ~10.8k lines). No build step.
- Clone fresh each session: `git clone https://ahmedashraf-cyber:TOKEN@github.com/ahmedashraf-cyber/flowops.git flowops_push`
- Working file: `/home/claude/flowops_final.html`
- Push dir: `/home/claude/flowops_push/`
- After every edit: brace balance check `{ count == } count`, no duplicate functions
- All dropdowns use `nx-select-wrap` — never hand-roll selects
- Every screen uses `class="screen"` + `app-shell` grid
- Do NOT modify CSS variables without checking all screens first
- After push: wait for GitHub Pages green → hard refresh `Ctrl+Shift+R`
- Live: https://ahmedashraf-cyber.github.io/flowops/index.html
