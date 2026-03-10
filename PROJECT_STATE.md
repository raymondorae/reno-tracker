# Oikome — Project State Document

*Last updated: 7 March 2026*

---

## 🎯 Project Overview

**Product:** Oikome — a personal/household financial management platform  
**Tagline:** "Your household family office"  
**Vision:** Enterprise-grade financial planning made accessible to households

**Current phase:** MVP Build — Week 8 complete. Core features and demo polish done. Preparing for investor outreach.

**Live URLs:**
- **Landing page:** `https://getoikome.com` (custom domain live)
- **App:** `https://getoikome.com/login`
- **Vercel:** `oikome-91f10lx4o-raymondoraes-projects.vercel.app`

**Repo:** `github.com/raymondorae/Oikome` (private, branch: main)

**Local path:** `~/Documents/oikome/05-app`

**Tech stack:**
- **Frontend:** Next.js 16 (App Router), TypeScript, Tailwind CSS, shadcn/ui, Recharts
- **Backend:** Supabase (PostgreSQL + Auth + RLS + auto-generated API)
- **AI:** Anthropic API (Claude Sonnet for transaction categorisation + natural language scenario parsing)
- **Hosting:** Vercel (auto-deploys from GitHub)
- **Forms:** Tally (waitlist capture), Formspree (feedback collection)

---

## ✅ Completed Features

### Session: 7 March 2026 (continued)
- Auth loading animations — LoadingButton component using useFormStatus() on login and signup pages
- Floating feedback bubble on all dashboard pages — hover-expands from small icon to "Leave Feedback" pill, submits to Formspree (https://formspree.io/f/mnjgqeod), bug/feature/general types with success animation
- Removed email from dashboard header — shows first name from profiles table instead
- 6 real product screenshots added to landing page (dashboard, accounts, insights, budget, cashflow, scenarios) integrated into feature cards
- New Product Tour section on landing page with accounts and budget screenshots
- Improved landing page scroll indicator — "See how it works" text + bouncing chevron + smooth scroll to features section
- **Natural language scenario queries — THE DEMO MOMENT.** Users type plain English like "What if I go part-time?" and AI parses it into structured scenario modifications via Claude Sonnet API route. Includes example query chips, auto-generates scenario name/time horizon/modifications, rate limited 20 req/user/hour.
- Moved project documentation into repo under docs/ (BUG_LOG.md, PROJECT_STATE.md)

### Landing Page (getoikome.com)
- Hero section with headline "Your household family office"
- Problem/solution messaging
- Feature highlights (4 cards): Balance Sheet, AI Insights, Scenario Planning, Cash Flow
- Real product screenshots on each feature card
- Product Tour section with accounts and budget screenshots
- Social proof section
- Tally waitlist form embed (captures name + email)
- Sticky header with Sign In and Join Waitlist CTAs
- Scroll indicator with "See how it works" + bounce animation + smooth scroll
- Responsive design with dark theme
- Custom domain with SSL

### Access Control System
- **invite_codes table** in Supabase with:
  - code, expires_at, max_uses, use_count, created_for, is_active
  - RLS enabled (service role only)
- **Signup validation:** Requires valid invite code
- **Code features:** Single-use or multi-use, expiry dates, manual disable
- **Service role client** for secure server-side validation
- Test codes: DEMO2026 (30 days, 10 uses), INVESTOR-TEST (7 days, 1 use)
- Batch generation of single-use codes available

### Foundation & Auth
- Next.js project with TypeScript, Tailwind, shadcn/ui component library
- Supabase project with 14+ tables, all with row-level security (RLS)
- Email signup/login with session management + invite code validation
- LoadingButton on login/signup with automatic pending detection (useFormStatus)
- Middleware redirect for unauthenticated users
- Password reset flow (full end-to-end working)
- Forgot password link on login page
- GitHub repo connected to Vercel for auto-deploy
- Root URL shows landing page, /login for app access
- Email confirmation disabled for prototype speed

### Mock Data & Seeding
- **6 accounts:** Barclays Current, Barclays Savings, Amex Gold (credit card), Nationwide Mortgage, Vanguard ISA, Home property (manual)
- **12+ months** of transaction data (Mar 2025 – Mar 2026) with realistic UK merchant names
- **16 recurring transaction patterns** detected
- **12 monthly net worth snapshots** (upward trend)
- **4 uncategorised transactions** included for AI categorisation demo
- `seed_demo_data()` Supabase RPC function with auto-seed on signup
- Every new user sees populated dashboard immediately

### Dashboard (/dashboard)
- Personalised greeting: "Good morning/afternoon/evening, [First Name]" based on time of day
- First name displayed in header (email removed for privacy)
- Net Worth card with total assets, total liabilities, net worth, monthly change
- Net Worth trend chart (Recharts line chart, 12+ months)
- Accounts list grouped by type with balances
- Recent Transactions list (clickable, links to transaction detail)
- Loading skeleton with animate-pulse placeholders
- Empty state: "Welcome to Oikome" with Add Account CTA
- Page load fade-in animations

### Accounts Page (/dashboard/accounts)
- Split into: Short-Term Assets, Long-Term Assets, Short-Term Liabilities, Long-Term Liabilities
- Summary cards: Total Assets, Total Liabilities, Net Worth
- Click into any account to see its transactions (/dashboard/accounts/[id])
- Back navigation to accounts list
- Colour-coded by account type
- Responsive text overflow handling on summary cards
- Loading skeleton and empty state

### Transactions Page (/dashboard/transactions)
- Summary cards: Income, Expenses, Net for the latest month
- Clickable Income/Expenses cards with category breakdown panels
- Search by name/merchant
- Filter by category and account
- "Uncategorised" filter option (working correctly)
- Transactions grouped by month with headers
- Manual categorisation dropdown for uncategorised transactions
- AI Auto-categorise button with loading states and result feedback
- Loading skeleton and empty state

### AI Transaction Categorisation
- **Full security implementation:**
  - Server-side only (API key never exposed to client)
  - Auth verification required
  - User ownership check (can only categorise own transactions)
  - Rate limiting: 50 requests/user/hour
  - Batch limit: max 20 transactions per request
- Uses Claude Sonnet (`claude-sonnet-4-20250514`)
- Returns category_id and confidence score
- Shows detailed result message after categorisation
- AI badge displayed on auto-categorised transactions
- Graceful handling of billing errors with user-friendly messages

### Transaction Insights (/dashboard/transactions/insights)
- **4 clickable metric cards** that expand into detail panels:
  - Avg Daily Spend → spend by day of week with bar chart + insight text
  - Recurring Spend → tabs for recurring vs one-off merchants with totals
  - vs Last Month → side-by-side month comparison + category changes
  - Savings Rate → income/expenses/saved totals + monthly breakdown
- **Always-visible panels:** Top Spending Categories, Top Merchants, Unusual Transactions (>2.5x average)
- "Insights" text visible on mobile (not just icon)
- Responsive layout with overflow handling
- Loading skeleton and empty state

### Budget Page (/dashboard/budget)
- **Three-view architecture:** Choose → Wizard → Active Tracking
- **Choose View:** Smart Budget (auto-generated from spending patterns) and Fresh Budget (manual)
- **Wizard (3-step flow):**
  - Step 1: Set Amounts — category list with checkboxes, editable amounts, add/remove
  - Step 2: Review Coverage — budgeted (green), uncovered (amber), accounts (blue)
  - Step 3: Confirm — final review with buffer/tight indicators
- **Active Tracking:** Summary cards, status badges (over/approaching/on track), expandable transaction drill-down, month-over-month comparison, unbudgeted spending section
- "Set as Active" button with localStorage persistence
- Active budget indicator (badge)
- Edit pencil icon on budget names for editability hint
- Multiple budgets can coexist
- Smart Budget rounds up averages to nearest £10, filters trivial categories (<£5/month)

### Cash Flow Page (/dashboard/cashflow)
- 90-day forward projection engine based on 16 recurring transactions
- Handles monthly, weekly, biweekly, quarterly, annual frequencies
- **4 summary cards:** Current Balance, Projected Balance (with % change), Monthly Net Flow, Lowest Point (with warning colours)
- Balance Projection area chart with confidence band shading (widens over time)
- 30d/60d reference markers, zero line, custom tooltip with daily breakdown
- Time range selector: 30/60/90 days
- Upcoming Transactions panel (next 30 days) with urgency labels
- Recurring Breakdown: monthly income vs expenses, ratio bar, top 5 expenses
- Shows which budget is active ("Based on: [Budget Name]")
- Loading skeleton and empty state

### Scenarios Page (/dashboard/scenarios) — THE DIFFERENTIATOR
- Full scenario planning engine
- Baseline scenario auto-generated from recurring transactions
- **Natural Language AI Input:**
  - Users type plain English questions (e.g. "What if I go part-time?")
  - Claude Sonnet parses into structured scenario modifications via API route
  - Example query chips for quick access
  - Auto-generates scenario name, time horizon, and modifications
  - Rate limited: 20 requests/user/hour
- **Scenario Creator modal with:**
  - Name and time horizon (1-30 years slider)
  - Income changes (with quick templates: 5% Raise, Job Loss, etc.)
  - Expense changes (Cancel Netflix, Car Payment, School Fees)
  - One-time events (Buy Car, Home Deposit, Wedding)
  - Rate changes (investment returns, mortgage rates)
- Modifications list with edit/delete
- 10-year Net Worth projection chart
- Side-by-side scenario comparison (up to 3 scenarios)
- Active scenario selection with visual indicator
- Scenario persistence via localStorage
- Impact summaries comparing scenarios
- Checkboxes to select scenarios for comparison
- Different line styles (solid for active, dashed for comparison)

### Tax Planning Page (/dashboard/tax) — MOCK SCREEN
- **Mock screen for investor demos** (hardcoded realistic data)
- Summary cards: Estimated Income Tax, National Insurance, Capital Gains, Total Tax Liability
- Tax Year selector (visual only)
- Income Breakdown panel (Employment, Savings Interest, Dividends)
- Tax Bands visualisation with breakdown
- "Tax Saving Opportunities" section with actionable suggestions
- "Full tax planning with HMRC integration coming soon" banner
- Added to sidebar navigation with Receipt icon

### Settings Page (/dashboard/settings)
- Profile section with editable Full Name
- Save Changes with success feedback
- Account section showing email (read-only)
- "Send Password Reset Email" button
- Password reset flow with /auth/callback route
- Update password page (/update-password)
- Danger Zone with Sign Out button

### Feedback System
- Floating feedback bubble on all dashboard pages
- Default: small circle icon, expands to "Leave Feedback" pill on hover
- Three feedback types: Bug Report, Feature Request, General Feedback
- Submits to Formspree (https://formspree.io/f/mnjgqeod)
- Success animation with auto-close
- Dynamic placeholders per feedback type

### Polish & UX
- Page load fade-in animations on dashboard pages
- Forgot password link on login page
- Mobile improvements: Insights button shows text, responsive fixes
- Edit pencil icons as visual hint on editable elements
- Text overflow fixes on summary cards
- Loading skeletons (animate-pulse) on all dashboard routes
- Empty states with icons and CTAs on all pages
- Responsive/mobile layout: sidebar collapses, cards stack, charts scale

---

## 🐛 Known Issues

### Open Issues
1. **GitHub contributions not showing** — Private repo; enable "Private contributions" in GitHub profile settings to display activity

### Non-Blocking Warnings (Vercel build logs)
- `missingSuspenseWithCSRBailout` warning — Config key invalid in Next.js 16, harmless
- `"middleware" file convention is deprecated` — Should migrate to "proxy" convention eventually

---

## 📋 Backlog / Enhancements

| Priority | Item | Description | Status |
|----------|------|-------------|--------|
| **Medium** | Extended cash flow horizons | Add 180d/360d options + compare cash flows across scenarios | Backlog |
| **Medium** | Waitlist email verification | Add verification code step to confirm email ownership | Backlog |
| **Medium** | Crypto account mock data | Add "Coinbase" with BTC/ETH holdings, buy/sell transactions | Backlog |
| **Medium** | Plaid integration | Connect real bank accounts via production API | Pending |
| **Low** | Pre-built scenario templates | "What if I go part-time?", "What if we have a child?" etc. | Backlog |
| **Low** | Scenario duration selector | Support "X months/years" in addition to calendar date | Backlog |
| **Low** | Mobile app | Native mobile experience | Post-funding |

---

## 📂 Key File Locations

### Documentation
- Bug/Enhancement Log: `docs/BUG_LOG.md`
- Project State: `docs/PROJECT_STATE.md`

### Landing Page
- Main page: `src/app/page.tsx`

### Auth
- Auth pages: `src/app/(auth)/login/page.tsx`, `src/app/(auth)/signup/page.tsx`
- Auth actions: `src/app/(auth)/actions.ts`
- Loading button: `src/components/ui/loading-button.tsx`
- Password update: `src/app/(auth)/update-password/page.tsx`
- Auth callback: `src/app/auth/callback/route.ts`

### Dashboard
- Layout: `src/app/dashboard/layout.tsx`
- Sidebar: `src/components/layout/sidebar.tsx`
- Dashboard: `src/app/dashboard/page.tsx`

### Features
- Accounts: `src/app/dashboard/accounts/page.tsx`
- Account Detail: `src/app/dashboard/accounts/[id]/page.tsx`
- Transactions: `src/app/dashboard/transactions/page.tsx`
- Transaction Actions: `src/app/dashboard/transactions/actions.ts`
- Insights: `src/components/transactions/transaction-insights.tsx`
- Budget: `src/components/budget/budget-manager.tsx`
- Budget Utils: `src/lib/budget/get-active-budget.ts`
- Cash Flow: `src/components/cashflow/cash-flow-manager.tsx`
- Scenarios: `src/components/scenarios/scenarios-manager.tsx`
- Scenario Creator: `src/components/scenarios/scenario-creator.tsx`
- Scenario Engine: `src/lib/scenarios/projection-engine.ts`
- Scenario Types: `src/types/scenarios.ts`
- Scenario Storage: `src/lib/scenarios/storage.ts`
- Tax: `src/app/dashboard/tax/page.tsx`
- Settings: `src/components/settings/settings-manager.tsx`
- Settings Actions: `src/app/dashboard/settings/actions.ts`

### Natural Language Scenarios
- API Route: `src/app/api/scenarios/parse/route.ts`
- Input Component: `src/components/scenarios/natural-language-input.tsx`

### AI Categorisation
- Categorisation logic: `src/lib/ai/categorise-transactions.ts`
- Server action: `src/app/dashboard/transactions/actions.ts` (categoriseTransactionsWithAI)

### Feedback
- Feedback bubble: `src/components/feedback/feedback-bubble.tsx`

### Supabase
- Client: `src/lib/supabase/server.ts`
- Service client: `src/lib/supabase/service.ts`
- Middleware: `src/lib/supabase/middleware.ts`
- Seed SQL: `supabase/seed_demo_data.sql`

---

## 🏗️ Key Decisions & Architecture

| Decision | Rationale |
|----------|-----------|
| **Supabase (not separate backend)** | PostgreSQL + auth + auto-generated API + RLS in one package, zero backend to deploy |
| **Next.js App Router** | SSR for landing page, API routes for webhooks, single Vercel deploy target |
| **Server + Client Components** | Server for data fetching (keeps keys server-side), client for interactivity |
| **RLS on all tables** | `auth.uid() = user_id` pattern, automatic security without manual filters |
| **Anthropic API server-side only** | Security — API key never exposed to client |
| **Rate limiting on AI calls** | 50/user/hour for categorisation, 20/user/hour for scenario parsing |
| **localStorage for scenarios/budget** | Fast iteration without database migration; can add DB persistence later |
| **Positive = expense, Negative = income** | Convention used throughout transactions table and all components |
| **Categories are system-level** | Shared taxonomy (`user_id IS NULL`), user corrections tracked via `ai_category_id` |
| **GBP currency throughout** | Single currency for prototype, multi-currency in roadmap |
| **ignoreBuildErrors in next.config** | Prototype speed over type safety; fix properly post-funding |
| **<Suspense> wrapper pattern** | Required for `useSearchParams()` in Next.js production builds on Vercel |
| **Demo data auto-seeds on signup** | RPC function called in signup action so every new user sees populated dashboard |
| **Invite codes for access control** | Service role validates codes server-side; supports expiry, max uses, manual disable |
| **Custom domain via Vercel** | getoikome.com with auto-SSL, professional for investor demos |
| **Formspree for feedback** | Zero backend needed, free tier, submissions go to email |

---

## 📅 Sprint Progress

| Week | Deliverable | Status | Notes |
|------|-------------|--------|-------|
| **1** | Auth + DB schema + deploy | ✅ Complete | All foundation work done |
| **2** | Dashboard + balance sheet + net worth | ✅ Complete | Extended to 12+ months |
| **3** | AI categorisation + monthly P&L | ✅ Complete | Transaction insights + AI API integrated |
| **4** | Budgeting + cash flow forecast | ✅ Complete | Both features fully functional |
| **5-6** | Scenario planning engine | ✅ Complete | THE differentiator for demo |
| **6** | Tax planning mock + polish | ✅ Complete | Mock screen + UX polish |
| **7** | Landing page + access control + domain | ✅ Complete | getoikome.com live |
| **8** | Auth polish + feedback + screenshots + NL scenarios + docs | ✅ Complete | All demo polish done |

---

## 💡 Strategic Notes

### Demo Readiness
- ✅ Landing page live at getoikome.com
- ✅ Waitlist capturing leads via Tally
- ✅ Access controlled via invite codes
- ✅ Real screenshots on landing page
- ✅ Loading animations on auth pages
- ✅ Feedback collection live via Formspree
- ✅ Email removed from header for privacy
- ✅ Natural language scenario queries — AI demo moment
- ✅ Project docs in repo under docs/

### Investor Demo Flow (10 minutes)
1. Landing page (getoikome.com) → show value prop
2. Sign up with invite code → dashboard
3. Dashboard: "Here's my actual financial life, consolidated"
4. Transactions: "AI categorises automatically"
5. Insights: "Spending patterns and anomalies"
6. Budget: "Smart budget generation"
7. Cash Flow: "90-day forward projection"
8. Scenarios: "What happens under three different futures" (THE DEMO MOMENT)
9. AI Scenarios: "Type any what-if question and watch the future change"
10. Tax: "Where we're heading — full tax planning"

### Bootstrap vs Fundraise Decision
- Product is demo-ready
- Could get paying customers without funding
- Plaid free tier = 200 connections (enough for beta)
- Consider: Get 10-20 paying customers FIRST, then raise on better terms
- Key blocker: Personal runway situation

---

## 📝 Important Notes for Next Session

### Database
- Database trigger `on_auth_user_created` has been DROPPED — seeding happens in application code now. Do NOT re-create the trigger.
- The `is_asset` column on accounts is GENERATED — never include it in INSERT statements.
- Supabase queries should use `select('*')` and let RLS handle user filtering.
- The accounts table uses `current_balance` (not balance) and `institution_name` comes from FK join.
- Transaction convention: positive amounts = expenses, negative = income. GBP throughout.
- Profiles table has `full_name` column — set during signup and used for dashboard greeting + header display.
- Use `categories!category_id` for FK disambiguation (transactions have both `category_id` and `ai_category_id`).
- **invite_codes table** — validate codes via service role client, increment use_count on successful signup.

### API Keys
- Anthropic API: `ANTHROPIC_API_KEY` configured in both `.env.local` and Vercel env vars
- Supabase Service Role: `SUPABASE_SERVICE_ROLE_KEY` configured in both `.env.local` and Vercel env vars
- Credits: $10 added to Anthropic account

### Vercel Deployment
- Project name: `oikome` (lowercase required by Vercel)
- Connected to GitHub `raymondorae/Oikome`, auto-deploys on push to main
- Environment variables set: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `ANTHROPIC_API_KEY`, `SUPABASE_SERVICE_ROLE_KEY`
- Custom domain: `getoikome.com` (SSL auto-provisioned)
- Production URL: `getoikome.com`

### Supabase Auth
- Redirect URLs configured for:
  - `https://getoikome.com/**`
  - `https://www.getoikome.com/**`
  - `https://oikome-91f10lx4o-raymondoraes-projects.vercel.app/**`

### Next Session Priorities
1. Extended cash flow time horizons — 180d/360d + scenario comparison
2. Crypto account with mock data
3. Pitch deck and demo materials
