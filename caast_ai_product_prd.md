# Caast AI Product PRD

> **Comprehensive Frontend & Backend Specification**
> Login · Influence Intelligence · Attribution Dashboard · Content Plan · Paid Campaign · Settings

---

| | |
|---|---|
| **Version** | 1.0 |
| **Date** | 2026-05-02 |
| **Classification** | Confidential — Engineering |
| **Owner** | Caast Co-Founders |
| **Visual specification** | `caast_ai_product_paid.html` (v1) |
| **Companion documents** | Connection Guide PRD; VCI/BA PRD; `meta_ads_tutorial.md` (Sections 5–7) |

---

## Table of contents

- [How to read this document](#how-to-read-this-document)
- [1. Product overview](#1-product-overview)
- [2. Goals and success metrics](#2-goals-and-success-metrics)
- [3. Personas and primary scenarios](#3-personas-and-primary-scenarios)
- [4. Phasing strategy](#4-phasing-strategy)
- [5. Login and authentication](#5-login-and-authentication)
- [6. Screen 1 — Influence Intelligence](#6-screen-1--influence-intelligence)
- [7. Screen 2 — Attribution Dashboard](#7-screen-2--attribution-dashboard)
- [8. Screen 3 — Content Plan](#8-screen-3--content-plan)
- [9. Screen 4 — Paid Campaign](#9-screen-4--paid-campaign)
- [10. Screen 5 — Settings](#10-screen-5--settings)
- [11. Cross-cutting backend services](#11-cross-cutting-backend-services)
- [12. Core data models](#12-core-data-models)
- [13. Non-functional requirements, risks, references](#13-non-functional-requirements-risks-references)

---

## How to read this document

This PRD specifies the complete Caast AI Product — the post-activation product the brand uses every day after completing the landing-page activation flow. It covers everything visible in `caast_ai_product_paid.html`: login, the five product surfaces (Influence Intelligence, Attribution Dashboard, Content Plan, Paid Campaign, Settings), and all backend services that power them.

Every requirement is tagged with a stable ID and labelled either **[FRONTEND]** (UI/UX, what the user sees and interacts with) or **[BACKEND]** (services, APIs, data models, integrations).

**ID conventions.** `UI-XXX` = frontend requirement. `BE-XXX` = backend requirement. `NFR-XXX` = non-functional requirement. The first digits identify the surface:

| Range | Surface |
|---|---|
| `1XX` | Login |
| `2XX` | Influence Intelligence |
| `3XX` | Attribution Dashboard |
| `4XX` | Content Plan |
| `5XX` | Paid Campaign |
| `6XX` | Settings |
| `7XX` | Cross-cutting backend services |
| `8XX` | Non-functional requirements |

The visual specification is the HTML file. When this document and the HTML conflict, **the HTML wins for visual details** (colors, spacing, typography), and **this document wins for behavior, logic, and backend contracts**. Where the HTML implies behavior that isn't described here, implement what the HTML shows.

**Phasing.** Each requirement carries a phase tag in the rightmost column. **MVP** requirements ship at v1 launch. **Phase 2** requirements ship within the following 3–4 months. Phase 2 work that depends on external timelines (Meta App Review for Partnership Ads, DOL legal updates) is flagged explicitly.

---

## 1. Product overview

### 1.1 What this product is

The Caast AI Product is the brand-facing dashboard for the Caast Influence Network. After a brand completes activation (covered by the landing page PRD), they log in here to monitor their fleet of Digital Opinion Leaders (DOLs), review and approve content, manage paid amplification, and measure the resulting business impact.

It is not a content scheduler. It is not a social media management tool. It is the brand-side cockpit of an autonomous influence engine — the surface through which a Shopify SMB sees, controls, and trusts the work Caast does on their behalf.

### 1.2 The five product surfaces

| Surface | What it does | Primary user job |
|---|---|---|
| **Influence Intelligence** | Live radar visualization, AI assistant, weekly highlights across business / fleet / competitive lenses | Get the morning pulse — what's working, what changed |
| **Attribution Dashboard** | Funnel-level KPIs across Awareness, Consideration, Conversion, Paid performance with charts and period selector | Quantify business impact and prove ROI |
| **Content Plan** | Per-DOL view of upcoming and recent posts. Approve, edit, or flag for paid promotion | Govern the creative going out under each DOL persona |
| **Paid Campaign** | Live view of which DOL posts are amplified on Meta with per-post ROAS / CPA / Hook rate | See and control paid amplification |
| **Settings** | Account, plan, connections (organic + Meta paid), Approval-mode toggles for organic and paid, weekly paid budget | Configure the system and stay in control |

### 1.3 Strategic context

This release is the moment Caast goes from "always-on organic influence" to "always-on growth engine." The paid amplification capability sits on top of the existing organic infrastructure (DOL fleet, Brand Assimilation, Vertical Content Intelligence), creating a flywheel: organic content seeds, paid validates fast, paid winners shape the next organic, the loop compounds.

The strategic premise grounding the paid module: in 2026 Meta paid social is no longer a media-buying problem — it is a **creative-supply problem**. Andromeda + Advantage+ Sales Campaigns automate audience selection, placement, bidding, and budget allocation; the remaining job is feeding the algorithm fresh, on-brand creative. Caast's DOL fleet is exactly that creative supply. (See `meta_ads_tutorial.md` Sections 1–2 for the platform paradigm and Sections 5–7 which anchor specific paid requirements throughout this PRD.)

### 1.4 What's in scope

- All five product surfaces in `caast_ai_product_paid.html`, including the login flow that gates them.
- Frontend implementation of every screen, tab, card, control, animation, and state transition shown in the HTML.
- Backend services that power the product: authentication, the Influence Intelligence aggregation layer, the attribution and analytics engine, the Content Plan service, the Paid Autopilot service (Meta-only via the Meta MCP layer), the Settings and connection management service, and the notification system.
- All API contracts between frontend and backend, and all integration contracts with external systems (Meta Business Manager via MCP, Shopify, Instagram Graph API, TikTok read-only).
- The complete data model: brands, DOLs, posts, campaigns, ads, attribution events, settings, sessions.

### 1.5 What's out of scope

- The activation flow (landing page) — covered by a separate PRD.
- DOL persona generation, Vertical Content Intelligence, and Brand Assimilation — covered by the VCI/BA PRD.
- TikTok paid amplification — deferred to a future release. TikTok remains a read-only organic surface in this version.
- DOL fleet provisioning, persona content production internals, and the Production Factory itself — these are upstream services the AI Product treats as black boxes.

### 1.6 Glossary

| Term | Meaning |
|---|---|
| **DOL** | Digital Opinion Leader — a Caast-owned AI persona that publishes organic content under a vertical specialty |
| **AUI** | Audience Under Influence — people exposed to DOL content 3+ times in 30 days who took at least one intent action |
| **EMV** | Earned Media Value — the equivalent paid spend the organic engagement would have cost |
| **ASC** | Advantage+ Sales Campaign — Meta's automated ecommerce campaign type, the only paid campaign type in v1 |
| **CAPI** | Conversions API — server-side event delivery from Caast to Meta, bypassing browser-based Pixel limits |
| **EMQ** | Event Match Quality — Meta's 0–10 score of how well events identify users; ≥7 required for paid launch |
| **MER** | Marketing Efficiency Ratio — total revenue ÷ total marketing spend; the blended truth metric |
| **Approval-mode** | Default state where every post (organic) or campaign (paid) requires brand approval before going live |
| **Auto-publish** | Organic state where the fleet publishes without per-post approval |
| **Auto-advertise** | Paid state where the autopilot launches and scales campaigns without per-campaign approval, capped by weekly budget |
| **Meta MCP** | The Meta Model Context Protocol layer that lets Caast manage Meta ads programmatically once the brand fixes the budget |

---

## 2. Goals and success metrics

### 2.1 Product goals

The product exists to do four jobs for the brand.

- **Make the brand confident** that the influence engine is working. Replace the anxiety of "I'm paying $1,000/month, what am I getting?" with a glanceable daily answer.
- **Translate organic + paid activity into business outcomes** the brand can repeat to their team — followers, engagement, store visits, attributed revenue, ROAS, MER.
- **Give the brand control without overhead**. They can intervene anytime (approve, edit, pause, change budget) but the default is autopilot.
- **Close the organic-to-paid loop.** Surface the moment a DOL post is performing well enough to amplify, and let the brand send it to paid with one tap — or do it autonomously when Auto-advertise is on.

### 2.2 Success metrics

| # | Metric | MVP target | Phase 2 |
|---|---|---|---|
| 1 | Time-to-insight (login → first KPI rendered) | < 2s | < 1s |
| 2 | Daily active brands (DAB / WAB ratio) | > 35% | > 50% |
| 3 | % of brands who approve at least one post per week (organic engagement) | > 70% | > 85% |
| 4 | % of paid-eligible brands who turn paid on | > 30% | > 60% |
| 5 | Median time to launch first paid campaign after enabling | < 24h | < 1h |
| 6 | Paid blended MER (across all paid-active brands) | > 2.5 | > 3.5 |
| 7 | Approval queue latency (post created → visible in Content Plan) | < 60s | < 10s |
| 8 | EMQ score for brands with paid on | ≥ 7 | ≥ 8.5 |
| 9 | % uptime | 99.0% | 99.5% |

### 2.3 Non-goals

These are explicitly **not** goals of v1, even though they may seem adjacent. Naming them prevents scope creep.

- **Becoming a creative authoring tool.** Brands do not draft posts here; the DOL fleet does. The brand can edit copy and approve/reject; that is the limit of authoring.
- **Becoming a multi-channel ad manager.** Meta is the only paid channel in v1. TikTok and others are deferred.
- **Becoming a competitor to Triple Whale or Northbeam.** The dashboard reports Caast's contribution; it does not aspire to be the brand's full BI / attribution stack.

---

## 3. Personas and primary scenarios

### 3.1 Primary persona: SMB DTC brand owner

**Profile.** Founder or marketing lead at a Shopify DTC brand doing $200K–$3M annual revenue, 2–20 employees, primarily in beauty / wellness / pet verticals. Spends ~$1,000/month on Caast and $1K–$3K/month on Meta paid (often underperforming because of creative-supply scarcity and broken tracking). Time-starved; checks Caast 3–5 times per week, mostly mobile, sessions of 2–6 minutes.

**Goals when they open the product:**

- On Monday morning: *"Tell me what happened last week."* → Influence Intelligence radar + weekly highlights.
- Mid-week, after a board email: *"What's my ROAS / MER this month?"* → Attribution Dashboard, Conversion + Paid performance tabs.
- Twice a week: *"Approve the next batch of posts."* → Content Plan, per-DOL tabs.
- When inventory is tight: *"Pause anything promoting the low-stock SKU."* → Settings + Content Plan.
- When budget changes: *"Increase paid to $400/week."* → Settings, Paid campaigns card.

### 3.2 Secondary persona: agency / fractional CMO managing the brand

Some brands' Caast accounts are operated by an external marketer. The product must support the same UX without dedicated agency features in v1 — multi-brand views and white-labeling are explicit Phase 2 / Phase 3.

### 3.3 The five primary scenarios

| # | Scenario | Surfaces touched | Success criterion |
|---|---|---|---|
| 1 | Morning pulse check | Login → Influence Intelligence | Brand sees AUI, ROAS, content volume, top 3 highlights in < 30s |
| 2 | Approving the week's content | Login → Content Plan → per-DOL tabs | All pending posts reviewed and approved/edited in < 4 min |
| 3 | Reporting numbers up | Login → Attribution Dashboard → export or screenshot | Brand can answer "what's our ROAS?" with confidence in < 60s |
| 4 | Turning on / adjusting paid | Login → Settings → Paid campaigns | Brand sets weekly budget, picks Approval-mode or Auto-advertise, in < 90s |
| 5 | Promoting a winner manually | Content Plan → tap `$`-icon on a post → next morning Paid Campaign tab shows it live | One-tap promotion; live status visible within 24h |

---

## 4. Phasing strategy

v1 ships the full product surface area for the brand. Some specific capabilities are deferred to Phase 2 because of external dependencies (Meta App Review for Partnership Ads, DOL legal updates for whitelisting) or because they require operational telemetry from v1 to design well.

| Area | Phase 1 / MVP (months 0–3) | Phase 2 (months 4–7) |
|---|---|---|
| **Login & auth** | Email+password, 2FA via email or SMS, password reset | SSO (Google, Apple), passkeys |
| **Influence Intelligence** | Radar with 4 floating KPIs incl. ROAS, AI chatbox with deterministic suggestions, all 4 highlight tabs | AI chatbox open-ended Q&A, voice input, daily push digest |
| **Attribution Dashboard** | All 4 funnel tabs incl. Paid performance, period selector, charts | MER blended view, custom date ranges, CSV/PDF export |
| **Content Plan** | Per-DOL tabs, approve / edit / promote-to-paid actions, Approval-mode and Auto-publish | Bulk approve, scheduled approval rules, comment-thread replies |
| **Paid Campaign** | Per-DOL tabs, live ASC, per-post ROAS/CPA/Hook rate, brand-account ads | Partnership Ads (whitelisted DOL accounts), creative concept harvesting back to organic, per-platform breakdowns |
| **Settings** | Account, plan, connections, organic + paid Approval/Auto toggles, weekly budget cap | Multi-user accounts, role permissions, audit log |
| **Paid optimization (backend)** | Single ASC, lowest-cost bid, broad audience, Caast-managed creative pool, EMQ≥7 launch gate, +20%/3–4d scaling, learning-phase fallback to Add-to-Cart | Cost-cap bidding, multi-objective campaigns, automated kill rules with custom thresholds, Custom Audience auto-build from organic engagers |
| **Notifications** | In-app: pending approvals, EMQ degradation, low-stock auto-pause | Email digest, mobile push, Slack integration |

---

## 5. Login and authentication

The login flow gates every other surface. It must be fast, recoverable, and feel like a Caast surface, not a stock auth library.

### 5.1 Visual specification (HTML reference)

Two sequential steps inside a centered card on the dark-pink Caast background.

**Step 1.** Email + password with show/hide toggle, Remember-me checkbox, Forgot-password link, primary CTA "Log in" (disabled until both fields valid), and a footer "Start your free activation" link.

**Step 2.** Two-step verification — method picker (Email or Phone, pre-filled with masked address e.g. `r***l@glosslab.com` and `***-***-0187`), then a 6-digit code input with auto-focus across boxes, resend countdown timer (60s), and a final success ring before transitioning into Influence Intelligence.

### 5.2 [FRONTEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `UI-101` | Render login screen at unauthenticated route `/login`. Page transitions to product on success. | MVP |
| `UI-102` | Step 1 form: email field (validates format on blur), password field with show/hide toggle. "Log in" button disabled until both fields are non-empty and email format is valid. | MVP |
| `UI-103` | On submit, show inline error under the offending field for the two known cases: "Account not found" (email) and "Incorrect password" (password). Generic auth errors fall back to a top-of-card banner. | MVP |
| `UI-104` | "Remember me" checkbox checked by default. When checked, refresh-token TTL extends to 30 days; unchecked, 24 hours. | MVP |
| `UI-105` | "Forgot password?" link opens a separate flow that sends a reset email. Out of scope for the main login flow but reachable. | MVP |
| `UI-106` | Step 2 method picker: two cards (Email, Phone) with masked details, single-select. Default to Email if both available. | MVP |
| `UI-107` | On Send code, show 6-digit code input. Auto-advance focus from box to box on input; auto-back on Backspace from empty box. Submit on the 6th digit. | MVP |
| `UI-108` | Resend code link appears below the input, disabled with countdown (60s) before becoming active. Three resends per hour cap. | MVP |
| `UI-109` | On code success, show the Caast green checkmark ring for 800ms before navigating to Influence Intelligence. Animation must not block keyboard navigation. | MVP |
| `UI-110` | If Auth API returns rate-limit error, show a banner: "Too many attempts. Try again in N minutes." | MVP |
| `UI-111` | "Start your free activation" footer link routes to the activation flow at `/activate` (handled by the landing-page product). | MVP |
| `UI-112` | Mobile responsive down to 360px. Code-input boxes never overflow. | MVP |
| `UI-113` | SSO buttons (Google, Apple) above the divider line. Replace the divider "or" with the SSO row. | Phase 2 |

### 5.3 [BACKEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `BE-101` | `POST /api/v1/auth/login` — accepts `{email, password}`, returns either `{step: "2fa", challenge_id, methods: [{type, masked}]}` or, for accounts without 2FA enabled (rare, but admin-overridable), `{step: "complete", access_token, refresh_token}`. | MVP |
| `BE-102` | `POST /api/v1/auth/2fa/send` — accepts `{challenge_id, method: "email"\|"phone"}`, sends 6-digit code via the chosen channel. Code TTL 10 minutes. Returns `{sent: true, expires_at}`. | MVP |
| `BE-103` | `POST /api/v1/auth/2fa/verify` — accepts `{challenge_id, code}`, returns `{access_token (JWT, 15-min TTL), refresh_token (rotating, TTL based on Remember-me), user, brand_id}`. | MVP |
| `BE-104` | `POST /api/v1/auth/refresh` — rotating refresh token. Old refresh tokens are invalidated on successful refresh. | MVP |
| `BE-105` | `POST /api/v1/auth/logout` — invalidates the refresh token. Access token expires naturally. | MVP |
| `BE-106` | `POST /api/v1/auth/password/forgot` — accepts `{email}`, sends reset link if account exists. Always returns 200 (do not leak account existence). | MVP |
| `BE-107` | `POST /api/v1/auth/password/reset` — accepts `{reset_token, new_password}`. Reset tokens are single-use, 1-hour TTL. | MVP |
| `BE-108` | Password rules: minimum 10 characters, at least one number, must not be in the bcrypt-hashed top-10K common passwords list. Bcrypt cost 12. | MVP |
| `BE-109` | Rate limiting: 5 failed login attempts per email per 15 min triggers 15-min lockout; 10 failed 2FA verifications per challenge invalidates the challenge. | MVP |
| `BE-110` | All auth events (login success, login failure, 2FA send, 2FA verify, password reset) write to an immutable `auth_audit` log with IP, user agent, timestamp. | MVP |
| `BE-111` | SSO via Google OIDC and Apple. SSO accounts skip 2FA on the same device fingerprint within 30 days. | Phase 2 |

---

## 6. Screen 1 — Influence Intelligence

The product's home screen and the brand's morning pulse. A live radar visualization at the center, four floating KPI cards positioned around it, an AI chatbox below, and a four-tab highlights panel beneath that.

### 6.1 Visual specification

Top bar: Caast logo left, page title "Influence intelligence" centered, brand tag (e.g. GlossLab) right. Status row beneath: green pulsing dot + "Influence network active" + "Live! vs. last 7D" timestamp.

Center stage: animated radar with sweeping line, breathing polygon connecting DOL data points, occasional blip dots for events; in the radar's center, a stack reading `18,240 / Audience under influence / +22% vs. last week`.

Four floating KPI cards positioned at top, right, bottom, left of the radar:

- **Top:** Reached audience — large value, label, +delta, color `#F43460`
- **Right:** Return on ad spend — large value (e.g. `3.4x`), label, +delta, color `#EA7A3B` *(this replaces the previous "Equivalent ad spend" KPI)*
- **Bottom:** Content volume — e.g. `63`, with sub-label "3 DOLs active", color `#EC274F`
- **Left:** Equivalent labor saved — e.g. `$2,520 / 63hrs of work`, color `#81226B`

AI chatbox: Caast magenta border, sparkle icon, "Ask me anything" header. Input field with rotating ghost-text placeholders (cycle every ~3 seconds, pause when user types or focuses).

The 4-tab highlights panel below toggles between "This week's highlights" (default), "Business impact", "Fleet performance", "Competitive intel". Each tab contains a mix of full-width highlight cards (with badge, name, description, right-side value) and grid mini-cards (4-up KPI grid).

### 6.2 [FRONTEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `UI-201` | Render Influence Intelligence at `/app` or `/app/intelligence`. Default landing surface after login. | MVP |
| `UI-202` | Render the radar SVG with: continuous sweep animation (18s loop), breathing polygon (3s pulse), blip dots on data events (fade in/out 2s). | MVP |
| `UI-203` | Position 4 floating KPI cards: top-center, right-middle, bottom-center, left-middle of the radar. Cards must not overlap radar axes. | MVP |
| `UI-204` | KPI cards display: large value, label, delta with up/down arrow, color per design tokens. Each card receives data from `GET /api/v1/intelligence/{brand_id}/kpis`. | MVP |
| `UI-205` | The Return on ad spend KPI card replaces the previous Equivalent ad spend card. The card is rendered even when the brand has no paid running — in that case it shows `—` and a "Connect paid" link. | MVP |
| `UI-206` | AI chatbox renders below radar with: sparkle icon, header "Ask me anything", text input, send button. Ghost-text placeholder cycles through `aiExamples` (initial set: "Why did engagement spike on Tuesday?", "What is my CPA this week?", "What should I promote next based on inventory?"). | MVP |
| `UI-207` | Placeholder cycling pauses on input focus and when input contains text; resumes on blur if input is empty. Cycle interval ~3s. | MVP |
| `UI-208` | On send: clear input, show "Thinking…" in ghost, call `POST /api/v1/intelligence/{brand_id}/ask`, render response as a chat bubble below the input. | MVP |
| `UI-209` | Highlights panel: 4 tabs (This week's highlights, Business impact, Fleet performance, Competitive intel). Default to "This week's highlights". Tab switch animates content panel cross-fade 200ms. | MVP |
| `UI-210` | "This week's highlights" panel renders highlight cards with badges (Trending green, Competitive amber, Inventory yellow, Opportunity magenta). Each card: badge column, body (name + description), right value column. | MVP |
| `UI-211` | "Business impact" panel renders a 4-up grid of mini-cards: Equivalent ad spend saved (organic), Attributed revenue (paid), Return on investment (organic), Return on ad spend (paid). Plus a featured card highlighting the top-performing promo or campaign. | MVP |
| `UI-212` | "Fleet performance" panel renders a 4-up grid: Audience engagement rate, Conversions, Followers growth, Website traffic. Plus a featured card on comment sentiment with positive %. | MVP |
| `UI-213` | "Competitive intel" panel renders highlight cards covering brand visibility share-of-voice, competitor ad spend changes, and quiet wins (unprompted mentions). | MVP |
| `UI-214` | Each panel exposes a period selector (`7D / 1M / YTD / Max`) that re-fetches the panel's data scoped to the period. | MVP |
| `UI-215` | All KPI deltas show up/down arrow with color: `#5DCAA5` for up (positive), `#EA7A3B` for down (negative), `#B8B6AF` for flat. | MVP |
| `UI-216` | Loading state: skeleton screens for KPI cards and highlight cards during initial fetch. No layout shift. | MVP |
| `UI-217` | Error state: if any KPI source fails, show `—` with a small refresh icon. Other KPIs render normally. | MVP |
| `UI-218` | AI chatbox accepts open-ended natural language queries. v1 supports a deterministic set of templated questions; out-of-template queries return a graceful "I can't answer that yet, but I can show you…" with deep-link suggestions. | Phase 2 |

### 6.3 [BACKEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `BE-201` | `GET /api/v1/intelligence/{brand_id}/kpis` — returns the 4 floating KPIs plus the radar center value (Audience Under Influence). Response includes value, delta, period, color hint, and source provenance. | MVP |
| `BE-202` | AUI computation: people exposed to DOL content 3+ times in past 30 days who took at least one intent action (like, save, share, click, comment, follow). Computed nightly from social platform engagement data, cached 4h. | MVP |
| `BE-203` | Reached audience: unique audience reached by DOL content in the period, deduplicated across DOLs where possible. From IG / TikTok / Pinterest reach APIs. | MVP |
| `BE-204` | Return on ad spend: paid attributed revenue ÷ paid ad spend, period-scoped. Returns null when no paid spend in period. | MVP |
| `BE-205` | Content volume: count of DOL posts published by the fleet in the period, broken down by DOL and platform. | MVP |
| `BE-206` | Equivalent labor saved: `content_volume × hours_per_post (default 1) × hourly_rate (default $40)`. Brand-configurable in settings (Phase 2). | MVP |
| `BE-207` | `GET /api/v1/intelligence/{brand_id}/highlights?tab={tab}&period={period}` — returns the cards for the requested tab. Supported tabs: `weekly`, `business_impact`, `fleet_performance`, `competitive`. | MVP |
| `BE-208` | Highlight card scoring: each candidate signal (trend match, competitive event, inventory event, opportunity, performance milestone) is scored on novelty, magnitude, and brand relevance. Top N per tab returned. | MVP |
| `BE-209` | `POST /api/v1/intelligence/{brand_id}/ask` — accepts `{query}`, returns `{answer, supporting_data, deep_link?}`. v1 routes through a templated intent classifier covering ~20 supported question types (engagement, CPA, ROAS, content recommendations, inventory). | MVP |
| `BE-210` | Question templates supported in v1: engagement spike explanation, CPA / ROAS / MER lookup, top-performing post, content recommendation given inventory, follower growth, competitor activity summary, fleet status. Out-of-template returns the "I can't answer that yet" graceful fallback with a deep-link suggestion. | MVP |
| `BE-211` | Highlight pipeline writes structured `Highlight` records (see Data Models) with TTL based on signal type: Trending (24h), Competitive (3d), Inventory (live until resolved), Opportunity (until campaign ends). | MVP |
| `BE-212` | All KPI endpoints support `period={7d, 1m, ytd, max}` with the previous period auto-derived for delta computation. | MVP |
| `BE-213` | Open-ended AI Q&A backed by an LLM with read-only access to the brand's data via a function-calling interface. PII redaction applied before LLM context. | Phase 2 |

---

## 7. Screen 2 — Attribution Dashboard

The dashboard quantifies what the influence engine produces — organically and through paid amplification. Four funnel-aligned tabs (Awareness, Consideration, Conversion, Paid performance), each with a KPI grid and a chart, plus a global period selector.

### 7.1 Visual specification

Top bar identical to Influence Intelligence with title "Attribution dashboard". Period selector pill row (`7D / 1M / YTD / Max`). Below, the 4-tab horizontal bar with colored dot icons:

- Awareness (`#F43460`)
- Consideration (`#F43460`)
- Conversion (`#F43460`)
- Paid performance (`#EA7A3B`)

Each panel renders a KPI grid above and a chart below.

### 7.2 Per-tab KPI specification

| Tab | KPIs | Chart |
|---|---|---|
| Awareness | Impressions, Unique reach, Follower growth, Content volume, CPM | Daily impressions vs unique reach, last period |
| Consideration | Engagement rate, Saves & shares, Comment sentiment, Website traffic from DOLs, CPE | Daily engagement events stacked by type |
| Conversion | AUI funnel (Aware/Engaged/Converted), Attributed orders, Attributed revenue, Conversion rate, CPA | Funnel chart + revenue trend |
| Paid performance | ROAS, CPA, CPM, CTR, Hook rate, Conversions | Daily paid spend vs revenue with attribution mini-cards (revenue $, orders count) for the period |

### 7.3 [FRONTEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `UI-301` | Render dashboard at `/app/dashboard`. Persist last-active tab and period in URL query string for shareable links. | MVP |
| `UI-302` | Period selector applies globally across all 4 tabs. Switching period re-fetches the active tab; other tabs lazy-fetch on switch. | MVP |
| `UI-303` | KPI cards render: label, large value (with currency / count formatting), delta vs previous period with up/down/flat indicator, sub-label. | MVP |
| `UI-304` | **Awareness tab** — 5 KPI cards: Impressions, Unique reach, Follower growth, Content volume, CPM. CPM compares to industry avg (e.g. "vs IG avg $8.50") in the sub-label. | MVP |
| `UI-305` | **Consideration tab** — 5 KPI cards: Engagement rate, Saves & shares, Comment sentiment, Website traffic, CPE. | MVP |
| `UI-306` | **Conversion tab** — KPIs: AUI funnel (3-stage visualization), Attributed orders, Attributed revenue, Conversion rate, CPA. | MVP |
| `UI-307` | **Paid performance tab** — 5 KPI cards: ROAS, CPA, CPM, CTR, Hook rate, Conversions. (See section 9 for full paid metric definitions.) | MVP |
| `UI-308` | Each tab renders a chart below the KPI grid using Chart.js or Recharts. Tooltips on hover show value + date. | MVP |
| `UI-309` | On the chart, append a header showing the chart title and a legend swatch row. To the right of the period stamp on the bottom-of-chart, render two attribution mini-cards: revenue figure (`$`) and orders count for the period. | MVP |
| `UI-310` | Loading state: KPI cards render as skeleton; chart shows a centered shimmer block. | MVP |
| `UI-311` | Empty state when no paid data: Paid performance tab shows a single CTA card "Turn on paid amplification" — deep link to Settings › Paid campaigns. | MVP |
| `UI-312` | CSV / PDF export per tab. Custom date range picker. | Phase 2 |

### 7.4 [BACKEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `BE-301` | `GET /api/v1/dashboard/{brand_id}/awareness?period={p}` — returns the 5 Awareness KPIs and a daily series for the chart. | MVP |
| `BE-302` | `GET /api/v1/dashboard/{brand_id}/consideration?period={p}` — returns the 5 Consideration KPIs and series. | MVP |
| `BE-303` | `GET /api/v1/dashboard/{brand_id}/conversion?period={p}` — returns conversion funnel + KPIs + revenue series. | MVP |
| `BE-304` | `GET /api/v1/dashboard/{brand_id}/paid?period={p}` — returns paid KPIs + daily spend/revenue series + total revenue & orders for the period. | MVP |
| `BE-305` | Attribution model v1: order is attributed to a DOL touchpoint if the customer interacted with DOL content (engaged or clicked) within 30 days before purchase, identified by promo code, UTM, or pixel/CAPI fingerprint match. **Last-touch attribution** among DOL touchpoints. | MVP |
| `BE-306` | Paid attribution: order is attributed to paid if the customer last clicked a paid ad within the Meta-default **7-day click + 1-day view** window. Aligns with Meta's standard attribution to keep platform reports and Caast reports reconcilable. | MVP |
| `BE-307` | `CPM (paid) = (paid spend ÷ paid impressions) × 1000` | MVP |
| `BE-308` | `CTR (paid) = paid clicks ÷ paid impressions` | MVP |
| `BE-309` | `Hook rate (paid, video) = 3-second video views ÷ video impressions` | MVP |
| `BE-310` | `CPA (paid) = paid spend ÷ attributed orders`. Returns null when `attributed_orders = 0`. | MVP |
| `BE-311` | `ROAS (paid) = attributed revenue (paid) ÷ paid spend` | MVP |
| `BE-312` | `Engagement rate (organic) = (likes + comments + shares + saves) ÷ follower count`, computed per platform and aggregated. | MVP |
| `BE-313` | Comment sentiment: NLP classifier returning `{positive, neutral, negative}` with confidence. Aggregated over the period. | MVP |
| `BE-314` | All metrics cached at 15-min granularity. Cache key includes `brand_id, metric_id, period, last_event_timestamp`. | MVP |
| `BE-315` | MER computed by ingesting brand's total revenue (Shopify) and total marketing spend (Meta + any other channels the brand connects). Surfaced as a separate "True ROI" card distinct from platform ROAS, with an explanatory tooltip. | Phase 2 |
| `BE-316` | CSV / PDF export endpoints per tab. | Phase 2 |

---

## 8. Screen 3 — Content Plan

Where the brand reviews, approves, edits, or paid-promotes the content the DOL fleet produces. Per-DOL tabs at the top; each panel renders the next 3 posts per DOL with full visual preview, caption, trigger context, engagement (when live), and three action buttons: **Approve**, **Edit**, **`$`** (paid promotion toggle).

### 8.1 Visual specification

Top bar identical to other surfaces with title "Content plan". Below: a horizontal tab bar with one tab per DOL in the brand's fleet. Each tab shows a circular avatar (initials, gradient background per DOL color) and the DOL's display name.

The active panel renders a 3-column grid of post cards. Each card has:

- a header (DOL avatar + handle + platform badge + scheduled time)
- a square visual preview area (rendered SVG mock or actual image when published)
- a trigger row showing what signal generated the post (e.g. `Trending #cleangelnails +340%`)
- the caption (handle + text + hashtags)
- an engagement row when published (likes / comments / shares / saves)
- a 3-button action row: Approve, Edit, `$`

### 8.2 [FRONTEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `UI-401` | Render Content Plan at `/app/content`. Persist active DOL tab in URL. | MVP |
| `UI-402` | Render one tab per DOL assigned to the brand. Tab order matches fleet roster. Avatar background uses the DOL's signature gradient. | MVP |
| `UI-403` | Each panel renders up to 3 upcoming or recently published posts per DOL, ordered by `scheduled_time` ascending. Lazy-load older posts on scroll. | MVP |
| `UI-404` | Post card structure: header (avatar + handle + IG/TT/PIN badge + scheduled time), visual area, trigger badge + label, caption block, engagement row (when status = `published`), 3-button action row. | MVP |
| `UI-405` | Trigger badges: Trending (`#5DCAA5` on rgba 12% bg), Competitive (`#EA7A3B` on 15%), Inventory (`#D4A656` on 12%), Opportunity (`#F43460` on 10%), Product signal (`#EC274F` on 12%), High performer (`#81226B` on 10%). The full list is in the Highlight badges palette. | MVP |
| `UI-406` | Approve button: tap toggles to the "approved" state, label changes to "Approved", sends POST to backend, button visually green-tinted and disabled. | MVP |
| `UI-407` | Edit button: opens an inline editor modal with caption text, hashtags, and platform-specific overrides. Save calls PATCH backend; close without save discards changes. | MVP |
| `UI-408` | `$` button: toggles paid-promotion intent for this post. Two states: gray (default, not promoted) and pink (`#F43460`, promoted/intended). Tap calls `POST /api/v1/posts/{id}/paid-toggle`. | MVP |
| `UI-409` | When the brand is in Approval-mode for organic, posts default to `status=pending_approval` and the Approve button is the primary action. When Auto-publish is on, posts default to `status=auto_approved` and the Approve button is hidden (or shown as "Auto-approved" disabled). | MVP |
| `UI-410` | Engagement row only renders when `status=published`. Counts pulled from social platform APIs. | MVP |
| `UI-411` | Empty state per DOL: "No posts scheduled yet. The fleet is composing the next batch — they'll appear here within the hour." | MVP |
| `UI-412` | Loading state: skeleton post cards with shimmer. | MVP |
| `UI-413` | Bulk select (checkbox per card) with Approve-All / Reject-All actions. Inline comment thread on each post (brand can leave notes for the fleet). | Phase 2 |

### 8.3 [BACKEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `BE-401` | `GET /api/v1/posts?brand_id={b}&dol_id={d}&status={s}&limit={n}` — returns posts for the brand, optionally filtered. Default ordering: `scheduled_time ASC`. | MVP |
| `BE-402` | `POST /api/v1/posts/{id}/approve` — transitions status from `pending_approval` to `approved`. Triggers downstream publish job. 200 returns updated post. | MVP |
| `BE-403` | `POST /api/v1/posts/{id}/reject` — transitions status to `rejected`. Optional `{reason}` body. Notifies fleet of feedback. | MVP |
| `BE-404` | `PATCH /api/v1/posts/{id}` — accepts `{caption?, hashtags?, scheduled_time?}`. Validates platform-specific constraints (caption length, hashtag count). Status remains `pending_approval` after edit. | MVP |
| `BE-405` | `POST /api/v1/posts/{id}/paid-toggle` — toggles the `paid_intent` flag. When true, the post becomes a candidate for the Paid Autopilot to amplify (subject to engagement thresholds and budget). When false, paid amplification is suppressed for this post. | MVP |
| `BE-406` | Posts data model includes: `id, brand_id, dol_id, platform, status, scheduled_time, caption, hashtags, visual_assets, trigger (signal type + reference), engagement (when published), paid_intent (bool), paid_status (enum: not_promoted, intended, queued, live, completed, rejected_by_meta)`. | MVP |
| `BE-407` | Approval-mode logic: when `brand.organic_mode = approval`, every new post is created with `status=pending_approval`. When `organic_mode = auto`, `status=auto_approved` and the publish job runs immediately (subject to `scheduled_time`). | MVP |
| `BE-408` | Approval-mode → Auto-publish transition cancels Approval-mode immediately: any `pending_approval` posts older than 24h auto-promote to `auto_approved` on switch. | MVP |
| `BE-409` | Comment-sentiment ingestion: published posts have engagement and comments synced every 30 min for the first 48h, then hourly for 7d, then daily. | MVP |
| `BE-410` | Bulk approval endpoint `POST /api/v1/posts/bulk-approve` with `{post_ids[]}`. | Phase 2 |

---

## 9. Screen 4 — Paid Campaign

Live view of which DOL posts are amplified on Meta and how they're performing. Same per-DOL tab structure as Content Plan, but the panels show only posts with `paid_status=live`, plus per-post ROAS, CPA, and Hook rate KPIs. A live status bar at the top confirms the autopilot is actively optimizing.

> **This section is anchored in `meta_ads_tutorial.md` Sections 5–7:**
> - **Section 5** grounds the creative format requirements (the 9:16 / 1:1 / Story / Carousel / static frame variants and the concept-level testing framework)
> - **Section 6** grounds the metric stack (CPM, CTR, Hook Rate, CVR, CPA, ROAS, MER) and the Pixel + CAPI + EMQ≥7 launch gate
> - **Section 7** grounds the learning-phase economics (50 events/week, Add-to-Cart fallback) and the 20%-every-3–4-days vertical scaling rule

### 9.1 Visual specification

Top bar with title "Active paid campaign" and subhead "9 posts amplified across 3 DOLs · live on Meta". Beneath: a live-bar with pulsing green dot and label "Paid campaign is live · auto-optimizing on Meta".

Per-DOL tab bar identical to Content Plan. Each panel renders the same 3-column post grid, but each card now includes a 3-up KPI mini-row beneath the action buttons: **ROAS**, **CPA**, **Hook rate** — same horizontal alignment, vertical alignment with the post card width. The `$`-action button on each card is always pink (already promoted).

### 9.2 [FRONTEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `UI-501` | Render Paid Campaign at `/app/paid`. Visible only when brand has paid enabled (otherwise show empty CTA "Connect Meta to enable paid amplification" that deep-links to Settings). | MVP |
| `UI-502` | Live status bar at top of screen: pulsing green dot + label "Paid campaign is live · auto-optimizing on Meta". When campaign is paused (e.g. EMQ degradation), bar turns amber and label changes to "Campaign paused — fix tracking to resume" with a CTA. | MVP |
| `UI-503` | Per-DOL tab bar identical to Content Plan. Tabs render only DOLs with at least one live paid post. | MVP |
| `UI-504` | Each panel renders posts where `paid_status=live`. Card layout matches Content Plan, with ROAS/CPA/Hook rate KPI row appended beneath the 3 action buttons. | MVP |
| `UI-505` | ROAS color coding: ≥3.0 `#5DCAA5` (good), 2.0–2.99 `#EA7A3B` (warning), <2.0 `#EC274F` (critical). CPA and Hook rate styled neutrally. | MVP |
| `UI-506` | `$` button on each card always pink. Tapping it offers the option to remove from paid (confirmation dialog "Stop amplifying this post? It will continue to live organically"). | MVP |
| `UI-507` | Empty state: "No paid posts live yet. The autopilot is selecting candidates from organic winners. First posts typically go live within 24 hours of enabling paid." | MVP |
| `UI-508` | Per-post breakdown drawer (tap card to expand) showing creative variant performance (which of the 5 reformatted variants is winning), platform breakdown (Feed vs Reels vs Stories), and a kill-this-creative button. | Phase 2 |

### 9.3 [BACKEND] requirements — Paid Autopilot service

The Paid Autopilot is the largest backend system in this PRD. It manages Meta campaigns autonomously through the **Meta MCP layer** once the brand fixes a weekly budget. Caast never owns the brand's ad account; it operates in the brand's own Meta Business Manager via the MCP-mediated permission grant established during connection.

#### 9.3.1 Pre-launch gates

| ID | Requirement | Phase |
|---|---|---|
| `BE-501` | Paid cannot launch unless ALL of: (a) Meta BM connected with valid token; (b) Shopify connected; (c) CAPI events flowing for the last 7 days with EMQ ≥ 7; (d) at least 8 organic DOL posts published in the last 14 days that meet engagement floor (eligible creative pool); (e) brand has set `weekly_budget > 0`; (f) brand has acknowledged the paid disclosure (one-time modal in Settings). | MVP |
| `BE-502` | If any gate fails, surface a structured `GapList` to the frontend so the Settings page can show the exact unmet condition and a remediation CTA. | MVP |
| `BE-503` | EMQ continuously polled from Meta. If EMQ drops below 7 for 24h while paid is active, autopilot auto-pauses (`paid_status=paused_emq`) and surfaces an in-product alert. | MVP |

#### 9.3.2 Campaign structure (anchored in tutorial §2, §7)

| ID | Requirement | Phase |
|---|---|---|
| `BE-510` | Create exactly one Advantage+ Sales Campaign (ASC) per brand in the brand's Meta BM via MCP. Objective: Sales. Optimization event: Purchase (with Add-to-Cart fallback per `BE-512`). | MVP |
| `BE-511` | Audience: country (from brand profile) + age 18–65. **No interest targeting, no lookalikes in v1.** Existing customer cap default 50%, brand-adjustable in Phase 2. | MVP |
| `BE-512` | **Learning-phase fallback:** if brand averages < 25 purchase events/week over the previous 4 weeks, default optimization event = Add-to-Cart instead of Purchase. Surface this decision to the brand via a one-time toast: "Optimizing for Add-to-Cart while we build conversion volume" with a learn-more link. | MVP |
| `BE-513` | Bid strategy: **lowest-cost** (Meta default). Cost-cap and bid-cap are Phase 2. | MVP |
| `BE-514` | Budget allocation: campaign-level daily budget = `weekly_budget ÷ 7`. Hard ceiling: never exceeds `weekly_budget` over any rolling 7-day window. | MVP |
| `BE-515` | **Vertical scaling rule:** while campaign ROAS ≥ `brand_target_roas` (default 2.0), increase daily budget by max **+20% every 3–4 days**. Exit scaling and hold when ROAS drops below target for 2 consecutive days. | MVP |

#### 9.3.3 Creative pool management (anchored in tutorial §5)

| ID | Requirement | Phase |
|---|---|---|
| `BE-520` | **Eligibility engine:** a DOL post becomes a paid candidate when (a) brand toggled `$`-icon on, OR (b) post exceeds DOL's rolling 7-day median on hook rate, save rate, or share rate (auto-eligibility threshold), OR (c) post generated purchase-intent comments detected by NLP. | MVP |
| `BE-521` | **Creative variant generation:** for each eligible post, the Production Factory generates up to **5 ad variants**: 9:16 Reel (vertical video), 1:1 Feed (square), 9:16 Story, Carousel (when source has multi-image), static frame from video. Variant generation is non-blocking; ads launch with whatever variants are ready. | MVP |
| `BE-522` | Variant assets are uploaded to Meta as Ad Creatives via MCP. Each Ad Creative carries metadata: `source_post_id, dol_id, variant_format, hook_type, format, talent`. | MVP |
| `BE-523` | **Concept-level testing:** ads are grouped by *concept* (the angle, e.g. "before/after", "honest comparison", "save the date promo") per the tutorial §5 framework. Minimum **4-day, $500/concept window** before declaring a winner. | MVP |
| `BE-524` | **Iteration on winners:** once a concept wins, `BE-520` prioritizes future eligible posts in the same concept family for next-cycle promotion. | MVP |
| `BE-525` | Caption + CTA: ad copy comes from the source post; if the source caption exceeds Meta primary-text limits (~125 chars optimal), generate a tightened variant via LLM with the same Brand DNA. Always include a CTA (Shop Now default; brand-overridable in Phase 2). | MVP |
| `BE-526` | **Brand-account ads only in v1.** All ads run from the brand's Facebook Page / Instagram Business account. Partnership Ads (whitelisted from DOL accounts) deferred to Phase 2 (depends on DOL legal terms updates and Meta App Review for Branded Content / Partnership Ads Hub permissions). | MVP / Phase 2 |

#### 9.3.4 Tracking and EMQ (anchored in tutorial §6)

| ID | Requirement | Phase |
|---|---|---|
| `BE-530` | **Conversions API gateway:** Caast operates a CAPI gateway service that forwards Shopify webhook events (`orders/create`, `checkouts/create`) to Meta with hashed `user_data` (email, phone, fbp, fbc, client_ip, client_user_agent) per Meta's PII spec. | MVP |
| `BE-531` | **Pixel deduplication:** events sent via both Pixel (browser) and CAPI (server) share an `event_id` to allow Meta to deduplicate. The Caast pixel snippet auto-injected via Shopify Custom Pixels API where supported; otherwise the brand is given a copy-paste snippet. | MVP |
| `BE-532` | **EMQ monitoring:** poll Meta's EMQ score every 6 hours for each event type (Purchase, AddToCart, InitiateCheckout, ViewContent). Surface the lowest score in Settings → Paid campaigns. **Auto-pause autopilot scaling** (not autopilot itself) if any tracked event drops below EMQ 6. | MVP |
| `BE-533` | Attribution window: **7-day click + 1-day view** (Meta default). Surfaced as the basis of paid attribution numbers on the dashboard. | MVP |
| `BE-534` | Support for Conversions API for Conversion Leads, custom event mapping, and a server-side test events tool inside Settings. | Phase 2 |

#### 9.3.5 Kill rules and safety (anchored in tutorial §7)

| ID | Requirement | Phase |
|---|---|---|
| `BE-540` | Hourly job evaluates each ad against kill criteria. Auto-kill if: CPA exceeds `1.5× brand_target_cpa` for 3 consecutive days; OR frequency exceeds 4.0 within 7 days; OR CTR drops below 0.5% for 2 days. Surface kill events in the in-app notification stream. | MVP |
| `BE-541` | **Brand-safety overlay:** any ad on a post containing inventory-sensitive promo (e.g. SPRING15) is killed immediately if Shopify inventory webhooks indicate stock < 25 units for promoted SKUs. | MVP |
| `BE-542` | **Meta policy disapprovals:** when an ad is disapproved, surface the rejection reason on the Paid Campaign card and queue an alternative variant from the same source post. | MVP |
| `BE-543` | Brand-configurable kill thresholds, cost caps, dayparting. | Phase 2 |

#### 9.3.6 Paid → organic feedback (the flywheel)

| ID | Requirement | Phase |
|---|---|---|
| `BE-550` | **Concept harvester:** when a concept wins in paid (≥3.0× ROAS over the 4-day testing window), emit a brief to VCI / Production Factory to generate 3 more organic posts in the same concept family within 7 days. | Phase 2 |
| `BE-551` | **Audience expansion learnings:** parse Meta breakdown reports for which broad audience segments respond to a winning creative; map signals back to which DOL voice / tone the fleet should lean into for organic. | Phase 2 |
| `BE-552` | **Retargeting fatigue feedback:** when retargeting CTR drops on a creative, signal the fleet to rotate to a different DOL voice for organic touch points to prevent same-face fatigue across paid + organic. | Phase 2 |

#### 9.3.7 Per-post KPI surface

| ID | Requirement | Phase |
|---|---|---|
| `BE-560` | `GET /api/v1/posts/{id}/paid-kpis` — returns ROAS, CPA, Hook rate (and CTR, CPM in Phase 2) for the post's aggregate paid performance across all variants. | MVP |
| `BE-561` | KPI computation cadence: pulled from Meta Insights every 30 min for live ads, hourly for paused. Cached. | MVP |
| `BE-562` | When variant rollup is requested: `GET /api/v1/posts/{id}/paid-kpis?breakdown=variant` returns per-format performance. | Phase 2 |

---

## 10. Screen 5 — Settings

All configuration in one place. Four category tabs: **Your account**, **Your plan**, **Connections & content**, **Paid campaigns**. The two toggles that govern autonomous behavior — organic Approval-mode/Auto-publish and paid Approval-mode/Auto-advertise — live here, alongside the Meta Business Manager connection and weekly budget input.

### 10.1 Visual specification

Top bar with title "Settings". Beneath: 4-tab horizontal bar with colored dot icons:

- Your account (`#F43460`)
- Your plan (`#81226B`)
- Connections & content (`#EC274F`)
- Paid campaigns (`#EA7A3B`)

Each panel is an independent section with form fields, status rows, toggles, and CTAs.

### 10.2 Tab specifications

| Tab | Contents |
|---|---|
| **Your account** | Business name, contact name, email, phone (read-only with Edit), Change-password link |
| **Your plan** | Current plan name + price + active state, Change-plan link, billing details (cycle, next payment, member since), payment method card, Update payment method, Download invoices |
| **Connections & content** | Connect-all CTA, status row per connection (Shopify, Instagram, TikTok), Content-approval toggle (Approval-mode · Auto-publish) + recommendation banner |
| **Paid campaigns** | Meta Business Manager status row (or Connect CTA if not connected), Campaign-approval toggle (Approval-mode · Auto-advertise) + recommendation banner, Target weekly budget input ($/week) |

### 10.3 [FRONTEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `UI-601` | Render Settings at `/app/settings`. Persist active tab in URL (`?tab=account|plan|connections|paid`). | MVP |
| `UI-602` | **Your account tab:** prefill Business name, Contact name, Email, Phone from brand profile. Inputs use the Caast "prefilled" styling. "Change password" link triggers the standard reset flow. | MVP |
| `UI-603` | **Your plan tab:** show plan name + price (e.g. "Growth · $1,000/month"), Active badge, Change-plan link. Below: billing details grid. Below that: card-on-file (e.g. VISA ending 4242, expires 08/2028), Update-payment-method and Download-invoices links. | MVP |
| `UI-604` | **Connections & content tab:** row for each platform with icon, name, status (Connected / Not connected), and a connect/disconnect action. Below: Content-approval segmented toggle with two states (Approval-mode · Auto-publish) and the recommendation banner. | MVP |
| `UI-605` | Switching from Approval-mode to Auto-publish: shows a confirmation modal "Turn on Auto-publish? Pending posts will be auto-approved." On confirm, calls backend; banner text replaces with the auto-publishing copy. | MVP |
| `UI-606` | Switching from Auto-publish to Approval-mode: no confirmation needed; banner reverts; from now on every new post is gated by approval. | MVP |
| `UI-607` | **Paid campaigns tab:** Meta BM row showing connection status with green-pulse dot when connected. If not connected, the row is a Connect CTA that triggers the Meta connection flow (uses the Connection Guide pattern). | MVP |
| `UI-608` | Campaign-approval segmented toggle (Approval-mode · Auto-advertise) with recommendation banner: "Approval-mode is recommended for your first 30 days on paid. You review every campaign before it spends. Switch to Auto-advertise to publish and scale autonomously within your budget cap." Switching to Auto-advertise replaces banner with: "Fleet is advertising autonomously — paid campaigns launch and scale without approval, capped by your weekly budget. You can review every active campaign in your Paid campaign tab." | MVP |
| `UI-609` | Target weekly budget input: numeric, `$`-prefix, `/week`-suffix, min 0. Helper text: "Hard ceiling. Caast never spends above this. Daily ramp +20% every 3–4 days." Save on blur with debounce 800ms, show inline saved-confirmation tick. | MVP |
| `UI-610` | Tab disabled states: Paid campaigns tab is rendered as enabled but the Approval / budget controls are disabled until Meta BM is connected. Disabled controls show tooltip explaining the gate. | MVP |
| `UI-611` | Multi-user invitations, role permissions, audit log per setting change. | Phase 2 |

### 10.4 [BACKEND] requirements

| ID | Requirement | Phase |
|---|---|---|
| `BE-601` | `GET /api/v1/brands/{brand_id}/settings` — returns the full settings object (account, plan, connections, paid). | MVP |
| `BE-602` | `PATCH /api/v1/brands/{brand_id}/account` — update business name, contact name, phone. Email change requires re-verification. | MVP |
| `BE-603` | `GET /api/v1/brands/{brand_id}/plan` — returns current plan, price, billing cycle, next payment date, member since. `POST /api/v1/brands/{brand_id}/plan/change` initiates a plan change via Stripe. | MVP |
| `BE-604` | `GET /api/v1/brands/{brand_id}/billing/payment-method` — returns the masked card on file. `POST /api/v1/brands/{brand_id}/billing/payment-method/update` returns a Stripe SetupIntent for the frontend to complete. | MVP |
| `BE-605` | `GET /api/v1/brands/{brand_id}/billing/invoices` — returns a paginated invoice list with download URLs (Stripe-hosted PDFs). | MVP |
| `BE-606` | `GET /api/v1/brands/{brand_id}/connections` — returns connection state per platform (status, account ID, scopes_granted, last_refreshed). The Connection Guide service is the source of truth. | MVP |
| `BE-607` | `PATCH /api/v1/brands/{brand_id}/organic-mode` — accepts `{mode: "approval" | "auto"}`. On switch to "auto", all `pending_approval` posts older than 24h auto-promote to `auto_approved`. | MVP |
| `BE-608` | `PATCH /api/v1/brands/{brand_id}/paid-mode` — accepts `{mode: "approval" | "auto"}`. Validates that all paid pre-launch gates (`BE-501`) are satisfied before allowing "auto". | MVP |
| `BE-609` | `PATCH /api/v1/brands/{brand_id}/paid-budget` — accepts `{weekly_budget_cents}`. Hard ceiling: enforced by the autopilot. Lower is allowed at any time; higher is allowed but applies on the next learning-phase cycle. | MVP |
| `BE-610` | PATCH endpoints write to `settings_audit` log (Phase 2 surfaces this in UI). | MVP |
| `BE-611` | Multi-user, RBAC, audit log surface. | Phase 2 |

---

## 11. Cross-cutting backend services

Beyond the per-screen backends, three services support the whole product.

### 11.1 Notification service

| ID | Requirement | Phase |
|---|---|---|
| `BE-701` | In-app notification stream per brand, accessed via `GET /api/v1/notifications` and SSE / WebSocket subscription `/api/v1/notifications/stream`. | MVP |
| `BE-702` | Notification types in v1: `post_pending_approval`, `post_published`, `campaign_pending_approval`, `campaign_launched`, `campaign_paused_emq`, `ad_disapproved`, `low_stock_auto_pause`, `paid_winner_detected`, `weekly_summary_ready`. | MVP |
| `BE-703` | Each notification: `id, brand_id, type, severity (info / warn / critical), title, body, deep_link, created_at, read_at`. | MVP |
| `BE-704` | `POST /api/v1/notifications/{id}/read` marks a notification read. Bulk `mark-all-read` supported. | MVP |
| `BE-705` | Email digest, mobile push, Slack integration via webhook. | Phase 2 |

### 11.2 Aggregation / metrics service

A single service compiles the metrics feeding the radar, dashboard, and Paid Campaign KPIs. It runs on a schedule (15-min cadence for paid metrics, hourly for organic, nightly for AUI) and writes to a metrics warehouse.

| ID | Requirement | Phase |
|---|---|---|
| `BE-710` | **Sources:** Shopify (orders, products, inventory), Meta Insights API (paid impressions, spend, conversions, EMQ, breakdowns), Meta Graph API (organic IG insights), TikTok Display / Research APIs (organic), Pinterest (Phase 2), internal Caast event log (post publish events, approval events). | MVP |
| `BE-711` | **Output:** a metrics warehouse table per metric × brand × day, plus aggregations by period (`7d, 1m, ytd, max`). | MVP |
| `BE-712` | **Backfill on connection:** when Shopify or Meta is freshly connected, backfill the prior 90 days of data. Surface progress in Settings. | MVP |
| `BE-713` | **Reconciliation:** nightly job compares Caast attributed revenue vs Shopify total revenue and flags anomalies > 15%. | MVP |

### 11.3 Meta MCP integration layer

The Meta MCP layer is how Caast manages campaigns inside the brand's own Meta BM. Once the brand fixes the budget, every other action (campaign creation, ad creative upload, audience config, scaling, kill rules, creative rotation) flows through MCP.

| ID | Requirement | Phase |
|---|---|---|
| `BE-720` | Maintain a long-lived Meta BM access token per brand in the encrypted token vault. Rotation handled by the Connection Guide refresh service. | MVP |
| `BE-721` | All Meta API calls go through a single `MetaMCPClient` module that handles: rate limiting, retries with exponential backoff, error classification (transient vs permanent), audit logging. | MVP |
| `BE-722` | **Permission gate:** every write operation (create campaign, upload creative, change budget) checks the brand's `organic_mode + paid_mode` settings before executing. | MVP |
| `BE-723` | **Idempotency:** every write call carries a deterministic idempotency key derived from `{brand_id, operation, source_id, version}`. Re-issued calls are safe. | MVP |
| `BE-724` | **Failure handling:** 5xx errors trigger retry with backoff; 4xx errors classified — token errors trigger reconnection prompts, validation errors surface as in-product alerts, rate-limit errors back off. | MVP |
| `BE-725` | **Sandbox / staging:** a Meta BM sandbox account exists for development. Feature flag controls which token vault entries are sandbox vs production. | MVP |

---

## 12. Core data models

Listed in dependency order. Field types abbreviated; the developer is expected to translate to the chosen ORM.

### 12.1 Brand

| Field | Type | Description |
|---|---|---|
| `brand_id` | uuid | Primary key |
| `business_name` | string | Display name (e.g. GlossLab) |
| `contact_name` | string | Primary contact |
| `email` | string | Login email; unique |
| `phone` | string | E.164 format |
| `country` | iso2 | Country for ASC targeting |
| `plan_id` | fk | Reference to plans table |
| `organic_mode` | enum | `approval | auto` |
| `paid_mode` | enum | `approval | auto` |
| `weekly_budget_cents` | int | Paid hard ceiling in cents |
| `paid_target_roas` | float | Default 2.0 |
| `paid_target_cpa_cents` | int | Optional |
| `created_at, updated_at` | timestamp | Standard |

### 12.2 DOL (read-only from Fleet Studio)

| Field | Type | Description |
|---|---|---|
| `dol_id` | uuid | PK |
| `display_name` | string | Public DOL name |
| `handle_ig, handle_tiktok, handle_pinterest` | string | Per-platform handles |
| `vertical` | enum | `skincare | fragrance | supplements | mens_grooming | pet_wellness | menswear | jewelry` |
| `avatar_gradient` | `[hex,hex]` | DOL color signature |
| `assigned_brands` | `array<uuid>` | Brands using this DOL |

### 12.3 Post

| Field | Type | Description |
|---|---|---|
| `post_id` | uuid | PK |
| `brand_id` | fk | |
| `dol_id` | fk | |
| `platform` | enum | `ig | tiktok | pinterest` |
| `status` | enum | `pending_approval | approved | auto_approved | rejected | published | archived` |
| `scheduled_time` | timestamp | When the publish job will run |
| `caption` | text | |
| `hashtags` | `array<string>` | |
| `visual_assets` | `array<asset_ref>` | Images / videos in object storage |
| `trigger_type` | enum | `trending | competitive | inventory | opportunity | product_signal | high_performer` |
| `trigger_label` | string | e.g. "#cleangelnails +340%" |
| `engagement` | json | `{likes, comments, shares, saves}` when published |
| `paid_intent` | bool | Brand or autopilot has flagged for amplification |
| `paid_status` | enum | `not_promoted | intended | queued | live | paused | completed | rejected_by_meta` |
| `paid_kpis` | json | `{roas, cpa, hook_rate, ctr, cpm, conversions}` |
| `created_at, updated_at, published_at` | timestamp | |

### 12.4 Campaign / Ad / AdCreative

Mirrors Meta's hierarchy. One Campaign per brand, one or more Ad Sets per campaign (v1: a single ASC ad set), many Ads (one per source post + variant).

| Field | Type | Description |
|---|---|---|
| `campaign_id` | uuid | Caast PK |
| `meta_campaign_id` | string | Meta-side ID |
| `brand_id` | fk | |
| `objective` | enum | `sales` (v1) |
| `optimization_event` | enum | `purchase | add_to_cart` |
| `status` | enum | `draft | active | paused_by_brand | paused_emq | ended` |
| `daily_budget_cents` | int | |
| `weekly_cap_cents` | int | Hard ceiling |
| `created_at, updated_at` | timestamp | |

### 12.5 Highlight

| Field | Type | Description |
|---|---|---|
| `highlight_id` | uuid | PK |
| `brand_id` | fk | |
| `tab` | enum | `weekly | business_impact | fleet_performance | competitive` |
| `badge_type` | enum | `Trending | Competitive | Inventory | Opportunity | Product signal | High performer` |
| `title` | string | Card name |
| `description` | string | Card body |
| `right_value` | string | Right-column value |
| `right_sub` | string | Right-column sub-label |
| `score` | float | Ranking score |
| `expires_at` | timestamp | TTL by signal type |

### 12.6 ConnectionRecord

Owned by the Connection Guide service; this product reads it via `BE-606`.

---

## 13. Non-functional requirements, risks, references

### 13.1 Non-functional requirements

| ID | Requirement | Phase |
|---|---|---|
| `NFR-801` | Page time-to-interactive < 2s on 3G Fast simulated profile for the Influence Intelligence screen. | MVP |
| `NFR-802` | All API endpoints authenticated via JWT bearer. Refresh tokens rotating, hardware-bound when WebAuthn used. | MVP |
| `NFR-803` | All inbound webhook endpoints validate signatures (Shopify HMAC, Meta SHA256). | MVP |
| `NFR-804` | PII encrypted at rest (AES-256 envelope encryption via KMS). PII includes email, phone, payment-method tokens, OAuth tokens. | MVP |
| `NFR-805` | GDPR + CCPA compliant: data export and deletion endpoints. Brand data fully removable within 30 days of deletion request. | MVP |
| `NFR-806` | All money handling via Stripe; Caast never stores card data. | MVP |
| `NFR-807` | All metric values displayed to the brand must include a provenance trail accessible via developer tools (`X-Caast-Source` response header) for debugging. | MVP |
| `NFR-808` | Frontend bundle size < 400KB gzip for the main app. Chart libraries lazy-loaded per surface. | MVP |
| `NFR-809` | Logs scrub PII before egress. Sensitive fields tokenized in observability stack. | MVP |
| `NFR-810` | 99.0% monthly uptime SLO in MVP, measured at the API edge. Status page at `status.caast.ai`. | MVP |

### 13.2 Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Brand's CAPI is broken or EMQ < 7 | Paid optimization burns budget on bad signal | Pre-launch gate (`BE-501`); EMQ continuous monitoring (`BE-503`, `BE-532`); auto-pause on degradation |
| Brand's purchase volume too low for ASC learning phase | Campaign never exits learning; ROAS volatile | Add-to-Cart fallback (`BE-512`); surface fallback to brand |
| Meta MCP layer changes its surface area | Backend integration breaks | Encapsulate all Meta calls in `MetaMCPClient` (`BE-721`); contract tests against Meta sandbox |
| Meta policy disapproves DOL ads as AI-generated | Ads rejected at scale | Phase 2 Partnership Ads disclosure compliance; v1 brand-account ads only; ethical guardrail (DOLs curate, never simulate) reduces risk |
| Brand expects platform-ROAS = truth | Trust loss when MER reveals lower true ROI | Phase 2 MER surface (`BE-315`) with explanatory tooltip; communicate attribution honesty in onboarding copy |
| Switching to Auto-advertise causes runaway spend | Brand sees a $500 bad ad | Hard weekly cap (`BE-514`); kill rules (`BE-540`); 30-day Approval-mode default recommendation |
| Token vault breach | Multiple brand Meta accounts compromised | Per-brand envelope encryption keys; KMS audit logs; quarterly key rotation |

### 13.3 Assumptions

- v1 ships English-only. i18n is Phase 3.
- Brands are billed in USD. Multi-currency is Phase 3.
- Brands have at most one connected Meta BM, one Shopify store, and one IG / TikTok pair. Multi-store / multi-region is Phase 3.
- The DOL fleet, VCI, BA, and Production Factory are operating per their PRDs. This product treats them as upstream services with stable contracts.
- Stripe is the billing provider; the brand's plan + invoices live there.

### 13.4 References

- `caast_ai_product_paid.html` — visual specification for every screen described here.
- `caast_landing_page_paid.html` — the activation flow that precedes this product. Out of scope; covered by the landing-page PRD.
- **Caast Connection Guide PRD** — owns the Meta / Shopify / IG / TikTok OAuth flows referenced in `BE-606`.
- **Caast VCI / BA PRD** — owns the Vertical Content Intelligence and Brand Assimilation modules referenced as upstream sources for the Production Factory.
- `meta_ads_tutorial.md` — the strategic and tactical reference for paid social on Meta in 2026. **Sections 5 (Creative), 6 (Tracking), and 7 (Optimization) directly anchor the paid module requirements.**
- `Paid_product_discussion_high_level.docx` — the strategic memo behind the paid module. Source for the cross-pollination logic in Section 9.3.6.

---

*End of document. The visual specification (`caast_ai_product_paid.html`) is the canonical visual reference; this PRD is the canonical behavioral and backend reference. When in doubt about visual details, consult the HTML; when in doubt about behavior, consult this document.*
