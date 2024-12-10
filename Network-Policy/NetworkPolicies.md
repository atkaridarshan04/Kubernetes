# **Network Policies in Kubernetes**

Network Policies in Kubernetes are used to control traffic between pods or namespaces. They define rules for how pods communicate internally or with external services, enhancing security and traffic management.

---

### **How Network Policies Work**
Network Policies use Kubernetes labels to decide which pods or namespaces the rules apply to, controlling both incoming (ingress) and outgoing (egress) traffic. Here’s how they work:  
- They are **namespace-specific**, meaning they only apply to pods in the same namespace.  
- They do **not affect** pods or namespaces unless those have the specified labels.  
- Multiple policies can work together, as they are **additive**.  

> **Note:** By default, Network Policies don't block any traffic. If a pod isn't covered by any policy, it allows all traffic.
---

### **Components of a Network Policy**

1. **Pod Selector**: Identifies the pods the policy applies to (based on labels).  
2. **Ingress Rules**: Define allowed incoming traffic (source pod, namespace, or IP).  
3. **Egress Rules**: Define allowed outgoing traffic (destination pod, namespace, or IP).  
4. **Namespace Selector**: Targets traffic between namespaces.  
5. **Policy Types**:  
   - `Ingress`: Controls incoming traffic.  
   - `Egress`: Controls outgoing traffic.  
   - `Ingress,Egress`: Applies both types of control simultaneously.

---

### **CNI Plugin Support**

A compatible **CNI plugin** is required to enforce Network Policies. Commonly used plugins include:  
- **Calico**  
- **Weave**  
- **Cilium**

> Ensure your CNI plugin supports pod-to-pod rules, label selectors, and complex ingress/egress policies.

---

## **Example Network Policies**

### **1. Allow Traffic From Specific IP Range**

Restricts access to pods, allowing traffic only from the `192.168.0.0/16` range.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific-ip-range
  namespace: example
spec:
  podSelector: {}
  ingress:
    - from:
        - ipBlock:
            cidr: 192.168.0.0/16
```

---

### **2. Allow Traffic Only on Specific Ports**

Allows traffic to pods only on port `80` (e.g., HTTP).

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-http-only
  namespace: example
spec:
  podSelector: {}
  ingress:
    - ports:
        - protocol: TCP
          port: 80
```

---

### **3. Allow Traffic Between Pods with Specific Labels**

Enables communication between pods with the label `role: backend`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-traffic
  namespace: example
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: backend
```

---

### **4. Allow Traffic Between Specific Namespaces**

Allows traffic from `namespace-b` to `namespace-a`.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-namespace-a-to-b
  namespace: namespace-a
spec:
  podSelector: {}
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: namespace-b
```

---

### **5. Egress Control – Restrict Outgoing Traffic**

Restricts pods to communicate only with Google DNS (`8.8.8.8`).

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress-to-ip
  namespace: example
spec:
  podSelector: {}
  egress:
    - to:
        - ipBlock:
            cidr: 8.8.8.8/32
  policyTypes:
    - Egress
```

---

### **6. Deny All Ingress Traffic by Default**

Blocks all incoming traffic while allowing specific egress traffic.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: example
spec:
  podSelector: {}
  ingress: []  # Deny all ingress traffic
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              name: allowed-namespace
  policyTypes:
    - Ingress
    - Egress
```

---

### **7. Allow Traffic Based on Protocol**

Allows only UDP traffic on port `53` (DNS).

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-udp-dns
  namespace: example
spec:
  podSelector: {}
  ingress:
    - ports:
        - protocol: UDP
          port: 53
```

---

### **8. Combine Ingress and Egress Policies**

Allows communication between `pod-a` and `pod-b`, restricting other egress traffic.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: complex-policy
  namespace: example
spec:
  podSelector: {}
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: pod-b
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/16
  policyTypes:
    - Ingress
    - Egress
```

--- 