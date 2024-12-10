## **Install Docker**

1. **Update the apt package index:**
   ```bash
   sudo apt-get update
   ```

2. **Install Docker Engine:**
   ```bash
   sudo apt-get install docker.io
   ```

3. **Verify Docker installation:**
   ```bash
   docker --version
   ```

4. **Add your user to the Docker group:**
   ```bash
   sudo usermod -aG docker $USER
   ```

   After this, log out and log back in, or run:
   ```bash
   newgrp docker
   ```
   *This allows you to run Docker commands without using `sudo`.*

---

## **Install Docker Compose**

1. **Install Docker Compose:**
   ```bash
   sudo apt-get install docker-compose
   ```

2. **Verify Docker Compose installation:**
   ```bash
   docker-compose --version
   ```

---

## **Install Kubectl**

1. **Update the apt package index:**
   ```bash
   sudo apt-get update
   ```

2. **Install `kubectl` using apt:**
   ```bash
   sudo apt-get install -y kubectl
   ```

3. **Verify kubectl installation:**
   ```bash
   kubectl version --client
   ```

---
