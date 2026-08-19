# Kubernetes Monitoring Guide: Prometheus, ServiceMonitor, PodMonitor, & Alertmanager

A comprehensive guide explaining core Prometheus concepts, configuration comparisons, Kubernetes-native operators, and real-time incident workflows during high-traffic events (such as an e-commerce flash sale).

---

## Table of Contents
1. [Understanding Prometheus Scrape Configurations](#1-understanding-prometheus-scrape-configurations)
2. [Scrape Config vs. ServiceMonitor](#2-scrape-config-vs-servicemonitor)
3. [Vanilla Prometheus vs. Prometheus Operator](#3-vanilla-prometheus-vs-prometheus-operator)
4. [Real-Time Scenario: E-Commerce Flash Sale](#4-real-time-scenario-e-commerce-flash-sale)
5. [The Role of PodMonitor & Alertmanager in Action](#5-the-role-of-podmonitor--alertmanager-in-action)

---

## 1. Understanding Prometheus Scrape Configurations

A **scrape configuration (`scrape_configs`)** in Prometheus defines how, where, and how often Prometheus collects metrics from monitored systems and applications. Because Prometheus operates on a **pull-based model** (actively fetching data via HTTP GET requests rather than waiting for apps to push data), the scrape configuration acts as the core blueprint directing the Prometheus server on what targets to watch.

### Core Elements of a Scrape Config
A typical block in the `prometheus.yml` file contains several fundamental parameters:
* **`job_name`**: A unique identifier for the scrape job. Used to label and organize metrics gathered from that specific group of sources.
* **`scrape_interval`**: How frequently Prometheus pulls metrics from targets (e.g., `15s` for every 15 seconds).
* **`scrape_timeout`**: The maximum duration Prometheus will wait for a target to respond before marking the scrape as failed.
* **`metrics_path`**: The HTTP resource path where the application or exporter exposes its metrics (defaults to `/metrics`).
* **`scheme`**: The protocol used for scraping, typically `http` or `https`.
* **`static_configs` or Service Discovery**: Defines exact endpoints or dynamic resources to target.

### Example Scrape Configuration
```yaml
scrape_configs:
  - job_name: 'spring-boot-app'
    scrape_interval: 15s
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']
    labels:
      environment: 'production'
