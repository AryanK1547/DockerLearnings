# Docker & Kubernetes Study & Revision Guide

## 1. Session Executive Summary

This study session covered practical hands-on exercises transitioning from multi-container orchestration with **Docker Compose** to container management, workload scaling, configuration management, and persistent storage in **Kubernetes**.

### Key Milestones Achieved
1. **Docker Compose Orchestration**: Defined and launched multi-container environments (web server and client), managed volumes, and resolved YAML syntax errors.
2. **Kubernetes Core Workloads**: Transitioned from imperative `kubectl run` pod creation to declarative `Deployment` configurations using YAML.
3. **Scaling & Self-Healing**: Scaled deployments up and down, verified pod replica state, and observed self-healing when terminating active pods.
4. **Networking & Services**: Exposed workloads using `NodePort` services, tested label selectors, and forwarded local ports for testing.
5. **Rollouts & Versioning**: Executed zero-downtime rolling updates, tracked revision histories, and rolled back deployments.
6. **Configuration & Secrets**: Managed application settings dynamically using `ConfigMap` and securely stored encoded data with `Secret`.
7. **Storage Mechanics**: Compared temporary `emptyDir` pod volumes against persistent storage backed by `PersistentVolumeClaim` (PVC) and `StorageClass` providers.

---

## 2. Docker & Docker Compose Concepts

### Multi-Container Networking & Volumes
* **Default Networks**: Docker Compose automatically creates an isolated network named `<project_name>_default`.
* **Service Discovery**: Containers on the same Compose network communicate using their **service name** (e.g., `http://web`) as the hostname, not `localhost`.
* **Volume Persistence**: Named volumes (e.g., `compose-demo_webdata`) persist data across `docker compose down` and `up` cycles until explicitly removed using `docker volume rm` or `docker compose down -v`.

### Common Docker Commands Used

```bash
# Docker Compose Commands
docker compose up -d              # Build and start containers in detached mode
docker compose ps                 # List running containers managed by Compose
docker compose exec <service> sh  # Open an interactive shell inside a service container
docker compose down               # Stop and remove containers and networks

# Docker Engine Commands
docker run -d --name <name> <img-[# Run container in background
docker inspect <container-id>     # Detailed JSON configuration/state inspection
docker volume ls                  # List persistent volumes
```

---

## 3. Kubernetes Fundamentals & Core Resources

### Imperative vs. Declarative Management
* **Imperative**: Fast for testing (e.g., `kubectl run`, `kubectl create deployment`, `kubectl expose`).
* **Declarative**: Recommended for production using YAML manifest files applied via `kubectl apply -f <file.yaml>`.

---

### Core Kubernetes Objects

| Object | Purpose | Key Attributes |
| :--- | :--- | :--- |
| **Pod** | Atomic unit of deployment containing one or more containers. | Shared network IP, shared storage. |
| **Deployment** | Manages declarative updates for Pods and ReplicaSets. | Replicas, rolling update strategy, pod templates. |
| **ReplicaSet** | Ensures a specified number of pod replicas are running. | Managed automatically by Deployments. |
| **Service** | Exposes an abstract set of Pods as a network service. | `ClusterIP`, `NodePort`, or `LoadBalancer`. Matches pods via `selectors`. |
| **ConfigMap** | Stores non-confidential key-value configuration data. | Injected via environment variables or volume mounts. |
| **Secret** | Stores confidential data (base64 encoded by default). | Types: `Opaque`, `kubernetes.io/dockerconfigjson`, etc. |
| **PVC** | Request for storage resources by a user. | `AccessModes` (`ReadWriteOnce`), `StorageClass`, `Capacity`. |

---

## 4. Kubernetes YAML Syntax & Field Mapping

Understanding exact casing and structural hierarchy in Kubernetes YAML manifests is crucial.

### Common Syntax Rules & Common Errors
1. **RFC 1123 Naming**: Resource names (`metadata.name`) must be **lowercase** alphanumeric characters, `-` or `.`.
   * ❌ `nginx-yaml-Deployment` (Invalid: uppercase characters)
   * ✅ `nginx-yaml-deployment`
2. **Plural vs. Singular Array Fields**: Fields taking lists require array notation `-`.
   * ❌ `containers:` as an object.
   * ✅ `containers:` as an array list item `- name: ...`.
3. **Field Casing Precision**:
   * Casing matters: `accessModes` vs ❌ `accesModes`; `command` vs ❌ `comamnd`.
   * Correct casing: `ConfigMap` (Kind) vs ❌ `configMap` or `ConfiMap`.

---

### YAML Manifest Reference Template

#### 1. Deployment with Resource Limits (`deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-yaml-deployment
  labels:
    app: nginx-yaml
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-yaml
  template:
    metadata:
      labels:
        app: nginx-yaml
    spec:
      containers:
      - name: nginx-yaml-containers
        image: nginx:1.27
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

#### 2. Service Manifest (`service.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-yaml-service
spec:
  type: NodePort
  selector:
    app: nginx-yaml
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30326
```

#### 3. ConfigMap Manifest (`configmap.yaml`)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  APP_MODE: "development"
  APP_MESSAGE: "HELLO from Kubernetes ConfigMap!!"
```

#### 4. Secret Manifest (`secret.yaml`)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_USERNAME: "admin"
  DB_PASSWORD: "123dbpass"
```

#### 5. PersistentVolumeClaim (`pvc.yaml`)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## 5. Kubernetes Command Cheat Sheet

### Deployment Management & Rollouts
```bash
# Apply manifest
kubectl apply -f <filename>.yaml

# Rollout operations
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>
kubectl rollout undo deployment/<deployment-name>

# Manual scaling
kubectl scale deployment/<deployment-name> --replicas=3
```

### Debugging & Inspection
```bash
# General cluster inspection
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl describe pod <pod-name>
kubectl describe service <service-name>

# Dynamic watching
kubectl get pods -w

# Executing commands in container (Note modern syntax with '--')
kubectl exec -it <pod-name> -- sh

# Port Forwarding for local development testing
kubectl port-forward service/<service-name> 8085:80
```

---

## 6. Storage Architecture Comparison

| Storage Mechanism | Lifecycle | Data Survival on Pod Deletion | Best Used For |
| :--- | :--- | :--- | :--- |
| **`emptyDir` Volume** | Tied directly to the Pod lifecycle. | ❌ Lost when Pod dies or is recreated. | Temporary scratch space, caching. |
| **`PersistentVolumeClaim` (PVC)** | Independent of Pod lifecycle. | ✅ Data persists across Pod deletions. | Databases, stateful applications. |

---

## 7. Key Pitfalls & Solutions Log

| Observed Error | Cause | Solution |
| :--- | :--- | :--- |
| `exec [POD] [COMMAND] is not supported anymore` | Deprecated syntax in modern `kubectl`. | Use `kubectl exec -it <pod> -- <command>` |
| `cannot unmarshal string into Go struct...` | Incorrect data type in YAML (e.g., string instead of map/list). | Ensure proper YAML structure and spacing under `spec`. |
| `strict decoding error: unknown field ...` | Typos in YAML keys (e.g., `accesModes`, `comamnd`, `mount-path`). | Fix spelling: `accessModes`, `command`, `mountPath`. |
| `no matches for kind "ConfiMap"` | Typo in `kind` field or standard apiVersion missing. | Correct casing: `kind: ConfigMap` with `apiVersion: v1`. |
| Container name conflict (`/my-nginx is already in use`) | Container with the same name exists in stopped/exited state. | Run `docker rm <container>` or use `docker start`. |