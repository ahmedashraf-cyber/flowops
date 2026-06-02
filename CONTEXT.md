# FIELD — Project Context File
**Last updated:** June 2026  
**Purpose:** Full handover document for any new chat session. Read this top to bottom and you will know everything — what was built, why, every decision made, all access credentials, current state, and what comes next.

---

## 1. What Is This Project?

**FIELD** is a Training Operations Management Platform for **Hudl Egypt's training department**. It manages the full lifecycle of training batches — from interview/acceptance through supervised training to completion.

It is a **single HTML file** (`index.html`) deployed via GitHub Pages. No build tools, no framework, no backend. Everything runs in the browser using Firebase Auth + Firestore, Google Sheets API, EmailJS, and Chart.js.

**Live app:** https://ahmedashraf-cyber.github.io/flowops/index.html  
**GitHub repo:** https://github.com/ahmedashraf-cyber/flowops  
**GitHub token:** `ASK_AHMED_FOR_TOKEN`  
**Working file:** `/home/claude/flowops_final.html` (always clone fresh at session start)  
**Push directory:** `/home/claude/flowops_push/`

---

## 2. Session Setup (Run Every Time)

```bash
cd /home/claude && git clone https://ahmedashraf-cyber:ASK_AHMED_FOR_TOKEN@github.com/ahmedashraf-cyber/flowops.git flowops_push
cp /home/claude/flowops_push/index.html /home/claude/flowops_final.html
cd /home/claude/flowops_push && git config user.email "ahmed.ashraf@hudl.com" && git config user.name "Ahmed Ashraf"
```

After every edit, check JS syntax before pushing:
```bash
node -e "
const html = require('fs').readFileSync('/home/claude/flowops_final.html', 'utf8');
const scripts = [...html.matchAll(/<script(?![^>]*src)[^>]*>([\s\S]*?)<\/script>/g)];
const js = scripts[scripts.length-1][1];
try { new Function(js); console.log('✅ Clean'); }
catch(e) { console.log('❌', e.message); const lines=js.split('\n'); for(let i=0;i<lines.length;i++){try{new Function(lines.slice(0,i+1).join('\n'));}catch(e2){if(e2.message===e.message){console.log('L'+(i+1)+':',lines[i].substring(0,100));break;}}} }
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
- **API Key:** `ASK_AHMED_FOR_FIREBASE_KEY`
- **Auth Domain:** `hudl-training-ops.firebaseapp.com`
- **Project ID:** `hudl-training-ops`
- **Storage Bucket:** `hudl-training-ops.appspot.com`
- **Messaging Sender ID:** `in the HTML file`
- Uses: Firebase Auth (email/password) + Firestore (NoSQL database)

### Google Sheets API
- **API Key:** `ASK_AHMED_FOR_SHEETS_KEY`
- **Interview Sheet ID:** `190Zih7R1HswY2yVxl4WanfH-QhGlt4X8bOBteFrNtiY` (tabs: Interview Score 1/2/3, batch col=r[17], status=col L includes 'accept')
- **Matches Sheet ID:** `1dPwnYhIOiLUy_aBuVijPH3xtU6kxnEu-8FF115kXjSc`
- **Team/Trainers Sheet ID:** `1bErhs3yQiJMl6PXRJFgH512wLgfm2dM6Cpj2owimLuw` (tab: "Trainers", columns: A=HR Code, B=Name, C=Role, D=Type, E=Mail, F=Passwords)

### Design System
- **Accent color:** `#E8590C` (orange), CSS var: `--p2`
- **Font:** Inter (headings) + DM Sans (body) + JetBrains Mono (codes)
- **Theme:** Dark-first Apple-style
- **Dropdown system:** All selects use `nxWrapSelect(sel, onChangeFn)` — NEVER use plain `<select>` visible to user
- **Screens:** All use `class="screen"` + `app-shell` grid layout
- **`nxSelectValue(id)`** — reads nx-wrapped select value (reads `wrap.dataset.nxValue` first)
- **`nxDateInit()`** — wraps date inputs with dark calendar picker

---

## 4. Roles & Their Gates

| Role | Firestore value | Screen | Notes |
|------|----------------|--------|-------|
| Batch Coordinator | `coordinator` | `coordScreen` | Ends batches, sends notifications |
| Batch Supervisor | `batchsup` | `bsupScreen` | Manages their assigned trainees, group assignment |
| Batch Trainer | `trainer` | `trainerScreen` | Sees only their assigned group |
| Batch Manager | `batchmanager` | `bmScreen` | 4 tabs: Batches, Assign Supervisors, Attendance, Tasks |
| Training Supervisor | `trainsup` | *(pending)* | Not yet built |
| Training Manager | `manager` | `coordScreen` + role switcher | Super access, can view as any supervisor or trainer |
| Trainee | `trainee` | *(pending)* | Not yet built |

### Role Switcher (Training Manager only)
- Fixed panel bottom-right (`id="roleSwitcher"`, `position:fixed`, `z-index:99999`)
- **Batch Supervisor** button → dropdown of all BM_SUPS → `switchRoleAsSup(member)` → shows BSup gate as that supervisor
- **Batch Trainer** button → dropdown fetched live from Trainers sheet → `switchRoleAsTrainer(trainer)` → shows Trainer gate
- **Training Manager → Current** label
- **⚙️ Team Account Setup** at bottom → `showSetupScreen()`
- Exit view buttons return to Training Manager state

---

## 5. Team Roster (Hardcoded in JS)

```javascript
const TEAM_ROSTER = [
  {code:'A-024', name:'Fady Mamdouh Maurice Abdelnour',        role:'batchmanager',  email:'fady.mamdouh@hudl.com',        password:'ASK_AHMED'},
  {code:'A-031', name:'Ahmed Atef Atteya Abdelmageed',         role:'batchmanager',  email:'ahmed.atef@hudl.com',           password:'ASK_AHMED'},
  {code:'A-043', name:'Mohamed Elsayed Abdelhafiz Shamseldin', role:'batchsup',       email:'mohamed.elsayed@hudl.com',      password:'ASK_AHMED'},
  {code:'A-047', name:'Tarek Mohamed Elsayed Mohamed Mahmoud', role:'batchsup',       email:'tarek.muhammed@hudl.com',       password:'ASK_AHMED'},
  {code:'A-070', name:'Omar Mohamed Ali Mahmoud',              role:'batchsup',       email:'omar.attia@hudl.com',           password:'ASK_AHMED'},
  {code:'A-082', name:'Ahmed Adel Abdellateif Mourad',         role:'coordinator',    email:'ahmed.morad@hudl.com',          password:'ASK_AHMED'},
  {code:'A-246', name:'Mohab Mohamed Ramadan Abosaada',        role:'batchsup',       email:'mohab.mohamed@hudl.com',        password:'ASK_AHMED'},
  {code:'A-248', name:'Mahmoud Reda Saleh Abdullah',           role:'batchsup',       email:'mahmoud.reda@hudl.com',         password:'ASK_AHMED'},
  {code:'A-275', name:'Ahmed Adel Salah Farghaly',             role:'batchsup',       email:'ahmed.farghaly@hudl.com',       password:'ASK_AHMED'},
  {code:'A-317', name:'Ali Mohamed Ali Ahmed',                 role:'batchsup',       email:'ali.mohamed@hudl.com',          password:'ASK_AHMED'},
  {code:'A-551', name:'Ahmed Magdy Mohamed Abdelhamid',        role:'batchsup',       email:'ahmed.magdymohamed@hudl.com',   password:'ASK_AHMED'},
  {code:'A-774', name:'Mostafa Saad Eldin Abdelrahman Ahmed',  role:'batchsup',       email:'mustafa.saad@hudl.com',         password:'ASK_AHMED'},
  {code:'A-1589',name:'Abdelrahman Ali Hamed Saber Ahmed',     role:'batchsup',       email:'abdelrahman.hamed@hudl.com',    password:'ASK_AHMED'},
];
const BM_SUPS = TEAM_ROSTER.filter(m => m.role === 'batchsup').map(m => m.code);
```

**Batch Trainers** are NOT hardcoded — fetched live from the Trainers tab of the Team Sheet. Columns: A=HR Code, B=Name, C=Role, D=Type, E=Mail (index 4), F=Password (index 5).

**Helper functions:** `getRosterMember(code)`, `getSupName(code)`, `getSupByEmail(email)`

---

## 6. Firestore Collections

| Collection | Purpose |
|-----------|---------|
| `batch_notifications` | Coordinator end-of-batch events + task alerts. `type:'end_of_batch'` or `type:'task'` |
| `accepted_trainees/sup_{code}` | BM-assigned trainees per supervisor (with trainingCodes). Doc ID = `sup_A_551` etc. |
| `training_status/{trainingCode}` | Training status per trainee code |
| `group_assignments/batch_{label}` | BSup group assignments (trainees → trainer groups) |
| `batch_trainer_assignments/{batchName}` | BM trainer assignments per batch. Field `assignments: {supCode: [trainers]}` |
| `bm_tasks` | BM task tickets |
| `bm_sup_assignments` | BM supervisor assignments history |
| `users/{uid}` | Firebase Auth profiles. Fields: role, supervisorCode (for sups), trainerCode (for trainers), displayName, email |

---

## 7. Data Flow (Full Pipeline)

```
1. Coordinator "End of Batch"
   → saves to batch_notifications (type:'end_of_batch', acceptedCandidates[])
   → reads from Interview Sheet (Google Sheets API)

2. BM Batches page
   → bmLoadAllData() reads batch_notifications
   → overlays supervisor + trainingCode from accepted_trainees
   → BM clicks "Generate Training Codes" → saves to training_status + accepted_trainees

3. BM Assign Supervisors (4-step flow)
   Step 1: Select batch
   Step 2: Drag supervisors from right panel → drop zone pills
   Step 3: Slider for trainees per supervisor + preview per sup
   Step 4: Auto-distribute trainers (5-6 trainees each) → editable pills + add dropdown
   → bmConfirmAssignment() saves to:
     - accepted_trainees/sup_{code} (trainees per sup, with codes)
     - batch_trainer_assignments/{batch} (trainers per sup)
     - bm_sup_assignments

4. BSup gate
   → loadBsupTrainees() reads ONLY from accepted_trainees WHERE supervisorCode === A.user.supervisorCode
   → loadMyTrainersFromBM() reads batch_trainer_assignments for his trainers only
   → Group Assignment: auto-split trainees across his trainers, BSup adjusts

5. Trainer gate
   → btLoginWithCode(code) reads group_assignments
   → Shows only their assigned trainees
   → Auto-login if trainer signs in with their own account (uses A.user.trainerCode)
```

---

## 8. Key JS Architecture

### Global State Objects
```javascript
const A = { user: null, cands: [], batch: {} }; // App-wide auth state
const BM = { trainees:[], tsStatuses:{}, filters:{}, tasks:[], _pendingTrainers:null, _allTrainers:null };
const BSUP = { trainees:[], myTrainers:[], groups:[], groupAssign:{} };
const BT = { trainerCode:'', allAssignments:[], _allTrainers:[] };
```

### Critical Functions
- `fetchTrainersFromSheet()` — fetches live trainer list from Google Sheet, cached per session
- `bmLoadAllData()` — loads all trainees for BM from Firestore
- `bmGenerateTrainingCodes()` — generates sequential codes, saves to Firestore
- `bmPreviewAssignment()` — previews sup distribution + triggers trainer Step 4
- `bmConfirmAssignment()` — saves everything to Firestore
- `bmLoadTrainerAssignment(sups, total)` — auto-distributes trainers, renders editable cards
- `bmRenderTrainerAssignment()` — renders trainer pills with remove + add dropdown (uses DOM events, NOT onclick attrs)
- `loadBsupTrainees()` — BSup reads their trainees from accepted_trainees
- `loadMyTrainersFromBM()` — BSup reads their assigned trainers from batch_trainer_assignments
- `initBsupDashboard()` — initializes full BSup gate
- `initTrainerDashboard()` — fetches trainers from sheet, populates picker
- `btLoginWithCode(code)` — logs trainer in, loads their group, updates identity card
- `switchRoleAsSup(member)` — TM views as supervisor
- `switchRoleAsTrainer(trainer)` — TM views as trainer
- `renderSetupRoster()` — async, shows TEAM_ROSTER + fetches sheet trainers
- `runSetup()` — creates Firebase accounts for all TEAM_ROSTER + sheet trainers
- `showScreen(id)` — navigates between screens (sets opacity:1 after display:flex to prevent black screen)

### nxWrapSelect System (CRITICAL — never bypass)
```javascript
nxWrapSelect(sel, onChangeFn)  // wraps plain <select> into dark glass panel
nxSelectValue(id)              // reads from wrap.dataset.nxValue first, fallback to el.value
nxDateInit()                   // wraps date inputs
```
All dropdowns MUST use this system. The only exception is selects inside dynamically-rendered JS strings that use DOM `addEventListener` after render (like `bm-tr-add` class selects in trainer assignment).

---

## 9. Identity Cards (Recently Added)

Both BSup and Trainer first pages have an orange identity card at the top:
- **BSup** (`id="bsup_identity_card"`) — shows supervisor code + name from TEAM_ROSTER (NOT A.user.name which is TM's name)
- **Trainer** (`id="bt_identity_card"`) — shows trainer code + name from sheet

Updated in `initBsupDashboard()` and `btLoginWithCode()` respectively.

---

## 10. Recurring Bug Patterns (Always Check)

1. **`''+var+''` in onclick strings** → `SyntaxError`. Fix: use `data-*` attributes + `addEventListener`
2. **`btn.outerHTML`** → event listeners lost. Fix: use `appendChild`
3. **Duplicate `function` definitions** — last wins, earlier are dead. Check before push.
4. **`display:none` twice in inline style** → JS `.style.display='flex'` overridden. Fix: clean style string first
5. **Top-level access before `const` declaration** → ReferenceError
6. **`transform` on `.screen.active`** → traps `position:fixed` role switcher
7. **`showScreen()` sets all screens to `display:none`** → black screen. Fix: force `opacity:1` after navigation
8. **Regex in JS strings** — `\s` and `\.` need to be written as `\\s` and `\\.` in Python string replacements but plain in JS. When using Python to replace JS code, be careful with escape sequences.
9. **`_trainersCache`** — trainer list is cached per session. If sheet column mapping changes, call `_trainersCache = null` before re-fetching.
10. **Black screen after navigation** — always check `showScreen()` sets `opacity:1` on the target screen.

---

## 11. Team Account Setup Screen

Accessible from: Training Manager → Role Switcher → ⚙️ Team Account Setup

- Renders TEAM_ROSTER immediately, then fetches Trainers sheet asynchronously
- Trainers appear below with blue **TRAINER** badge
- "Create All Accounts" (`runSetup()`) creates Firebase Auth accounts for everyone
- Trainer profiles get `trainerCode` field; others get `supervisorCode`
- Safe to run multiple times — skips existing accounts
- "Fix my role" link on sign-in page → `applyRoleFix()` → updates Firestore role + code from roster

---

## 12. What Is Complete ✅

- **Batch Coordinator gate** — full flow, end-of-batch, email notifications
- **Batch Manager gate** — 4 tabs fully working:
  - Batches: view all trainees, edit status/supervisor/notes, generate training codes
  - Assign Supervisors: 4-step drag-drop flow with trainer assignment (editable)
  - Attendance: read-only view
  - Tasks: create/edit/delete tickets with real-time alerts
- **Batch Supervisor gate** — accepted trainees, group assignment (assigns trainers to groups), attendance, training status, match assignment, notifications
- **Batch Trainer gate** — login with code (or auto-login), sees only assigned group
- **Training Manager gate** — coordinator view + role switcher (view as any sup or trainer)
- **Team Account Setup** — creates all accounts including trainers from sheet
- **Identity cards** — BSup and Trainer first pages show code + name
- **Live trainer fetch** — trainers always read from Google Sheet, never hardcoded

---

## 13. What Is Pending 🔲

### Roles not yet built:
- **Training Supervisor** — labeled "Soon" in role switcher
- **Trainee** — labeled "Soon" in role switcher

### ASHRAF (separate project — future):
A separate AI training coach product (separate GitHub repo) for the Tornado Batches training program. Will be built as a standalone app first, then potentially integrated into FIELD later. Key decisions made:
- Single HTML file approach (same as FIELD)
- Free tier at launch: Groq API (Llama) for brain, browser Web Speech for voice
- Upgrade path: Claude/GPT brain + ElevenLabs voice, config-only swap
- Hosted on GitHub Pages, public (no company account for trainees)
- Trainee access: public link + batch code
- Ahmed needs a Groq API key (free at console.groq.com) before building starts

### Known minor issues to revisit:
- The "Assign Supervisors" page doesn't reset Step 4 trainer cards when a new batch is selected (minor UX)
- BSup Group Assignment auto-split logic could be smarter (currently random, not tier-balanced)

---

## 14. Important Decisions & Why

| Decision | Why |
|----------|-----|
| Single HTML file | No build tools, no deployment pipeline, instant iteration, owner can inspect directly |
| Firebase Auth | Team already has accounts; secure; free tier sufficient |
| Trainers from Google Sheet (not hardcoded) | New trainers added to sheet auto-appear in app without code changes |
| BM generates training codes (not BSup) | BM has full visibility of all batches; codes should be centrally managed |
| Trainer assignment saved to `batch_trainer_assignments` (separate from `accepted_trainees`) | Cleaner separation; trainer assignments can be updated independently of trainee assignments |
| DOM `addEventListener` instead of `onclick="fn(arg)"` | Prevents SyntaxError when args contain special characters (codes like `A-551`) |
| Never use `btn.outerHTML` | `outerHTML` serializes the element and loses all event listeners |
| nxWrapSelect for all dropdowns | Consistent dark glass UI; plain browser selects look wrong in the design |
| `_trainersCache = null` in `renderSetupRoster` | Ensures fresh column mapping after the column fix (was fetching wrong columns before) |

---

## 15. The Product Owner

**Ahmed Ashraf** — Training Manager at Hudl Egypt. Non-developer, acts as Product Director. Works with this codebase through Claude. All decisions go through him. He tests on the live app after every push.

Email: `ahmed.ashraf@hudl.com`

---

*This file lives at `/CONTEXT.md` in the flowops GitHub repo. Any new chat session should read it via:*
```
https://raw.githubusercontent.com/ahmedashraf-cyber/flowops/main/CONTEXT.md
```
