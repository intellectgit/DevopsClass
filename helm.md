## **What is Helm and Charts in EKS?**  
**Helm** is a package manager for **Kubernetes**, similar to how **apt** (Ubuntu) or **yum** (Amazon Linux) works for Linux. It simplifies the deployment and management of applications in Kubernetes by using **Helm Charts**—pre-packaged Kubernetes resources with configurable templates.

### **Key Components of Helm:**
1. **Helm CLI** → Command-line tool to install and manage Helm charts.
2. **Helm Chart** → A collection of YAML files that define Kubernetes resources (Deployment, Service, ConfigMaps, etc.).
3. **Helm Repository** → A storage location for charts (e.g., ArtifactHub, Bitnami Helm Repo).
4. **Values File (`values.yaml`)** → Configurable parameters for the Helm chart.

---

## **Why is Helm Important?**
In an **enterprise EKS (Elastic Kubernetes Service) environment**, managing multiple microservices manually using raw YAML files becomes complex. Helm provides:
**Simplification** → Manage multiple Kubernetes objects as a single package.  
**Reusability** → Use the same chart across multiple environments with different configurations.  
**Versioning & Rollbacks** → Easily roll back to a previous version if a deployment fails.  
**Parameterization** → Customize deployments with values.yaml instead of modifying YAML files manually.  
**Dependency Management** → Helps manage dependent services (e.g., deploying a database alongside an app).  

---

## **Common Use Cases of Helm in EKS**
1. **Deploying Enterprise Applications** → Use Helm to deploy and manage microservices-based applications.
2. **Installing Monitoring Tools** → Prometheus, Grafana, Loki, and Fluentd are installed via Helm in EKS.
3. **Managing Database Deployments** → Deploy MySQL, PostgreSQL, or MongoDB using Helm charts.
4. **CI/CD Integration** → Automate Helm deployments in **Jenkins, GitHub Actions, or ArgoCD**.
5. **Managing Configurations** → Helm **values.yaml** helps manage different configurations for dev, staging, and prod environments.

---

## **Example 1: Install Nginx Using Helm on EKS**
### **Step 1: Install Helm (if not installed)**
```sh
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### **Step 2: Add the Bitnami Helm Repository**
```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### **Step 3: Install Nginx**
```sh
helm install my-nginx bitnami/nginx
```
**What happens?**  
This installs an **Nginx Deployment and Service** on EKS using Bitnami’s Helm chart.

### **Step 4: Verify the Deployment**
```sh
kubectl get pods
kubectl get svc
```

### **Step 5: Uninstall Nginx**
```sh
helm uninstall my-nginx
```

---


---
### **Deploy a Custom Application Using Helm**
---

# **Helm Chart for Node.js App – Step-by-Step Guide**  

### **Step 1: Create a New Helm Chart**
Run the following command to create a Helm chart named `mynodeapp`:  
```sh
helm create mynodeapp
```

It creates a directory structure like:
```
my-app/
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl
│   └── NOTES.txt
├── values.yaml
├── Chart.yaml
└── README.md
```

---

### **Step 2: Update Helm Chart Files**  

## **Update `values.yaml`**  
Modify `values.yaml` to define service as `NodePort` and set up necessary configurations:  
```yaml
replicaCount: 1

image:
  repository: node
  pullPolicy: IfNotPresent
  tag: latest

service:
  type: NodePort
  port: 3000
  nodePort: 32000  # Choose a port in range 30000-32767

ingress:
  enabled: false

resources: {}

autoscaling:
  enabled: false

nodeSelector: {}

tolerations: []

affinity: {}
```

---

## **Update `templates/deployment.yaml`**  
Ensure the Deployment file correctly defines the Node.js container:  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nodeapp
  template:
    metadata:
      labels:
        app: nodeapp
    spec:
      containers:
        - name: nodeapp
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          command: ["node", "-e", "require('http').createServer((req, res) => res.end('Hello from Node.js!')).listen(3000)"]
          ports:
            - containerPort: 3000
```

---

## **Update `templates/service.yaml`**  
Modify `service.yaml` to support both `ClusterIP` and `NodePort`:  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  type: {{ .Values.service.type }}
  selector:
    app: nodeapp
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 3000
      {{- if eq .Values.service.type "NodePort" }}
      nodePort: {{ .Values.service.nodePort }}
      {{- end }}
```
---

### **Step 3: Remove Unnecessary Files**
Since we are not using a **ServiceAccount** and **HPA**, remove these files:  
```sh
rm mynodeapp/templates/serviceaccount.yaml
rm mynodeapp/templates/hpa.yaml
```


---

### **Step 4: Install or Upgrade the Helm Chart**
Deploy the Helm chart using:  
```sh
helm install my-node-app ./mynodeapp
```
If already installed, upgrade with:  
```sh
helm upgrade my-node-app ./mynodeapp
```

---

### **Step 5: Verify the Deployment**
Check if the pod is running:  
```sh
kubectl get pods
```
Check the service type:  
```sh
kubectl get svc my-node-app-service
```

---

### **Step 6: Access the Application**
Find the **Node IP**:  
```sh
kubectl get nodes -o wide
```
Find the **NodePort**:  
```sh
kubectl get svc my-node-app-service -o yaml | grep nodePort
```
Access the application:  
```sh
curl http://<NODE_IP>:32000
```
or open the browser:  
```
http://<NODE_IP>:32000
```

---


| **Command** | **Syntax** | **Example Command** | **Description** |
|------------|-----------|----------------------|----------------|
| **Search for a Helm chart** | `helm search repo <chart-name>` | `helm search repo bitnami/nginx` | Searches for a chart in configured Helm repositories. |
| **Search in Helm Hub** | `helm search hub <keyword>` | `helm search hub nginx` | Searches for a chart in Helm Hub (Artifact Hub). |
| **Install a Helm chart** | `helm install <release-name> <chart-name>` | `helm install my-nginx bitnami/nginx` | Installs a Helm chart in the Kubernetes cluster. |
| **Install with custom values** | `helm install <release-name> <chart-name> --set <key>=<value>` | `helm install my-nginx bitnami/nginx --set service.type=LoadBalancer` | Installs a Helm chart with overridden values. |
| **List installed Helm releases** | `helm list` | `helm list` | Displays all installed Helm releases. |
| **Upgrade a Helm release** | `helm upgrade <release-name> <chart-name>` | `helm upgrade my-nginx bitnami/nginx` | Upgrades an existing Helm release to a newer version. |
| **Uninstall a Helm release** | `helm uninstall <release-name>` | `helm uninstall my-nginx` | Deletes a Helm release from the cluster. |
| **Pull a Helm chart (without installing)** | `helm pull <chart-name>` | `helm pull bitnami/nginx` | Downloads a Helm chart archive (`.tgz` file). |
| **Extract (untar) a Helm chart** | `helm pull <chart-name> --untar` | `helm pull bitnami/nginx --untar` | Downloads and extracts a Helm chart locally. |
| **Check installed repositories** | `helm repo list` | `helm repo list` | Lists all configured Helm repositories. |
| **Add a Helm repository** | `helm repo add <repo-name> <repo-url>` | `helm repo add bitnami https://charts.bitnami.com/bitnami` | Adds a new Helm repository for searching and installing charts. |
| **Update Helm repositories** | `helm repo update` | `helm repo update` | Refreshes the Helm repository list to fetch the latest charts. |
| **Get details of a Helm release** | `helm status <release-name>` | `helm status my-nginx` | Shows the status and health of a Helm release. |
| **Get values of an installed release** | `helm get values <release-name>` | `helm get values my-nginx` | Displays the values used for an installed Helm release. |
| **Preview a Helm installation (Dry Run)** | `helm install <release-name> <chart-name> --dry-run --debug` | `helm install test-nginx bitnami/nginx --dry-run --debug` | Simulates a Helm install to preview the outcome without applying changes. |
| **Render Helm templates without installing** | `helm template <release-name> <chart-name> --debug` | `helm template my-nginx bitnami/nginx --debug` | Displays the Kubernetes YAML manifest generated by Helm. |
| **Package a Helm chart** | `helm package <chart-directory>` | `helm package my-custom-chart/` | Packages a Helm chart into a `.tgz` archive for distribution. |
| **View history of a Helm release** | `helm history <release-name>` | `helm history my-nginx` | Shows the revision history of a Helm release. |
| **Rollback to a previous release** | `helm rollback <release-name> <revision>` | `helm rollback my-nginx 2` | Rolls back a Helm release to a previous version. |
| **Delete Helm release history** | `helm delete <release-name>` | `helm delete my-nginx` | Deletes a Helm release along with its history. |

---
