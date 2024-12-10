# **Kubernetes RBAC Example with KIND**

This guide demonstrates how to configure Role-Based Access Control (RBAC) in a Kubernetes cluster for three types of users: Admin, General, and Others. It includes deploying a sample application (`2048`), setting up RBAC rules, and securely accessing the cluster using `kubeconfig`.

---

## **Overview**

We define three types of user roles in the cluster:

- **Admin**: Has unrestricted access to all resources in the cluster.
- **General**: Can view and describe all resources across the cluster but cannot modify them.
- **Others**: Can only view resources within the `default` namespace.

These users will authenticate using service account tokens, and their permissions will be restricted based on the RBAC roles defined.

---

## **Prerequisite YAML Files**

### **1. Admin Role (`admin.yaml`)**
```yaml
# admin.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: admin-role
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-binding
subjects:
  - kind: ServiceAccount
    name: admin
    namespace: default
roleRef:
  kind: ClusterRole
  name: admin-role
  apiGroup: rbac.authorization.k8s.io
```

**Permissions**:
- Full access to all resources in all API groups.
- Can create, update, delete, and view resources cluster-wide.

---

### **2. General Role (`general.yaml`)**
```yaml
# general.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: general
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: general-role
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apps", "batch", "extensions", "autoscaling"]
    resources: ["*"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: general-binding
subjects:
  - kind: ServiceAccount
    name: general
    namespace: default
roleRef:
  kind: ClusterRole
  name: general-role
  apiGroup: rbac.authorization.k8s.io
```

**Permissions**:
- Can view (`get`, `list`, `watch`) resources across all namespaces.
- Cannot create, update, or delete resources.

---

### **3. Others Role (`others.yaml`)**
```yaml
# others.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: others
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: others-role
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: others-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: others
    namespace: default
roleRef:
  kind: Role
  name: others-role
  apiGroup: rbac.authorization.k8s.io
```

**Permissions**:
- Can view (`get`, `list`, `watch`) resources only in the `default` namespace.
- No access to resources in other namespaces.

---

## **Step-by-Step Implementation**

### **1. Create a KIND Cluster**
Create a Kubernetes cluster using KIND:
```bash
kind create cluster
```

Verify the cluster:
```bash
kubectl cluster-info --context kind-kind
```

---

### **2. Deploy the Application**
Deploy the `2048` application using the `game-2048.yaml` file located in the directory:

```bash
kubectl apply -f game-2048.yaml
```

Verify the deployment:
```bash
kubectl get all
```

---

### **3. Apply RBAC Configurations**
Apply the RBAC configurations for Admin, General, and Others:
```bash
kubectl apply -f admin.yaml
kubectl apply -f general.yaml
kubectl apply -f others.yaml
```

---

### **4. Generate Tokens for Service Accounts**
Generate tokens for each service account:
- **Admin**:
  ```bash
  kubectl -n default create token admin
  ```
- **General**:
  ```bash
  kubectl -n default create token general
  ```
- **Others**:
  ```bash
  kubectl -n default create token others
  ```

Save the tokens for later use.

---

### **5. Fetch Cluster Information**
Extract the API server URL and CA certificate:
```bash
cat ~/.kube/config
```

Note the `server` value (API server URL) and path to the `certificate-authority` (CA certificate).

---

### **6. Create Kubeconfig on Another Instance**
On the remote instance, create `kubeconfig.yaml` for each user. Here's an example for the General user:

```yaml
apiVersion: v1
kind: Config
clusters:
- name: my-cluster
  cluster:
    server: https://<cluster-url>
    certificate-authority: /path/to/ca.crt
users:
- name: general
  user:
    token: <general-user-token>
contexts:
- name: general-context
  context:
    cluster: my-cluster
    user: general
current-context: general-context
```

---

### **7. Export and Use Kubeconfig**
Export the kubeconfig file:
```bash
export KUBECONFIG=/path/to/kubeconfig.yaml
```

Verify the user's access:
```bash
kubectl get pods
```

Ensure the user's permissions align with their role.

--- 

This step-by-step guide ensures a secure and practical implementation of RBAC in your Kubernetes cluster.