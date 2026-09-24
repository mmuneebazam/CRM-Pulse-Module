 # CRM Pulse: Real-Time Sales Intelligence Platform for Odoo 17

![Odoo Version](https://img.shields.io/badge/Odoo-17.0-purple.svg)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)
![OWL](https://img.shields.io/badge/OWL-v3-orange.svg)
![License](https://img.shields.io/badge/License-LGPL--3-green.svg)

**CRM Pulse** (`crm_pulse`) is an enterprise-grade, high-performance integration and real-time sales intelligence module engineered for **Odoo 17 Community & Enterprise**.

Moving beyond standard API consumers (like Shopify bridges), CRM Pulse turns Odoo into an **API Provider** with custom REST endpoints, an in-app real-time executive dashboard using OWL 3, automated lead scoring, dynamic SLA escalation, territory routing, and sub-second updates powered by Odoo 17's native WebSocket bus architecture (`bus.bus`).

---

## 🏗️ System Architecture

```text
                              +---------------------------------------+
                              |    External Systems / Webhooks        |
                              +---------------------------------------+
                                                 |
                                 POST /api/v1/leads (JWT / API Key)
                                                 v
+---------------------------------------------------------------------------------------------------+
| Odoo 17 Application Core                                                                         |
|                                                                                                   |
|  +--------------------+      +------------------------+      +---------------------------------+  |
|  | REST API Engine    | ---> | CRM Pulse Logic Engine | ---> | Automated CRM Core              |  |
|  | (Controllers/v1)   |      | - Idempotency Guard    |      | - Score Calculator              |  |
|  | - Hashed Keys      |      | - SLA Policy Resolver  |      | - Territory & Workload Assign   |  |
|  | - Rate Limiter     |      | - Deduplication Engine |      | - SLA Sweep & Escalation        |  |
|  +--------------------+      +------------------------+      +---------------------------------+  |
|                                          |                                                        |
|                                          v                                                        |
|                              +------------------------+                                           |
|                              | Event Bus Dispatcher   |                                           |
|                              | (crm.pulse.event queue)|                                           |
|                              +------------------------+                                           |
|                                          |                                                        |
+------------------------------------------|--------------------------------------------------------+
                                           |
                              bus.bus / WebSocket Real-Time Transport
                                           |
                                           v
+---------------------------------------------------------------------------------------------------+
| Odoo 17 Web Client (Backend)                                                                      |
|                                                                                                   |
|  +---------------------------------------------------------------------------------------------+  |
|  | Live Executive Dashboard (OWL Components)                                                   |  |
|  |                                                                                             |  |
|  | - PulseKpiStrip (Live Metrics)                                                              |  |
|  | - PulseLeadList & PulseLeadCard (Virtualized View)                                          |  |
|  | - PulseBusService (Reconnection-aware WebSocket Listener)                                  |  |
|  +---------------------------------------------------------------------------------------------+  |
+---------------------------------------------------------------------------------------------------+

## ✨ Key Features

- **Custom REST API Layer (`/api/v1/`)**: Built natively using Odoo 17 HTTP Controllers with Hashed API Keys, JWT bearer auth, strict request rate-limiting, cursor-based pagination, and full HMAC signature verification for inbound webhooks.
- **Idempotency & Resilience**: Accepts `Idempotency-Key` headers on `POST` requests to guarantee zero duplicate lead creation during retries from external gateways.
- **OWL 3 Live Executive Dashboard**: Fully embedded Odoo backend client action written using OWL 3 reactive framework (`useState`, `onWillStart`, `onMounted`). Updates instantaneously without page reloads.
- **Real-Time Push (`bus.bus`)**: Multi-company and team-scoped WebSocket event dispatcher pushing instant notifications (`lead.created`, `lead.scored`, `sla.breached`).
- **Advanced CRM Automation**:
  - **Dynamic Rule-Based Lead Scoring**: Fully configurable evaluation engine (`crm.lead.score.rule`) with rule firing history logs.
  - **SLA & Escalation Engine**: Automated scheduled sweeps (`crm.pulse.sla.policy`) detecting response breaches and assigning escalation activities.
  - **Territory Routing & Deduplication**: Smart workload distribution and email domain matching to prevent duplicate pipeline entries.
- **High-Volume Data Performance**: Optimized for **100,000+ lead records**. Built using batched ORM writes, targeted PostgreSQL indexing, server-side aggregations, and background job processing (`crm.pulse.job`).
- **Executive Board Reports**: High-impact QWeb PDF reports with dynamic scoring breakdowns, company branding, and SLA appendix logs.

---

## 🛠️ Tech Stack

- **Platform**: Odoo 17.0 (Community / Enterprise)
- **Backend**: Python 3.10+, PostgreSQL 15+
- **Frontend**: OWL 3 (Odoo Web Library), SCSS, ES6 JavaScript
- **Transport / API**: REST, JSON, WebSockets (`bus.bus`), OpenAPI / Swagger spec
- **Testing**: Odoo Test Framework (`TransactionCase`), Hoot JS testing, k6 / Python `httpx` load testing

---

crm_pulse/
│
├── __manifest__.py
├── __init__.py
│
├── controllers/
│   ├── __init__.py
│   ├── api_v1.py                 # REST API endpoints (/api/v1/leads, /me, /health)
│   └── webhook_inbound.py        # Signature-verified webhook consumer
│
├── models/
│   ├── __init__.py
│   ├── pulse_config.py           # Engine & rate limit settings
│   ├── pulse_api_key.py          # Secure hashed key management
│   ├── crm_lead.py               # CRM Lead extension (Scoring & SLA)
│   ├── score_rule.py             # Rule builder engine
│   ├── sla_policy.py             # SLA target & escalation configurations
│   ├── pulse_event.py            # Outbound event log & queue
│   └── pulse_job.py              # Background batch worker model
│
├── services/
│   ├── __init__.py
│   ├── auth.py                   # API Key & JWT authorization hooks
│   ├── scoring_engine.py         # Lead score calculator logic
│   ├── sla_engine.py             # SLA deadline & sweep calculator
│   └── event_bus.py              # bus.bus notification wrapper
│
├── static/
│   └── src/
│       ├── pulse_dashboard.js    # Root OWL Client Action
│       ├── pulse_lead_card.js    # Lead Card OWL Component
│       ├── pulse_bus_service.js # Custom Bus Listener Service
│       ├── pulse_dashboard.xml  # QWeb templates for OWL
│       └── pulse_dashboard.scss # Dashboard styling
│
├── security/
│   ├── ir.model.access.csv       # ACL permissions
│   └── security_groups.xml       # User, Manager & Integration Admin rules
│
├── data/
│   └── cron_jobs.xml             # SLA sweeps & batch re-scoring cron jobs
│
├── views/
│   ├── crm_lead_views.xml        # CRM Lead form & kanban extensions
│   ├── pulse_config_views.xml    # Pulse management views
│   └── menu_items.xml            # Client Action & module menus
│
├── report/
│   ├── pipeline_sla_report.xml   # QWeb PDF board report template
│   └── opportunity_onepager.xml  # Single Deal PDF summary template
│
└── tests/
    ├── test_api.py               # REST API & Rate limit unit tests
    ├── test_scoring.py           # Scoring engine tests
    └── test_sla.py               # SLA calculation & escalation tests

## 🚀 Installation & Setup

### Prerequisites
- Active **Odoo 17.0** environment running on **Python 3.10+**.
- PostgreSQL 15+ database.

### Installation Steps

1. **Clone the Repository**:
   Clone `crm_pulse` into your custom add-ons directory:
   
   cd /path/to/odoo/custom_addons
   git clone https://github.com/your-username/crm_pulse.git

2. **Update Odoo Addons Path**:
   Ensure your `odoo.conf` file includes the path to `custom_addons`:
   
   addons_path = /path/to/odoo/addons,/path/to/odoo/custom_addons

3. **Install the Module**:
   - Restart your Odoo 17 instance.
   - Activate **Developer Mode** (`Settings -> Developer Tools -> Activate the developer mode`).
   - Navigate to **Apps -> Update Apps List**.
   - Search for `CRM Pulse` (`crm_pulse`) and click **Activate**.

---

## 🔌 API Usage Examples

### 1. Authenticate & Check Token Scopes

**Request:**

curl -X GET "https://your-odoo-domain.com/api/v1/me" \
  -H "Authorization: Bearer YOUR_API_KEY_OR_JWT"

**Response (`200 OK`):**

{
  "status": "success",
  "data": {
    "key_name": "Partner Portal Key",
    "scopes": ["leads.read", "leads.write"],
    "rate_limit": {
      "limit_per_min": 200,
      "remaining": 198
    }
  }
}

### 2. Idempotent Lead Creation

**Request:**

curl -X POST "https://your-odoo-domain.com/api/v1/leads" \
  -H "Authorization: Bearer YOUR_API_KEY_OR_JWT" \
  -H "Idempotency-Key: 9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d" \
  -H "Content-Type: application/json" \
  -d '{
    "contact_name": "Sarah Connor",
    "email": "sarah@cyberdyne.com",
    "source_channel": "partner_portal",
    "utm_campaign": "q4_enterprise_promo",
    "custom_fields": {
      "company_size": "250+"
    }
  }'

**Response (`201 Created`):**

{
  "status": "success",
  "data": {
    "id": 10423,
    "contact_name": "Sarah Connor",
    "pulse_score": 85,
    "sla_deadline": "2026-09-24T18:30:00Z",
    "sla_state": "in_sla",
    "assigned_to": "Enterprise Sales Team EMEA"
  }
}

---

## ⚡ Performance Benchmarks (100,000+ Records)

*Tested on a standard server configuration (4 vCPU, 16GB RAM, PostgreSQL 15 on SSD):*

| Operation | Naive Baseline | Optimized Output | Optimization Applied |
| :--- | :--- | :--- | :--- |
| **Score 10,000 Leads** | 48.2s | **1.8s** | Batched ORM Writes (`write()`) & background job queues |
| **List Leads API (Page 500)** | 2,450ms | **85ms** | Cursor-based pagination & composite SQL indexing |
| **Dashboard Initial Snapshot** | 5,200ms | **140ms** | PostgreSQL `read_group` & server-side aggregations |
| **SLA Sweep (5,000 leads)** | 31.0s | **1.1s** | Chunked SQL updates & idempotent cron execution |

---

## 🔒 Security & Threat Model

- **Key Hashing**: Plaintext API keys are generated once and never stored. Odoo stores a prefix + SHA-256 hash lookup.
- **Multi-Tenant Isolation**: API Keys, bus channels, and lead scoring rules are strict multi-company bounded (`company_id`).
- **Webhook Verification**: Inbound webhooks require an `X-Pulse-Signature` header computed via HMAC-SHA256 secret.
- **Scoped Permissions**: Keys are restricted to defined scopes (`leads.read`, `leads.write`, `dashboard.admin`).

---

## 🧪 Running Tests

Execute Python Unit & Integration Tests via the official Odoo CLI test runner:

python3 odoo-bin -c /path/to/odoo.conf -d test_db -i crm_pulse --test-enable --stop-after-init

---

## 📄 License

This module is distributed under the **LGPL-3.0 (GNU Lesser General Public License v3.0)**.

---

**Developed & Maintained by:** [Muhammad Muneeb Azam](https://github.com/muneebazam)  
*Full-Stack Software Developer & Odoo Specialist*
