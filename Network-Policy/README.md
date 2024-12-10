# **Network Policies in Kubernetes**

This guide will help you implement and test network policies in Kubernetes to control traffic between pods and namespaces.

---

## **Prerequisites**

Before implementing network policies, ensure that your Kubernetes cluster has a **CNI (Container Network Interface)** plugin that supports Network Policies, such as **Calico** or **Weave**.  

> By default, Kubernetes does not enforce Network Policies unless a compatible CNI plugin is installed.  

### Install **Calico**:
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

## **Restricting Traffic Between Pods**

### **Step 1: Create a Namespace**

Create a namespace to isolate the resources.
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: example
```

Apply the namespace configuration:
```bash
kubectl apply -f namespace.yaml
```

---

### **Step 2: Create Pods in the Namespace**

#### **`pod-a.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
  namespace: example
spec:
  containers:
    - name: nginx
      image: nginx
```

#### **`pod-b.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
  namespace: example
spec:
  containers:
    - name: nginx
      image: nginx
```

Apply the pod configurations:
```bash
kubectl apply -f pod-a.yaml
kubectl apply -f pod-b.yaml
```

---

### **Step 3: Create a Network Policy**

Allow only `pod-b` to communicate with `pod-a`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-pod-b
  namespace: example
spec:
  podSelector:
    matchLabels:
      app: pod-a
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: pod-b
```

Apply the policy:
```bash
kubectl apply -f network-policy.yaml
```

---

### **Step 4: Label Pods**

To ensure the Network Policy works, label `pod-b` for identification:
```bash
kubectl label pod pod-b app=pod-b --namespace=example
```

---

### **Step 5: Test the Policy**

1. **Allow communication from `pod-b` to `pod-a`:**
   ```bash
   kubectl exec -it pod-b -n example -- curl pod-a:80
   ```
   This should succeed because the policy allows traffic from `pod-b`.

2. **Block communication from other pods (e.g., `pod-c`):**
   ```bash
   kubectl exec -it pod-c -n example -- curl pod-a:80
   ```
   This should fail because the policy blocks traffic from other pods.

---

## **Restricting Traffic Between Namespaces**

### **Step 1: Create Namespaces**

#### **`namespace-a.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: namespace-a
```

#### **`namespace-b.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: namespace-b
```

Apply the namespace configurations:
```bash
kubectl apply -f namespace-a.yaml
kubectl apply -f namespace-b.yaml
```

---

### **Step 2: Create Pods in Each Namespace**

#### **`pod-a.yaml` (in `namespace-a`)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
  namespace: namespace-a
spec:
  containers:
    - name: nginx
      image: nginx
```

#### **`pod-b.yaml` (in `namespace-b`)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
  namespace: namespace-b
spec:
  containers:
    - name: nginx
      image: nginx
```

Apply the pod configurations:
```bash
kubectl apply -f pod-a.yaml
kubectl apply -f pod-b.yaml
```

---

### **Step 3: Create a Namespace Restriction Policy**

Prevent traffic from `namespace-b` to `namespace-a`:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-namespace-b
  namespace: namespace-a
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: namespace-b
```

Apply the policy:
```bash
kubectl apply -f namespace-policy.yaml
```

---

### **Step 4: Test the Policy**

1. **Allow communication within `namespace-a`:**
   Traffic within the same namespace should succeed.

2. **Deny communication from `namespace-b` to `namespace-a`:**
   ```bash
   kubectl exec -it pod-b -n namespace-b -- curl pod-a.namespace-a:80
   ```
   This should fail because traffic from `namespace-b` is restricted.

--- 