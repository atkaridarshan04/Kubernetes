# Kubernetes RBAC

Kubernetes RBAC (Role-Based Access Control) is a system used to define and enforce policies governing what actions users, applications, and services can perform within a Kubernetes cluster. With RBAC, you can control permissions on resources like Pods, Services, and ConfigMaps, providing security and preventing unauthorized access. Here’s a breakdown of the main concepts and components of Kubernetes RBAC:

---
### 1. **Key Components of Kubernetes RBAC**

   - **Roles and ClusterRoles**: Define a set of permissions to access certain Kubernetes resources.
     - **Role**: Permissions within a specific namespace.
     - **ClusterRole**: Permissions across the entire cluster.

   - **RoleBindings and ClusterRoleBindings**: Assign permissions to users, groups, or service accounts.
     - **RoleBinding**: Links a Role to a subject (user, group, or service account) within a namespace.
     - **ClusterRoleBinding**: Links a ClusterRole to a subject across the cluster.

   - **Subjects**: The entities (users, groups, or service accounts) that are granted permissions.

---
### 2. **How RBAC Works in Kubernetes**

   RBAC works by defining Roles or ClusterRoles that specify what actions can be performed on which resources. These roles are then bound to subjects (users, groups, or service accounts) using RoleBindings or ClusterRoleBindings.

   Example:
   - You could create a Role that allows read-only access to Pods in a specific namespace.
   - Then, you bind this Role to a specific user using a RoleBinding, granting that user the permissions defined in the Role.

---
### 3. **Core RBAC Resources**

   - **Role**: Grants permissions within a specific namespace.
     ```yaml
     apiVersion: rbac.authorization.k8s.io/v1
     kind: Role
     metadata:
       namespace: my-namespace
       name: pod-reader
     rules:
     - apiGroups: [""]
       resources: ["pods"]
       verbs: ["get", "list"]
     ```

   - **ClusterRole**: Grants permissions cluster-wide.
     ```yaml
     apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRole
     metadata:
       name: cluster-pod-reader
     rules:
     - apiGroups: [""]
       resources: ["pods"]
       verbs: ["get", "list"]
     ```

   - **RoleBinding**: Binds a Role to users or groups within a namespace.
     ```yaml
     apiVersion: rbac.authorization.k8s.io/v1
     kind: RoleBinding
     metadata:
       name: read-pods
       namespace: my-namespace
     subjects:
     - kind: User
       name: jane-doe
       apiGroup: rbac.authorization.k8s.io
     roleRef:
       kind: Role
       name: pod-reader
       apiGroup: rbac.authorization.k8s.io
     ```

   - **ClusterRoleBinding**: Binds a ClusterRole to users or groups cluster-wide.
     ```yaml
     apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRoleBinding
     metadata:
       name: read-pods-cluster-wide
     subjects:
     - kind: User
       name: jane-doe
       apiGroup: rbac.authorization.k8s.io
     roleRef:
       kind: ClusterRole
       name: cluster-pod-reader
       apiGroup: rbac.authorization.k8s.io
     ```

---
### 4. **RBAC Permissions**

   RBAC permissions are defined by verbs and resource types in Kubernetes:
   - **Verbs**: Actions that can be taken on resources (e.g., `get`, `list`, `create`, `delete`).
   - **Resources**: Kubernetes objects on which actions are performed (e.g., `pods`, `services`, `configmaps`).
---