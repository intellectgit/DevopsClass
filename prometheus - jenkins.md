install the prometheus plugin 

prometheus metrics plugin

scrape_config
===================
Use scrape_config if you are running Prometheus manually (e.g., standard Docker containers, virtual machines, or static environments) 
where Kubernetes Custom Resources are unavailable.
where it live -
Inside the Prometheus configuration file (prometheus.yml).

ServiceMonitor
==============
Use ServiceMonitor if your application is deployed on Kubernetes alongside the Prometheus Operator, 
as it automates target lifecycle management and keeps your configurations modular.

where it live - As a standalone Kubernetes Custom Resource (YAML manifest applied via kubectl).
service monitor 

Real-Time Use Case: Dynamic E-Commerce Microservices
=========================================================
Imagine you are managing an online retail platform running on a Kubernetes cluster that experiences massive traffic spikes during flash sales.

Without Prometheus Operator (Vanilla Approach)
-----------------------------------------------------
The Challenge: Your development team deploys 20 new microservices for an upcoming sale, and each service constantly scales up its number of Pod replicas based on user load.

The Pain Point: Every time a new microservice instance or pod is spawned, its IP address changes. To monitor it, a platform engineer must manually update the prometheus.yml file or reconfigure a ConfigMap with the new endpoints, then trigger a reload of the Prometheus server.

The Risk: If an autoscaling event happens in the middle of the night, Prometheus misses metrics from the new pods because they aren't written in the static configuration, creating blind spots during high-stakes incidents.

With Prometheus Operator (Kubernetes-Native Approach)
-----------------------------------------------------------
The Setup: You define a ServiceMonitor custom resource once, configured to look for any service labeled app: e-commerce and scrape the /metrics endpoint.

The Real-Time Workflow:

A flash sale triggers a sudden surge in traffic, and Kubernetes automatically scales your checkout-service from 5 pods to 50 pods in seconds.

The Prometheus Operator dynamically detects the newly created pods through the Kubernetes API without any manual intervention.

It instantly updates the internal scrape configuration, automatically pulling real-time latency, CPU, and transaction metrics from all 50 active pods.

The Result: Your Grafana dashboard reflects real-time resource utilization and error rates of the exact running pods instantly, allowing your team to monitor system health seamlessly during critical scaling events.


Prometheus-operator configuration via service monitor 
----------------------------------------------------------

