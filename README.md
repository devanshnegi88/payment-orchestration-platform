# 💳 PayFlow — Payment Orchestration Layer

> **A production-grade payment orchestration platform that intelligently routes transactions across multiple payment gateways with fault-tolerant failover, idempotency, webhook reconciliation, and real-time monitoring.**

<div align="center">

**Multi-Gateway Routing • High Availability • Idempotency • Reconciliation • Observability**

</div>

---

## 🛠️ Tech Stack

<div align="center">

![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-API-009688?style=flat-square&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-Cache%20%26%20Queue-DC382D?style=flat-square&logo=redis&logoColor=white)
![Celery](https://img.shields.io/badge/Celery-Async%20Processing-37814A?style=flat-square&logo=celery&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-E6522C?style=flat-square&logo=prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/Grafana-Dashboards-F46800?style=flat-square&logo=grafana&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?style=flat-square&logo=docker&logoColor=white)
![Razorpay](https://img.shields.io/badge/Razorpay-Gateway-3395FF?style=flat-square)
![Stripe](https://img.shields.io/badge/Stripe-Gateway-635BFF?style=flat-square&logo=stripe&logoColor=white)
![PayU](https://img.shields.io/badge/PayU-Gateway-00AEEF?style=flat-square)

</div>

---

# 📌 Overview

**PayFlow** is a backend payment orchestration system designed to improve payment reliability, availability, and transaction success by dynamically routing payments across multiple payment providers.

Instead of coupling an application directly to a single payment gateway, PayFlow provides an orchestration layer that evaluates gateway health and performance before selecting an appropriate provider.

The platform combines:

- 💳 Multi-gateway payment processing
- 🧠 Intelligent gateway selection
- ⚡ Low-latency failover
- 🔄 Circuit breaker protection
- 🔐 Idempotency management
- 📨 Webhook reconciliation
- 📒 Audit trail generation
- 📊 Real-time monitoring
- ♻️ Retry and recovery mechanisms

---

# 🎯 Problem Statement

E-commerce platforms often depend heavily on external payment providers.

A gateway can experience:

- Temporary outages
- Increased latency
- High failure rates
- Network issues
- Provider-side errors
- Capacity or availability problems

A single-gateway architecture can therefore turn a gateway failure into a complete payment-processing failure.

PayFlow introduces an orchestration layer:

```text
                    E-Commerce Application
                            │
                            ▼
                   ┌─────────────────┐
                   │     PayFlow     │
                   │  Orchestration  │
                   └────────┬────────┘
                            │
                ┌───────────┼───────────┐
                │           │           │
                ▼           ▼           ▼
           Razorpay      Stripe        PayU
                │           │           │
                └───────────┼───────────┘
                            │
                            ▼
                       Payment Result
```

This allows the application to interact with **one orchestration API** while PayFlow manages gateway selection and failure handling internally.

---

# ✨ Core Features

## 💳 Multi-Gateway Orchestration

Supported payment providers include:

- Razorpay
- Stripe
- PayU
- UPI

The orchestration layer abstracts gateway-specific processing behind a common payment workflow.

---

## 🧠 Intelligent Gateway Selection

Gateway selection can consider:

- Success rate
- Latency
- Cost
- Gateway health
- Current traffic distribution

```text
Payment Request
      │
      ▼
Evaluate Gateway Health
      │
      ▼
Calculate Gateway Scores
      │
      ├──────────────┐
      │              │
      ▼              ▼
  Razorpay        Stripe
      │              │
      └──────┬───────┘
             │
             ▼
      Select Gateway
             │
             ▼
       Process Payment
```

---

# ⚡ High Availability

PayFlow is designed around failure-tolerant payment processing.

### Reliability mechanisms

- Circuit breaker pattern
- Automatic failover
- Retry mechanisms
- Dead Letter Queue
- Webhook reconciliation
- Payment state validation

```text
                    Payment Request
                           │
                           ▼
                    Gateway Selection
                           │
                           ▼
                    Primary Gateway
                           │
                  ┌────────┴────────┐
                  │                 │
               Success            Failure
                  │                 │
                  ▼                 ▼
               Complete       Circuit Breaker
                                    │
                                    ▼
                              Failover Gateway
                                    │
                                    ▼
                              Retry Payment
                                    │
                                    ▼
                              Final Result
```

---

# 🔐 Payment Reliability

Payment systems require stronger guarantees than ordinary API requests.

PayFlow incorporates:

### Idempotency Protection

Prevents duplicate processing when the same payment request is retried.

```text
Request
   │
   ▼
Idempotency Key
   │
   ▼
Already Processed?
   │
 ┌─┴─────────┐
 │           │
Yes          No
 │           │
 ▼           ▼
Return      Process
Existing    Payment
Result        │
              ▼
          Store Result
```

This protects against duplicate charges caused by:

- Client retries
- Network timeouts
- Gateway delays
- Application retries
- Duplicate requests

---

# 🔄 Webhook Reconciliation

Payment providers may send asynchronous webhook events after the original transaction request.

PayFlow processes these events to reconcile the internal payment state.

```text
Payment Gateway
      │
      │ Webhook
      ▼
┌─────────────────┐
│ PayFlow Webhook │
│     Handler     │
└────────┬────────┘
         │
         ▼
Validate Event
         │
         ▼
Deduplicate
         │
         ▼
Validate State Transition
         │
         ▼
Update Payment State
         │
         ▼
Audit Event
```

Webhook deduplication prevents the same provider event from being processed multiple times.

---

# 🔄 Payment State Machine

Payment transactions follow controlled state transitions.

```text
                    ┌──────────────┐
                    │   CREATED   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   PENDING    │
                    └──────┬───────┘
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
          ┌────────────┐      ┌────────────┐
          │  SUCCESS   │      │   FAILED   │
          └────────────┘      └─────┬──────┘
                                    │
                                    ▼
                             Failover / Retry
                                    │
                                    ▼
                              Reconciliation
```

Invalid or unexpected transitions can be rejected rather than blindly updating transaction state.

---

# 🧠 Intelligent Routing Engine

The routing engine evaluates available payment gateways before selecting a provider.

Conceptually:

```text
Gateway Score
      │
      ├── Success Rate
      ├── Latency
      ├── Cost
      ├── Health
      └── Traffic
      │
      ▼
Normalized Gateway Score
      │
      ▼
Gateway Ranking
      │
      ▼
Selected Provider
```

This creates a flexible foundation for future routing strategies.

---

# 📊 Gateway Health Monitoring

PayFlow tracks gateway health to support routing and failover decisions.

```text
                  Gateway Monitor
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
     Razorpay         Stripe           PayU
        │               │               │
        ▼               ▼               ▼
    Success Rate     Latency          Errors
        │               │               │
        └───────────────┼───────────────┘
                        │
                        ▼
                 Gateway Health
                        │
                        ▼
                  Routing Engine
```

Monitoring metrics can include:

- Transaction success rate
- Gateway latency
- Error frequency
- Current availability
- Payment throughput

---

# 🔌 Gateway Abstraction

The orchestration layer separates business logic from individual payment providers.

Conceptually:

```text
                 Payment Service
                       │
                       ▼
              Gateway Interface
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       ▼               ▼               ▼
   Razorpay          Stripe           PayU
   Adapter           Adapter          Adapter
```

This allows gateway-specific APIs to remain isolated from the rest of the payment workflow.

---

# 📨 Asynchronous Processing

Background processing is handled using **Celery + Redis**.

Typical asynchronous workloads can include:

```text
Payment Events
      │
      ▼
Message Queue
      │
      ▼
Celery Worker
      │
      ├── Webhook Processing
      ├── Reconciliation
      ├── Retry Jobs
      └── Recovery Tasks
```

Redis provides the queue/broker infrastructure for asynchronous workloads.

---

# ☠️ Dead Letter Queue

Transactions or events that repeatedly fail processing can be moved into a **Dead Letter Queue (DLQ)** for further investigation or controlled retry.

```text
Event
  │
  ▼
Processing
  │
  ├── Success ───────► Complete
  │
  └── Failure
         │
         ▼
       Retry
         │
         ├── Success ─► Complete
         │
         └── Repeated Failure
                  │
                  ▼
                 DLQ
```

This prevents repeatedly failing events from blocking the normal processing pipeline.

---

# 📒 Audit Trail

Every important payment event can be recorded for traceability.

Example lifecycle:

```text
Payment Created
      ↓
Gateway Selected
      ↓
Payment Initiated
      ↓
Gateway Response
      ↓
Webhook Received
      ↓
Payment Reconciled
      ↓
Final State
```

Audit information is useful for:

- Debugging
- Reconciliation
- Operational investigations
- Payment lifecycle visibility
- Failure analysis

---

# 📊 Monitoring & Analytics

PayFlow includes an observability layer using:

```text
Prometheus
     │
     ▼
Metrics Collection
     │
     ▼
Grafana
     │
     ▼
Dashboards
```

Potential dashboard metrics include:

| Metric | Purpose |
|---|---|
| Payment Success Rate | Track transaction outcomes |
| Gateway Latency | Monitor provider performance |
| Gateway Errors | Detect provider problems |
| Failover Count | Measure fallback activity |
| Webhook Processing Time | Monitor event processing |
| Transaction Throughput | Track system load |
| Reconciliation Results | Identify state mismatches |

---

# 🏗️ System Architecture

```text
                         ┌─────────────────────┐
                         │   E-Commerce App    │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     FastAPI API     │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Payment Orchestrator│
                         └──────────┬──────────┘
                                    │
                   ┌────────────────┼────────────────┐
                   │                │                │
                   ▼                ▼                ▼
             Routing Engine   State Machine    Idempotency
                   │                │                │
                   └────────────────┼────────────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Gateway Abstraction │
                         └──────────┬──────────┘
                                    │
                  ┌─────────────────┼─────────────────┐
                  │                 │                 │
                  ▼                 ▼                 ▼
              Razorpay           Stripe             PayU
                  │                 │                 │
                  └─────────────────┼─────────────────┘
                                    │
                                    ▼
                           Payment Providers
                                    │
                                    ▼
                               Webhooks
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │ Reconciliation      │
                         │ Engine              │
                         └──────────┬──────────┘
                                    │
                         ┌──────────┴──────────┐
                         │                     │
                         ▼                     ▼
                    PostgreSQL              Redis
                         │                     │
                         │                     ▼
                         │                  Celery
                         │                     │
                         │                     ▼
                         │               Async Workers
                         │
                         ▼
                    Audit Ledger

                         Monitoring
                             │
                    ┌────────┴────────┐
                    ▼                 ▼
               Prometheus          Grafana
```

---

# 🔄 Complete Payment Workflow

The end-to-end flow is:

```text
1. Receive Payment Request
            │
            ▼
2. Validate Request
            │
            ▼
3. Check Idempotency
            │
            ▼
4. Evaluate Gateway Health
            │
            ▼
5. Calculate Gateway Scores
            │
            ▼
6. Select Gateway
            │
            ▼
7. Initiate Payment
            │
       ┌────┴────┐
       │         │
    Success    Failure
       │         │
       │         ▼
       │    Circuit Breaker
       │         │
       │         ▼
       │    Select Fallback
       │         │
       │         ▼
       │       Retry
       │         │
       └────┬────┘
            ▼
8. Record Transaction
            │
            ▼
9. Receive Webhook
            │
            ▼
10. Validate State
            │
            ▼
11. Reconcile
            │
            ▼
12. Audit Event
            │
            ▼
13. Update Monitoring Metrics
```

---

# 🚀 Getting Started

## Prerequisites

Make sure the following are available:

- Docker
- Docker Compose
- Git

---

## 1. Clone Repository

```bash
git clone https://github.com/devanshnegi88/payment-orchestration-platform.git

cd payflow
```

---

## 2. Configure Environment

Create the environment file:

```bash
cp .env.example .env
```

Update the required configuration values in `.env`.

Typical configuration may include:

```env
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
RAZORPAY_KEY_ID=...
RAZORPAY_KEY_SECRET=...
STRIPE_SECRET_KEY=...
PAYU_MERCHANT_KEY=...
```

Use the variables defined by the project's `.env.example` as the source of truth for the actual configuration.

---

## 3. Start Services

```bash
docker-compose up -d
```

Check running containers:

```bash
docker-compose ps
```

---

# ❤️ Health Check

Verify that the API is running:

```bash
curl http://localhost:8000/api/v1/health
```

---

# 🌐 Service Access

| Service | URL |
|---|---|
| API | `http://localhost:8000` |
| Swagger API Docs | `http://localhost:8000/docs` |
| Grafana | `http://localhost:3000` |
| Flower | `http://localhost:5555` |

---

# 📈 Performance Targets

The current project defines the following performance targets:

| Metric | Target |
|---|---:|
| ⚡ Payment Initiation | **< 500 ms** |
| 🔄 Failover Latency | **< 2 seconds** |
| 📨 Webhook Processing | **< 200 ms** |
| 🚀 Throughput | **100+ requests/sec** |
| 🔐 Idempotency Check | **< 10 ms** |

> **Note:** These are project performance targets, not presented here as independently benchmarked production measurements.

---

# 🛡️ Reliability Architecture

PayFlow combines multiple reliability patterns rather than relying on a single mechanism.

```text
                 Payment Request
                       │
                       ▼
                 Idempotency
                       │
                       ▼
               Gateway Selection
                       │
                       ▼
                Circuit Breaker
                       │
                       ▼
                    Retry
                       │
                       ▼
                  Failover
                       │
                       ▼
               Webhook Event
                       │
                       ▼
                Reconciliation
                       │
                       ▼
                  Audit Log
                       │
                       ▼
                 Monitoring
```

Each component addresses a different failure mode.

---

# 🔐 Idempotency Strategy

Idempotency is especially important in payment systems because retrying an HTTP request must not accidentally create a second charge.

```text
Client
  │
  │ Payment + Idempotency Key
  ▼
PayFlow
  │
  ▼
Idempotency Store
  │
  ├── Existing Request
  │       │
  │       ▼
  │   Return Previous Result
  │
  └── New Request
          │
          ▼
      Process Payment
          │
          ▼
      Store Result
```

This provides protection against duplicate requests and retry storms.

---

# 🔄 Circuit Breaker

The circuit breaker protects the system from continuously sending traffic to an unhealthy gateway.

Conceptually:

```text
             ┌─────────────┐
             │   CLOSED    │
             │ Normal Flow │
             └──────┬──────┘
                    │
              Failures Increase
                    │
                    ▼
             ┌─────────────┐
             │    OPEN     │
             │ Fail Fast   │
             └──────┬──────┘
                    │
               Recovery Test
                    │
                    ▼
             ┌─────────────┐
             │ HALF-OPEN   │
             │ Test Gateway│
             └──────┬──────┘
                    │
              ┌─────┴─────┐
              │           │
            Success     Failure
              │           │
              ▼           ▼
           CLOSED        OPEN
```

---

# 🧾 Reconciliation Engine

The reconciliation engine ensures that the internal payment state eventually matches the payment provider's final state.

```text
Internal State
      │
      ▼
Webhook / Provider Event
      │
      ▼
Validate Event
      │
      ▼
Compare States
      │
      ▼
Apply Valid Transition
      │
      ▼
Persist Result
      │
      ▼
Audit Event
```

This provides a mechanism for handling asynchronous payment confirmation.

---

# 🔮 Future Enhancements

Planned areas for future development include:

### 🤖 AI-Powered Routing

Use machine-learning models to dynamically improve gateway selection based on historical transaction behavior.

### 🌍 Global Gateway Support

Expand beyond the current providers to support additional regional and international payment gateways.

### 📈 Predictive Failure Detection

Identify gateway degradation before failure rates become significant.

### 💰 Dynamic Cost Optimization

Optimize routing based on real-time gateway costs in addition to reliability.

### 🧠 Machine Learning Gateway Scoring

Use historical performance data to improve gateway scoring and traffic distribution.

---

# 🧩 Engineering Highlights

### Multi-Gateway Abstraction

Payment-provider-specific implementation is isolated behind an orchestration layer.

### Fault Tolerance

Circuit breakers, retries, failover, and DLQ handling provide multiple levels of failure protection.

### Payment Safety

Idempotency and state-machine validation help prevent duplicate processing and invalid payment transitions.

### Eventual Reconciliation

Webhooks provide asynchronous confirmation and reconciliation of payment states.

### Observability

Prometheus and Grafana provide infrastructure for monitoring transaction and gateway performance.

### Asynchronous Workloads

Celery and Redis provide background processing for tasks that do not need to block the primary request path.

---

# 📋 Key Capabilities

```text
💳 Multi-Gateway Payment Routing
        │
        ▼
🧠 Intelligent Gateway Selection
        │
        ▼
⚡ Low-Latency Failover
        │
        ▼
🔄 Circuit Breaker Protection
        │
        ▼
🔐 Idempotency Management
        │
        ▼
📨 Webhook Reconciliation
        │
        ▼
📒 Audit Trail
        │
        ▼
📊 Real-Time Monitoring
```

---

# 👨‍💻 Author

<div align="center">

### Devansh Negi

**Backend / AI Engineer**

Python • FastAPI • PostgreSQL • Redis • Distributed Systems • AI/ML

[![GitHub](https://img.shields.io/badge/GitHub-devanshnegi88-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/devanshnegi88)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Devansh%20Negi-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/devansh-negi005)

</div>

---

<div align="center">

## 💳 PayFlow

**Reliable payment orchestration with intelligent routing, fault tolerance, and reconciliation.**

</div>
