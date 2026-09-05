Service - myapp-service
Ingress - myapp-ingress


When you use a **NodePort service** (`myapp_service`) tied to an Ingress (`myapp_ingress`), the **AWS Load Balancer Controller** automates the AWS infrastructure setup using **Instance-type Target Groups**.

The step-by-step traffic and provisioning flow unfolds through these stages:

---

### 1. Target Group Creation & Worker Node Registration

* **Target Type:** Because your backend is a NodePort service, the controller creates a Target Group with a **Target Type of `instance**`.
* **The Targets:** Instead of pointing directly to the individual application pods, the Target Group registers **all your EKS worker nodes (EC2 instances)** using their private EC2 IP addresses.
* **The Port:** The target group port is set to the dynamically allocated **NodePort** of your `myapp_service` (e.g., `31234`).

### 2. Listener Creation

* **The Port:** The controller reads the frontend listener configuration from your Ingress annotations (or defaults to port `80` for HTTP or `443` for HTTPS).
* **The Action:** It creates a corresponding **Listener** on the ALB for that port (e.g., an HTTP Listener listening on port `80`).

### 3. Listener Rules / Path Routing

* **The Paths:** The controller inspects the `spec.rules` inside your `myapp_ingress` manifest (such as path `/app` or host `myapp.example.com`).
* **The Action:** It creates a **Listener Rule** tied to the ALB Listener. This rule tells the ALB: *"If incoming traffic matches this specific path or host, forward it to the NodePort Target Group."*

---

### The Complete Traffic Flow

When a user hits your application, the request travels through this chain:

```text
[Internet User / Client]
         │
         ▼
[ALB Listener (Port 80/443)] ──(Applies Ingress Rules / Path Routing)
         │
         ▼
[Instance-Type Target Group] ──(Points to Worker Nodes via NodePort, e.g., :31234)
         │
         ▼
[EKS Worker Node IP:NodePort] ──(Picked up by kube-proxy)
         │
         ▼
[Kubernetes NodePort Service (myapp_service)]
         │
         ▼
[Your Application Pod]

```
