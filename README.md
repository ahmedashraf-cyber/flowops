# FIELD — Training Operations Management

> Single-file web app · Firebase Auth + Firestore · Google Sheets API · EmailJS · Chart.js
> Live: https://ahmedashraf-cyber.github.io/flowops/index.html

FIELD is the training-operations platform for **Hudl Egypt**. It manages the
full lifecycle of training batches — coordination, supervision, training,
interviews, matches, and performance — and is the dashboard side of the same
operation that the **[MARK](https://github.com/ahmedashraf-cyber/mark-app)**
desktop review app feeds into. Quality scores produced by reviewers in MARK
surface here against the people and batches they belong to.

---

## Roles

FIELD is role-based; each role gets its own screens and permissions. The
Training Manager role includes a role switcher for oversight.

| Role | Status |
|---|---|
| Batch Coordinator | ✅ complete |
| Batch Supervisor | ✅ complete |
| Batch Trainer | ✅ complete |
| Training Manager (role switcher) | ✅ complete |
| Training Supervisor | ⏳ pending |
| Trainee | ⏳ pending |

---

## Architecture

- **One file.** The entire app is `index.html` (~10.8k lines). No build step;
  GitHub Pages serves it directly.
- **Auth + data:** Firebase Authentication and Cloud Firestore (project
  `hudl-training-ops`, shared with MARK).
- **External data:** Google Sheets API (matches sheet, interview sheet — batch
  in `r[17]`, status in column L containing "accept").
- **Notifications:** EmailJS.
- **Charts:** Chart.js for dashboards.

### Design system (Dark-first, Apple-style)

- **Accent:** `#E8590C` (orange).
- **Type:** Inter (headings) · DM Sans (body) · JetBrains Mono (codes).
- **Dropdowns:** all use the shared `nx-select-wrap` system — do not hand-roll
  selects.
- **Layout:** every screen uses `class="screen"` inside the `app-shell` grid.

> ⚠️ Do not modify the CSS variables or the `nx-select` dropdown system without
> checking the existing code first — many screens depend on them.

---

## Documentation

| File | Contents |
|---|---|
| [CONTEXT.md](./CONTEXT.md) | Running session log — scope decisions, Firebase-quota history, architecture notes |
| [CALCULATOR.md](./CALCULATOR.md) | Firestore usage estimation (reads/writes) and FIELD-side optimisation options |

---

## Operational notes

- **Two repos only:** `flowops` (FIELD) and `mark-app` (MARK). No third repo, no
  third backend.
- **Source of truth for reference data:** Google Sheets (matches) and the error
  taxonomy file in MARK (`src/data/tagging_scenarios.js`).
- **Firebase quota** is shared with MARK. If reviewers get blocked by quota, the
  agreed path is upgrading Firebase to Blaze (pay-as-you-go); FIELD-side
  optimisations (caching batch/session lists, reducing dashboard listeners) are
  available when convenient — see CALCULATOR.md.

---

## Relationship to MARK

FIELD and MARK share the `hudl-training-ops` Firebase project. Reviewers tag
collector errors in MARK; the resulting quality scores and session records are
written to Firestore and read by FIELD's dashboards. See the MARK repo's
`CHANGELOG.md` and `DECISIONS.md` for the review-side logic (including the
audit-quality model that determines those scores).
