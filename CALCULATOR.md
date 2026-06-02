# FIELD — Firebase Usage Calculator

Use this every time a new role, feature, or scale change is added to estimate daily and monthly Firestore reads.

---

## How To Use

1. Fill in your current numbers in the **Inputs** section
2. Add any new features to the **Features** section with their read cost
3. Run the calculation
4. Check against the **Limits** section

---

## Firebase Free Tier Limits

| Limit | Amount | Notes |
|-------|--------|-------|
| Daily reads | 50,000 | Hard limit — resets midnight Pacific time |
| Daily writes | 20,000 | Hard limit |
| Daily deletes | 20,000 | Hard limit |
| Monthly storage | 1 GB | Very unlikely to hit |
| Auth users | 10,000/month | Very unlikely to hit |

---

## Current Scale Inputs (update these)

```
BATCH_MANAGERS     = 2
BATCH_SUPERVISORS  = 20
BATCH_TRAINERS     = 70
COORDINATORS       = 1
TRAINING_MANAGERS  = 1
TOTAL_STAFF        = 94   (sum of above)
TRAINEES_PER_BATCH = 200  (doesn't use FIELD directly but data is read)
SIMULTANEOUS_BATCHES = 5  (realistic overlap at any one time)
ACTIVE_DAYS_PER_MONTH = 25
TRANSITION_DAYS_PER_MONTH = 3  (days when new batches start)
NORMAL_DAYS_PER_MONTH = 22
```

---

## Read Cost Per Action (current features)

Update this table whenever a new feature is added.

### WITHOUT Optimization (current state)

| Action | Reads per trigger | Triggers/day | Daily reads |
|--------|------------------|--------------|-------------|
| BM opens Batches page | TRAINEES + SUPERVISORS + TRAINERS (~270) | 8 | 2,160 |
| BM opens Assign page | TRAINEES + SUPERVISORS (~220) | 5 | 1,100 |
| BSup opens gate (reads entire accepted_trainees) | SUPERVISORS (20) | 4 per sup | 1,600 |
| Trainer opens gate | 15 | 3 per trainer | 3,150 |
| Real-time listener fires | TOTAL_STAFF (94) | 20 | 1,880 |
| Group assignment reads | TRAINEES / BATCH (~100) | 15 | 1,500 |
| Training status reads | TRAINEES (~200) | 8 | 1,600 |
| Task alerts | TOTAL_STAFF (94) | 15 | 1,410 |
| **TOTAL NORMAL DAY** | | | **~14,400** |
| **TOTAL WORST DAY (transition)** | | | **~52,000** |

### WITH Optimization (after implementing Section 16 of CONTEXT.md)

| Action | Reads per trigger | Triggers/day | Daily reads |
|--------|------------------|--------------|-------------|
| BM opens Batches (cached 1 min) | 270 first load, 0 after | ~2 cache misses/day | 540 |
| BM opens Assign (cached 1 min) | 220 first load, 0 after | ~1 cache miss/day | 220 |
| BSup opens gate (direct doc read) | 1 (not 20) | 4 per sup | 80 |
| Trainer opens gate (cached) | 3 | 1 cache miss/day | 210 |
| Real-time listeners (targeted) | ~5 | 20 fires | 100 |
| Group assignments (cached 1 min) | 10 first load, 0 after | ~2 misses/day | 20 |
| Training status (cached 1 min) | 50 first load, 0 after | ~2 misses/day | 100 |
| Task alerts (real-time, unchanged) | 94 | 15 | 1,410 |
| **TOTAL NORMAL DAY** | | | **~2,680** |
| **TOTAL WORST DAY** | | | **~24,500** |

---

## Formula for New Features

When adding a new feature, estimate its read cost using this formula:

```
DAILY_READS = READS_PER_TRIGGER × TRIGGERS_PER_USER_PER_DAY × USERS_WHO_USE_IT

Example — new "Trainee Progress" page for BSup:
  READS_PER_TRIGGER = 200 (reads all trainees' progress docs)
  TRIGGERS_PER_USER = 5 (BSup opens it 5 times/day)
  USERS = 20 (all supervisors)
  DAILY_READS = 200 × 5 × 20 = 20,000 ← significant! needs caching
```

### Decision Rules for New Features

| Estimated daily reads | Action required |
|-----------------------|----------------|
| < 1,000 | Safe to add as-is |
| 1,000 - 5,000 | Add to optimization plan (cache it) |
| 5,000 - 15,000 | Must cache before deploying to full scale |
| > 15,000 | Redesign the data fetch (targeted reads, pagination) |

---

## Calculation Worksheet

Copy this and fill in whenever evaluating new feature or scale:

```
=== FIELD Firebase Read Estimate ===

INPUTS:
  Batch Managers:      ___
  Batch Supervisors:   ___
  Batch Trainers:      ___
  Coordinators:        ___
  Training Managers:   ___
  Total Staff:         ___
  Trainees per batch:  ___
  Simultaneous batches: ___
  
EXISTING FEATURES (daily reads):
  Current baseline (optimized): ~2,680/day
  OR current baseline (not optimized): ~14,400/day
  
NEW FEATURE COSTS:
  Feature 1: ___ reads/trigger × ___ triggers/user × ___ users = ___/day
  Feature 2: ___ reads/trigger × ___ triggers/user × ___ users = ___/day
  Feature 3: ___ reads/trigger × ___ triggers/user × ___ users = ___/day
  
NEW ROLE COSTS:
  Role name: ___ users × ___ reads/session × ___ sessions/day = ___/day

TOTALS:
  Normal day:     ___/day  (target: < 30,000)
  Worst day:      ___/day  (hard limit: < 50,000)
  Monthly (25 days): ___   (hard limit: < 1,500,000)
  
STATUS:
  [ ] Safe — proceed
  [ ] Caution — add caching before full rollout
  [ ] Danger — redesign data fetch
```

---

## Cost of Pending Roles (Estimate)

### Training Supervisor (not yet built)
Assuming similar to Batch Supervisor:
- ~40 reads/session, ~3 sessions/day, ~5 supervisors
- **Daily addition: ~600 reads** — safe to add as-is

### Trainee Role (not yet built)
Trainees will read their own data only (1 doc per trainee):
- ~10 reads/session, ~3 sessions/day, ~200 trainees
- **Daily addition: ~6,000 reads** — needs caching when added

### Combined impact of all roles added:
```
Current (optimized):      2,680/day
+ Training Supervisor:      600/day
+ Trainee role:           6,000/day
─────────────────────────────────────
Total (all roles):        9,280/day  ← still well under 50,000 ✅
Worst day estimate:      ~35,000/day ← still under 50,000 ✅
```

---

## Quick Reference: Current Status

```
TODAY (without optimization):
  Normal day:  ~14,400 reads  ✅ under 50,000
  Worst day:   ~52,000 reads  🔴 over limit on batch transition days

AFTER OPTIMIZATION (do this last):
  Normal day:   ~2,680 reads  ✅ extremely safe
  Worst day:   ~24,500 reads  ✅ safe
  
  Safe for: up to 10 simultaneous batches
  Or realistically: 40+ batches/quarter (staggered, 3-4 overlap at once)
```

