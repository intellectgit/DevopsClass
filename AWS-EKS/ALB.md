Create a file named `cluster.yaml`:

```yaml

\\\\\\\apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: my-cluster
  region: us-east-1
  version: "1.30"

# This forces both control plane and nodes into private-only architecture
privateCluster:
  enabled: true

managedNodeGroups:
  - name: standard-nodes
    instanceType: t3.medium
    desiredCapacity: 2
    minSize: 1
    maxSize: 3
    ssh:
      allow: true

```

For Cluster created in a private subnet and both nodes are mapping Security group in ( SG-A )
 one node1 created in us-east-1a another node2 created in us-east-1b
  vpc id - ram-vpc  
and cluster created in the ram-private-subnet

Create a Application named `Account`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: account-deployment
  namespace: default
  labels:
    app: account
spec:
  replicas: 2
  selector:
    matchLabels:
      app: account
  template:
    metadata:
      labels:
        app: account
    spec:
      containers:
      - name: account
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: account-service
  namespace: default
  labels:
    app: account
spec:
  type: NodePort
  selector:
    app: account
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP

```

Create a application named `payment`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment
  namespace: default
  labels:
    app: payment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: payment
  template:
    metadata:
      labels:
        app: payment
    spec:
      containers:
      - name: payment
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: payment-service
  namespace: default
  labels:
    app: payment
spec:
  type: NodePort
  selector:
    app: payment
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP


```

Create a Ingress named `Ingress`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: main-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "alb"
    alb.ingress.kubernetes.io/scheme: "internet-facing"
    alb.ingress.kubernetes.io/target-type: "nodeport"
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  rules:
    - http:
        paths:
          - path: /account 
            pathType: Prefix
            backend:
              service:
                name: account-service
                port:
                  number: 80
          - path: /payment
            pathType: Prefix
            backend:
              service:
                name: payment-service
                port:
                  number: 80
				  
```

Same cluster AWS load balancer(application load balancer) is running when I apply 

```bash
kubectl apply -f <deploymenr>
kubectl apply -f <service>
kubectl apply -f <Ingress>

```
---


##  How Alb will create the fallowing details ? 


#  Target group 

  Health check -- health check path , health check timecout , unhealthy threashold , healthcheck timeout 
  selecting the instances (ec21, ec22)  

# ALB 

vpc network 

security group 

Listeners and routing 
       hhtp   - 80
	   forward to target group
	   
	   Map the target group 
	   
creating DNS
 Network mapping 

# Answer

When you apply your deployments, services, and Ingress (`main-ingress`) in your private cluster setup, the **AWS Load Balancer Controller** automatically provisions and maps the infrastructure elements.

Since you defined **two backend services** (`account-service` and `payment-service`) in your Ingress manifest, the controller will create **two separate Target Groups** (one for each service) and map them to a single ALB.

---

## 1. Target Group Creation

Because you specified `alb.ingress.kubernetes.io/target-type: "nodeport"`, the controller creates **Instance-type Target Groups**. Two target groups will be provisioned (e.g., one for `account` and one for `payment`):

* **Health Check Configuration (Default AWS Defaults unless overridden by annotations):**
* **Health Check Path:** `/` (Since your Nginx container returns a `200 OK` on root, it will pass).
* **Health Check Timeout:** `5` seconds.
* **Unhealthy Threshold:** `2` failed consecutive checks before marking a node target unhealthy.
* **Health Check Interval:** `15` seconds.
* **Healthy Threshold:** `2` successful consecutive checks.


* **Selecting the Instances (EC2 Nodes):**
* The controller automatically queries your EKS cluster nodes and registers **`ec21` (in `us-east-1a`)** and **`ec22` (in `us-east-1b`)** into *both* target groups.
* The traffic is targeted to the respective dynamic **NodePort** assigned by Kubernetes to `account-service` and `payment-service`.



---

## 2. ALB Infrastructure & Network Mapping

* **VPC Network (`ram-vpc`):**
* The ALB is created inside your `ram-vpc`.


* **Network Mapping (Subnets):**
* Because your Ingress specifies `alb.ingress.kubernetes.io/scheme: "internet-facing"`, the AWS Load Balancer Controller automatically looks up your VPC configuration and maps the ALB across your **Public Subnets** (spanning `us-east-1a` and `us-east-1b`) so it can accept public incoming traffic from the internet. *(Note: Even though your EKS worker nodes live in a private subnet, the ALB itself sits in public subnets to bridge external clients to private nodes).*


* **Security Group:**
* The controller automatically creates and attaches a security group to the ALB allowing inbound traffic on port `80` from `0.0.0.0/0`. It also configures security group rules between the ALB and your worker nodes' security group (`SG-A`) to allow traffic flow onto the dynamic NodePorts.



---

### 3. Listeners and Routing Rules

* **HTTP Listener (`Port 80`):**
* The controller sets up a listener on the ALB listening on **HTTP Port 80** as requested by your `alb.ingress.kubernetes.io/listen-ports` annotation.


* **Path-Based Routing Rules:**
* The controller reads your Ingress rules and maps them directly into ALB Listener Rules:
* **Rule 1:** If request path matches `/account*` $\rightarrow$ Forward traffic to the **Account Target Group** (which routes to `ec21` and `ec22` on the account NodePort).
* **Rule 2:** If request path matches `/payment*` $\rightarrow$ Forward traffic to the **Payment Target Group** (which routes to `ec21` and `ec22` on the payment NodePort).
* **Default Rule:** Returns a `404 Not Found` if any other path is requested.





---

### 4. DNS Creation

* **External DNS Name:**
* AWS provisions a public DNS name for the ALB (e.g., `k8s-default-mainingre-xxxxxx.us-east-1.elb.amazonaws.com`).
* This DNS name is automatically written back into the status field of your Kubernetes Ingress resource (`kubectl get ingress main-ingress`). You can point a CNAME record from your custom domain (e.g., `app.yourdomain.com`) to this ALB DNS name.


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

## Scenario - 503 ERROR (Service Temparorly Unavailable)

In target group all the nodes are healthy 

 In target group

 node1 - EC2 Instance 1 - us-east-1a
 node2 - EC2 Instance 2 - us-east-1b

 but In ALB network mapping is mapped us-east-1c

 even though the target group 2 instances (node1, node2) are health , in network mapping is no available zone is mapping.

 that's why 503 error 


 ## Complete Flow

 <img width="900" height="1600" alt="WhatsApp Image 2026-09-05 at 5 49 18 PM" src="https://github.com/user-attachments/assets/7c4a5ba2-641a-42b2-91a6-1a0ed87db791" />


 ### 1. Network Issues & Troubleshooting

#### Issue A: Target Group Health Check Failures (`Target.Timeout` or `Unhealthy`)

* **Symptom:** In the AWS EC2 console, targets (`ec21`, `ec22`) show as **Unhealthy** under the target groups, and your application returns a `502 Bad Gateway` error.
* **Cause:** The ALB cannot reach the worker nodes on the dynamically assigned NodePorts. This is usually caused by security group rules blocking traffic or the health check path returning a non-200 code.
* **How to Troubleshoot:**
1. Go to the EC2 Console $\rightarrow$ **Target Groups** $\rightarrow$ select your target group $\rightarrow$ check the **Targets** tab to look at the health reason code (`Timeout`, `Refused`, or `ResponseCodeMismatch`).
2. Verify that Security Group **SG-A** attached to your nodes allows inbound traffic from the **ALB's Security Group** on the dynamic NodePort range (e.g., `30000-32768`).


* **How to Fix:**
* Update Security Group **SG-A** to explicitly allow TCP traffic on port range `30000-32768` originating from the ALB Security Group ID.
* If your Nginx pod root (`/`) path doesn't match a custom health endpoint, add an explicit annotation to your Ingress to match your app:
```yaml
alb.ingress.kubernetes.io/healthcheck-path: "/"

```





#### Issue B: Private Cluster Subnet Routing Failure

* **Symptom:** The ALB is provisioned, but it cannot communicate with the EKS worker nodes.
* **Cause:** Misconfiguration between public subnets (where the internet-facing ALB sits) and private subnets (where `ram-private-subnet` and your nodes live).
* **How to Troubleshoot:**
* Check if the ALB subnets have proper route table entries pointing to an Internet Gateway (`IGW`), and verify that your private subnets have route tables pointing to a NAT Gateway for outbound traffic.


* **How to Fix:** Ensure your Ingress annotation specifies public subnets for an internet-facing ALB, or use internal subnets if traffic is strictly internal.

---

### 2. Common Application Access Issues

#### Issue C: `404 Not Found` When Accessing `/account` or `/payment`

* **Symptom:** The ALB loads successfully, but requests to `http://<alb-dns>/account` return a `404 Not Found`.
* **Cause:** Path mismatch between what the Ingress expects and what your Nginx container is listening to, or incorrect path prefix matching.
* **How to Troubleshoot:**
* Check the ALB Listener Rules via the AWS Console to verify that path patterns `/account*` are correctly mapped to the right Target Group.
* Check if your Nginx container expects just `/` instead of `/account`. If Nginx receives `/account`, it might throw a 404 because the file path doesn't exist inside the container container-side.


* **How to Fix:**
* Use the **Rewrite Target** annotation if your backend application does not support the prefix path natively:
```yaml
annotations:
  alb.ingress.kubernetes.io/rewrite-target: /$2

```


*(And update your Ingress path to use a regex capture group like `/account(/|$)(.*)`)*.



#### Issue D: `503 Service Temporarily Unavailable`

* **Symptom:** The browser returns a `503` error when hitting the ALB URL.
* **Cause:** All targets in the Target Group are currently **Unhealthy**, meaning the ALB has no healthy backend instances/pods to route traffic to.
* **How to Troubleshoot:**
* Run `kubectl get pods -n default` to check if your `account-deployment` and `payment` pods are actually running (`Running` status and `1/1` ready).
* Run `kubectl describe svc account-service` to ensure the NodePort service is matching the correct pod labels (`app: account`).


* **How to Fix:**
* If pods are crashing, fix the container image or resource limits.
* If endpoints are missing, verify that the `selector` in your `Service` definition matches the `labels` in your `Deployment` template exactly.



---



 
