Service - myapp-service
Ingress - myapp-ingress


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

