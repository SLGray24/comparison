# Mortgage Comparison Tool — Build & Deploy Guide

A complete system for generating custom-branded mortgage comparison tools for every loan officer on your team. One engine, hundreds of customized tools.

## What's Included

| File | Purpose |
|---|---|
| `builder-dashboard.html` | The admin dashboard. You and your LOs fill out a form here; it generates a downloadable, ready-to-embed comparison tool. |
| `comparison-tool.html` | The customer-facing tool template. The dashboard injects each LO's config into this file. |
| `google-apps-script.gs` | The lead-capture webhook. Catches every submitted lead into a Google Sheet (with optional email alerts). |
| `README.md` | This file. |
| `compliance-checklist.md` | A field-tested compliance reference. **Read this before going live.** |

## How It Works

1. **You host the Builder Dashboard** on your company website (or any static host). LOs visit it.
2. **Each LO fills out their profile** — name, NMLS, photo, booking link, pricing, etc. — and clicks **Generate Tool**.
3. **The dashboard creates a custom HTML file** with that LO's branding and config baked in.
4. **The LO uploads their custom HTML** to their personal site (or you host all of them centrally) and either links to it or embeds it via an iframe snippet (shown in the dashboard).
5. **Customers use the tool**, upload their existing mortgage quote or enter it manually, then enter their contact info to view the comparison.
6. **Leads flow into a Google Sheet** via the Apps Script webhook, and the LO gets the booking-CTA-driven results page.

## Quick Start (30 minutes)

### Step 1 — Set up the Google Sheet & webhook (10 min, one-time per LO or team)
1. Create a new Google Sheet. Name the first tab `Leads`.
2. Extensions → Apps Script. Paste in `google-apps-script.gs`. Save.
3. Deploy → New deployment → Web app → Execute as **Me**, Access **Anyone** → Deploy.
4. Authorize, then copy the Web App URL. (Looks like `https://script.google.com/macros/s/.../exec`.)
5. Recommended: edit `FALLBACK_NOTIFY_EMAIL` in the script if you want lead-arrival emails.

### Step 2 — Host the Builder Dashboard (5 min)
- Upload `builder-dashboard.html` AND `comparison-tool.html` to the same folder on a web server, S3 bucket, Netlify, Vercel, GitHub Pages, or wherever your site lives. They must be siblings — the dashboard reads the comparison-tool.html template at runtime.
- (Alternative: open `builder-dashboard.html` locally in Chrome. Some browsers block local `fetch()` of sibling files; if you hit that, host them.)

### Step 3 — Generate the first LO tool (5 min)
- Open the dashboard. Sean's profile is preloaded. Customize it to whichever LO you're building for.
- Paste in the webhook URL from Step 1.
- Click **Update Preview** to verify everything looks right.
- Click **Generate Tool**. An HTML file downloads.

### Step 4 — Deploy that LO's tool (10 min)
- Upload the generated `mortgage-comparison-<lo-name>.html` wherever the LO wants it. Easy options:
  - **Squarespace/Wix:** Upload to file storage and link from a button. For embed, use the iframe snippet from the dashboard's Embed tab inside a Code Block.
  - **WordPress:** Upload via Media Library or FTP. Embed via the Custom HTML block.
  - **Custom site / GitHub Pages / Netlify:** Drop into your repo.
- Test the full flow with a fake submission. Verify the row appears in the Google Sheet.

## Builder Dashboard Reference

### Fields

| Field | Required | Notes |
|---|---|---|
| Full Name | Yes | LO's name as shown on the tool |
| NMLS # | Yes | Required for compliance display |
| Photo URL | No | Headshot URL; falls back to LO's initials in a brand-colored circle |
| Company / Company NMLS | Yes / No | Appears in header and disclosures |
| Booking Link | Yes | Reclaim, Calendly, etc. The big CTA on the results page |
| 1003 Application Link | No | Secondary "Start an Application" button |
| States Licensed | Yes | Shown as chips in the header |
| Brand / Accent Color | Defaults provided | Hex values; affects all primary UI |
| Estimate Mode | Yes | **Conservative** (range) or **Specific** (one number from your pricing). LO can flip per-tool. |
| Pricing | Optional | Base par rates by loan type/term. Used in specific mode; centers the range in conservative mode. |
| Lender Fees | Optional | Used in fees comparison & break-even calc |
| Webhook URL | Optional | Leads logged to console only if blank |
| Compliance Footer | Yes | Appears on every page — customize per state |

### Saving Drafts
Saved drafts live in your browser's localStorage. They are tied to the device + browser you saved them on — they do not sync between machines. To move a profile, export the JSON and re-import it on the other machine.

## Conservative vs Specific Mode

**Conservative (recommended):** The tool displays a rate *range* (e.g. *6.75% – 7.00%*) and a savings *range* (e.g. *$95 – $145/mo*). The LO never makes a specific rate claim. This is the safest, most defensible mode and is the default. Best for cold leads.

**Specific:** The tool shows one specific rate using the LO's configured pricing. Higher conversion, but the LO must keep pricing current and add stronger disclosure that it is an estimate. Best for warm leads coming from rate-shopping touchpoints.

Each LO can toggle per tool. You can also build two versions (cold + warm) and use them in different funnels.

## PDF Upload — What's Actually Happening

The tool uses [PDF.js](https://mozilla.github.io/pdf.js/) to extract text from a Loan Estimate PDF entirely in the customer's browser — **the file is never uploaded to any server**. We then use regex pattern matching to pull out:
- Loan amount
- Interest rate
- Term
- Monthly P&I
- Closing costs
- Property taxes / insurance / MI
- Loan type (Conventional, FHA, VA, USDA, Jumbo)
- Property ZIP

Loan Estimates use a standardized CFPB format, so accuracy is high for standard LEs. For non-standard quote sheets, the customer reviews and corrects the extracted values before continuing.

## Compliance Highlights

This is a high-level summary — see `compliance-checklist.md` for the full list.

- **Never an offer of credit.** Every page includes "This tool provides an illustrative comparison only and is not an offer to extend credit or a commitment to lend."
- **TCPA opt-in.** Lead capture requires explicit, written consent for phone/SMS contact with clear language about automated dialing, prerecorded messages, and opt-out.
- **NMLS + Equal Housing Lender** displayed prominently on every page.
- **State licensing** shown for each LO.
- **No prohibited claims.** Tool copy avoids "guaranteed", "lowest", "best rate", or specific APR promises.
- **Print/save respects compliance.** The print stylesheet keeps the disclaimer footer visible.

## Multi-LO Strategy

For a team rollout, recommended pattern:

1. **One Google Sheet per LO** (or one shared sheet, depending on routing needs).
2. **One generated HTML per LO**, hosted at predictable URLs (e.g. `yoursite.com/tools/sean/` or `seangraymlo.yoursite.com/comparison`).
3. **Train your LOs** to update their tool when pricing changes meaningfully (recommend monthly review).
4. **Build a template for your LO marketing pages** that embeds the tool inline so it feels native to each LO's site.

## Troubleshooting

| Problem | Likely cause |
|---|---|
| "Could not load comparison-tool.html template" | The two HTML files aren't siblings, or you're opening from `file://` and the browser blocks fetch. Host them on a web server. |
| Leads not appearing in sheet | Webhook URL is wrong, or Apps Script wasn't deployed as "Anyone has access". |
| Preview iframe is blank | Pop-ups blocked, or the comparison-tool.html template can't be reached. Check browser console. |
| PDF extraction missed fields | Customer can edit any field before submitting. Loan Estimates from less common lenders may use non-standard labels. |
| CORS errors on lead submit | The tool uses `mode: 'no-cors'` to avoid preflight. If you see CORS in console, double-check the script returned a 200. The response is opaque but the row will still post. |

## Need to Modify the Tool Itself?

`comparison-tool.html` is one self-contained file. Common tweaks:
- **Different color scheme defaults** → search for `--brand` and `--accent` in CSS.
- **Different lead capture fields** → edit `renderLeadCapture()` and the corresponding state object.
- **Different math** → edit `monthlyPI()` and `buildComparison()`.
- **Different copy** → all UI strings are in the `render*` functions.

After edits, re-run **Generate Tool** in the dashboard to produce new files for each LO using the updated template.

---

Questions or compliance review needed before launch? Have your in-house counsel or compliance officer sign off on the disclaimer text in `compliance-checklist.md` before going live.
