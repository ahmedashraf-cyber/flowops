<!-- ============================================================= -->
<!-- SESSION 2026-06-24 — MARK v7.5.x + ecosystem handoff          -->
<!-- ============================================================= -->

## Latest Session: 2026-06-24 — MARK module scores + handoff docs

### TL;DR for FIELD
- **No FIELD code changed this session** — work was on MARK (the review side that
  feeds FIELD's scores). FIELD pending work is unchanged: **Training Supervisor**
  and **Trainee** roles.
- **MARK is now at v7.5.7.** Major additions feeding into FIELD's quality picture:
  - **Collector/reviewer identity** resolved to real `Name (A-####)` + email
    (from the collection app's `EventHistory.authorInfo.hrcode`, merged with a
    Firestore roster seeded from `users_finalized_*.csv`).
  - **Per-module quality scores** (Base, Pressure, Players, Location, Extras,
    Freeze Frame) — shipped, but the denominator is being reconciled with the
    analysis team (MARK ~375 vs reference ~350). See the MARK repo's
    `MODULE_SCORES.md`. When this settles, FIELD may surface per-module scores
    per collector/batch, not just an overall score.
- **New ecosystem doc:** there is a THIRD app — the **Video Feedback Dashboard**
  (Next.js/Supabase/Telegram/Drive/Gmail) that delivers per-collector video
  feedback and emails collector reports. It carries the authoritative analysis-
  team module-score CSVs. Full map in the MARK repo's `ECOSYSTEM.md`. (The
  earlier "only two repos" rule from 2026-06-11 predates this; the dashboard is a
  sibling app owned separately, not a backend change to MARK/FIELD.)
- **Handoff prompt** for a fresh Claude chat: MARK repo `HANDOFF_PROMPT.md`.

### Reminders for FIELD work (unchanged)
- Single HTML file. Dark Apple-style; accent `#E8590C`; Inter/DM Sans/JetBrains
  Mono. All dropdowns use `nx-select-wrap`; screens use `class="screen"` +
  `app-shell` grid. Do NOT modify CSS vars or nx-select without checking existing
  code. Clone fresh each session; brace-balance + duplicate-function check after
  edits; after push, wait for Pages green then hard-refresh.

---

<!-- ============================================================= -->
<!-- SESSION 2026-06-11 — FOCUS NARROWED · MARK 4.x + FIELD ONLY -->
<!-- ============================================================= -->

## Latest Session: 2026-06-11 — Scope Decisions

### TL;DR for FIELD

1. **Fady (PocketBase experiment) — archived.** The `fady-app` repo (https://github.com/ahmedashraf-cyber/fady-app) is read-only as of 2026-06-11. Did not impact FIELD directly.
2. **MARK 5.0.0 architecture migration — cancelled.** This DOES affect the Firebase-quota strategy that was outlined in the 2026-06-09 session below. The "MARK 5.0.0 drops Firestore usage ~99%" assumption no longer holds. If quota becomes a problem, the path is now Firebase Blaze upgrade rather than re-architecting MARK.
3. **MARK has been brought to version 4.1.0** (2026-06-11). New errors-taxonomy data file added (`src/data/tagging_scenarios.js`), two new events added (Pass Interception, Card). See MARK_CONTEXT.md in the mark-app repo for full details.
4. **Going forward: only two repos, both stay.** mark-app and flowops. No experiments. No third repo.
5. **FIELD pending work, unchanged:** Training Supervisor role, Trainee role (per system prompt for the project).

### Quota strategy update

Without MARK 5.0.0's quota cuts, the assumed cushion in the 2026-06-09 session is smaller. Practically:

- **If reviewers get blocked again by quota:** upgrade Firebase to Blaze (pay-as-you-go) — costs are small for a small team. See the Firebase upgrade guidance that was discussed on 2026-06-11 (process documented in conversation; not in MD form).
- **FIELD-side optimizations remain on the table** when convenient (cache batch/session lists, reduce dashboard listeners, push static reference data to Google Sheets) — see CALCULATOR.md for usage estimation.

### Hard scope rules (set 2026-06-11)

- Don't change the current MARK structure unless explicitly approved.
- Don't introduce a third repo or third backend.
- Sheet-style data sources are the source of truth (matches in Google Sheets, error taxonomy in `src/data/tagging_scenarios.js` from `Untitled_spreadsheet (1).xlsx`).

---

<!-- ============================================================= -->
<!-- SESSION 2026-06-09 - FIREBASE QUOTA + INFRASTRUCTURE DECISIONS -->
<!-- ============================================================= -->

## Latest Session: 2026-06-09 - Firebase Quota Wall + Big Picture Architecture

### TL;DR (FIELD-relevant)

- Firebase Firestore quota EXHAUSTED today across the project shared with MARK
  - 125K reads (2.5x over 50K/day free limit)
  - 20K writes (at the 20K/day limit, cap hit)
- Reviewers were blocked from working when quota ran out
- Root cause is mostly in MARK (bridge self-amplifying loop) - not FIELD
- FIELD dashboards still contribute meaningfully to read load
- Major architectural changes planned in MARK (5.0.0) that drop usage ~99%
- Bigger conversation pending about whether to leave Firebase entirely

### What Happened Today (FIELD perspective)

1. Reviewer complained that MARK would not let them start a review session
2. Firebase console showed "Quota Exceeded" error from Firestore
3. Investigation showed the 125K read / 20K write usage on the chart
4. FIELD dashboard reads identified as a meaningful contributor:
   - Training Manager dashboard loads all batches + sessions + users on each open
   - Each role page (Coordinator, Supervisor, Trainer) reads from its collections
   - No client-side caching means every navigation = fresh reads
5. MARK bridge identified as the primary culprit (~60% of waste)

### Solution Path

#### Phase 1 - Ship MARK 5.0.0 (next session)
- Drops Firestore usage ~99% by eliminating bridge writes/reads entirely
- After this, free tier comfortably handles ~100 reviewers
- See ahmedashraf-cyber/mark-app/SYNC_PROBLEM_CONTEXT.md for full details

#### Phase 2 - Optimize FIELD reads (later, when needed)
- Cache user/batch/match lists client-side
- Move static reference data to Google Sheets (matches already there, can extend to users/batches)
- Reduce snapshot listeners on large collections

#### Phase 3 - Bigger Decision Pending (no commitment yet)
- Whether to migrate FIELD off Firebase entirely
- Options discussed in MARK's SYNC_PROBLEM_CONTEXT.md (Infrastructure Discussion section)
- User position: $0 budget, framed as personal experiment helping the team

### Decisions Made

OK Ship MARK 5.0.0 - reduces Firestore load 99% - PRIORITY
PENDING Keep Firebase Auth for now (cheap, handles infinite users on free tier)
PENDING Optimize FIELD dashboards in a later release (5.1.0?)
PENDING Whether to migrate off Firebase (PocketBase / Supabase / Oracle Free Tier all considered)

### Infrastructure Options Discussed (see MARK docs for full detail)

- Supabase free tier - drop-in Firebase replacement, generous limits, pauses on inactivity
- PocketBase self-hosted - single binary, full ownership, needs always-on server
- Google Sheets + Apps Script - already works (proven today for MARK session results), too slow for realtime
- Oracle Cloud Free Tier - 2 free VMs forever, painful signup
- Hetzner / DigitalOcean / Vultr - paid VPS rental, EUR 4-6/mo
- Stay on Firebase + MARK 5.0.0 - the path of least change, currently recommended

### Files in This Repo (current state, unchanged today)

- README.md
- CONTEXT.md (this file)
- CALCULATOR.md (Firebase usage calculator)
- index.html (the FIELD app itself)

### Files Modified Today

- None directly in this repo (bash environment was broken throughout the session)
- This documentation was pushed via Claude in Chrome browser extension at session end

### Reviewers' Status Right Now

- Firebase quota will auto-reset at midnight Pacific time (~10am Cairo)
- Until then, reviewers can NOT start new sessions
- Tomorrow morning they resume normally
- MARK 5.0.0 next session permanently eliminates this risk

### Next Session

Command to start: "ship MARK 5.0.0"

After MARK 5.0.0 ships, FIELD itself is fine - no urgent changes needed.

---

# FIELD â Project Context File
**Last updated:** June 2026  
**Purpose:** Full handover document for any new chat session. Read this top to bottom and you will know everything â what was built, why, every decision made, all access credentials, current state, and what comes next.

> ### ð© SESSION HANDOFF â READ FIRST
> **FIELD** is a 7-role training-ops platform (single `index.html`, Firebase, ~600KB) â
> all role gates are built and live. **MARK** (its companion Windows review app) is also
> built and shipping at **v2.2.0**, integrated with FIELD through the same Firebase project.
>
> The defining technical win of this project: **MARK â collection-app video sync is SOLVED**
> (see Section 20, and `SYNC_PROBLEM_CONTEXT.md` in the MARK repo). ~20 approaches failed before
> the breakthrough â a Firestore bridge injected into the collection app â eliminated the
> mouse-click problem entirely. Zero clicks, both videos move together, error tagging flows
> live into FIELD.
>
> **Tokens:** the GitHub tokens shared in past chats were regenerated and are dead. Use fresh
> tokens; never commit a token into these docs. **Treat every credential in a chat as temporary.**
>
> Everything below is the full record. The project is in a strong, working state.

---

## 1. What Is This Project?

**FIELD** is a Training Operations Management Platform for **Hudl Egypt's training department**. It manages the full lifecycle of training batches â from interview/acceptance through supervised training to completion.

It is a **single HTML file** (`index.html`) deployed via GitHub Pages. No build tools, no framework, no backend. Everything runs in the browser using Firebase Auth + Firestore, Google Sheets API, EmailJS, and Chart.js.

**Live app:** https://ahmedashraf-cyber.github.io/flowops/index.html  
**GitHub repo:** https://github.com/ahmedashraf-cyber/flowops  
**GitHub token:** `ASK_AHMED_DIRECTLY`  
**Working file:** `/home/claude/flowops_final.html` (always clone fresh at session start)  
**Push directory:** `/home/claude/flowops_push/`

---

## 2. Session Setup (Run Every Time)

```bash
cd /home/claude && git clone https://ahmedashraf-cyber:TOKEN@github.com/ahmedashraf-cyber/flowops.git flowops_push
cp /home/claude/flowops_push/index.html /home/claude/flowops_final.html
cd /home/claude/flowops_push && git config user.email "ahmed.ashraf@hudl.com" && git config user.name "Ahmed Ashraf"
```

After every edit, check JS syntax before pushing:
```bash
node -e "
const html = require('fs').readFileSync('/home/claude/flowops_final.html', 'utf8');
const scripts = [...html.matchAll(/<script(?![^>]*src)[^>]*>([\s\S]*?)<\/script>/g)];
const js = scripts[scripts.length-1][1];
try { new Function(js); console.log('â Clean'); }
catch(e) { console.log('â', e.message); const lines=js.split('\n'); for(let i=0;i<lines.length;i++){try{new Function(lines.slice(0,i+1).join('\n'));}catch(e2){if(e2.message===e.message){console.log('L'+(i+1)+':',lines[i].substring(0,100));break;}}} }
"
```

Push:
```bash
cp /home/claude/flowops_final.html /home/claude/flowops_push/index.html
cd /home/claude/flowops_push && git add index.html && git commit -m "your message" && git push origin main
```

After pushing: tell user to wait for green on GitHub Pages then **Ctrl+Shift+R**.

---

## 3. Tech Stack & Credentials

### Firebase
- **Project:** `hudl-training-ops`
- **API Key:** `ASK_AHMED_DIRECTLY`
- **Auth Domain:** `hudl-training-ops.firebaseapp.com`
- **Project ID:** `hudl-training-ops`
- Uses: Firebase Auth (email/password) + Firestore (NoSQL database)

### Google Sheets API
- **API Key:** `ASK_AHMED_DIRECTLY`
- **Interview Sheet ID:** `190Zih7R1HswY2yVxl4WanfH-QhGlt4X8bOBteFrNtiY` (tabs: Interview Score 1/2/3, batch col=r[17], status=col L includes 'accept')
- **Matches Sheet ID:** `1dPwnYhIOiLUy_aBuVijPH3xtU6kxnEu-8FF115kXjSc`
- **Team/Trainers Sheet ID:** `1bErhs3yQiJMl6PXRJFgH512wLgfm2dM6Cpj2owimLuw` (tab: "Trainers", columns: A=HR Code, B=Name, C=Role, D=Type, E=Mail, F=Passwords) â **must be publicly shared (Anyone with link â Viewer)**

### Design System
- **Accent color:** `#E8590C` (orange), CSS var: `--p2`
- **Font:** Inter (headings) + DM Sans (body) + JetBrains Mono (codes)
- **Theme:** Dark-first Apple-style
- **Dropdown system:** All selects use `nxWrapSelect(sel, onChangeFn)` â NEVER use plain `<select>` visible to user
- **Screens:** All use `class="screen"` + `app-shell` grid layout
- **`nxSelectValue(id)`** â reads nx-wrapped select value
- **`nxDateInit()`** â wraps date inputs
- **`animateTabIn(el)`** â inner page transition (0.18s fadeIn+slideUp) â call on every tab switch
- **Gate transitions:** `switchRole()` handles cinematic fade 0.25s out â 0.35s in
- **Inner page transitions:** `animateTabIn()` handles 0.18s fade+slide â lighter/faster than gate transitions

---

## 4. Roles & Their Gates

| Role | Firestore value | Screen | Status |
|------|----------------|--------|--------|
| Batch Coordinator | `coordinator` | `coordScreen` | â Complete |
| Batch Supervisor | `batchsup` | `bsupScreen` | â Complete |
| Batch Trainer | `trainer` | `trainerScreen` | â Complete |
| Batch Manager | `batchmanager` | `bmScreen` | â Complete |
| Training Supervisor | `trainsup` | pending | ð² Not built |
| Training Manager | `manager` | `coordScreen` + role switcher | â Complete |
| Trainee | `trainee` | pending | ð² Not built |

### Role Switcher (Training Manager only)
- Fixed panel bottom-right (`id="roleSwitcher"`, `position:fixed`, `z-index:99999`)
- **Batch Supervisor** â dropdown of all BM_SUPS â `switchRoleAsSup(member)` â views as that supervisor
- **Batch Trainer** â dropdown fetched live from Trainers sheet â `switchRoleAsTrainer(trainer)` â views as that trainer
- **No banners** â identity card on first page of each gate is enough (removed blue/green banners)
- **Exit view** buttons return to Training Manager
- **âï¸ Team Account Setup** at bottom â `showSetupScreen()`

---

## 5. Team Roster (Hardcoded in JS)

```javascript
const TEAM_ROSTER = [
  {code:'A-024', name:'Fady Mamdouh Maurice Abdelnour',        role:'batchmanager',  email:'fady.mamdouh@hudl.com'},
  {code:'A-031', name:'Ahmed Atef Atteya Abdelmageed',         role:'batchmanager',  email:'ahmed.atef@hudl.com'},
  {code:'A-043', name:'Mohamed Elsayed Abdelhafiz Shamseldin', role:'batchsup',       email:'mohamed.elsayed@hudl.com'},
  {code:'A-047', name:'Tarek Mohamed Elsayed Mohamed Mahmoud', role:'batchsup',       email:'tarek.muhammed@hudl.com'},
  {code:'A-070', name:'Omar Mohamed Ali Mahmoud',              role:'batchsup',       email:'omar.attia@hudl.com'},
  {code:'A-082', name:'Ahmed Adel Abdellateif Mourad',         role:'coordinator',    email:'ahmed.morad@hudl.com'},
  {code:'A-246', name:'Mohab Mohamed Ramadan Abosaada',        role:'batchsup',       email:'mohab.mohamed@hudl.com'},
  {code:'A-248', name:'Mahmoud Reda Saleh Abdullah',           role:'batchsup',       email:'mahmoud.reda@hudl.com'},
  {code:'A-275', name:'Ahmed Adel Salah Farghaly',             role:'batchsup',       email:'ahmed.farghaly@hudl.com'},
  {code:'A-317', name:'Ali Mohamed Ali Ahmed',                 role:'batchsup',       email:'ali.mohamed@hudl.com'},
  {code:'A-551', name:'Ahmed Magdy Mohamed Abdelhamid',        role:'batchsup',       email:'ahmed.magdymohamed@hudl.com'},
  {code:'A-774', name:'Mostafa Saad Eldin Abdelrahman Ahmed',  role:'batchsup',       email:'mustafa.saad@hudl.com'},
  {code:'A-1589',name:'Abdelrahman Ali Hamed Saber Ahmed',     role:'batchsup',       email:'abdelrahman.hamed@hudl.com'},
];
const BM_SUPS = TEAM_ROSTER.filter(m => m.role === 'batchsup').map(m => m.code);
```

**Batch Trainers** â NOT hardcoded. Fetched live from Trainers tab of Team Sheet.
Sheet columns: A=HR Code, B=Name, C=Role, D=Type, E=Mail (index 4), F=Password (index 5).
Function: `fetchTrainersFromSheet()` â cached per session in `_trainersCache`.
Cache cleared by setting `_trainersCache = null`.

**Helper functions:** `getRosterMember(code)`, `getSupName(code)`, `getSupByEmail(email)`

---

## 6. Firestore Collections

| Collection | Purpose |
|-----------|---------|
| `batch_notifications` | Coordinator end-of-batch events + task alerts. `type:'end_of_batch'` or `type:'task'` |
| `accepted_trainees/sup_{code}` | BM-assigned trainees per supervisor with trainingCodes. Doc ID = `sup_A_551` etc. |
| `training_status/{trainingCode}` | Training status per trainee code |
| `group_assignments/batch_{label}` | BSup group assignments (trainees â trainer groups) |
| `batch_trainer_assignments/{batchName}` | BM trainer assignments. Field: `assignments: {supCode: [trainers]}` |
| `bm_tasks` | BM task tickets |
| `bm_sup_assignments` | BM supervisor assignments history |
| `users/{uid}` | Firebase Auth profiles. Fields: role, supervisorCode (sups), trainerCode (trainers), displayName, email |

---

## 7. Data Flow (Full Pipeline)

```
1. Coordinator "End of Batch"
   â saves to batch_notifications (type:'end_of_batch', acceptedCandidates[])
   â reads from Interview Sheet (Google Sheets API)

2. BM Batches page
   â bmLoadAllData() reads batch_notifications
   â overlays supervisor + trainingCode from accepted_trainees
   â BM clicks "Generate Training Codes" â saves to training_status + accepted_trainees

3. BM Assign Supervisors (4-step flow)
   Step 1: Select batch
   Step 2: Drag supervisors from right panel â drop zone pills
   Step 3: Slider for trainees per supervisor + preview per sup
   Step 4: Auto-distribute trainers (5-6 trainees each) â editable pills + add dropdown (nx-wrapped)
   Confirm â saves to: accepted_trainees/sup_{code}, batch_trainer_assignments/{batch}, bm_sup_assignments

4. BSup gate
   â loadBsupTrainees() reads ONLY accepted_trainees WHERE supervisorCode === A.user.supervisorCode
   â loadMyTrainersFromBM() reads batch_trainer_assignments for his trainers only
   â Group Assignment: auto-split trainees across his trainers, BSup adjusts

5. Trainer gate
   â btLoginWithCode(code) reads group_assignments
   â Shows only their assigned trainees
   â Auto-login if trainer signs in with their account (uses A.user.trainerCode from Firestore profile)
```

---

## 8. Key JS Architecture

### Global State Objects
```javascript
const A = { user: null, cands: [], batch: {} }; // App-wide auth state
// A.user fields: uid, email, name, role, supervisorCode, trainerCode, rosterCode
const BM = { trainees:[], tsStatuses:{}, filters:{}, tasks:[], _pendingTrainers:null, _allTrainers:null };
const BSUP = { trainees:[], myTrainers:[], groups:[], groupAssign:{} };
const BT = { trainerCode:'', allAssignments:[], _allTrainers:[] };
const TRAINER_CODES = []; // populated live from sheet
const TEAM_SHEET_ID = '1bErhs3yQiJMl6PXRJFgH512wLgfm2dM6Cpj2owimLuw';
let _trainersCache = null; // session cache for trainer list
```

### Critical Functions
- `fetchTrainersFromSheet()` â fetches live trainer list, cached per session
- `bmLoadAllData()` â loads all trainees for BM from Firestore
- `bmGenerateTrainingCodes()` â generates sequential codes, saves to Firestore
- `bmPreviewAssignment()` â previews sup distribution + triggers trainer Step 4
- `bmConfirmAssignment()` â saves everything to Firestore
- `bmLoadTrainerAssignment(sups, total)` â auto-distributes trainers, renders editable cards
- `bmRenderTrainerAssignment()` â renders trainer pills (uses DOM events NOT onclick attrs)
- `loadBsupTrainees()` â BSup reads their trainees from accepted_trainees
- `loadMyTrainersFromBM()` â BSup reads their assigned trainers from batch_trainer_assignments
- `initBsupDashboard()` â initializes full BSup gate + updates identity card
- `initTrainerDashboard()` â fetches trainers from sheet, populates picker
- `btLoginWithCode(code)` â logs trainer in, loads group, updates identity card
- `switchRoleAsSup(member)` â TM views as supervisor (no banner, identity card updates)
- `switchRoleAsTrainer(trainer)` â TM views as trainer (no banner, identity card updates)
- `renderSetupRoster()` â async, shows TEAM_ROSTER + fetches sheet trainers
- `runSetup()` â creates Firebase accounts for TEAM_ROSTER + sheet trainers
- `animateTabIn(el)` â smooth 0.18s page-in animation for inner tab switches
- `showScreen(id)` â navigates between screens
- `showTab(t)` â Coordinator inner tabs (calls animateTabIn)
- `bmShowTab(tab)` â BM inner tabs (calls animateTabIn)
- `showBsupTab(tab)` â BSup inner tabs (calls animateTabIn)
- `btShowPage(page)` â Trainer inner pages (calls animateTabIn)

### nxWrapSelect System (CRITICAL)
```javascript
nxWrapSelect(sel, onChangeFn)  // wraps plain <select> into dark glass panel
nxSelectValue(id)              // reads from wrap.dataset.nxValue first
nxDateInit()                   // wraps date inputs
```
All dropdowns MUST use this. Exception: dynamically-rendered selects use DOM addEventListener after render.

---

## 9. Identity Cards

Both BSup and Trainer first pages have an orange identity card:
- **BSup** (`id="bsup_identity_card"`) â code from `A.user.supervisorCode`, name from `getRosterMember(code).name` (NOT A.user.name which is TM's name when viewing as)
- **Trainer** (`id="bt_identity_card"`) â code + name from `BT._allTrainers` sheet data

Updated in `initBsupDashboard()` and `btLoginWithCode()`.

---

## 10. Transitions (Fully Reviewed & Fixed)

| Transition Type | Implementation | Duration | Status |
|----------------|---------------|----------|--------|
| Gate-to-gate (between roles) | `switchRole()` â opacity fade out then fade in | 0.25s out + 0.35s in | â |
| Inner tab switch (all gates) | `animateTabIn()` â @keyframes pageIn (opacity+translateY) | 0.18s | â Fixed |
| Coordinator tabs (`showTab`) | `animateTabIn()` | 0.18s | â Fixed |
| BM tabs (`bmShowTab`) | `animateTabIn()` | 0.18s | â Fixed |
| BSup tabs (`showBsupTab`) | `animateTabIn()` | 0.18s | â Fixed |
| Trainer pages (`btShowPage`) | `animateTabIn()` | 0.18s | â Fixed |

**Before the fix:** all inner tab switches were instant (display:none/block, no animation).
**Design principle:** gate transitions are cinematic and slower; inner page transitions are subtle and fast.

---

## 11. Recurring Bug Patterns (Always Check)

1. **`''+var+''` in onclick strings** â SyntaxError. Fix: use `data-*` attributes + `addEventListener`
2. **`btn.outerHTML`** â event listeners lost. Fix: use `appendChild`
3. **Duplicate `function` definitions** â last wins, earlier are dead
4. **`display:none` twice in inline style** â JS `.style.display='flex'` overridden
5. **Top-level access before `const` declaration** â ReferenceError
6. **`transform` on `.screen.active`** â traps `position:fixed` role switcher
7. **Black screen after navigation** â force `opacity:1` after `showScreen()`
8. **Regex in Python string replacements** â `\s` needs `\\s`, `\.` needs `\\.`
9. **`_trainersCache`** â clear with `_trainersCache = null` if column mapping changes
10. **Escaped quotes in JS template strings** â `'JetBrains Mono'` inside JS string = SyntaxError. Use `JetBrains Mono` without quotes inside JS-built HTML strings

---

## 12. Team Account Setup

- Accessible: Training Manager â Role Switcher â âï¸ Team Account Setup
- Renders TEAM_ROSTER immediately, fetches Trainers sheet asynchronously
- Trainers show with blue **TRAINER** badge
- `runSetup()` creates Firebase Auth for everyone (TEAM_ROSTER + sheet trainers)
- Trainer profiles: `trainerCode` field. Others: `supervisorCode` field
- Safe to run multiple times â skips existing accounts
- `_trainersCache = null` called before fetch to ensure fresh column data
- "Fix my role" on sign-in page â `applyRoleFix()` â updates Firestore role + code

---

## 13. What Is Complete â

- **Batch Coordinator gate** â full flow, end-of-batch, email notifications
- **Batch Manager gate** â 4 tabs:
  - Batches: view all trainees, edit status/supervisor/notes, generate training codes, prevent re-assign
  - Assign Supervisors: 4-step drag-drop with editable trainer assignment (nx-wrapped dropdowns)
  - Attendance: read-only
  - Tasks: create/edit/delete with real-time alerts
- **Batch Supervisor gate** â accepted trainees, group assignment, attendance, training status, match assignment, notifications. Identity card on first page.
- **Batch Trainer gate** â auto-login with trainerCode, sees only assigned group. Identity card on first page.
- **Training Manager gate** â coordinator view + role switcher (view as any sup or trainer from sheet)
- **Team Account Setup** â creates all accounts including trainers from sheet
- **Identity cards** â BSup and Trainer first pages show code + correct name
- **Live trainer fetch** â always from Google Sheet, never hardcoded
- **Smooth transitions** â gate transitions cinematic, inner tabs subtle (all fixed)
- **CONTEXT.md** â this file, in repo for session handover
- **MARK review app (v2.2.0)** â BUILT & LIVE. Video review + tornado error tagging + quality scores, integrated with FIELD via shared Firebase. Sync between MARK and the Statsbomb collection app SOLVED via the Firestore bridge (see Section 20 + `SYNC_PROBLEM_CONTEXT.md` in the MARK repo).

---

## 14. What Is Pending ð²

### Roles not yet built:
- **Training Supervisor** â "Soon" in role switcher
- **Trainee** â "Soon" in role switcher

### Firebase Optimization (DO THIS LAST â after all features are complete)
See Section 16 for full technical specification.

### ASHRAF (separate project â future)
See Section 17.

---

## 15. Important Decisions & Why

| Decision | Why |
|----------|-----|
| Single HTML file | No build tools, instant iteration, owner can inspect directly |
| Trainers from Google Sheet (not hardcoded) | New trainers added to sheet auto-appear without code changes |
| BM generates training codes (not BSup) | BM has full visibility of all batches, codes centrally managed |
| DOM `addEventListener` not `onclick="fn(arg)"` | Prevents SyntaxError when args contain special chars (codes like A-551) |
| Never use `btn.outerHTML` | Loses all event listeners on serialization |
| nxWrapSelect for all dropdowns | Consistent dark glass UI |
| No banners for view-as mode | Identity card on page is cleaner and sufficient |
| Inner tabs use animateTabIn (0.18s) not gate transition (0.35s) | Different feel: snappy inside a role, cinematic between roles |
| Identity card uses getRosterMember(code) not A.user.name | A.user.name is the TM's name when viewing as another supervisor |

---

## 16. Firebase Optimization â Full Technical Specification

### â ï¸ DO THIS LAST â only after all roles and features are complete

### Why It's Needed
Firebase free tier: 50,000 reads/day. Without optimization, worst-case day (batch transition, everyone active) = ~52,000 reads. With optimization = ~24,500 reads. Safely under limit for up to 10 simultaneous batches.

### Read Budget Analysis (after optimization)

| Scale | Normal Day | Worst Day | Status |
|-------|-----------|-----------|--------|
| 5 batches (200 trainees, 70 trainers, 20 sups) | ~6,800 | ~24,500 | â Safe |
| 10 batches | ~13,600 | ~49,000 | â Safe |
| 11 batches | ~14,960 | ~53,900 | ð´ Over daily |

**Reality check:** Batches are staggered, not simultaneous. 10 batches/quarter staggered = only 3-4 overlap at any time = ~19,600 worst day. Effectively unlimited headroom.

**Monthly (25 active days/month):**
- 5 batches: ~223,100/month = 15% of 1,500,000 monthly limit
- 10 batches: ~446,200/month = 30% of monthly limit

### Cache TTL Decisions (agreed)

| Data | Cache TTL | Reason |
|------|-----------|--------|
| Trainee lists | 1 minute | Changes occasionally |
| Group assignments | 1 minute | Changes occasionally |
| Training status | 1 minute | Changes rarely |
| Trainer assignments | 5 minutes | Almost never changes mid-day |
| Task alerts | Real-time (onSnapshot) | Must be instant |
| Notifications | Real-time (onSnapshot) | Must be instant |

### Exact Code Changes Required

#### Change 1: Session Cache Object
Add at top of JS, after global state objects:
```javascript
const CACHE = {
  _store: {},
  _ttls: { trainees: 60000, groups: 60000, status: 60000, trainerAssign: 300000 },
  set(key, data){
    this._store[key] = { data, ts: Date.now() };
  },
  get(key){
    const entry = this._store[key];
    if(!entry) return null;
    const ttl = this._ttls[key.split('_')[0]] || 60000;
    if(Date.now() - entry.ts > ttl){ delete this._store[key]; return null; }
    return entry.data;
  },
  clear(key){ if(key) delete this._store[key]; else this._store = {}; }
};
```

#### Change 2: Cache bmLoadAllData()
```javascript
async function bmLoadAllData(){
  const cacheKey = 'trainees_bm';
  const cached = CACHE.get(cacheKey);
  if(cached){
    BM.trainees = cached.trainees;
    BM.tsStatuses = cached.tsStatuses;
    bmRenderBatchTable();
    return;
  }
  // ... existing fetch code ...
  // At the end, before return:
  CACHE.set(cacheKey, { trainees: BM.trainees, tsStatuses: BM.tsStatuses });
}
```

#### Change 3: Direct doc read in loadBsupTrainees()
```javascript
// BEFORE (reads entire collection):
const snap = await fbDb.collection('accepted_trainees').get();

// AFTER (reads only this supervisor's doc):
const myCode = A.user.supervisorCode;
const docId = 'sup_' + myCode.replace(/-/g,'_');
const doc = await fbDb.collection('accepted_trainees').doc(docId).get();
// 1 read instead of 20+
```

#### Change 4: Cache loadMyTrainersFromBM()
```javascript
async function loadMyTrainersFromBM(){
  const cacheKey = 'trainerAssign_' + (A.user.supervisorCode||'');
  const cached = CACHE.get(cacheKey);
  if(cached){ BSUP.myTrainers = cached; return; }
  // ... existing fetch code ...
  CACHE.set(cacheKey, BSUP.myTrainers);
}
```

#### Change 5: Cache group assignments in loadSavedGroups()
```javascript
async function loadSavedGroups(){
  const cacheKey = 'groups_' + (A.user.supervisorCode||'');
  const cached = CACHE.get(cacheKey);
  if(cached){ BSUP.groups = cached; renderGroups(); return; }
  // ... existing fetch code ...
  CACHE.set(cacheKey, BSUP.groups);
}
```

#### Change 6: Cache training status reads
```javascript
async function initTrainingStatus(){
  const cacheKey = 'status_' + (A.user.supervisorCode||'');
  const cached = CACHE.get(cacheKey);
  if(cached){ /* use cached data */ renderTrainingStatus(cached); return; }
  // ... existing fetch code ...
  CACHE.set(cacheKey, statusData);
}
```

#### Change 7: Replace collection onSnapshot with targeted listeners
```javascript
// BEFORE (listens to entire collection â fires for ALL users):
fbDb.collection('batch_notifications').onSnapshot(snap => { ... });

// AFTER (listen only to docs relevant to this user):
fbDb.collection('batch_notifications')
  .where('type', '==', 'task')
  .onSnapshot(snap => { ... });
// Reduces reads on every notification fire from N_users to 1
```

#### Change 8: Add manual Refresh buttons
On BM Batches page and BSup Accepted Trainees page, add:
```javascript
function forceRefresh(){
  CACHE.clear(); // wipe all cache
  bmLoadAllData(); // or initBsupDashboard()
}
// HTML: <button onclick="forceRefresh()">ð Refresh</button>
```

#### Change 9: Lazy setup status check
```javascript
// BEFORE: checks all 61 accounts at once
// AFTER: only check accounts currently visible in viewport
const observer = new IntersectionObserver(entries => {
  entries.forEach(e => {
    if(e.isIntersecting){
      const code = e.target.dataset.code;
      checkSingleAccountStatus(code);
      observer.unobserve(e.target);
    }
  });
});
document.querySelectorAll('[data-code]').forEach(el => observer.observe(el));
```

### What Changes in User Experience
| Feature | Before | After |
|---------|--------|-------|
| Trainee list freshness | Always live | Updates every 1 min (or manual refresh) |
| Group assignment freshness | Always live | Updates every 1 min |
| Training status freshness | Always live | Updates every 1 min |
| Task alerts | Instant | Instant (unchanged) |
| Notifications | Instant | Instant (unchanged) |
| Page load speed | Network-dependent | Instant after first load |
| Offline resilience | Breaks | Shows last cached data |

---

## 17. ASHRAF â Future Separate Project

**What:** AI training coach that plays the Supporter role for Tornado Batches. Separate from FIELD.
**Status:** Planned, not started. Full spec exists (shared as docx in conversation).
**Key decisions made:**
- Separate GitHub repo (different audience â external trainees vs internal staff)
- Single HTML file approach (same as FIELD)
- Free at launch: Groq API (Llama) for brain, browser Web Speech for voice
- Upgrade path: Claude/GPT brain + ElevenLabs voice â config-only swap, no rewrite
- Hosted on GitHub Pages, public (no company account for trainees)
- Trainee access: public link + batch code
- Ahmed needs Groq API key (free at console.groq.com) before building
- Integration with FIELD planned after ASHRAF matures

**Build stages:**
1. Trainee login + chat (text first, voice second)
2. Supervisor dashboard + knowledge base editor
3. Quizzes, Final Checks, progress tracking, outlier detection
4. Shared Q&A board, live sessions, PWA

---

## 18. The Product Owner

**Ahmed Ashraf** â Training Manager at Hudl Egypt. Non-developer, Product Director.
Email: `ahmed.ashraf@hudl.com`
All decisions go through him. He tests on the live app after every push.
He will never upgrade to Firebase Blaze paid plan â all solutions must work within free tier.

---

## 19. Firebase Usage Calculator

See `CALCULATOR.md` in this repo for the interactive usage calculator.
Run it to estimate daily/monthly reads whenever a new role or feature is added.


---

## 20. MARK â Review Application (BUILT & LIVE â)

**Status:** Built, shipping, in use. Current version **v2.2.0**. Companion product to FIELD.

**Name origin:** Reviewers mark errors, mark moments; their marks define collector quality.

**GitHub:** `https://github.com/ahmedashraf-cyber/mark-app` (separate repo from FIELD)
**Releases:** auto-built Windows `.msi`/`.exe` via GitHub Actions on push to main
**Full sync history & architecture:** see `SYNC_PROBLEM_CONTEXT.md` in the MARK repo

### What MARK Is
A Windows desktop app (**Tauri 2** = Rust backend + React 19 in a WebView2 webview) for
Batch Trainers/Reviewers. It runs alongside the **Statsbomb Tag Once collection app**
(Electron, closed source) on the same machine. The reviewer watches match video in MARK,
tags errors using tornado event shortcuts, and â critically â **both videos stay in sync**
so the reviewer navigates once and sees the same frame in both apps.

### THE SYNC SOLUTION (the hard-won core â v2.1.0)
Syncing two independent Chromium apps where one is closed-source was the single hardest
problem of the whole project. ~20 approaches failed (all keystroke/focus injection methods
hit a wall: stealing focus to drive the collection app put MARK's own webview keyboard to
sleep, needing a physical mouse click every cycle). The collection app's remote-debugging
port is **disabled at build time**, killing all standard automation (CDP, electron-inject, etc.).

**The winning architecture â a Firestore bridge:**
```
Reviewer presses arrow in MARK
   â MARK writes navCommand to Firestore: mark_sessions/{sessionId}.navCommand = {action, shift, ts}
   â a "bridge script" running INSIDE the collection app listens via onSnapshot
   â bridge moves the collection app's video natively: document.querySelector('video').currentTime += step
```
Because the bridge runs *inside* the collection app, it controls the video with **zero focus
stealing and zero clicks.** MARK never touches the collection app's focus.

**Getting the bridge in:** MARK's â¡ **Inject Bridge** button (Rust `inject_bridge_script`)
focuses the collection app, opens its DevTools (Alt+Ctrl+I), types `allow pasting` via
**Unicode key injection** (Chromium treats Unicode-injected chars as *typed*, bypassing the
self-XSS paste guard), then pastes + runs the bridge script from the clipboard. Reviewer
clicks Inject Bridge once per session, signs into the bridge panel with their FIELD account
(Firebase caches it â later sessions skip login), green "Connected" panel appears, done.

### Reviewer experience polish (v2.1.1 / v2.2.0)
- After injecting, MARK auto-closes the collection app's DevTools â clean screen, just the video.
- The MARK Bridge panel is hidden by default; it only appears if sign-in is needed, then
  disappears once connected. Returning reviewers (cached login) never see it. Sync runs silently.
- MARK's top bar shows the live version: `MARK Â· Review App Â· vX.Y.Z`.

### Tech Stack
| Layer | Choice |
|-------|--------|
| Desktop shell | Tauri 2 (Windows), Rust backend |
| UI | React 19 + FIELD design tokens (dark, #E8590C, Inter + DM Sans + JetBrains Mono) |
| Auth | Firebase Auth â same accounts as FIELD, no second login |
| Database | Firebase Firestore â **same project as FIELD** (`hudl-training-ops`) |
| Sync | **Firestore bridge** (navCommand doc â injected listener in collection app) |
| Real-time | Firestore onSnapshot (same as FIELD) |

### Navigation Shortcuts (matched to the collection app â CORRECTED)
- `â` / `â`  : seek **400ms** forward/back
- `Shift + â` / `Shift + â` : seek **40ms** forward/back
- `Space` : play / pause
(These exact values match the collection app's own steps so the two videos stay aligned.)

### Error Tagging (stays in MARK's own window)
Tornado event shortcut keys open a tag modal; `Y` = Missing Event. Each tag â `mark_error_tags`.
Tagging, timeline, and quality score all live in MARK's window (writing to Firebase â FIELD).
Deliberately NOT moved into the bridge panel â tagging always worked; only sync was broken.

### Quality Score Formula
```
Quality Score (%) = 100 - ((Tagged Errors / Total Reviewed Events) Ã 100)
```
Higher = better. 100 = perfect.

### How MARK Connects To FIELD (the integration)
FIELD and MARK share one Firebase project, so data flows live between them:
- **Trainer gate:** Review Results tab + real-time session dashboard
- **Batch Supervisor gate:** live review dashboard, quality scores per reviewer
- **Batch Manager gate:** aggregate review scores across batches
- **Trainee profile:** their own quality scores and error history
- A MARK session opening/completing is a signal FIELD can use for daily-task detection

### Shared Firestore Collections (same Firebase project as FIELD)
| Collection | Purpose |
|-----------|---------|
| `mark_sessions` | Review sessions â match, half, reviewer, collector, status, scores. **Also carries `navCommand` for the sync bridge.** |
| `mark_error_tags` | Individual error tags â timestamp, type, extras, session ref |
| `mark_locks` | Half-level locks â `{matchId}_{half}` â reviewer |
| `mark_match_assignments` | Reviewer â match/collector assignments |

### Session Flow
1. Open collection app, log in, load video
2. Open MARK â Firebase reads identity from existing FIELD session â start session, load same video
3. Click â¡ **Inject Bridge** â bridge panel appears in collection app â sign in (first time only)
4. Navigate with arrows in MARK â **both videos seek together, no clicks**
5. Tag errors with tornado shortcuts (in MARK's window) â saved to `mark_error_tags`
6. Press Done â enter Total Reviewed Events â quality score â results appear in FIELD instantly

### Known Caveats (for future maintainers)
- The Inject Bridge auto-injection uses timed `SendInput` steps; on a slow machine the sleeps
  in `inject_bridge_windows` may need tuning. Fallback: paste `bridge_script.js` manually.
- Antivirus on locked-down corporate machines may flag the auto-DevTools-typing behavior.
- Collection app requires login each launch; cannot be auto-relaunched.
- Hard constraints (unchanged): do not modify the collection app's files; do not depend on
  the collection app dev team.
