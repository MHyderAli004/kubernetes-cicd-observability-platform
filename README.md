# Sentinel: Self-Verifying Kubernetes CI/CD & Observability Platform

**Sentinel** is a production-grade, three-tier web application platform deployed on Kubernetes (K3s). It features a fully automated Jenkins CI/CD pipeline that verifies its own deployments, backed by a comprehensive Prometheus/Alertmanager observability stack that pages Slack in real-time when cluster health degrades.

![Architecture](https://img.shields.io/badge/Architecture-3--Tier-blue)
![Platform](https://img.shields.io/badge/Platform-K3s%20on%20AWS-FF9900)
![CI/CD](https://img.shields.io/badge/CI%2FCD-Jenkins-D24939)
![Monitoring](https://img.shields.io/badge/Monitoring-Prometheus%20%7C%20Alertmanager-E6522C)

---

## 🏗 Architecture Diagram

```text
                     +-------------------+
                     |     Jenkins       |
                     | (CI/CD & Ops)     |
                     +---------+---------+
                               | build, push, deploy, verify
                               v
        +---------------------------------------------------------+
        |                    K3s Cluster (AWS EC2)                |
        |                                                         |
        |  +----------+    +-----------+    +------------------+  |
        |  | Frontend |===>|  Backend  |===>|  MySQL Database  |  |
        |  |  (Nginx) |    |  (Java)   |    | (Stateful Pod)   |  |
        |  +----------+    +-----------+    +------------------+  |
        |        ^                                                  |
        |        | Scrape Metrics                                   |
        |  +-----+------+  +--------------+  +------------------+   |
        |  | Prometheus |=>| Alertmanager |=>| Slack (#alerts)  |   |
        |  +-----+------+  +--------------+  +------------------+   |
        +---------------------------------------------------------+

<img width="721" height="615" alt="slack alert" src="https://github.com/user-attachments/assets/81da118c-8de2-42ce-8ad8-df6e3e142cd9" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/f06dc80e-1dae-47ab-bc5e-dcfb6c127eb5" />

