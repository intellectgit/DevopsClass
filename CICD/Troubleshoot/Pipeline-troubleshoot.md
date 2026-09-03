stage- Build & test 

stage - code coverage 

here for the code coverage , unit test is the input and how many lines of code are tested?
still how many lines of code is unused ?

ex- there are 4 test cases are executed , for this 4 test cases only 150 lines are tested out of 1000 lines of code 

remaining 850 lines of code is not tested.

tool - jacaco 

Jenkins pipeline failed suddenly, but code was not changed. what will you check ?
========================================================================================
When a Jenkins pipeline fails suddenly without any changes to the application code, it almost always points to an **external infrastructure, environment, or credential failure**.

Here is the exact step-by-step troubleshooting and remediation playbook, following your exact requested order.

---

### 1. Jenkins Controller Health

* **How to troubleshoot:**
* Check the Jenkins Controller dashboard URL. If it fails to load or shows a `502 Bad Gateway` / `504 Gateway Time-out`, the master node is down or frozen.
* Check the controller system logs (via SSH or container logs):
```bash
kubectl logs deployment/jenkins -n <namespace> --tail=200

```


Look for `java.lang.OutOfMemoryError` (GC overhead limit exceeded) or deadlocks.


* **How to fix:**
* **OOM / Crash:** Restart the Jenkins controller pod (`kubectl rollout restart deployment/jenkins -n <namespace>`).
* **Long-term fix:** Increase JVM heap memory allocations (`-Xmx` and `-Xms` settings) in your Jenkins deployment configuration.



---

### 2. Jenkins Agent Status (Kubernetes as Agent)

* **How to troubleshoot:**
* Open the pipeline build logs. If it is stuck indefinitely at *“Pending — Waiting for next available executor”* or throws pod provisioning errors, the dynamic Kubernetes agent failed to spin up.
* Check the cluster for failed agent pods:
```bash
kubectl get pods -n <namespace> -l jenkins=agent
kubectl describe pod <stuck-agent-pod-name> -n <namespace>

```


Look for errors like `ImagePullBackOff` (cannot pull the Jenkins JNLP agent image) or resource quota exhaustion.


* **How to fix:**
* Verify that the Jenkins Kubernetes Cloud plugin configuration has the correct agent pod template, service accounts, and image tags.
* Clean up orphaned stuck pods or stuck PVCs if the cluster is out of scheduling capacity.



---

### 3. CPU / Memory / Disk Space

* **How to troubleshoot:**
* **Disk Space (The #1 silent killer):** Jenkins build nodes frequently run out of disk space due to workspace accumulation, large Docker layers, or unpruned Maven/Gradle dependencies. Run inside the agent or node:
```bash
df -h

```


* **CPU/Memory:** Check node resource utilization via Grafana or `kubectl top nodes`.


* **How to fix:**
* **Disk full:** Clean up workspace data, purge old build directories, or ensure your pipeline workspace cleanup step runs successfully. Configure automatic disk cleanup policies in Jenkins.
* **Resource constraints:** Increase pod CPU/memory limits in the Kubernetes cloud agent plugin configuration.



---

### 4. Credentials / Token Expire

* **How to troubleshoot:**
* Look at the failed stage logs (e.g., *Push to AWS ECR*, *SonarQube Analysis*, or *Git Push*). Look for `401 Unauthorized`, `403 Forbidden`, or `Access Denied`.
* Check if external tokens have expired (e.g., GitHub Personal Access Tokens, AWS IAM session tokens/keys, SonarQube user tokens).


* **How to fix:**
* Generate a new token/credential on the target platform (GitHub, AWS, SonarQube).
* Go to **Jenkins Dashboard > Manage Jenkins > Credentials**, update the corresponding credential ID, and re-run the build.



---

### 5. Git Connectivity

* **How to troubleshoot:**
* Check the `Git-checkout` or `Update Deployment File` stage logs. Errors like `Could not resolve hostname github.com` or `Connection timed out` indicate network blocks to GitHub.
* Verify webhook status on GitHub repository settings if the job is triggered automatically.


* **How to fix:**
* Test network routing from the Jenkins agent pod using `kubectl exec` or a temporary debug container to ping/curl `github.com`.
* Check if GitHub is experiencing an active public outage (check GitHub Status page).



---

### 6. JFrog / ECR Availability

* **How to troubleshoot:**
* Check the `Push to AWS ECR` or artifact resolution (`mvn deploy`) stages. Errors like `no such host`, `dial tcp: connect: connection refused`, or HTTP 500/503 from the registry mean the container registry is unreachable.


* **How to fix:**
* **AWS ECR:** Verify that the IAM role attached to your Jenkins agent still has permissions to push/pull to ECR, and re-verify the region endpoint.
* **JFrog Artifactory:** Check if the Artifactory service is running, and test authentication credentials.



---

### 7. Network / Proxy Issues

* **How to troubleshoot:**
* If your corporate environment uses an HTTP proxy or a restricted Virtual Private Cloud (VPC), network route changes or firewall rule updates can block outbound traffic from Jenkins agents to external APIs (Maven Central, Docker Hub, GitHub).
* Check build logs for `Connection refused` or `Network is unreachable`.


* **How to fix:**
* Verify that environment variables for proxies (`http_proxy`, `https_proxy`, `no_proxy`) are correctly passed to the Jenkins controller and Kubernetes agent pods.
* Coordinate with your network/security team to ensure firewall rules haven't blocked required outbound ports (443/80).



---

### 8. SSL / Certificate Expiry

* **How to troubleshoot:**
* Look for errors like `PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException` or `SSLHandshakeException` in Maven build steps or SonarQube analysis. This happens when an internal tool (like SonarQube, Nexus, or GitHub Enterprise) renewed its SSL certificate, and the Jenkins Java truststore doesn't trust it.


* **How to fix:**
* Import the new self-signed or corporate root/intermediate SSL certificate into the Java runtime truststore (`cacerts`) used by your Jenkins agent or JDK tool configuration:
```bash
keytool -import -trustcacerts -alias custom-cert -file cert.crt -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit

```





---

### 9. Recent Jenkins Plugin or Configuration Changes

* **How to troubleshoot:**
* Go to **Manage Jenkins > System Log** or check the Jenkins audit trail plugin to see if an administrator recently updated plugins (e.g., Git plugin, Kubernetes plugin, SonarQube plugin) or changed global system configurations.
* Breaking changes in plugin updates are a frequent cause of sudden pipeline script parse failures or authentication errors.


* **How to fix:**
* If a plugin update broke the pipeline, downgrade the affected plugin to the previous stable version via **Manage Jenkins > Plugins > Advanced/Installed** (or rollback the Jenkins container image version).






