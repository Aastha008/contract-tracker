# Contract Tracker

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Framework](https://img.shields.io/badge/framework-FastAPI-009688.svg)](https://fastapi.tiangolo.com/)
[![Database](https://img.shields.io/badge/database-SQLite%20%7C%20SQLAlchemy-red.svg)](https://www.sqlalchemy.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Code Style](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

A centralized web application and RESTful engine designed to manage vendor subscriptions, service-level agreements (SLAs), and renewal timelines. Contract Tracker delivers real-time visibility into operational spend, contract metadata, and critical cancellation notice windows to eliminate surprise auto-renewals.

---

## Problem Statement & Target Audience

### The Problem
Growing engineering, operations, and finance teams often track software vendors across disparate spreadsheets, corporate credit card statements, and disconnected email threads. This decentralized approach creates severe organizational blind spots:
- **Unplanned Auto-Renewals:** Missing strict 30-, 60-, or 90-day cancellation notice periods locks organizations into unwanted multi-year agreements.
- **Runaway & Redundant Spend:** Duplicate tools across departments remain undetected due to a lack of centralized vendor indexing.
- **Compromised Negotiation Windows:** Procurement and engineering leads lack lead time to negotiate pricing, terms, or seat tiers prior to renewal lock-in.

### Target Audience
- **FinOps & Procurement Teams:** Needing unified visibility into Annual Contract Value (ACV), payment schedules, and renewal commitments.
- **Engineering Managers & Tech Leads:** Overseeing third-party SaaS vendors, developer tooling, and infrastructure licenses.
- **Operations Directors:** Requiring automated notification pipelines to enforce structured procurement governance.

---

## Key Features

- **Interactive Contract Repository:** Web interface powered by Jinja2 and Tailwind CSS featuring live search, categorization (SaaS, Cloud Infrastructure, Tooling), and visual status badges (`Active`, `Pending Renewal`, `Terminated`).
- **Structured Vendor & Contract Intake:** Validated ingestion engine handling vendor profiles, ACV, billing cadence (Monthly, Quarterly, Annual), notice window lead days, and document reference metadata.
- **FastAPI Core & REST Endpoints:** High-performance API routes backed by Pydantic models for strict payload validation, timeline calculations, and full lifecycle CRUD operations.
- **Renewal Alert Engine:** Automated business logic that calculates effective cancellation cut-off dates against active renewal intervals and triggers webhook or email alerts before notice windows expire.

---

## Architecture & System Design

Contract Tracker adopts a modular layer architecture separating presentation, business logic, data models, and database access.

```
+------------------------------------------------------------------+
|                    Client Layer                                  |
|  - Web UI (Jinja2 + Tailwind CSS)      - REST API Consumers      |
+---------------------------------+--------------------------------+
                                  |
                                  v
+------------------------------------------------------------------+
|                   FastAPI Routing Layer (src/api.py)            |
|  - Web View Renderers                  - JSON API Endpoints      |
|  - Request Body Validation             - HTTP Exception Handling |
+---------------------------------+--------------------------------+
                                  |
                                  v
+------------------------------------------------------------------+
|                Core Business Engine (src/core.py)                |
|  - Renewal Date & Notice Calculations  - Status State Evaluator  |
|  - Alert Dispatchers (Webhook/Email)   - Filtering & Search Ops  |
+---------------------------------+--------------------------------+
                                  |
                                  v
+------------------------------------------------------------------+
|                Data Layer (src/models.py)                        |
|  - Pydantic Ingestion Schemas          - SQLAlchemy ORM Entities |
|  - SQLite Storage Engine Engine        - Transaction Lifecycles  |
+------------------------------------------------------------------+
```

### Module Responsibilities

- **`src/models.py`**: Defines SQLAlchemy ORM entities (`Vendor`, `Contract`) alongside corresponding Pydantic schemas for data validation, serialization, and deserialization.
- **`src/core.py`**: Implements domain business rules, specifically computing notice cut-off deadlines (`renewal_date - notice_period_days`), resolving contract lifecycle states, and executing scheduled notification sweeps.
- **`src/api.py`**: Serves UI templates, exposes RESTful CRUD endpoints, coordinates data mapping between schemas and storage, and handles HTTP errors.
- **`src/main.py`**: Handles runtime lifecycle setup, database initialization, and application launch configuration.

---

## Technology Stack

- **Backend Framework:** FastAPI (ASGI Python web framework)
- **Data Validation:** Pydantic v2
- **ORM & Storage:** SQLAlchemy 2.0 with SQLite
- **Template Rendering:** Jinja2
- **Styling:** Tailwind CSS (CDN-delivered UI utility styling)
- **ASGI Server:** Uvicorn
- **Testing Suite:** pytest

---

## Installation & Virtual Environment Setup

### Prerequisites
- Python 3.10 or higher
- Git

### Step-by-Step Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-org/contract-tracker.git
   cd contract-tracker
   ```

2. **Create and activate a dedicated virtual environment:**
   ```bash
   # Linux/macOS
   python3 -m venv .venv
   source .venv/bin/activate

   # Windows (PowerShell)
   python -m venv .venv
   .venv\Scripts\Activate.ps1
   ```

3. **Install dependencies:**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

---

## Environment Variables Configuration

Copy the example environment configuration to create your local `.env` file:

```bash
cp .env.example .env
```

### Configuration Options

| Variable | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `APP_ENV` | String | `development` | Deployment environment (`development`, `production`, `test`). |
| `DATABASE_URL` | String | `sqlite:///./contracts.db` | SQLAlchemy connection string. |
| `SERVER_HOST` | String | `127.0.0.1` | Local bind address for the Uvicorn server. |
| `SERVER_PORT` | Integer | `8000` | Port for the web application. |
| `ALERT_NOTICE_THRESHOLD_DAYS`| Integer | `30` | Default alert horizon to flag upcoming notice dates. |
| `NOTIFICATION_WEBHOOK_URL` | String | `""` | Destination webhook URL for automated renewal alerts. |

---

## Usage: CLI Commands & API Endpoints

### Running the Application

Start the web server locally using the application runner:

```bash
python -m src.main
```

Alternatively, use `uvicorn` directly with auto-reload enabled:

```bash
uvicorn src.main:app --host 127.0.0.1 --port 8000 --reload
```

Once running:
- **Interactive UI Dashboard:** Open `http://127.0.0.1:8000/` in your browser.
- **Interactive Swagger Documentation:** Open `http://127.0.0.1:8000/docs`.
- **ReDoc Alternative:** Open `http://127.0.0.1:8000/redoc`.

---

### REST API Documentation & Examples

#### 1. Ingest a New Contract

`POST /api/v1/contracts`

**Request:**
```bash
curl -X POST "http://127.0.0.1:8000/api/v1/contracts" \
     -H "Content-Type: application/json" \
     -d '{
       "vendor_name": "Datadog",
       "category": "Cloud Infrastructure",
       "annual_contract_value": 36000.00,
       "billing_cycle": "Annual",
       "start_date": "2024-01-01",
       "end_date": "2025-01-01",
       "notice_period_days": 30,
       "auto_renew": true,
       "owner_email": "ops-lead@company.internal"
     }'
```

**Response (`201 Created`):**
```json
{
  "id": 1,
  "vendor_name": "Datadog",
  "category": "Cloud Infrastructure",
  "annual_contract_value": 36000.00,
  "billing_cycle": "Annual",
  "start_date": "2024-01-01",
  "end_date": "2025-01-01",
  "notice_period_days": 30,
  "notice_deadline": "2024-12-02",
  "auto_renew": true,
  "status": "Active",
  "owner_email": "ops-lead@company.internal"
}
```

#### 2. Query Contracts by Status or Category

`GET /api/v1/contracts?status=Active&category=Cloud%20Infrastructure`

**Request:**
```bash
curl -X GET "http://127.0.0.1:8000/api/v1/contracts?status=Active"
```

#### 3. Run Renewal Notification Checks

`POST /api/v1/contracts/alerts/trigger`

Triggers notification evaluation across all active contracts and dispatches payloads for agreements with approaching cancellation deadlines.

**Request:**
```bash
curl -X POST "http://127.0.0.1:8000/api/v1/contracts/alerts/trigger"
```

---

## Automated Testing Instructions

The suite uses `pytest` with a decoupled in-memory SQLite database instance to validate business logic and API endpoints.

### Execute All Tests
```bash
pytest
```

### Run Tests with Verbose Output and Coverage Summary
```bash
pytest -v --tb=short
```

### Run Specific Test Suites
```bash
# Execute core business logic tests (deadline calculations, alert boundaries)
pytest tests/test_core.py -v

# Execute REST API endpoint tests (FastAPI TestClient CRUD operations)
pytest tests/test_api.py -v
```

---

## Project Structure

```
contract-tracker/
├── .env.example              # Sample environment configuration template
├── .gitignore                # Git ignore configuration
├── requirements.txt          # Python dependencies definition
├── src/
│   ├── __init__.py           # Package marker
│   ├── api.py                # FastAPI routes, HTML UI handlers, and API exceptions
│   ├── core.py               # Notice deadline calculations, state engine, notification rules
│   ├── main.py               # Uvicorn entry point and application lifespan setup
│   └── models.py             # SQLAlchemy models and Pydantic validation schemas
└── tests/
    ├── __init__.py           # Test package marker
    ├── test_api.py           # Endpoint integration tests via FastAPI TestClient
    └── test_core.py          # Unit tests for notice windows, statuses, and core rules
```

---

## Roadmap & Future Enhancements

- [ ] **Document Parsing Engine:** PDF/OCR parsing for automated extraction of cancellation clauses from uploaded vendor agreements.
- [ ] **Multi-Channel Integrations:** First-class Slack and Microsoft Teams notification apps to alert operational channels on upcoming notice windows.
- [ ] **Role-Based Access Control (RBAC):** Support for granular read-only and administrative roles integrated with Okta / Google Workspace SAML.
- [ ] **Spend Analytics Visualizations:** Department-level trend charts and budget burn rate projections.

---

## License

This project is licensed under the terms of the [MIT License](LICENSE).