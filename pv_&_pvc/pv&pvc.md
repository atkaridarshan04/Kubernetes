# **Kubernetes Persistent Volumes**

Persistent storage is essential for stateful applications in Kubernetes. Persistent Volumes (PV) and Persistent Volume Claims (PVC) allow pods to use storage that persists beyond the pod's lifecycle. Kubernetes also introduces **Storage Classes** to manage dynamic provisioning of storage more efficiently.

---

### **1. Persistent Volume (PV)**

- A **Persistent Volume** (PV) is a storage resource pre-provisioned by an administrator or automatically provisioned through a **Storage Class**.
- **Access Modes**: Defines how the volume can be mounted:
  - **ReadWriteOnce** (RWO): Mounted by a single pod for read-write.
  - **ReadOnlyMany** (ROX): Mounted by multiple pods for read-only.
  - **ReadWriteMany** (RWX): Mounted by multiple pods for read-write.
- **Reclaim Policy**: Defines the fate of PV after PVC deletion:
  - **Retain**: Needs manual cleanup.
  - **Delete**: Deleted with PVC.
  - **Recycle**: (Deprecated) Cleans and reuses the volume.
- **Resource Requests**: Defines the required capacity of the volume.

---

### **2. Persistent Volume Claim (PVC)**

- A **PVC** is a request for storage by a user or application, specifying the size and access mode.
- **Binding**: PVC binds to a matching PV based on size and access modes. If no matching PV exists and dynamic provisioning is enabled, a new PV will be created.

---

### **3. Storage Class**

- A **Storage Class** defines the parameters and characteristics of storage.
- It enables **dynamic provisioning** of PVs, where storage backends like `aws-ebs`, `gce-pd`, or `nfs` are defined.
- **Reclaim Policy**: Determines what happens to the volume after PVC deletion (`Retain`, `Delete`).
- **Volume Binding Mode**: Controls when the PV is bound to PVC:
  - **Immediate**: Bound as soon as PVC is created.
  - **WaitForFirstConsumer**: Bound after a pod is scheduled.

---

### **4. Static vs. Dynamic Provisioning**

| **Feature**                | **Static Provisioning**                               | **Dynamic Provisioning**                              |
|----------------------------|-----------------------------------------------------|-----------------------------------------------------|
| **Definition**             | Admin creates PV manually.                          | Kubernetes provisions PVs automatically via Storage Class. |
| **Flexibility**            | Fixed storage setup.                                | On-demand provisioning as PVCs are created.          |
| **Use Cases**              | Suitable for static environments.                   | Ideal for scalable, cloud-native environments.       |

---
