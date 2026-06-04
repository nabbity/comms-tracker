# Commissions Tracker — POC

A single-file HTML/JS web app that calculates monthly sales commission payouts for the Fomo Energy residential sales team.

Built to replace a shared Excel sheet that had a logic bug causing late-approved projects to fall through the cracks and never get paid.

---

## The problem it solves

The old Excel formula checked whether the **payment date** fell in the current 17–17 window. If the 2-pager was approved late (after the payment date), the project was never picked up in any future window.

**The fix:** commission is counted in the window where **all three criteria are finally met**, not where the payment was made.

```
Trigger date = MAX(payment date, FM approval date, 2-pager approval date)
```

---

## How to run

Open `index.html` in any modern browser (Chrome, Edge, Safari). No server, no install, no internet required. Data is saved to the browser's localStorage.

To reset to seed data: DevTools → Application → Local Storage → delete `fomo_comms_v1`.

---

## Files

| File | Purpose |
|---|---|
| `index.html` | The app — all HTML, CSS, and JS in one file |
| `COMMISSIONS_TRACKER_SPEC.md` | Full spec and integration notes for the tech team |
| `000 Sales Tracker 20260515 for testing.xlsx` | Source data used to build and validate seed records |

---

## Commission rules

- **Payout window:** 17th of month N → 17th of month N+1, paid on the 20th
- **Three criteria:** customer payment received + FM approved + 2-pager approved
- **Rate:** Dev margin × DP% × 60% (DP commission) and Dev margin × (1−DP%) × 60% (balance commission)
- **Trigger date:** MAX of all three criteria dates — applied to both DP and balance

---

## Status

POC. Data lives in localStorage on one machine (intentional — avoids Excel overwrite problem). Production integration to be handled by the tech team using `COMMISSIONS_TRACKER_SPEC.md` as the spec.

---

## Sales team

Pandora Chew · Dionne Sim · Lew Jen Sern · Hans Lee · Zeng Guowei · Ler Yee Yoong
