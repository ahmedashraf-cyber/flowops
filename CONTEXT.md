# FIELD — Project Context File
**Last updated:** June 2026  
**Purpose:** Full handover document for any new chat session. Read this top to bottom and you will know everything — what was built, why, every decision made, all access credentials, current state, and what comes next.

---

## 1. What Is This Project?

**FIELD** is a Training Operations Management Platform for **Hudl Egypt's training department**. It manages the full lifecycle of training batches — from interview/acceptance through supervised training to completion.

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
- **API Key:** `ASK_AHMED_DIRECTLY`
- **Auth Domain:** `hudl-training-ops.firebaseapp.com`
- **Project ID:** `hudl-training-ops`
- Uses: Firebase Auth (email/password) + Firestore (NoSQL database)

### Google Sheets API
- **API Key:** `ASK_AHMED_DIRECTLY`
- **Interview Sheet ID:** `190Zih7R1HswY2yVxl4WanfH-QhGlt4X8bOBteFrNtiY` (tabs: Interview Score 1/2/3, batch col=r[17], status=col L includes 'accept')
- **Matches Sheet ID:** `1dPwnYhIOiLUy_aBuVijPH3xtU6kxnEu-8FF115kXjSc`
- **Team/Trainers Sheet ID:** `1bErhs3yQiJMl6PXRJFgH512wLgfm2dM6Cpj2owimLuw` (tab: "Trainers", columns: A=HR Code, B=Name, C=Role, D=Type, E=Mail, F=Passwords) — **must be publicly shared (Anyone with link → Viewer)**

### Design System
- **Accent color:** `#E8590C` (orange), CSS var: `--p2`
- **Font:** Inter (headings) + DM Sans (body) + JetBrains Mono (codes)
- **Theme:** Dark-first Apple-style
- **Dropdown system:** All selects use `nxWrapSelect(sel, onChangeFn)` — NEVER use plain `<select>` visible to user
- **Screens:** All use `class="screen"` + `app-shell` grid layout
- **`nxSelectValue(id)`** — reads nx-wrapped select value
- **`nxDateInit()`** — wraps date inputs
- **`animateTabIn(el)`** — inner page transition (0.18s fadeIn+slideUp) — call on every tab switch
- **Gate transitions:** `switchRole()` handles cinematic fade 0.25s out → 0.35s in
- **Inner page transitions:** `animateTabIn()` handles 0.18s fade+slide — lighter/faster than gate transitions

---

## 4. Roles & Their Gates

| Role | Firestore value | Screen | Status |
|------|----------------|--------|--------|
| Batch Coordinator | `coordinator` | `coordScreen` | ✅ Complete |
| Batch Supervisor | `batchsup` | `bsupScreen` | ✅ Complete |
| Batch Trainer | `trainer` | `trainerScreen` | ✅ Complete |
| Batch Manager | `batchmanager` | `bmScreen` | ✅ Complete |
| Training Supervisor | `trainsup` | pending | 🔲 Not built |
| Training Manager | `manager` | `coordScreen` + role switcher | ✅ Complete |
| Trainee | `trainee` | pending | 🔲 Not built |

### Role Switcher (Training Manager only)
- Fixed panel bottom-right (`id="roleSwitcher"`, `position:fixed`, `z-index:99999`)
- **Batch Supervisor** → dropdown of all BM_SUPS → `switchRoleAsSup(member)` → views as that supervisor
- **Batch Trainer** → dropdown fetched live from Trainers sheet → `switchRoleAsTrainer(trainer)` → views as that trainer
- **No banners** — identity card on first page of each gate is enough (removed blue/green banners)
- **Exit view** buttons return to Training Manager
- **⚙️ Team Account Setup** at bottom → `showSetupScreen()`

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

**Batch Trainers** — NOT hardcoded. Fetched live from Trainers tab of Team Sheet.
Sheet columns: A=HR Code, B=Name, C=Role, D=Type, E=Mail (index 4), F=Password (index 5).
Function: `fetchTrainersFromSheet()` — cached per session in `_trainersCache`.
Cache cleared by setting `_trainersCache = null`.

**Helper functions:** `getRosterMember(code)`, `getSupName(code)`, `getSupByEmail(email)`

---

## 6. Firestore Collections

| Collection | Purpose |
|-----------|---------|
| `batch_notifications` | Coordinator end-of-batch events + task alerts. `type:'end_of_batch'` or `type:'task'` |
| `accepted_trainees/sup_{code}` | BM-assigned trainees per supervisor with trainingCodes. Doc ID = `sup_A_551` etc. |
| `training_status/{trainingCode}` | Training status per trainee code |
| `group_assignments/batch_{label}` | BSup group assignments (trainees → trainer groups) |
| `batch_trainer_assignments/{batchName}` | BM trainer assignments. Field: `assignments: {supCode: [trainers]}` |
| `bm_tasks` | BM task tickets |
| `bm_sup_assignments` | BM supervisor assignments history |
| `users/{uid}` | Firebase Auth profiles. Fields: role, supervisorCode (sups), trainerCode (trainers), displayName, email |

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
   Step 4: Auto-distribute trainers (5-6 trainees each) → editable pills + add dropdown (nx-wrapped)
   Confirm → saves to: accepted_trainees/sup_{code}, batch_trainer_assignments/{batch}, bm_sup_assignments

4. BSup gate
   → loadBsupTrainees() reads ONLY accepted_trainees WHERE supervisorCode === A.user.supervisorCode
   → loadMyTrainersFromBM() reads batch_trainer_assignments for his trainers only
   → Group Assignment: auto-split trainees across his trainers, BSup adjusts

5. Trainer gate
   → btLoginWithCode(code) reads group_assignments
   → Shows only their assigned trainees
   → Auto-login if trainer signs in with their account (uses A.user.trainerCode from Firestore profile)
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
- `fetchTrainersFromSheet()` — fetches live trainer list, cached per session
- `bmLoadAllData()` — loads all trainees for BM from Firestore
- `bmGenerateTrainingCodes()` — generates sequential codes, saves to Firestore
- `bmPreviewAssignment()` — previews sup distribution + triggers trainer Step 4
- `bmConfirmAssignment()` — saves everything to Firestore
- `bmLoadTrainerAssignment(sups, total)` — auto-distributes trainers, renders editable cards
- `bmRenderTrainerAssignment()` — renders trainer pills (uses DOM events NOT onclick attrs)
- `loadBsupTrainees()` — BSup reads their trainees from accepted_trainees
- `loadMyTrainersFromBM()` — BSup reads their assigned trainers from batch_trainer_assignments
- `initBsupDashboard()` — initializes full BSup gate + updates identity card
- `initTrainerDashboard()` — fetches trainers from sheet, populates picker
- `btLoginWithCode(code)` — logs trainer in, loads group, updates identity card
- `switchRoleAsSup(member)` — TM views as supervisor (no banner, identity card updates)
- `switchRoleAsTrainer(trainer)` — TM views as trainer (no banner, identity card updates)
- `renderSetupRoster()` — async, shows TEAM_ROSTER + fetches sheet trainers
- `runSetup()` — creates Firebase accounts for TEAM_ROSTER + sheet trainers
- `animateTabIn(el)` — smooth 0.18s page-in animation for inner tab switches
- `showScreen(id)` — navigates between screens
- `showTab(t)` — Coordinator inner tabs (calls animateTabIn)
- `bmShowTab(tab)` — BM inner tabs (calls animateTabIn)
- `showBsupTab(tab)` — BSup inner tabs (calls animateTabIn)
- `btShowPage(page)` — Trainer inner pages (calls animateTabIn)

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
- **BSup** (`id="bsup_identity_card"`) — code from `A.user.supervisorCode`, name from `getRosterMember(code).name` (NOT A.user.name which is TM's name when viewing as)
- **Trainer** (`id="bt_identity_card"`) — code + name from `BT._allTrainers` sheet data

Updated in `initBsupDashboard()` and `btLoginWithCode()`.

---

## 10. Transitions (Fully Reviewed & Fixed)

| Transition Type | Implementation | Duration | Status |
|----------------|---------------|----------|--------|
| Gate-to-gate (between roles) | `switchRole()` — opacity fade out then fade in | 0.25s out + 0.35s in | ✅ |
| Inner tab switch (all gates) | `animateTabIn()` — @keyframes pageIn (opacity+translateY) | 0.18s | ✅ Fixed |
| Coordinator tabs (`showTab`) | `animateTabIn()` | 0.18s | ✅ Fixed |
| BM tabs (`bmShowTab`) | `animateTabIn()` | 0.18s | ✅ Fixed |
| BSup tabs (`showBsupTab`) | `animateTabIn()` | 0.18s | ✅ Fixed |
| Trainer pages (`btShowPage`) | `animateTabIn()` | 0.18s | ✅ Fixed |

**Before the fix:** all inner tab switches were instant (display:none/block, no animation).
**Design principle:** gate transitions are cinematic and slower; inner page transitions are subtle and fast.

---

## 11. Recurring Bug Patterns (Always Check)

1. **`''+var+''` in onclick strings** → SyntaxError. Fix: use `data-*` attributes + `addEventListener`
2. **`btn.outerHTML`** → event listeners lost. Fix: use `appendChild`
3. **Duplicate `function` definitions** — last wins, earlier are dead
4. **`display:none` twice in inline style** → JS `.style.display='flex'` overridden
5. **Top-level access before `const` declaration** → ReferenceError
6. **`transform` on `.screen.active`** → traps `position:fixed` role switcher
7. **Black screen after navigation** — force `opacity:1` after `showScreen()`
8. **Regex in Python string replacements** — `\s` needs `\\s`, `\.` needs `\\.`
9. **`_trainersCache`** — clear with `_trainersCache = null` if column mapping changes
10. **Escaped quotes in JS template strings** — `'JetBrains Mono'` inside JS string = SyntaxError. Use `JetBrains Mono` without quotes inside JS-built HTML strings

---

## 12. Team Account Setup

- Accessible: Training Manager → Role Switcher → ⚙️ Team Account Setup
- Renders TEAM_ROSTER immediately, fetches Trainers sheet asynchronously
- Trainers show with blue **TRAINER** badge
- `runSetup()` creates Firebase Auth for everyone (TEAM_ROSTER + sheet trainers)
- Trainer profiles: `trainerCode` field. Others: `supervisorCode` field
- Safe to run multiple times — skips existing accounts
- `_trainersCache = null` called before fetch to ensure fresh column data
- "Fix my role" on sign-in page → `applyRoleFix()` → updates Firestore role + code

---

## 13. What Is Complete ✅

- **Batch Coordinator gate** — full flow, end-of-batch, email notifications
- **Batch Manager gate** — 4 tabs:
  - Batches: view all trainees, edit status/supervisor/notes, generate training codes, prevent re-assign
  - Assign Supervisors: 4-step drag-drop with editable trainer assignment (nx-wrapped dropdowns)
  - Attendance: read-only
  - Tasks: create/edit/delete with real-time alerts
- **Batch Supervisor gate** — accepted trainees, group assignment, attendance, training status, match assignment, notifications. Identity card on first page.
- **Batch Trainer gate** — auto-login with trainerCode, sees only assigned group. Identity card on first page.
- **Training Manager gate** — coordinator view + role switcher (view as any sup or trainer from sheet)
- **Team Account Setup** — creates all accounts including trainers from sheet
- **Identity cards** — BSup and Trainer first pages show code + correct name
- **Live trainer fetch** — always from Google Sheet, never hardcoded
- **Smooth transitions** — gate transitions cinematic, inner tabs subtle (all fixed)
- **CONTEXT.md** — this file, in repo for session handover

---

## 14. What Is Pending 🔲

### Roles not yet built:
- **Training Supervisor** — "Soon" in role switcher
- **Trainee** — "Soon" in role switcher

### Firebase Optimization (DO THIS LAST — after all features are complete)
See Section 16 for full technical specification.

### ASHRAF (separate project — future)
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

## 16. Firebase Optimization — Full Technical Specification

### ⚠️ DO THIS LAST — only after all roles and features are complete

### Why It's Needed
Firebase free tier: 50,000 reads/day. Without optimization, worst-case day (batch transition, everyone active) = ~52,000 reads. With optimization = ~24,500 reads. Safely under limit for up to 10 simultaneous batches.

### Read Budget Analysis (after optimization)

| Scale | Normal Day | Worst Day | Status |
|-------|-----------|-----------|--------|
| 5 batches (200 trainees, 70 trainers, 20 sups) | ~6,800 | ~24,500 | ✅ Safe |
| 10 batches | ~13,600 | ~49,000 | ✅ Safe |
| 11 batches | ~14,960 | ~53,900 | 🔴 Over daily |

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
// BEFORE (listens to entire collection — fires for ALL users):
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
// HTML: <button onclick="forceRefresh()">🔄 Refresh</button>
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

## 17. ASHRAF — Future Separate Project

**What:** AI training coach that plays the Supporter role for Tornado Batches. Separate from FIELD.
**Status:** Planned, not started. Full spec exists (shared as docx in conversation).
**Key decisions made:**
- Separate GitHub repo (different audience — external trainees vs internal staff)
- Single HTML file approach (same as FIELD)
- Free at launch: Groq API (Llama) for brain, browser Web Speech for voice
- Upgrade path: Claude/GPT brain + ElevenLabs voice — config-only swap, no rewrite
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

**Ahmed Ashraf** — Training Manager at Hudl Egypt. Non-developer, Product Director.
Email: `ahmed.ashraf@hudl.com`
All decisions go through him. He tests on the live app after every push.
He will never upgrade to Firebase Blaze paid plan — all solutions must work within free tier.

---

## 19. Firebase Usage Calculator

See `CALCULATOR.md` in this repo for the interactive usage calculator.
Run it to estimate daily/monthly reads whenever a new role or feature is added.

