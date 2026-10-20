# FinWise AI — Shared Technical Contract & Interface Specifications

**Document ID:** DOC-CONTRACT-003  
**Contract Version:** 1.0.0 (Freeze Status: **LOCKED**)  
**Last Updated:** 20-Oct-2026  
**Effective Date:** 20-Oct-2026  
**Authority:** Approved by Aman & Vivek for Parallel Development  

---

## 1. Revision History & Contract Freeze Statement

| Version | Date | Author | Summary of Changes | Impacted Modules |
|---|---|---|---|---|
| **1.0.0** | 20-Oct-2026 | Architect | Initial formal freeze. Resolved CON-01 through CON-07. Established `core/contracts.py` Protocol, Auth tokens, API schemas, and SQLite database migration ownership. | ALL |

> [!IMPORTANT]
> **Contract Freeze Notice (M0 Milestone):**  
> As of Version 1.0.0, all function signatures, REST route paths, request/response models, database table definitions, and environment variable names documented herein are **FROZEN**. Neither Aman nor Vivek may alter these specifications unilaterally. Any proposed alteration must follow the RFC procedure defined in `docs/ARCHITECTURE_AND_MODULE_OWNERSHIP.md`.

---

## 2. Authentication & Authorization Contract

### 2.1 HTTP Authorization Header
All protected API endpoints require an HTTP `Authorization` header containing an RFC 6750 Bearer JWT token:
```http
Authorization: Bearer <jwt_access_token>
```

### 2.2 JWT Payload Structure
Tokens are signed using `HS256` with secret key `FINWISE_JWT_SECRET`.
```json
{
  "sub": "user_id_as_string",
  "email": "user@example.com",
  "is_demo": false,
  "exp": 1735689600,
  "iat": 1735603200
}
```

### 2.3 Dependency Injection Protocol (`get_current_user`)
- **Location:** `app.core.auth.dependencies.get_current_user`
- **Output Type:** `AuthenticatedUser` model:
```python
from pydantic import BaseModel, ConfigDict

class AuthenticatedUser(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    
    id: int
    email: str
    is_demo: bool = False
    currency: str = "INR"
```

### 2.4 Aman's Test Double (Standalone Auth Bypass)
For independent testing of the Money module without requiring Vivek's Auth implementation:
```python
# In backend/app/core/auth/dependencies.py (or in test fixtures)
async def get_current_user_override() -> AuthenticatedUser:
    return AuthenticatedUser(id=1, email="aman.dev@local.test", is_demo=False, currency="INR")
```

---

## 3. Inter-Module In-Process Service Contract (`core/contracts.py`)

The primary inter-module runtime dependency is Vivek's Decision Engine and Overview Dashboard requesting discretionary spending data from Aman's Money module.

### 3.1 Pydantic Data Transfer Objects (DTOs)
```python
from datetime import date
from enum import Enum
from typing import Optional, List
from pydantic import BaseModel, Field

class SpendingRuleApplied(str, Enum):
    BUDGET_REMAINING = "BUDGET_REMAINING"
    MONTHLY_PROJECTION = "MONTHLY_PROJECTION"
    ZERO_BUFFER = "ZERO_BUFFER"

class SpendingMoneyResult(BaseModel):
    user_id: int
    as_of_date: date
    available_balance: float = Field(..., description="Liquid bank balance (INR)")
    fixed_costs_reserved: float = Field(..., description="Reserved for rent, bills, EMIs")
    savings_goals_reserved: float = Field(..., description="Allocated to active savings goals")
    emergency_buffer_reserved: float = Field(..., description="Minimum safety reserve threshold")
    discretionary_spending_money: float = Field(
        ..., 
        description="Net safe spending money: Balance - (Fixed + Savings + Emergency)"
    )
    rule_applied: SpendingRuleApplied
    days_remaining_in_month: int
    daily_budget_velocity: float = Field(..., description="Safe spending rate per day remaining")

class MoneySummaryDTO(BaseModel):
    user_id: int
    month: int
    year: int
    total_income: float
    total_expense: float
    net_savings: float
    savings_rate_percent: float
    health_score: int = Field(..., ge=0, le=100)
```

### 3.2 Formal Protocol Definition
```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class MoneyModuleProtocol(Protocol):
    """Formal protocol implemented by Aman's MoneyService and consumed by Vivek's Decision Engine."""
    
    def get_spending_money(self, user_id: int, as_of: date) -> SpendingMoneyResult:
        """Calculates net safe discretionary spending money for purchase evaluation (Rule 1)."""
        ...
        
    def get_monthly_summary(self, user_id: int, month: int, year: int) -> MoneySummaryDTO:
        """Retrieves aggregated cash flow summary for the Overview Dashboard."""
        ...
```

### 3.3 Vivek's Test Double (`MockMoneyService`)
Vivek must use this exact mock implementation when building the Decision Engine and Overview features:
```python
class MockMoneyService:
    """Deterministic stub used by Vivek to develop Decision Engine without Aman's module."""
    
    def __init__(self, discretionary_override: float = 15000.0):
        self._discretionary = discretionary_override

    def get_spending_money(self, user_id: int, as_of: date) -> SpendingMoneyResult:
        return SpendingMoneyResult(
            user_id=user_id,
            as_of_date=as_of,
            available_balance=50000.0,
            fixed_costs_reserved=20000.0,
            savings_goals_reserved=10000.0,
            emergency_buffer_reserved=5000.0,
            discretionary_spending_money=self._discretionary,
            rule_applied=SpendingRuleApplied.BUDGET_REMAINING,
            days_remaining_in_month=12,
            daily_budget_velocity=round(self._discretionary / 12, 2)
        )

    def get_monthly_summary(self, user_id: int, month: int, year: int) -> MoneySummaryDTO:
        return MoneySummaryDTO(
            user_id=user_id,
            month=month,
            year=year,
            total_income=80000.0,
            total_expense=45000.0,
            net_savings=35000.0,
            savings_rate_percent=43.75,
            health_score=82
        )
```

---

## 4. Shared REST API Endpoints Contract

### 4.1 Error Envelope Schema
Every HTTP 4xx and 5xx response must conform to this schema:
```json
{
  "detail": {
    "code": "VALIDATION_ERROR",
    "message": "Transaction amount must be greater than zero.",
    "field": "amount"
  }
}
```

### 4.2 Endpoint Catalog

#### Group A: Core Infrastructure & Health (AMAN)
- `GET /health` -> `{"status": "ok", "version": "1.0.0", "timestamp": "2026-10-20T12:00:00Z"}`
- `GET /health/db` -> `{"database": "healthy", "journal_mode": "wal"}`

#### Group B: Authentication & User Settings (VIVEK)
- `POST /api/v1/auth/register`
  - Request: `{"email": "user@test.com", "password": "SecurePassword123!"}`
  - Response (201): `{"id": 1, "email": "user@test.com", "created_at": "..."}`
- `POST /api/v1/auth/login`
  - Request: `{"email": "user@test.com", "password": "SecurePassword123!"}`
  - Response (200): `{"access_token": "eyJhbGciOi...", "token_type": "bearer", "expires_in": 86400}`
- `GET /api/v1/settings`
  - Response (200): `{"currency": "INR", "emergency_months": 3, "dark_mode": true}`
- `PUT /api/v1/settings`
  - Request: `{"currency": "INR", "emergency_months": 6, "dark_mode": true}`

#### Group C: Money Module (AMAN)
- `GET /api/v1/transactions`
  - Query Params: `page=1&limit=20&category_id=3&start_date=2026-10-01&end_date=2026-10-31`
  - Response (200): `{"items": [...], "total": 142, "page": 1, "limit": 20}`
- `POST /api/v1/transactions`
  - Request: `{"account_id": 1, "category_id": 2, "amount": 1450.00, "date": "2026-10-20", "description_raw": "Groceries at Supermart"}`
  - Response (201): Transaction object
- `POST /api/v1/imports/upload`
  - Multipart/form-data: `file=@statement.csv`, `bank_type=HDFC`
  - Response (201): `{"import_id": "imp_987", "total_rows": 45, "staged_rows": 43, "duplicates_discarded": 2, "expires_at": "..."}`
- `POST /api/v1/imports/{import_id}/commit`
  - Response (200): `{"status": "committed", "transactions_created": 43}`
- `GET /api/v1/budgets/current`
  - Response (200): `[{"category_id": 1, "allocated": 15000.0, "spent": 11200.0, "remaining": 3800.0}]`
- `GET /api/v1/goals`
  - Response (200): `[{"id": 1, "name": "Laptop Fund", "target_amount": 90000.0, "saved_amount": 45000.0, "target_date": "2026-12-31"}]`

#### Group D: Pricing & Decision Engine (VIVEK)
- `GET /api/v1/products`
  - Response (200): `[{"id": 1, "title": "Sony WH-1000XM5", "url": "https://...", "current_price": 26990.0, "min_price": 24990.0, "is_demo": false}]`
- `POST /api/v1/products`
  - Request: `{"title": "Sony WH-1000XM5", "url": "https://...", "initial_price": 26990.0}`
  - Response (201): Product object
- `POST /api/v1/products/{id}/prices`
  - Request: `{"price": 25490.0, "observed_at": "2026-10-20T10:00:00Z"}`
  - Response (201): Price observation object
- `POST /api/v1/decisions/evaluate`
  - Request: `{"product_id": 1, "intended_price": 25490.0, "priority_rating": 3}`
  - Response (200):
    ```json
    {
      "verdict": "WAIT",
      "confidence": "HIGH",
      "cooling_off_hours": 48,
      "cooling_off_until": "2026-10-22T10:00:00Z",
      "reasons": [
        "Purchase price exceeds 50% of available discretionary spending money (INR 15,000.00).",
        "Historical lowest price was INR 24,990.00 (saving opportunity)."
      ],
      "rules_triggered": ["RULE_1_AFFORDABILITY", "RULE_3_PRICE_HISTORY"],
      "financial_snapshot": {
        "discretionary_money": 15000.00,
        "price": 25490.00,
        "impact_percent": 169.93
      }
    }
    ```

#### Group E: Overview Dashboard (VIVEK)
- `GET /api/v1/overview`
  - Response (200):
    ```json
    {
      "spending_money": 15000.00,
      "health_score": 82,
      "monthly_income": 80000.00,
      "monthly_expense": 45000.00,
      "active_goals_count": 2,
      "tracked_products_count": 5,
      "pending_cooldowns_count": 1
    }
    ```

---

## 5. Database Schema & Migration Governance

### 5.1 Single Database / Schema Segregation
The application uses a single SQLite database (`finwise.db`) running in WAL mode. Tables are strictly partitioned by module ownership:

```
+----------------------------------------------------------------------------------+
|                            SQLITE DATABASE (finwise.db)                          |
|                                                                                  |
|  [CORE / AUTH (VIVEK)]          [MONEY MODULE (AMAN)]    [PRICING/DECIDE (VIVEK)]|
|  - users                        - accounts               - products              |
|  - user_settings                - categories             - price_observations    |
|  - notifications                - transactions           - decisions             |
|  - audit_log                    - budgets                - cooling_off_periods   |
|                                 - savings_goals                                  |
|                                 - imports                                        |
|                                 - import_staged_rows                             |
+----------------------------------------------------------------------------------+
```

### 5.2 Migration Execution Ownership
- **Exclusive Owner:** **AMAN** owns the `alembic/` migration repository.
- **Workflow:** 
  - To prevent migration branching conflicts, Aman generates the canonical initial schema migration `001_initial_schema.py` covering all tables (Core, Auth, Money, Pricing, Decision) based on the schemas in PRD §7.
  - Vivek **must not run `alembic revision`** on his branch.
  - If Vivek requires a table column addition, he submits an RFC to Aman, who creates the subsequent linear migration (`002_...`).

---

## 6. Environment Configuration Contract (`.env.example`)

Both developers must use this exact `.env.example` template:

```ini
# Environment Mode
ENVIRONMENT=development
DEBUG=True

# Application & Networking
HOST=127.0.0.1
PORT=8000
FRONTEND_ORIGIN=http://localhost:5173

# Database Configuration
DATABASE_URL=sqlite:///./finwise.db
SQLITE_TIMEOUT=15.0

# Cryptography & Security
SECRET_KEY=dev-secret-key-change-in-production-min-32-chars!
FERNET_KEY=Z1J2U3V4X5Y6Z7A8B9C0D1E2F3G4H5I6J7K8L9M0N1O=
FINWISE_JWT_SECRET=dev-jwt-secret-min-32-chars-long-example-value!
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# Deterministic Clock Override (Optional, format: YYYY-MM-DD)
SYSTEM_DATE_OVERRIDE=
```
