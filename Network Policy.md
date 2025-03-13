### **Kubernetes Network Policies**  

#### **What is a Network Policy in Kubernetes?**  
A **NetworkPolicy** in Kubernetes is like a **firewall rule** for your Pods. It controls **which Pods can talk to which other Pods** (or external entities) **and on which ports**.  

By default, Kubernetes allows **all Pods to communicate with each other freely**. Network policies help **restrict this communication** for security reasons.

---

### **Key Points About Network Policies**  
1. **Applied at the Pod level** – Not at the Node level.  
2. **Only affect traffic within the cluster** – They do not control ingress/egress outside the cluster.  
3. **Work with network plugins** – Not all CNI (Container Network Interface) plugins support them (e.g., Calico, Cilium, WeaveNet).  
4. **Default behavior** – If no policy is applied, all Pods can talk to each other.  
5. **Are namespaced** – Network policies apply to Pods within the same namespace.

---

### **Basic Structure of a Network Policy**
A **NetworkPolicy** is defined as a YAML manifest with:  
1. **Pod Selector** → Defines which Pods the policy applies to.  
2. **Ingress Rules** → Controls incoming traffic.  
3. **Egress Rules** → Controls outgoing traffic.  
4. **Policy Types** → Defines if the policy is for **Ingress, Egress, or both**.  

---

### **Example 1: Allow Traffic Only From a Specific Pod**  
#### **Scenario:**  
You have a **frontend** Pod and a **backend** Pod. You want to **allow the frontend to talk to the backend, but no other Pods should access it**.

#### **Network Policy YAML:**  
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: backend  # Apply this policy to backend pods
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend  # Allow traffic only from frontend pods
      ports:
        - protocol: TCP
          port: 80  # Allow traffic only on port 80
```
#### **Explanation:**  
✅ Applies to Pods labeled as `app: backend`.  
✅ Allows incoming traffic **only from Pods labeled `app: frontend`**.  
✅ Only allows traffic on **port 80**.  
❌ Other Pods cannot access the backend.

---

### **Example 2: Deny All Ingress Traffic to a Pod**  
#### **Scenario:**  
You have a **database Pod**, and you **don’t want any Pod to access it** (except manually using port-forwarding).  

#### **Network Policy YAML:**  
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress: []  # No ingress rules, so all incoming traffic is blocked
```
#### **Explanation:**  
✅ Applies to Pods labeled as `app: database`.  
❌ No incoming traffic is allowed from any Pod.  
✅ Database Pod can still **make outgoing requests**.

---

### **Example 3: Allow a Pod to Access the Internet But Not Other Pods**  
#### **Scenario:**  
A logging service needs to **send logs to an external server** but should not communicate with any other Pod.

#### **Network Policy YAML:**  
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-only
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: logging-service
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0  # Allows traffic to the internet
      ports:
        - protocol: TCP
          port: 443  # Allows HTTPS traffic
```
#### **Explanation:**  
✅ Applies to Pods labeled as `app: logging-service`.  
✅ Allows outgoing traffic **to the internet** (`0.0.0.0/0`).  
✅ Only allows traffic **on port 443** (HTTPS).  
❌ Cannot communicate with other Pods.

---

### **Example 4: Allow Traffic Between Pods in the Same Namespace Only**  
#### **Scenario:**  
Pods should be able to communicate **only within the same namespace** but not with other namespaces.

#### **Network Policy YAML:**  
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
  namespace: default
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: default  # Allow traffic only within the same namespace
```
#### **Explanation:**  
✅ Applies to **all Pods in the default namespace**.  
✅ Allows traffic **only from Pods in the same namespace**.  
❌ Blocks traffic from Pods in other namespaces.

---

### **Final Notes**  
- **Network policies are "whitelists" (deny by default once a policy is applied).**  
- **If no policy exists, all traffic is allowed.**  
- **To fully enforce isolation, you may need multiple policies.**  
- **Use `kubectl get networkpolicy -n <namespace>` to check existing policies.**  

### **Real-World Use Case: Securing a Multi-Tier Web Application**  

#### **Scenario:**  
You have a **3-tier application** running in a Kubernetes cluster with the following components:  
1. **Frontend** (React or Angular)  
2. **Backend** (Node.js, Django, or Flask)  
3. **Database** (PostgreSQL or MySQL)  

#### **Security Requirements:**  
✅ **Frontend should talk only to the backend.**  
✅ **Backend should talk only to the database.**  
✅ **Database should not accept traffic from any other Pod except the backend.**  
✅ **No Pod should be able to communicate with Pods in another namespace.**  

---

### **Step 1: Restrict Frontend to Backend Communication**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: app-namespace
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend  # Allow only frontend Pods to talk to backend
      ports:
        - protocol: TCP
          port: 5000  # Backend runs on port 5000
```
✅ **Allows only frontend Pods** to communicate with the backend.  
❌ **Blocks other Pods** from accessing the backend.

---

### **Step 2: Restrict Backend to Database Communication**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-database
  namespace: app-namespace
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend  # Only backend can access database
      ports:
        - protocol: TCP
          port: 5432  # Database runs on PostgreSQL port 5432
```
✅ **Only backend Pods** can talk to the database.  
❌ **Blocks frontend and other services** from directly accessing the database.

---

### **Step 3: Block Traffic Between Namespaces**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-cross-namespace-traffic
  namespace: app-namespace
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: app-namespace  # Allow traffic only within this namespace
```
✅ **Allows only internal namespace communication.**  
❌ **Blocks traffic from other namespaces.**  

---

### **Outcome of This Setup**  
🚀 **Frontend → Backend ✅ (Allowed on Port 5000)**  
🚀 **Backend → Database ✅ (Allowed on Port 5432)**  
⛔ **Frontend → Database ❌ (Blocked)**  
⛔ **Other Pods → Database ❌ (Blocked)**  
⛔ **Cross-namespace traffic ❌ (Blocked)**  

This ensures **zero-trust networking** where only **essential communications** happen, improving **security** and **performance**.  

---
### **Testing Kubernetes Network Policies Using `kubectl`**  

Now that we have applied network policies, let's verify if they are working correctly using `kubectl exec` and `curl`.

---

### **1️⃣ Deploy Sample Pods for Testing**  

#### **Deploy Frontend, Backend, and Database Pods**  
Run these commands to create test Pods and services.

```yaml
# frontend.yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  containers:
    - name: frontend
      image: nginx  # Using Nginx as a test frontend
---
# backend.yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  containers:
    - name: backend
      image: python  # Using Python as a test backend
---
# database.yaml
apiVersion: v1
kind: Pod
metadata:
  name: database
  labels:
    app: database
spec:
  containers:
    - name: database
      image: postgres  # Using PostgreSQL as a test database
```

Apply the manifests:  
```sh
kubectl apply -f frontend.yaml
kubectl apply -f backend.yaml
kubectl apply -f database.yaml
```

---

### **2️⃣ Verify Communication Between Pods**  

#### **🔹 Test if Frontend Can Reach Backend**  
Run this command from the **frontend** Pod:
```sh
kubectl exec -it frontend -- curl http://backend:5000
```
✅ Expected Result: Should work because we allowed frontend to talk to backend.

---

#### **🔹 Test if Frontend Can Reach Database**  
Run this command from the **frontend** Pod:
```sh
kubectl exec -it frontend -- curl http://database:5432
```
⛔ Expected Result: Should **fail** because frontend is **not allowed** to talk to the database.

---

#### **🔹 Test if Backend Can Reach Database**  
Run this command from the **backend** Pod:
```sh
kubectl exec -it backend -- curl http://database:5432
```
✅ Expected Result: Should work because backend is **allowed** to talk to the database.

---

#### **🔹 Test if Pods in Another Namespace Can Access the Backend**  
Create a new namespace and deploy a test Pod:
```sh
kubectl create namespace test-namespace
kubectl run testpod --image=busybox --restart=Never -n test-namespace -- sleep 3600
```
Now, try to reach the backend from this test Pod:
```sh
kubectl exec -it testpod -n test-namespace -- curl http://backend:5000
```
⛔ Expected Result: Should **fail** because we blocked cross-namespace communication.

---

### **3️⃣ Debugging Network Policies**
If something is not working as expected, check:  

- **List network policies in the namespace:**
  ```sh
  kubectl get networkpolicy -n app-namespace
  ```
- **Describe a specific network policy:**
  ```sh
  kubectl describe networkpolicy allow-frontend-to-backend -n app-namespace
  ```
- **Check logs of failed network requests using a tool like `netcat` or `tcpdump`.**

---

### **Final Thoughts**
✅ Successfully restricted **frontend-to-backend**, **backend-to-database**, and **namespace isolation**.  
✅ Kubernetes **Network Policies** provide a **zero-trust security model** for cluster networking.  
### **Implementing Network Policies in a CI/CD Pipeline** 🚀  

Now, let's integrate Kubernetes **Network Policies** into a **CI/CD pipeline** using GitHub Actions + Argo CD.  
This ensures that security policies are automatically applied when deploying applications.

---

## **🚀 CI/CD Workflow Overview**
1️⃣ **Developers push code to GitHub**  
2️⃣ **GitHub Actions builds and pushes the Docker image** to a container registry (ECR/Docker Hub)  
3️⃣ **Argo CD syncs the Kubernetes manifests** (including Network Policies) to the cluster  
4️⃣ **Network Policies are automatically applied**, securing communication  

---

## **1️⃣ Setup GitHub Actions for CI**
Create a GitHub Actions workflow file `.github/workflows/deploy.yaml` to build and push images.

```yaml
name: Deploy App to Kubernetes

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2

      - name: Login to Docker Hub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin

      - name: Build and Push Frontend Image
        run: |
          docker build -t myorg/frontend:latest ./frontend
          docker push myorg/frontend:latest

      - name: Build and Push Backend Image
        run: |
          docker build -t myorg/backend:latest ./backend
          docker push myorg/backend:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Manifests Repo
        uses: actions/checkout@v2
        with:
          repository: myorg/k8s-manifests
          token: ${{ secrets.REPO_ACCESS_TOKEN }}

      - name: Update Deployment Manifests
        run: |
          sed -i 's|image: myorg/frontend:.*|image: myorg/frontend:latest|' frontend/deployment.yaml
          sed -i 's|image: myorg/backend:.*|image: myorg/backend:latest|' backend/deployment.yaml

      - name: Commit and Push Changes
        run: |
          git config --global user.email "ci-bot@github.com"
          git config --global user.name "CI Bot"
          git add .
          git commit -m "Updated images for deployment"
          git push
```
✅ **This pipeline:**  
- Builds and pushes the latest frontend & backend images  
- Updates Kubernetes manifests in a Git repo  
- Argo CD (set up separately) will detect and sync changes  

---

## **2️⃣ Kubernetes Manifests (Including Network Policies)**  

### **Deployment Manifests**
```yaml
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: app-namespace
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: myorg/frontend:latest
          ports:
            - containerPort: 80
---
# backend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: app-namespace
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: myorg/backend:latest
          ports:
            - containerPort: 5000
```

✅ These **deployments auto-update** when the CI/CD pipeline pushes new images.

---

## **3️⃣ Network Policy Manifests**
Now, we ensure secure networking by **deploying Network Policies with Argo CD**.

```yaml
# network-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: app-namespace
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 5000
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-to-database
  namespace: app-namespace
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend
      ports:
        - protocol: TCP
          port: 5432
```

✅ **Security is now built into the pipeline**.  
✅ Whenever Argo CD syncs changes, **Network Policies are automatically enforced**.  

---

## **4️⃣ Set Up Argo CD for Continuous Deployment**
To **automatically apply changes**, install Argo CD and create an Application.

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### **Create an Argo CD Application**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app-deployment
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/myorg/k8s-manifests.git'
    targetRevision: HEAD
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: app-namespace
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```
Apply it:  
```sh
kubectl apply -f argo-app.yaml
```

✅ **Now, whenever GitHub Actions updates the manifests, Argo CD automatically deploys the latest version.**  

---

## **🎯 End-to-End Flow**
✔ Developer pushes code → **GitHub Actions builds and updates images**  
✔ GitHub Actions updates Kubernetes manifests in **GitHub Repo**  
✔ **Argo CD syncs** manifests to Kubernetes  
✔ **Network Policies are automatically enforced**  
✔ Secure communication: **Frontend ↔ Backend** & **Backend ↔ Database**  

---

### **🔹 Final Verification**
Run:
```sh
kubectl exec -it frontend -- curl http://backend:5000  # ✅ Should work
kubectl exec -it frontend -- curl http://database:5432  # ❌ Should fail
kubectl exec -it backend -- curl http://database:5432  # ✅ Should work
kubectl exec -it backend -- curl http://frontend:80  # ❌ Should fail
```
✅ **This proves our CI/CD pipeline ensures Kubernetes security policies are in place!**  

---

### **📊 Extending CI/CD with Prometheus & Grafana for Monitoring**  

Now that we've secured our Kubernetes network with **Network Policies**, let’s set up **Prometheus & Grafana** to monitor our **CI/CD deployments, pods, and network traffic**.  

---

## **📌 What We’ll Monitor**
1️⃣ **Argo CD & GitHub Actions Deployments**  
2️⃣ **Frontend & Backend App Metrics**  
3️⃣ **Kubernetes Cluster Health (CPU, Memory, Network)**  
4️⃣ **Network Policy Traffic (Who is talking to whom?)**  

---

## **1️⃣ Install Prometheus & Grafana in Kubernetes**
We’ll use Helm to install both.

### **Step 1: Add Helm Repositories**
```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### **Step 2: Install Prometheus Stack**
```sh
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```
✅ This installs:
- **Prometheus** (for metrics collection)  
- **Alertmanager** (for notifications)  
- **Node Exporter** (for Kubernetes node stats)  

---

## **2️⃣ Install Grafana**
```sh
helm install grafana grafana/grafana -n monitoring
```
✅ **Access Grafana**
```sh
kubectl port-forward svc/grafana 3000:80 -n monitoring
```
- Open `http://localhost:3000`  
- Default **Username: `admin`**, Password: **Run**:
  ```sh
  kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode
  ```
- **Data Source** → Add **Prometheus** (`http://monitoring-prometheus.monitoring:9090`)

---

## **3️⃣ Set Up Dashboards for CI/CD & Network Policies**
Now, let’s **visualize our deployments and network security.**

### **🔹 Dashboard 1: GitHub Actions CI/CD Metrics**
✅ Add GitHub Actions Exporter:
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: github-actions-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: github-actions-exporter
  endpoints:
    - port: http
      interval: 30s
```
- **Grafana Dashboard:** Import **ID: 14834** (GitHub Actions Metrics)  

---

### **🔹 Dashboard 2: Argo CD Deployments**
- **Prometheus Query to Track Deployments:**
```promql
kube_deployment_status_replicas{namespace="argocd"}
```
- **Grafana Dashboard ID:** **14752** (ArgoCD + Kubernetes Deployments)  

---

### **🔹 Dashboard 3: Network Policy Traffic**
To monitor network traffic:
1️⃣ **Install Cilium for Network Flow Monitoring**
```sh
helm install cilium cilium/cilium --namespace kube-system
```
2️⃣ **Enable Network Flow Export**
```sh
cilium monitor --type drop
```
3️⃣ **Prometheus Query for Network Drops**
```promql
cilium_drop_count_total
```
- **Grafana Dashboard ID:** **12003** (Cilium Network Policy Observability)  

---

## **🎯 Final Workflow with Monitoring**
✔ **CI/CD Deployments tracked in Grafana**  
✔ **Network Policies enforced & visualized**  
✔ **Real-time alerts for policy violations**  
✔ **Kubernetes health (CPU, Memory, Network) in Grafana**  

---
