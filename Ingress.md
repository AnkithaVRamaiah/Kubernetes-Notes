### **What is Ingress in Kubernetes?**

Imagine you have several web applications inside your Kubernetes cluster, like a **UI** app and an **API** app. These apps need to be accessible from the internet, but you don’t want to expose each app separately to the outside world. Instead, you want to manage all external traffic to these apps in a centralized way.

**Ingress** is like a **traffic controller** for web traffic (HTTP/HTTPS) inside your Kubernetes cluster. It allows you to define rules for how to route external traffic to the correct internal service.

### **Basic Concept:**

- **Ingress Resource**: A Kubernetes object that defines routing rules for HTTP/HTTPS traffic.
- **Ingress Controller**: A piece of software (running inside your Kubernetes cluster) that manages and implements the routing rules defined in the Ingress Resource. 

**Example scenario:**
- You have two applications running in your Kubernetes cluster:
  - A **UI** application (`ui-service`).
  - An **API** application (`api-service`).
- You want to make both applications available at the same domain:
  - `www.example.com/ui` → Goes to the UI service.
  - `www.example.com/api` → Goes to the API service.

Without Ingress, you’d have to expose each service with a separate external load balancer, which isn’t efficient. Instead, you can define an Ingress to handle the routing.

### **How Does Ingress Work?**

Let’s walk through an example where we have a **UI service** and an **API service**, and we want to route the traffic based on the **URL path** (`/ui` and `/api`).

1. **Ingress Resource**: The rules for routing traffic are defined in a Kubernetes **Ingress resource**.

#### **Example YAML for Ingress Resource**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: www.example.com
    http:
      paths:
      - path: /ui
        pathType: Prefix
        backend:
          service:
            name: ui-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

- **host**: The domain `www.example.com` is specified. All traffic coming to this domain will be routed according to the rules inside the `Ingress` resource.
- **paths**:
  - The path `/ui` routes traffic to the `ui-service`.
  - The path `/api` routes traffic to the `api-service`.

### **How Traffic Gets Routed**:

- If someone types `www.example.com/ui` in their browser, the Ingress Controller will send that traffic to the `ui-service` inside the cluster.
- If someone types `www.example.com/api` in their browser, the Ingress Controller will send that traffic to the `api-service`.

### **What Does the Ingress Controller Do?**

The **Ingress Controller** is the actual software (like **NGINX**, **Traefik**, or **HAProxy**) that listens for changes in the Ingress resource and enforces the traffic routing based on the rules.

- When the Ingress resource is created, the Ingress Controller reads the configuration.
- It listens for incoming traffic to `www.example.com` and checks the path.
  - If it’s `/ui`, the traffic goes to the `ui-service`.
  - If it’s `/api`, the traffic goes to the `api-service`.

### **TLS (SSL) Termination**:

In real-world applications, you usually want to secure the communication using HTTPS. Ingress allows you to **terminate SSL** (decrypt the HTTPS traffic) at the Ingress Controller level. This means that the traffic between the client and the Ingress controller is encrypted, but the traffic between the Ingress and your services can be unencrypted (HTTP).

#### **Example for SSL Termination**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  rules:
  - host: www.example.com
    http:
      paths:
      - path: /ui
        pathType: Prefix
        backend:
          service:
            name: ui-service
            port:
              number: 80
  tls:
  - hosts:
    - www.example.com
    secretName: tls-secret
```

Here:
- **tls-secret** is a Kubernetes **Secret** that contains your SSL certificate and private key.
- The **Ingress Controller** will decrypt HTTPS traffic at the Ingress level, ensuring secure communication from the client to your cluster.

### **Real-Life Example of Ingress in Action**:

Let’s say you’re building a blog platform with these services:
- **Frontend/UI** for viewing posts and articles.
- **API** for fetching and posting data.

Without Ingress, you’d need to create separate Load Balancers for each service. But with Ingress, you can define rules like this:
- `www.myblog.com/` → Loads the **UI** app (frontend).
- `api.myblog.com/` → Loads the **API** app (backend).

This simplifies your architecture and reduces the need for multiple Load Balancers.

---

### **Advantages of Using Ingress**:

1. **Centralized Traffic Management**: You don’t need multiple Load Balancers. Ingress lets you route traffic to different services based on the domain or URL path.
2. **SSL Termination**: Ingress can handle SSL/TLS certificates, making it easy to secure communication.
3. **Simplified Routing**: You can create complex routing rules (host-based or path-based) to direct traffic to the appropriate service.
4. **Cost-Efficient**: Instead of having multiple public IPs for each service, you can use a single Ingress IP for all services.

### **Disadvantages of Using Ingress**:

1. **Complexity**: Setting up and managing Ingress controllers and resources can be a bit complex, especially with multiple services.
2. **Ingress Controller Dependency**: Your traffic routing relies on the Ingress Controller, so it’s important to choose a reliable controller (like NGINX, Traefik, etc.).
3. **Limited Protocols**: Ingress mainly works for HTTP and HTTPS. It doesn’t support other protocols like FTP or WebSockets out of the box.

---

### **When to Use Ingress**:

- When you need to expose multiple services (like **UI** and **API**) under the same domain.
- When you want to **centralize SSL/TLS management** (i.e., handle HTTPS traffic in one place).
- When you want to **simplify routing** of external traffic to different services based on host/domain or URL path.

---

### **Conclusion:**

Ingress is a Kubernetes resource that helps manage **external HTTP(S) traffic** and route it to internal services based on rules you define (like the host or path). It simplifies exposing services, handling SSL termination, and routing traffic efficiently. It works in conjunction with an **Ingress Controller** (like NGINX or Traefik) that enforces the routing logic.

