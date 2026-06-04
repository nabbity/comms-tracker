# Fomo Energy — Commissions Tracker
## Full Spec & Handover Document

**Status:** POC (HTML/JS, single file)  
**Built by:** Claude (Sonnet 4.6) with Dionne Sim  
**Date:** June 2026  
**File:** `index.html` — open in any browser, no server or install needed

---

## 1. What this is

A commissions payout tracker for the Fomo Energy residential sales team. It calculates how much each salesperson is owed on the 20th of each month, based on a rolling 17th-to-17th window.

This replaces a shared Excel sheet that had two problems:
1. **Overwrite / mis-filter risk** — people on the same sheet corrupted each other's data
2. **Logic bug** — the old formula checked whether the *payment date* fell in the window, which caused projects with late 2-pager approvals to fall through the cracks and never get paid

---

## 2. Payout schedule

| Window | Pay date |
|---|---|
| 17 May → 17 Jun | 20 Jun |
| 17 Jun → 17 Jul | 20 Jul |
| … | … |

The window is always **17th of month N to 17th of month N+1**, paid on the **20th of month N+1**.

---

## 3. Commission rules

### Three criteria (all three must be met)
1. Customer payment received (downpayment or balance payment)
2. Financial Model (FM) approved by manager
3. 2-pager approved by manager

### Commission amounts
```
DP commission    = Dev margin × DP%   × 60%
Balance commission = Dev margin × (1 − DP%) × 60%
```

- **Dev margin** = development margin in S$, pulled from the financial model
- **DP%** = the split agreed with the salesperson (typically 40%)
- **60%** = sales team's share of the dev margin (Fomo keeps 40%)

### Trigger date logic (the fix vs. old Excel)

The commission is counted in the window where **all three criteria are finally met**, not where the payment was made.

```
Trigger date (DP)      = MAX(dp_payment_date, fm_approval_date, twopager_approval_date)
Trigger date (Balance) = MAX(bal_payment_date, fm_approval_date, twopager_approval_date)
```

A commission row is included in a window if:
```
trigger_date >= window_from AND trigger_date <= window_to
```

**Example of the edge case this fixes:**

| Event | Date |
|---|---|
| DP paid by customer | 23 Apr |
| FM approved | 25 Apr |
| 2-pager approved (late) | 17 May |
| **Trigger date** | **17 May** |
| **Window** | **17 May → 17 Jun** |
| **Paid** | **20 Jun** |

Under the old Excel formula, this project would never appear because the DP date (23 Apr) was outside the current window.

---

## 4. Data model

Each project record has the following fields:

| Field | Type | Notes |
|---|---|---|
| `id` | string (UUID) | Auto-generated |
| `project` | string | Project name / address |
| `salesperson` | string | One of the 6 sales team members |
| `dev_margin` | number (S$) | Development margin from FM |
| `dp_pct` | number (0–100) | DP split percentage |
| `dp_date` | date or null | Date customer made downpayment |
| `bal_date` | date or null | Date customer made balance payment |
| `fm_ok` | boolean | FM approved by manager |
| `fm_date` | date or null | Date FM was approved |
| `twopager_ok` | boolean | 2-pager approved by manager |
| `twopager_date` | date or null | Date 2-pager was approved |
| `notes` | string | Free text |

**Computed fields (not stored):**
- `dp_comm` = dev_margin × (dp_pct/100) × 0.6
- `bal_comm` = dev_margin × (1 − dp_pct/100) × 0.6
- `dp_trigger` = MAX(dp_date, fm_date, twopager_date) — null if any are missing
- `bal_trigger` = MAX(bal_date, fm_date, twopager_date) — null if any are missing

---

## 5. Sales team

```
Pandora Chew
Dionne Sim
Lew Jen Sern
Hans Lee
Zeng Guowei
Ler Yee Yoong
```

All team members share the same commission rate (60% of dev margin).

---

## 6. App structure (POC)

Single file: `index.html` (HTML + embedded CSS + embedded JS, ~400 lines)  
Data storage: `localStorage` key `fomo_comms_v1` (JSON array of project records)

### Tabs

#### This Month
- Date range picker (defaults to current 17–17 window based on today's date)
- Summary cards: grand total + per-salesperson totals for the window
- Table: all projects with a trigger date in the window, showing type (DP/Balance), trigger date breakdown, commission amount
- Export to CSV

#### All Projects
- Full table of all projects
- Filter by salesperson and by status (all criteria met / pending / no payment)
- Edit any project inline via modal
- Add new projects
- Export to CSV

#### Pending
- Projects where at least one payment has been made but FM or 2-pager approval is missing
- Shows what's outstanding (FM / 2-pager / both)
- Shows commission amounts at stake
- Management view: "this person is missing comms because X"

### Add/Edit modal
- All fields editable
- FM and 2-pager date fields auto-populate with today's date when checkbox is ticked
- Commission preview updates live as margin and DP% are typed
- Delete button on edit (with confirmation)

---

## 7. Edge cases handled

| Scenario | Behaviour |
|---|---|
| 2-pager approved after payment — in a later window | Trigger date moves to approval date; commission counted in correct later window |
| FM or 2-pager not approved | Commission not counted in any window; project appears in Pending tab |
| DP% = 100% | Balance commission = S$0; no balance row appears |
| DP% = 0% | DP commission = S$0; all commission on balance payment |
| Balance payment made but FM not done | Both DP and balance commission held; project stays in Pending |
| FM and 2-pager done but no payment received | No commission counted; project does not appear in Pending (no payment = not yet a liability) |
| Project has both DP and balance — each may fall in different windows | Each is evaluated independently with its own trigger date |

---

## 8. Integration notes (for tech team)

### Data ownership — what lives where

| Field | Owner | Notes |
|---|---|---|
| DP date, balance date | Sales tracker | Read into Commissions Tracker via sync/API |
| Dev margin | Sales tracker (FM record) | Pulled when FM is linked |
| Salesperson | Sales tracker | |
| **FM approved + date** | **Commissions Tracker** | Approver action lives here, not in the sales tracker |
| **2-pager approved + date** | **Commissions Tracker** | Approver action lives here, not in the sales tracker |

**Key decision:** FM and 2-pager approvals are performed inside the Commissions Tracker by the approving manager. The sales tracker is read-only input for payment data. Do not build approval flows in the sales tracker.

### Technical notes

1. **Sales tracker sync**: Payment data (DP date, balance date, dev margin, salesperson) should sync into the Commissions Tracker automatically — either via webhook on record update, or a scheduled pull. The Commissions Tracker should not require manual data entry for these fields.

2. **Approval write-back** (optional): If the sales tracker needs to know that FM/2-pager are approved, write back `fm_ok` and `fm_date` after the approver acts in the Commissions Tracker. This is optional — the Commissions Tracker is the source of truth for these fields.

3. **Commission rate**: Currently hardcoded at 60% in the JS (`COMM_RATE = 0.6`). Should be a config value in production.

4. **Window logic**: The `currentWindow()` function computes the active 17–17 window based on today's date. This is stateless and can be ported as-is.

5. **Trigger date function**: The core logic is three lines. Port this exactly — it is the fix for the Excel bug:
   ```javascript
   function dpTrigger(p) {
     if (!p.dp_date || !p.fm_ok || !p.twopager_ok || !p.fm_date || !p.twopager_date) return null;
     return Math.max(new Date(p.dp_date), new Date(p.fm_date), new Date(p.twopager_date));
   }
   ```

6. **Data migration**: The POC ships with 31 seed projects from the real sales tracker (May–Jun 2026 era). These are in the `seedData()` function and can be used to validate the integration output matches the POC output.

7. **Recommended fields to add in production**:
   - `project_id` — link to the main sales tracker record
   - `hubspot_deal_id` — for CRM linkage
   - `created_at`, `updated_at` timestamps
   - Audit log of approval actions (who approved, when, from which device)

---

## 9. How to run the POC

1. Open `index.html` in any modern browser (Chrome, Edge, Safari)
2. No internet connection required
3. Data is saved automatically to the browser's localStorage
4. To reset to seed data: open browser DevTools → Application → Local Storage → delete `fomo_comms_v1`
5. To share data between machines: use the Export CSV buttons and re-import manually (or wait for the production integration)

---

## 10. Known limitations of the POC

- Data lives in localStorage on one machine — not shared between team members (by design, to avoid the Excel overwrite problem)
- No authentication — anyone with the file can edit
- FM and 2-pager approval dates are manually entered — no audit trail
- No historical payout records — once a window passes, there's no "mark as paid" state
- No duplicate detection

These are all intentional scope decisions for the POC. The production integration should address all of them.
