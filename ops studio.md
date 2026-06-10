# Deployment Studio - Comprehensive Technical Documentation

**Version:** 1.0.0  
**Last Updated:** June 2026  
**Project Name:** deployment_studio_dev (Ops Studio)

---

## Table of Contents

1. [Executive Overview](#executive-overview)
2. [High-Level End-to-End Workflow](#high-level-end-to-end-workflow)
3. [Application Architecture](#application-architecture)
4. [Technology Stack](#technology-stack)
5. [System Integration & Observability](#system-integration--observability)
6. [Component Deep Dive](#component-deep-dive)
7. [Data Flow & Request Lifecycle](#data-flow--request-lifecycle)
8. [Deployment Architecture](#deployment-architecture)
9. [Security & Authentication](#security--authentication)
10. [API Reference](#api-reference)

---

## Executive Overview

**Deployment Studio** is an enterprise-grade observability and multi-cloud infrastructure management platform designed to monitor, analyze, and optimize AI agent deployments, workflows, and infrastructure across Azure cloud environments. It provides real-time insights into application performance, cost analytics, and agent execution metrics.

### Key Capabilities

- **Multi-Cloud Observability**: Monitor applications across Azure with deep integration into Azure Monitor, Cost Management, and Log Analytics
- **AI Agent Monitoring**: Track AI agent execution, model usage, token consumption, and performance metrics
- **Workflow Management**: Define, execute, and monitor complex workflows with agent orchestration
- **Cost Analytics**: Real-time cost tracking by resource, model, and application
- **Unified Dashboard**: Centralized monitoring interface for all infrastructure and application metrics
- **Role-Based Access Control**: Enterprise-grade authentication via Azure AD/Entra ID with session management

---

## High-Level End-to-End Workflow

### User Journey

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DEPLOYMENT STUDIO USER FLOW                         │
└─────────────────────────────────────────────────────────────────────────────┘

1. USER AUTHENTICATION
   ├─ User navigates to login page
   ├─ Browser redirected to Azure AD OAuth2 endpoint
   ├─ User authenticates with Azure credentials
   ├─ Authorization code exchanged for JWT token
   ├─ Token stored in secure local storage
   └─ Session created in database with expiry tracking

2. DASHBOARD INITIALIZATION
   ├─ Frontend loads React application
   ├─ AppContext initialized with auth state
   ├─ JWT token verified against session in database
   ├─ User metadata loaded (role, preferences, currency type)
   ├─ API interceptor configured with Bearer token
   └─ Dashboard rendered with user's accessible applications

3. INFRASTRUCTURE DISCOVERY
   ├─ User selects/creates application
   ├─ System queries Azure Resource Graph for resources
   ├─ Discovered resources registered in database
   ├─ Agent environments configured (Agno, Strands, MAF)
   ├─ Log Analytics workspaces linked
   └─ Cost tracking initialized

4. MONITORING & METRICS COLLECTION
   ├─ System polls Azure Monitor APIs
   ├─ Metrics collected from:
   │  ├─ Azure App Service Plans (CPU, memory, requests)
   │  ├─ Azure Kubernetes Service (pod metrics, node health)
   │  ├─ Databases (performance, connections)
   │  └─ Cost Management API (resource costs)
   ├─ AI agent metrics aggregated:
   │  ├─ Token usage per model
   │  ├─ Invocation counts and latencies
   │  ├─ Error rates and traces
   │  └─ Workflow execution status
   └─ Metrics stored in PostgreSQL

5. DASHBOARD VISUALIZATION
   ├─ Frontend retrieves paginated metrics from backend
   ├─ Charts rendered (pie, line, bar charts via Nivo/ApexCharts)
   ├─ Real-time updates via periodic API polling
   ├─ Cost analytics displayed with currency conversion
   ├─ Agent performance matrices shown
   └─ Workflow topology visualized with @xyflow/react

6. ALERTING & ANALYSIS
   ├─ User examines trends and anomalies
   ├─ Cost breakdowns analyzed by resource type
   ├─ Agent performance compared across models
   ├─ Workflow execution paths traced
   └─ ChatBot provides intelligent insights (optional)

7. REPORTING & EXPORT
   ├─ User generates reports on demand
   ├─ Data exported to requested format
   ├─ Historical trends analyzed
   └─ Compliance audits performed
```

### Metrics Collection Pipeline

```
┌──────────────┐      ┌─────────────────┐      ┌──────────────┐      ┌────────────┐
│   Azure      │      │   FastAPI       │      │ PostgreSQL   │      │  React     │
│   Services   │─────▶│   Backend       │─────▶│  Database    │─────▶│ Frontend   │
│              │      │                 │      │              │      │            │
│ - Monitor    │      │ - Aggregation   │      │ - Metrics    │      │ - Charts   │
│ - Cost Mgmt  │      │ - Filtering     │      │ - Sessions   │      │ - Tables   │
│ - Kusto      │      │ - Pagination    │      │ - Users      │      │ - Alerts   │
│ - Resources  │      │ - Auth/Filter   │      │ - Apps       │      │ - Insights │
└──────────────┘      └─────────────────┘      └──────────────┘      └────────────┘
       ▲                                                                       │
       │                                                                       │
       └───────────────────── User Actions & Queries ────────────────────────┘
```

---

## Application Architecture

### High-Level System Design

```
┌────────────────────────────────────────────────────────────────────────────┐
│                        DEPLOYMENT STUDIO ARCHITECTURE                      │
└────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                          PRESENTATION LAYER                             │
│  ┌────────────────────────────────────────────────────────────────┐    │
│  │  React 19.2 Frontend (Vite Build)                             │    │
│  │                                                                │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │    │
│  │  │Dashboard │  │   Apps   │  │Workflows │  │ Platform │     │    │
│  │  │  Page    │  │  Page    │  │  Page    │  │  Page    │     │    │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │    │
│  │                                                                │    │
│  │  ┌──────────────────────────────────────────────────────┐    │    │
│  │  │   Reusable Components                               │    │    │
│  │  │   - Navigation, ChatBot, Alerts, SSO Loader        │    │    │
│  │  │   - Chart Components (Pie, Line, Topology)         │    │    │
│  │  └──────────────────────────────────────────────────────┘    │    │
│  │                                                                │    │
│  │  ┌────────────┐  ┌──────────────────┐  ┌──────────────┐    │    │
│  │  │ AppContext │  │ React Router v7  │  │ Secure Store │    │    │
│  │  │ State Mgmt │  │ Routing          │  │ JWT + Data   │    │    │
│  │  └────────────┘  └──────────────────┘  └──────────────┘    │    │
│  └──────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┐
                                 ▲                                     │
                                 │ HTTPS / REST API                   │
                                 ▼                                     │
┌─────────────────────────────────────────────────────────────────────┤
│                        APPLICATION LAYER                            │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │  FastAPI 0.118 Backend (Python 3.12)                      │    │
│  │                                                            │    │
│  │  ┌─────────────────────────────────────────────────────┐  │    │
│  │  │ API Routers (Endpoints)                           │  │    │
│  │  │                                                   │  │    │
│  │  │ ├─ auth_router         → Auth & Session Mgmt    │  │    │
│  │  │ ├─ agent_router        → AI Agent Metrics      │  │    │
│  │  │ ├─ llm_router          → LLM & Model Metrics   │  │    │
│  │  │ ├─ workflow_router     → Workflow Execution    │  │    │
│  │  │ ├─ cost_metrics_router → Cost Analytics        │  │    │
│  │  │ ├─ database_router     → Database Metrics      │  │    │
│  │  │ ├─ custom_infra_router → Infrastructure CRUD   │  │    │
│  │  │ ├─ app_service_router  → App Service Metrics   │  │    │
│  │  │ └─ aks_router          → AKS Cluster Metrics   │  │    │
│  │  └─────────────────────────────────────────────────┘  │    │
│  │                                                        │    │
│  │  ┌─────────────────────────────────────────────────┐  │    │
│  │  │ Middleware                                      │  │    │
│  │  │ ├─ CORS (Cross-Origin Resource Sharing)       │  │    │
│  │  │ ├─ Security Headers (CSP, HSTS, X-Frame-Opt)  │  │    │
│  │  │ ├─ JWT Bearer Token Validation                │  │    │
│  │  │ ├─ Request Logging & Tracing                  │  │    │
│  │  │ └─ Error Handling                             │  │    │
│  │  └─────────────────────────────────────────────────┘  │    │
│  │                                                        │    │
│  │  ┌─────────────────────────────────────────────────┐  │    │
│  │  │ Business Logic (Modules)                        │  │    │
│  │  │ ├─ metrics.py          → Calculation Engine   │  │    │
│  │  │ ├─ workflow_metrics.py → Workflow Analysis    │  │    │
│  │  │ ├─ azure_* modules     → Azure Integration    │  │    │
│  │  │ ├─ jwt_auth.py         → Token Management     │  │    │
│  │  │ └─ db.py               → Database Interface   │  │    │
│  │  └─────────────────────────────────────────────────┘  │    │
│  │                                                        │    │
│  │  ┌─────────────────────────────────────────────────┐  │    │
│  │  │ Models (Pydantic)                              │  │    │
│  │  │ ├─ Request validators                          │  │    │
│  │  │ ├─ Response schemas                            │  │    │
│  │  │ └─ Database ORM models                         │  │    │
│  │  └─────────────────────────────────────────────────┘  │    │
│  │                                                        │    │
│  │  ┌─────────────────────────────────────────────────┐  │    │
│  │  │ Utilities                                       │  │    │
│  │  │ ├─ jwt_auth.py  → JWT/Token operations        │  │    │
│  │  │ ├─ logger.py    → Structured logging (Loguru) │  │    │
│  │  │ ├─ config.py    → YAML config management      │  │    │
│  │  │ └─ azure_creds  → Azure credential handling   │  │    │
│  │  └─────────────────────────────────────────────────┘  │    │
│  │                                                        │    │
│  │  ┌─────────────────────────────────────────────────┐  │    │
│  │  │ External Service Integrations                   │  │    │
│  │  │ ├─ Azure SDKs (Monitor, Cost, Resources)      │  │    │
│  │  │ ├─ Azure AD/Entra ID (MSAL)                   │  │    │
│  │  │ ├─ OpenTelemetry (Distributed Tracing)        │  │    │
│  │  │ └─ Azure Storage & Kusto (ADX)                │  │    │
│  │  └─────────────────────────────────────────────────┘  │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┤
                          ▲           ▲           ▲                │
                          │           │           │                │
                   SQL Protocol      Azure      Secrets             │
                          │           │           │                │
┌─────────────────────────┼───────────┼───────────┼──────────────┐ │
│                         ▼           ▼           ▼              │ │
│                    DATA LAYER                                   │ │
│  ┌──────────────────────────────────────────────────────────┐ │ │
│  │  PostgreSQL Database (Pooled Connections)               │ │ │
│  │  ├─ sessions           → User sessions & tokens          │ │ │
│  │  ├─ users              → User profiles & preferences     │ │ │
│  │  ├─ applications       → Application configurations      │ │ │
│  │  ├─ agents             → AI agent definitions            │ │ │
│  │  ├─ workflows          → Workflow definitions & runs     │ │ │
│  │  ├─ azureresources     → Azure resource registry        │ │ │
│  │  ├─ costmetrics        → Cost tracking data             │ │ │
│  │  ├─ appmetrics         → Application metrics            │ │ │
│  │  └─ agentenvironment   → Framework definitions          │ │ │
│  └──────────────────────────────────────────────────────────┘ │ │
└────────────────────────────────────────────────────────────────┘ │
                                                                    │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  EXTERNAL SERVICES                                       │   │
│  │                                                          │   │
│  │  Azure Cloud:                                           │   │
│  │  ├─ Azure Monitor (Metrics & Logs)                     │   │
│  │  ├─ Azure Cost Management API                          │   │
│  │  ├─ Azure Resource Graph (Inventory)                   │   │
│  │  ├─ Azure Kusto (ADX) (Log Analytics)                  │   │
│  │  ├─ Azure Storage (Blob, Queue)                        │   │
│  │  ├─ Azure App Service Plans                            │   │
│  │  ├─ Azure Kubernetes Service (AKS)                     │   │
│  │  └─ Azure AD/Entra ID (Authentication)                 │   │
│  │                                                          │   │
│  │  Other Services:                                        │   │
│  │  ├─ HashiCorp Vault (Secret Management)                │   │
│  │  ├─ Kubernetes Cluster (Deployment)                    │   │
│  │  └─ Container Registry (Image Storage)                 │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### Component Interaction Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    COMPONENT INTERACTION FLOW                           │
└─────────────────────────────────────────────────────────────────────────┘

Client Browser                Backend Service              External Services
      │                              │                             │
      ├──── 1. Login Request ───────▶│                             │
      │                              ├─── 2. OAuth2 Flow ────────▶│
      │                              │    (Azure AD)              │
      │◀─── 3. Auth Redirect ────────┤◀── 4. ID Token ───────────┤
      │                              │                            │
      │                              ├──── 5. Create Session ────▶│
      │                              │    (PostgreSQL)            │
      │◀─── 6. Access Token ────────┤                            │
      │      (JWT + Session ID)      │                            │
      │                              │                            │
      ├──── 7. GET /api/applications │                            │
      │      (Bearer Token) ────────▶│                            │
      │                              ├─── 8. Query Resources ────▶│
      │                              │   (Azure Resource Graph)   │
      │                              │                            │
      │                              ├─── 9. Fetch Metrics ──────▶│
      │                              │    (Azure Monitor)         │
      │                              │                            │
      │                              ├─── 10. Query Costs ───────▶│
      │                              │  (Cost Management API)     │
      │                              │                            │
      │                              ├─── 11. Aggregate & Cache ─▶│
      │                              │    (PostgreSQL)            │
      │◀────12. JSON Response ───────┤                            │
      │    (Metrics Data)            │                            │
      │                              │                            │
      ├─── 13. POST /api/workflows   │                            │
      │        (Trigger) ──────────▶│                            │
      │                              ├─── 14. Execute Workflow ──▶│
      │                              │   (Agent Framework)        │
      │                              │                            │
      │                              ├─── 15. Track Execution ───▶│
      │                              │    (PostgreSQL)            │
      │◀────16. Workflow Status ─────┤                            │
      │                              │                            │
      └──────────────────────────────┴────────────────────────────┘
```

---

## Technology Stack

### Frontend Stack

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Framework** | React | 19.2.0 | UI component framework |
| **Build Tool** | Vite | 5.4.11 | Fast module bundler |
| **Routing** | React Router DOM | 7.13.0 | Client-side routing |
| **UI Libraries** | Material-UI (MUI) | 7.3.8 | Component library |
| | Ant Design | 6.3.0 | Enterprise UI components |
| | Lucide React | 1.16.0 | Icon library |
| **Charting** | Nivo | 0.99.0 | Line, pie, bar charts |
| | ApexCharts | 5.4.0 | Advanced charts |
| **Animation** | Framer Motion | 12.34.0 | Motion graphics |
| **Visualization** | @xyflow/react | 12.10.2 | Topology/DAG visualization |
| | Dagre | 0.8.5 | Graph layout engine |
| **HTTP Client** | Axios | 1.13.5 | REST API calls |
| **Security** | JWT-Decode | 4.0.0 | Token decoding |
| | react-secure-storage | 1.3.2 | Encrypted local storage |
| **Styling** | Tailwind CSS | 3.4.0 | Utility-first CSS |
| | PostCSS | 8.4.38 | CSS transformation |
| **Linting** | ESLint | 9.39.1 | Code quality |

### Backend Stack

| Category | Technology | Version | Purpose |
|----------|-----------|---------|---------|
| **Framework** | FastAPI | 0.118.0 | Modern async web framework |
| **ASGI Server** | Uvicorn | 0.37.0 | ASGI application server |
| **Process Manager** | Gunicorn | 23.0.0 | Production WSGI HTTP server |
| **Database** | PostgreSQL | 13+ | Relational database |
| **ORM** | SQLAlchemy | 2.0.43 | Python SQL toolkit |
| **Async DB Driver** | asyncpg | 0.30.0 | PostgreSQL async driver |
| **DB Connection Pool** | databases | 0.9.0 | Async database API |
| **Authentication** | MSAL (Microsoft Auth Library) | 1.34.0 | Azure AD/Entra ID |
| | python-jose | 3.5.0 | JWT handling |
| | PyJWT | 2.10.1 | JWT encoding/decoding |
| | cryptography | 46.0.1 | Encryption (AES-GCM, Fernet) |
| **Data Processing** | Pandas | 2.3.3 | Data manipulation |
| | NumPy | 2.4.1 | Numerical computing |
| **Azure SDKs** | azure-identity | 1.25.0 | Azure authentication |
| | azure-mgmt-web | 10.0.0 | App Service management |
| | azure-mgmt-monitor | 7.0.0 | Azure Monitor API |
| | azure-mgmt-costmanagement | 4.0.1 | Cost Management API |
| | azure-mgmt-rdbms | 10.1.1 | Database management |
| | azure-kusto-ingest | 6.0.1 | Log Analytics ingestion |
| | azure-storage-blob | 12.26.0 | Blob storage |
| | azure-monitor-opentelemetry | 1.8.3 | Observability |
| **Configuration** | python-dotenv | 1.1.1 | Environment variables |
| | PyYAML | 6.0.3 | Configuration files |
| **Logging** | loguru | 0.7.3 | Structured logging |
| **Task Scheduling** | APScheduler | 3.11.0 | Job scheduling |
| **Retry Logic** | Tenacity | 9.1.2 | Retry decorator |
| **OpenTelemetry** | opentelemetry-sdk | 1.39.0 | Distributed tracing |
| | opentelemetry-instrumentation-fastapi | 0.60b0 | FastAPI instrumentation |
| **Python Version** | Python | 3.12 | Runtime |

### Infrastructure Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Containerization** | Docker | Container image building and running |
| **Container Registry** | Azure Container Registry (ACR) | Image storage and management |
| **Orchestration** | Kubernetes (AKS) | Container orchestration |
| **Secrets Management** | HashiCorp Vault | Centralized secret storage |
| **Service Account** | Kubernetes SA | Pod authentication to Vault |
| **Init Containers** | Custom Shell Script | Secret injection at pod startup |
| **Persistent Storage** | Kubernetes PersistentVolume | RSA key storage |
| **Namespace** | ds-test | Logical isolation in cluster |

---

## System Integration & Observability

### Azure Integrations

#### 1. **Authentication (Azure AD / Entra ID)**
- **Protocol**: OAuth 2.0 with OpenID Connect
- **Library**: MSAL (Microsoft Authentication Library)
- **Flow**: Authorization Code Flow with PKCE
- **Endpoints**:
  - Login: `GET /api/auth/login` → Redirects to Azure AD
  - Callback: `GET /api/auth/redirect` → Processes auth code
  - Token Refresh: `POST /api/auth/token/refresh`
  - Logout: `GET /api/auth/logout` → Invalidates session

#### 2. **Resource Discovery (Azure Resource Graph)**
```python
# Query: List all resources for a subscription
# Returns: Applications, VMs, Databases, Container registries, etc.
# Used for: Infrastructure mapping and cost allocation
```

#### 3. **Monitoring Metrics (Azure Monitor)**
- **App Service Metrics**:
  - CPU Percentage, Memory Percentage
  - Request Count, Response Time
  - Failed Request Count
  
- **AKS Cluster Metrics**:
  - Pod CPU/Memory Usage
  - Node Health Status
  - Network I/O Bytes
  
- **Database Metrics**:
  - Active Connections
  - CPU Percentage
  - Failed Connections
  - Replication Lag

#### 4. **Cost Analytics (Cost Management API)**
- **Real-time Cost Tracking**: By resource type, service, time period
- **Currency Conversion**: Multi-currency support
- **Cost Forecasting**: Trend analysis and predictions
- **Allocation**: Cost attribution per application

#### 5. **Log Analytics (Azure Kusto/ADX)**
- **Query**: KQL (Kusto Query Language)
- **Purpose**: Agent execution logs, error traces, performance telemetry
- **Retention**: Configurable (typically 30-90 days)

#### 6. **Storage & Blob**
- **Purpose**: Configuration files, export data, snapshots
- **Access**: Managed identity or connection string

### Agent Framework Integrations

The system supports multiple AI agent frameworks:

**1. Agno Framework**
- Agent definitions and configurations
- Environment setup and dependencies
- Execution environment isolation

**2. Strands Agents**
- Agent metadata and capabilities
- Execution context and state management
- Integration with workflow engine

**3. Microsoft Agentic Framework (MAF)**
- Tool definitions and schemas
- Agent orchestration
- Multi-step reasoning support

### Observability Stack

#### Distributed Tracing (OpenTelemetry)
```
Application Code
    ↓
OpenTelemetry Instrumentation
    ├─ FastAPI automatic instrumentation
    ├─ Database queries traced
    ├─ HTTP calls instrumented
    └─ Custom spans for business logic
    ↓
Azure Monitor OpenTelemetry Exporter
    ↓
Application Insights (Azure)
    ├─ Trace visualization
    ├─ Performance bottleneck identification
    └─ End-to-end request flow analysis
```

#### Structured Logging
- **Library**: Loguru (structured JSON logging)
- **Levels**: DEBUG, INFO, WARNING, ERROR, CRITICAL
- **Correlation ID**: Tracks requests across services
- **Context**: User ID, application ID, session ID

#### Metrics Collection Pipeline
```
Azure Services (every 60-300 seconds)
    ↓
FastAPI Backend (aggregation & caching)
    ↓
PostgreSQL (persistent storage)
    ↓
React Frontend (visualization)
    ↓
User Dashboard (charts & tables)
```

---

## Component Deep Dive

### Frontend Components

#### 1. **Pages**

**ApplicationDetails Page**
- Displays infrastructure and application metrics
- Sub-components:
  - Agent-wise metrics and performance
  - Model-wise metrics and token usage
  - Functional analytics with time range filtering
  - Status cards showing health indicators

**Dashboard Page**
- Overview of all monitored applications
- Key performance indicators (KPIs)
- Cost trends and alerts
- Agent activity summary

**Configuration Page**
- System and user settings
- Currency preferences
- Application management
- Infrastructure configuration

**LoginPage**
- SSO login form
- Redirect to Azure AD
- Token handling post-login

**PlatformsPage**
- Multi-cloud infrastructure view
- Cloud provider selection
- Resource inventory

#### 2. **Reusable Components**

**Navigation Component**
- Header and sidebar navigation
- User profile menu
- Logout functionality
- Active route highlighting

**ChatBot Component**
- Real-time conversation interface
- Typing indicators
- Message history
- Context-aware responses

**AlertComponent**
- Toast notifications
- Success/error/warning messages
- Auto-dismiss functionality

**Chart Components**
- PieChartGraph (resource distribution)
- AnalyticsGraph (time-series trends)
- AnalyticsTable (data table display)
- ModelPiechart, AgentPiechart (model/agent breakdown)

### Backend Routers

#### 1. **Authentication Router** (`auth_router`)
```
Endpoints:
├─ GET /api/auth/login
│  └─ Initiates OAuth2 flow with Azure AD
│
├─ GET /api/auth/redirect
│  └─ Handles OAuth callback, exchanges auth code for tokens
│
├─ POST /api/auth/token/refresh
│  └─ Refreshes expired JWT tokens
│
├─ GET /api/auth/logout
│  └─ Invalidates session and clears tokens
│
└─ PUT /api/auth/update-currency
   └─ Updates user currency preference
```

**Database Tables Used**:
- `sessions` (session tracking, expiry)
- `users` (user profiles, preferences)

#### 2. **Agent Metrics Router** (`agent_router`)
```
Endpoints:
├─ POST /api/agent/agent-wise-metrics-paginated
│  └─ Returns paginated agent metrics with sorting/filtering
│
├─ POST /api/agent/model-wise-metrics-paginated
│  └─ Model performance and usage metrics
│
├─ POST /api/agent/user-wise-metrics-paginated
│  └─ User-level agent usage statistics
│
├─ POST /api/agent/dashboard-metrics
│  └─ KPIs for dashboard visualization
│
└─ POST /api/agent/token-metrics
   └─ Token usage per model/agent
```

**Query Parameters**:
- `app_id`: Application identifier
- `page_number`, `page_size`: Pagination
- `search_filter`: Text search
- `start_date`, `end_date` OR `duration_hours`: Time range
- `sort_column`, `sort_order`: Sorting

#### 3. **Workflow Router** (`workflow_router`)
```
Endpoints:
├─ POST /api/workflow/workflow-wise-metrics-paginated
│  └─ Workflow execution metrics
│
├─ POST /api/workflow/workflow-runs
│  └─ Individual workflow run details
│
├─ POST /api/workflow/workflow-topology
│  └─ Workflow DAG and agent dependencies
│
└─ POST /api/workflow/agent-workflows
   └─ Agents within workflows
```

#### 4. **Cost Metrics Router** (`cost_metrics_router`)
```
Endpoints:
├─ POST /api/cost/overall-costs
│  └─ Total cost by time period
│
├─ POST /api/cost/model-wise-costs
│  └─ Cost breakdown by LLM model
│
└─ POST /api/cost/resource-wise-costs
   └─ Cost by Azure resource type
```

#### 5. **Database Router** (`database_router`)
```
Endpoints:
├─ GET /api/database/clusters
│  └─ PostgreSQL cluster information
│
└─ GET /api/database/summary
   └─ Database health and metrics
```

#### 6. **App Service Router** (`app_service_router`)
```
Endpoints:
├─ GET /api/appservice/metrics
│  └─ App Service plan metrics
│
├─ POST /api/appservice/paginated
│  └─ Paginated app service listings
│
└─ GET /api/appservice/summary
   └─ App Service overview
```

#### 7. **AKS Router** (`aks_router`)
```
Endpoints:
├─ POST /api/aks/cluster-metrics
│  └─ AKS cluster health and performance
│
├─ POST /api/aks/node-metrics
│  └─ Individual node metrics
│
└─ POST /api/aks/pod-metrics
   └─ Pod resource usage
```

#### 8. **Custom Infrastructure Router** (`custom_infra_router`)
```
Endpoints:
├─ POST /api/custominfra/applications
│  └─ CRUD operations on applications
│
├─ GET /api/custominfra/applications/{app_id}
│  └─ Application details
│
├─ POST /api/custominfra/resources
│  └─ Resource management
│
└─ GET /api/custominfra/summary
   └─ Infrastructure overview
```

### Database Schema

#### Key Tables

**sessions**
```sql
id (UUID, PK)
user_id (UUID, FK)
session_id (VARCHAR, UNIQUE)
created_at (TIMESTAMP)
expiry_time (TIMESTAMP)
status (ENUM: active, inactive)
token (TEXT)
```

**users**
```sql
id (UUID, PK)
email (VARCHAR, UNIQUE)
name (VARCHAR)
role (VARCHAR, DEFAULT: user)
currency_type (VARCHAR, DEFAULT: USD)
created_at (TIMESTAMP)
updated_at (TIMESTAMP)
```

**applications**
```sql
id (UUID, PK)
name (VARCHAR)
app_id (VARCHAR, UNIQUE)
environment (VARCHAR)
status (ENUM: healthy, unhealthy, unknown)
created_by (UUID, FK)
created_at (TIMESTAMP)
updated_at (TIMESTAMP)
```

**agents**
```sql
id (UUID, PK)
name (VARCHAR)
framework (VARCHAR: agno, strands, maf)
app_id (UUID, FK)
status (VARCHAR)
created_at (TIMESTAMP)
```

**workflows**
```sql
id (UUID, PK)
name (VARCHAR)
app_id (UUID, FK)
definition (JSONB)
status (VARCHAR)
created_at (TIMESTAMP)
last_executed (TIMESTAMP)
```

**costmetrics**
```sql
id (UUID, PK)
app_id (UUID, FK)
resource_type (VARCHAR)
cost (DECIMAL)
currency (VARCHAR)
period_start (TIMESTAMP)
period_end (TIMESTAMP)
```

**agentenvironment**
```sql
id (UUID, PK)
name (VARCHAR: agno, strands, maf)
configuration (JSONB)
dependencies (JSONB)
created_at (TIMESTAMP)
```

---

## Data Flow & Request Lifecycle

### Example: Fetching Agent Metrics

```
1. USER INITIATES REQUEST (Frontend)
   ├─ User navigates to Applications > App Details > Agent-Wise Metrics
   ├─ Frontend creates request payload:
   │  {
   │    "app_id": "app-12345",
   │    "page_number": 1,
   │    "page_size": 10,
   │    "start_date": "2026-06-01",
   │    "end_date": "2026-06-10"
   │  }
   └─ Axios sends: POST /api/agent/agent-wise-metrics-paginated

2. REQUEST REACHES BACKEND
   ├─ FastAPI receives request
   ├─ CORS middleware validates origin
   ├─ JWT Bearer token validation
   │  ├─ Extract token from Authorization header
   │  ├─ Verify signature using public key (RS256)
   │  ├─ Check token expiry
   │  └─ Query sessions table for session status
   ├─ Authenticate JWTBearer dependency
   └─ Route to agent_router handler

3. BUSINESS LOGIC EXECUTION
   ├─ Parse and validate request (Pydantic model)
   ├─ Convert date range to ISO format
   ├─ Build database query:
   │  SELECT * FROM agents
   │  WHERE app_id = $1
   │  AND created_at BETWEEN $2 AND $3
   │  ORDER BY {sort_column} {sort_order}
   │  LIMIT {page_size}
   │  OFFSET {(page_number - 1) * page_size}
   │
   ├─ Execute async query on PostgreSQL
   ├─ Aggregate metrics if needed:
   │  ├─ Token count per model
   │  ├─ Average latency
   │  ├─ Error rates
   │  └─ Success rates
   └─ Pagination metadata calculated

4. RESPONSE FORMATTING
   ├─ Convert UUID objects to strings (JSON serializable)
   ├─ Create response envelope:
   │  {
   │    "status": "success",
   │    "code": 200,
   │    "message": "Fetched agent metrics",
   │    "timestamp": "2026-06-10T15:30:00Z",
   │    "content": {
   │      "agents": [...],
   │      "total_count": 45,
   │      "page_number": 1,
   │      "page_size": 10,
   │      "total_pages": 5
   │    }
   │  }
   └─ Return JSON response

5. FRONTEND RECEIVES RESPONSE
   ├─ Axios interceptor processes response
   ├─ React state updated via setState/useReducer
   ├─ AppContext triggers component re-render
   ├─ Chart components receive new data
   └─ UI updates with:
      ├─ Agent list in table
      ├─ Pie chart showing agent distribution
      ├─ Line chart showing trends
      └─ Pagination controls

6. USER SEES VISUALIZATION
   ├─ Metrics displayed in Material-UI tables
   ├─ Nivo pie charts rendered
   ├─ Real-time updates enabled
   └─ User can filter/sort by clicking UI controls
```

### Authentication Flow

```
1. LOGIN INITIATION
   Browser → GET /api/auth/login
   ├─ No authentication required
   ├─ Generate authorization URL
   └─ Redirect to Azure AD

2. AZURE AD AUTHENTICATION
   Browser → Azure AD Login Portal
   ├─ User enters credentials
   ├─ MFA (if configured)
   ├─ Azure validates identity
   └─ Redirects back with auth code

3. AUTHORIZATION CODE EXCHANGE
   Browser → GET /api/auth/redirect?code=AUTH_CODE
   ├─ Backend receives auth code
   ├─ Exchange code for ID token + access token using MSAL
   └─ Extract user information from ID token

4. SESSION CREATION
   Backend → PostgreSQL
   ├─ Create user record if new
   ├─ Create session record
   ├─ Generate JWT token (RS256 signed)
   ├─ Encrypt JWT payload (AES-GCM double encryption)
   └─ Hash credentials for session validation

5. TOKEN DELIVERY
   Backend → Browser
   ├─ Set secure HTTP-only cookie with JWT
   ├─ Include session ID in response
   ├─ Redirect to frontend dashboard
   └─ Frontend stores in secure local storage

6. SUBSEQUENT REQUESTS
   Browser → Backend (with Authorization header)
   ├─ Include JWT in Bearer token
   ├─ Backend validates:
   │  ├─ JWT signature (RS256)
   │  ├─ JWT expiry
   │  ├─ Session status in database
   │  └─ Session expiry time
   └─ If valid → Proceed with request
      If invalid → Return 401 Unauthorized

7. TOKEN REFRESH
   When JWT expires:
   ├─ Frontend detects 401 response
   ├─ Calls POST /api/auth/token/refresh
   ├─ Backend generates new JWT
   ├─ Updates session record
   └─ Frontend retries original request

8. LOGOUT
   User clicks Logout:
   ├─ Frontend calls GET /api/auth/logout
   ├─ Backend:
   │  ├─ Updates session status to "inactive"
   │  ├─ Clears sensitive data
   │  └─ Returns success response
   ├─ Frontend clears secure storage
   └─ Redirects to login page
```

---

## Deployment Architecture

### Docker Build Process

```dockerfile
Base Image: Python 3.12
   ├─ Install kubectl (Kubernetes CLI)
   ├─ Install Node.js 20
   ├─ Install jq (JSON query tool)
   ├─ Install UV (Python package manager)
   │
   ├─ Copy project files
   │
   ├─ Install Python dependencies
   │  ├─ UVicorn (ASGI server)
   │  ├─ All pip packages from requirements.txt
   │  └─ Dev dependencies
   │
   ├─ Build Frontend
   │  ├─ Run frontend.sh
   │  ├─ npm install (React dependencies)
   │  ├─ npm run build (Vite compilation)
   │  └─ Output to dist/
   │
   ├─ Set environment variables:
   │  ├─ sec_key (AES encryption key)
   │  ├─ PRIVATE_KEY (RSA private key for JWT)
   │  └─ PUBLIC_KEY (RSA public key for JWT)
   │
   └─ Entry Point: start.sh (Gunicorn with Uvicorn workers)
       └─ 2 worker processes, binding to 0.0.0.0:8012
```

### Kubernetes Deployment

```yaml
Namespace: ds-test
├─ Deployment: ds-test-backend-deployment
│  ├─ Replicas: 1 (can be scaled horizontally)
│  ├─ Service Account: ds-sa
│  │
│  ├─ Init Container: secret-init
│  │  ├─ Fetches Kubernetes JWT token
│  │  ├─ Authenticates to HashiCorp Vault
│  │  ├─ Retrieves secrets:
│  │  │  ├─ Database credentials
│  │  │  ├─ Azure credentials
│  │  │  ├─ API keys
│  │  │  └─ JWT keys
│  │  └─ Writes secrets to filesystem
│  │
│  ├─ Main Container:
│  │  ├─ Image: aksuatregistry.azurecr.io/hexa-tensai-automation/deployment_studio_dev-test:HASH
│  │  │
│  │  ├─ Resource Requests:
│  │  │  ├─ CPU: 100m
│  │  │  └─ Memory: 800Mi
│  │  │
│  │  ├─ Resource Limits:
│  │  │  ├─ CPU: 800m
│  │  │  └─ Memory: 1800Mi
│  │  │
│  │  ├─ Port: 8012
│  │  │
│  │  ├─ Mounts:
│  │  │  └─ filespvc (PersistentVolume for keys)
│  │  │
│  │  ├─ Liveness Probe:
│  │  │  ├─ HTTP GET /health
│  │  │  ├─ Initial Delay: 30s
│  │  │  └─ Period: 10s
│  │  │
│  │  └─ Readiness Probe:
│  │     ├─ HTTP GET /ready
│  │     ├─ Initial Delay: 10s
│  │     └─ Period: 5s
│  │
│  └─ Service: ds-test-backend-service
│     ├─ Type: ClusterIP
│     ├─ Port: 8012
│     └─ Target Port: 8012
│
└─ PersistentVolumeClaim: filespvc
   ├─ Storage Class: default
   ├─ Access Mode: ReadWriteOnce
   └─ Size: 100Mi
```

### Secret Management Flow

```
Kubernetes Cluster
    │
    ├─ ServiceAccount: ds-sa
    │  └─ Mounts /var/run/secrets/kubernetes.io/serviceaccount/token
    │
    ├─ Init Container executes:
    │  │
    │  ├─ Step 1: Read Kubernetes JWT
    │  │  cat /var/run/secrets/kubernetes.io/serviceaccount/token
    │  │
    │  ├─ Step 2: Authenticate to Vault
    │  │  POST https://vault-uat.hexagents.ai/v1/auth/kubernetes/login
    │  │  {
    │  │    "role": "deployment-studio-test",
    │  │    "jwt": "K8S_JWT"
    │  │  }
    │  │
    │  ├─ Step 3: Receive Vault Token
    │  │  Response:
    │  │  {
    │  │    "auth": {
    │  │      "client_token": "VAULT_TOKEN",
    │  │      "lease_duration": 3600
    │  │    }
    │  │  }
    │  │
    │  └─ Step 4: Retrieve Secrets
    │     GET https://vault-uat.hexagents.ai/v1/secret/data/ds-test/...
    │     Header: X-Vault-Token: VAULT_TOKEN
    │     │
    │     ├─ db_password
    │     ├─ azure_client_id
    │     ├─ azure_client_secret
    │     ├─ jwt_private_key
    │     └─ jwt_public_key
    │
    └─ Write to /ds/keys/
       └─ Pod filesystem (mounted volume)

Application Container
    │
    └─ Reads secrets from filesystem
       ├─ env.sh sources environment
       ├─ Environment variables set
       └─ FastAPI application starts with secrets
```

### Port Configuration

| Port | Service | Purpose |
|------|---------|---------|
| 8012 | FastAPI Backend | Main application HTTP server |
| 5432 | PostgreSQL | Database connection (external) |
| 443 | HTTPS | Secure communication with Azure services |

### Scaling Strategy

- **Horizontal Scaling**: Increase pod replicas
- **Load Balancing**: Kubernetes Service distributes traffic
- **Database Connection Pooling**: Max 5 concurrent connections per instance
- **Caching**: Metrics cached in PostgreSQL, invalidated on updates

---

## Security & Authentication

### JWT Token Structure

```
Header:
{
  "alg": "RS256",
  "typ": "JWT"
}

Payload (Encrypted with AES-GCM):
{
  "sub": "user_id",
  "email": "user@example.com",
  "role": "itam_admin",
  "exp": 1234567890,
  "iat": 1234567000,
  "session_id": "session_uuid"
}

Signature:
RSASSA-PKCS1-v1_5 sign(header.payload, private_key)
```

### Encryption Mechanisms

**1. JWT Payload Encryption (AES-GCM)**
```python
# Double encryption for enhanced security
IV = 12 bytes (random)
Key = 32 bytes (from environment)
Mode = GCM (Galois/Counter Mode)
Auth Tag = 16 bytes

Encrypted = Base64(IV + Auth_Tag + Ciphertext)
```

**2. Credential Encryption (Fernet Symmetric)**
```python
# For storing temporary credentials
Key = Fernet key (from config)
Algorithm = Fernet (AES-128 in CBC mode + HMAC)
```

**3. Database Password Hashing**
```python
# For session validation
Algorithm = SHA-256
Output = First 16 characters (64-bit)
```

### Security Headers

```http
Content-Security-Policy: frame-ancestors 'none'; object-src 'none'; script-src 'self'
Strict-Transport-Security: max-age=31536000
X-Frame-Options: SAMEORIGIN
Referrer-Policy: no-referrer-when-downgrade
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### Cookie Configuration

```http
Set-Cookie: authToken=JWT; 
  Secure;           (HTTPS only)
  HttpOnly;         (No JavaScript access)
  SameSite=Lax;     (CSRF protection)
  Path=/;
  Max-Age=900       (15 minutes)
```

### CORS Policy

```python
allow_origins = [
  "https://deployment-studio-test.tensai.run"
]
allow_credentials = True
allow_methods = ["*"]
allow_headers = ["*"]
```

### Role-Based Access Control (RBAC)

```
Azure AD Groups → Application Roles → API Permissions

Required Role: "itam_admin"
├─ Full read access to all resources
├─ Write access to infrastructure
├─ Can create/modify applications
└─ Can manage user sessions
```

---

## API Reference

### Authentication Endpoints

#### 1. Initiate Login
```http
GET /api/auth/login
```
**Response**: Redirect to Azure AD OAuth endpoint

#### 2. OAuth Callback Handler
```http
GET /api/auth/redirect?code=AUTH_CODE
```
**Response**:
```json
{
  "status": "success",
  "message": "Authenticated successfully",
  "auth_token": "JWT_TOKEN",
  "user": {
    "id": "user_uuid",
    "name": "John Doe",
    "email": "john@example.com",
    "role": "itam_admin"
  }
}
```

#### 3. Refresh Token
```http
POST /api/auth/token/refresh
Authorization: Bearer JWT_TOKEN
```
**Response**:
```json
{
  "status": "success",
  "auth_token": "NEW_JWT_TOKEN"
}
```

#### 4. Logout
```http
GET /api/auth/logout
Authorization: Bearer JWT_TOKEN
```

### Agent Metrics Endpoints

#### Paginated Agent Metrics
```http
POST /api/agent/agent-wise-metrics-paginated
Content-Type: application/json
Authorization: Bearer JWT_TOKEN

{
  "app_id": "app-12345",
  "page_number": 1,
  "page_size": 10,
  "search_filter": "search_text",
  "start_date": "2026-06-01",
  "end_date": "2026-06-10",
  "sort_column": "created_at",
  "sort_order": "DESC"
}
```

**Response**:
```json
{
  "status": "success",
  "code": 200,
  "message": "Fetched paginated agent-wise metrics successfully",
  "content": {
    "agents": [
      {
        "id": "agent_uuid",
        "name": "Agent-1",
        "status": "healthy",
        "invocations": 1250,
        "avg_latency_ms": 245,
        "error_rate": 0.02,
        "tokens_used": 15000
      }
    ],
    "total_count": 45,
    "page_number": 1,
    "page_size": 10,
    "total_pages": 5
  }
}
```

### Workflow Endpoints

#### Get Workflow Metrics
```http
POST /api/workflow/workflow-wise-metrics-paginated
Content-Type: application/json
Authorization: Bearer JWT_TOKEN

{
  "app_id": "app-12345",
  "page_number": 1,
  "page_size": 10,
  "duration_hours": 24
}
```

### Cost Analytics Endpoints

#### Get Overall Costs
```http
POST /api/cost/overall-costs
Content-Type: application/json
Authorization: Bearer JWT_TOKEN

{
  "app_id": "app-12345",
  "start_date": "2026-06-01",
  "end_date": "2026-06-10",
  "currency_type": "USD"
}
```

**Response**:
```json
{
  "status": "success",
  "content": {
    "total_cost": 1250.50,
    "currency": "USD",
    "period": "2026-06-01 to 2026-06-10",
    "daily_breakdown": [
      {
        "date": "2026-06-01",
        "cost": 150.25
      }
    ]
  }
}
```

### Infrastructure Endpoints

#### List Applications
```http
GET /api/custominfra/applications
Authorization: Bearer JWT_TOKEN
```

#### Create Application
```http
POST /api/custominfra/applications
Content-Type: application/json
Authorization: Bearer JWT_TOKEN

{
  "name": "My Application",
  "platform": "azure",
  "environment": "production",
  "agent_framework": "agno"
}
```

---

## Summary

**Deployment Studio** is a comprehensive observability platform that:

1. **Integrates seamlessly** with Azure cloud services for resource discovery, monitoring, and cost analysis
2. **Provides real-time dashboards** for infrastructure and AI agent monitoring
3. **Implements enterprise-grade security** with JWT tokens, encryption, and role-based access control
4. **Scales horizontally** via Kubernetes with configurable resource limits
5. **Manages secrets securely** using HashiCorp Vault with Kubernetes authentication
6. **Supports multiple AI frameworks** (Agno, Strands, Microsoft Agentic Framework)
7. **Offers comprehensive APIs** for metrics, workflows, costs, and infrastructure management
8. **Maintains full observability** with distributed tracing, structured logging, and performance monitoring

The application architecture follows cloud-native best practices with containerization, orchestration, and microservices principles, making it production-ready and scalable for enterprise deployments.
