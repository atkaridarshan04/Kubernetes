# **Kubernetes Ingress**

## **What is Ingress in Kubernetes?**
Ingress is a Kubernetes API object that manages external access to services in a cluster, typically HTTP and HTTPS traffic. It provides HTTP routing, SSL/TLS termination, and more, allowing services to be exposed to the outside world. Ingress is used to define how external requests should be routed to the services within the cluster.

### **Key Concepts:**
- **Ingress Controller**: A Kubernetes component that watches the Ingress resources and implements the rules specified. It listens for changes and manages routing logic for incoming traffic. Popular controllers include NGINX, Traefik, and HAProxy.
  
- **Ingress Resource**: A collection of rules that define how to route external HTTP(S) requests to services within the cluster. The Ingress resource is defined as a YAML file that specifies routing configurations and additional features (e.g., SSL/TLS termination, host-based routing).

### **Why Use Ingress?**
- **Single Entry Point**: Instead of exposing multiple services to the outside world with LoadBalancers or NodePorts, you can consolidate routing into a single endpoint.
- **Load Balancing**: Ingress can provide load balancing for incoming HTTP traffic.
- **SSL/TLS Termination**: It can handle SSL/TLS encryption and decryption (HTTPS), making your services accessible securely.
- **Host and Path-based Routing**: It allows routing based on domain names (host) or URL paths.
- **Authentication and Rate Limiting**: Many Ingress controllers support authentication mechanisms, rate-limiting, and other advanced routing capabilities.

---

## **Ingress Components**

### **Ingress Resource**
An Ingress resource consists of:
1. **Host-based Routing**: You can route traffic based on the domain name (e.g., `myapp.example.com`).
2. **Path-based Routing**: Allows routing traffic based on the URL path (e.g., `example.com/products`).
3. **Backend Service**: The service to which the traffic should be routed.
4. **TLS/SSL Termination**: Handles HTTPS traffic by specifying SSL/TLS certificates.

#### Example of an Ingress Resource:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: default
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure  # HTTPS
    traefik.ingress.kubernetes.io/router.tls: "true"  # Enable TLS
spec:
  rules:
    - host: myapp.example.com  # The domain or host to route traffic
      http:
        paths:
          - path: /products  # Path-based routing
            pathType: Prefix
            backend:
              service:
                name: product-service
                port:
                  number: 80
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls-secret  # Secret storing the SSL certificate
```

### **Ingress Controller**
The Ingress Controller is the actual component that processes the Ingress resource and manages the routing of traffic to the services. Examples include:
- **NGINX Ingress Controller**
- **Traefik Ingress Controller**
- **HAProxy Ingress Controller**

### **Annotations**
Annotations in the Ingress resource provide additional configurations for the Ingress controller, such as:
- Enabling SSL/TLS termination
- Setting up load balancing rules
- Specifying authentication mechanisms

---

## **How Ingress Works**

1. **Ingress Controller**: The Ingress Controller is deployed in your cluster and watches for Ingress resources. It processes the routing rules specified in these resources.
2. **Traffic Routing**: The Ingress Controller uses the rules to route external HTTP(S) traffic to the appropriate services inside the cluster.
3. **TLS Termination**: Ingress can terminate SSL/TLS connections if configured with certificates. It decrypts incoming HTTPS traffic and forwards the unencrypted traffic to the backend services over HTTP.
4. **Path & Host-based Routing**: Ingress allows fine-grained control over routing traffic to different services based on the host (e.g., `myapp.example.com`) and path (e.g., `/products`).

---

## **Advanced Ingress Features**

### **SSL/TLS Termination**
Ingress can be configured to automatically terminate HTTPS traffic by using TLS certificates. The TLS certificates can be provided through Kubernetes Secrets or automatically generated using ACME (e.g., Let’s Encrypt).

Example:
```yaml
tls:
  - hosts:
      - myapp.example.com
    secretName: myapp-tls-secret
```

### **Path-based Routing**
Ingress can route traffic based on URL paths, directing different requests to different services based on the URL. For example:
```yaml
rules:
  - host: myapp.example.com
    http:
      paths:
        - path: /products
          backend:
            service:
              name: product-service
              port:
                number: 80
```

### **Host-based Routing**
Ingress can route traffic based on the hostname. For example:
```yaml
rules:
  - host: myapp.example.com
    http:
      paths:
        - path: /
          backend:
            service:
              name: demo-service
              port:
                number: 80
```

### **Rate Limiting and Authentication**
Some Ingress controllers (e.g., NGINX, Traefik) support rate-limiting, basic authentication, and other advanced features through annotations.

Example (NGINX Annotations):
```yaml
annotations:
  nginx.ingress.kubernetes.io/limit-connections: "1"
  nginx.ingress.kubernetes.io/limit-rps: "5"
```

---

## **Ingress vs. LoadBalancer**
- **Ingress**: Provides a flexible and centralized way to route HTTP(S) traffic to services. It can be used for path and host-based routing and SSL/TLS termination.
- **LoadBalancer**: Exposes a service externally, typically on a cloud provider, but requires a LoadBalancer to be provisioned (usually expensive).
---
