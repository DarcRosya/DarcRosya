<div align="center">

  <img src="./assets/header.gif" width="100%" style="border-radius: 12px; box-shadow: 0 8px 16px rgba(0,0,0,0.4);" alt="Black Hole Header" />

  <h1>I'm Hlib Shloma | Python Backend Engineer</h1>
  <p>Architecting for resilience, scalability, and real-world impact.</p>

  <blockquote>
    <p>The world hasn't ended yet.</p>
  </blockquote>

</div>

<h3 align="center">Core Tech Stack</h3>

<div align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white" />
  <img src="https://img.shields.io/badge/SQLAlchemy-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white" />
  <br/>

  <img src="https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white" />
  <img src="https://img.shields.io/badge/TimescaleDB-FDB515?style=for-the-badge&logo=timescale&logoColor=black" />
  <img src="https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white" />
  <img src="https://img.shields.io/badge/ARQ-000000?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Celery-37814A?style=for-the-badge&logo=celery&logoColor=white" />
  <br/>

  <img src="https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=prometheus&logoColor=white" />
  <img src="https://img.shields.io/badge/Grafana-F46800?style=for-the-badge&logo=grafana&logoColor=white" />
  <img src="https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" />
</div>

<br/>

## Featured Engineering Projects

### 🛡️ [Watchdog HTTP](https://github.com/DarcRosya/watchdog-http) | Event-Driven Uptime Monitoring and Incident Alerting Platform

Watchdog HTTP is an asynchronous monitoring platform that continuously probes HTTP/S endpoints, persists time-series check history, and dispatches incident alerts through Telegram. It is designed to reduce detection lag, suppress alert noise, and keep incident communication reliable under unstable network conditions.

> **Business Value:** Reduces mean time to detection and lowers outage impact by converting raw endpoint behavior into actionable incident workflows for operators.<br/>
> **Operational Indicators (Streamlit UI):** <kbd>Rate</kbd> scheduled and executed checks over time, <kbd>Errors</kbd> failed checks and incident count, <kbd>Duration</kbd> probe duration (duration_ms) and latency trends from persisted check logs.

#### Architecture

FastAPI manages monitor lifecycle, API-key auth, and stats endpoints while Redis serves as both the ARQ broker and operational state store. A scheduler worker scans due monitors from a Redis sorted set and enqueues probe jobs; monitoring workers execute async HTTP/TLS checks and persist results in TimescaleDB/PostgreSQL; alerting workers send Telegram notifications from a separate queue so delivery latency never backpressures monitoring. Operator visibility is delivered through persisted logs/history and Streamlit dashboards.

<table>
  <tr>
    <th align="left">Engineering Focus</th>
    <th align="left">Implementation</th>
  </tr>
  <tr>
    <td><strong>Queue-Segmented Fault Isolation</strong></td>
    <td>Monitoring and alerting run in separate ARQ queues with independent retry budgets, isolating probe throughput from notification channel instability.</td>
  </tr>
  <tr>
    <td><strong>Anti-Flapping State Intelligence</strong></td>
    <td>Consecutive failure counters and transition-aware rules suppress transient noise while preserving true first-failure and valid recovery alerts.</td>
  </tr>
  <tr>
    <td><strong>Resilient Cache-First Path</strong></td>
    <td>Monitor configuration is fetched from Redis first with database fallback and automatic cache repopulation for continuity under cache churn.</td>
  </tr>
  <tr>
    <td><strong>Throughput-Oriented Async Runtime</strong></td>
    <td>Pooled async HTTP clients, explicit timeout budgets, and scheduler caps sustain high-volume checks without unbounded queue growth.</td>
  </tr>
  <tr>
    <td><strong>Operational Forensics by Design</strong></td>
    <td>Each check persists start time, duration, status code, success flag, and error context, enabling reliable post-incident reconstruction.</td>
  </tr>
</table>

<br/>

### ⚙️ [OrderFlow Resilience Engine](https://github.com/DarcRosya/saga-orchestrator-pattern) | Distributed Saga Orchestrator

OrderFlow Resilience Engine coordinates distributed order processing across Billing, Inventory, and Logistics using a strict saga state machine with deterministic compensation. It eliminates partial-commit drift by enforcing idempotent intake, bounded retries, and recovery workflows for long-running failures.

> **Business Value:** Prevents financial mismatch, inventory distortion, and fulfillment leakage when downstream services fail after order acceptance.<br/>
> **RED + Reliability Metrics (Prometheus/Grafana):** <kbd>Rate</kbd> saga_execution_total and saga_step_execution_total, <kbd>Errors</kbd> compensation failures and manual-intervention backlog, <kbd>Duration</kbd> saga_step_duration_seconds and API p99 latency.

#### Architecture

Requests enter through Nginx to FastAPI, where payloads are validated, idempotency keys are enforced, and order aggregates are persisted in PostgreSQL. Redis-backed ARQ queues drive asynchronous saga execution: billing runs first, then inventory and logistics execute in parallel; any failure transitions the order to compensating flow. Scheduler workers detect stale processing/compensation states, re-dispatch recovery, and escalate unresolved workflows to manual intervention. Prometheus metrics and Grafana dashboards provide business and platform observability across API and workers.

<table>
  <tr>
    <th align="left">Engineering Focus</th>
    <th align="left">Implementation</th>
  </tr>
  <tr>
    <td><strong>Selective Compensation with Consistency Control</strong></td>
    <td>Rollback acquires row-level locks and derives reverse actions from persisted step state, preventing over-compensation and race-driven double rollbacks.</td>
  </tr>
  <tr>
    <td><strong>Bounded Retry and Escalation</strong></td>
    <td>Failed compensations are re-enqueued with deferred retries and retry counters, then escalated to manual intervention after strict limits.</td>
  </tr>
  <tr>
    <td><strong>Autonomous Stale-Saga Recovery</strong></td>
    <td>Scheduler polling identifies stale processing or compensating orders and triggers recovery using timestamped job IDs to avoid dedup collisions.</td>
  </tr>
  <tr>
    <td><strong>Idempotent Order Intake</strong></td>
    <td>Unique idempotency keys convert duplicate create attempts into deterministic reads, blocking duplicate financial and fulfillment execution paths.</td>
  </tr>
  <tr>
    <td><strong>Metrics Grounded in System Truth</strong></td>
    <td>Manual-intervention gauge values are synchronized from database state, avoiding drift across retries, admin actions, and out-of-band updates.</td>
  </tr>
</table>

<br/>

## Contact & Connect

<div align="center">
  <table>
    <tr>
      <td align="center" width="33%">
        <strong>LinkedIn</strong><br/>
        <a href="https://www.linkedin.com/in/hlib-shloma-a2a039348">https://www.linkedin.com/in/hlib-shloma-a2a039348</a>
      </td>
      <td align="center" width="33%">
        <strong>Email</strong><br/>
        <a href="mailto:hlib.shloma.work@gmail.com">hlib.shloma.work@gmail.com</a>
      </td>
      <td align="center" width="33%">
        <strong>Telegram</strong><br/>
        <a href="https://t.me/hlib_dev">@hlib_dev</a>
      </td>
    </tr>
  </table>
</div>

<blockquote>
  Open to backend platform engineering, distributed systems, and reliability-focused product teams.
</blockquote>
