# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A single-page web app (`index (1).html`) that generates personal promissory note agreements with stock pledge collateral. Pete (Peter W Davidsmeier) is always the borrower. The lender is whoever Pete hands the device to. No server, no build step — open the HTML file in a browser.

## Architecture

Everything lives in one self-contained HTML file:

- **CSS** (lines 12-310): Dark theme, gold accent, mobile-first. All custom properties in `:root`.
- **HTML** (lines 312-591): 5-step wizard flow — Tier Selection > Loan Terms > Lender Info > Borrower Info > Review & Export.
- **JS** (lines 593-end): State management, tier/share calculation, signature canvases, localStorage persistence, two PDF generators (contract + stock certificate).

### Key Data Flow

```
Tier Selection (fixed or dropdown)
  → calcShares(amount)  // 20% up to $10k, 25% on excess
  → updateTermsBox()    // computes interest at 5% APR for 6 months
  → renderReview()      // populates Step 5 summary
  → buildPDF()          // contract PDF via jsPDF
  → buildCertificatePDF()  // landscape stock certificate via jsPDF
```

### Share Calculation Formula

```
amount <= $10,000:  shares = amount * 0.20
amount >  $10,000:  shares = (10000 * 0.20) + ((amount - 10000) * 0.25)
```

Fixed tiers ($4k, $6k, $8k) have hardcoded share counts in `TIERS[]` that must stay consistent with `calcShares()`. The custom tier uses a `<select>` dropdown in $500 increments up to $50k.

### Persistence

`localStorage` key: `dw_loan_v2`. Saves all form fields, selected tier, custom amount, and reference number. Restore banner shown on reload if saved data exists. Signatures are canvas-based and do NOT persist across page reloads.

### PDF Generation

Two PDFs via jsPDF (loaded from CDN):
- **Contract PDF** (`buildPDF()`): Portrait letter-size, full legal terms (12 clauses), both signatures, Zelle payment instructions.
- **Stock Certificate PDF** (`buildCertificatePDF()`): Landscape letter-size, ornamental border, share count, both signatures, collateral disclaimer.

Both embed signature canvas images via `canvas.toDataURL()`.

## External Dependencies (CDN)

- `jspdf 2.5.1` — PDF generation
- `jspdf-autotable 3.5.28` — table plugin (loaded but not currently used)
- Google Fonts: Cormorant Garamond, DM Sans, DM Mono

## Borrower Info (Pre-filled, Do Not Change Without Pete's OK)

Pete's info is hardcoded as `value=` attributes on the borrower form fields (Step 4). The address, phone, and email are real. Don't modify these unless explicitly asked.

## Legal Constants (Hardcoded by Design)

These values are intentional and should not be refactored into config:
- Interest rate: 5% per annum
- Repayment term: 6 months or VC close ($500k threshold)
- Late fee: 5% (editable by user in Step 2)
- Pete's ownership: 40,000 shares (40%)
- Company: DealWhisper Inc. (Florida C Corp)
- Governing law: Florida
- Zelle phone: 727-400-2225

## Development

No build, no server, no tests. Open `index (1).html` in a browser. Changes are instant on reload. To test PDF output, click the export buttons on Step 5 — both require reaching the review step with at least a tier selected.
