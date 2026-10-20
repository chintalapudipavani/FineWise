# FinWise AI — Architecture & Module Ownership Specification

**Document ID:** DOC-ARCH-002  
**Version:** 1.0.0 (Contract Frozen)  
**Effective Date:** 20-Oct-2026  
**Target Milestone:** M0 (Contract Freeze)  
**Authors:** Senior Software Architect & Lead Integration Specialist  

---

## 1. Architectural Philosophy & System Topography

FinWise AI is architected as an **In-Process Modular Monolith**. It deliberately avoids the operational complexity, network latency, and infrastructure cost of distributed microservices while enforcing the strict isolation, decoupled lifecycles, and bounded contexts typical of enterprise service-oriented architectures.

```
+-----------------------------------------------------------------------------------+
|                                 FRONTEND (React 18 + Vite)                        |
|  +--------------------+  +--------------------+  +-----------------------------+  |
|  | Shell, Auth, Nav   |  | Money UI & Reports |  | Pricing, Decision, Settings |  |
|  | (VIVEK)            |  | (AMAN)             |  | (VIVEK)                     |  |
|  +--------------------+  +--------------------+  +-----------------------------+  |
|  +-----------------------------------------------------------------------------+  |
|  | Shared Design System / Primitives & Generated API Client (SHARED)           |  |
+--+-----------------------------------------------------------------------------+--+
                                        |  HTTP / REST (JSON)
                                        v
+-----------------------------------------------------------------------------------+
|                                 BACKEND (FastAPI Layer)                           |
|  +-----------------------------------------------------------------------------+  |
|  | Root Application Router, CORS, Security Headers, Middleware (AMAN)          |  |
|  +-----------------------------------------------------------------------------+  |
|  | Core Infrastructure: Config, DB Engine, Clock, Crypto, Health (AMAN)        |  |
|  +-----------------------------------------------------------------------------+  |
|  | Core Services: Auth, User Management, Notifications, Overview (VIVEK)       |  |
|  +-----------------------------------------------------------------------------+  |
|  |                                                                             |  |
|  |   MODULE: Money & Wealth (AMAN)            MODULE: Pricing & Catalogs(VIVEK)|  |
|  |   - Transactions & Categories              - Product Registry               |  |
|  |   - Budgets & Rollovers                    - Manual Price Observation Log   |  |
|  |   - CSV Ingestion & Staging Engine         - Price Drop & Volatility Math   |  |
|  |   - Savings Goals & Progress               +--------------------------------+  |
|  |   - Category Breakdowns & Trends           MODULE: Decision Engine (VIVEK)  |  |
|  |                                            - Rules 1 to 5 Evaluation Engine |  |
|  |                                            - Cooling-Off Period Tracker     |  |
|  |                                            - Purchase Verdict Generator     |  |
|  |                                                                             |  |
|  |   INTER-MODULE CONTRACT: `core/contracts.py` (STRICT PROTOCOL BOUNDARY)     |  |
|  |   Pricing & Decision NEVER queries Money tables directly.                   |  |
|  |   Pricing & Decision invokes: `get_spending_money(user_id, as_of)`          |  |
|  +-----------------------------------------------------------------------------+  |
|                                       |
|                                       v
|                             PERSISTENCE (SQLite WAL)                              |
|   - Multi-table relational schema managed by Alembic                              |
|   - Full isolation of table writes per module                                     |
+-----------------------------------------------------------------------------------+
```

---

## 2. Module Boundaries & Ownership Breakdown

### 2.1 Backend Ownership Matrix

| Subsystem / Directory | Domain Scope | Primary Owner | Secondary Access | Communication Mechanism |
|---|---|---|---|---|
| `backend/app/core/config.py` | Environment parsing, Pydantic settings | **AMAN** | Read-Only (VIVEK) | Direct Python import |
| `backend/app/core/db.py` | SQLAlchemy Engine, SessionLocal, WAL Pragmas | **AMAN** | Read-Only (VIVEK) | `get_db` FastAPI dependency |
| `backend/app/core/clock.py` | Deterministic system clock provider | **AMAN** | Read-Only (VIVEK) | Direct Python import |
| `backend/app/core/crypto.py` | Argon2id & Fernet cryptographic utilities | **AMAN** | Read-Only (VIVEK) | Direct Python import |
| `backend/app/core/middleware.py` | CORS, Security Headers, Request ID logging | **AMAN** | No Access | Injected at FastAPI boot |
| `backend/app/core/logging.py` | Structured logger formatter | **AMAN** | Read-Only (VIVEK) | Standard library `logging` |
| `backend/app/core/health.py` | Liveness and DB readiness endpoints | **AMAN** | Read-Only (VIVEK) | HTTP endpoint `/health` |
| `backend/app/core/contracts.py` | Inter-module protocol definitions & stubs | **SHARED** | Shared / Frozen | Strict PEP 544 Protocol |
| `backend/app/core/auth/` | JWT, user registration, login, auth middleware | **VIVEK** | Read-Only (AMAN) | `get_current_user` dependency |
| `backend/app/core/notifications/`| In-app notification creation & queries | **VIVEK** | Read-Only (AMAN) | `NotificationService` call |
| `backend/app/core/settings/` | User settings, currency, notification prefs | **VIVEK** | Read-Only (AMAN) | HTTP endpoints |
| `backend/app/core/overview.py` | Dashboard financial health aggregation | **VIVEK** | No Access | HTTP endpoint `/api/overview` |
| `backend/app/modules/money/` | Transactions, Budgets, CSV Import, Goals | **AMAN** | **STRICTLY PROHIBITED** | Inter-module contract protocol |
| `backend/app/modules/pricing/` | Product registry, Price observations | **VIVEK** | **STRICTLY PROHIBITED** | Independent REST endpoints |
| `backend/app/modules/decision/`| Decision rules, Cooling-off periods | **VIVEK** | **STRICTLY PROHIBITED** | Independent REST endpoints |
| `backend/app/modules/invest/` | [Stretch] Investment simulator | **AMAN** | No Access | Independent REST endpoints |
| `backend/alembic/` | Database migration scripts | **AMAN** | Read-Only (VIVEK) | CLI `alembic upgrade head` |

### 2.2 Frontend Ownership Matrix

| Feature / Directory | UI Scope | Primary Owner | Secondary Access | Integration Boundary |
|---|---|---|---|---|
| `frontend/src/app/` | Root Providers, QueryClient, Layout router | **VIVEK** | Read-Only (AMAN) | Aman registers money routes |
| `frontend/src/shared/ui/` | Primitives (Button, Modal, Input, Badge, Table) | **VIVEK** | Read-Only (AMAN) | Reused by Aman components |
| `frontend/src/shared/api/` | Generated OpenAPI Axios/fetch client | **SHARED** | Generated | Run `npm run codegen` |
| `frontend/src/features/auth/` | Sign In, Sign Up, Password Reset screens | **VIVEK** | No Access | Global AuthContext |
| `frontend/src/features/shell/` | Header, Sidebar, Bottom nav, Theme toggle | **VIVEK** | No Access | Layout wrapper |
| `frontend/src/features/money/` | Transaction list, Add/Edit modal, Import CSV | **AMAN** | **STRICTLY PROHIBITED** | Feature component exports |
| `frontend/src/features/pricing/`| Products list, Add Price modal, Price chart | **VIVEK** | **STRICTLY PROHIBITED** | Feature component exports |
| `frontend/src/features/decision/`| Purchase check form, Verdict banner, Cooldown | **VIVEK** | **STRICTLY PROHIBITED** | Feature component exports |
| `frontend/src/features/overview/`| Dashboard KPIs, Health score, Spending gauge | **VIVEK** | No Access | Feature component exports |
| `frontend/src/features/settings/`| Profile form, Currency dropdown, Notification | **VIVEK** | No Access | Feature component exports |

---

## 3. Communication Rules Between Modules

To prevent circular dependencies, tight coupling, and merge conflicts, the following structural rules are strictly enforced:

### Rule 1: No Cross-Module ORM Querying
- The Decision module or Vivek's code **must never import models or query tables** belonging to the Money module (e.g. `from app.modules.money.models import Transaction` is an immediate CI failure).
- The Money module **must never import models or query tables** belonging to Pricing or Decision.

### Rule 2: Contract-Only Backend Communication
- When Vivek's Decision Engine needs financial numbers (available balance, budget caps, savings targets) to evaluate a purchase, it queries the contract protocol:
  ```python
  from app.core.contracts import MoneyModuleProtocol, get_money_service
  
  money_service: MoneyModuleProtocol = get_money_service()
  spending_data = money_service.get_spending_money(user_id=user.id, as_of=clock.today())
  ```
- During independent development, Vivek uses a test double / stub (`MockMoneyService`) that implements `MoneyModuleProtocol`.

### Rule 3: Autonomous Frontend Routing
- Vivek provides a clean feature plug-in point in `frontend/src/app/routes.tsx`.
- Aman creates and maintains all components inside `frontend/src/features/money/`.
- Aman tests `frontend/src/features/money/` independently by rendering the feature within a local test wrapper or using Vite dev server proxy.

---

## 4. File Ownership & Modification Policy

To guarantee zero accidental file overwrites during concurrent git operations, all repository files are assigned to one of five strict ownership tiers:

```
[Tier 1: AMAN Owned] --------> Only Aman may edit. Vivek may read.
[Tier 2: VIVEK Owned] -------> Only Vivek may edit. Aman may read.
[Tier 3: Centrally Controlled] -> Modification requires formal RFC & mutual sign-off.
[Tier 4: Shared Read-Only] ---> Static references; immutable during development.
[Tier 5: Auto-Generated] ------> Never manually edited; overwritten by build tooling.
```

### 4.1 Detailed File Classification Table

| File / Pattern | Tier | Owner | Modification Protocol |
|---|---|---|---|
| `backend/app/core/config.py` | Tier 1 | AMAN | Aman maintains environment settings. |
| `backend/app/core/db.py` | Tier 1 | AMAN | Aman maintains database pooling and engine. |
| `backend/app/core/clock.py` | Tier 1 | AMAN | Aman maintains system clock. |
| `backend/app/core/crypto.py` | Tier 1 | AMAN | Aman maintains hashing & Fernet encryption. |
| `backend/app/core/middleware.py` | Tier 1 | AMAN | Aman maintains HTTP middleware. |
| `backend/app/core/logging.py` | Tier 1 | AMAN | Aman maintains logging formatters. |
| `backend/app/core/health.py` | Tier 1 | AMAN | Aman maintains health endpoints. |
| `backend/app/modules/money/**` | Tier 1 | AMAN | Aman has exclusive read/write ownership. |
| `backend/app/modules/invest/**` | Tier 1 | AMAN | Aman has exclusive read/write ownership. |
| `backend/alembic/**` | Tier 1 | AMAN | Aman writes all schema migrations. |
| `frontend/src/features/money/**` | Tier 1 | AMAN | Aman writes all money UI components. |
| `scripts/**` | Tier 1 | AMAN | Aman maintains seed and backup scripts. |
| `backend/app/core/auth/**` | Tier 2 | VIVEK | Vivek maintains auth and token lifecycle. |
| `backend/app/core/notifications/**`| Tier 2 | VIVEK | Vivek maintains in-app notification engine. |
| `backend/app/core/settings/**` | Tier 2 | VIVEK | Vivek maintains user settings endpoints. |
| `backend/app/core/overview.py` | Tier 2 | VIVEK | Vivek maintains dashboard endpoint. |
| `backend/app/modules/pricing/**` | Tier 2 | VIVEK | Vivek has exclusive read/write ownership. |
| `backend/app/modules/decision/**` | Tier 2 | VIVEK | Vivek has exclusive read/write ownership. |
| `frontend/src/app/**` | Tier 2 | VIVEK | Vivek maintains shell, providers, and layout. |
| `frontend/src/shared/ui/**` | Tier 2 | VIVEK | Vivek creates and maintains UI primitives. |
| `frontend/src/features/auth/**` | Tier 2 | VIVEK | Vivek maintains login/registration views. |
| `frontend/src/features/pricing/**`| Tier 2 | VIVEK | Vivek maintains pricing views. |
| `frontend/src/features/decision/**`| Tier 2 | VIVEK | Vivek maintains decision engine views. |
| `frontend/src/features/overview/**`| Tier 2 | VIVEK | Vivek maintains overview dashboard views. |
| `frontend/src/features/settings/**`| Tier 2 | VIVEK | Vivek maintains user settings views. |
| `backend/app/core/contracts.py` | Tier 3 | SHARED | Frozen in M0. Requires formal RFC for change. |
| `backend/pyproject.toml` | Tier 3 | SHARED | Aman owns base dependencies; Vivek proposes additions via RFC. |
| `frontend/package.json` | Tier 3 | SHARED | Vivek owns base dependencies; Aman proposes additions via RFC. |
| `backend/app/main.py` | Tier 3 | SHARED | Scaffolding created by Aman; Vivek mounts his routers. |
| `docs/SHARED_TECHNICAL_CONTRACT.md` | Tier 3 | SHARED | Single source of truth for API and model contracts. |
| `FinWise AI — Product Requirements Document.md` | Tier 4 | SHARED | Read-only canonical source of requirements. |
| `docs/api-spec.yaml` | Tier 5 | AUTO | Generated from FastAPI via script `export_openapi.py`. |
| `frontend/src/shared/api/*` | Tier 5 | AUTO | Generated from `api-spec.yaml` via openapi-typescript. |

---

## 5. Change Proposal & Shared-File RFC Procedure

When either developer requires an addition or modification to a **Tier 3 (Centrally Controlled)** file, the following 4-step RFC procedure must be executed:

```
[1. Issue RFC] -> [2. Impact Review] -> [3. Formal Approval] -> [4. Atomic Merge]
```

1. **Issue RFC:** The proposing developer creates an RFC entry in `docs/DECISION_LOG.md` detailing:
   - File to be modified.
   - Exact diff or proposed function signature change.
   - Justification and requirement ID triggering the change.
2. **Impact Review:** The receiving developer reviews the RFC within 24 hours to identify any breaking impact on their unintegrated branch.
3. **Formal Approval:** Both developers must digitally approve the entry in `docs/DECISION_LOG.md` (setting Status to `APPROVED`).
4. **Atomic Merge:** The change is committed directly to `main` (or the integration branch), and both developers immediately rebase their active feature branches.

---

## 6. Engineering Conventions & Coding Standards

Both developers must strictly adhere to identical tooling, linting, and architectural styles:

### 6.1 Backend Standards (Python 3.12+)
- **Type Hints:** 100% type annotation coverage. Run `mypy --strict app` with zero errors.
- **Code Style & Formatting:** Enforced via Ruff (`ruff check .` and `ruff format .`). Line length: 100 characters.
- **SQLAlchemy 2.0 Syntax:** Use 2.0 `select()` and `session.scalars()`. Do not use legacy 1.x `session.query()`.
- **Database Schema Models:** All models inherit from `app.core.db.Base`. Use `mapped_column()` with explicit types and constraints.
- **Pydantic v2:** Use `model_config = ConfigDict(from_attributes=True)` instead of v1 `orm_mode = True`.
- **Error Responses:** All errors must raise `HTTPException` returning standard error envelope:
  ```json
  {
    "detail": {
      "code": "STRING_IDENTIFIER",
      "message": "Human readable explanation",
      "field": "optional_field_name"
    }
  }
  ```

### 6.2 Frontend Standards (React 18 + TypeScript)
- **TypeScript Strictness:** `"strict": true`, `"noImplicitAny": true`, `"exactOptionalPropertyTypes": true`.
- **Styling:** Tailwind CSS utility classes exclusively. Colors defined in `tailwind.config.js` design tokens.
- **State Management:**
  - Server state: TanStack Query `useQuery` and `useMutation`.
  - Local component state: `useState`, `useReducer`.
  - Session/Auth state: `AuthContext`.
- **Icons:** `lucide-react` icons exclusively.
- **Form Handling:** React Hook Form + Zod schema validation.

### 6.3 Git Branch & Commit Hygiene
- **Branch Naming:**
  - Aman: `feature/aman-money-backend`, `feature/aman-money-frontend`, `feature/aman-scripts`
  - Vivek: `feature/vivek-auth-shell`, `feature/vivek-pricing-decision`, `feature/vivek-overview`
- **Commit Format:** Conventional Commits:
  - `feat(money): add CSV ingestion parser for HDFC statements`
  - `fix(decision): correct Rule 3 cooling-off timer calculation`
  - `test(auth): add Argon2id password hash verification test`
- **Never force-push (`git push --force`)** to the integration or main branches.
