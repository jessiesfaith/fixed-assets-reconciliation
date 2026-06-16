# Fixed Assets Reconciliation — repo notes

Generic workflow rules live in `~/.claude/CLAUDE.md` (they apply to every repo). This file holds
**repo-specific facts only**. Source of truth = this repo, not the chat transcript.

## Repo facts (created 2026-06-16)
- **What:** Single-page static **full-cycle fixed-assets (PP&E) reconciliation** workspace. A tabbed
  dashboard that ties the **FA subledger → GL control** across **two roll-forwards** (gross cost +
  accumulated depreciation → net book value) plus a **CIP sub-roll-forward**, and covers the SOX close:
  capitalization policy, CIP-to-in-service, physical existence/tagging, depreciation, impairment,
  disposals, transfers/reclasses, ASC 842 leases, book-vs-tax depreciation, capex authorization (AFE/DOA),
  and insurance/property-tax schedules. **Workflow & analysis only — posts no journal entries, produces
  no financial statements, not accounting/tax/audit advice.** All client-side; state in `localStorage`;
  no accounts/backend.
- **Sample domain:** same fictional manufacturer as the inventory tool — **`Helixa Biosystems, Inc.`**,
  a ~$500M-revenue biotech (labs, bioprocessing plant, cleanrooms, instruments, leaseholds). Period
  **May 2026**, `TODAY='2026-06-15'`.
- **Package manager / framework:** none. Vanilla HTML/CSS/JS, no `package.json`, no build step, **no
  CDN deps** (Google Fonts only). The 24-month trend chart is hand-rolled inline SVG (`faTrendChart()`).
- **Key file:** `index.html` (entire tool: inlined CSS + JS). **20 tabs** via `state.tab` + `VIEWS{}`,
  presented through a **left vertical sidebar** (`renderTabbar()` builds `#sidenav` from `GROUPS`):
  9 groups (Overview / Acquisition & CIP / Register & Existence / Depreciation & Valuation /
  Disposals & Transfers / Leases / Accounting / Tax & Reporting / Analytics). Tabs numbered 1–20 via
  `TAB_ORDER`/`TAB_NUM`. Order: recon, additions, cip, cappolicy, register, verify, locations, deprec,
  impair, lives, disposals, transfers, leases, gl, liab, approvals, tax, insure, analytics, review.
- **Engine — dual roll-forward (`compute()`):** journal-entry driven. `TXN_TYPES` maps each posting
  type → Dr/Cr accounts (`ACC`) + which roll-forward it moves (`book:'cost'|'cip'|'accum'`, `sign`).
  `@class` postings carry `t.acct` (the asset-class account). Derives:
  - **Gross cost (1500–1560):** `openGrossCost + additions + cip_capit − disp_cost` → sample **$255,850,000**.
  - **CIP (1590):** `openCIP + cip_spend − cip_capit` → sample **$19,400,000**.
  - **Accum. dep. (1600):** `openAccum + depreciation + impairment − disp_accum` → sample **$92,940,000**.
  - **Net book value per GL:** `endCost + endCIP − endAccum` → sample **$182,310,000**.
  - **Tie-out:** GL net vs **FA subledger control** (`subCost/subCIP/subAccum`, set to tie → `varNetSub=0`,
    status "✓ Subledger ties to GL") vs **physically-verified NBV** (`physNet`, sample $240K existence
    variance → flagged exception). Reclasses are a **net-$0 memo** line (gross shown).
- **Roll-forward tie color (two colors):** teal (`--rf` / `.rf-tied`) marks figures flowing into the
  **gross-cost / CIP** roll-forwards; pink (`--acc` / `.acc-tied`) marks figures flowing into the
  **accumulated-depreciation** roll-forward. `rfMovements(c)` is the single source for the 8 movement
  lines; `rfTiePanel(tabKey)` renders a "Ties to the PP&E roll-forwards" card on each contributing tab
  (additions, cip, deprec, impair, disposals) with `rfLineDetail()` click-to-expand to the postings that
  sum to each line. The recon roll-forward lines also expand inline (`rfRow()`/`toggleRF()`/`rfSetAll()`)
  and carry `↳ #N` jump links to the tying tab.
- **Approvals (`APPROVAL` 5-tier DOA / AFE matrix, ~$500M company):** Dept Mgr ≤$25K / Controller ≤$100K /
  VP Finance–Capital Cmte ≤$500K / CFO (AFE) ≤$2.5M dual / CFO+CEO (Board) >$2.5M dual; `approvalFor()`.
  Manual GL entries post `src:'manual',apprStatus:'Pending'` and route to the Approvals tab; **disposals
  additionally require disposal-authorization** (`d.approved`, `approveDisp()`). SoD: preparer ≠ approver.
- **Tab numbering + GL cross-ref:** GL Transactions tab has a Chart-of-Accounts list mapping each `ACC`
  account → the numbered tab(s) that tie it out (`ACC_TABS` + `tabNums()`, clickable). Accounts in the
  roll-forwards tagged `RF_ACCTS`.
- **Review & Export:** exception log, three-role sign-off (preparer≠reviewer SoD flag), period lock,
  CSV + Markdown export (both roll-forwards, tie-out, CIP, disposals, leases, book-vs-tax, capex analytics).
- **localStorage key `fa_recon_v1`** (bump `defaultState().v` + the `load()` check when the data shape changes).
- **Dev (localhost):** root `.claude/launch.json` config `fixed-assets-reconciliation` on **port 8738**
  (`npx http-server`). The machine `python` is a non-functional Windows Store alias — use Node's http-server.
- **Build / test / lint / typecheck:** none configured. Verification = manual + preview panel +
  `node --check` on the extracted `<script>`. Engine self-checks via `compute()` (sample ties to the
  figures above; `varNetSub` must be 0).
- **Deploy:** **DEPLOYED 2026-06-16 via Vercel CLI** (`npx vercel deploy --prod --yes` from this dir).
  Vercel project `fixed-assets-reconciliation`, account `jessica-dougherty-s-projects`.
  **Production:** **https://fixed-assets-reconciliation.vercel.app** (clean alias was available).
  Latest deployment id `dpl_H2g6SLeKgm4CSNDsEsez7AM2xBYt`.
- **GitHub:** **not yet git-connected** (deploy was CLI-only). To git-connect like the other tools:
  create `jessiesfaith/fixed-assets-reconciliation`, push via the SSH alias `github-jessica`, and
  `vercel git connect` (temporarily set origin to the https URL for the connect, then restore the alias).

## Conventions (match the other Fast Insights tools)
- `vercel.json` = `{ "cleanUrls": true }`. `.vercelignore` keeps everything but `index.html` +
  `vercel.json` out of the deploy.
- Dark Fast Insights palette (near-black `#0a0a0a` + green accent `#00c805`), Outfit body +
  Cormorant Garamond headings. Same CSS framework as inventory-reconciliation / month-end-close.
- Guardrail tone: workflow/analysis only; never present as posting entries or giving accounting/tax/
  audit advice. Every tab carries the planning-only disclaimer.

## Environment notes (this machine)
- **OneDrive Files-On-Demand:** this repo is inside a OneDrive-synced vault. Files may be cloud
  placeholders that tools intermittently fail to see. Retry once.
- **Claude preview `screenshot` tool times out** (renderer issue, not the page) — verify via
  `preview_eval` DOM reads + `compute()` instead. Multiple preview servers run at once and share one
  preview browser — confirm `location.href` is `localhost:8738` before eval/verify.
- **GitHub = Jessica (`jessiesfaith`).** Confirm Chris vs. Jessica before any GitHub work.
