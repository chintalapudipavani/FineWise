# FinWise AI — Architecture & Engineering Decision Log

**Document ID:** DOC-DEC-009  
**Version:** 1.0.0  
**Authority:** Architectural Review Board (Aman & Vivek)  
**Last Updated:** 20-Oct-2026  

---

## 1. Governance & RFC Approval Process

This log records all foundational architectural choices, cross-document contradiction resolutions, and Request for Changes (RFCs) affecting the shared contract, module boundaries, or dependencies.

- **Status Field Values:** `PROPOSED`, `IN_REVIEW`, `APPROVED`, `REJECTED`, `SUPERSEDED`.
- **Approval Rule:** Any decision impacting shared interfaces, database schemas, or dependencies requires explicit approval from **both Aman and Vivek**.

---

## 2. Approved Architectural & Engineering Decisions

### DEC-01: Argon2id Parameter Configuration for Password Security
- **Decision ID:** `DEC-01`
- **Date:** 20-Oct-2026
- **Status:** **APPROVED**
- **Decision Owner:** Aman & Vivek
- **Question / Issue:** What password hashing algorithm and cost parameters should FinWise AI use to guarantee academic security compliance without overwhelming low-spec evaluation hardware?
- **Proposed Solution:** Use `argon2-cffi` implementing Argon2id with parameters: `time_cost=3`, `memory_cost=65536` (64 MB), `parallelism=4`.
- **Impact on AMAN:** Implements `hash_password` and `verify_password` inside `app.core.crypto`.
- **Impact on VIVEK:** Consumes `app.core.crypto` inside `app.core.auth.services.login`.
- **Rationale:** Meets PRD SEC-02 requirements and OWASP recommendations while executing in under 150ms on standard Windows laptops.

---

### DEC-02: Cryptographic Token Storage (Fernet vs Raw AES)
- **Decision ID:** `DEC-02`
- **Date:** 20-Oct-2026
- **Status:** **APPROVED**
- **Decision Owner:** Aman
- **Question / Issue:** How should sensitive tokens or confidential fields stored in SQLite be encrypted at rest?
- **Proposed Solution:** Use Python `cryptography.fernet.Fernet` (AES-128-CBC with PKCS7 padding and HMAC-SHA256 authentication). The encryption key is loaded from `FERNET_KEY` in `.env`.
- **Impact on AMAN:** Implements `encrypt_token` and `decrypt_token` in `app.core.crypto`.
- **Impact on VIVEK:** None directly for Core.
- **Rationale:** Zero external C-library dependencies; prevents tamper attacks through authenticated ciphertext.

---

### DEC-03: Frontend State Management Architecture
- **Decision ID:** `DEC-03`
- **Date:** 20-Oct-2026
- **Status:** **APPROVED**
- **Decision Owner:** Vivek
- **Question / Issue:** Should the frontend use Redux Toolkit, Zustand, or TanStack Query + Context?
- **Proposed Solution:** Use **TanStack Query v5** (`@tanstack/react-query`) for all server state, cache invalidation, and optimistic mutations; use native **React Context** exclusively for client session auth state.
- **Impact on AMAN:** Money components fetch data using standard `useQuery` hooks.
- **Impact on VIVEK:** Scaffolds `QueryClientProvider` in `frontend/src/app/App.tsx`.
- **Rationale:** Eliminates boilerplate reducer/slice code, avoids sync bugs, and provides out-of-the-box caching and loading states.

---

### DEC-04: Inter-Module Spending Money Contract Protocol
- **Decision ID:** `DEC-04`
- **Date:** 20-Oct-2026
- **Status:** **APPROVED**
- **Decision Owner:** Architect (Aman & Vivek sign-off)
- **Question / Issue:** CON-05 identified conflicting method signatures between PRD §6.5 (`get_remaining_spending_money(user_id)`) and Role files (`get_spending_money(user_id, today)`).
- **Proposed Solution:** Standardize on `get_spending_money(user_id: int, as_of: date) -> SpendingMoneyResult` returning:
  `available_balance`, `fixed_costs_reserved`, `savings_goals_reserved`, `emergency_buffer_reserved`, `discretionary_spending_money`, `rule_applied`, `days_remaining_in_month`, `daily_budget_velocity`.
- **Impact on AMAN:** Must implement `MoneyModuleProtocol` with this exact signature in `app.modules.money.services.spending_service`.
- **Impact on VIVEK:** Consumes this contract in Decision Engine Rules 1 and 2; uses `MockMoneyService` matching this model during development.
- **Rationale:** Passing `as_of: date` allows deterministic unit testing and time-travel simulation without altering system time.

---

### DEC-05: UI Styling & Component Strategy
- **Decision ID:** `DEC-05`
- **Date:** 20-Oct-2026
- **Status:** **APPROVED**
- **Decision Owner:** Vivek
- **Question / Issue:** Should the team import an external component library (MUI, Ant Design, Chakra) or build with Tailwind CSS primitives?
- **Proposed Solution:** Build custom UI primitives (`Button`, `Modal`, `Input`, `Badge`, `Card`) using **Tailwind CSS 3.4** and `clsx` / `tailwind-merge`.
- **Impact on AMAN:** Aman reuses Vivek's primitives in `frontend/src/shared/ui/` for Money components.
- **Impact on VIVEK:** Vivek creates atomic primitives in Task T4.
- **Rationale:** Keeps bundle size tiny, avoids heavy third-party CSS overrides, and ensures responsive light/dark theme consistency.

---

### DEC-06: Resolution of Cross-Document Contradictions (CON-01 to CON-07)
- **Decision ID:** `DEC-06`
- **Date:** 20-Oct-2026
- **Status:** **APPROVED**
- **Decision Owner:** Architect
- **Question / Issue:** Ambiguities identified during PRD and repository audit.
- **Proposed Resolutions:**
  - **CON-01 (Expired Imports):** Expired imports transition to `cancelled` status; staged rows deleted.
  - **CON-02 (Export Guard):** Formula escaping applied to `description_raw` and `description_normalized`; no phantom `notes` column.
  - **CON-03 (MONEY-014 Tag):** Core encompasses category breakdown and trends; comparative income vs expense bars is Plus tier.
  - **CON-04 (Settings Directory):** Adopt dedicated subdirectories `backend/app/core/settings/` and `core/overview.py`.
  - **CON-06 (Rule 3 Confidence):** High-confidence data volume is a prerequisite gate; verdict confidence remains Medium.
  - **CON-07 (Demo Account Isolation):** Demo financial transactions restricted to demo account; demo products permitted on all accounts.
- **Impact on AMAN & VIVEK:** Documented in handbooks and locked in shared contract.
- **Rationale:** Removes all ambiguity prior to parallel code execution.

---

### DEC-07: Academic Policy Resolution on AI-Assisted Code (BLK-10)
- **Decision ID:** `DEC-07`
- **Date:** 20-Oct-2026
- **Status:** **APPROVED**
- **Decision Owner:** Aman & Vivek
- **Question / Issue:** University regulations regarding AI code generation tools.
- **Proposed Solution:** AI-assisted development is fully permitted by institution. Both developers are cleared to use automated code and documentation assistants.
- **Impact on AMAN & VIVEK:** Removes B-P0 blocker.
- **Rationale:** Confirmed by student team lead.

---

## 3. RFC Template for Future Contract Changes

If either developer requires a change to `docs/SHARED_TECHNICAL_CONTRACT.md`, they must append an RFC entry below:

```markdown
### RFC-XX: [Brief Title]
- **Proposed By:** [AMAN / VIVEK]
- **Date:** [YYYY-MM-DD]
- **Status:** PROPOSED
- **Problem Statement:** [What is broken or missing?]
- **Proposed Diff / Interface Change:** [Exact code or schema diff]
- **Impact on Aman:** [Changes required in Aman's module]
- **Impact on Vivek:** [Changes required in Vivek's module]
- **Approval Sign-off:** [ ] Aman [ ] Vivek
```
