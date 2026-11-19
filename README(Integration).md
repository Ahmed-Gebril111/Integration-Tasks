<div align="center">

# 🔌 IBM ACE Integration Projects

**Hands-on enterprise integration solutions built with IBM App Connect Enterprise (ACE)**

SOAP • REST • MQ • Aggregation • DFDL • OAuth 2.0 • Config-driven Gateways

![IBM ACE](https://img.shields.io/badge/IBM-App%20Connect%20Enterprise-052FAD?style=for-the-badge&logo=ibm&logoColor=white)
![ESQL](https://img.shields.io/badge/Language-ESQL-blue?style=for-the-badge)
![Java](https://img.shields.io/badge/Java-Compute%20Node-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![IBM MQ](https://img.shields.io/badge/IBM-MQ-1F70C1?style=for-the-badge&logo=ibm&logoColor=white)
![Status](https://img.shields.io/badge/Status-Training%20Portfolio-success?style=for-the-badge)

</div>

---

## 📖 About

This repository is a collection of **IBM App Connect Enterprise (ACE)** integration tasks that progress from a basic SOAP service to a multi-application, security-aware **API gateway capstone**.

Each task focuses on a real integration pattern used in enterprise environments: protocol bridging, message validation, asynchronous messaging, scatter-gather aggregation, fixed-length file parsing, and centralized logging with externalized configuration.

> Built and tested with **IBM ACE Toolkit** and a local Integration Node.

---

## 🗂️ Repository Structure

```
IBM-ACE_Projects/
├── Task-1-Calculator/        # SOAP service, validation, routing, error handling
├── Task-2-Calculator/        # REST API (OpenAPI) wrapping a SOAP backend
├── Task-3-Calculator/        # Async, queue-based multi-application design (IBM MQ)
├── Task-4-Mock-Service/      # Scatter-gather aggregation over mocked services
├── Task-5-DFDL/              # DFDL message model for fixed-length records
├── Task-6-Image/             # Base64 image validation & format detection
└── Capstone-Project/         # SOAP-to-REST gateway with OAuth, config & DB logging
```

---

## 🧩 Projects Overview

| # | Project | Pattern | Key ACE Concepts |
|---|---------|---------|------------------|
| 1 | **Calculator (SOAP)** | Request/Response web service | SOAP Input/Reply, WSDL, RouteToLabel, subflows, error handlers |
| 2 | **Calculator (REST)** | REST façade over SOAP | OpenAPI (`T2Front.yaml`), REST API, subflows, structured error responses |
| 3 | **Calculator (Async/MQ)** | Decoupled multi-app messaging | HTTP Input, MQ Input/Output, Route, FlowOrder, correlation storage |
| 4 | **Mock Service Aggregator** | Scatter-gather | Aggregate Control/Request/Reply, HTTP Async Request, Java Compute, UDP policy |
| 5 | **DFDL Message Model** | Fixed-length record parsing | DFDL schema, message model library |
| 6 | **Image Validator** | Content inspection | BLOB handling, `BASE64DECODE`, magic-number detection |
| ⭐ | **Capstone Gateway** | API gateway / orchestration | SOAP ↔ JSON, OAuth 2.0, security profile, external config, DB audit logging |

---

## 📌 Task Details

### Task 1 — Calculator (SOAP)
A SOAP web service exposing Add / Subtract / Multiply / Divide, defined by WSDL contracts (front-end and back-end).

- Validates the request (`requestId`, numeric operands, allowed operation, divide-by-zero)
- Routes to the correct operation using **Route To Label**
- Dedicated **Catch / Failure / HTTP-Timeout** handler subflows
- Builds a response and logs each transaction

### Task 2 — Calculator (REST)
A RESTful adaptation of the SOAP calculator, designed **API-first** with an OpenAPI spec.

- `performCalculation` operation with JSON request/response
- Distinct responses for **bad request**, **validation errors**, **backend faults**, and **technical errors**
- Reusable `LogTransaction` subflow

### Task 3 — Calculator (Asynchronous, MQ-based)
Three cooperating applications communicating through IBM MQ queues (`VST3Q1`, `VST3Q2`, `VST3Q3`).

| Application | Responsibility |
|-------------|----------------|
| **APP1** | Receives HTTP requests on `/VST3`, validates, stores correlation data, puts message on queue, and later replies to the original caller |
| **APP2** | Consumes the queue, validates business rules, transforms **JSON → SOAP**, calls the external calculator service, and publishes the result |
| **APP3** | Consumes the log queue and builds audit log entries |

### Task 4 — Mock Service Aggregator
Calls **three mock services in parallel** and merges the results into a single response.

- `/aggregate` endpoint → Aggregate Control → three async HTTP requests
- `Flow2` collects replies with Aggregate Reply and builds the combined response
- Timeout handling with a dedicated response builder
- Java Compute nodes + a **User-Defined Policy** (UDP) for service configuration
- Designed to work with Postman mock servers

### Task 5 — DFDL Message Model
A **DFDL** model (`DmdReq.xsd`) that parses fixed-length transaction records (transaction ID, type, dates, PSP/bank identifiers, amount, currency, payer details, etc.) into structured messages.

### Task 6 — Image Validator
Accepts a Base64-encoded image in JSON and inspects its **binary signature** to determine the real format — no reliance on file extensions.

- Detects: `JPEG`, `PNG`, `GIF`, `BMP`, `WEBP`, `TIFF`
- Returns `isImage`, `isJpeg`, `detectedFormat`, and a human-readable message
- Endpoint: `/imageTest`

### ⭐ Capstone Project — Calculator Gateway
A multi-application gateway that exposes a **SOAP front-end** and orchestrates **secured REST back-ends**.

```
 SOAP Client
     │
     ▼
┌───────────────────┐   action = sum / sub / multi
│   App1 (Gateway)  │ ─────────────────────────────┐
│  SOAP → JSON      │                              │
│  Routing + Audit  │                              ▼
└───────────────────┘                   ┌─────────────────────┐
     │  ▲                                │  App2 / App3 / App4 │
     │  │ JSON → SOAP                    │  OAuth 2.0 token    │
     ▼  │                                │  + secured REST call│
┌───────────────────┐                   └─────────────────────┘
│   SQL Database    │
│ CalculatorGateway │
│       Log         │
└───────────────────┘
```

**Highlights**
- SOAP front-end (WSDL/XSD) transformed to JSON for the back-ends
- **Action-based routing** (`sum`, `sub`, `multi`) driven by an external JSON config
- **OAuth 2.0 client-credentials** flow to obtain a bearer token before calling the backend
- **Security Profile** policy for basic authentication on the gateway
- **Four-stage audit logging** to a database (`RECEIVED → ROUTED → BACKEND_OK → COMPLETED`) using a correlation ID, with each stage toggleable via config flags
- Custom Java helper (`ConfigFileReader`) to read values from the config file

---

## 🛠️ Tech Stack

- **IBM App Connect Enterprise** (Toolkit + Integration Node)
- **ESQL** and **Java Compute Nodes**
- **IBM MQ** (asynchronous messaging)
- **SOAP / WSDL / XSD**, **REST / OpenAPI**, **JSON**
- **DFDL** (Data Format Description Language)
- **OAuth 2.0**, Security Profiles, User-Defined Policies
- **SQL database** (ODBC datasource) for audit logging
- **Postman** mock servers for testing

---

## ⚙️ Getting Started

### Prerequisites
- IBM App Connect Enterprise (Toolkit) installed
- A running Integration Node / Integration Server
- IBM MQ with a queue manager (required for **Task 3**)
- A database with an ODBC datasource (required for the **Capstone**)
- [Postman](https://www.postman.com/) (optional, for mock services and testing)

### Import into the Toolkit
1. Clone the repository
   ```bash
   git clone https://github.com/<your-username>/IBM-ACE_Projects.git
   ```
2. Open **IBM ACE Toolkit** and choose **File → Import → General → Existing Projects into Workspace**.
3. Select the task folder you want and import all projects inside it.
4. Create a **BAR file** (or deploy directly) to your Integration Server.

### Configuration
Some projects need environment-specific setup before they run:

| Project | What to configure |
|---------|-------------------|
| Task 3 | Create queues `VST3Q1`, `VST3Q2`, `VST3Q3` on your queue manager |
| Task 4 | Replace mock endpoints in the flow / `Service3UDP` policy with your own Postman mock URLs |
| Capstone | Create a `gatewayConfig.json` (routing URLs, OAuth settings, logging flags) and update the `CONFIG_FILE` path in the ESQL; create the `CalculatorGatewayLog` table and ODBC datasource |

> 🔐 **Never commit real credentials.** Keep `gatewayConfig.json`, client secrets, and API keys out of version control.

---

## 🧪 Example Requests

**Task 6 — Image Validator**
```http
POST http://localhost:7800/imageTest
Content-Type: application/json

{
  "image": "<base64-encoded-image>"
}
```

**Task 6 — Sample response**
```json
{
  "isImage": true,
  "detectedFormat": "PNG",
  "isJpeg": false,
  "message": "This is an image, but not JPEG (detected: PNG)."
}
```

**Task 3 — Calculator request**
```http
POST http://localhost:7800/VST3
Content-Type: application/json

{
  "requestId": "REQ-001",
  "Num1": 10,
  "Num2": 5,
  "operation": "ADD"
}
```

---

## 🎯 Skills Demonstrated

- Designing synchronous and asynchronous integration flows
- Contract-first (WSDL) and API-first (OpenAPI) development
- Robust error handling: catch, failure, and timeout paths
- Message transformation between SOAP, XML, and JSON
- Scatter-gather orchestration with timeouts
- Externalized configuration and policy-driven design
- Securing integrations with OAuth 2.0 and security profiles
- Audit logging with correlation IDs
- Binary / fixed-length data parsing with DFDL

---

## 🚀 Roadmap

- [ ] Add Postman collections for every task
- [ ] Add BAR files / deployment scripts
- [ ] Add architecture diagrams per task
- [ ] Containerize with the ACE Docker image
- [ ] CI/CD pipeline for building and testing BAR files
- [ ] Complete Capstone **App4** and Task 5 flow

---


