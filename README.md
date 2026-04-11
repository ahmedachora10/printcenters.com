# Product Requirements Document — Alnaasik Print Center (v1.0 Complete)

Full scope for a ground-up rebuild covering **32 modules**: core operations, POS, commissions, customers, B2B agents, loyalty, incentives, and full inventory management.

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Goals & Success Metrics](#2-goals--success-metrics)
3. [User Roles & Permissions](#3-user-roles--permissions)
4. [Module Specifications M01–M32](#4-module-specifications)
5. [Non-Functional Requirements](#5-non-functional-requirements)
6. [Out of Scope](#6-out-of-scope)
7. [Tech Stack](#7-tech-stack)
8. [Database Schema Reference](#8-database-schema-reference)
9. [Deliverables](#9-deliverables)
10. [Timeline & Pricing](#10-timeline--pricing)

---

## 1. Project Overview

**Product:** مركز النعائس للطباعة — Internal Management System
**Business:** Multi-branch print center, Saudi Arabia
**Language:** Arabic (RTL primary), English for admin labels
**Currency:** SAR — formatted `1,234.50 ر.س`
**VAT:** 15% default, configurable per branch

**What it does:** Web-based platform for print center staff to create service and product invoices, track commissions and incentives, manage customer relationships and loyalty, handle B2B agent rebates, track branch inventory and purchases, and view analytics. A public-facing price list page serves walk-in customers.

**Why rebuild:** Laravel 9 + Blade stack with 5+ years of technical debt — duplicate services per branch, two invoice types in one table, no customer entity, commission in three places, no inventory, no loyalty, no B2B agents.

---

## 2. Goals & Success Metrics

| Goal | Metric |
|------|--------|
| Single source of truth | Zero discrepancies between invoice totals, commission ledger, and sale records |
| Zero data loss | 100% of historical data migrated and verified |
| Fast POS | Invoice created in < 60 seconds |
| Eliminate service duplication | One master template catalogue; branch overrides only |
| Customer traceability | Every invoice linked to a customer or walk-in entry |
| Loyalty accuracy | Points and tier discounts applied correctly on every eligible invoice |
| Agent accountability | Rebate computed and stored per invoice automatically |
| Inventory accuracy | Stock computed from immutable movements; no manual edits to `current_stock` |
| RTL support | All UI components correct in RTL mode |
| RBAC | Every action protected by a named Spatie permission, server-side |

---

## 3. User Roles & Permissions

### 3.1 Roles

| Role | Scope | Description |
|------|-------|-------------|
| **SuperAdmin** | All branches | Full access everywhere |
| **BranchAdmin** | Own branch | Full access within branch |
| **Accountant** | Own branch | Product invoices, refunds, expenses, reports |
| **Employee** | Own branch | Service invoices, own commission, own incentive progress |
| **Agent** | Own data only | Read-only external portal (invoices + rebates) |

### 3.2 Permission Matrix

| Permission | SA | BA | Acc | Emp | Agent |
|------------|:--:|:--:|:---:|:---:|:-----:|
| Manage branches/cities | ✅ | ❌ | ❌ | ❌ | ❌ |
| Manage users | ✅ | ✅ | ❌ | ❌ | ❌ |
| Manage service templates (global) | ✅ | ❌ | ❌ | ❌ | ❌ |
| Manage branch services + tiers | ✅ | ✅ | ❌ | ❌ | ❌ |
| Manage product/expense categories | ✅ | ✅ | ✅ | ❌ | ❌ |
| Manage coupons | ✅ | ✅ | ❌ | ❌ | ❌ |
| Create service invoice | ✅ | ✅ | ❌ | ✅ | ❌ |
| Create product invoice | ✅ | ✅ | ✅ | ❌ | ❌ |
| Process refund | ✅ | ✅ | ✅ | ❌ | ❌ |
| Pay employee commission | ✅ | ✅ | ❌ | ❌ | ❌ |
| Manage incentive plans | ✅ | ✅ | ❌ | ❌ | ❌ |
| Pay bonuses | ✅ | ✅ | ❌ | ❌ | ❌ |
| View branch report | ✅ | ✅ | ✅ | ❌ | ❌ |
| View all-branches report | ✅ | ❌ | ❌ | ❌ | ❌ |
| View own commission/incentive | ✅ | ✅ | ❌ | ✅ | ❌ |
| Manage customers | ✅ | ✅ | ✅ | ✅ | ❌ |
| Manage agents + pay rebates | ✅ | ✅ | ❌ | ❌ | ❌ |
| View own agent data | ❌ | ❌ | ❌ | ❌ | ✅ |
| Configure loyalty program | ✅ | ✅ | ❌ | ❌ | ❌ |
| Redeem loyalty points at POS | ✅ | ✅ | ✅ | ✅ | ❌ |
| Manage inventory / suppliers / POs | ✅ | ✅ | ✅ | ❌ | ❌ |
| Run stock reconciliation | ✅ | ✅ | ❌ | ❌ | ❌ |
| Manage settings | ✅ | ✅ | ❌ | ❌ | ❌ |

> All permissions managed via **Spatie Laravel Permission**. Old `role` enum on `users` is retired.

---

## 4. Module Specifications

---

### M01 — Authentication

**Description:** Secure login, session management, and password control.

**User Stories:**
- As any user, I can log in with username and password and be redirected to my role-appropriate page.
- As any user, I can log out and change my own password.
- As a SuperAdmin, I can reset another user's password.

**Acceptance Criteria:**
- [ ] Login form: username + password only. No email login.
- [ ] Failed login: generic "بيانات الدخول غير صحيحة" (no username enumeration).
- [ ] Session expires after configurable idle timeout (default: 120 min).
- [ ] Post-login redirects: SuperAdmin → all-branches dashboard; Agent → `/agent-portal`; others → branch dashboard.
- [ ] Change password requires current password confirmation.
- [ ] Logout invalidates the server-side session immediately.

---

### M02 — Dashboard

**Description:** Role-scoped landing page with KPIs and charts.

**User Stories:**
- As BranchAdmin/Accountant, I see today's sales, returns, expenses, and net revenue.
- As Employee, I see my sales total, pending commission, and incentive progress.
- As SuperAdmin, I see an aggregated view across all branches.

**Acceptance Criteria:**
- [ ] KPI cards: service sales, product sales, returns, expenses, net revenue (today).
- [ ] Employee: pending commission card, "هدفك هذا الشهر" progress widget, payment notification badge.
- [ ] SuperAdmin: one row per active branch + totals row.
- [ ] All amounts SAR 2 decimal places. Date picker to view past day snapshots.
- [ ] Mini line chart: last 7 days trend (service vs product split).
- [ ] Low-stock alert banner for BranchAdmin (links to M29).

---

### M03 — Branch Management

**Description:** CRUD for physical branch locations.

**Acceptance Criteria:**
- [ ] Fields: Name, City FK, Phone, Address, Business Type, Commercial Reg No., Tax Number, Logo (upload), VAT Rate override %, Is Active.
- [ ] Soft delete. Cannot delete branch with associated invoices.
- [ ] Status toggle from list (no form open needed).
- [ ] Customer export to Excel: Name, Phone, Type, Tier, Invoice Count, Total Spend, Last Visit.

---

### M04 — City Management

**Acceptance Criteria:**
- [ ] Fields: Name Arabic, Name English, Is Active.
- [ ] Cannot delete city assigned to a branch. Status toggle from list.

---

### M05 — User Management

**Description:** Manage internal staff accounts (SuperAdmin, BranchAdmin, Accountant, Employee).

**User Stories:**
- As SuperAdmin, I can create users for any branch with any internal role.
- As BranchAdmin, I can create Accountant or Employee users for my branch.
- As manager, I can assign services to employees with per-service commission overrides.

**Acceptance Criteria:**
- [ ] Fields: Username, Password (create only), Full Name, Phone, Branch (SA only), Role, Salary, Base Commission %, Joined Date, CV upload, Is Active.
- [ ] Service assignment (Employee): checkbox list of branch services; per-service commission override % (nullable = use branch default).
- [ ] Soft delete. Cannot delete user with associated invoices.
- [ ] WhatsApp button: `https://wa.me/{phone}` (new tab).

---

### M06 — Settings

**Description:** Global and branch-level configuration, payment methods, loyalty config tab.

**Acceptance Criteria:**
- [ ] Global: default VAT %, app name.
- [ ] Branch: VAT override, logo, WhatsApp template message.
- [ ] Payment methods: CRUD. Cannot delete method referenced by invoices.
- [ ] VAT changes not retroactive.
- [ ] Settings tabs include: General | Payment Methods | Loyalty Program (M28) | Inventory Alerts.

---

### M07 — Service Templates (Master Catalogue)

**Description:** Global service types replacing per-branch duplicated `services` table.

**Acceptance Criteria:**
- [ ] Service Template: Name AR, Name EN, Description, Is Active.
- [ ] Branch Service: Template FK, Branch FK, Base Commission %, Max Discount %, Is Tahazir, Is Active.
- [ ] Unique constraint: (branch_id, service_template_id).
- [ ] Deactivating global template does NOT auto-deactivate branch services.
- [ ] "Manage Tiers" button per branch service (M15 tiered commission config).

---

### M08 — Product Categories

**Acceptance Criteria:**
- [ ] Fields: Name, Branch FK, Is Active.
- [ ] Cannot delete if referenced by invoice lines or inventory products.

---

### M09 — Expense Categories

**Acceptance Criteria:**
- [ ] Fields: Name, Branch FK, Is Active.
- [ ] Cannot delete if referenced by expense records.

---

### M10 — Coupon Management

**User Stories:**
- As BranchAdmin, I can create, edit, deactivate, and delete unused coupons.
- As POS user, I can validate and apply a coupon to an invoice.

**Acceptance Criteria:**
- [ ] Fields: Code (unique per branch, case-insensitive), Discount Type (percentage/fixed), Discount Value, Capacity (nullable = unlimited), Expires At (nullable), Is Active.
- [ ] Validation: `GET /coupons/validate?code=X` → `{valid, type, value, remaining_capacity}`.
- [ ] On invoice save: `coupon_id` FK stored; `used_count` incremented atomically in DB transaction.
- [ ] Cannot delete coupon applied to any invoice.

---

### M11 — Accountant POS (Product Invoice)

**Description:** Invoice creation for accountants selling physical products.

**User Stories:**
- As Accountant/BranchAdmin, I can create product invoices with dynamic line items, apply coupon, loyalty points redemption, and agent link.
- As Accountant, I can save as Paid or Due (subject to customer credit limit).

**Acceptance Criteria:**
- [ ] Line items: Product (searchable by SKU/name from M29, or free-text), Qty, Unit Price (auto-filled from product selling price), Discount %.
- [ ] Totals: Subtotal → Tier Discount → Coupon Discount → Points Redemption → VAT → Total.
- [ ] Customer: searchable combo-box (M22). Walk-in: stores name + phone as text, no customer record.
- [ ] Agent selector (optional): auto-applies discount or stores rebate link per M26 mode.
- [ ] Loyalty: tier discount auto-applied; points redemption toggle (M28).
- [ ] Due invoice: credit limit check. Status Paid stores `paid_at`.
- [ ] Invoice number: `INV-{branch_code}-{seq}`.
- [ ] On save with SKU lines: `stock_movements` type=`sale_out` inserted in same DB transaction (M29).
- [ ] Soft delete only. List filterable by date, status, customer, agent.

---

### M12 — Employee POS (Service Invoice)

**Description:** Invoice creation for employees performing print-center services.

**User Stories:**
- As Employee, I can create service invoices from my assigned services with real-time commission preview.
- As BranchAdmin, I can change a line's discount after the invoice is saved.

**Acceptance Criteria:**
- [ ] Service selector: employee's assigned services only, searchable. Shows commission % badge.
- [ ] Line item: Service, Qty, Unit Price, Discount % (max = `max_discount_pct`, client + server validated). Real-time commission preview per line. Tahazir lines show violet badge.
- [ ] Tiered commission (M15): shows active tier label in commission preview.
- [ ] Totals: same structure as M11.
- [ ] Agent selector: same as M11.
- [ ] Loyalty: same as M11 for individual customers.
- [ ] Employee saves as Paid only. BranchAdmin can save as Due (with credit check).
- [ ] On save: `service_invoice_lines` + `commission_ledger` entries (with tier_applied, is_tahazir) in one transaction. Loyalty points earned if individual customer + Paid.
- [ ] "Change discount" post-save: BranchAdmin only; recalculates commission ledger in same transaction.
- [ ] Invoice number: `SINV-{branch_code}-{seq}`.
- [ ] BranchAdmin sees all employees' invoices; Employee sees own only.

---

### M13 — Invoice Viewing & Printing

**Acceptance Criteria:**
- [ ] Detail page: branch header, invoice no., date, customer (+ company name if corporate), agent name, line items, totals breakdown (subtotal, tier discount, coupon, points redemption, VAT, total), payment method, status badge.
- [ ] A4 print: logo, tax no., VAT breakdown, ZATCA QR placeholder (80×80), footer.
- [ ] Thermal print (80mm): condensed, no logo, essential fields, `window.print()`.
- [ ] Attachment: BranchAdmin uploads PDF/image; stored in private storage, signed URL.
- [ ] Points redemption shown on printout: "استبدال نقاط الولاء: - X ر.س".

---

### M14 — Refunds / Returns

**Acceptance Criteria:**
- [ ] Fields: Amount, Source Type (Service/Product Invoice), Invoice FK (optional), Reason, Refunded By (auto), Branch (auto).
- [ ] If linked to product invoice with SKU lines: prompt "هل تريد إعادة المخزون؟" → inserts `stock_movements` type=`return_in`.
- [ ] Refunds appear as negative deductions in sales report.
- [ ] List filterable by date, type, invoice. Soft delete.

---

### M15 — Commission System

**Description:** Track earnings per invoice line, support tiered rates, manage payments, and provide a unified commission dashboard.

**User Stories:**
- As BranchAdmin, I can define tiered commission rates per branch service and pay employee commissions.
- As Employee, I can see my pending commission and payment history.

**Acceptance Criteria:**

#### Tiered Rates (extends M07)
- [ ] Per (branch_service, user) pair: up to 3 tiers — threshold_amount + commission_pct. Flat rate used when no tiers defined.
- [ ] `commission_ledger.tier_applied` (1/2/3/NULL) tagged at invoice time.
- [ ] Commission report shows current-month tier status per employee per service.

#### Ledger & Payments
- [ ] One ledger entry per service invoice line: user, branch, invoice_line, amount, is_tahazir, tier_applied, earned_at, paid_at (nullable).
- [ ] Summary: grouped employee → date. Columns: date, total_sale, net_amount, commission, PAID/UNPAID.
- [ ] "Pay now" (date, employee): creates immutable `commission_payments` record; sets `paid_at` on all matching ledger entries in one transaction.
- [ ] Reversal: new negative adjustment record with mandatory reason.
- [ ] Employee self-view: unpaid total + last 30 days earned.

#### Unified Dashboard (`/commissions/overview`)
- [ ] Three tabs: عمولات الموظفين | عمولات التحاضير | عمولات المندوبين (links to M26 agent reports).
- [ ] Summary cards: إجمالي المستحقة هذا الشهر, إجمالي المدفوع, المتبقي.

#### Real-Time POS Preview
- [ ] In M12: "عمولتك المتوقعة" updates in real time as lines are added/modified.
- [ ] Shows active tier label when tiered rates are configured.

---

### M16 — Expense Purchases

**Description:** Non-inventory branch expenses (utilities, rent, marketing). Products tracked in inventory use M29 Purchase Orders instead.

**Acceptance Criteria:**
- [ ] Fields: Expense Category FK, Qty, Unit Price, Total (computed), Supplier Name (free text, optional), Receipt Reference (text or file upload), Comment, Branch (auto), Purchased By (auto), Date.
- [ ] Soft delete. List filterable by date range and category. Totals row.

---

### M17 — Sales Report

**Acceptance Criteria:**
- [ ] Columns per day: Date, Service Sales, Product Sales, Total Sales, Employee Commission, Agent Rebates, Loyalty Points Value Redeemed, Service Returns, Product Returns, Expenses, Net Revenue.
- [ ] Totals row. Date range picker (default: current month). Excel export.
- [ ] SuperAdmin: additional Branch column with sub-totals.

---

### M18 — Employee Commission Report

**Acceptance Criteria:**
- [ ] Grouped by employee → date. Columns: Employee, Date, Total Sales, Net Amount, Commission, Tier Applied, Status.
- [ ] Drill-down to individual ledger entries (invoice no., service, amount, is_tahazir, tier).
- [ ] Date range + employee filters.

---

### M19 — Public Service Catalogue

**Description:** Public-facing read-only price list (no auth required).

**Acceptance Criteria:**
- [ ] Three-level nav: Category tabs → Sub-category tabs (AJAX) → Service price list.
- [ ] Price row: Name, price range (min–max SAR). Base price on hover via popover.
- [ ] Inactive items hidden. Arabic RTL. SEO-friendly (SSR initial load).
- [ ] Fixed header (logo, "قائمة الأسعار", phone). Floating WhatsApp button (bottom-left).
- [ ] Real-time search filters visible prices. Mobile-responsive (375px min).

---

### M20 — Catalogue CRUD (Admin)

**Acceptance Criteria:**
- [ ] Category CRUD: Name, Image, Sort (drag-and-drop), Is Active.
- [ ] Sub-category CRUD: Name, Category FK, Image, Sort, Is Active.
- [ ] Service Price CRUD: Name, Sub-cat FK, Min/Max/Base Price, Sort, Is Active. Unique: (sub_category_id, name).
- [ ] Import Excel (upsert on sub_category + name). Export Excel. Status toggle from list.

---

### M21 — Notifications

**Acceptance Criteria:**
- [ ] Bell icon: unread badge. Dropdown: 10 most recent, "Mark all as read" button.
- [ ] Click navigates to relevant resource.
- [ ] Notification types:
  - Employee commission paid: "تم دفع عمولتك ليوم [date] — [amount] ر.س"
  - Employee bonus paid: "تم صرف مكافأتك لشهر [month] — [amount] ر.س"
  - Low stock: "وصل مخزون [product] للحد الأدنى" (BranchAdmin)
  - Tier upgrade: "عميلك [name] أصبح [Silver/Gold]" (BranchAdmin)
- [ ] Polling: 60 seconds (or Laravel Echo).

---

### M22 — Customer System (نظام العملاء)

**Description:** Complete customer management — contact, credit, loyalty tier, corporate accounts, financial history, CRM tools, and segmentation.

**User Stories:**
- As POS user, I can search, select, or create a customer inline during invoice creation.
- As BranchAdmin, I can enforce credit limits, segment customers, export lists, and merge duplicates.
- As BranchAdmin, I can view a customer's full financial summary and loyalty panel.

**Acceptance Criteria:**

#### Customer Fields
- [ ] Full Name, Phone (unique per branch), Email (optional), Customer Type (فردي/مؤسسة), Company Name (if Corporate), Credit Limit (decimal SAR, NULL = cash only), Agent Link (agent_id FK, nullable), Notes, Is Active, Branch FK.
- [ ] Computed fields (not editable): Loyalty Tier, Points Balance, Cumulative Spend.

#### Credit Enforcement
- [ ] Due invoice check: `(existing unpaid Due total) + (new invoice total) ≤ credit_limit`. Exceeded → block + "تجاوز الحد الائتماني — الرصيد المتاح: X ر.س".

#### Customer Profile
- [ ] Financial panel: invoice count, total billed, total paid, outstanding, credit limit, available credit, progress bar.
- [ ] Loyalty panel (M28): tier badge, points, cumulative spend, points to next tier, points history table.
- [ ] Invoice history (service + product). Notes textarea. WhatsApp button.

#### Customer List
- [ ] Columns: Name, Phone, Type, Tier, Agent, Invoice Count, Total Spend, Days Since Last Visit (green <30 / amber 30–90 / red >90), Outstanding Balance.
- [ ] Filters: tier, type, agent, outstanding > 0, last visit range, min spend, search by name/phone.
- [ ] Export to Excel. Sortable.

#### Outstanding Balance Report
- [ ] Table: Customer, Phone, Company, Total Due, Oldest Due Date, Days Outstanding. Export.

#### Merge & Migration
- [ ] Merge: primary + secondary → re-link all secondary invoices → soft-delete secondary.
- [ ] Data migration: existing invoice name/phone combos de-duplicated into customer records.

---

### M23 — CRM & Customer Analytics

**Acceptance Criteria:**
- [ ] Customer activity log (read-only): auto-entries for invoice, payment, refund, tier change, points earned/redeemed.
- [ ] WhatsApp bulk action: filtered selection → generate WA links per customer.
- [ ] Customer stats in analytics (M25 loyalty tab): tier distribution donut, points earned vs redeemed chart.

---

### M24 — Commission Payment Ledger

**Description:** Immutable audit trail of all commission payments.

**Acceptance Criteria:**
- [ ] Payment record: Employee, Branch, Period Start, Period End, Total Amount, Paid By, Paid At, Notes.
- [ ] Detail: expandable list of covered `commission_ledger` entries (invoice no., service, amount, date, tier, is_tahazir).
- [ ] Immutable — no edit or delete after creation. Reversals via negative adjustment record with reason.
- [ ] Employee self-view: own payment records (date, amount, covered period).

---

### M25 — Advanced Analytics Dashboard

**Acceptance Criteria:**
- [ ] Charts (Recharts): daily revenue line (service vs product), top 10 services bar, employee performance bar, sales breakdown donut.
- [ ] SuperAdmin: branch comparison bar chart.
- [ ] Date range picker applies to all charts simultaneously.
- [ ] Loyalty analytics tab: tier distribution donut, monthly points earned vs redeemed line chart.
- [ ] Export each chart to PNG.

---

### M26 — Agent System (نظام المندوب)

**Description:** B2B intermediary management. Agents bring bulk orders and receive automatic discounts or monthly rebate commissions.

**User Stories:**
- As BranchAdmin, I can create agent accounts with discount or rebate terms.
- As POS user, I can link any invoice to an agent.
- As BranchAdmin, I can view agent invoices and mark rebates as paid.
- As Agent, I can log into a read-only portal to see my invoices and earned rebates.

**Acceptance Criteria:**

#### Agent Entity
- [ ] Fields: Name, Type (فرد/شركة), Phone, Email, Commercial Reg No. (optional), Discount Mode (discount/rebate), Rate % (decimal), Branch FK, Is Active, Notes.
- [ ] Agent user account: separate login, Role = `Agent` (Spatie).

#### Invoice Integration
- [ ] Agent selector (optional) on both M11 and M12 POS.
- [ ] Mode `discount`: invoice-level discount pre-filled with agent rate %; read-only (BranchAdmin can override).
- [ ] Mode `rebate`: no auto-discount; `agent_id` stored; rebate = `total_amount × rate/100` stored on invoice.

#### Rebate Report & Payment
- [ ] `/agents/:id/report`: filterable by date. Columns: invoice no., date, customer, total, rate %, rebate amount, status. Totals row. Export Excel.
- [ ] "دفع العمولة": creates immutable `agent_payments` record.

#### Agent Portal (`/agent-portal`)
- [ ] Isolated route group, Agent role only.
- [ ] Dashboard: إجمالي فواتيري, إجمالي عمولتي المستحقة.
- [ ] My Invoices table (read-only). My Rebates table (date, amount, paid/unpaid).
- [ ] Agent cannot create, edit, or delete any data.

---

### M27 — Incentives & Rewards System (نظام الحوافز والمكافئات)

**Description:** Monthly target-based bonus system on top of regular commissions.

**User Stories:**
- As BranchAdmin, I can set monthly targets and bonus rules per employee, approve achievements, and pay bonuses.
- As Employee, I can track my progress on the dashboard and receive a notification when bonus is paid.

**Acceptance Criteria:**

#### Incentive Plan
- [ ] Fields: Employee FK, Branch FK, Period (month + year), Sales Target (SAR), Bonus Type (fixed/percentage of sales above target), Bonus Value, Status (Active/Achieved/Missed/Paid), Notes.
- [ ] Unique constraint: (employee_id, period_month, period_year).

#### Calculation & Approval
- [ ] Scheduled job (end of month) or on-demand "احتساب الحوافز" button.
- [ ] Compares target vs `SUM(service_invoices.total_amount)` for employee in period.
- [ ] Achieved → bonus computed + stored; queued in BranchAdmin approval list. Missed → status updated.

#### Payment
- [ ] "صرف المكافأة": confirmation dialog → on confirm: status Paid; `bonus_payments` record created; employee notification (M21) fired.
- [ ] Bonus payments shown separately from commission payments in employee history.

#### Employee Dashboard Widget
- [ ] "هدفك هذا الشهر": Target, Current Sales, Progress bar, Expected bonus (if on track).
- [ ] Hidden if no active plan for current month.

#### Leaderboard (`/incentives/leaderboard`)
- [ ] Columns: Rank, Employee, Total Sales, Target, Achievement %, Status.
- [ ] Top 3 rows: gold/silver/bronze background. Filter by month.

---

### M28 — Loyalty System (نظام الولاء)

**Description:** Dual loyalty program — redeemable points + automatic tier discounts for individual customers.

**User Stories:**
- As BranchAdmin, I can configure points earning/redemption rates, tier thresholds, and tier discount percentages.
- As POS user, I can redeem a customer's points as a discount during invoice creation.
- As BranchAdmin, I can view loyalty status per customer and run a loyalty report.

**Acceptance Criteria:**

#### Configuration (`/settings/loyalty`)
- [ ] Points earning: X pts per SAR (default: 1 pt/SAR). Configurable per branch.
- [ ] Redemption: Y pts = 1 SAR (default: 100 pts = 1 SAR). Minimum redemption threshold (default: 500 pts).
- [ ] Tier table (all values configurable):

| Tier | Arabic | Min Cumulative Spend | Auto Discount % |
|------|--------|---------------------|----------------|
| None | بدون | 0 SAR | 0% |
| Bronze | برونزي | 500 SAR | 2% |
| Silver | فضي | 2,000 SAR | 5% |
| Gold | ذهبي | 5,000 SAR | 8% |

- [ ] Program toggled active/inactive per branch.

#### Points Earning
- [ ] On paid invoice for individual customer: `earned_points = FLOOR(total_amount × earning_rate)`.
- [ ] Insert `loyalty_transactions`: type=earn, points, balance_after.
- [ ] Points NOT earned on Due invoices — credited when Due invoice is marked Paid.
- [ ] Points NOT earned on discount portions — only on amount actually paid.

#### Tier Management
- [ ] Tier = computed from `SUM(all paid invoice totals)` cumulative all-time; never resets.
- [ ] Recalculated after each payment. Tiers never downgrade (manual override by BranchAdmin only).
- [ ] Tier discount auto-applied at POS when customer selected; labeled "خصم مستوى الولاء: X%"; only BranchAdmin can override.
- [ ] BranchAdmin notification (M21) on tier upgrade.

#### Points Redemption at POS (M11 & M12)
- [ ] "استبدال النقاط" toggle shown when customer has redeemable points.
- [ ] Shows: "رصيد النقاط: X نقطة (يعادل Y ر.س)".
- [ ] Slider/input for points amount; real-time SAR equivalent. Cannot exceed invoice total.
- [ ] On invoice save: `loyalty_transactions` record type=redeem (negative). Shown on printout.
- [ ] Applies to individual customers only (not Corporate or Agent-linked).

#### Loyalty Report (`/reports/loyalty`)
- [ ] Table: Customer, Tier, Current Points, Total Earned, Total Redeemed, Cumulative Spend. Filter by tier. Export Excel.
- [ ] Summary cards: customer count per tier.

---

### M29 — Inventory & Warehouse System (نظام المشتريات والمخازن)

**Description:** Full SKU-level inventory — product catalogue, purchase orders, immutable stock movements, reconciliation, and reports.

**User Stories:**
- As BranchAdmin, I can define products with SKU, cost/selling price, and minimum stock level.
- As BranchAdmin/Accountant, I can create purchase orders and receive stock against them.
- As the system, I deduct inventory automatically when a product is sold via M11.
- As BranchAdmin, I am alerted when stock hits minimum level and can run reconciliation.

**Acceptance Criteria:**

#### Products (`/inventory/products`)
- [ ] Fields: SKU (unique per branch, auto or manual), Name AR, Name EN (optional), Product Category FK (M08), Unit FK (product_units lookup), Cost Price, Selling Price, Min Stock Level, Current Stock (computed — read-only), Barcode (optional), Is Active.
- [ ] List: SKU, Name, Category, Current Stock (red ≤ min level), Min, Cost, Selling Price, Valuation.
- [ ] Low-stock banner: "⚠️ X منتجات وصلت للحد الأدنى".

#### Suppliers (`/inventory/suppliers`)
- [ ] CRUD: Name, Phone, Email, Notes, Branch FK.

#### Purchase Orders (`/inventory/purchase-orders`)
- [ ] PO fields: PO Number (auto: PO-{branch}-{seq}), Supplier FK (optional), Branch FK, Created By, Order Date, Expected Delivery (optional), Status (مسودة/مُرسل/مُستلم جزئياً/مُستلم كلياً/ملغي), Notes.
- [ ] PO lines: Product FK, Ordered Qty, Received Qty, Unit Cost, Subtotal.
- [ ] "استلام البضاعة": per-line received qty input; on save: `stock_movements` type=`purchase_in` inserted in same DB transaction; PO status auto-updated.
- [ ] List filterable by status, supplier, date. Export Excel.

#### Stock Movements (`/inventory/movements`)

| Type | Arabic | Trigger |
|------|--------|---------|
| `purchase_in` | استلام مشتريات | PO receipt |
| `sale_out` | مبيعات | M11 product invoice save |
| `return_in` | مرتجع وارد | M14 refund with stock reversal |
| `adjustment_in` | تعديل موجب | Reconciliation increase |
| `adjustment_out` | تعديل سالب | Reconciliation decrease |
| `opening_stock` | رصيد افتتاحي | Initial stock entry |

- [ ] Movement fields: product_id, branch_id, type, qty, unit_cost, reference_id, reference_type, notes, created_by, created_at. Immutable — no UPDATE or DELETE.
- [ ] History page: filterable by product, type, date. Columns: date, type, reference (linked), qty, unit cost, balance after.

#### M11 POS Integration
- [ ] SKU product search → auto-fills selling price.
- [ ] On invoice save: `stock_movements` type=`sale_out` per SKU line in same DB transaction.
- [ ] Negative-stock warning dialog: "الكمية المتاحة: X — هل تريد الاستمرار؟" (block vs warn-only configurable in settings M06).

#### Stock Reconciliation (`/inventory/reconciliation`)
- [ ] Initiate: select products (or all). Generates sheet: SKU, Name, System Qty, Physical Count (input), Variance.
- [ ] On submit: `stock_movements` adjustment inserted per variance ≠ 0 with mandatory reason note.
- [ ] Reconciliation history: date, initiated by, items adjusted, total variance.

#### Inventory Reports
- [ ] **Valuation** (`/reports/inventory/valuation`): SKU, Name, Category, Qty, Cost, Selling, Value at Cost, Value at Selling, Margin. Grand totals. Export Excel.
- [ ] **Movement Report** (`/reports/inventory/movements`): Product, Opening Stock, Purchased, Sold, Returned, Adjusted, Closing Stock. Per product per period. Export Excel.
- [ ] **Purchase Cost Report** (`/reports/inventory/purchases`): sourced from POs. Date range, supplier, category filters. Export Excel.

---

## 5. Non-Functional Requirements

### 5.1 Performance
- Dashboard and POS pages load < 2 seconds.
- Invoice list pages with date filters < 1 second for up to 1 year of data.
- Excel exports ≤ 12 months: generated within 10 seconds; queued job + download link if > 5 seconds.
- Stock movements computed from indexed `stock_movements` — no full-table scans.

### 5.2 Security
- All routes except public catalogue and agent portal login require authentication.
- All actions: Spatie permission checked server-side.
- No raw SQL interpolation — Eloquent or parameterised query builder only.
- Files stored in private storage — never in `public/`.
- CSRF on all POST/PUT/DELETE.
- Amounts stored as `DECIMAL(12,2)` — no float.
- Agent portal isolated — Agents cannot access any internal management route.

### 5.3 Localisation
- All UI text in `lang/ar/*.php` + `lang/en/*.php`.
- Default locale: Arabic (RTL). `dir="rtl"` on `<html>`.
- Dates: display `DD/MM/YYYY`, store `YYYY-MM-DD`.
- Amounts: `1,234.50 ر.س`.

### 5.4 Browser Support
- Chrome 110+, Firefox 110+, Safari 16+, Edge 110+.
- Mobile-responsive (375px min): public catalogue, employee POS, agent portal.
- Admin: desktop-first (1024px min).

### 5.5 Reliability & Transactions
- All money-affecting operations in a DB transaction (invoice save, commission ledger write, points earn/redeem, coupon counter, stock movements, payment records). Failure at any step → full rollback.
- Soft deletes on all primary entities.
- Model events logged to `activity_log` (Spatie Activity Log).
- `stock_movements`, `commission_ledger`, `loyalty_transactions` are immutable.

### 5.6 Data Integrity
- All foreign keys enforced at DB level.
- `DECIMAL` for all monetary/percentage columns.
- `ENUM` or FK-to-lookup for status columns.
- Critical unique constraints: `coupons.(code, branch_id)` | `catalog_prices.(subcategory_id, name)` | `branch_services.(branch_id, service_template_id)` | `products.(sku, branch_id)` | `incentive_plans.(user_id, period_month, period_year)` | `agents.(phone, branch_id)`.

---

## 6. Out of Scope

| Item | Notes |
|------|-------|
| Multi-currency | SAR only |
| Online payment gateway | Manual payment recording only |
| ZATCA e-invoicing API | QR placeholder only; API submission is a separate engagement |
| Mobile native app | Web-responsive only |
| Full HR/payroll | Commission + bonuses only; no salary slips, leave, or deductions |
| Multi-language public catalogue | Arabic only |
| Email / SMS notifications | In-app + WhatsApp links only |
| POS hardware integration | No printer API, barcode scanner driver, or cash drawer |
| Barcode scanning | Field stored for future use; hardware not integrated |
| Multi-warehouse | One inventory store per branch; no inter-branch stock transfers |
| FIFO / costing methods | Last purchase cost per movement; no costing method selection |

---

## 7. Tech Stack

| Layer | Technology |
|-------|-----------|
| **Framework** | Laravel 13 (PHP 8.3) |
| **Frontend bridge** | Inertia.js (React starter kit) |
| **UI components** | shadcn/ui (Radix UI primitives) |
| **Styling** | Tailwind CSS 4.x (RTL configured) |
| **Auth** | Laravel Breeze (Inertia/React variant) |
| **Permissions** | Spatie Laravel Permission |
| **Activity log** | Spatie Laravel Activity Log |
| **Excel** | Maatwebsite Laravel Excel |
| **Charts** | Recharts (React) |
| **Database** | MySQL 8.0+ |
| **Queue** | Laravel Queue (database driver, Redis-upgradeable) |
| **File storage** | Laravel Storage (local or S3-compatible) |
| **Testing** | Pest PHP (feature + unit) |
| **CI** | GitHub Actions |

---

## 8. Database Schema Reference

### Core Tables
```
branches, cities, users (+ Spatie roles/permissions), settings, payment_methods
```

### Service Catalogue
```
service_templates, branch_services [UNIQUE(branch_id, service_template_id)],
commission_tiers (branch_service_id, user_id, tier_number, threshold_amount, commission_pct),
user_services (user_id, branch_service_id, commission_override_pct)
```

### Categories & Lookups
```
product_categories, expense_categories, product_units,
catalog_categories, catalog_subcategories, catalog_prices [UNIQUE(subcategory_id, name)]
```

### Coupons
```
coupons [UNIQUE(code, branch_id)] (discount_type ENUM, discount_value, capacity, used_count, expires_at)
```

### Customers
```
customers [UNIQUE(phone, branch_id)] (customer_type ENUM('individual','corporate'), company_name,
  credit_limit, agent_id FK, points_balance INT, cumulative_spend DECIMAL, tier ENUM('none','bronze','silver','gold'))
```

### Agents
```
agents [UNIQUE(phone, branch_id)] (agent_type ENUM, discount_mode ENUM('discount','rebate'), rate),
agent_users (agent_id, user_id),
agent_payments (agent_id, branch_id, period_start, period_end, total_invoices, total_rebate, paid_by, paid_at)
```

### Invoices (Class Table Inheritance)
```
product_invoices (branch_id, user_id, customer_id, agent_id, coupon_id, invoice_number,
  subtotal, tier_discount_pct, tier_discount_amount, coupon_discount, points_redeemed, points_discount,
  vat_pct, vat_amount, total_amount, payment_method_id, status ENUM('paid','due','cancelled'), paid_at, attachment_path)
product_invoice_lines (invoice_id, product_id NULLABLE, product_name, qty, unit_price, discount_pct, subtotal)

service_invoices (same header columns as product_invoices + employee_commission DECIMAL)
service_invoice_lines (invoice_id, branch_service_id, service_name, qty, unit_price, discount_pct, subtotal, commission_pct, commission_amount, tier_applied)
```

### Commission & Payments
```
commission_ledger (user_id, branch_id, invoice_line_id, amount, is_tahazir, tier_applied, earned_at, paid_at) -- IMMUTABLE
commission_payments (user_id, branch_id, period_start, period_end, total_amount, paid_by, paid_at, notes) -- IMMUTABLE
```

### Refunds
```
refunds (branch_id, user_id, source_type ENUM('service','product'), invoice_id NULLABLE, amount, reason, stock_reversed BOOL)
```

### Incentives
```
incentive_plans [UNIQUE(user_id, period_month, period_year)]
  (user_id, branch_id, period_month, period_year, target_amount, bonus_type ENUM('fixed','percentage'),
   bonus_value, achieved_amount, status ENUM('active','achieved','missed','paid'), notes)
bonus_payments (incentive_plan_id, paid_by, amount, paid_at, notes)
```

### Loyalty
```
loyalty_config (branch_id, earning_rate, redemption_rate, min_redemption_points,
  bronze_threshold, silver_threshold, gold_threshold,
  bronze_discount_pct, silver_discount_pct, gold_discount_pct, is_active)
loyalty_transactions (customer_id, invoice_id NULLABLE, type ENUM('earn','redeem','manual_adjust','expire'),
  points, balance_after, notes, created_at) -- IMMUTABLE
```

### Inventory
```
products [UNIQUE(sku, branch_id)] (branch_id, sku, name_ar, name_en, category_id, unit_id,
  cost_price, selling_price, min_stock_level, current_stock, barcode, is_active)
product_units (id, name)
suppliers (branch_id, name, phone, email, notes)
purchase_orders (po_number, branch_id, supplier_id, ordered_by, order_date, expected_delivery,
  status ENUM('draft','sent','partial','received','cancelled'), notes)
purchase_order_lines (po_id, product_id, ordered_qty, received_qty, unit_cost, subtotal)
stock_movements (product_id, branch_id, type ENUM('purchase_in','sale_out','return_in',
  'adjustment_in','adjustment_out','opening_stock'), qty, unit_cost,
  reference_id, reference_type, notes, created_by, created_at) -- IMMUTABLE
stock_reconciliations (branch_id, initiated_by, completed_at, notes)
stock_reconciliation_lines (reconciliation_id, product_id, system_qty, physical_qty, variance, movement_id FK)
```

---

## 9. Deliverables

| # | Deliverable | Description |
|---|-------------|-------------|
| 1 | **Source code** | Full Laravel 13 + Inertia/React app in private Git repository |
| 2 | **Database migrations** | All 32 modules' migrations + seeders (lookup tables, roles, permissions) |
| 3 | **Data migration scripts** | Artisan commands: old DB → new schema, with dry-run mode |
| 4 | **Test suite** | Pest feature tests for all critical POS, commission, loyalty, and inventory flows |
| 5 | **Deployment guide** | Step-by-step `.md` for VPS/shared hosting deployment |
| 6 | **Environment config** | `.env.example` with all required variables documented |
| 7 | **Agent portal** | Isolated route group + read-only views for B2B agents |
| 8 | **User manual** | Arabic PDF per role (optional add-on) |

---
