pod is running there is no issue with deployment , but user getting 503 error , even there is no issue in the grafana dashboard ? who do you triage this issue ?

If the pod status is Running (no crashes), Grafana shows no resource spikes or application error logs, but users are getting HTTP 503 (Service Unavailable) errors, this points directly to a networking, traffic routing, or health check/probe failure rather than an application code crash.

Here is the exact triage and troubleshooting playbook for this scenario:

Step 1: Check Kubernetes Endpoints (The Most Common Culprit)
Even if a pod is "Running", if its Readiness Probe is failing, Kubernetes removes that pod from the Service endpoints. This means incoming traffic has nowhere to go, resulting in a 503 error.

Check if the Service has active endpoints:

-
$kubectl get endpoints <your-service-name>
-
What to look for: If the output shows <none> or has zero IP addresses under endpoints, your service is not routing traffic to any pods because the readiness probes are failing.

Describe the Pod to check probe failures:

Bash
kubectl describe pod <pod-name>
What to look for: Look at the Events section at the bottom for Liveness probe failed or Readiness probe failed messages (e.g., HTTP probe failed with statuscode: 500 on /actuator/health).

Step 2: Inspect Ingress Controller, API Gateway, or Service Mesh
If endpoints do exist and pods are ready, the block is happening higher up in the networking layer (Ingress NGINX, AWS ALB, Istio, etc.).

Check Ingress / Load Balancer Logs:

Bash
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller --tail=100
Look for upstream connection errors, such as no live upstreams while connecting to upstream.

Verify Target Group Health (If using AWS ALB/ELB):

Go to the AWS EC2/ALB Console and check the Target Groups associated with your cluster.

Check if the targets are marked as Unhealthy. If they are unhealthy, the load balancer is intentionally returning 503s to users.

Step 3: Test Internal Connectivity (Port-Forwarding)
Bypass the ingress and load balancer completely to see if the container itself is actually responding internally.

Port-forward directly to the pod:

Bash
kubectl port-forward <pod-name> 8080:8080
Hit the application locally via curl:

Bash
curl -I http://localhost:8080/actuator/health
If this returns HTTP 200: The app is fine. The issue is strictly 100% network routing, service selectors, or ingress configuration.

If this returns HTTP 503: The application's internal web server (like Tomcat/Spring Boot embedded server) is overwhelmed, misconfigured, or returning 503 from an internal filter.

Step 4: Check Service Selector Mismatch
With Blue/Green deployments, it's very easy for a label mismatch to happen during the routing switch.

Check your Service YAML labels:

Bash
kubectl get svc <service-name> -o yaml
Ensure the selector (e.g., version: green) matches the actual labels assigned to your running deployment pods (kubectl get pods --show-labels). If they don't match precisely, traffic gets lost.
