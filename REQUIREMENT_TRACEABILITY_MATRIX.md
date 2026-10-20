# FinWise AI — Requirement Traceability Matrix (RTM)

**Document ID:** DOC-RTM-010  
**Version:** 1.0.0  
**Authority:** Technical Documentation Engineer & QA Lead  
**Scope:** 100% Traceability of All PRD Requirements (Core, Plus, Stretch, Security, NFR)  
**Last Updated:** 20-Oct-2026  

---

## 1. Traceability Methodology

Every significant requirement follows an unbroken end-to-end engineering verification chain:
$$\text{Source Requirement} \longrightarrow \text{Task ID} \longrightarrow \text{Module/File} \longrightarrow \text{Owner} \longrightarrow \text{Test Specification} \longrightarrow \text{Acceptance Criteria} \longrightarrow \text{Integration Result}$$

- **Core Tier (76 Requirements):** Mandatory for capstone completion and viva passing.
- **Plus Tier (7 Requirements):** High-value differentiation tier (charts, formula export guards).
- **Stretch Tier (12 Requirements):** Post-core investment portfolio simulation.

---

## 2. Master Traceability Matrix

### 2.1 Authentication & User Identity (AUTH)

| Req ID | PRD Source | Task ID | Owner | Target Implementation File | Verification Test | Acceptance Criteria | Integration Verification |
|---|---|---|---|---|---|---|---|
| **AUTH-001** | PRD §4.1 | T3-V1 | VIVEK | `core/auth/services/register.py` | `tests/test_auth.py::test_user_registration` | Unique email required; returns 201; user saved to DB | J-1 Onboarding flow |
| **AUTH-002** | PRD §4.1 | T3-V1 | VIVEK | `core/auth/services/password.py` | `tests/test_auth.py::test_password_strength` | Rejects passwords <8 chars or lacking upper/lower/num/sym | Form error UI |
| **AUTH-003** | PRD §4.1 | T3-V1 | VIVEK | `core/auth/services/login.py` | `tests/test_auth.py::test_argon2id_login` | Validates hash via Argon2id; returns Bearer JWT | UI login flow |
| **AUTH-004** | PRD §4.1 | T3-V1 | VIVEK | `core/auth/dependencies.py` | `tests/test_auth.py::test_jwt_validation` | Injects AuthenticatedUser; rejects expired tokens | Protected routes |
| **AUTH-005** | PRD §4.1 | T3-V2 | VIVEK | `frontend/src/features/auth/LoginView.tsx` | Vitest LoginView specs | Renders email/password inputs with validation feedback | Browser E2E |
| **AUTH-006** | PRD §4.1 | T3-V2 | VIVEK | `frontend/src/features/auth/RegisterView.tsx` | Vitest RegisterView specs | Displays password strength meter; navigates to onboarding | Browser E2E |
| **AUTH-007** | PRD §4.1 | T3-V1 | VIVEK | `core/auth/services/password_reset.py` | `tests/test_auth.py::test_reset_mock` | Simulates local dev token generation for password reset | Dev mailbox test |
| **AUTH-008** | PRD §4.1 | T3-V1 | VIVEK | `core/auth/services/demo_auth.py` | `tests/test_auth.py::test_demo_login` | Single-click login as `demo@finwise.local` | J-6 Viva demo |

---

### 2.2 Shell, Navigation & Layout (SHELL)

| Req ID | PRD Source | Task ID | Owner | Target Implementation File | Verification Test | Acceptance Criteria | Integration Verification |
|---|---|---|---|---|---|---|---|
| **SHELL-001** | PRD §4.1 | T4-V2 | VIVEK | `frontend/src/features/shell/AppLayout.tsx` | Vitest AppLayout specs | Responsive sidebar on desktop, bottom navigation on mobile | Visual smoke test |
| **SHELL-002** | PRD §4.1 | T4-V2 | VIVEK | `frontend/src/features/shell/ThemeToggle.tsx` | Vitest ThemeToggle specs | Toggles dark/light class on `<html>` element; persists in localStorage | Visual inspection |

---

### 2.3 In-App Notifications (NOTIF)

| Req ID | PRD Source | Task ID | Owner | Target Implementation File | Verification Test | Acceptance Criteria | Integration Verification |
|---|---|---|---|---|---|---|---|
| **NOTIF-001** | PRD §4.1 | T10-V2 | VIVEK | `core/notifications/services.py` | `tests/test_notifications.py` | Creates unread alert when purchase cooling-off period expires | Header bell icon |

---

### 2.4 Money & Wealth Management (MONEY)

| Req ID | PRD Source | Task ID | Owner | Target Implementation File | Verification Test | Acceptance Criteria | Integration Verification |
|---|---|---|---|---|---|---|---|
| **MONEY-001** | PRD §4.2 | T5-A1 | AMAN | `modules/money/routers/transactions.py` | `tests/test_transactions.py::test_create` | Logs date, amount, description, account, category | J-2 Manual log |
| **MONEY-002** | PRD §4.2 | T5-A1 | AMAN | `modules/money/services/category_service.py` | `tests/test_transactions.py::test_hash` | Generates SHA-256 hash; prevents duplicate entries | Duplicate warning |
| **MONEY-003** | PRD §4.2 | T7-A1 | AMAN | `modules/money/services/import_service.py` | `tests/test_csv_import.py::test_staging` | Ingests CSV to staging; purges staged rows after 30 min | J-2 CSV import |
| **MONEY-004** | PRD §4.2 | T7-A1 | AMAN | `modules/money/services/import_service.py` | `tests/test_csv_import.py::test_commit` | Atomic commit moves approved rows to transactions table | Balance update |
| **MONEY-005** | PRD §4.2 | T6-A1 | AMAN | `modules/money/services/budget_service.py` | `tests/test_budgets.py::test_allocation` | Allocates monthly amounts across standard expense categories | Budget overview |
| **MONEY-006** | PRD §4.2 | T6-A1 | AMAN | `modules/money/services/budget_service.py` | `tests/test_budgets.py::test_50_30_20` | Computes Needs (50%), Wants (30%), Savings (20%) breakdown | Onboarding setup |
| **MONEY-007** | PRD §4.2 | T6-A1 | AMAN | `modules/money/services/budget_service.py` | `tests/test_budgets.py::test_rollover` | Calculates unspent budget rollover and daily burn velocity | Gauge update |
| **MONEY-008** | PRD §4.2 | T6-A2 | AMAN | `modules/money/services/spending_service.py` | `tests/test_spending_money.py` | Returns accurate Discretionary Spending Money ($S$) via contract | Decision Engine |
| **MONEY-009** | PRD §4.2 | T6-A1 | AMAN | `modules/money/services/goal_service.py` | `tests/test_goals.py::test_create_goal` | Tracks target amount, saved amount, deadline date | Goal list view |
| **MONEY-010** | PRD §4.2 | T6-A1 | AMAN | `modules/money/services/goal_service.py` | `tests/test_goals.py::test_lock_funds` | Prevents goal-allocated funds from appearing in spending money | Contract check |
| **MONEY-011** | PRD §4.2 | T6-A2 | AMAN | `modules/money/services/spending_service.py` | `tests/test_spending_money.py::test_buffer` | Reserves minimum emergency fund buffer (e.g., 3 months fixed costs) | Decision check |
| **MONEY-012** | PRD §4.2 | T5-A1 | AMAN | `modules/money/routers/transactions.py` | `tests/test_transactions.py::test_filter` | Multi-filters by date range, category, amount, keyword | Filtered table |
| **MONEY-013** | PRD §4.2 | T5-A1 | AMAN | `modules/money/models/transaction.py` | `tests/test_transactions.py::test_soft_del` | `is_deleted=True` flag hides transaction while preserving audit trail | Audit trail view |
| **MONEY-014** | PRD §4.2 | T7-A2 | AMAN | `frontend/src/features/money/SpendCharts.tsx`| Vitest SpendCharts specs | Renders category donut and monthly spending line chart | Dashboard charts |
| **MONEY-014b**| PRD §4.2 | T7-A2 | AMAN | `frontend/src/features/money/IncomeExpenseBar.tsx`| Vitest IncomeExpenseBar specs | Renders monthly income vs expense comparative stacked bars | Analytics tab |
| **MONEY-015** | PRD §4.2 | T7-A1 | AMAN | `modules/money/services/export_service.py` | `tests/test_csv_injection.py` | Prepends `'` to cells starting with `=`, `+`, `-`, `@` upon export | Downloaded CSV |

---

### 2.5 Price Tracking (PRICE)

| Req ID | PRD Source | Task ID | Owner | Target Implementation File | Verification Test | Acceptance Criteria | Integration Verification |
|---|---|---|---|---|---|---|---|
| **PRICE-001** | PRD §4.3 | T8-V1 | VIVEK | `modules/pricing/services/product_service.py` | `tests/test_pricing.py::test_add_prod` | Stores product title, target URL, category, created timestamp | Product card |
| **PRICE-002** | PRD §4.3 | T8-V1 | VIVEK | `modules/pricing/services/price_service.py` | `tests/test_pricing.py::test_log_price` | Appends manual price observation with UTC timestamp | Observations list |
| **PRICE-003** | PRD §4.3 | T8-V1 | VIVEK | `modules/pricing/services/price_service.py` | `tests/test_pricing.py::test_metrics` | Computes $P_{\min}$, $P_{\max}$, $P_{\text{avg}}$, and volatility percentage | Analytics badge |
| **PRICE-004** | PRD §4.3 | T8-V1 | VIVEK | `modules/pricing/routers/products.py` | `tests/test_pricing.py::test_demo_prods` | Demo products loadable into any user account | J-3 Tracking |
| **PRICE-005** | PRD §4.3 | T8-V2 | VIVEK | `frontend/src/features/pricing/PriceChart.tsx` | Vitest PriceChart specs | Renders Recharts SVG showing price changes over time with min/avg lines | Product detail |

---

### 2.6 Decision Engine & Smart Purchasing (DECIDE)

| Req ID | PRD Source | Task ID | Owner | Target Implementation File | Verification Test | Acceptance Criteria | Integration Verification |
|---|---|---|---|---|---|---|---|
| **DECIDE-001**| PRD §4.4 | T10-V2| VIVEK | `frontend/src/features/decision/EvaluateForm.tsx`| Vitest EvaluateForm specs | Submits product, intended price, and priority rating (1-5) | J-4 Decision flow |
| **DECIDE-002**| PRD §4.4 | T10-V1| VIVEK | `modules/decision/services/rules/rule1.py` | `tests/test_decision.py::test_rule1` | Triggers AVOID if price $> 100\%$ spending money; WAIT if $> 50\%$ | Verdict output |
| **DECIDE-003**| PRD §4.4 | T10-V1| VIVEK | `modules/decision/services/rules/rule2.py` | `tests/test_decision.py::test_rule2` | Checks category budget velocity; warns if pace exceeded | Verdict output |
| **DECIDE-004**| PRD §4.4 | T10-V1| VIVEK | `modules/decision/services/rules/rule3.py` | `tests/test_decision.py::test_rule3` | Triggers WAIT if price $> P_{\text{avg}}$; AVOID if high span & high price | Verdict output |
| **DECIDE-005**| PRD §4.4 | T10-V1| VIVEK | `modules/decision/services/rules/rule4.py` | `tests/test_decision.py::test_rule4` | Triggers AVOID if priority $\le 2$ and daily velocity is negative | Verdict output |
| **DECIDE-006**| PRD §4.4 | T10-V1| VIVEK | `modules/decision/services/rules/rule5.py` | `tests/test_decision.py::test_rule5` | Checks savings goals impact; triggers WAIT if goal delayed $>30$ days | Verdict output |
| **DECIDE-007**| PRD §4.4 | T10-V1| VIVEK | `modules/decision/services/engine.py` | `tests/test_decision.py::test_verdicts` | Combines rules into final BUY, WAIT, or AVOID with justifications | Verdict card |
| **DECIDE-008**| PRD §4.4 | T10-V1| VIVEK | `modules/decision/services/cooldown.py` | `tests/test_decision.py::test_cooldown` | Enforces 24h to 168h cooling-off duration based on price/spending ratio | Cooldown record |
| **DECIDE-009**| PRD §4.4 | T10-V2| VIVEK | `frontend/src/features/decision/CooldownTimer.tsx`| Vitest CooldownTimer specs | Displays real-time countdown timer; disables purchase confirmation until done | Active timer UI |
| **DECIDE-010**| PRD §4.4 | T10-V1| VIVEK | `modules/decision/services/decision_service.py`| `tests/test_decision.py::test_override` | Requires user confirmation acknowledgment before overriding WAIT/AVOID | Audit log entry |

---

### 2.7 Overview & Settings (OVR / SET)

| Req ID | PRD Source | Task ID | Owner | Target Implementation File | Verification Test | Acceptance Criteria | Integration Verification |
|---|---|---|---|---|---|---|---|
| **OVR-001** | PRD §4.5 | T11-V1| VIVEK | `backend/app/core/overview.py` | `tests/test_overview.py::test_kpis` | Aggregates income, expense, net savings, active cooldowns | Dashboard view |
| **OVR-002** | PRD §4.5 | T11-V1| VIVEK | `frontend/src/features/overview/SpendingGauge.tsx`| Vitest SpendingGauge specs | Visual gauge displaying safe spending money remaining for current month | J-5 Overview |
| **OVR-003** | PRD §4.5 | T11-V1| VIVEK | `frontend/src/features/overview/HealthScore.tsx` | Vitest HealthScore specs | Displays composite financial health score (0 to 100) | Dashboard KPI |
| **SET-001..005**| PRD §4.6 | T11-V1| VIVEK | `backend/app/core/settings/` | `tests/test_settings.py` | Currency selector (INR default), emergency fund target months, theme | Settings page |

---

### 2.8 Security & Non-Functional Requirements (SEC / NFR)

| Req ID | PRD Source | Task ID | Owner | Target Implementation File | Verification Test | Acceptance Criteria | Integration Verification |
|---|---|---|---|---|---|---|---|
| **SEC-01** | PRD §9 | T0-A3 | AMAN | `core/crypto.py` | `tests/test_crypto.py` | Passwords hashed using Argon2id | DB inspection |
| **SEC-02** | PRD §9 | T0-A3 | AMAN | `core/crypto.py` | `tests/test_crypto.py` | Sensitive tokens encrypted via Fernet (AES-128-CBC) | DB inspection |
| **SEC-03** | PRD §9 | T0-A2 | AMAN | `core/db.py` | `tests/test_wal_concurrency.py` | SQLite runs in WAL mode with 5000ms busy timeout | Concurrency run |
| **SEC-04** | PRD §9 | T0-A1 | AMAN | `core/middleware.py` | `tests/test_security_headers.py` | Returns CSP, X-Frame-Options, X-Content-Type-Options headers | HTTP response |
| **NFR-01** | PRD §10| T12-A1| AMAN | Performance benchmarks | `tests/test_benchmarks.py` | All API response times $< 200\text{ms}$ on standard laptop | Benchmark log |
| **NFR-02** | PRD §10| T11-A1| AMAN | `scripts/seed_demo.py` | `test_seed.py` | Ingests 90-day demo database in $< 3.0\text{ seconds}$ | Viva readiness |
