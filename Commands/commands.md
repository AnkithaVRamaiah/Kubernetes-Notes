# Table of Content:

1. [Cluster Information Commands](#cluster-information-commands)
2. [Working with Pods](#working-with-pods)
3. [Working with Deployments](#working-with-deployments)
4. [Working with ReplicaSets](#working-with-replicasets)
5. [Working with Services](#working-with-services)
6. [Working with Namespaces](#working-with-namespaces)
7. [Working with ConfigMaps](#working-with-configmaps)
8. [Working with Secrets](#working-with-secrets)
9. [Working with Ingress](#working-with-ingress)
10. [Working with StatefulSets](#working-with-statefulsets)
11. [Working with DaemonSets](#working-with-daemonsets)
12. [Working with Jobs and CronJobs](#working-with-jobs-and-cronjobs)
13. [Working with Persistent Volumes (PVs) and Persistent Volume Claims (PVCs)](#working-with-persistent-volumes-pvs-and-persistent-volume-claims-pvcs)
14. [Managing Configurations](#managing-configurations)
15. [Scaling and Rolling Updates](#scaling-and-rolling-updates)
16. [Accessing the Kubernetes Dashboard](#accessing-the-kubernetes-dashboard)
17. [Other Useful Commands](#other-useful-commands)
18. [Helm (Package Manager for Kubernetes)](#helm-package-manager-for-kubernetes)

---

### **Cluster Information Commands**

1. **Get cluster info**  
   ```bash
   kubectl cluster-info
   ```
   This command provides details about your Kubernetes cluster’s services, such as the master API server and core DNS.

2. **Get current context**  
   ```bash
   kubectl config current-context
   ```
   Displays the current Kubernetes context. The context is used to reference a particular cluster configuration, including the cluster’s API server and user credentials.

3. **List all contexts**  
   ```bash
   kubectl config get-contexts
   ```
   Lists all available contexts in your kubeconfig file, showing details about clusters and namespaces.

4. **Set a context**  
   ```bash
   kubectl config use-context <context-name>
   ```
   Switch to a specified context from the list of available contexts. Useful for working with multiple clusters or environments.

5. **Get Kubernetes version**  
   ```bash
   kubectl version
   ```
   Displays the Kubernetes client and server versions. This is useful for troubleshooting version mismatches or compatibility issues.

---

### **Working with Pods**

1. **List all Pods**  
   ```bash
   kubectl get pods
   ```
   Lists all pods in the default namespace. To view Pods in a different namespace, use `-n <namespace>`.

2. **List Pods in a specific namespace**  
   ```bash
   kubectl get pods -n <namespace>
   ```
   Shows all Pods in a particular namespace.

3. **Get detailed information about a Pod**  
   ```bash
   kubectl describe pod <pod-name>
   ```
   Displays detailed information about a specific Pod, including events, containers, resource usage, etc.

4. **Get logs from a Pod**  
   ```bash
   kubectl logs <pod-name>
   ```
   Retrieve the logs from the specified Pod.

5. **Get logs from a specific container inside a Pod**  
   ```bash
   kubectl logs <pod-name> -c <container-name>
   ```
   Useful when a Pod contains multiple containers, and you need to fetch logs for a specific container.

6. **Run a Pod interactively**  
   ```bash
   kubectl run -i --tty <pod-name> --image=<image-name>
   ```
   Runs a Pod interactively, allowing you to interact with the container via the terminal.

7. **Delete a Pod**  
   ```bash
   kubectl delete pod <pod-name>
   ```
   Deletes a Pod from the cluster.

---

### **Working with Deployments**

1. **Create a deployment**  
   ```bash
   kubectl create deployment <deployment-name> --image=<image-name>
   ```
   Creates a new Deployment in the cluster with the specified name and container image.

2. **Get all deployments**  
   ```bash
   kubectl get deployments
   ```
   Displays a list of all Deployments in the default namespace.

3. **Describe a deployment**  
   ```bash
   kubectl describe deployment <deployment-name>
   ```
   Shows detailed information about a Deployment, including its replica count, images, and other configurations.

4. **Update a deployment (e.g., change image)**  
   ```bash
   kubectl set image deployment/<deployment-name> <container-name>=<new-image-name>
   ```
   This command allows you to update the image used in a Deployment without changing the configuration.

5. **Scale a deployment**  
   ```bash
   kubectl scale deployment <deployment-name> --replicas=<number-of-replicas>
   ```
   Changes the number of replicas for a given Deployment, allowing you to scale up or down based on load.

6. **Delete a deployment**  
   ```bash
   kubectl delete deployment <deployment-name>
   ```
   Deletes the Deployment and all associated resources.

7. **Roll back a deployment to a previous version**  
   ```bash
   kubectl rollout undo deployment/<deployment-name>
   ```
   Reverts a Deployment to its previous state, useful when a new update leads to issues.

---

### **Working with ReplicaSets**

1. **Get ReplicaSets**  
   ```bash
   kubectl get replicasets
   ```
   Lists all ReplicaSets in the current namespace.

2. **Describe a ReplicaSet**  
   ```bash
   kubectl describe replicasets <replicaset-name>
   ```
   Shows detailed information about a ReplicaSet, including the number of Pods, available Pods, and the template used.

3. **Delete a ReplicaSet**  
   ```bash
   kubectl delete replicasets <replicaset-name>
   ```
   Deletes a ReplicaSet, ensuring that all managed Pods are also removed.

---

### **Working with Services**

1. **Create a service**  
   ```bash
   kubectl expose pod <pod-name> --port=<port>
   ```
   Exposes a Pod as a Service, making it accessible within the cluster via the specified port.

2. **Get all services**  
   ```bash
   kubectl get services
   ```
   Lists all Services within the current namespace.

3. **Describe a service**  
   ```bash
   kubectl describe service <service-name>
   ```
   Shows detailed information about a Service, such as its type, selector, and port configurations.

4. **Delete a service**  
   ```bash
   kubectl delete service <service-name>
   ```
   Deletes a specified Service from the cluster.

5. **Expose a deployment as a service**  
   ```bash
   kubectl expose deployment <deployment-name> --port=<port> --target-port=<target-port>
   ```
   Exposes a Deployment as a Service, where the target port is the port inside the containers.

---

### **Working with Namespaces**

1. **Get all namespaces**  
   ```bash
   kubectl get namespaces
   ```
   Lists all available namespaces in your cluster.

2. **Create a new namespace**  
   ```bash
   kubectl create namespace <namespace-name>
   ```
   Creates a new namespace for organizing resources in the cluster.

3. **Delete a namespace**  
   ```bash
   kubectl delete namespace <namespace-name>
   ```
   Deletes a namespace and all resources within it.

4. **Set namespace context**  
   ```bash
   kubectl config set-context --current --namespace=<namespace-name>
   ```
   Changes the current context to operate within the specified namespace.

---

### **Working with ConfigMaps**

1. **Create a ConfigMap from a file**  
   ```bash
   kubectl create configmap <configmap-name> --from-file=<file-path>
   ```
   Creates a ConfigMap from a file. The file content is stored as key-value pairs within the ConfigMap.

2. **Get all ConfigMaps**  
   ```bash
   kubectl get configmaps
   ```
   Lists all ConfigMaps in the current namespace.

3. **Describe a ConfigMap**  
   ```bash
   kubectl describe configmap <configmap-name>
   ```
   Displays detailed information about a ConfigMap, including its key-value pairs.

4. **Delete a ConfigMap**  
   ```bash
   kubectl delete configmap <configmap-name>
   ```
   Deletes a ConfigMap from the cluster.

---

### **Working with Secrets**

1. **Create a secret from a file**  
   ```bash
   kubectl create secret generic <secret-name> --from-file=<file-path>
   ```
   Creates a secret from a file. The content is stored as base64-encoded data.

2. **Get all secrets**  
   ```bash
   kubectl get secrets
   ```
   Lists all secrets in the current namespace.

3. **Describe a secret**  
   ```bash
   kubectl describe secret <secret-name>
   ```
   Provides details about a secret, including its name and type, though the secret values themselves are not displayed in plain text.

4. **Delete a secret**  
   ```bash
   kubectl delete secret <secret-name>
   ```
   Removes a secret from the cluster.

---

### **Working with Ingress**

1. **Create an Ingress**  
   ```bash
   kubectl apply -f <ingress-definition.yaml>
   ```
   Creates an Ingress resource based on the definition provided in a YAML file. Ingress is used to manage external access to services in the cluster.

2. **Get all Ingress resources**  
   ```bash
   kubectl get ingress
   ```
   Lists all Ingress resources.

3. **Describe an Ingress**  
   ```bash
   kubectl describe ingress <ingress-name>
   ```
   Displays detailed information about an Ingress, including its backend services and rules for routing traffic.

4. **Delete an Ingress**  
   ```bash
   kubectl delete ingress <ingress-name>
   ```
   Deletes an Ingress resource from the cluster.

---

### **Working with StatefulSets**

1. **Get all StatefulSets**  
   ```bash
   kubectl get statefulsets
   ```
   Lists all StatefulSets. StatefulSets are used for managing stateful applications, ensuring that each pod in the set has a unique identity.

2. **Create a StatefulSet**  
   ```bash
   kubectl apply -f <statefulset-definition.yaml>
   ```
   Deploys a StatefulSet based on the configuration provided in a YAML file.

3. **Describe a StatefulSet**  
   ```bash
   kubectl describe statefulset <statefulset-name>
   ```
   Provides detailed information about a StatefulSet, including the number of pods and their individual names.

4. **Delete a StatefulSet**  
   ```bash
   kubectl delete statefulset <statefulset-name>
   ```
   Deletes a StatefulSet and its associated Pods.

---

### **Working with DaemonSets**

1. **Get all DaemonSets**  
   ```bash
   kubectl get daemonsets
   ```
   Lists all DaemonSets. DaemonSets ensure that a Pod runs on all (or some) nodes in the cluster.

2. **Create a DaemonSet**  
   ```bash
   kubectl apply -f <daemonset-definition.yaml>
   ```
   Creates a DaemonSet based on the definition provided in a YAML file.

3. **Describe a DaemonSet**  
   ```bash
   kubectl describe daemonset <daemonset-name>
   ```
   Displays detailed information about a DaemonSet and its associated Pods.

4. **Delete a DaemonSet**  
   ```bash
   kubectl delete daemonset <daemonset-name>
   ```
   Deletes a DaemonSet and its Pods from the cluster.

---

### Working with Jobs and CronJobs

- **Create a Job**  
  `kubectl create job <job-name> --image=<image-name>`  
  Creates a new Job that runs a specified container image.

- **Get all Jobs**  
  `kubectl get jobs`  
  Lists all Jobs in the current namespace.

- **Describe a Job**  
  `kubectl describe job <job-name>`  
  Shows detailed information about a Job, including its status and pods.

- **Delete a Job**  
  `kubectl delete job <job-name>`  
  Deletes a specified Job from the cluster.

- **Create a CronJob**  
  `kubectl create cronjob <cronjob-name> --image=<image-name> --schedule="<schedule>"`  
  Creates a CronJob that runs at the specified schedule.

- **Get all CronJobs**  
  `kubectl get cronjobs`  
  Lists all CronJobs in the current namespace.

- **Describe a CronJob**  
  `kubectl describe cronjob <cronjob-name>`  
  Displays detailed information about a CronJob, including its schedule and associated Jobs.

- **Delete a CronJob**  
  `kubectl delete cronjob <cronjob-name>`  
  Deletes a specified CronJob from the cluster.

---

### Working with Persistent Volumes (PVs) and Persistent Volume Claims (PVCs)

- **Create a Persistent Volume (PV)**  
  `kubectl apply -f <pv-definition.yaml>`  
  Creates a Persistent Volume using the specified YAML definition.

- **Get all Persistent Volumes**  
  `kubectl get pv`  
  Lists all Persistent Volumes in the cluster.

- **Describe a Persistent Volume**  
  `kubectl describe pv <pv-name>`  
  Provides detailed information about a Persistent Volume, including capacity and access modes.

- **Create a Persistent Volume Claim (PVC)**  
  `kubectl apply -f <pvc-definition.yaml>`  
  Creates a Persistent Volume Claim based on the given YAML definition.

- **Get all Persistent Volume Claims**  
  `kubectl get pvc`  
  Lists all Persistent Volume Claims in the current namespace.

- **Describe a Persistent Volume Claim**  
  `kubectl describe pvc <pvc-name>`  
  Displays detailed information about a Persistent Volume Claim, including the associated PV.

- **Delete a Persistent Volume**  
  `kubectl delete pv <pv-name>`  
  Deletes a specified Persistent Volume from the cluster.

- **Delete a Persistent Volume Claim**  
  `kubectl delete pvc <pvc-name>`  
  Deletes a Persistent Volume Claim from the cluster.

---

### Managing Configurations

- **View the current configuration**  
  `kubectl config view`  
  Displays the current configuration of Kubernetes contexts, clusters, and users.

- **Set the current namespace**  
  `kubectl config set-context --current --namespace=<namespace-name>`  
  Changes the current context to operate within the specified namespace.

- **Set a context for a cluster**  
  `kubectl config set-context <context-name> --cluster=<cluster-name>`  
  Creates or updates a context for a specified cluster.

- **Rename a context**  
  `kubectl config rename-context <old-context-name> <new-context-name>`  
  Renames a Kubernetes context.

---

### Scaling and Rolling Updates

- **Scale a deployment**  
  `kubectl scale deployment <deployment-name> --replicas=<number-of-replicas>`  
  Changes the number of replicas for a given Deployment, allowing you to scale up or down based on load.

- **Roll out a deployment**  
  `kubectl rollout deploy <deployment-name>`  
  Deploys changes to a Deployment, such as new images or configurations.

- **Roll back a deployment**  
  `kubectl rollout undo deployment/<deployment-name>`  
  Reverts a Deployment to its previous state, useful when a new update leads to issues.

- **Get rollout status**  
  `kubectl rollout status deployment/<deployment-name>`  
  Displays the status of a deployment rollout.

- **Perform a rolling update**  
  `kubectl set image deployment/<deployment-name> <container-name>=<new-image-name>`  
  Updates the image for a container in the Deployment and performs a rolling update.

---

### Accessing the Kubernetes Dashboard

- **Get the Kubernetes Dashboard URL**  
  `kubectl proxy`  
  Starts a local proxy to the Kubernetes API server. You can then access the Kubernetes Dashboard at `http://localhost:8001/ui`.

- **Access the Dashboard**  
  Navigate to `http://localhost:8001/ui` in your web browser after starting the proxy with the `kubectl proxy` command.

- **Create an admin user for the Dashboard**  
  `kubectl apply -f <admin-user-definition.yaml>`  
  Creates an admin user to access the Dashboard using a YAML file with the necessary RBAC roles.

---

### Other Useful Commands

- **Get cluster info**  
  `kubectl cluster-info`  
  Provides details about the Kubernetes cluster, including master and DNS services.

- **Get all namespaces**  
  `kubectl get namespaces`  
  Lists all namespaces in the cluster.

- **View logs of a pod**  
  `kubectl logs <pod-name>`  
  Retrieves the logs of a Pod.

- **Get resources usage**  
  `kubectl top pod`  
  Displays the resource usage of all pods in the cluster.

- **Get resource usage for nodes**  
  `kubectl top nodes`  
  Displays the resource usage of all nodes in the cluster.

---

### Helm (Package Manager for Kubernetes)

- **Install Helm**  
  `curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash`  
  Installs Helm on your local machine.

- **Add a Helm repository**  
  `helm repo add <repo-name> <repo-url>`  
  Adds a Helm repository to your Helm client.

- **Install a chart**  
  `helm install <release-name> <chart-name>`  
  Installs a chart from a Helm repository.

- **Upgrade a release**  
  `helm upgrade <release-name> <chart-name>`  
  Upgrades an existing Helm release with a new version of the chart.

- **List all Helm releases**  
  `helm list`  
  Displays a list of all the releases in the current namespace.

- **Uninstall a release**  
  `helm uninstall <release-name>`  
  Deletes a Helm release from the cluster.

---
