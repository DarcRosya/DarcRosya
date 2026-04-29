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

Watchdog HTTP is an asynchronous uptime and HTTP performance monitoring system built around a FastAPI API and ARQ background workers. It continuously probes HTTP/S endpoints, persists time-series check history, and dispatches incident alerts through Telegram while suppressing alert noise under unstable network conditions.

> **Business Value:** Reduces mean time to detection and lowers outage impact by converting raw check data into actionable, deduplicated incident workflows for operators.<br/>
> **Operational Indicators (Prometheus + Grafana):** <kbd>Rate</kbd> scheduled checks, executed checks, and queue depth, <kbd>Errors</kbd> failed checks and alert count, <kbd>Duration</kbd> probe latency and worker runtime, with worker metrics pushed via Pushgateway.

#### Architecture

FastAPI manages the monitor lifecycle, API-key authentication, and telemetry endpoints. Redis functions dually as the ARQ message broker and the operational state store (caching monitor configs, failure counters, and last-check timestamps).

To ensure high availability and fault isolation, the background workload is segmented:
* **Scheduler:** Scans a Redis sorted set for due monitors and enqueues probe jobs with strict rate limits to prevent thundering herd issues.
* **Monitoring Workers:** Execute asynchronous HTTP/TLS probes via `httpx`, persist time-series `ResultLog` data in TimescaleDB, and update the shared Redis state.
* **Alerting Workers:** Operate on an entirely separate queue. This critical design choice guarantees that external delivery latency (e.g., Telegram API rate limits or outages) never backpressures the core monitoring throughput.

**Observability Stack:** Prometheus actively scrapes metrics from the FastAPI layer, while ephemeral ARQ workers push custom telemetry to a Pushgateway. All data is aggregated and visualized in Grafana.

*C4 diagrams (C1, C2-Core, C2-Observability, C3-API) are available in the repository to rigorously document the system context, runtime components, observability pipeline, and API boundaries.*

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
    <td>Failure counters with TTL and transition-aware rules suppress transient noise while preventing stale recovery alerts.</td>
  </tr>
  <tr>
    <td><strong>Distributed Cache Hydration Lock</strong></td>
    <td>Redis SET NX lock prevents concurrent cache rehydration; monitor config cache falls back to the database and repopulates on miss.</td>
  </tr>
  <tr>
    <td><strong>Backlog-Aware Scheduling</strong></td>
    <td>Scheduler caps due checks per tick and records backlog and queue depth metrics to avoid runaway growth.</td>
  </tr>
  <tr>
    <td><strong>SSL Expiry Guardrails</strong></td>
    <td>HTTPS monitors are checked at most once per day with Redis TTL dedupe to avoid alert spam.</td>
  </tr>
  <tr>
    <td><strong>Operational Forensics by Design</strong></td>
    <td>Each check persists start time, duration, status code, success flag, and error context for post-incident reconstruction.</td>
  </tr>
</table>

<br/>

### ⚙️ [OrderFlow Resilience Engine](https://github.com/DarcRosya/saga-orchestrator-pattern) | Distributed Saga Orchestrator

OrderFlow Resilience Engine coordinates distributed order processing across Billing, Inventory, and Logistics using a strict saga state machine with deterministic compensation. It eliminates partial-commit drift by enforcing idempotent intake, bounded retries, and recovery workflows for long-running failures.

> **Business Value:** Prevents financial mismatch, inventory distortion, and fulfillment leakage when downstream services fail after order acceptance.<br/>
> **RED + Reliability Metrics (Prometheus + Grafana):** <kbd>Rate</kbd> saga_execution_total and saga_step_execution_total, <kbd>Errors</kbd> compensation failures and manual-intervention backlog, <kbd>Duration</kbd> saga_step_duration_seconds and API p99 latency.

#### Architecture

Requests enter through Nginx to FastAPI, which handles payload validation, enforces idempotency keys, and persists order aggregates in PostgreSQL. Redis functions as the ARQ message broker driving the distributed state machine.

To ensure strict consistency and recovery, the workload is divided:
* **Saga Worker:** Executes the primary workflow (billing runs first, followed by concurrent inventory and logistics tasks). Any failure instantly transitions the order into a deterministic compensation flow (rollback).
* **Scheduler Worker:** Runs autonomous reconciliation loops to detect stale states or stuck compensations, re-dispatches recovery tasks, and escalates exhausted workflows to a manual intervention queue.

**Observability Stack:** Prometheus actively scrapes system health and business RED metrics from the FastAPI layer, background workers, and infrastructure exporters (Postgres, Redis, Node). The entire pipeline is visualized in Grafana.

*C4 diagrams (C1, C2-Core, C2-Observability) and the detailed architectural document are available in the repository.*

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
