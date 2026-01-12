# 🚀 Migration Tracker - Monitoramento de Migração (VMware -> OpenShift)
## 🚀 Migration Tracker - Migration Monitoring (VMware -> OpenShift)

> **Status:** Active / Stable
> **License:** MIT

**Migration Tracker** is a *self-hosted* solution designed to orchestrate, monitor, and audit the process of migrating workloads from VMware environments to OpenShift (OCP).

Focused on operational transparency and efficiency, it provides real-time metrics, incident management, and state control for each Virtual Machine (VM) during migration waves.

---

## ⚠️ Disclaimer

**This is a community and personal project.**

Although developed by a professional affiliated with Red Hat®, this software:

1. Is **NOT** an official Red Hat Inc. product.
2. Does **NOT** have official technical support (GSS).
3. Is **NOT** covered by corporate warranties or SLAs.

The software is provided "as is" for operational facilitation and learning purposes.

---

## 🛠️ Chosen Technologies

The architecture was designed to be **resilient, lightweight, and easy to deploy** in any environment that supports containers.

### Backend & Core

* **Go (Golang):** Chosen for high performance, static typing, and the ability to generate single, lightweight binaries. The use of the **Gin** framework ensures a fast and efficient RESTful API.
* **GORM:** ORM used to ensure secure database transactions and protection against SQL Injection.

### Database

* **PostgreSQL 15:** Chosen for robustness, reliability, and support for complex queries needed for KPI and audit reports.

### Frontend

* **Server-Side Rendering (Go Templates):** To keep the architecture simple and monolithic (no need for a separate Node.js server), HTML is rendered directly by Go.
* **Bootstrap 5:** Ensures responsiveness and a clean, professional interface without the complexity of heavy JS frameworks.
* **Chart.js:** Lightweight library for rendering KPI charts and business metrics.

### Infrastructure & Security

* **Docker & Docker Compose:** The entire environment spins up with a single command, ensuring parity between development and production. Compatible with **Podman**.
* **Caddy Server (Reverse Proxy):** Acts as the entry point (Gateway). Chosen for automatically managing TLS/SSL certificates (HTTPS), ensuring security from the first deploy.

### Observability

* **Prometheus:** Collects technical metrics (RAM usage, CPU, Go Routines) and custom business metrics (e.g., How many migrations failed in the last 5 minutes).
* **Grafana:** Advanced visualization of data collected by Prometheus, allowing for the creation of executive dashboards.

---

## ✨ Key Features

### 1. 📊 Operational Dashboard

* Overview of all registered VMs.
* Advanced filtering by Cluster, Wave, Status, and Hostname.
* Real-time status updates (Pre-Check, Migrating, Post-Check, Completed, Rollback).

### 2. 📈 KPIs and Business Metrics

* **Success Rate:** Automatic calculation of success percentage vs. rollback.
* **Lead Time:** Analysis of the average time a VM spends in each stage.
* **Bottlenecks:** Charts identifying where the process is stalling (e.g., excessive time in Post-Check).
* **Team Performance:** Ranking of migrations completed by operator.

### 3. 🚨 Incident Management

* Reporting of errors linked to specific VMs.
* Classification by severity (Low, Medium, High, Critical) and Known Errors.
* Resolution workflow with root cause logging.

### 4. 📝 Audit and History

* Every status change or note generates an immutable log.
* Know exactly **who** changed the status, **when**, and **what** was noted.

### 5. ⚡ Productivity Tools

* **Bulk Update:** Update the status of 50 VMs at once (useful for moving an entire batch from "Scheduled" to "Pre-Check").
* **CSV Export:** Download all data for external reports or Excel.
* **Admin CLI:** Built-in command-line tool for administrative tasks (reset database, create users, seed data).

---

## 🚀 How to Run (Quickstart)

### Prerequisites

* Docker and Docker Compose (or Podman + Podman Compose)
* Make (Optional, but recommended)

### Steps

1. **Clone the repository:**
```bash
git clone [https://github.com/your-username/migration-tracker.git](https://github.com/your-username/migration-tracker.git)
cd migration-tracker

```


2. **Spin up the environment:**
We use a Makefile to simplify commands.
```bash
make up

```


*This will compile the API, download the Postgres, Grafana, Prometheus, and Caddy images, and start everything.*
3. **Populate the database (First time):**
Run the setup script that creates tables and default users:
```bash
make setup-prod

```


4. **Access:**
* **Application:** `https://localhost:8443` (Accept the self-signed certificate)
* **Grafana:** `http://localhost:3000`
* **Prometheus:** `http://localhost:9090`



---

## 🔐 Default Credentials (Setup)

If you ran `make setup-prod`, the created users will be:

| User | Password | Profile |
| --- | --- | --- |
| `admin` | `1q2w3e` | Administrator |
| `lagomes` | `1q2w3e` | Administrator |
| `liz` | `1q2w3e` | Standard User |

---

## 🛠️ Administrative Commands (Makefile)

The project includes shortcuts in the `Makefile` to perform maintenance, backup, and restore tasks locally via the administrative CLI.

**Prerequisite:** Ensure you have the `.env` file configured in the project root.

| Command | Description | Usage Example |
| --- | --- | --- |
| `make cli-setup` | **Full Setup:** Initializes the database, populates the error catalog, and creates default users. | `make cli-setup` |
| `make cli-backup` | Generates a full backup (`.tar.gz`) of the database in the project root. | `make cli-backup` |
| `make cli-backup-custom` | Generates a backup in a specific directory. | `make cli-backup-custom PATH_BKP=/tmp/backups` |
| `make cli-restore` | Restores the database from a backup file. | `make cli-restore FILE=/tmp/backup_2025.tar.gz` |
| `make cli-seed` | **Reset & Populate:** Clears the entire database and generates random test data. | `make cli-seed` |
| `make cli-users` | Recreates only the default users listed above. | `make cli-users` |

---

## 📂 Project Structure

```text
.
├── cmd/api             # API Entry Point (Main)
├── cmd/admin           # Administrative CLI tool
├── internal
│   ├── handlers        # Controller Logic (HTTP)
│   ├── models          # Data Structures (Structs)
│   ├── database        # GORM Connection and Queries
│   └── middleware      # Authentication and Prometheus Metrics
├── templates           # HTML Files (Frontend)
├── static              # CSS, JS, and Images
├── grafana             # Pre-configured Dashboards (JSON)
└── docker-compose.yml  # Container Orchestration


```

---

## 🤝 Contribution

Contributions are welcome! Feel free to open Issues or Pull Requests.

1. Fork the project.
2. Create a Branch for your Feature (`git checkout -b feature/AmazingFeature`).
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`).
4. Push to the Branch (`git push origin feature/AmazingFeature`).
5. Open a Pull Request.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://www.google.com/search?q=LICENSE) file for details.

Copyright (c) 2025 Lauro de Paula