# FinWise AI — Parallel Development Tracker

**Document ID:** DOC-TRK-007  
**Version:** 1.0.0  
**Current Milestone:** M0 (Contract Freeze Complete)  
**Last Updated:** 20-Oct-2026  
**Tracking Authority:** Project Coordinator & Integration Specialist  

---

## 1. Lifecycle Milestones & Transition Gates

The project is governed by 7 milestone gates. Aman and Vivek progress through M1 to M4 asynchronously.

| Milestone | Title | Entry Criteria | Exit Criteria | Status |
|---|---|---|---|---|
| **M0** | **Contract Freeze** | PRD audit complete; CON-01..07 resolved. | `SHARED_TECHNICAL_CONTRACT.md` locked. | **DONE** |
| **M1** | **Independent Setup** | M0 locked. Dev branches created. | Both devs can boot local servers & tests. | **IN_PROGRESS** |
| **M2** | **Independent Implementation** | M1 verified. | All tasks implemented against mocks/stubs. | NOT_STARTED |
| **M3** | **Independent Verification** | M2 code complete. | Unit/Integration tests pass (>85% cov). | NOT_STARTED |
| **M4** | **Integration Readiness** | M3 passed. Clean git tree on feature branch. | PR submitted; handoff checklist signed. | NOT_STARTED |
| **M5** | **Integration & Wire-Up** | M4 approved for both branches. | Branches merged; mocks swapped for real. | NOT_STARTED |
| **M6** | **Release Readiness** | M5 merged. E2E tests passing. | Demo script verified; ready for viva. | NOT_STARTED |

---

## 2. Allowed Status Transitions & Verification Evidence

To advance a task from one status to another, the developer must provide the required objective evidence:

```
NOT_STARTED ──> IN_PROGRESS ──> READY_FOR_REVIEW ──> VERIFIED ──> READY_FOR_INTEGRATION ──> INTEGRATED ──> DONE
      │              │
      └──> BLOCKED <─┘
```

- **NOT_STARTED $\rightarrow$ IN_PROGRESS:** Branch created, dev environment active.
- **IN_PROGRESS $\rightarrow$ BLOCKED:** Blocker documented in tracker and escalation raised.
- **IN_PROGRESS $\rightarrow$ READY_FOR_REVIEW:** Code written, local tests pass, commit pushed to branch.
- **READY_FOR_REVIEW $\rightarrow$ VERIFIED:** Independent test command passes in clean environment; 0 lint/mypy errors.
- **VERIFIED $\rightarrow$ READY_FOR_INTEGRATION:** Handoff checklist complete, PR submitted.
- **READY_FOR_INTEGRATION $\rightarrow$ INTEGRATED:** Merged to `integration/parallel-development`, real services wired.
- **INTEGRATED $\rightarrow$ DONE:** End-to-end integration test passes, smoke test verified.

---

## 3. Comprehensive Task Matrix

| Task ID | Task Name | Owner | Pri | Dependencies | Owned Files / Module | Shared Interface | Independent Test Command | Acceptance Criteria | Status | Evidence / PR | Int Status | Blocker / Decision |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **T0-A1** | Python Project Scaffolding | AMAN | P0 | None | `backend/pyproject.toml`, `backend/app/main.py` | None | `uv sync && uv run ruff check .` | uv installs dependencies, ruff & mypy pass | NOT_STARTED | - | PENDING | None |
| **T0-A2** | Core Database & WAL Setup | AMAN | P0 | T0-A1 | `backend/app/core/db.py` | None | `uv run pytest tests/test_core_db.py` | SQLite initializes in WAL mode with 5000ms timeout | NOT_STARTED | - | PENDING | None |
| **T0-A3** | Cryptography & Clock Utils | AMAN | P0 | T0-A1 | `backend/app/core/crypto.py`, `clock.py` | None | `uv run pytest tests/test_core_crypto.py` | Argon2id and Fernet encryption work round-trip | NOT_STARTED | - | PENDING | None |
| **T2-A1** | Alembic Initial Schema | AMAN | P0 | T0-A2 | `backend/alembic/**` | DB Schema | `uv run alembic upgrade head` | 13 relational tables created cleanly | NOT_STARTED | - | PENDING | None |
| **T4-V1** | Vite & React Shell Setup | VIVEK | P0 | None | `frontend/package.json`, `vite.config.ts` | None | `cd frontend && npm run build` | React 18, Vite, Tailwind CSS compile cleanly | NOT_STARTED | - | PENDING | None |
| **T4-V2** | Responsive Layout & Primitives | VIVEK | P1 | T4-V1 | `frontend/src/features/shell/**`, `shared/ui/**` | UI Primitives | `npm run test:unit` | Sidebar, Header, and Button/Modal render properly | NOT_STARTED | - | PENDING | None |
| **T3-V1** | Auth Models & JWT Engine | VIVEK | P0 | T4-V1 | `backend/app/core/auth/**` | Bearer JWT | `uv run pytest tests/test_auth.py` | Register, login, JWT issue/verify pass | NOT_STARTED | - | PENDING | None |
| **T3-V2** | Auth Frontend Screens | VIVEK | P1 | T3-V1, T4-V2 | `frontend/src/features/auth/**` | `/api/v1/auth/*` | Vitest / Playwright auth specs | User can register and log in via UI | NOT_STARTED | - | PENDING | None |
| **T5-A1** | Transactions CRUD & Hash | AMAN | P0 | T2-A1 | `backend/app/modules/money/**` | Mock Auth | `uv run pytest tests/test_transactions.py` | Transactions persist with SHA256 duplicate hash | NOT_STARTED | - | PENDING | None |
| **T6-A1** | Budgets & 50/30/20 Engine | AMAN | P0 | T5-A1 | `backend/app/modules/money/services/budget*` | None | `uv run pytest tests/test_budgets.py` | 50/30/20 budget allocations computed accurately | NOT_STARTED | - | PENDING | None |
| **T6-A2** | Spending Money Contract | AMAN | P0 | T6-A1 | `backend/app/modules/money/services/spending*` | `core/contracts.py` | `uv run pytest tests/test_spending_money.py` | Implements `MoneyModuleProtocol.get_spending_money` | NOT_STARTED | - | PENDING | None |
| **T7-A1** | CSV Statement Staging Engine | AMAN | P1 | T5-A1 | `backend/app/modules/money/services/import*` | None | `uv run pytest tests/test_csv_import.py` | Parses HDFC/SBI/ICICI, 30m expiry, atomic commit | NOT_STARTED | - | PENDING | None |
| **T7-A2** | Money Subsystem Frontend UI | AMAN | P1 | T6-A2, T7-A1 | `frontend/src/features/money/**` | None | `cd frontend && npm run build` | Transaction table, import modal, budget gauges | NOT_STARTED | - | PENDING | None |
| **T8-V1** | Product Catalog & Price Log | VIVEK | P0 | T3-V1 | `backend/app/modules/pricing/**` | None | `uv run pytest tests/test_pricing.py` | Product registration & price observation history | NOT_STARTED | - | PENDING | None |
| **T8-V2** | Price History Recharts UI | VIVEK | P1 | T8-V1, T4-V2 | `frontend/src/features/pricing/**` | None | Vitest price chart specs | Renders historical price chart with min/avg lines | NOT_STARTED | - | PENDING | None |
| **T10-V1**| Decision Rules 1 to 5 Engine | VIVEK | P0 | T8-V1 | `backend/app/modules/decision/**` | `MockMoneyService` | `uv run pytest tests/test_decision_rules.py` | Rules 1 to 5 output BUY / WAIT / AVOID verdicts | NOT_STARTED | - | PENDING | None |
| **T10-V2**| Cooling-Off Tracker & UI | VIVEK | P1 | T10-V1 | `frontend/src/features/decision/**` | None | Vitest decision form specs | Renders verdict banner and countdown timer | NOT_STARTED | - | PENDING | None |
| **T11-A1**| Demo Data Seed Script | AMAN | P1 | T6-A2 | `scripts/seed_demo.py` | None | `uv run python scripts/seed_demo.py` | Ingests 90-day realistic demo state in <3s | NOT_STARTED | - | PENDING | None |
| **T11-V1**| Overview Dashboard & Settings | VIVEK | P1 | T10-V1 | `backend/app/core/overview.py`, `features/ovr*` | `MockMoneyService` | `uv run pytest tests/test_overview.py` | Aggregates health score, spending gauge, KPIs | NOT_STARTED | - | PENDING | None |
| **T12-A1**| SQLite WAL Concurrency Stress | AMAN | P1 | T0-A2, T5-A1 | `tests/test_wal_concurrency.py` | None | `uv run pytest tests/test_wal_concurrency.py` | 10 concurrent threads insert with 0 locked errors | NOT_STARTED | - | PENDING | None |
| **T12-V1**| Playwright E2E User Journeys | VIVEK | P1 | T10-V2, T11-V1 | `tests/e2e/**` | Full Stack | `npx playwright test` | Automates J-1 through J-6 end-to-end | NOT_STARTED | - | PENDING | None |
| **T14-INT**| Full System Integration & Wireup | BOTH | P0 | All Above | `main.py`, `routes.tsx` | Real Services | Complete test suite & smoke test | System functions as single integrated application | NOT_STARTED | - | PENDING | None |

---

## 4. Asynchronous Timeline Handling

If one developer finishes faster than the other:
1. **If Aman finishes first:**
   - Aman reaches M4 (Integration Readiness).
   - Aman's branch (`feature/aman-platform-money`) is code-frozen, tested, and archived as a release candidate.
   - Vivek continues developing against `docs/SHARED_TECHNICAL_CONTRACT.md` and `MockMoneyService`.
   - Aman does not idle; he proceeds to Stretch task `T13` (Investment Portfolio Simulator) on an isolated branch `feature/aman-invest-stretch`.
2. **If Vivek finishes first:**
   - Vivek reaches M4.
   - Vivek tests the entire UI using `MockMoneyService` and mock auth sessions.
   - When Aman completes his branch, Vivek joins the Integration Owner for the deterministic M5 merge sequence.
