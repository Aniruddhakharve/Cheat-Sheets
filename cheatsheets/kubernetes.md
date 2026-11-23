# Kubernetes (K8s) Master Cheat Sheet
## 90 Days of DevOps — Day 50 to Day 60

> **Purpose:** A complete revision guide for daily practice and Kubernetes interviews.
>
> This sheet is based on my recorded Day 50–60 hands-on labs and the WordPress + MySQL capstone. For each concept, it explains what it is, why it exists, the problem it solves, the commands used, and how I verified it.
>
> **Lab environment:** Windows 11, VS Code Git Bash, Docker Desktop, KIND, Kubernetes v1.37.0 and kubectl v1.36.1. The Day 59 Helm lab used Helm v4.3.1.
>
> **Capstone:** [kubernetes-wordpress-mysql-capstone](https://github.com/Aniruddhakharve/kubernetes-wordpress-mysql-capstone)  
> **Day-by-day logs:** [90DaysOfDevOps-shubham-londe](https://github.com/Aniruddhakharve/90DaysOfDevOps-shubham-londe/tree/master/2026)

---

# Table of Contents

1. [Kubernetes in One Minute](#1-kubernetes-in-one-minute)
2. [Day 50 — Architecture and Cluster Setup](#2-day-50--architecture-and-cluster-setup)
3. [kubectl — Command Structure and Context](#3-kubectl--command-structure-and-context)
4. [Day 51 — Pods, Manifests, Labels and Validation](#4-day-51--pods-manifests-labels-and-validation)
5. [Day 52 — Namespaces, Deployments and Rollouts](#5-day-52--namespaces-deployments-and-rollouts)
6. [Day 53 — Services and Service Discovery](#6-day-53--services-and-service-discovery)
7. [Day 54 — ConfigMaps and Secrets](#7-day-54--configmaps-and-secrets)
8. [Day 55 — Volumes, PV, PVC and StorageClasses](#8-day-55--volumes-pv-pvc-and-storageclasses)
9. [Day 56 — StatefulSets and Headless Services](#9-day-56--statefulsets-and-headless-services)
10. [Day 57 — Requests, Limits, QoS and Probes](#10-day-57--requests-limits-qos-and-probes)
11. [Day 58 — Metrics Server and HPA](#11-day-58--metrics-server-and-hpa)
12. [Day 59 — Helm](#12-day-59--helm-kubernetes-package-manager)
13. [Day 60 — WordPress + MySQL Capstone](#13-day-60--wordpress--mysql-capstone)
14. [RBAC and Headlamp](#14-kubernetes-rbac-and-headlamp-capstone-extension)
15. [NFS-Ganesha Shared Storage](#15-nfs-ganesha--capstone-shared-storage)
16. [kubectl Command Master Table](#16-kubectl-command-master-table)
17. [YAML Patterns to Remember](#17-yaml-patterns-to-remember)
18. [Troubleshooting Playbook](#18-common-troubleshooting-playbook)
19. [Important Differences](#19-important-differences--interview-revision)
20. [Interview Questions and Answers](#20-kubernetes-interview-questions-and-answers)
21. [Project Interview Explanation](#21-how-to-explain-my-day-60-project-in-an-interview)
22. [Final Revision Checklist](#22-final-day-5060-revision-checklist)

---

# 1. Kubernetes in One Minute

## What is Kubernetes?

Kubernetes is an open-source container orchestration platform. It automates deploying, scaling, networking and managing containerized applications across a cluster of machines.

## Why Kubernetes if we already have Docker?

Docker can build images and run containers. Kubernetes adds the orchestration layer needed to manage workloads across a cluster.

It helps solve problems such as:

- A Pod crashes — recreate it to maintain the desired state.
- An app needs more capacity — increase replicas.
- A Pod gets a new IP — use a stable Service endpoint and DNS name.
- Data must survive Pod replacement — use persistent storage.
- An app is not ready — keep it out of Service traffic with a readiness probe.
- An app becomes unhealthy — use a liveness probe to restart it.
- CPU load rises — use HPA to scale the replica count.

## Kubernetes mental model

```text
I declare the desired state
          |
          v
    Kubernetes API
          |
          v
 Controllers reconcile desired state
 with actual cluster state
          |
          v
 Pods / Services / Storage / Scaling
```

**Remember:** Kubernetes is declarative. I describe the state I want, and controllers continuously work to make actual state match it.

| Term | Meaning |
|---|---|
| Cluster | Group of machines managed by Kubernetes |
| Node | A machine in the cluster |
| Control Plane | Components that manage cluster state |
| Worker Node | Node that runs application workloads |
| Pod | Smallest deployable Kubernetes workload unit |
| Container | Application process and its runtime environment |
| Manifest | YAML/JSON file describing a Kubernetes resource |
| Controller | Component that reconciles actual and desired state |
| Namespace | Logical grouping of namespaced resources |
| Service | Stable network endpoint for selected Pods |
| Volume | Storage mounted into a Pod/container |
| kubectl | CLI used to communicate with the Kubernetes API |

---

# 2. Day 50 — Architecture and Cluster Setup

## 2.1 Kubernetes architecture

Kubernetes has two major parts:

1. Control Plane
2. Worker Nodes

```text
                       KUBERNETES CLUSTER
                               |
                +--------------+--------------+
                |                             |
          CONTROL PLANE                   WORKER NODE
                |                             |
        +-------+-------+              +------+------+
        |       |       |              |      |      |
   API Server  etcd  Scheduler       kubelet kube-proxy
        |                              |
 Controller Manager               Container Runtime
        |                              |
        +---------------------------> Pods
```

## 2.2 Control Plane components

| Component | What it does | Why it is needed |
|---|---|---|
| kube-apiserver | Front door to the Kubernetes API | Receives, validates and processes API requests |
| etcd | Distributed key-value store | Persists Kubernetes cluster state |
| kube-scheduler | Selects a node for an unscheduled Pod | Places workloads on suitable nodes |
| kube-controller-manager | Runs controllers | Reconciles actual state with desired state |

### API Server

Every normal Kubernetes API operation goes through the API Server.

Example:

```bash
kubectl apply -f pod.yaml
```

kubectl sends a request to the API Server, which handles authentication, authorization and validation. Accepted cluster state is persisted through etcd.

### etcd

etcd stores persistent cluster state. It is not where application container files are stored.

**Interview point:** etcd backups are important in production because it contains the cluster's persistent state.

### Scheduler

The scheduler selects a node for a Pod that has not yet been assigned to a node. It considers CPU/memory requests, node selectors/affinity, taints/tolerations and other scheduling constraints.

**Important:** Scheduler selects the node. It does not start the container. Kubelet and the container runtime perform that work.

### Controller Manager

Controllers continuously compare actual state with desired state.

Example:

```text
Desired replicas = 3
Actual replicas  = 2
        |
        v
ReplicaSet controller detects mismatch
        |
        v
Creates another Pod
        |
        v
Actual replicas = 3
```

## 2.3 Worker Node components

| Component | Responsibility |
|---|---|
| kubelet | Node agent that makes sure assigned Pods are running and reports status |
| kube-proxy | Implements Service networking rules on nodes |
| Container Runtime | Pulls and runs containers, e.g. containerd or CRI-O |

Kubelet communicates with compatible runtimes through the Container Runtime Interface (CRI).

## 2.4 What happens after `kubectl apply`?

1. kubectl sends the request to the API Server.
2. API Server authenticates/authorizes and validates it.
3. Accepted state is persisted in etcd.
4. Controllers observe the resource and create/manage dependent objects.
5. Scheduler selects a node for an unscheduled Pod.
6. Scheduler records the node assignment through the API Server.
7. Kubelet on that node notices the assigned Pod.
8. Kubelet asks the runtime through CRI to prepare/run containers.
9. Runtime pulls the image if needed and starts the container.
10. Kubelet reports Pod/container status through the API Server.

For a Deployment, Deployment and ReplicaSet controllers also participate in creating and maintaining Pods.

## 2.5 What if a component fails?

| Failure | General effect |
|---|---|
| API Server unavailable | kubectl/API operations fail; existing containers may continue running, but cluster management/reconciliation is impaired |
| Worker Node unavailable | Pods on that node become unavailable; controllers respond and replacement Pods may be scheduled elsewhere if capacity exists |
| Scheduler unavailable | Existing Pods can continue; new unscheduled Pods cannot be assigned normally |
| etcd unavailable | API Server cannot reliably read/write persistent cluster state; control-plane operations are severely affected |
| Runtime unavailable | kubelet cannot start/operate containers normally on that node |

## 2.6 KIND — Kubernetes in Docker

KIND creates Kubernetes clusters using Docker containers as Kubernetes nodes. I selected it because Docker Desktop was already part of my local setup and KIND supports multi-node local clusters.

```bash
# Create a named cluster
kind create cluster --name devops-cluster

# Create from a configuration file
kind create cluster --config kind-config.yml

# List KIND clusters
kind get clusters

# List nodes/containers belonging to a cluster
kind get nodes --name devops-cluster

# Delete a named cluster
kind delete cluster --name devops-cluster
```

A simple two-node config:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: devops-cluster
nodes:
  - role: control-plane
  - role: worker
```

The Day 50/51–59 training cluster used one control-plane and one worker. The Day 60 capstone used a separate cluster named `my-cluster`, also with one control-plane and one worker.

## 2.7 Verify the cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl get pods -n kube-system
```

`-o wide` displays additional details such as node and IP information.

## 2.8 kubeconfig and contexts

A kubeconfig stores cluster connection details, user credentials and contexts. Default location:

```text
~/.kube/config
```

A context groups cluster, user and default namespace settings used by kubectl.

```bash
kubectl config current-context
kubectl config get-contexts
kubectl config view
kubectl config use-context kind-devops-cluster
kubectl config set-context --current --namespace=dev
```

| Command | Purpose |
|---|---|
| `current-context` | Show active context |
| `get-contexts` | List contexts |
| `use-context` | Switch context |
| `view` | Inspect kubeconfig |
| `set-context --current --namespace=dev` | Set current context's default namespace |

**Gotcha:** Changing the default namespace affects commands without `-n`. It does not move existing resources.

---

# 3. kubectl — Command Structure and Context

## 3.1 General syntax

```bash
kubectl <verb> <resource> <name> [flags]
```

Examples:

```bash
kubectl get pods
kubectl describe pod nginx-pod
kubectl delete pod nginx-pod
kubectl apply -f deployment.yml
```

- Verb: `get`, `describe`, `apply`, `delete`, `logs`, `exec`.
- Resource: `pods`, `deployments`, `services`, `configmaps`.
- Name: optional specific resource name.
- Flags: namespace, output format, file path, watch, etc.

## 3.2 Namespace and output flags

```bash
kubectl get pods                    # Current/default namespace
kubectl get pods -n dev             # Specific namespace
kubectl get pods -A                 # All namespaces
kubectl get pods -o wide            # Extra columns
kubectl get pod nginx-pod -o yaml   # Full YAML
kubectl get pods --show-labels      # Show labels
kubectl get pods -w                 # Watch changes
kubectl get events --sort-by=.metadata.creationTimestamp
```

| Flag | Purpose |
|---|---|
| `-n` | Select namespace |
| `-A` | All namespaces |
| `-o wide` | Extra information |
| `-o yaml` | YAML representation |
| `-o json` | JSON representation |
| `--show-labels` | Display labels |
| `-w` | Watch resource changes |
| `--sort-by` | Sort output by a field |

## 3.3 Explain/apply/delete

```bash
kubectl explain pod
kubectl explain deployment.spec
kubectl apply -f file.yml
kubectl delete -f file.yml
kubectl get pod <name> -o yaml
```

- `explain`: inspect resource fields/schema.
- `apply`: create/update from desired-state manifest.
- `delete -f`: delete resources described by a manifest.
- `get -o yaml`: inspect current object configuration.


---

# 4. Day 51 — Pods, Manifests, Labels and Validation

## 4.1 What is a Pod?

A Pod is the smallest deployable Kubernetes workload unit. It contains one or more tightly coupled containers that share the Pod network and can share mounted volumes.

Kubernetes schedules and manages Pods as the workload placement unit.

### Simple Pod manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

| Field | Purpose |
|---|---|
| `apiVersion` | API group/version for the resource |
| `kind` | Resource type |
| `metadata.name` | Resource name |
| `metadata.namespace` | Namespace |
| `metadata.labels` | Key/value metadata for organizing/selecting resources |
| `spec` | Desired configuration |
| `spec.containers` | Container definitions |
| `name` | Container name inside Pod |
| `image` | Container image |
| `containerPort` | Documents the port the container is expected to listen on; does not expose it by itself |

## 4.2 Imperative vs declarative

Imperative: tell Kubernetes what action to perform.

```bash
kubectl run redis-pod --image=redis:latest
kubectl run nginx-pod --image=nginx:latest
```

Declarative: define desired state in YAML and apply it.

```bash
kubectl apply -f manifests/nginx-pod.yml
```

| Imperative | Declarative |
|---|---|
| Direct command performs an action | Manifest declares desired state |
| Fast for one-off tests | Better for repeatability, review and version control |
| Example: `kubectl run` | Example: `kubectl apply -f` |

## 4.3 Inspect and troubleshoot Pods

```bash
# List Pods in current namespace
kubectl get pods

# Show node and IP information
kubectl get pods -o wide

# Inspect configuration, conditions, labels, node assignment and events
kubectl describe pod nginx-pod

# Read container stdout/stderr
kubectl logs nginx-pod

# Follow logs continuously
kubectl logs -f nginx-pod

# Access a container shell
kubectl exec -it nginx-pod -- /bin/bash

# If bash is not installed in the image, use sh
kubectl exec -it nginx-pod -- /bin/sh

# Run one command inside a container
kubectl exec nginx-pod -- nginx -v

# Test Nginx from inside its own container
kubectl exec nginx-pod -- curl localhost:80
```

For a multi-container Pod, specify the container:

```bash
kubectl logs <pod-name> -c <container-name>
kubectl exec -it <pod-name> -c <container-name> -- sh
```

### `kubectl logs` vs `kubectl exec`

| Command | Purpose |
|---|---|
| `kubectl logs` | Read stdout/stderr produced by the container |
| `kubectl exec` | Run a process/command inside the container |

## 4.4 Container command and process

A container remains running while its main process remains running.

Example BusyBox Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox-pod
spec:
  containers:
    - name: busybox
      image: busybox:latest
      command: ["sh", "-c", "echo Hello from BusyBox && sleep 3600"]
```

Why `sleep 3600`?
- `echo` prints the message and exits.
- Without another long-running process, the container exits.
- `sleep 3600` keeps its main process alive for one hour so it can be inspected.

```bash
kubectl exec -it busybox-pod -- ps
kubectl logs busybox-pod
```

## 4.5 Generate YAML using dry-run

```bash
# Print generated YAML without creating the Pod
kubectl run test-pod --image=nginx --dry-run=client -o yaml

# Save generated YAML to a file
kubectl run test-pod --image=nginx \
  --dry-run=client -o yaml > test-pod.yml

# Inspect the file
cat test-pod.yml
```

This is useful when I know the imperative command but want a starter manifest to customize.

## 4.6 Client-side vs server-side dry-run

```bash
# Local/client-side preview
kubectl apply -f manifests/nginx-pod.yml --dry-run=client

# Ask API Server to validate without persisting resource
kubectl apply -f manifests/nginx-pod.yml --dry-run=server
```

| Client-side dry-run | Server-side dry-run |
|---|---|
| Does not send the request to API Server | Sends request to API Server for validation |
| Useful for local preview | Tests server-side schema/admission validation |
| Does not prove API Server will accept it | Closer to server validation without storing object |

**Practical Day 51 lesson:** an invalid manifest with an empty `image:` field was accepted by client-side dry-run in that test, while server-side dry-run rejected it with `spec.containers[0].image: Required value`.

**Remember:** successful client-side dry-run does not guarantee server acceptance.

## 4.7 Labels and selectors

A label is a key/value pair attached to a Kubernetes object. Labels help group and select resources.

Example:

```yaml
metadata:
  labels:
    app: nginx
    environment: dev
    team: devops
```

Commands:

```bash
# Display labels
kubectl get pods --show-labels

# Select Pods by label
kubectl get pods -l app=nginx
kubectl get pods -l environment=dev

# Add/change a label
kubectl label pod nginx-pod environment=production

# Remove a label (trailing hyphen)
kubectl label pod nginx-pod environment-
```

Why labels matter: Services and workload controllers use selectors to identify Pods to manage or route traffic to.

**Important:** In a Deployment, `spec.selector.matchLabels` must match `spec.template.metadata.labels`.

## 4.8 Pod deletion and lifecycle

```bash
kubectl delete pod nginx-pod
kubectl delete pod nginx-pod --wait=true
kubectl delete -f manifests/nginx-pod.yml
```

A standalone Pod is not automatically recreated just because it was deleted. A Deployment or StatefulSet controller creates replacement Pods to maintain its desired workload.

---

# 5. Day 52 — Namespaces, Deployments and Rollouts

## 5.1 Namespaces

A Namespace logically groups and scopes namespaced Kubernetes resources in the same cluster. It is not a separate cluster.

Default namespaces observed in the training cluster:

| Namespace | Purpose |
|---|---|
| `default` | Where resources are created when no namespace is specified |
| `kube-system` | Kubernetes internal/system components |
| `kube-public` | Publicly readable cluster information |
| `kube-node-lease` | Node heartbeat/lease objects |
| `local-path-storage` | Local-path storage provisioner namespace in this KIND cluster |

Commands:

```bash
kubectl get namespaces

# Create namespaces imperatively
kubectl create namespace dev
kubectl create namespace staging

# Create namespace declaratively
kubectl apply -f manifests/namespaces.yml

# List Pods in a namespace
kubectl get pods -n dev

# List all namespaces
kubectl get pods -A

# Create a Pod in a namespace
kubectl run nginx-dev --image=nginx:latest -n dev

# Change default namespace for current context
kubectl config set-context --current --namespace=dev

# Delete namespace and namespaced resources inside it
kubectl delete namespace dev
```

Namespace manifest:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

**Caution:** deleting a Namespace deletes namespaced application resources inside it. It is a broad cleanup action.

## 5.2 What is a Deployment?

A Deployment manages ReplicaSets, and ReplicaSets maintain the desired number of Pods.

```text
Deployment
    |
    v
ReplicaSet
    |
    v
Pods
```

A Deployment solves the standalone Pod limitation: when a managed Pod is deleted or fails, its ReplicaSet creates a replacement. Deployments also support scaling, rollout updates and rollback.

### Deployment manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.24
          ports:
            - containerPort: 80
```

| Field | Purpose |
|---|---|
| `apiVersion: apps/v1` | Deployment API version |
| `kind: Deployment` | Resource type |
| `metadata.name` | Deployment name |
| `metadata.namespace` | Namespace |
| `spec.replicas` | Desired Pod count |
| `spec.selector.matchLabels` | Identifies Pods managed by Deployment |
| `spec.template` | Pod blueprint |
| `template.metadata.labels` | Labels on newly created Pods |
| `containers.image` | Container image |

### Apply and inspect

```bash
kubectl apply -f manifests/nginx-deployment.yaml
kubectl get deployments -n dev
kubectl get replicasets -n dev
kubectl get pods -n dev
kubectl describe deployment nginx-deployment -n dev
```

Deployment output columns:

| Column | Meaning |
|---|---|
| READY | Ready replicas / desired replicas |
| UP-TO-DATE | Replicas using the latest Pod template |
| AVAILABLE | Replicas currently available to serve |

## 5.3 Self-healing test

```bash
kubectl get pods -n dev
kubectl delete pod <one-deployment-managed-pod> -n dev
kubectl get pods -n dev -w
```

The ReplicaSet detects fewer Pods than desired and creates a replacement with a different generated name.

## 5.4 Scale a Deployment

Imperative scaling:

```bash
# Scale to five replicas
kubectl scale deployment nginx-deployment --replicas=5 -n dev

# Scale down to two replicas
kubectl scale deployment nginx-deployment --replicas=2 -n dev

# Verify
kubectl get deployment nginx-deployment -n dev
kubectl get pods -n dev
```

Declarative scaling changes the `replicas` field in YAML and applies the file:

```yaml
spec:
  replicas: 4
```

```bash
kubectl apply -f manifests/nginx-deployment.yaml
```

## 5.5 Rolling update

A rolling update gradually replaces old Pods with Pods created from the updated Pod template.

Day 52 image update:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev
```

Monitor:

```bash
kubectl rollout status deployment/nginx-deployment -n dev
kubectl get pods -n dev
kubectl get replicasets -n dev
```

Check rollout history:

```bash
kubectl rollout history deployment/nginx-deployment -n dev
```

### RollingUpdate strategy

The Day 60 WordPress Deployment used:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

| Setting | Meaning |
|---|---|
| `maxUnavailable` | Maximum desired Pods allowed to be unavailable during update |
| `maxSurge` | Maximum extra Pods allowed above desired count during update |

## 5.6 Rollback

Undo most recent rollout:

```bash
kubectl rollout undo deployment/nginx-deployment -n dev
```

Rollback to a specific revision:

```bash
kubectl rollout history deployment/nginx-deployment -n dev
kubectl rollout undo deployment/nginx-deployment --to-revision=1 -n dev
```

Other rollout commands:

```bash
kubectl rollout pause deployment/nginx-deployment -n dev
kubectl rollout resume deployment/nginx-deployment -n dev
kubectl rollout restart deployment/nginx-deployment -n dev
```

**Remember:** `kubectl rollout undo` restores an earlier Deployment Pod template. It does not restore external data or undo database migrations.

---

# 6. Day 53 — Services and Service Discovery

## 6.1 Why do we need a Service?

Pod IP addresses are temporary. A Pod can be deleted and replaced with a new IP.

A Service gives a stable network abstraction and routes traffic to eligible backend Pods selected by labels.

```text
Client
  |
  v
Stable Service IP/DNS
  |
  v
Service selector
  |
  +------ Pod A
  +------ Pod B
  +------ Pod C
```

## 6.2 containerPort vs port vs targetPort vs nodePort

| Field | Where it belongs | Meaning |
|---|---|---|
| `containerPort` | Pod/container manifest | Documents the port the container listens on; does not expose it by itself |
| `port` | Service | Port exposed by Service |
| `targetPort` | Service | Destination port on selected Pod |
| `nodePort` | NodePort Service | Port exposed on eligible nodes |

Traffic:

```text
Client -> NodeIP:30080 -> Service port 80 -> Pod targetPort 80 -> NGINX
```

## 6.3 ClusterIP

ClusterIP is the default Service type. It exposes a Service through an internal cluster IP and is commonly used for internal communication.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
```

Commands:

```bash
kubectl apply -f clusterip-service.yaml
kubectl get services
kubectl describe svc web-app-clusterip
kubectl get endpoints web-app-clusterip
kubectl get endpointslices -l kubernetes.io/service-name=web-app-clusterip
```

Test inside cluster:

```bash
kubectl run test-client --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside BusyBox:

```sh
wget -qO- http://web-app-clusterip
```

## 6.4 Kubernetes Service DNS

CoreDNS provides DNS-based Service discovery inside the cluster.

Typical Service DNS:

```text
<service-name>.<namespace>.svc.cluster.local
```

Example:

```text
web-app-clusterip.default.svc.cluster.local
```

Check CoreDNS:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system
```

Test from a temporary Pod:

```bash
kubectl run dns-test --image=busybox:latest --rm -it --restart=Never -- sh
```

Inside:

```sh
nslookup web-app-clusterip
```

## 6.5 NodePort

NodePort exposes a Service through a port on each eligible node. Standard Kubernetes NodePort range is 30000–32767.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Commands:

```bash
kubectl apply -f nodeport-service.yaml
kubectl get services
kubectl describe svc web-app-nodeport
kubectl get endpoints web-app-nodeport
```

Day 53 KIND test:

```bash
docker ps
docker exec -it devops-cluster-worker bash
```

Inside worker:

```bash
curl http://localhost:30080
```

**Important lab finding:** KIND was created without `extraPortMappings`, so Docker Desktop did not publish port 30080 to Windows. The NodePort was verified inside the KIND worker; direct Windows host access was not configured or verified.

## 6.6 LoadBalancer

LoadBalancer requests an external load-balancing implementation from the cluster infrastructure.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-app-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
    - port: 80
      targetPort: 80
```

```bash
kubectl apply -f loadbalancer-service.yaml
kubectl get svc web-app-loadbalancer
kubectl describe svc web-app-loadbalancer
```

**Day 53 KIND observation:** External IP stayed `<pending>` because no external LoadBalancer integration was configured. This is expected for that local lab.

In AWS EKS, a LoadBalancer Service can integrate with AWS infrastructure to provision an NLB when the appropriate integration/controller is configured. An ALB is commonly provisioned via AWS Load Balancer Controller with Ingress/Gateway resources.

## 6.7 Compare Service types

| Feature | ClusterIP | NodePort | LoadBalancer |
|---|---|---|---|
| Internal ClusterIP | Yes | Yes | Yes |
| NodePort | No | Yes | Usually allocated |
| External LB requested | No | No | Yes |
| Internal cluster access | Yes | Yes | Yes |
| Typical use | Internal app/database access | Basic node-level exposure | External infrastructure LB |

## 6.8 Endpoints and EndpointSlices

EndpointSlices hold backend endpoint information for Services. Kubernetes updates them as matching Pods become eligible or unavailable.

```bash
# Legacy Endpoints view
kubectl get endpoints web-app-clusterip

# Preferred modern EndpointSlice view
kubectl get endpointslices -l kubernetes.io/service-name=web-app-clusterip
```

In the Day 53 lab, the three Services selected the same three NGINX Pods, so their EndpointSlices contained the same three backend Pod IPs and port 80.

---

# 7. Day 54 — ConfigMaps and Secrets

## 7.1 ConfigMap

A ConfigMap stores non-sensitive configuration as key-value data.

### Why use ConfigMaps?

They separate configuration from the container image and application code. The same image can run in dev, staging or production with different configuration.

Examples:
- Application environment.
- Feature flags.
- Port values.
- NGINX configuration files.

Create from literal values:

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```

Inspect:

```bash
kubectl get configmaps
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

ConfigMap values are strings, including values that look numeric or Boolean.

### Create from a file

```bash
kubectl create configmap nginx-config \
  --from-file=default.conf=day-54/config/default.conf
```

The filename becomes the ConfigMap key, and the file contents become the value.

### Consume as environment variables

One key:

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

All keys:

```yaml
envFrom:
  - configMapRef:
      name: wordpress-config
```

Verify:

```bash
kubectl exec <pod-name> -- printenv APP_ENV
kubectl exec <pod-name> -- printenv APP_PORT
```

### Consume as a volume

```yaml
volumeMounts:
  - name: nginx-config-volume
    mountPath: /etc/nginx/conf.d
    readOnly: true

volumes:
  - name: nginx-config-volume
    configMap:
      name: nginx-config
```

Verify file:

```bash
kubectl exec <pod-name> -- cat /etc/nginx/conf.d/default.conf
kubectl exec <pod-name> -- curl -s http://localhost/health
```

## 7.2 ConfigMap update behavior

| Consumption method | Behavior after ConfigMap update |
|---|---|
| Environment variable | Existing container does not receive new environment value; restart/recreate to use it |
| Mounted ConfigMap volume | Projected files update eventually |
| Mounted using `subPath` | Does not receive automatic update |

A mounted file update does not guarantee the application reloads it. The app must reread/reload the file.

Update a ConfigMap using a generated manifest piped to apply:

```bash
kubectl create configmap live-config \
  --from-literal=message="Hello from updated config" \
  --dry-run=client -o yaml | kubectl apply -f -
```

This is useful for changing a key without manually editing YAML. The Day 54 test observed live volume file changes without Pod restart; the Pod's logs then showed subsequent values after the projected file updated.

## 7.3 Secret

A Secret stores sensitive values such as database passwords, tokens, certificates and keys.

Create a generic Secret:

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_USER=<username> \
  --from-literal=DB_PASSWORD='<password>'
```

Inspect metadata safely:

```bash
kubectl get secrets
kubectl describe secret db-secret
```

A raw YAML view may display Base64 data:

```bash
kubectl get secret db-secret -o yaml
```

### Base64 is not encryption

```bash
echo -n 'admin' | base64
echo -n 'YWRtaW4=' | base64 --decode
```

Base64 encodes data; anyone who can read the Secret can decode it.

### Secret using stringData

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: capstone
type: Opaque
stringData:
  MYSQL_ROOT_PASSWORD: "<local-secret-value>"
  MYSQL_DATABASE: "wordpress"
  MYSQL_USER: "wordpress"
  MYSQL_PASSWORD: "<local-secret-value>"
```

`stringData` lets me write string values directly in YAML; Kubernetes converts them into Secret data when storing the object.

**Never commit real passwords.** In the capstone, `manifests/mysql-secret.yml` was excluded from Git.

### Consume one Secret key as environment variable

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

### Consume all Secret keys

```yaml
envFrom:
  - secretRef:
      name: mysql-secret
```

### Consume Secret as a volume

```yaml
volumeMounts:
  - name: db-secret-volume
    mountPath: /etc/db-credentials
    readOnly: true

volumes:
  - name: db-secret-volume
    secret:
      secretName: db-secret
```

Verify files exist without printing credentials:

```bash
kubectl exec <pod-name> -- ls -l /etc/db-credentials
```

## 7.4 ConfigMap vs Secret

| Feature | ConfigMap | Secret |
|---|---|---|
| Purpose | Non-sensitive configuration | Sensitive information |
| Examples | Port, feature flags, app mode | Passwords, tokens, certificates |
| Environment variables | Yes | Yes |
| Volume mounts | Yes | Yes |
| Representation | Plain string values | Secret data commonly shown Base64-encoded |
| Mounted file updates | Eventually | Eventually |

Security reminders:
- Base64 is not encryption.
- Enable encryption at rest for Secrets in etcd in production.
- Restrict Secret access with RBAC.
- Do not print credentials into terminal logs/screenshots.
- Keep Secret manifests and kubeconfigs out of Git.

---

# 8. Day 55 — Volumes, PV, PVC and StorageClasses

## 8.1 Why persistent storage?

A container writable filesystem is ephemeral. If the Pod is deleted and recreated, data in its temporary filesystem may be lost.

Kubernetes storage objects separate storage lifecycle from Pod lifecycle.

## 8.2 emptyDir

`emptyDir` is a temporary volume created for a Pod.

- Data survives a container restart while the same Pod still exists.
- Data is removed when the Pod is deleted.

Use it for temporary scratch/cache data, not durable database files.

## 8.3 PersistentVolume (PV)

A PV represents provisioned storage for use in a cluster.

- PV is cluster-scoped.
- It represents a piece of storage.
- It describes capacity metadata, access modes, reclaim policy and backend.

## 8.4 PersistentVolumeClaim (PVC)

A PVC is a namespaced request for storage made by a workload.

- PVC is namespaced.
- It requests storage capacity and access modes.
- Kubernetes matches it to a suitable PV or dynamically provisions one through a StorageClass.

### PV vs PVC

| PV | PVC |
|---|---|
| Represents provisioned storage | Requests storage |
| Cluster-scoped | Namespaced |
| Defines backend and capacity | Defines requested capacity/access |
| Can be static/dynamic | Consumed by Pods |

## 8.5 Access modes

| Access mode | Meaning |
|---|---|
| RWO — ReadWriteOnce | Read-write mounting by a single node at a time (backend semantics apply) |
| ROX — ReadOnlyMany | Read-only mounting by many nodes when supported |
| RWX — ReadWriteMany | Read-write mounting by many nodes when supported |

**Common confusion:** RWO does not simply mean "one Pod only." It describes node-level mounting behavior; storage implementation and workload constraints matter.

## 8.6 Static provisioning

An administrator manually creates a PV first. A PVC then requests matching storage.

Day 55 PV example:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /tmp/k8s-pv-data
    type: DirectoryOrCreate
```

PVC example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 500Mi
```

Apply and inspect:

```bash
kubectl apply -f manual-pv.yml
kubectl get pv
kubectl describe pv manual-pv

kubectl apply -f pvc.yaml
kubectl get pvc
kubectl describe pvc manual-pvc
```

The Day 55 test bound a `500Mi` claim to a `1Gi` PV. The PVC reported the capacity of the bound PV.

## 8.7 Mount a PVC into a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
    - name: busybox
      image: busybox:latest
      command: ["sh", "-c", "echo persistent-data > /data/message.txt && sleep 3600"]
      volumeMounts:
        - name: persistent-storage
          mountPath: /data
  volumes:
    - name: persistent-storage
      persistentVolumeClaim:
        claimName: manual-pvc
```

Test persistence:

```bash
kubectl apply -f pvc-pod.yml
kubectl exec pvc-pod -- cat /data/message.txt

# Delete Pod, not the PVC
kubectl delete pod pvc-pod

# Recreate using the same PVC
kubectl apply -f pvc-pod.yml
kubectl exec pvc-pod -- cat /data/message.txt
```

The data remains because the Pod reuses the same PVC/PV.

**Important:** Do not let the startup command overwrite old data during a persistence test. In Day 55, the startup command was changed before recreating the Pod so it did not replace the original file.

## 8.8 Dynamic provisioning and StorageClass

Dynamic provisioning lets a provisioner create a PV automatically when a PVC requests storage using a StorageClass.

Inspect:

```bash
kubectl get storageclass
kubectl describe storageclass
kubectl get pv
kubectl get pvc
```

Day 55 KIND cluster observed:

| Setting | Value |
|---|---|
| StorageClass | `standard` (default) |
| Provisioner | `rancher.io/local-path` |
| Reclaim policy | `Delete` |
| Volume binding mode | `WaitForFirstConsumer` |
| Expansion | Disabled |

Dynamic PVC example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 500Mi
```

Apply and inspect:

```bash
kubectl apply -f dynamic-pvc.yml
kubectl get pvc
kubectl get pv
```

With `WaitForFirstConsumer`, PVC provisioning/binding may wait until a Pod uses it so scheduling topology can be considered.

**Special setting:** `storageClassName: ""` disables dynamic provisioning for a PVC.

## 8.9 Static vs dynamic provisioning

| Static | Dynamic |
|---|---|
| PV created manually | Provisioner creates PV automatically |
| PVC binds to pre-created PV | PVC request triggers provisioning |
| Example: `manual-pv` | Example: `standard` StorageClass |
| More administrator preparation | Less manual PV creation |

## 8.10 Reclaim policies and PV lifecycle

| Reclaim policy | Behavior after PVC deletion |
|---|---|
| Retain | Underlying storage is kept; PV can become `Released` and may need manual reclamation |
| Delete | PV and provisioned storage are deleted by the provisioner/backend |

PV statuses observed:

```text
Available -> Bound -> Released
```

- `Available`: PV exists and is not claimed.
- `Bound`: PV is matched to a PVC.
- `Released`: PVC was deleted, but retained PV still has claim history and needs reclamation before reuse.

Day 55 practical result:
- Manual PV used `Retain`, so after PVC deletion it remained `Released`.
- Dynamic PV used `Delete`, so it was automatically deleted after PVC deletion.

### hostPath warning

`hostPath` is useful for local learning. It ties storage to a path on a particular node and is not a general production storage solution. If a Pod moves to another node, the data may not be where the Pod expects it.

## 8.11 Storage commands

```bash
kubectl get pv
kubectl get pvc
kubectl get pvc -A
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name>
kubectl get storageclass
kubectl describe storageclass standard
kubectl delete pod <pod-name>
kubectl delete pvc <pvc-name>
kubectl delete pv <pv-name>
```

**Critical:** deleting a PVC may trigger a `Delete` reclaim policy and permanently remove provisioned data. Check reclaim policy and backups first.


---

# 9. Day 56 — StatefulSets and Headless Services

## 9.1 Why a StatefulSet?

A Deployment creates interchangeable Pods with generated names. Stateful applications may require stable identity and an individual persistent storage claim for each replica.

A StatefulSet provides:
- Stable, predictable Pod names.
- Stable ordinal identity (`web-0`, `web-1`, ...).
- Stable network identity when paired with a Headless Service.
- Individual PVCs using `volumeClaimTemplates`.
- Ordered Pod creation and termination by default.

## 9.2 Deployment vs StatefulSet

| Deployment | StatefulSet |
|---|---|
| Usually used for stateless replicas | Used when stable identity/storage association matters |
| Generated Pod names | Stable ordinal Pod names |
| Replicas are interchangeable | Each replica has a stable identity |
| No `volumeClaimTemplates` | Can create a PVC per Pod |
| Example: WordPress frontend | Example: MySQL in Day 60 |

## 9.3 Headless Service

A Headless Service has no virtual ClusterIP. It can provide DNS records for individual Pods selected by the Service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None
  selector:
    app: nginx-stateful
  ports:
    - port: 80
      targetPort: 80
```

Apply and inspect:

```bash
kubectl apply -f headless-service.yml
kubectl get services
kubectl describe service web-headless
```

Expected ClusterIP:

```text
None
```

The Service selector must match the labels on the StatefulSet Pod template.

## 9.4 StatefulSet manifest

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web-headless
  replicas: 3
  selector:
    matchLabels:
      app: nginx-stateful
  template:
    metadata:
      labels:
        app: nginx-stateful
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
          ports:
            - containerPort: 80
          volumeMounts:
            - name: web-data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: web-data
      spec:
        accessModes:
          - ReadWriteOnce
        storageClassName: standard
        resources:
          requests:
            storage: 100Mi
```

Apply and inspect:

```bash
kubectl apply -f web-statefulset.yml
kubectl get statefulsets
kubectl get pods -w
kubectl get pvc
kubectl get pv
```

Pods are created in ordinal order:

```text
web-0
web-1
web-2
```

PVCs are created per Pod:

```text
web-data-web-0
web-data-web-1
web-data-web-2
```

## 9.5 StatefulSet Pod DNS

With a Headless Service, a StatefulSet Pod has a stable DNS name:

```text
<pod-name>.<headless-service>.<namespace>.svc.cluster.local
```

Day 56 example:

```text
web-0.web-headless.default.svc.cluster.local
web-1.web-headless.default.svc.cluster.local
web-2.web-headless.default.svc.cluster.local
```

Day 60 MySQL example:

```text
mysql-0.mysql.capstone.svc.cluster.local
```

Test from a temporary BusyBox Pod:

```bash
kubectl run dns-test --image=busybox:latest --restart=Never --command -- sleep 3600

kubectl exec dns-test -- nslookup web-0.web-headless.default.svc.cluster.local
```

## 9.6 Persistence after Pod deletion

```bash
# Write data to the StatefulSet Pod's mounted PVC
kubectl exec web-0 -- sh -c 'echo stable-data > /usr/share/nginx/html/index.html'

# Delete Pod
kubectl delete pod web-0

# Watch StatefulSet recreate it
kubectl get pods -w

# Verify same name and persistent data
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
```

The StatefulSet recreates `web-0` and reattaches its existing PVC `web-data-web-0`.

## 9.7 StatefulSet scaling

```bash
# Scale up
kubectl scale statefulset web --replicas=5

# Verify Pods and claims
kubectl get pods
kubectl get pvc

# Scale down
kubectl scale statefulset web --replicas=3

# Verify remaining Pods and all claims
kubectl get pods
kubectl get pvc
```

Day 56 observation:
- Scale-up created `web-3` then `web-4`.
- Scale-down removed higher ordinals first: `web-4`, then `web-3`.
- PVCs for scaled-down replicas remained.

**Important:** Scaling down a StatefulSet does not automatically delete its PVCs. This protects data from being casually removed. Delete a PVC only when its data is intentionally no longer needed.

---

# 10. Day 57 — Requests, Limits, QoS and Health Probes

## 10.1 Resource requests

Requests define the amount of CPU and memory Kubernetes uses when scheduling a Pod.

The scheduler checks whether a node has sufficient allocatable resources for the requests.

## 10.2 Resource limits

Limits cap container resource use.

- CPU limit: CPU is throttled when the container tries to exceed the configured limit.
- Memory limit: exceeding the memory limit can cause the container to be terminated by the OOM mechanism.

Example from the Day 57 resource Pod:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

## 10.3 CPU and memory units

| Quantity | Meaning |
|---|---|
| `100m` | 100 millicores = 0.1 CPU core |
| `250m` | 0.25 CPU core |
| `1000m` | 1 CPU core |
| `128Mi` | 128 mebibytes |
| `1Gi` | 1 gibibyte |

**Critical mistake from Day 57:** `cpu: "100"` means 100 CPU cores, not 100m. `memory: "128Gi"` requests 128 GiB. Incorrectly large requests can keep a Pod Pending.

## 10.4 QoS classes

Kubernetes classifies Pods into Quality of Service classes:

| Class | Rule / meaning |
|---|---|
| Guaranteed | Every container has CPU and memory requests equal to their respective limits |
| Burstable | Some resource requests/limits are set, but Guaranteed criteria are not met |
| BestEffort | No CPU or memory requests/limits are configured |

Inspect a Pod QoS class:

```bash
kubectl describe pod resource-pod
```

Look for:

```text
QoS Class: Burstable
```

## 10.5 OOMKilled

OOM means Out Of Memory. When a container exceeds its memory limit, the Linux OOM mechanism can terminate a process.

Day 57 test:
- Stress process attempted about 200 MB.
- Container memory limit was 100Mi.
- Observed termination reason: `OOMKilled`.
- Observed exit code: `137`.

Check the last termination reason:

```bash
kubectl get pod oom-pod -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}{"\n"}{.status.containerStatuses[0].lastState.terminated.exitCode}{"\n"}'
```

**Remember:** 137 is commonly `128 + 9` (SIGKILL), but exit code 137 alone does not prove OOM. Check `lastState.terminated.reason` or `kubectl describe pod`.

## 10.6 Pending Pod due to insufficient resources

A Pod can stay Pending if the scheduler cannot find a node that satisfies its requests or other scheduling rules.

Inspect:

```bash
kubectl get pods -o wide
kubectl describe pod pending-pod
kubectl get nodes
kubectl describe node <node-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Look for `FailedScheduling`, `Insufficient cpu`, `Insufficient memory`, or untolerated taints.

## 10.7 Liveness probe

A liveness probe determines whether a container is still healthy. If it fails enough times, Kubernetes restarts the container.

Day 57 used an `exec` probe:

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
  periodSeconds: 5
  failureThreshold: 3
```

The container created `/tmp/healthy`, waited 30 seconds, removed it, and kept running. The liveness probe failed three times and Kubernetes restarted the container.

## 10.8 Readiness probe

A readiness probe decides whether a Pod is ready to receive Service traffic.

Day 57 used an HTTP probe:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
  periodSeconds: 5
```

When the NGINX welcome page was removed, HTTP readiness failed. Observed:
- Pod remained Running.
- Ready changed to `0/1`.
- Restart count stayed at 0.
- Service endpoints became empty.

This prevents Services from routing traffic to a Pod that is not ready.

## 10.9 Startup probe

A startup probe gives a slow-starting application time to initialize.

```yaml
startupProbe:
  exec:
    command:
      - cat
      - /tmp/started
  periodSeconds: 5
  failureThreshold: 12
```

The Day 57 container waited 20 seconds and then created `/tmp/started`. During startup, the probe failed four times but the container was not restarted because the threshold was 12.

While startup probe has not succeeded, Kubernetes does not run liveness/readiness probes for that container.

Approximate startup failure budget:

```text
periodSeconds × failureThreshold
5 seconds × 12 = 60 seconds
```

## 10.10 Liveness vs Readiness vs Startup

| Probe | Question | On failure |
|---|---|---|
| Startup | Has application finished starting? | Container can be restarted after threshold; liveness/readiness are held back until startup succeeds |
| Liveness | Is container still healthy? | Container is restarted after threshold |
| Readiness | Can Pod receive traffic now? | Pod is marked not ready and removed from eligible Service endpoints |

**Interview memory trick:**
- Startup = give it time to boot.
- Liveness = restart if unhealthy.
- Readiness = do not send traffic yet.

## 10.11 Inspect resources and probes

```bash
kubectl describe pod <pod-name>
kubectl get pod <pod-name> -o yaml
kubectl get pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl logs <pod-name>
```


---

# 11. Day 58 — Metrics Server and HPA

## 11.1 Metrics Server

Metrics Server collects CPU and memory usage metrics from kubelets and exposes them through the Kubernetes Metrics API.

HPA uses these metrics for resource-based autoscaling.

**Metrics Server collects metrics; HPA makes scaling decisions.**

### Install Metrics Server

Day 58 downloaded the upstream manifest:

```bash
curl -LO https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

For the local KIND lab, this argument was added to the Metrics Server container:

```yaml
- --kubelet-insecure-tls
```

It was a local lab workaround for kubelet TLS certificate verification.

**Security note:** `--kubelet-insecure-tls` disables kubelet certificate verification. It was used only as a local KIND workaround; production clusters should use properly configured and trusted certificates.

Apply and verify:

```bash
kubectl apply -f components.yaml
kubectl get pods -n kube-system | grep metrics-server
kubectl get deployment metrics-server -n kube-system
kubectl get apiservice v1beta1.metrics.k8s.io
```

## 11.2 kubectl top

```bash
# Node CPU and memory
kubectl top nodes

# Pod CPU and memory in current namespace
kubectl top pods

# All Pods across namespaces
kubectl top pods -A

# Sort all Pods by CPU
kubectl top pods -A --sort-by=cpu
```

| Information | Meaning |
|---|---|
| Requests | Scheduling baseline |
| Limits | Container resource ceiling |
| `kubectl top` | Actual observed CPU and memory consumption |

Metric values change over time. Do not memorize the sample CPU/memory numbers from one run.

## 11.3 Why CPU requests matter for HPA

CPU utilization percentage is calculated relative to the CPU request, not the CPU limit.

Example:
- CPU request = 200m.
- HPA target = 50%.
- Target CPU usage = 100m per Pod.

```text
200m × 50% = 100m
```

Without a CPU request for the relevant container, HPA cannot calculate CPU utilization percentage reliably; the metric can appear unknown.

## 11.4 Horizontal Pod Autoscaler (HPA)

HPA automatically changes a target workload's replica count based on configured metrics.

Day 58 used PHP-Apache to demonstrate CPU-based scaling.

### PHP-Apache Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-apache
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
        - name: php-apache
          image: registry.k8s.io/hpa-example
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 200m
```

Apply and inspect:

```bash
kubectl apply -f php-apache-deployment.yml
kubectl get deployments
kubectl get pods -l app=php-apache
kubectl top pods -l app=php-apache
```

Create a stable Service endpoint:

```bash
kubectl expose deployment php-apache --port=80
kubectl get services php-apache
kubectl get endpoints php-apache
```

## 11.5 Imperative HPA creation

Day 58 command:

```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

This created an HPA with:
- CPU target: 50%.
- Minimum replicas: 1.
- Maximum replicas: 10.

Inspect:

```bash
kubectl get hpa
kubectl describe hpa php-apache
```

**Version note from the lab:** kubectl displayed a deprecation warning for `--cpu-percent` and recommended the newer `--cpu` syntax. The command still worked in the exercise. For repeatable project work, prefer a YAML manifest using `autoscaling/v2`.

## 11.6 Generate load and observe HPA

Day 58 BusyBox load generator repeatedly sent HTTP requests to the Service:

```bash
kubectl run load-generator \
  --image=busybox:1.36 \
  --restart=Never \
  -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
```

Monitor:

```bash
kubectl get hpa php-apache --watch
kubectl get deployment php-apache --watch
kubectl get pods -l app=php-apache -w
```

Day 58 lab result:
- Initial replicas: 1.
- Under load: 10 replicas.
- Configured maximum: 10.

Stop load:

```bash
kubectl delete pod load-generator
```

Scale-down may not happen immediately because HPA uses a stabilization window and scaling policies.

## 11.7 HPA formula

Basic formula:

```text
desiredReplicas =
ceil(currentReplicas × (currentMetric / targetMetric))
```

Example:

```text
Current replicas = 1
Current CPU utilization = 150%
Target CPU utilization = 50%

Desired replicas = ceil(1 × 150 / 50)
                 = 3
```

Actual scaling is also affected by policies, stabilization windows, minimum/maximum replicas, available resources and metric availability.

## 11.8 Declarative HPA — autoscaling/v2

Day 58 used this type of manifest:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
```

Apply and inspect:

```bash
kubectl apply -f php-apache-hpa.yml
kubectl get hpa
kubectl describe hpa php-apache
```

### Important HPA fields

| Field | What it controls |
|---|---|
| `scaleTargetRef` | Deployment/workload HPA controls |
| `minReplicas` | Lowest allowed replica count |
| `maxReplicas` | Highest allowed replica count |
| `metrics` | Metric used for scaling |
| `averageUtilization` | Target average utilization relative to resource request |
| `behavior.scaleUp` | Scale-up rate/stabilization |
| `behavior.scaleDown` | Scale-down rate/stabilization |

### HPA behavior from Day 58

Scale-up:

```yaml
scaleUp:
  stabilizationWindowSeconds: 0
  policies:
    - type: Percent
      value: 100
      periodSeconds: 15
```

This allows up to a 100% increase in the current replica count during a 15-second policy period, subject to HPA calculations and available cluster capacity.

Scale-down:

```yaml
scaleDown:
  stabilizationWindowSeconds: 300
```

A five-minute stabilization window helps avoid scaling down too quickly. HPA considers recent recommendations and uses the highest recommendation within the window.

## 11.9 autoscaling/v1 vs autoscaling/v2

| Feature | autoscaling/v1 | autoscaling/v2 |
|---|---|---|
| CPU utilization | Supported | Supported |
| Memory resource metrics | Not supported in v1 | Supported |
| Multiple metrics | Not supported in v1 | Supported |
| Custom scaling behavior | Not supported in v1 | Supported |
| Scale-up/down policies | Not configurable in v1 | Configurable |

### Verify HPA and metrics

```bash
kubectl get apiservice v1beta1.metrics.k8s.io
kubectl top nodes
kubectl top pods -n <namespace>
kubectl get hpa -n <namespace>
kubectl describe hpa <hpa-name> -n <namespace>
```

**HPA vs Cluster Autoscaler:**
- HPA scales application Pod replicas.
- Cluster Autoscaler adjusts cluster node count when configured and supported.
- HPA does not create worker nodes.

---

# 12. Day 59 — Helm: Kubernetes Package Manager

## 12.1 What is Helm?

Helm is a package manager for Kubernetes. It simplifies installing, configuring, upgrading and managing applications by packaging Kubernetes templates into reusable Charts.

Instead of maintaining many YAML files manually, Helm renders templates using configurable values and creates Kubernetes resources.

Day 59 lab used Helm v4.3.1 and the Bitnami chart repository.

## 12.2 Core Helm concepts

| Concept | Meaning |
|---|---|
| Chart | Package containing templates, default values and metadata |
| Release | Installed instance of a chart |
| Repository | Location that stores/distributes charts |
| `values.yaml` | Default configuration values |
| `Chart.yaml` | Chart metadata and versions |
| `templates/` | Go templates that render Kubernetes manifests |

One chart can be installed as multiple releases:

```bash
helm install my-nginx bitnami/nginx
helm install custom-nginx bitnami/nginx
```

These are separate releases using the same chart.

## 12.3 Install and verify Helm

```bash
helm version
helm env
```

- `helm version`: installed Helm version.
- `helm env`: Helm cache, configuration and data directories.

## 12.4 Add repositories and search charts

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm repo list
helm search repo nginx
helm search repo bitnami
```

| Command | Purpose |
|---|---|
| `helm repo add` | Register a chart repository |
| `helm repo update` | Refresh local repository indexes |
| `helm repo list` | List configured repositories |
| `helm search repo` | Search charts in configured repositories |

`helm repo update` refreshes the chart index. It does not upgrade already-installed releases.

## 12.5 Inspect and install a chart

```bash
# Inspect default chart values
helm show values bitnami/nginx

# Install a chart: my-nginx is the release name
helm install my-nginx bitnami/nginx

# List releases
helm list

# Inspect release
helm status my-nginx

# Inspect generated Kubernetes manifest stored in release
helm get manifest my-nginx

# Inspect Kubernetes objects
kubectl get all
```

The Bitnami NGINX chart version observed in the lab was 25.2.14 and app version 1.31.8; these are historical lab values and can change in a repository.

## 12.6 Customize values with --set or -f

Single/short overrides:

```bash
helm install custom-nginx bitnami/nginx \
  --set replicaCount=3 \
  --set service.type=NodePort
```

Use a values file for larger/reusable configuration:

```yaml
replicaCount: 2

service:
  type: NodePort

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 250m
    memory: 256Mi
```

Install with the values file:

```bash
helm install nginx-from-file bitnami/nginx -f custom-values.yml
```

Inspect user-supplied values:

```bash
helm get values nginx-from-file
```

| Option | Use |
|---|---|
| `--set key=value` | Override a small number of values from command line |
| `-f values.yaml` | Provide multiple configuration values through YAML |

## 12.7 Upgrade and rollback

Upgrade a release:

```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=5
```

Check history:

```bash
helm history my-nginx
```

Rollback to revision 1:

```bash
helm rollback my-nginx 1
```

Verify:

```bash
helm history my-nginx
helm list
kubectl get deployments
kubectl get pods
```

**Important:** Rollback creates a new revision. It does not delete or overwrite later revision history.

Example:

```text
Revision 1: initial install - 1 replica
Revision 2: upgrade - 5 replicas
Revision 3: rollback to revision 1 - 1 replica
```

## 12.8 Create a custom chart

```bash
helm create my-app
```

Typical chart structure:

```text
my-app/
├── Chart.yaml
├── charts/
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── NOTES.txt
└── values.yaml
```

| File/folder | Purpose |
|---|---|
| `Chart.yaml` | Chart metadata/version |
| `values.yaml` | Default configuration |
| `templates/` | Kubernetes resource templates |
| `_helpers.tpl` | Reusable template helpers |
| `charts/` | Chart dependencies |
| `NOTES.txt` | Post-install user instructions |

Chart metadata example:

```yaml
apiVersion: v2
name: my-app
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: "1.16.0"
```

- `version`: chart package version.
- `appVersion`: application version represented by the chart.

## 12.9 Helm Go templates

Example:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}
```

With `replicaCount: 3`, Helm renders:

```yaml
spec:
  replicas: 3
```

Common template expressions:

| Expression | Meaning |
|---|---|
| `{{ .Values.replicaCount }}` | Read setting from values.yaml |
| `{{ .Values.image.repository }}` | Read image repository |
| `{{ .Chart.Name }}` | Chart name |
| `{{ .Release.Name }}` | Current release name |

## 12.10 Validate, render, install and upgrade custom chart

```bash
# Check chart structure/templates
helm lint my-app

# Render templates locally; does not install
helm template my-release ./my-app

# Install
helm install my-release ./my-app

# Upgrade
helm upgrade my-release ./my-app --set replicaCount=5

# Inspect
helm list
helm history my-release
kubectl get deployments
kubectl get pods
kubectl get svc
```

Difference:
- `helm template`: renders YAML locally; does not install a release.
- `helm install`: creates a release and Kubernetes resources.
- `helm lint`: checks for common chart structure/template problems.

## 12.11 Helm cleanup

```bash
helm uninstall my-nginx
helm uninstall custom-nginx
helm uninstall nginx-from-file
helm uninstall my-release
```

Verify:

```bash
helm list
kubectl get deployments
kubectl get pods
kubectl get svc
```

A release uninstall removes chart-managed resources. Do not assume every external data store or manually-created resource is automatically removed or backed up.

---

# 13. Day 60 — WordPress + MySQL Capstone

## 13.1 Project goal

Deploy a WordPress application backed by MySQL on KIND, combining the concepts learned from Days 50–59.

Project repository:

[https://github.com/Aniruddhakharve/kubernetes-wordpress-mysql-capstone](https://github.com/Aniruddhakharve/kubernetes-wordpress-mysql-capstone)

The project was later extended with shared NFS storage, Metrics Server/HPA, Headlamp and RBAC.

## 13.2 Project architecture

```text
KIND CLUSTER
|
+-- namespace: capstone
|   |
|   +-- WordPress Service (NodePort 30080)
|   |       |
|   |       +---- WordPress Deployment
|   |                 |
|   |                 +---- WordPress Pod 1
|   |                 +---- WordPress Pod 2
|   |                         |
|   |                         +---- ConfigMap: wordpress-config
|   |                         +---- Secret: mysql-secret
|   |                         +---- NFS PVC (uploads, RWX)
|   |
|   +-- MySQL Headless Service (clusterIP: None)
|   |       |
|   |       +---- MySQL StatefulSet
|   |                  |
|   |                  +---- mysql-0
|   |                  +---- PVC (RWO, 1Gi)
|   |
|   +-- NFS PV and PVC for WordPress uploads
|   +-- wordpress-hpa
|
+-- namespace: kube-system
|   +-- Metrics Server
|
+-- namespace: headlamp
    +-- Headlamp Helm release
    +-- Viewer ServiceAccount + read-only RBAC
    +-- Admin ServiceAccount + cluster-admin binding
```

## 13.3 Project resources and why they exist

| Component | Resource/type | Problem it solves |
|---|---|---|
| Application namespace | Namespace `capstone` | Groups application resources |
| Database | MySQL StatefulSet | Stable DB Pod identity and persistent storage association |
| Database networking | Headless Service `mysql` | Stable MySQL Pod DNS |
| Database data | PVC via `volumeClaimTemplates` | Keeps DB data across Pod recreation |
| App configuration | ConfigMap `wordpress-config` | Stores non-sensitive DB host/name |
| Credentials | Secret `mysql-secret` | Stores DB credentials |
| App | WordPress Deployment | Manages stateless WordPress replicas |
| App networking | NodePort Service `wordpress-service` | Exposes WordPress on NodePort 30080 |
| Shared media | NFS PV/PVC (RWX) | Shares uploads between replicas |
| Autoscaling | HPA `wordpress-hpa` | Manages WordPress replicas using CPU |
| Metrics | Metrics Server | Supplies node/Pod CPU and memory metrics |
| Dashboard | Headlamp | Browser-based Kubernetes dashboard |
| Access | ServiceAccounts + RBAC | Separates viewer and administrator permissions |

## 13.4 MySQL Headless Service

Actual project manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: capstone
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

Why headless?
- `clusterIP: None` means no virtual Service ClusterIP.
- DNS can resolve the individual StatefulSet Pod using its stable identity.
- WordPress can target `mysql-0.mysql.capstone.svc.cluster.local`.

## 13.5 MySQL StatefulSet

Important actual project settings:

| Property | Value |
|---|---|
| StatefulSet | `mysql` |
| Image | `mysql:8.0` |
| Replicas | 1 |
| Service name | `mysql` |
| Container port | 3306 |
| Secret config | `envFrom` -> `mysql-secret` |
| Data directory | `/var/lib/mysql` |
| PVC template | `mysql-storage` |
| Access mode | RWO |
| StorageClass | `standard` |
| Requested storage | 1Gi |
| CPU request | 250m |
| Memory request | 512Mi |
| CPU limit | 500m |
| Memory limit | 1Gi |

The StatefulSet used:
- `startupProbe` with `mysqladmin ping`.
- `livenessProbe` with `mysqladmin ping`.
- `readinessProbe` running an authenticated `SELECT 1`.

Why three probes?
- Startup gives MySQL enough time to initialize.
- Liveness restarts it if it becomes unhealthy.
- Readiness advertises it as ready only after it can serve an authenticated query.

## 13.6 WordPress ConfigMap

Actual project keys:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-config
  namespace: capstone
data:
  WORDPRESS_DB_HOST: "mysql-0.mysql.capstone.svc.cluster.local:3306"
  WORDPRESS_DB_NAME: "wordpress"
```

The ConfigMap stores non-sensitive DB hostname/name. Username and password are sourced separately from the Secret.

## 13.7 WordPress Secret

Day 60 challenge keys:

- `MYSQL_ROOT_PASSWORD`
- `MYSQL_DATABASE`
- `MYSQL_USER`
- `MYSQL_PASSWORD`

The challenge used `stringData` to write local values as plain strings in the manifest.

**Never commit actual passwords.** In this repository, `manifests/mysql-secret.yml` is excluded from Git.

## 13.8 WordPress Deployment

Actual deployment highlights:

- WordPress image: `wordpress:latest`.
- Deployment label selector: `app: wordpress`.
- ConfigMap consumed using `envFrom`.
- DB username/password consumed using `secretKeyRef`.
- Init container `wait-for-mysql` loops until MySQL DNS resolves and an authenticated `SELECT 1` succeeds.
- WordPress CPU request: 100m; memory request: 128Mi.
- WordPress CPU limit: 500m; memory limit: 512Mi.
- Startup/liveness/readiness probes use HTTP `/wp-login.php` on port 80.
- NFS PVC mounted at `/var/www/html/wp-content/uploads`.
- RollingUpdate: `maxUnavailable: 1`, `maxSurge: 1`.

The init container avoids starting WordPress's database initialization before MySQL accepts authenticated connections.

## 13.9 WordPress Service

Actual manifest:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
  namespace: capstone
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
      protocol: TCP
```

For local browser access, the project used port forwarding:

```bash
kubectl port-forward svc/wordpress-service 8080:80 -n capstone
```

Then open:

```text
http://localhost:8080
```

Direct Windows host access to a KIND NodePort depends on Docker networking/port mappings.

## 13.10 Capstone self-healing and persistence

The challenge tested:
1. Delete a WordPress Pod — Deployment recreates it.
2. Delete `mysql-0` — StatefulSet recreates it with the same name and reattaches the PVC.
3. After MySQL recovers, refresh WordPress — database content should remain.

Commands:

```bash
kubectl get pods -n capstone
kubectl delete pod <wordpress-pod-name> -n capstone
kubectl get pods -n capstone -w

kubectl delete pod mysql-0 -n capstone
kubectl get pods -n capstone -w
kubectl get pvc -n capstone
```

The underlying persistent claim is separate from the Pod lifecycle.

## 13.11 Capstone HPA

Actual manifest: `manifests/wordpress-hpa.yml`

| Setting | Value |
|---|---|
| Target | Deployment `wordpress` |
| API | `autoscaling/v2` |
| CPU target | 50% |
| Minimum replicas | 2 |
| Maximum replicas | 5 |
| Scale-up stabilization | 0 seconds |
| Scale-up policy | Up to 2 Pods per 15 seconds |
| Scale-down stabilization | 300 seconds |
| Scale-down policy | Remove 1 Pod per 30 seconds |

The capstone test demonstrated:

```text
Scale out: 2 -> 3 -> 5
Scale in:  5 -> 4 -> 3 -> 2
```

The static `replicas` field was removed from the WordPress Deployment manifest so HPA could own the desired replica count.

## 13.12 Capstone checks

```bash
kubectl get all -n capstone
kubectl get pods -n capstone -o wide
kubectl get deployments -n capstone
kubectl get statefulsets -n capstone
kubectl get services -n capstone
kubectl get pv
kubectl get pvc -n capstone
kubectl get hpa -n capstone
kubectl top pods -n capstone
kubectl describe hpa wordpress-hpa -n capstone
```

---

# 14. Kubernetes RBAC and Headlamp — Capstone Extension

## 14.1 Headlamp

Headlamp is a web-based Kubernetes dashboard. It was installed in the `headlamp` namespace using Helm chart version 0.46.0.

The chart's automatic `cluster-admin` ClusterRoleBinding was disabled so custom viewer/admin permissions could be configured.

Add repository and inspect:

```bash
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
helm repo update
helm search repo headlamp
helm show values headlamp/headlamp
```

Render before installation:

```bash
helm template headlamp headlamp/headlamp \
  --version 0.46.0 \
  --namespace headlamp \
  --set clusterRoleBinding.create=false \
  > /tmp/headlamp-rendered.yaml
```

Install:

```bash
helm install headlamp headlamp/headlamp \
  --version 0.46.0 \
  --namespace headlamp \
  --create-namespace \
  --set clusterRoleBinding.create=false \
  --set config.unsafeUseServiceAccountToken=false
```

Verify and access:

```bash
helm list -n headlamp
kubectl get all -n headlamp
kubectl get serviceaccount -n headlamp
kubectl port-forward -n headlamp svc/headlamp 8080:80
```

Open `http://localhost:8080`.

## 14.2 RBAC concepts

RBAC = Role-Based Access Control. It controls what users, groups and ServiceAccounts can do with Kubernetes resources.

| Resource | Purpose |
|---|---|
| Role | Defines permissions within a namespace |
| ClusterRole | Defines permissions at cluster scope or reusable permissions |
| RoleBinding | Grants a Role or ClusterRole within a namespace |
| ClusterRoleBinding | Grants a ClusterRole across the cluster |

A Role/ClusterRole defines permissions. A Binding grants them to a subject.

## 14.3 ServiceAccounts

A ServiceAccount provides an identity for a process/client interacting with the Kubernetes API.

Creating a ServiceAccount does not automatically grant it permissions.

The project created:
- `headlamp-viewer`: read-only cluster access.
- `headlamp-admin`: administrator testing.

## 14.4 Viewer ClusterRole

Project viewer role granted read-only verbs:

```yaml
verbs:
  - get
  - list
  - watch
```

Resources:
- Core group: Pods, Services, Namespaces, Nodes, ConfigMaps, Events.
- `apps`: Deployments, ReplicaSets, StatefulSets, DaemonSets.
- `batch`: Jobs and CronJobs.

Apply and inspect:

```bash
kubectl apply -f manifests/headlamp-viewer-sa.yml
kubectl apply -f manifests/headlamp-viewer-role.yml
kubectl apply -f manifests/headlamp-viewer-binding.yml

kubectl get serviceaccounts -n headlamp
kubectl describe clusterrole headlamp-viewer
kubectl describe clusterrolebinding headlamp-viewer-binding
```

## 14.5 Administrator ServiceAccount

The project bound `headlamp-admin` to the built-in `cluster-admin` ClusterRole.

This is highly privileged and was only for administrator testing in this lab. Do not grant cluster-admin to normal dashboard users.

```bash
kubectl apply -f manifests/headlamp-admin.yml
kubectl get serviceaccount headlamp-admin -n headlamp
kubectl get clusterrolebinding headlamp-admin-binding
kubectl describe clusterrolebinding headlamp-admin-binding
```

## 14.6 Test RBAC permissions

`kubectl auth can-i` checks whether an identity is authorized to perform an action on a resource.

Viewer tests:

```bash
kubectl auth can-i list pods --all-namespaces \
  --as=system:serviceaccount:headlamp:headlamp-viewer

kubectl auth can-i delete pods -n capstone \
  --as=system:serviceaccount:headlamp:headlamp-viewer

kubectl auth can-i get secrets -n capstone \
  --as=system:serviceaccount:headlamp:headlamp-viewer
```

Observed/expected viewer result:
- list Pods: yes.
- delete Pods: no.
- get Secrets: no.

Administrator tests:

```bash
kubectl auth can-i list pods --all-namespaces \
  --as=system:serviceaccount:headlamp:headlamp-admin

kubectl auth can-i delete pods -n capstone \
  --as=system:serviceaccount:headlamp:headlamp-admin

kubectl auth can-i get secrets -n capstone \
  --as=system:serviceaccount:headlamp:headlamp-admin
```

The administrator identity was allowed these tested operations because of the `cluster-admin` binding.

## 14.7 Tokens and identity

Generate short-lived tokens:

```bash
kubectl create token headlamp-viewer -n headlamp --duration=1h
kubectl create token headlamp-admin -n headlamp --duration=1h
```

Tokens are credentials. Never commit them to GitHub, documentation, screenshots, chat messages or shell scripts.

Verify the authenticated identity:

```bash
kubectl auth whoami
```

**Important kubeconfig lesson:** the original kubeconfig used client-certificate authentication. An initial token attempt still showed `kubernetes-admin`. A separate temporary kubeconfig was used for token verification so the client-certificate identity did not take precedence.

```bash
TOKEN="$(kubectl create token headlamp-admin -n headlamp --duration=10m)"

export KUBECONFIG="$(cygpath -w /tmp/headlamp-token-test-config)"

kubectl config set-credentials headlamp-token-user --token="$TOKEN"
kubectl config set-context kind-my-cluster --user=headlamp-token-user
kubectl auth whoami
```

Expected identity:

```text
system:serviceaccount:headlamp:headlamp-admin
```

Cleanup temporary authentication state:

```bash
unset TOKEN
rm /tmp/headlamp-token-test-config
unset KUBECONFIG
kubectl config current-context
```

Use an isolated kubeconfig for temporary credential experiments. Avoid modifying the primary kubeconfig.

---

# 15. NFS-Ganesha — Capstone Shared Storage

## 15.1 Why did the capstone need shared storage?

WordPress stores media under:

```text
/var/www/html/wp-content/uploads
```

Multiple WordPress replicas need to see the same uploaded media. Separate container writable layers are not a shared media store.

The project used NFS-Ganesha to provide one shared uploads directory to both WordPress Pods.

**Only the uploads directory was shared.** WordPress themes/plugins/application files were not shared by this NFS mount. MySQL used its own StatefulSet claim.

## 15.2 NFS architecture in the capstone

```text
WordPress Pod 1 ─┐
                ├── wordpress-nfs-pvc (RWX)
WordPress Pod 2 ─┘          |
                            v
                   wordpress-nfs-pv
                            |
                     KIND worker/kubelet
                            |
                    NFSv4.1 / TCP 2049
                            |
                  NFS-Ganesha container
                    172.19.0.10
                      /exports
                            |
                    Docker volume
                      nfs-data
```

Lab configuration:

| Item | Value |
|---|---|
| Image | `ghcr.io/kubernetes-sigs/nfs-ganesha:v6.5` |
| Container name | `nfs-server` |
| NFS version | NFSv4.1 |
| Transport | TCP |
| Port | 2049 |
| KIND Docker network | `kind` |
| NFS container IP observed | `172.19.0.10` |
| Export directory | `/exports` |
| NFSv4 pseudo path | `/wordpress-uploads` |
| Docker volume | `nfs-data` |
| PV name | `wordpress-nfs-pv` |
| PVC name | `wordpress-nfs-pvc` |
| Access mode | RWX |
| PV reclaim policy | Retain |
| StorageClass name | `nfs-manual` |

## 15.3 Why NFS-Ganesha instead of kernel NFS server?

An attempt to start the kernel NFS server inside the KIND worker did not work in the Docker Desktop/WSL2 environment: NFS server thread count remained zero and no server port was listening.

The attempt was cleaned up. NFS-Ganesha, a user-space NFS server, was used in a separate Docker container instead.

A loaded `sunrpc` module alone does not prove an NFS server is running.

## 15.4 Why a Docker named volume for `/exports`?

The initial Windows bind-mounted directory appeared inside the Linux container as a 9P filesystem. Ganesha VFS FSAL could not obtain the required file handle and reported `Operation not supported`.

A Docker named volume provided an ext4 filesystem for `/exports`, resolving the lab's VFS export-root problem. Windows bind mounts were retained for config and logs.

## 15.5 Ganesha export configuration

Important config settings:

| Setting | Purpose |
|---|---|
| `Protocols = 4` | Use NFSv4 |
| `Path = /exports` | Actual directory inside server container |
| `Pseudo = /wordpress-uploads` | NFSv4 client-visible pseudo path |
| `Access_Type = RW` | Read/write export |
| `Squash = Root_Squash` | Map remote root to anonymous identity |
| `SecType = sys` | AUTH_SYS identity-based authentication |
| `Clients = 172.19.0.0/16` | Allowed lab client subnet |
| `FSAL Name = VFS` | Use Ganesha VFS filesystem abstraction |
| `Enable_UDP = false` | Use TCP in this NFSv4 lab |
| `Enable_RQUOTA = false` | Disable unused remote quota service |

## 15.6 Start NFS-Ganesha container

The image expected a runtime directory for its PID file, so it had to be created before starting Ganesha.

```bash
docker volume create nfs-data

docker run -d \
  --name nfs-server \
  --network kind \
  --ip 172.19.0.10 \
  --restart unless-stopped \
  --cap-add DAC_READ_SEARCH \
  -v nfs-data:/exports \
  -v "$(pwd)/nfs-server/config:/etc/ganesha:ro" \
  -v "$(pwd)/nfs-server/logs:/var/log/ganesha" \
  --entrypoint /bin/bash \
  ghcr.io/kubernetes-sigs/nfs-ganesha:v6.5 \
  -c 'mkdir -p /usr/local/var/run/ganesha && exec /usr/local/bin/ganesha.nfsd -F -L /dev/stderr -f /etc/ganesha/ganesha.conf'
```

`--cap-add DAC_READ_SEARCH` was needed by VFS FSAL file-handle operations in this lab. `--privileged` was not needed.

Verify:

```bash
docker ps --filter name=nfs-server
docker logs nfs-server
tail -n 100 nfs-server/logs/ganesha.log
docker inspect nfs-server --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

TCP 2049 listener diagnostic:

```bash
docker exec nfs-server sh -c 'grep -i ":0801" /proc/net/tcp /proc/net/tcp6'
```

`0801` is hexadecimal for TCP port 2049.

## 15.7 NFS PV and PVC

Actual NFS PV:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-nfs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-manual
  mountOptions:
    - proto=tcp
    - nfsvers=4.1
  nfs:
    server: 172.19.0.10
    path: /wordpress-uploads
```

Actual PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-nfs-pvc
  namespace: capstone
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-manual
  resources:
    requests:
      storage: 1Gi
  volumeName: wordpress-nfs-pv
```

Apply and verify:

```bash
kubectl apply -f manifests/nfs-pv.yml
kubectl apply -f manifests/nfs-pvc.yml
kubectl get pv
kubectl get pvc -n capstone
kubectl describe pv wordpress-nfs-pv
kubectl describe pvc wordpress-nfs-pvc -n capstone
```

**Important:** The 1Gi PV capacity is Kubernetes capacity metadata. It is not a hard quota on the NFS export.

## 15.8 WordPress uploads mount

The Deployment mounts the NFS PVC at:

```text
/var/www/html/wp-content/uploads
```

Important fields:

```yaml
volumeMounts:
  - name: wordpress-uploads
    mountPath: /var/www/html/wp-content/uploads

volumes:
  - name: wordpress-uploads
    persistentVolumeClaim:
      claimName: wordpress-nfs-pvc
```

The name in `volumeMounts` must match the Pod volume name. `claimName` must match the PVC name in the same namespace.

## 15.9 Root Squash and UID/GID permissions

Initially `/exports` was root-owned, mode 755. With `Root_Squash`, remote root mapped to anonymous UID/GID `4294967294:4294967294`. WordPress `www-data` was observed as UID/GID 33.

The lab set export ownership to match WordPress:

```bash
docker exec nfs-server chown 33:33 /exports
docker exec nfs-server chmod 755 /exports
docker exec nfs-server stat -c 'UID=%u GID=%g MODE=%a PERMISSIONS=%A' /exports
```

Observed:

```text
UID=33 GID=33 MODE=755
```

This was a lab configuration, not a complete production NFS security design. Production needs deliberate identity mapping, network restrictions, access control and backups.

## 15.10 NFS verification

Manual worker mount:

```bash
docker exec my-cluster-worker modprobe nfs
docker exec my-cluster-worker mkdir -p /mnt/nfs-uploads-test

docker exec my-cluster-worker mount -v \
  -t nfs4 \
  -o vers=4.1,proto=tcp \
  172.19.0.10:/wordpress-uploads \
  /mnt/nfs-uploads-test

docker exec my-cluster-worker findmnt /mnt/nfs-uploads-test
```

Worker writes; NFS server reads:

```bash
docker exec my-cluster-worker sh -c \
  'echo "Created from KIND worker through NFS" > /mnt/nfs-uploads-test/kind-nfs-test.txt'

docker exec nfs-server cat /exports/kind-nfs-test.txt
```

Pod 1 writes; Pod 2 and NFS server read:

```bash
kubectl exec -n capstone <wordpress-pod-1> -- \
  su -s /bin/sh www-data -c \
  'echo "Shared NFS test" > /var/www/html/wp-content/uploads/nfs-pod-test.txt'

kubectl exec -n capstone <wordpress-pod-2> -- \
  cat /var/www/html/wp-content/uploads/nfs-pod-test.txt

docker exec nfs-server cat /exports/nfs-pod-test.txt
```

Actual WordPress media upload was also tested through the WordPress Admin Dashboard. Original/resized media variants were visible in both Pods and in the NFS export.

## 15.11 NFS operational checks

```bash
docker ps --filter name=nfs-server
docker logs --tail 100 nfs-server
tail -n 100 nfs-server/logs/ganesha.log

kubectl get pv
kubectl get pvc -n capstone
kubectl get pods -n capstone -o wide

kubectl describe pod -n capstone <wordpress-pod-name>

kubectl exec -n capstone <wordpress-pod-name> -- \
  df -h /var/www/html/wp-content/uploads
```

**Do not unmount a filesystem while a Pod is actively using the PVC.**

---

# 16. kubectl Command Master Table

This is the quick-reference section for daily revision. Replace placeholders such as `<pod-name>` and `<namespace>` with actual names.

## 16.1 Cluster and context

| Command | What I use it for |
|---|---|
| `kubectl version --client` | Check kubectl client version |
| `kubectl cluster-info` | Check cluster/API endpoints |
| `kubectl get nodes` | List cluster nodes |
| `kubectl get nodes -o wide` | List nodes with extra information |
| `kubectl describe node <node>` | Inspect node capacity, conditions, taints and events |
| `kubectl get namespaces` | List namespaces |
| `kubectl config current-context` | Show active context |
| `kubectl config get-contexts` | List configured contexts |
| `kubectl config use-context <context>` | Switch context |
| `kubectl config view` | Inspect kubeconfig |
| `kubectl config set-context --current --namespace=<ns>` | Set default namespace for current context |

## 16.2 Pods and manifests

| Command | What I use it for |
|---|---|
| `kubectl get pods` | List Pods in current namespace |
| `kubectl get pods -A` | List Pods across all namespaces |
| `kubectl get pods -o wide` | Show Pod IP and node |
| `kubectl get pods --show-labels` | Display Pod labels |
| `kubectl get pods -l app=nginx` | Filter Pods by label |
| `kubectl get pods -w` | Watch Pod changes |
| `kubectl describe pod <pod>` | Inspect status, conditions, events and config |
| `kubectl logs <pod>` | Read container logs |
| `kubectl logs -f <pod>` | Follow logs live |
| `kubectl logs <pod> --previous` | Read previous terminated container's logs |
| `kubectl logs <pod> -c <container>` | Read one container in a multi-container Pod |
| `kubectl exec -it <pod> -- sh` | Open shell inside container |
| `kubectl exec <pod> -- <command>` | Run a command inside container |
| `kubectl run <name> --image=<image>` | Create a Pod imperatively |
| `kubectl get pod <name> -o yaml` | Inspect a Pod's current YAML |
| `kubectl apply -f <file>` | Create/update from manifest |
| `kubectl delete -f <file>` | Delete resources described by manifest |
| `kubectl delete pod <name>` | Delete a Pod |
| `kubectl explain pod` | Learn Pod API fields |
| `kubectl explain deployment.spec` | Learn Deployment spec fields |

## 16.3 YAML generation and validation

| Command | What I use it for |
|---|---|
| `kubectl run test --image=nginx --dry-run=client -o yaml` | Generate starter Pod YAML without creating it |
| `kubectl apply -f file.yml --dry-run=client` | Client-side preview/validation |
| `kubectl apply -f file.yml --dry-run=server` | API Server validation without persisting |
| `kubectl get <resource> <name> -o yaml` | Export current object configuration |

## 16.4 Labels

| Command | What I use it for |
|---|---|
| `kubectl get pods --show-labels` | Display labels |
| `kubectl get pods -l app=nginx` | Select Pods by label |
| `kubectl label pod <pod> key=value` | Add/change label |
| `kubectl label pod <pod> key-` | Remove label |

## 16.5 Deployments and rollout

| Command | What I use it for |
|---|---|
| `kubectl get deployments` | List Deployments |
| `kubectl describe deployment <name>` | Inspect Deployment |
| `kubectl get replicasets` | List ReplicaSets |
| `kubectl scale deployment <name> --replicas=5` | Scale a Deployment |
| `kubectl set image deployment/<name> <container>=<image>` | Update container image |
| `kubectl rollout status deployment/<name>` | Wait for rollout completion |
| `kubectl rollout history deployment/<name>` | View rollout revisions |
| `kubectl rollout undo deployment/<name>` | Undo latest rollout |
| `kubectl rollout undo deployment/<name> --to-revision=1` | Roll back to a revision |
| `kubectl rollout pause deployment/<name>` | Pause rollout |
| `kubectl rollout resume deployment/<name>` | Resume rollout |
| `kubectl rollout restart deployment/<name>` | Trigger a rolling restart |

## 16.6 Services and networking

| Command | What I use it for |
|---|---|
| `kubectl get services` | List Services |
| `kubectl get svc -o wide` | Show Service IP/type/ports |
| `kubectl describe svc <name>` | Inspect Service selector, ports and events |
| `kubectl get endpoints <name>` | View legacy Endpoints |
| `kubectl get endpointslices -l kubernetes.io/service-name=<name>` | View Service EndpointSlices |
| `kubectl expose deployment <name> --port=80` | Quickly create a Service for Deployment |
| `kubectl port-forward svc/<name> 8080:80` | Forward local port to a Service |
| `kubectl port-forward pod/<name> 8080:80` | Forward local port to a Pod |

## 16.7 ConfigMaps and Secrets

| Command | What I use it for |
|---|---|
| `kubectl get configmaps` | List ConfigMaps |
| `kubectl describe configmap <name>` | Inspect ConfigMap metadata and keys |
| `kubectl get configmap <name> -o yaml` | Inspect ConfigMap data |
| `kubectl create configmap <name> --from-literal=k=v` | Create from literal values |
| `kubectl create configmap <name> --from-file=<file>` | Create from a file |
| `kubectl get secrets` | List Secret metadata |
| `kubectl describe secret <name>` | Inspect Secret metadata/keys without printing data |
| `kubectl create secret generic <name> --from-literal=k=v` | Create generic Secret from a literal |

## 16.8 Storage and StatefulSets

| Command | What I use it for |
|---|---|
| `kubectl get pv` | List cluster PersistentVolumes |
| `kubectl get pvc` | List PVCs in current namespace |
| `kubectl get pvc -A` | List PVCs across namespaces |
| `kubectl describe pv <pv>` | Inspect PV |
| `kubectl describe pvc <pvc>` | Inspect PVC binding/events |
| `kubectl get storageclass` | List StorageClasses |
| `kubectl describe storageclass <name>` | Inspect provisioner/reclaim/binding behavior |
| `kubectl get statefulsets` | List StatefulSets |
| `kubectl describe statefulset <name>` | Inspect StatefulSet |
| `kubectl scale statefulset <name> --replicas=5` | Scale StatefulSet |
| `kubectl exec <pod> -- cat <file>` | Verify persistent file contents inside Pod |

## 16.9 Resource usage, Metrics Server and HPA

| Command | What I use it for |
|---|---|
| `kubectl top nodes` | View node CPU/memory usage |
| `kubectl top pods` | View Pod CPU/memory in current namespace |
| `kubectl top pods -A` | View Pod usage across namespaces |
| `kubectl top pods -A --sort-by=cpu` | Sort Pods by CPU |
| `kubectl get apiservice v1beta1.metrics.k8s.io` | Check Metrics API registration/availability |
| `kubectl get hpa` | List HPAs |
| `kubectl get hpa -w` | Watch HPA metrics/replica count |
| `kubectl describe hpa <name>` | Inspect targets, conditions, behavior and events |
| `kubectl autoscale deployment <name> --cpu-percent=50 --min=1 --max=10` | Create a quick CPU HPA (flag warned deprecated in lab) |

## 16.10 RBAC and authentication

| Command | What I use it for |
|---|---|
| `kubectl get serviceaccounts -n <ns>` | List ServiceAccounts |
| `kubectl get roles -A` | List Roles |
| `kubectl get clusterroles` | List ClusterRoles |
| `kubectl get rolebindings -A` | List RoleBindings |
| `kubectl get clusterrolebindings` | List ClusterRoleBindings |
| `kubectl describe clusterrole <name>` | Inspect permission rules |
| `kubectl describe clusterrolebinding <name>` | Inspect granted subject and role |
| `kubectl auth can-i list pods` | Check current identity authorization |
| `kubectl auth can-i delete pods -n capstone --as=<identity>` | Check authorization as an impersonated identity (requires permission to impersonate) |
| `kubectl create token <serviceaccount> -n <ns> --duration=1h` | Request a short-lived ServiceAccount token |
| `kubectl auth whoami` | Display authenticated API identity |

## 16.11 Helm

| Command | What I use it for |
|---|---|
| `helm version` | Check Helm version |
| `helm env` | Display Helm environment paths |
| `helm repo add <alias> <url>` | Add chart repository |
| `helm repo update` | Refresh local chart repository index |
| `helm repo list` | List repositories |
| `helm search repo <term>` | Search charts |
| `helm show values <repo/chart>` | Inspect chart defaults |
| `helm install <release> <repo/chart>` | Install a chart/release |
| `helm list` | List releases |
| `helm status <release>` | Inspect a release |
| `helm get values <release>` | Show user-supplied release values |
| `helm get manifest <release>` | Show rendered Kubernetes manifests |
| `helm upgrade <release> <chart>` | Upgrade release |
| `helm history <release>` | Inspect revision history |
| `helm rollback <release> <revision>` | Roll back to an earlier revision |
| `helm create <chart-dir>` | Create chart scaffold |
| `helm lint <chart-dir>` | Check chart for common issues |
| `helm template <release> <chart-dir>` | Render templates locally without install |
| `helm uninstall <release>` | Uninstall a release |

## 16.12 KIND, Docker and NFS commands from the capstone

| Command | What I use it for |
|---|---|
| `kind get clusters` | List KIND clusters |
| `kind get nodes --name my-cluster` | List nodes/containers in a KIND cluster |
| `kind delete cluster --name my-cluster` | Delete capstone cluster |
| `docker ps -a` | Inspect Docker containers |
| `docker image ls -a` | List local images |
| `docker volume ls` | List Docker volumes |
| `docker network ls` | List Docker networks |
| `docker network inspect kind` | Inspect KIND bridge network |
| `docker logs nfs-server` | Inspect NFS server logs |
| `docker exec nfs-server ...` | Run diagnostics inside NFS container |
| `docker volume inspect nfs-data` | Inspect NFS Docker volume |

**Cleanup safety:** do not run broad `docker volume prune` or network prune on a shared development machine without checking each resource. The environment also contains unrelated OpenClaw resources.

---

# 17. YAML Patterns to Remember

These are reference patterns. Adapt names, namespaces, images and values to the environment before applying.

## 17.1 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.25
      ports:
        - containerPort: 80
```

## 17.2 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.25
```

## 17.3 Service — ClusterIP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```

## 17.4 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  APP_PORT: "8080"
```

## 17.5 Secret using stringData

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  DB_USER: "<user>"
  DB_PASSWORD: "<password>"
```

Use local placeholder values only. Never commit actual credentials.

## 17.6 PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
```

## 17.7 StatefulSet + Headless Service pattern

Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      targetPort: 3306
```

StatefulSet key structure:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-storage
      spec:
        accessModes:
          - ReadWriteOnce
        storageClassName: standard
        resources:
          requests:
            storage: 1Gi
```

## 17.8 Resource requests and limits

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 250m
    memory: 256Mi
```

## 17.9 Liveness, readiness and startup probe examples

HTTP readiness:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 80
  periodSeconds: 5
```

Exec liveness:

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
  periodSeconds: 5
  failureThreshold: 3
```

Startup probe:

```yaml
startupProbe:
  exec:
    command:
      - cat
      - /tmp/started
  periodSeconds: 5
  failureThreshold: 12
```

## 17.10 HPA autoscaling/v2

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: wordpress-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: wordpress
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

---

# 18. Common Troubleshooting Playbook

## 18.1 Pod is Pending

Inspect:

```bash
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe node <node-name>
```

Check:
- Insufficient CPU/memory requests.
- Node taints without matching tolerations.
- PVC is not bound.
- Scheduling constraints/affinity.
- Incorrect units such as `cpu: "100"` instead of `100m`.

## 18.2 Pod is CrashLoopBackOff

Inspect:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
```

Check:
- Application startup command/config.
- Missing Secret/ConfigMap keys.
- Probe failures.
- OOMKilled termination.
- Database/dependency not ready.

## 18.3 Pod is Running but not Ready

Inspect:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get endpoints <service-name>
kubectl get endpointslices -l kubernetes.io/service-name=<service-name>
```

Check readiness probe, Service selector, target port, app startup and logs.

## 18.4 Service has no endpoints

```bash
kubectl describe svc <service-name>
kubectl get pods --show-labels
kubectl get endpoints <service-name>
kubectl get endpointslices -l kubernetes.io/service-name=<service-name>
```

Common issue: Service selector does not match ready Pod labels.

## 18.5 Service DNS is not resolving

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system
```

Run `nslookup <service-name>` from a temporary Pod in the cluster.

Check Service name, namespace, CoreDNS health and full DNS name.

## 18.6 PVC stays Pending

```bash
kubectl get pvc
kubectl describe pvc <pvc-name>
kubectl get pv
kubectl get storageclass
kubectl describe storageclass standard
```

Check:
- Capacity.
- Access mode.
- `storageClassName`.
- Whether a matching static PV exists.
- StorageClass provisioner.
- `WaitForFirstConsumer` behavior.
- Whether a consuming Pod has been created.

## 18.7 HPA shows unknown CPU

```bash
kubectl get pods -n kube-system | grep metrics-server
kubectl get apiservice v1beta1.metrics.k8s.io
kubectl top pods -n <namespace>
kubectl describe hpa <hpa-name> -n <namespace>
```

Check Metrics Server health, Metrics API availability, CPU requests and metric freshness.

## 18.8 NFS uploads fail in WordPress

```bash
kubectl get pv
kubectl get pvc -n capstone
kubectl describe pod -n capstone <wordpress-pod>
kubectl exec -n capstone <wordpress-pod> -- \
  df -h /var/www/html/wp-content/uploads

docker ps --filter name=nfs-server
docker logs nfs-server
docker exec nfs-server stat -c 'UID=%u GID=%g MODE=%a' /exports
```

Check network reachability, NFS server/export path, NFSv4 mount settings, Root Squash, filesystem ownership and WordPress `www-data` UID/GID.

## 18.9 Helm release issue

```bash
helm list
helm status <release-name>
helm history <release-name>
helm get values <release-name>
helm get manifest <release-name>
helm lint <chart-directory>
helm template <release-name> <chart-directory>
```

Check correct release/chart names, values and rendered output, then inspect revision history.

---

# 19. Important Differences — Interview Revision

| Comparison | Difference |
|---|---|
| Docker vs Kubernetes | Docker builds/runs containers; Kubernetes orchestrates containerized workloads across a cluster |
| Control Plane vs Worker Node | Control Plane manages cluster state; Worker Node runs workloads |
| Pod vs Container | Pod is Kubernetes workload unit; container is the application runtime inside it |
| Standalone Pod vs Deployment | Standalone Pod is not automatically recreated by a controller; Deployment maintains desired replicas |
| Deployment vs ReplicaSet | Deployment manages rollouts/ReplicaSets; ReplicaSet maintains Pod replica count |
| Namespace vs Cluster | Namespace is logical organization within a cluster, not another cluster |
| Imperative vs Declarative | Direct commands vs desired state in manifests |
| Client vs Server dry-run | Local preview vs API Server validation without persisting |
| Label vs Selector | Label is attached metadata; selector matches labels |
| containerPort vs Service port | Container port documents the listener; Service port is the network abstraction; targetPort is backend port |
| ClusterIP vs NodePort | Internal stable IP vs node-level port exposure |
| NodePort vs LoadBalancer | Node port exposure vs request for external LB integration |
| Service vs EndpointSlice | Service is stable logical endpoint; EndpointSlice stores backend endpoint data |
| ConfigMap vs Secret | Non-sensitive configuration vs sensitive configuration |
| Secret Base64 vs encryption | Base64 is encoding, not encryption |
| Env variable vs mounted config | Existing container env value does not update automatically; mounted files can update eventually |
| emptyDir vs PV/PVC | Pod-scoped temporary storage vs persistent storage lifecycle |
| PV vs PVC | Provisioned storage vs request for storage |
| Static vs Dynamic provisioning | PV created manually vs provisioner creates PV for a PVC |
| RWO vs RWX | Single-node read-write mount vs multi-node read-write mount, subject to backend support |
| Retain vs Delete | Keep underlying storage vs delete provisioned storage after claim release |
| Deployment vs StatefulSet | Interchangeable replicas vs stable identity and per-Pod storage claims |
| Headless vs normal Service | No ClusterIP; DNS can return individual Pod addresses vs stable virtual Service IP |
| Request vs Limit | Scheduling baseline vs resource ceiling |
| CPU vs Memory limit | CPU throttling vs memory overuse may cause OOM termination |
| Liveness vs Readiness | Restart unhealthy container vs keep unready Pod out of Service traffic |
| Startup vs Liveness | Startup grants initialization time; liveness checks ongoing health |
| Metrics Server vs HPA | Collects metrics vs consumes metrics and changes replica count |
| HPA vs Cluster Autoscaler | Scales Pods vs scales cluster nodes |
| Helm Chart vs Release | Package vs installed instance |
| `helm repo update` vs `helm upgrade` | Refresh chart index vs update installed release |
| `helm template` vs `helm install` | Render YAML locally vs create installed release |
| `helm rollback` vs uninstall | Restore earlier release config vs remove release |


# 20. Kubernetes Interview Questions and Answers

## Fundamentals and architecture

**Q1. What is Kubernetes?**  
Kubernetes is an open-source container orchestration platform that automates deploying, scaling, networking and managing containerized applications.

**Q2. Why do we need Kubernetes when we have Docker?**  
Docker can build and run containers, while Kubernetes manages workloads across a cluster: replica management, self-healing, scheduling, Services, storage and scaling.

**Q3. What are the main parts of Kubernetes architecture?**  
The Control Plane manages cluster state and scheduling. Worker Nodes run application Pods using kubelet, kube-proxy and a container runtime.

**Q4. What is etcd?**  
etcd is the distributed key-value store used to persist Kubernetes cluster state.

**Q5. What does kube-scheduler do?**  
It selects a suitable node for a Pod that has not yet been scheduled, considering resource requests and scheduling constraints.

**Q6. What does kubelet do?**  
Kubelet is the node agent that makes sure Pods assigned to its node are running and reports their status.

**Q7. What does the Controller Manager do?**  
It runs controllers that continuously reconcile actual cluster state with desired state.

**Q8. What is a reconciliation loop?**  
A controller repeatedly observes actual state, compares it to desired state and takes action to reduce the difference.

## Pods, Deployments and Namespaces

**Q9. What is a Pod?**  
A Pod is the smallest deployable Kubernetes unit and contains one or more tightly coupled containers sharing network and potentially storage.

**Q10. What is the difference between a Pod and a container?**  
A container runs the application process. A Pod is the Kubernetes unit that wraps one or more containers and provides shared Pod-level networking and volumes.

**Q11. Why use a Deployment instead of a standalone Pod?**  
A Deployment manages ReplicaSets, maintains replica count, replaces deleted/failed Pods and supports rolling updates and rollbacks.

**Q12. What is a ReplicaSet?**  
A ReplicaSet maintains the desired number of matching Pods. A Deployment manages ReplicaSets.

**Q13. What is a Namespace?**  
A Namespace logically groups and scopes namespaced resources in the same cluster.

**Q14. What happens when you delete a Pod managed by a Deployment?**  
The ReplicaSet detects fewer Pods than desired and creates a replacement Pod. The new Pod normally has a different generated name.

**Q15. What is a rolling update?**  
It gradually replaces existing Pods with new Pods based on the updated Deployment Pod template.

**Q16. How do you roll back a Deployment?**  
Use `kubectl rollout undo deployment/<name>` or specify a revision with `--to-revision`.

## Services and networking

**Q17. Why do we need a Service?**  
Pod IPs can change. A Service provides a stable endpoint and routes traffic to selected eligible Pods.

**Q18. What is ClusterIP?**  
The default Service type; it exposes a stable IP inside the cluster.

**Q19. What is NodePort?**  
It exposes a Service through a port on eligible Kubernetes nodes, normally in the 30000–32767 range.

**Q20. What is LoadBalancer?**  
It requests an external load-balancer implementation from cluster infrastructure. Without a configured provider/controller, external IP can remain Pending.

**Q21. What is the difference between port and targetPort?**  
`port` is the Service port; `targetPort` is the destination port on the selected Pod/container.

**Q22. What is Kubernetes Service DNS?**  
A Service can be accessed using a DNS name such as `<service>.<namespace>.svc.cluster.local`, avoiding hardcoded Pod IPs.

**Q23. What are EndpointSlices?**  
They hold backend endpoint information for a Service and are updated as matching Pods become eligible or unavailable.

## Configuration and storage

**Q24. What is a ConfigMap?**  
A ConfigMap stores non-sensitive configuration separately from container images and can be consumed through environment variables or mounted files.

**Q25. What is a Secret?**  
A Secret stores sensitive values such as credentials and tokens. Access should be restricted; Base64 is not encryption.

**Q26. Do ConfigMap environment variables update automatically?**  
No. Environment variables are set when the container starts; restart/recreate the container/Pod to use updated values.

**Q27. Do mounted ConfigMap files update automatically?**  
Projected ConfigMap volume files update eventually, but the application must reread/reload the files. A `subPath` mount does not receive automatic updates.

**Q28. What is a PV and PVC?**  
PV represents provisioned storage; PVC is a namespaced request for storage.

**Q29. What is the difference between static and dynamic provisioning?**  
Static provisioning requires an administrator to create a PV manually. Dynamic provisioning uses a StorageClass/provisioner to create PVs when PVCs request storage.

**Q30. What is a StorageClass?**  
A StorageClass defines a class of storage and identifies a provisioner and provisioning behavior for dynamic storage requests.

**Q31. What is the difference between RWO and RWX?**  
RWO allows a read-write mount from one node at a time; RWX allows read-write mounts from multiple nodes when supported by the storage backend.

**Q32. What is the difference between Retain and Delete reclaim policies?**  
Retain keeps the underlying storage after a PVC is deleted; Delete removes the PV and provisioned storage through the provisioner/backend.

**Q33. What is a StatefulSet?**  
A StatefulSet manages Pods with stable ordinal identities and can create individual PVCs for each replica using `volumeClaimTemplates`.

**Q34. What is a Headless Service?**  
A Service with `clusterIP: None`. It can provide DNS records for individual Pods, commonly used with StatefulSets.

**Q35. How does StatefulSet Pod DNS work?**  
A Pod DNS name follows `<pod>.<headless-service>.<namespace>.svc.cluster.local`, giving each StatefulSet Pod a stable network identity.

## Resources and probes

**Q36. What is a resource request?**  
A request is the amount of CPU/memory Kubernetes uses when deciding whether a Pod fits on a node.

**Q37. What is a resource limit?**  
A limit caps how much CPU or memory a container can consume. CPU is throttled at the limit; exceeding the memory limit can cause OOM termination.

**Q38. What are Kubernetes QoS classes?**  
Guaranteed, Burstable and BestEffort. They describe resource configuration and are considered during node-pressure eviction behavior.

**Q39. What is OOMKilled?**  
It means the container process was terminated by an out-of-memory mechanism. Check termination reason; exit code 137 alone is not proof.

**Q40. What is a liveness probe?**  
It checks whether a container is healthy. Repeated failure causes Kubernetes to restart the container.

**Q41. What is a readiness probe?**  
It checks whether a Pod should receive Service traffic. Failure marks it unready and removes it from eligible Service endpoints.

**Q42. What is a startup probe?**  
It gives a slow-starting application time to initialize before liveness/readiness probes are activated.

## Metrics, autoscaling and Helm

**Q43. What is Metrics Server?**  
Metrics Server collects node and Pod CPU/memory metrics from kubelets and exposes them through the Metrics API.

**Q44. What is HPA?**  
Horizontal Pod Autoscaler changes the number of workload replicas based on configured metrics such as CPU utilization.

**Q45. Why are CPU requests required for CPU-based HPA?**  
HPA calculates CPU utilization as a percentage of the container's CPU request. Without a request, the percentage can be unknown.

**Q46. What is the difference between HPA and Cluster Autoscaler?**  
HPA scales application Pod replicas. Cluster Autoscaler changes the number of cluster nodes when configured and supported.

**Q47. What is Helm?**  
Helm is a Kubernetes package manager that installs and manages applications using reusable Charts.

**Q48. What is a Helm Chart?**  
A package containing Kubernetes templates, default values and metadata.

**Q49. What is a Helm Release?**  
An installed instance of a chart. The same chart can have multiple releases with different configurations.

**Q50. What does `helm template` do?**  
It renders chart templates into Kubernetes YAML locally without installing a release.

**Q51. What happens when you roll back a Helm release?**  
Helm creates a new revision using the configuration from a selected prior revision; it does not overwrite old revision history.

## Capstone and RBAC

**Q52. Why did you use a StatefulSet for MySQL?**  
MySQL needs stable workload identity and persistent storage association. StatefulSet provides a stable Pod name and a claim associated with that replica.

**Q53. Why use a headless Service for MySQL?**  
It enables stable DNS for the MySQL StatefulSet Pod, such as `mysql-0.mysql.capstone.svc.cluster.local`.

**Q54. Why use NFS for WordPress uploads?**  
Multiple WordPress replicas need access to the same media files. NFS provides a shared RWX uploads directory.

**Q55. Why did you choose NFS-Ganesha?**  
The kernel NFS server could not be started successfully inside the KIND worker environment, so a user-space NFS server was run in a separate Docker container.

**Q56. What is RBAC?**  
Role-Based Access Control is Kubernetes authorization that controls which operations identities can perform on resources.

**Q57. What is a ServiceAccount?**  
A Kubernetes identity for a process/client interacting with the API. It does not receive extra permissions just by being created.

**Q58. What is the difference between Role and ClusterRole?**  
Role defines permissions within a namespace. ClusterRole defines cluster-scoped or reusable permissions.

**Q59. What is the difference between RoleBinding and ClusterRoleBinding?**  
RoleBinding grants permissions within a namespace. ClusterRoleBinding grants ClusterRole permissions across the cluster.

**Q60. What do `kubectl auth can-i` and `kubectl auth whoami` do?**  
`can-i` checks authorization; `whoami` displays the authenticated identity.

---

# 21. How to Explain My Day 60 Project in an Interview

"In my Day 60 Kubernetes capstone, I deployed a WordPress application backed by MySQL on a local KIND cluster.

I used a Deployment for WordPress and a StatefulSet for MySQL. MySQL had a headless Service so the WordPress application could reach the stable StatefulSet DNS name `mysql-0.mysql.capstone.svc.cluster.local`.

I separated non-sensitive database configuration into a ConfigMap and stored credentials in a Kubernetes Secret. The WordPress Deployment also used an init container to wait until MySQL was accepting authenticated queries, along with startup, liveness and readiness probes.

For persistence, MySQL used its own PVC. I configured an NFS-Ganesha server in Docker and exposed a Kubernetes RWX PV/PVC for the WordPress uploads directory. I verified that a file uploaded through one WordPress Pod could be read from the other Pod and from the NFS server.

I installed Metrics Server and configured HPA to target 50% CPU utilization, with a minimum of two and maximum of five WordPress replicas. I generated load and verified scale-out from two to five replicas, then scale-in back to two.

Finally, I installed Headlamp using Helm and created separate viewer and administrator ServiceAccounts with RBAC. I used `kubectl auth can-i` to test permissions and `kubectl auth whoami` to verify token-based identity.

The project helped me connect Kubernetes workload management, networking, storage, resource management, scaling, Helm, authentication and authorization in one application environment."

---

# 22. Final Day 50–60 Revision Checklist

## Day 50 — Architecture and setup

- [ ] Explain why Kubernetes is needed.
- [ ] Draw Control Plane and Worker Node architecture.
- [ ] Explain API Server, etcd, Scheduler and Controller Manager.
- [ ] Explain kubelet, kube-proxy and container runtime.
- [ ] Create, inspect and delete a KIND cluster.
- [ ] Explain kubeconfig and contexts.

## Day 51 — Pods and manifests

- [ ] Create a Pod using YAML and `kubectl run`.
- [ ] Explain imperative vs declarative.
- [ ] Use `get`, `describe`, `logs` and `exec`.
- [ ] Generate YAML using `--dry-run=client -o yaml`.
- [ ] Compare client and server dry-run.
- [ ] Add/remove labels and filter by selectors.
- [ ] Explain why a standalone Pod is not self-healing.

## Day 52 — Namespaces and Deployments

- [ ] Create and inspect namespaces.
- [ ] Create a Deployment with replicas.
- [ ] Explain Deployment → ReplicaSet → Pod.
- [ ] Test self-healing by deleting one Pod.
- [ ] Scale a Deployment up/down.
- [ ] Perform image rolling update.
- [ ] Inspect rollout history and roll back.

## Day 53 — Services

- [ ] Explain ClusterIP, NodePort and LoadBalancer.
- [ ] Explain port, targetPort and nodePort.
- [ ] Test Pod-to-Service connectivity.
- [ ] Explain Service DNS and CoreDNS.
- [ ] Inspect Endpoints and EndpointSlices.
- [ ] Explain why LoadBalancer can remain Pending in local KIND.

## Day 54 — Configuration

- [ ] Create ConfigMap using literals and files.
- [ ] Consume ConfigMap through environment variables.
- [ ] Mount ConfigMap as files.
- [ ] Create and consume a Secret.
- [ ] Explain Base64 is not encryption.
- [ ] Explain environment variables vs mounted file updates.
- [ ] Explain `subPath` update limitation.

## Day 55 — Storage

- [ ] Demonstrate ephemeral `emptyDir` data loss on Pod deletion.
- [ ] Create static PV and PVC.
- [ ] Explain PV vs PVC.
- [ ] Explain access modes: RWO, ROX, RWX.
- [ ] Explain static vs dynamic provisioning.
- [ ] Inspect StorageClass and provisioner.
- [ ] Explain `WaitForFirstConsumer`.
- [ ] Explain Retain vs Delete and PV status lifecycle.

## Day 56 — StatefulSets

- [ ] Explain Deployment vs StatefulSet.
- [ ] Create a Headless Service.
- [ ] Create a StatefulSet with `volumeClaimTemplates`.
- [ ] Explain stable Pod names/ordinal identity.
- [ ] Resolve individual Pod DNS.
- [ ] Prove data survives Pod recreation.
- [ ] Explain ordered scaling and PVC retention on scale-down.

## Day 57 — Resources and probes

- [ ] Explain CPU/memory request and limit.
- [ ] Convert `100m` to 0.1 CPU core.
- [ ] Explain Guaranteed, Burstable and BestEffort QoS.
- [ ] Diagnose OOMKilled vs Pending.
- [ ] Explain startup, liveness and readiness probes.
- [ ] Understand readiness failure removes a Pod from Service endpoints.
- [ ] Inspect events and Pod termination state.

## Day 58 — Metrics and HPA

- [ ] Install and verify Metrics Server.
- [ ] Use `kubectl top nodes` and `kubectl top pods`.
- [ ] Explain CPU request as HPA utilization baseline.
- [ ] Create HPA imperatively and declaratively.
- [ ] Explain `autoscaling/v2`.
- [ ] Calculate basic HPA desired replicas.
- [ ] Generate load and demonstrate scale-out/scale-in.
- [ ] Explain scale-down stabilization and HPA vs Cluster Autoscaler.

## Day 59 — Helm

- [ ] Explain Chart, Release and Repository.
- [ ] Add/update a chart repository.
- [ ] Search and inspect chart values.
- [ ] Install and inspect a release.
- [ ] Customize with `--set` and `-f`.
- [ ] Upgrade and roll back; explain revision history.
- [ ] Create a custom chart.
- [ ] Explain `Chart.yaml`, `values.yaml`, `templates/` and Go template expressions.
- [ ] Use `helm lint` and `helm template`.

## Day 60 — Capstone and RBAC extension

- [ ] Explain WordPress Deployment and MySQL StatefulSet.
- [ ] Explain MySQL headless Service DNS.
- [ ] Explain ConfigMap vs Secret use in the app.
- [ ] Explain init container waiting for MySQL.
- [ ] Explain startup/liveness/readiness checks.
- [ ] Explain separate database PVC and NFS uploads PVC.
- [ ] Explain NFS-Ganesha, RWX, Root Squash and UID/GID permissions.
- [ ] Explain HPA configuration and actual test result.
- [ ] Explain Headlamp Helm installation.
- [ ] Explain ServiceAccounts, ClusterRoles and bindings.
- [ ] Validate permissions using `kubectl auth can-i`.
- [ ] Verify identity with `kubectl auth whoami`.
- [ ] Explain the lab's security limitations and cleanup decisions.

---

# Final Memory Map

```text
KUBERNETES
|
+-- Cluster
|   +-- Control Plane
|   |   +-- API Server
|   |   +-- etcd
|   |   +-- Scheduler
|   |   +-- Controller Manager
|   |
|   +-- Worker Node
|       +-- kubelet
|       +-- kube-proxy
|       +-- Container Runtime
|
+-- Workloads
|   +-- Pod
|   +-- Deployment
|   |   +-- ReplicaSet
|   |       +-- Pods
|   +-- StatefulSet
|       +-- Stable Pod identity
|       +-- PVC per replica
|
+-- Networking
|   +-- Service
|   |   +-- ClusterIP
|   |   +-- NodePort
|   |   +-- LoadBalancer
|   +-- CoreDNS
|   +-- EndpointSlices
|   +-- Headless Service
|
+-- Configuration
|   +-- ConfigMap
|   +-- Secret
|
+-- Storage
|   +-- emptyDir
|   +-- PV
|   +-- PVC
|   +-- StorageClass
|   +-- Static / Dynamic Provisioning
|   +-- Reclaim Policy
|   +-- NFS / RWX
|
+-- Reliability
|   +-- Requests / Limits
|   +-- QoS
|   +-- Startup Probe
|   +-- Liveness Probe
|   +-- Readiness Probe
|   +-- Self-healing / Rollout / Rollback
|
+-- Scaling and Packaging
|   +-- Metrics Server
|   +-- kubectl top
|   +-- HPA
|   +-- Helm Chart / Release / Repository
|
+-- Security and Operations
    +-- ServiceAccounts
    +-- RBAC
    +-- Authentication / Authorization
    +-- Tokens / kubeconfig
    +-- Cleanup and data protection
```

**Final reminder:** Do not memorize commands without understanding the failure they prevent or the operational problem they solve. In an interview, explain the concept, the problem, the Kubernetes resource that addresses it, the command/manifest used, and how you verified the result.

**Days 50–60 Kubernetes learning: Completed.**
