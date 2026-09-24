# 🛡️ Distributed Location-Aware Threat Intelligence Platform

[![Azure](https://img.shields.io/badge/Cloud-Microsoft%20Azure-0078D4?logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/)
[![Suricata](https://img.shields.io/badge/IDS-Suricata%207.0-EF3B2C?logo=suricata&logoColor=white)](https://suricata.io/)
[![Python](https://img.shields.io/badge/Language-Python%203.11+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Docker](https://img.shields.io/badge/Deployment-Docker%20Compose-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Redis](https://img.shields.io/badge/Queue-Redis%207.0-DC382D?logo=redis&logoColor=white)](https://redis.io/)
[![PostgreSQL](https://img.shields.io/badge/Storage-PostGIS%20%2F%20Postgres-336791?logo=postgresql&logoColor=white)](https://postgis.net/)
[![React](https://img.shields.io/badge/Frontend-React%20%7C%20Leaflet-61DAFB?logo=react&logoColor=black)](https://react.dev/)

A distributed, enterprise-grade Threat Intelligence (TI) and Security Operations pipeline deployed across isolated **Microsoft Azure virtual machines**. Features real-time attack detection via Suricata IDS, resilient log shipping via Filebeat, multi-source IOC enrichment, geospatial mapping, and an interactive dark-mode analyst dashboard with **3–5 second end-to-end detection latency**.

---

## 🏗️ Distributed Architecture

```text
[ Threat Actor / Ingress Traffic ]
               │
               ▼
┌────────────────────────────────────────────────────────┐
│ VM-1: Attack Sensor & Honeypot Tier (Azure VM)         │
│  • OWASP Juice Shop (Target Application)               │
│  • Suricata IDS (AF-PACKET Live Capture & Rules)       │
│  • Filebeat (EVE JSON Parsing & Log Forwarding)        │
└───────────────────────────┬────────────────────────────┘
                            │ (Redis Protocol / Port 6379)
                            ▼
┌────────────────────────────────────────────────────────┐
│ VM-2: Ingestion Queue & TI Fetcher Tier (Azure VM)     │
│  • Redis Broker (Queue: suricata-events)               │
│  • Threat Intel Fetcher (VirusTotal, AbuseIPDB, OTX)   │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ VM-3: Stream Processor & Correlation Engine (Azure VM) │
│  • Event Normalizer & Deduplicator                     │
│  • Geolocation Enricher (IP-to-City/Country Lat/Long)  │
│  • Threat Correlator & Risk Scoring                    │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ VM-4: Persistent Spatial Database Tier (Azure VM)      │
│  • PostgreSQL 16 + PostGIS (Geospatial Indexing)       │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ VM-5: Analyst Dashboard & Presentation Tier (Azure VM) │
│  • FastAPI Backend (RESTful Endpoints & WebSockets)    │
│  • React + Leaflet (Interactive Neon Global Map)       │
│  • Nginx Reverse Proxy                                 │
└────────────────────────────────────────────────────────┘
```

---

## 🏛️ VM Tier Breakdown & Documentation

| Tier | Role & Technologies | Detailed Docs |
| :--- | :--- | :--- |
| **VM-1** | **Attack Sensor:** Suricata IDS (AF-PACKET), Filebeat 8.x, OWASP Juice Shop | [VM-1 Docs](./vm-1-sensor/readme.md) |
| **VM-2** | **Ingestion Queue:** Redis Message Broker, TI Cache, Threat Intel Fetcher | [VM-2 Docs](./vm-2-ingest-queue/readme.md) |
| **VM-3** | **Event Processor:** Event Normalization, GeoIP Enrichment, Correlator | [VM-3 Docs](./vm-3-processor/readme.md) |
| **VM-4** | **Database:** PostgreSQL with PostGIS extension for geospatial queries | [VM-4 Docs](./vm-4-database/readme.md) |
| **VM-5** | **UI & Notifier:** FastAPI REST API, React.js, Leaflet Map, Nginx Proxy | [VM-5 Docs](./vm-5-ui-notifier/readme.md) |

---

## ✨ Key Features & Detection Capabilities

* **Real-Time Network Intrusion Detection:** Suricata IDS tuned with `AF-PACKET` high-throughput packet capture.
* **15+ Custom Suricata Signatures:** Covers OWASP Top 10 attack classes:
  * SQL Injection (SQLi)
  * Cross-Site Scripting (XSS)
  * Path & Directory Traversal
  * Authentication Brute-Force
  * Network Reconnaissance & Port Scanning
* **Automated Threat Intelligence Enrichment:** Enriches inbound attacker IPs with reputation and maliciousness scores from **VirusTotal**, **AbuseIPDB**, and **AlienVault OTX**.
* **Geospatial Threat Visualization:** Real-time interactive global attack map using **Leaflet.js** and PostGIS spatial indexing.
* **Ultra-Low Latency:** Achieves **3–5 second end-to-end alert propagation** from initial network packet capture on VM-1 to visual rendering on VM-5.

---

## 🚀 Deployment Guide

### Prerequisites
* 5 Linux VMs (Ubuntu 22.04/24.04 LTS, minimum 2 vCPU / 4 GB RAM recommended per VM).
* Docker & Docker Compose installed on all VMs.
* API Keys: VirusTotal, AbuseIPDB, AlienVault OTX.
* *Note: For local testing, all five tiers can be run on a single host machine.*

### 🔷 Correct Service Startup Order (Strict)
Because data consumers depend on running message queues and storage backends, deploy in the following order:

```bash
# Step 1: Start Redis Broker on VM-2
cd vm-2-ingest-queue && docker-compose up -d

# Step 2: Start PostgreSQL / PostGIS on VM-4
cd vm-4-database && docker-compose up -d

# Step 3: Start Normalizer & Correlator on VM-3
cd vm-3-processor && docker-compose up -d

# Step 4: Start Suricata Sensor & Filebeat on VM-1
cd vm-1-sensor && docker-compose up -d

# Step 5: Start API & Analyst Dashboard on VM-5
cd vm-5-ui-notifier && docker-compose up -d
```

---

## 🔒 Security Best Practices for Production
* **Network Security Groups (NSGs):** Restrict Redis (6379) and Postgres (5432) access strictly to the private IPs of the processing VM.
* **Secrets Management:** Store all API keys and database credentials in `.env` files (never commit to git).
* **TLS Encryption:** Terminate HTTPS on Nginx in front of the dashboard and FastAPI.
