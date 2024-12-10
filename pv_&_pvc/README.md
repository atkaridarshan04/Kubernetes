# **Kubernetes Persistent Storage**

In this guide, we will set up a **MongoDB StatefulSet** with persistent storage in Kubernetes using **static provisioning**. We will verify data persistence by storing sample data in MongoDB, deleting the pod, and confirming that the data remains intact.

---

## **Steps: MongoDB with Persistent Storage**

### **1. Create a Persistent Volume (Static Provisioning)**

Define a **Persistent Volume (PV)** that uses a host path for storage.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

#### **Commands to Verify PV:**
- List all Persistent Volumes:
  ```bash
  kubectl get pv
  ```
- Describe a Persistent Volume:
  ```bash
  kubectl describe pv mongodb-pv
  ```

---

### **2. Create a Persistent Volume Claim**

Define a **Persistent Volume Claim (PVC)** to bind to the PV.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

#### **Commands to Verify PVC:**
- List all Persistent Volume Claims:
  ```bash
  kubectl get pvc
  ```
- Describe a Persistent Volume Claim:
  ```bash
  kubectl describe pvc mongodb-pvc
  ```

---

### **3. Deploy MongoDB as a StatefulSet**

Deploy MongoDB as a **StatefulSet**, attaching the PVC to the pod.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: "mongodb-service"
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        - name: mongodb
          image: mongo
          ports:
            - containerPort: 27017
          volumeMounts:
            - name: mongodb-storage
              mountPath: /data/db
      volumes:
        - name: mongodb-storage
          persistentVolumeClaim:
            claimName: mongodb-pvc
```

#### **Commands to Verify Pod:**
- List all Pods:
  ```bash
  kubectl get pods
  ```

---

## **Verification Steps**

### **1. Add Sample Data to MongoDB**
1. Execute into the MongoDB pod:
   ```bash
   kubectl exec -it mongodb-0 -- bash
   ```
2. Launch the MongoDB shell:
   ```bash
   mongo
   ```
3. Insert sample data:
   ```javascript
   use mydatabase
   db.sample.insert({ name: "Kubernetes", type: "Database Test" })
   ```
4. Query the data:
   ```javascript
   db.sample.find()
   ```

---

### **2. Check Persistent Data on Minikube Host**
1. SSH into the Minikube VM:
   ```bash
   minikube ssh
   ```
2. Verify the data directory:
   ```bash
   ls /mnt/data
   ```
   You should see the MongoDB data files.

---

### **3. Delete and Recreate the Pod**
1. Delete the MongoDB pod:
   ```bash
   kubectl delete pod mongodb-0
   ```
   Kubernetes will recreate the pod automatically as part of the StatefulSet.

2. Re-execute into the pod and verify data persistence:
   ```bash
   kubectl exec -it mongodb-0 -- bash
   mongo
   use mydatabase
   db.sample.find()
   ```
   The previously inserted data should still be present.

---

## **Notes**
- **Persistent Volume Reclaim Policy:** The PV's reclaim policy (`Retain`) ensures the data persists even after PVC deletion.
- **Host Path:** The `/mnt/data` directory is specific to the Minikube environment. For production clusters, use cloud-native storage solutions or Network File Systems (NFS).

---