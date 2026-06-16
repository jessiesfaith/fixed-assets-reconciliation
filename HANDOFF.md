# Fixed Assets Reconciliation — Handoff

Session-state snapshot for the next person/agent. Detailed repo facts live in `CLAUDE.md`; this file
is "where things stand right now."

_Last updated: 2026-06-16._

## TL;DR
Full-cycle **fixed-assets (PP&E) reconciliation** dashboard — a single self-contained `index.html`
(vanilla HTML/CSS/JS, no build, no CDN deps beyond Google Fonts). **Live, deployed.** Built to mirror the
inventory & AR tools' SOX/recon pattern, extended for the things that are FA-specific (two roll-forwards,
CIP, existence/tagging, impairment, book-vs-tax, leases).

## Live
- **Standalone:** https://fixed-assets-reconciliation.vercel.app
- **Branded:** https://app.fastinsights.io/fixed-assets-reconciliation/ (tile on the app.fastinsights.io picker —
  added to AR-Tool-Beta `vercel.json` + `Landing.tsx`, shipped in commit `0f01e10`).
- **Deploy:** **git-connected** — `git push origin main` (via the `github-jessica` SSH alias) → Vercel
  auto-deploys. Manual alt: `npx vercel deploy --prod --yes`. Vercel project `fixed-assets-reconciliation`,
  account `jessica-dougherty-s-projects`, GitHub `jessiesfaith/fixed-assets-reconciliation`.

## What it does (24 tabs, 10 sidebar groups)
> Added after the initial 20-tab build: **Intangibles**, **ARO (ASC 410)**, **Held for Sale (ASC 360-10)**, and
> **Consolidation & FX** (3-entity current-rate translation + intercompany elimination ledger + CTA; sample
> consolidated NBV **$266.34M**). localStorage key bumped to `fa_recon_v2`.
1. **Reconciliation** — the signature view: **gross-cost roll-forward + CIP roll-forward + accumulated-
   depreciation roll-forward → net book value**, then subledger→GL→physical-existence tie-out. Sample net
   book value ties to **$182,310,000** (`varNetSub = 0`).
2. Additions / Capex · 3. CIP / In-Progress · 4. Capitalize vs Expense (policy)
5. Asset Register (subledger) · 6. Physical Verification (existence/tagging) · 7. Locations & Custodians
8. Depreciation · 9. Impairment & Revaluation (ASC 360) · 10. Useful Lives & Components
11. Disposals & Retirements (gain/loss + authorization) · 12. Transfers & Reclasses
13. Leases — ROU (ASC 842): ROU asset + lease liability roll-forwards
14. GL Transactions (register + chart-of-accounts→tab cross-ref + per-entry approval)
15. Capex Liabilities (capex AP/accrual roll-forward + retention) · 16. Approvals (DOA/AFE + disposal auth)
17. Tax Depreciation (book vs tax/MACRS, bonus, §179, deferred tax) · 18. Insurance & Property Tax
19. Roll-forward Analytics (capex/NBV trend, MoM/QoQ/YTD/YoY) · 20. Review & Export (sign-off, lock, CSV/MD)

## Key mechanics (for the next change)
- **Engine:** `compute()` derives both control roll-forwards from `state.txns` (journal). `TXN_TYPES`
  carries `book` ('cost'|'cip'|'accum') + `sign`; `@class` postings use `t.acct`. The FA subledger control
  (`subCost/subCIP/subAccum`) is set to tie to GL by construction; `physNet` carries the existence variance.
- **Two tie colors:** teal `.rf-tied` = cost/CIP roll-forwards; pink `.acc-tied` = A/D roll-forward.
  `rfMovements(c)` is the single source for the movement lines; `rfTiePanel()` shows the teal/pink "ties"
  card on contributing tabs; `rfLineDetail()` resolves each line to its postings.
- **Approvals:** `APPROVAL` 5-tier DOA/AFE matrix; manual entries post Pending; disposals need separate
  authorization (`approveDisp`).
- **localStorage key `fa_recon_v2`** — bump it (and `defaultState().v` + the `load()` check) on any data-shape change.

## Dev / verify
- Local: root `.claude/launch.json` → `fixed-assets-reconciliation` on **port 8738**
  (`preview_start name:fixed-assets-reconciliation`, then browser → localhost:8738).
- No build/test/lint. Verify by loading and checking: cost RF ends **$255,850,000**, CIP **$19,400,000**,
  A/D **$92,940,000**, standalone NBV **$182,310,000**, consolidated NBV **$266,340,000**, `varNetSub = 0`,
  all **24** tabs render, no console errors. Syntax-check the inline script with `node --check` after
  extracting `<script>…</script>`.
- **Verified this session:** `compute()` ties to all the above; all 24 `VIEWS` render with no errors;
  live URL returns HTTP 200. (Preview `screenshot` tool times out on this machine — verify via
  `preview_eval` DOM reads + `compute()`/`consolRoll()`.)

## State: complete — built, verified, deployed, git-connected, cleaned & handed off
No known defects. Code cleaned (removed dead helpers `money2`/`monthsBetween` and unused `.coststack`/`.leg`/
`.zone` CSS carried over from the inventory tool). **Git-connected with push-to-deploy verified** —
`git push origin main` (SSH alias `github-jessica`) auto-deploys. Branded path live on `app.fastinsights.io`.
Possible next asks (not requested): calibrate the DOA/AFE thresholds, capitalization threshold, tax rates and
useful-life policy to the real company; import real subledger detail so the register lists the full population
instead of detail + aggregated remainder; add ROU/lease detail to tie to the dedicated `lease-accounting` tool.
