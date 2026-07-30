Day-to-Day Responsibilities as a Central DevOps Engineer
As a Central DevOps Engineer supporting 5 projects leveraging our core enterprise product (iTurmeric), my role bridges infrastructure stability, developer enablement, CI/CD reliability, and security governance across AWS EKS and OpenShift clusters.

Here is how I structure and execute my day-to-day responsibilities to ensure high availability and seamless developer collaboration:

1. Morning Health Checks & Proactive Monitoring
Grafana & Prometheus Oversight: My day starts with a comprehensive review of Grafana dashboards connected to Prometheus metrics. I monitor the overall health, resource saturation (CPU/Memory spikes), and pod-level metrics across all active namespaces in both our AWS EKS and enterprise OpenShift clusters.

Incident Triage: Identifying and resolving underlying infrastructure anomalies or resource bottlenecks before they impact production environments.

2. Daily Stand-Up & Cross-Functional Collaboration
Developer Alignment: I participate in daily stand-up calls with project teams and developers. We review the status of ongoing deployments, discuss build or release roadblocks, and triage incoming Jira tickets.

Release Management: Coordinating scheduled releases where tested artifacts are committed, validated, and triggered through the pipeline for deployment.

3. CI/CD Pipeline Management & New Microservice Onboarding
Jenkins Pipeline Configuration (End-to-End): When a project team introduces a new microservice utilizing iTurmeric, I set up robust, scalable Jenkins pipelines using Groovy/Declarative syntax.

Code Checkout & Linting: Pulling code from the repository and running static analysis.

Docker Build: Packaging the application and its dependencies into optimized Docker images.

Registry Push: Pushing the secure image to our centralized container registry.

Deployment Automation: Applying Kubernetes/OpenShift manifests to target EKS/OpenShift clusters.

Observability Integration: Ensuring Prometheus annotations and Grafana alerts are auto-provisioned.

Pipeline Maintenance: Continuously refactoring and optimizing existing Jenkins shared libraries and pipelines to reduce build times and improve feedback loops.

4. Handling Jira Tickets & Troubleshooting Common Issues
As a central team, we manage incoming developer and project team requests via Jira. Typical scenarios and my approach include:

OpenShift/Kubernetes YAML Syntax & Configuration Errors:

Scenario: A team's deployment YAML throws validation errors.

Action: I inspect the manifest, fix indentation, correct API version mismatches, and validate the resource definitions against cluster schemas using oc apply --dry-run=client.

Jenkins Build Failures (Path Mismatches):

Scenario: Builds fail because configuration files or property files are not copied into the correct container layers.

Action: I debug the Dockerfile or Jenkinsfile build steps, correcting the relative path mappings (COPY / ADD instructions) and verifying build context paths.

Pods Stuck in CrashLoopBackOff:

Scenario: A deployed pod repeatedly crashes upon startup.

Action: I immediately check pod logs (oc logs / kubectl logs), inspect previous container states (--previous), and review environment variable configurations, database connection strings, or missing secret mounts.

5. Performance Testing Support
Observability Tuning: When development teams run performance or load tests, I collaborate with them to build custom Grafana dashboards tailored to the specific pods and parameters under test.

Metric Tracking: We closely monitor real-time metrics such as response times, throughput, and CPU/memory utilization curves to help developers identify application bottlenecks and memory leaks.

6. Code Commits, Bug Fixes, and Feature Deployments
Collaborative Fixes: When developers provide reviewed bug fixes or new feature code, I assist in committing changes and triggering the corresponding Jenkins pipelines.

Feedback Loop: If a deployment or smoke test fails post-release, I quickly analyze the failure logs, isolate whether it's an infrastructural or code issue, and report back to the development team with actionable insights.

7. Cluster Security, Namespace Isolation, and Vulnerability Management
Namespace Isolation: Enforcing strict multi-tenancy and cluster security by maintaining isolated namespaces for each of the 5 project teams using Kubernetes NetworkPolicies, Role-Based Access Control (RBAC), and ResourceQuotas.

Binary Vulnerability Scanning: Regularly scanning container binaries and base images for common vulnerabilities and exposures (CVEs).

Dependency Upgrades: When vulnerable third-party Java libraries (JARs) are flagged, I coordinate with developers to upgrade the dependencies, rebuild the artifacts, and execute regression deployments to verify stability.

8. Zero-Downtime Production Deployments
Rolling Updates & Probes: I manage mission-critical deployments across AWS EKS production environments and enterprise OpenShift clusters, strictly implementing liveness, readiness, and startup probes. This ensures traffic is only routed to healthy pods, facilitating seamless, zero-downtime rolling updates.
