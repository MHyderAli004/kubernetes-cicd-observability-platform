# Sentinel · Self-Verifying Kubernetes CI/CD & Observability Platform

[![CI](https://img.shields.io/badge/CI-Jenkins-blue)](https://jenkins.io)
[![K8s](https://img.shields.io/badge/K8s-K3s-326ce5)](https://k3s.io)
[![Monitoring](https://img.shields.io/badge/Monitoring-Prometheus%20%7C%20Alertmanager-orange)](https://prometheus.io)
[![Alerts](https://img.shields.io/badge/Alerts-Slack-green)](https://slack.com)

**Sentinel** is a production-grade DevOps platform: a three-tier application
(Nginx → Java/JDBC → MySQL) on Kubernetes (K3s, AWS EC2), delivered by a Jenkins
pipeline that *verifies its own deployments*, and watched by a Prometheus stack
that pages Slack the moment anything degrades.

> Named *Sentinel* because the platform stands guard over itself:
> no build can claim success unless pods are proven healthy, and no failure
> happens silently.

## 🎬 Demo: Crew Onboarding app (screen recording)

<video src="Assets/crew-onboarding-app-ui.mp4" controls width="720"></video>

*If the player doesn't render, [watch it here](Assets/crew-onboarding-app-ui.mp4).*

---

## 🏗 Architecture

```text
        git push                +------------------+
   ────────────────▶            |     Jenkins      |
                                | build·test·push  |
                                | validate·deploy  |
                                | verify rollout   |
                                +--------+---------+
                                         | kubectl apply -f k8s/
                                         ▼
   +-------------------------------------------------------------+
   |                     K3s cluster (EC2)                       |
   |                                                             |
   |  +----------+   +----------+   +-----------+                |
   |  | frontend |──▶| backend  |──▶|   mysql   |  3-tier app    |
   |  |  nginx   |   | java/jdbc|   |  (PVC)    |                |
   |  +----------+   +----------+   +-----------+                |
   |        ▲                                                    |
   |        │ scrape                                             |
   |  +------------+   +--------------+   +-------------+        |
   |  | Prometheus |──▶| Alertmanager |──▶| Slack alerts|        |
   |  +------------+   +--------------+   +-------------+        |
   |   node-exporter · kube-state-metrics                        |
   +-------------------------------------------------------------+
