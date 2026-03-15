# Project Sentinel: Full-Stack Observability & Resilience Lab

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Ansible](https://img.shields.io/badge/ansible-%231A1918.svg?style=for-the-badge&logo=ansible&logoColor=white)
![ElasticSearch](https://img.shields.io/badge/-ElasticSearch-005571?style=for-the-badge&logo=elasticsearch)
![Kibana](https://img.shields.io/badge/KIBANA-005571?style=for-the-badge&logo=kibana&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white)
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![YAML](https://img.shields.io/badge/yaml-%23ffffff.svg?style=for-the-badge&logo=yaml&logoColor=151515)

## Overview
This laboratory environment simulates a production-grade observability stack built from scratch on an Ubuntu Server. 
Instead of relying on cloud-managed services, this project demonstrates hands-on experience in provisioning infrastructure (Ansible), containerizing services (Docker), generating synthetic traffic (DDoS simulation), and monitoring both logs and hardware metrics in real-time.

    ![Architecture & Data Flow](images/sentinel-architecture.png)

## Features
- **Infrastructure as Code (IaC):** Automated Docker provisioning and service management using Ansible.
- **Advanced Log Analytics:** Real-time ingestion of Nginx access/error logs via Filebeat to Elasticsearch. Custom Kibana Dashboards (KQL) identify botnet traffic (`fasthttp`) and monitor HTTP status codes.
- **Deep Observability:** Prometheus continuously scrapes hardware metrics via Node Exporter and container-level metrics via cAdvisor. Leveraged industry-standard community Grafana dashboards (IDs 1860 and 14282) for rapid and reliable metric visualization, while building custom KQL dashboards in Kibana from scratch.
- **Active Probing:** Blackbox Exporter continuously checks the HTTP endpoint health of the Nginx server.
- **Automated Incident Response:** Alertmanager evaluates PromQL rules (e.g., `CPU > 80%` or `probe_success == 0`) and automatically fires webhook notifications directly to a Discord channel.

## Tech Stack
- **OS & Virtualization:** Ubuntu Server 24.04 LTS (VMware Workstation Pro)
- **Automation & Containerization:** Ansible, Docker, Docker Compose
- **Web & Traffic:** Nginx, Bombardier (HTTP load generator)
- **Log Pipeline (ELK):** Filebeat, Elasticsearch, Kibana
- **Metrics & Alerting:** Prometheus, Grafana, Node Exporter, cAdvisor, Blackbox Exporter, Alertmanager

## Key Results (Screenshots)

### 1. Security Operations Center (Kibana)
Visualizing 13.6M logs during a simulated DDoS attack. Bombardier traffic (100%) is successfully isolated from normal users, with HTTP 200/404 ratios tracked.
![Kibana Dashboard](images/kibana-sentinel-dashboard.png)

### 2. Infrastructure Meltdown (Grafana)
Monitoring a critical CPU spike (95.6%) triggered by the Bombardier stress test, using Node Exporter and cAdvisor dashboards.
![Node Exporter Dashboard](images/grafana-node-exporter-dashboard.png)

![cAdvisor Dashboard](images/grafana-cadvisor-dashboard.png)

### 3. Incident Alerting (Discord Webhook)
Real-time critical alerts delivered from Prometheus/Alertmanager directly to Discord during the attack.
![Discord Alerts](images/discord-alerts.png)

### 4. Automated Provisioning (Ansible)
Idempotent Docker installation via `install-docker.yml`.
![Ansible Playbook](images/ansible-docker-install-completed.png)

## Usage / Reproducing the Lab
*Note: All configuration files (`docker-compose.yml`, `prometheus.yml`, etc.) are included in this repository.*

1. **Install Docker via Ansible:**
   ansible-playbook install-docker.yml -K
2. **Deploy the full observability stack:**
   docker compose up -d
3. **Trigger the DDoS simulation (Bombardier):**
  for i in {1..4}; do sudo docker run -d --rm alpine/bombardier -c 1000 -d 300s http://<YOUR_IP>:80; done
4. **Access the dashboards:**
- *Grafana:* http://<YOUR_IP>:3000
- *Kibana:* http://<YOUR_IP>:5601
- *Prometheus Alerts:* http://<YOUR_IP>:9090/alerts
