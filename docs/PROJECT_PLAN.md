# Convention Manager: project plan

Last updated: 2026-09-10

Status: agreed product direction. This document describes the intended product, not a claim that its features have been implemented.

## Purpose

Build a freely accessible app that helps convention artists understand their manufacturing costs, choose prices and deals, forecast convention profitability, record sales without reliable internet, and compare actual results with their plans.

The app should be useful to an individual artist without an account. Optional accounts provide cloud saving and cross-device access. All features remain free; an external Ko-fi link lets artists voluntarily support running costs without paid feature gates.

## Agreed decisions

| Area | Decision |
| --- | --- |
| Platform | Mobile-first web app, installable as an offline-capable Progressive Web App (PWA). No separate native app for the MVP. |
| Stack | React, TypeScript, and Vite; Cloudflare Pages for the frontend; Supabase Auth and PostgreSQL for accounts and cloud data. |
| Guest use | Save locally in IndexedDB, with export/import. An account is not required to calculate, plan, or record sales. |
| Accounts | Optional free accounts, explicit import of guest plans, and synchronization when connected. |
| Manufacturing | Structured manual entry of supplier quotes, quantities, shipping, and fees. Automatic quote parsing and supplier integrations are not MVP requirements. |
| Deals | Individual prices, buy-N-get-M-free deals, and fixed-price bundles. Forecast bundle and individual sales explicitly rather than automatically finding the cheapest basket. |
| During an event | An offline sales tracker called Sell mode, with one active selling device per table/convention. |
| Payments | Record the payment method only. Artists take payments separately; the app does not process cards or store payment-card details. |
| After an event | Actual-versus-forecast reporting from tracked sales, with manual totals as an alternative for artists who do not use Sell mode. |
| Later scope | Linked multi-convention inventory and stock carry-over, supported supplier integrations, and table layout planning progressing from 2D to 3D. |

## Artist workflow

| Step | Artist experience |
| --- | --- |
| Create a convention | Enter the name, dates, reporting currency, and expected table, travel, accommodation, and other expenses. |
| Add products and manufacturing costs | Enter products/variants and quote lines, quantities, shipping, and additional fees. Calculate landed cost per usable item. |
| Set prices and deals | Compare individual prices and promotions, including their contribution toward convention expenses. |
| Forecast sales | Estimate individual and bundle sales in conservative, expected, and optimistic scenarios. |
| Prepare for offline use | While online, save the app and the selected convention's products, prices, deals, currency rates, and opening stock on the selling device. |
| Record sales | Build a basket using large product buttons, select a deal where appropriate, record the payment method, and save the sale locally. |
| Review the convention | Use recorded sales or enter manual actual totals, add final expenses, and compare the outcome with the forecast. |

An artist can plan on a computer and prepare their phone for selling. Once prepared, the phone must not need Wi-Fi to reopen the app or record sales. Cross-device access does not mean simultaneous multi-device selling in the MVP.

## MVP requirements

### Products, manufacturing, and event expenses

- Maintain a reusable catalog of products and variants.
- Accept supplier quote line items with production quantities, usable quantities, original currencies, and costs.
- Explicitly allocate shared shipping, customs, and other quote charges across product lines. Show the allocation method and allow corrections rather than hiding assumptions.
- Calculate landed cost per usable item, including applicable shared charges.
- Keep manufacturing costs separate from event expenses to avoid counting the same cost twice.
- Record the stock available for the selected convention. Cross-convention stock allocation and automatic carry-over are later features.
- Snapshot convention-specific costs and prices. Updating catalog defaults must not rewrite previous forecasts or completed sales.

Quote entry means structured fields and optional pasted notes. It does not promise automatic extraction from arbitrary invoices, PDFs, screenshots, or supplier pages.

### Financial model

Show two distinct views:

| View | Meaning |
| --- | --- |
| Event profit | Sales revenue after the cost of stock sold or given away, convention expenses, and selling fees. |
| Cash required and recovered | Upfront manufacturing purchases and event spending funded for this plan, compared with money received. |

For example, manufacturing 200 keychains and selling 80 should not imply that all 200 were sold. Unsold items retain a recorded cost basis. Previously purchased stock is not a new cash purchase every time it is brought to a convention.

Include percentage-based payment fees and optional fixed per-transaction fees. Forecast transaction counts separately from item or bundle counts; one customer may buy several items in a single transaction.

Any break-even estimate must state the assumed product/deal mix. If the selected margins or available stock cannot cover costs, show that clearly rather than producing a misleading target.

Use decimal-safe arithmetic and currency-appropriate rounding. Do not assume every currency has two decimal places or use ordinary floating-point arithmetic throughout financial calculations. Label outputs as planning estimates and make tax assumptions explicit; this is not a tax-filing or full accounting system.

### Currencies

- Give each convention a reporting currency and each cost its original currency.
- Suggest a default from browser locale, ask the artist to confirm it, and remember their preference. A website cannot reliably read the user's actual operating-system currency preference.
- Support editable exchange-rate snapshots, initially entered manually. Automatic rate fetching can be added later.
- Retain the original amount, currency, conversion rate, and rate date/source where available.
- Preserve rates used in forecasts and actual results. New exchange rates must not silently change historical reports.

### Prices, deals, and forecasts

Support both requested promotion types:

- Buy N, get M free: revenue comes from the paid items, but all delivered items consume stock and incur their costs.
- Fixed-price bundles, such as three stickers for six units of the convention's currency: use the bundle price as revenue and account for every included item.

Define eligible products and the expected product mix for forecast bundles when manufacturing costs differ. Do not silently assume all eligible designs cost the same.

Forecast individual sales and bundle counts separately, consuming the same available stock. Make insufficient stock and invalid quantities visible.

In Sell mode, artists explicitly select applicable deals. Automatic deal stacking, overlapping promotion resolution, and cheapest-basket optimization are outside the MVP.

### Sell mode

Prioritize large product buttons, quantity controls, deal shortcuts, a clear basket total, a prominent "Record sale" action, and an "Undo last sale" action.

Record each completed sale's items, delivered quantities including free items, actual prices/deals, total, currency, payment method, and timestamp. Preserve the cost and pricing information needed to reproduce the result later.

Taking payment happens outside the app. Recording "card" is a record of the artist's chosen payment method, not confirmation from a payment processor.

Update local revenue and remaining stock after a successful local save. Corrections should retain the original record and explicitly reverse or adjust its effects instead of silently deleting history.

No customer names, contact details, or payment-card data are required for this workflow.

### Actual-versus-planned results

Use tracked sales to populate actual revenue and stock consumed, then let the artist enter final expenses and relevant adjustments.

Retain a manual-results path for artists who do not use the tracker. To estimate profit from manual results, collect actual revenue and quantities sold or given away, not revenue alone.

Treat tracked results and manual totals as alternative reporting sources. Do not add an entire manual event total on top of tracked sales. Any reconciliation adjustments must be explicit.

### Mobile and offline reliability

Mobile support is foundational, not a later responsive retrofit. Use short grouped forms, readable cards rather than wide spreadsheets, large touch targets, suitable numeric keyboards, accessible labels, and an easily accessible profit summary.

The offline implementation must include:

- A deliberate "Prepare for offline use" action that saves both the app resources and the selected convention's data. Show readiness only when the required resources and data are available locally.
- Cached app resources through a service worker and structured local data in IndexedDB. An installed icon alone does not guarantee an app can operate offline.
- Local-first recording. Save the sale and its pending-upload entry atomically before reporting success. A network request must not block recording.
- Stable unique IDs and server-side uniqueness so retries, reconnections, or duplicate upload attempts cannot duplicate sales.
- Visible states distinguishing local saving, pending uploads, successful cloud synchronization, and errors. Example: "Saved on this device - 12 sales waiting to sync."
- Foreground synchronization when the app is open and connected, plus a manual retry action. Do not depend on background synchronization while the app is closed.
- Continued access to prepared local sales recording if an online session expires. Re-authentication may be needed to sync; local data must remain associated with the correct owner.
- Clear failures if local storage cannot accept a sale. Never present an unsaved sale as recorded.
- Safe updates and migrations that preserve local data and pending uploads and do not interrupt a sale in progress.
- Persistent-storage requests where supported and offline export/import for backups. Persistence permission is not guaranteed and installation is not a backup.

Initial preparation requires internet. Clearing browser data, device loss, private-browsing behavior, or storage eviction can remove unsynced records. Explain this limitation clearly. Provide warnings before actions such as clearing local data or signing out when records remain unsynced.

Guest data stays local unless explicitly exported or imported into an account. Do not silently upload a guest's data or attach one user's pending sales to another account.

## Technical architecture

| Component | Responsibility |
| --- | --- |
| React + TypeScript + Vite | Mobile-first planner, Sell mode, reports, and PWA interface. |
| Pure TypeScript calculation module | Costs, currency conversion, price/deal calculations, stock consumption, forecasts, and report calculations. Independent of React and Supabase. |
| IndexedDB persistence layer | Guest plans, downloaded convention data, local sales/corrections, and pending synchronization records. |
| Service worker and app cache | Load the prepared application without a network connection. Keep app-resource caching separate from business-data storage. |
| Supabase Auth + PostgreSQL | Optional accounts and private cloud data. Row-level security limits access to the owning artist. |
| Synchronization layer | Explicit guest import, retry-safe sales uploads, acknowledgments, and visible handling of editable-plan conflicts. |
| Cloudflare Pages | Static frontend hosting. Privileged operations, if needed, belong in server-side functions rather than exposing secrets in the browser. |

Keep the calculation engine and storage interfaces separate so guest use, cloud-backed plans, and future layout tools share the same domain rules. Use a small modular application, not separate microservices.

Important data groups:

- User preferences, including default currency.
- Products and variants.
- Manufacturing quotes and quote lines.
- Conventions, event expenses, and exchange-rate snapshots.
- Convention product/cost/price snapshots and opening stock.
- Deal definitions, forecast scenarios, and forecast lines.
- Sales and sale lines, payment-method records, corrections, and event stock adjustments.
- Device-local synchronization metadata and pending uploads.

Use convention identifiers to separate event records from the start. The MVP focuses on one convention at a time; a shared inventory ledger and automatic movement between conventions are not required yet.

Do not resolve all cloud conflicts with silent last-write-wins overwrites. Financial records should be retry-safe and auditable; edits to mutable plans should detect conflicts and surface a resolution path.

## Delivery milestones

| Milestone | Deliverable |
| --- | --- |
| 1. Domain model and calculator | One convention and manufacturing quote, individual pricing, both deal types, currencies, stock consumption, and distinct profit/cash views. |
| 2. Mobile guest planner | Complete planning workflow, reusable products, scenarios, IndexedDB saving, and export/import. Establish local-first storage before cloud synchronization. |
| 3. Offline Sell mode | Offline preparation, one-device sales capture, manual deal selection, payment-method recording, local totals/stock, corrections, and visible local-save states. |
| 4. Optional accounts and synchronization | Authentication, private cloud storage, guest import, cross-device planning, retry-safe sales uploads, and visible conflict/error handling. |
| 5. Post-event reporting and public release | Tracked or manual actual results, forecast comparisons, final expense entry, Ko-fi link, privacy/data-deletion controls, backups, and usage monitoring. |

Milestones describe implementation order, not delivery dates. The offline tracker is part of the agreed MVP rather than a future native-app project.

### Release expectations

- Quote allocations, rounding, currency conversion, fee calculations, and both deal types produce reproducible results.
- Free items consume stock and incur cost without adding sales revenue.
- A prepared device can lose connectivity, close and reopen the app, and continue recording without losing saved sales.
- Local-save errors are visible, and repeated synchronization never creates duplicate sales.
- Later price/cost/rate edits do not rewrite historical sales.
- Reports do not double-count manual totals and tracked sales.
- Account access is private, and guest import or account switching does not expose or misattribute records.

## Later roadmap

These are later feature groups, not requirements that should delay the MVP. Their relative priority can be revisited after real convention use.

### Multiple conventions and stock carry-over

Support an artist's stock across successive conventions rather than requiring them to recreate it manually for every event.

After Convention A, show a per-product/variant stock summary covering opening stock, additions, items sold, free promotional items, other giveaways, damage/loss, returned items placed back in stock, and closing stock. Allow the artist to reconcile the result against a physical count using explicit adjustments.

When planning Convention B, let the artist select how much of that remaining stock to bring, add any newly manufactured stock, and use the resulting quantities as the next convention's opening stock. Carry-over should be reviewed and explicit, not an automatic copy that allocates the same stock to several events.

Preserve original manufacturing cost basis and quote/batch provenance. Carrying existing stock to another event must not create a new manufacturing cash expense or alter the previous convention's results.

Model the movement as inventory allocations/transfers with a history, rather than repeatedly overwriting a product's quantity. This should support a sequence of conventions and eventually a consolidated view of stock available, allocated, and remaining.

Before implementation, decide how to value mixed manufacturing batches, such as batch-specific costing, FIFO, or weighted average. That accounting choice is not settled by this plan.

### Supplier integrations

Investigate supported quote, pricing, or order access for Wooacry, Vograce, and any other suppliers artists use.

Confirm official capabilities and terms before promising an integration. Keep adapters separate from the calculation engine, preserve manual entry as a fallback, and do not make storefront scraping an MVP dependency.

Order/invoice import and automatic quote extraction are separate potential features, not capabilities implied by manual quote entry.

### Table planning, followed by 3D

Start with a dimension-aware 2D table layout editor covering tables, stands, shelves, signage, and product placement. Link equipment costs to the existing convention budget rather than building a second pricing system.

Then add a 3D view/editor using the same layout data. Keep dimensions and placement separate from the rendering implementation. Load 3D functionality only when needed so the main planner remains fast on mobile.

Native apps, simultaneous multi-device selling, payment processing, and specialist hardware integrations would require separate scope decisions; they are not promised by this roadmap.

## Running costs and operations

Aim for free-tier operation during development and early use, while keeping the product free to artists even if operating costs later increase. Donations are optional support, not a guaranteed infrastructure budget.

Free tiers have quotas and can change. As of this plan's date, Supabase's free tier pauses inactive projects and excludes automatic database backups. Plan for backups, monitoring, and an eventual paid hosting budget rather than assuming free hosting will always be sufficient.

Email-based sign-in requires production email delivery configuration; Supabase's built-in SMTP service is not a public-production email solution. Select the initial login method and, if needed, a transactional-email provider during account setup. A custom domain and email service may introduce costs.

Keep account export/deletion, local-data warnings, and ownership protections in the public-release scope. Do not collect customer or payment-card information for the sales tracker.

## Using this plan in future conversations

Use the repository-relative path `docs\PROJECT_PLAN.md` rather than the absolute path of a particular worktree.

Suggested prompt:

> Read docs\PROJECT_PLAN.md as the product source of truth for Convention Manager. Inspect the current implementation, then help me with [feature or milestone]. Keep later-roadmap features out of scope unless I explicitly ask for them, and update the plan when we agree to change a decision.

For planning-only work, add: "Do not implement anything yet."

The file must exist in the conversation's checkout. A new session does not automatically receive another session's uncommitted files or complete chat history. Commit and merge the plan into the default branch to make it available to ordinary new repository sessions, or explicitly use a branch containing the committed plan. For a conversation without repository access, attach or paste the document.

Keep this file current as decisions change. Future work should inspect the actual repository rather than treating these milestones as evidence of implementation progress.

## References

- [Supabase pricing](https://supabase.com/pricing)
- [Supabase production email configuration](https://supabase.com/docs/guides/auth/auth-smtp)
- [Cloudflare Pages limits](https://developers.cloudflare.com/pages/platform/limits/)
- [PWA offline data storage](https://web.dev/learn/pwa/offline-data)
- [Persistent browser storage](https://web.dev/articles/persistent-storage)
