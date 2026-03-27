<div align="center">

  <img src="./assets/header.gif" width="100%" style="border-radius: 12px; box-shadow: 0 8px 16px rgba(0,0,0,0.4);" alt="Event Horizon" />

  # I'm Hlib Shloma | Python Backend Engineer
  Architecting for resilience, scalability, and real-world impact.

  > "The world hasn't ended yet."

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

## 🆒 Projects

### 🛡️ [Watchdog HTTP](https://github.com/DarcRosya/watchdog-http)
**Autonomous Asynchronous Web-Monitoring System**

A resilient, fully decoupled background monitoring system that continuously runs HTTP health checks, stores time-series metrics, and dispatches real-time alerts.

* **Architecture:** Engineered a robust background check loop using **ARQ (Redis)** and **FastAPI**, with **TimescaleDB** for efficient time-series data storage.
* **Reliability:** Implemented custom anti-flapping and state-transition logic to prevent alert storms during transient network failures.
* **Observability:** Integrated real-time Telegram alerts via **aiogram**, a **Streamlit** dashboard for historical trends, and structured JSON logging.
* **Code Quality:** Maintained high reliability with **200+ unit/integration tests**, strict `mypy` typing, and fully containerized via **Docker Compose**.

**Stack:** `Python` `FastAPI` `ARQ` `TimescaleDB` `Redis` `Docker`

<br/>
