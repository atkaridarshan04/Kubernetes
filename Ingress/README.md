# **Ingress with Traefik**

This guide demonstrates how to deploy the **Traefik Ingress Controller** on Kubernetes, automatically manage SSL/TLS certificates using **Let's Encrypt**, and expose a sample application using **NodePort**. We'll also configure a free dynamic DNS using **DuckDNS** for external access to our application.

---

## **Contents**
1. [Overview](#overview)
2. [Step-by-Step Implementation](#step-by-step-implementation)
   - [1. Create a KIND Cluster](#1-create-a-kind-cluster)
   - [2. Deploy Traefik with Let's Encrypt](#2-deploy-traefik-with-lets-encrypt)
   - [3. Deploy a Sample Application](#3-deploy-a-sample-application)
   - [4. Create an Ingress Resource with HTTPS](#4-create-an-ingress-resource-with-https)
   - [5. Set Up DuckDNS](#5-set-up-duckdns)
   - [6. Access the Application](#6-access-the-application)

---

## **Overview**

1. **Set up a Kubernetes cluster** using KIND (Kubernetes in Docker).
2. **Deploy Traefik** as an Ingress Controller with automatic SSL/TLS management via **Let's Encrypt**.
3. **Expose a sample application** via **NodePort**.
4. **Configure DuckDNS** for free dynamic DNS to access the application from your browser.
5. Access the application over **HTTPS** with a valid SSL certificate.

---

## **Step-by-Step Implementation**

### **1. Create a KIND Cluster**

Start by creating a Kubernetes cluster with KIND. We'll expose the required ports for the application (80 and 443).

#### KIND Cluster Config (`kind-config.yaml`):
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080
        hostPort: 80
        protocol: TCP
      - containerPort: 30443
        hostPort: 443
        protocol: TCP
```

Create the cluster:
```bash
kind create cluster --config kind-config.yaml
```

---

### **2. Deploy Traefik with Let's Encrypt**

We will deploy Traefik as the Ingress Controller and configure it to automatically obtain SSL certificates from **Let's Encrypt**.

#### Traefik Deployment with Let's Encrypt:

Apply the following YAML file to set up Traefik with **Let's Encrypt**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: traefik
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: traefik
  template:
    metadata:
      labels:
        app: traefik
    spec:
      containers:
        - name: traefik
          image: traefik:v2.10
          args:
            - "--api.insecure=true"  # Enable Traefik dashboard (optional)
            - "--entrypoints.web.address=:80"  # HTTP entrypoint
            - "--entrypoints.websecure.address=:443"  # HTTPS entrypoint
            - "--providers.kubernetescrd"  # Enable Kubernetes CRD support
            - "--certificatesresolvers.myresolver.acme.httpchallenge=true"  # Use HTTP challenge for ACME
            - "--certificatesresolvers.myresolver.acme.httpchallenge.entrypoint=web"  # HTTP entrypoint for ACME
            - "--certificatesresolvers.myresolver.acme.email=your-email@example.com"  # Your email for Let's Encrypt
            - "--certificatesresolvers.myresolver.acme.storage=/data/acme.json"  # Store certificates data
```

Apply the configuration:
```bash
kubectl apply -f traefik-deployment.yaml
```

Verify Traefik deployment:
```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system
```

---

### **3. Deploy a Sample Application**

We'll deploy a simple application that listens on port 80 and expose it using a **NodePort** service.

#### Sample Application (`app.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
        - name: demo-app
          image: hashicorp/http-echo
          args:
            - "-text=Welcome to Kubernetes with Traefik!"
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: demo-app-service
  namespace: default
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
  selector:
    app: demo-app
```

Apply the manifest:
```bash
kubectl apply -f app.yaml
```

Verify the deployment:
```bash
kubectl get pods
kubectl get svc
```

---

### **4. Create an Ingress Resource with HTTPS**

Now, we'll create an Ingress resource that tells Traefik how to route traffic to the application. We'll use **HTTPS** for secure communication.

#### Ingress Manifest (`ingress.yaml`):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
  namespace: default
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure  # Use HTTPS
    traefik.ingress.kubernetes.io/router.tls: "true"  # Enable TLS
spec:
  rules:
    - host: <your-domain>.duckdns.org  # Replace with your DuckDNS domain
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: demo-app-service
                port:
                  number: 80
  tls:
    - hosts:
        - <your-domain>.duckdns.org
      secretName: demo-app-tls
```

Replace `<your-domain>` with your **DuckDNS** subdomain.

Apply the Ingress resource:
```bash
kubectl apply -f ingress.yaml
```

---

### **5. Set Up DuckDNS**

#### a) Create a DuckDNS Account:
1. Go to [DuckDNS](https://www.duckdns.org/).
2. Sign in using your preferred method (GitHub, Google, etc.).
3. Create a new subdomain (e.g., `my-k8s-app`).

#### b) Update DuckDNS with Your Public IP:
DuckDNS provides a token and URL to update your public IP. Use this token to configure your DNS entry.

Example:
```bash
curl -s "https://www.duckdns.org/update?domains=<your-domain>&token=<your-token>&ip="
```

Replace `<your-domain>` and `<your-token>` with your DuckDNS domain and token.

---

### **6. Access the Application**

Once DuckDNS is set up and Traefik has successfully generated an SSL certificate, your application will be available over **HTTPS** at:
```
https://<your-domain>.duckdns.org
```

Test the application:
```bash
curl https://<your-domain>.duckdns.org
```

Expected output:
```
Welcome to Kubernetes with Traefik!
```

---

## **Conclusion**

With this setup, you have:
1. A **Kubernetes cluster** running on KIND.
2. **Traefik** as the **Ingress Controller**, automatically managing SSL/TLS certificates with **Let's Encrypt**.
3. A **sample application** exposed using a **NodePort** service.
4. **DuckDNS** configured for free dynamic DNS.

This setup ensures that your application is securely exposed with an automatically managed SSL/TLS certificate using **Traefik** and **Let's Encrypt**.