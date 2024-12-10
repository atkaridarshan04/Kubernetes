# **Minikube Installation and Commands**

## **1. Install Minikube**

### **Step 1: Update System Packages**
```bash
sudo apt-get update
```
### **Step 2: Install Dependencies**
```bash
sudo apt-get install -y curl apt-transport-https
```

---

### **Step 3: Download Minikube**
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```

### **Step 4: Move the Binary to `PATH`**
```bash
sudo mv minikube /usr/local/bin/
```

### **Step 5: Make Minikube Executable**
```bash
sudo chmod +x /usr/local/bin/minikube
```

---

### **Step 6: Verify the Installation**
```bash
minikube version
```

---

## **2. Minikube Commands**

### **1. `minikube ssh`**
- **Description:** SSH into the Minikube virtual machine to run commands inside it.
- **Syntax:**
  ```bash
  minikube ssh
  ```

---

### **2. `minikube service --list`**
- **Description:** Lists all services in the Minikube cluster with their accessible URLs.
- **Syntax:**
  ```bash
  minikube service --list
  ```
- **Example Output:**
  ```bash
  |------------------|-------------------------------|
  | Service Name     | URL                           |
  |------------------|-------------------------------|
  | kubernetes       | https://192.168.99.100:8443   |
  | nginx            | http://192.168.99.100:30000   |
  |------------------|-------------------------------|
  ```

---

### **3. `minikube profile`**
- **Description:** Manage and switch between multiple Minikube profiles.
- **Syntax:**
  ```bash
  minikube profile
  ```
- **Subcommands:**
  - **List profiles:**
    ```bash
    minikube profile list
    ```
    **Example Output:**
    ```bash
    |------------------|------------------|
    | Profile          | VM Driver        |
    |------------------|------------------|
    | default          | virtualbox       |
    | minikube-prod    | docker           |
    |------------------|------------------|
    ```
  - **Switch profile:**
    ```bash
    minikube profile <profile-name>
    ```

---

### **4. `minikube start`**
- **Description:** Starts a Minikube cluster. Creates a new one if none exists.
- **Syntax:**
  ```bash
  minikube start
  ```
- **Example with specific settings:**
  ```bash
  minikube start --cpus 2 --memory 4096 --driver=docker
  ```

---

### **5. `minikube stop`**
- **Description:** Stops the Minikube cluster and its associated virtual machine.
- **Syntax:**
  ```bash
  minikube stop
  ```

---

### **6. `minikube delete`**
- **Description:** Deletes the Minikube cluster and removes the associated VM, freeing up resources.
- **Syntax:**
  ```bash
  minikube delete
  ```

---

### **7. `minikube addons`**
- **Description:** Manage Kubernetes addons in Minikube.
- **Syntax:**
  - **List available addons:**
    ```bash
    minikube addons list
    ```
  - **Enable an addon:**
    ```bash
    minikube addons enable <addon-name>
    ```
  - **Disable an addon:**
    ```bash
    minikube addons disable <addon-name>
    ```
- **Example:**
  ```bash
  minikube addons enable metrics-server
  ```

---

### **8. `minikube dashboard`**
- **Description:** Opens the Kubernetes Dashboard for your Minikube cluster in a web browser.
- **Syntax:**
  ```bash
  minikube dashboard
  ```

---

### **9. `minikube status`**
- **Description:** Displays the current status of the Minikube cluster.
- **Syntax:**
  ```bash
  minikube status
  ```

---

### **10. `minikube tunnel`**
- **Description:** Opens a network tunnel to expose Kubernetes services on the local machine, especially useful for LoadBalancer type services.
- **Syntax:**
  ```bash
  minikube tunnel
  ```

---

### **11. `minikube ip`**
- **Description:** Retrieves the IP address of the Minikube virtual machine.
- **Syntax:**
  ```bash
  minikube ip
  ```

---

### **12. `minikube update-context`**
- **Description:** Updates your `kubectl` context to point to the Minikube cluster.
- **Syntax:**
  ```bash
  minikube update-context
  ```
---