# **Kubernetes in Docker (Kind) Setup and Commands**

## **Install Kind**

### 1. Download and install Kind

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### 2. Verify the Installation  
```bash
kind version
```
---

## **Commonly Used Kind Commands**

Here is a list of commonly used **Kind** commands to manage your Kubernetes clusters in Docker.

### **1. `kind create cluster`**
Creates a new Kubernetes cluster using **Kind**.

**Syntax:**
```bash
kind create cluster --name <cluster-name>
```

**Example:**
```bash
kind create cluster --name my-cluster
```

---

### **2. `kind delete cluster`**
Deletes an existing Kubernetes cluster created by Kind.

**Syntax:**
```bash
kind delete cluster --name <cluster-name>
```

**Example:**
```bash
kind delete cluster --name my-cluster
```

---

### **3. `kind get clusters`**
Lists all clusters currently managed by Kind.

**Syntax:**
```bash
kind get clusters
```

**Example:**
```bash
kind get clusters
```

---

### **4. `kind load docker-image`**
Loads a Docker image into the Kind cluster.

**Syntax:**
```bash
kind load docker-image <image-name> --name <cluster-name>
```

**Example:**
```bash
kind load docker-image my-app:latest --name my-cluster
```

---

### **5. `kind export kubeconfig`**
Exports the **Kubeconfig** file of a Kind cluster to your local machine.

**Syntax:**
```bash
kind export kubeconfig --name <cluster-name>
```

**Example:**
```bash
kind export kubeconfig --name my-cluster
```

---

### **6. `kind create cluster --config <config-file>`**
Creates a Kind cluster using a custom configuration file.

**Syntax:**
```bash
kind create cluster --config <config-file>
```

**Example:**
```bash
kind create cluster --config kind-config.yaml
```

---

## **Expose NodePort or ContainerPort in Kind Cluster**

To access services running inside the Kind cluster from your browser, you need to expose the ports in the configuration.

1. **Edit the `kind-config.yaml` file** to expose ports via `NodePort`. For example, to expose `80` and `443` ports:
   
   ```yaml
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
     - role: control-plane
       extraPortMappings:
         - containerPort: 80
           hostPort: 30080
           protocol: TCP
         - containerPort: 443
           hostPort: 30443
           protocol: TCP
   ```

2. **Create or update the Kind cluster with the configuration file:**
   ```bash
   kind create cluster --config kind-config.yaml
   ```

3. **Access the service in your browser** by navigating to `http://localhost:30080` for HTTP or `https://localhost:30443` for HTTPS.

---