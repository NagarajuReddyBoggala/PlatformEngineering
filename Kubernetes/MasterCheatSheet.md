# Platform Engineering — Master Cheat Sheet

## Docker

### Mental Model

```text
Dockerfile
    ↓ docker build
Image
    ↓ docker run
Container
```

- Image → immutable template
- Container → running instance of an image
- Registry → stores/distributes images
- Volume → persistent storage
- Network → container communication

### Essential Commands

```bash
docker pull <image>
docker images
docker build -t <name>:<tag> .

docker run -d --name <container> <image>
docker ps
docker ps -a

docker stop <container>
docker start <container>
docker restart <container>
docker rm <container>

docker logs <container>
docker exec -it <container> sh
docker inspect <container>
docker stats <container>

docker rmi <image>
```

### Important `docker run` Options

```text
-d              detached
-p 8080:80      host:container port
-e KEY=value    environment variable
-v volume:/path mount
--name          container name
--network       connect to Docker network
```

### Dockerfile

```text
FROM        → base image
WORKDIR     → working directory
COPY        → copy files into image
RUN         → build-time command
ENV         → environment variable
ARG         → build-time argument
EXPOSE      → documents container port
CMD         → default runtime command
ENTRYPOINT  → main executable
```

Remember:

```text
RUN        → build time
CMD        → default command
ENTRYPOINT → main executable
```

### Image Optimization

```text
Minimal appropriate base image
↓
Avoid unnecessary runtime packages
↓
Good layer/cache ordering
↓
.dockerignore
↓
Multi-stage build when useful
```

Cache-friendly pattern:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

### Networking

```text
-p 8080:80
    ↓
Host → Container

--network mynetwork
    ↓
Container ↔ Container
```

User-defined network:

```text
backend → postgres:5432
```

Prefer service/container DNS names over container IPs.

Remember:

```text
127.0.0.1 → container itself
0.0.0.0   → listen on all interfaces
```

### Storage

```text
-v redis-data:/data
    → named volume

-v $(pwd)/redis-data:/data
    → bind mount
```

Named volume → Docker-managed persistent storage.

Bind mount → host directory mounted into container.

### Docker Compose

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs
docker compose logs -f <service>
docker compose exec <service> sh
docker compose up --build
```

Compose:

```text
services
+
network
+
volumes
+
environment
```

Service names provide DNS between Compose services.

---

# Kubernetes

## Core Architecture

```text
kubectl
   ↓
API Server
   ↓
etcd

Controller Manager
   ↓
Desired state reconciliation

Scheduler
   ↓
Selects node for Pod

Kubelet
   ↓
Runs/manages Pod on node

Container Runtime
   ↓
Runs containers
```

Mental model:

```text
Desired State
    ↓
API Server
    ↓
Controllers
    ↓
Scheduler
    ↓
Kubelet
    ↓
Container Runtime
```

## Workloads

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

```text
Deployment  → stateless applications
StatefulSet → stable identity/storage
DaemonSet   → Pod on each eligible node
Job         → finite task
CronJob     → scheduled Job
```

### Deployment / ReplicaSet

```text
Deployment Controller
    ↓
ReplicaSet
    ↓
ReplicaSet Controller
    ↓
Pods
```

Desired state:

```text
replicas = 3
```

If actual state becomes:

```text
Pods = 2
```

ReplicaSet controller creates a replacement Pod.

### Scheduler

Scheduler decides:

```text
Which node should this Pod run on?
```

Considers:

```text
CPU / memory requests
nodeSelector
nodeAffinity
taints / tolerations
pod affinity / anti-affinity
topology constraints
```

Important:

```text
Scheduler → WHERE
Kubelet   → RUN
```

## Resources

```text
requests → used for scheduling
limits   → maximum allowed runtime resource usage
```

Example:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi
```

## Scheduling Constraints

```text
nodeSelector → simple exact match
nodeAffinity → advanced node rules

taint        → repels Pods
toleration   → allows Pod to tolerate taint
```

A toleration does not force a Pod onto a node.

## Pod Troubleshooting

### Pending

Common causes:

```text
Insufficient resources
Taints
Affinity / selector mismatch
PVC problems
No suitable node
```

Commands:

```bash
k get pods
k describe pod <pod>
k get events
k top node
k describe node <node>
```

### ErrImagePull / ImagePullBackOff

```text
Image cannot be pulled
```

Common causes:

```text
Wrong image/tag
Registry authentication
Registry/network issue
```

### CrashLoopBackOff

```text
Container starts
    ↓
Container crashes
    ↓
Kubelet restarts it
    ↓
Repeated failure
    ↓
CrashLoopBackOff
```

Commands:

```bash
k logs <pod>
k logs <pod> --previous
k describe pod <pod>
```

## Services

Service provides:

```text
Stable virtual endpoint
+
DNS name
+
Traffic distribution to selected Pods
```

Common types:

```text
ClusterIP    → internal
NodePort     → node-level exposure
LoadBalancer → external load balancer integration
```

Important:

```text
port       → Service port
targetPort → Pod/application port
```

Example:

```yaml
port: 80
targetPort: 8080
```

means:

```text
Service :80
    ↓
Pod :8080
```

### Service Selection

```text
Pod labels
    ↓
Service selector
    ↓
EndpointSlice
    ↓
Pod IPs
```

Troubleshooting:

```bash
k get svc <service>
k describe svc <service>
k get pods --show-labels
k get endpointslice
```

## Ingress

```text
Ingress
    ↓
Service
    ↓
Pods
```

Ingress:

```text
Defines HTTP/HTTPS routing rules
```

Ingress Controller:

```text
Actually processes/enforces those routing rules
```

Example:

```text
/app  → service-a
/api  → service-b
```

## DNS

```text
Pod
  ↓
CoreDNS
  ↓
Service ClusterIP
```

Examples:

```text
web
web.default
web.default.svc.cluster.local
```

## ConfigMap / Secret

```text
ConfigMap → non-sensitive configuration
Secret    → sensitive configuration
```

Injection:

```text
envFrom → import all keys
env     → import specific values
```

Secret security depends on:

```text
RBAC
+
etcd encryption at rest
+
access control
+
secret-management practices
```

## Storage

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```

```text
PVC          → storage request
PV           → storage resource
StorageClass → provisioning mechanism
```

Dynamic provisioning:

```text
PVC
 ↓
StorageClass
 ↓
PV created automatically
```

## RBAC

```text
ServiceAccount
      ↓
Role / ClusterRole
      ↓
RoleBinding / ClusterRoleBinding
      ↓
Permissions
```

```text
Role → namespace scoped
ClusterRole → cluster-wide or reusable permissions

RoleBinding → namespace binding
ClusterRoleBinding → cluster-wide binding
```

Useful command:

```bash
k auth can-i <verb> <resource> \
  --as=system:serviceaccount:<namespace>:<sa>
```

## NetworkPolicy

```text
NetworkPolicy
    ↓
Controls allowed Pod traffic
```

The policy is stored by Kubernetes, but **CNI/network plugin enforces it**.

Typical selectors:

```text
Pod
Namespace
IP block
Port
Ingress
Egress
```

## Useful Kubernetes Commands

```bash
k get pods
k get pods -o wide
k get pods --show-labels

k describe pod <pod>
k logs <pod>
k logs <pod> --previous

k exec -it <pod> -- sh

k get svc
k describe svc <service>

k get endpointslice
k get events

k get nodes
k describe node <node>
k top node
k top pod

k get deployments
k get replicasets
k get daemonsets
k get statefulsets

k get pvc
k get pv
k get storageclass

k get role
k get rolebinding
k auth can-i ...
```