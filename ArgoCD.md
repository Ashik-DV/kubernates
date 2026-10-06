# Argo CD - Kubernetes GitOps Continuous Delivery

## What is Argo CD?

Argo CD is a **GitOps continuous delivery tool for Kubernetes**.

The main idea is simple:

```text
Git Repository = Desired State
        ↓
      Argo CD
        ↓
Kubernetes Cluster = Actual State
```

Argo CD continuously compares what is defined in Git with what is running in Kubernetes and works to keep the cluster synchronized with Git.

---

# Why use Argo CD?

Without GitOps, deployments may be performed manually:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

With Argo CD, the Kubernetes configuration is stored in Git.

```text
Developer
   ↓
GitHub
   ↓
Kubernetes manifests / Helm chart
   ↓
Argo CD
   ↓
Kubernetes Cluster
```

Git becomes the **single source of truth** for the desired application state.

---

# GitOps

GitOps means using Git as the source of truth for infrastructure and application configuration.

The GitOps model is:

```text
Developer changes configuration
             ↓
          Git commit
             ↓
        Argo CD detects it
             ↓
     Compare desired vs actual
             ↓
           Sync
             ↓
      Kubernetes Cluster
```

This gives us:

- Version control
- Audit history
- Easy rollback
- Consistent deployments
- Automated synchronization
- Reduced manual Kubernetes changes

---

# Argo CD Architecture

Important components include:

```text
                    Git Repository
                         ↓
                 ┌───────────────┐
                 │    Argo CD    │
                 └───────┬───────┘
                         ↓
              ┌────────────────────┐
              │ Application        │
              │ Controller         │
              └─────────┬──────────┘
                        ↓
                 Kubernetes API
                        ↓
              Kubernetes Resources
```

## Important Components

### API Server

Provides the API used by the Argo CD CLI, UI, and other clients.

### Repository Server

Connects to Git repositories and generates Kubernetes manifests from sources such as plain YAML, Kustomize, and Helm.

### Application Controller

Continuously compares the desired state from Git with the actual state in Kubernetes.

It detects differences and can synchronize the cluster.

### Redis

Used by Argo CD for caching and improving performance.

### Repo Server

Responsible for fetching application source and generating manifests.

### Argo CD Application

An Argo CD `Application` is a Kubernetes custom resource that connects:

```text
Git Repository
      +
Path / Helm Chart
      +
Target Kubernetes Cluster
      +
Target Namespace
```

---

# Desired State vs Actual State

This is one of the most important Argo CD concepts.

### Desired State

What you have defined in Git.

Example:

```yaml
replicas: 3
```

### Actual State

What is currently running in Kubernetes.

Example:

```text
replicas running = 2
```

Argo CD detects this difference.

```text
Git:        3 replicas
              ↓
           Argo CD
              ↓
Cluster:    2 replicas
```

The application becomes **OutOfSync**.

After synchronization:

```text
Git:        3 replicas
              ↓
           Argo CD
              ↓
Cluster:    3 replicas
```

The application becomes **Synced**.

---

# Argo CD Application

An Application defines where the application configuration comes from and where it should be deployed.

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ecommerce
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/ecommerce.git
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: ecommerce
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Important fields:

- `repoURL` - Git repository.
- `targetRevision` - Branch, tag, or commit.
- `path` - Directory containing manifests/chart.
- `destination.server` - Target Kubernetes cluster.
- `destination.namespace` - Target namespace.
- `syncPolicy` - Controls automatic synchronization.

---

# Sync

**Sync** means applying the desired state from Git to Kubernetes.

Manual sync:

```bash
argocd app sync ecommerce
```

Flow:

```text
Git
 ↓
Argo CD detects change
 ↓
Sync
 ↓
Kubernetes updated
```

---

# Refresh

Refresh means checking the source repository and current cluster state again.

CLI:

```bash
argocd app get ecommerce
argocd app refresh ecommerce
```

Refresh helps Argo CD detect the latest desired state and differences.

---

# Sync Status

An Argo CD application can commonly have statuses such as:

### Synced

Desired state and actual state match.

### OutOfSync

There is a difference between Git and the Kubernetes cluster.

Example:

```text
Git → image: 2.0
Cluster → image: 1.0
```

### Unknown

Argo CD cannot determine the current state correctly.

---

# Health Status

Argo CD also reports application health.

Common states include:

- Healthy
- Progressing
- Degraded
- Suspended
- Missing
- Unknown

Sync status tells you whether desired and actual configuration match.

Health status tells you whether the deployed resources are functioning as expected.

---

# Manual Sync vs Automatic Sync

## Manual Sync

Argo CD detects the change but waits for you to synchronize.

```bash
argocd app sync ecommerce
```

## Automated Sync

Argo CD automatically synchronizes changes from Git.

Example:

```yaml
syncPolicy:
  automated: {}
```

---

# Self Heal

Self-healing allows Argo CD to restore resources when someone manually changes them in the cluster.

Example:

```text
Git says replicas = 3
        ↓
Someone changes cluster to 1
        ↓
Argo CD detects drift
        ↓
Self Heal
        ↓
Replicas restored to 3
```

Configuration:

```yaml
syncPolicy:
  automated:
    selfHeal: true
```

---

# Prune

Pruning removes Kubernetes resources that are no longer defined in Git.

Example:

Initially Git contains:

```text
Deployment
Service
ConfigMap
```

You remove `ConfigMap` from Git.

With pruning enabled:

```text
Git no longer contains ConfigMap
             ↓
          Argo CD
             ↓
ConfigMap removed from Kubernetes
```

Configuration:

```yaml
syncPolicy:
  automated:
    prune: true
```

**Important:** pruning can delete resources, so use it carefully.

---

# Argo CD CLI

Login:

```bash
argocd login <ARGOCD_SERVER>
```

List applications:

```bash
argocd app list
```

Get application details:

```bash
argocd app get ecommerce
```

Create an application:

```bash
argocd app create ecommerce \
  --repo https://github.com/example/ecommerce.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace ecommerce
```

Sync:

```bash
argocd app sync ecommerce
```

Refresh:

```bash
argocd app refresh ecommerce
```

Delete an application:

```bash
argocd app delete ecommerce
```

Show application history:

```bash
argocd app history ecommerce
```

Rollback:

```bash
argocd app rollback ecommerce <ID>
```

---

# Useful Argo CD Commands

```bash
argocd version
argocd login <server>
argocd app list
argocd app get <app-name>
argocd app create <app-name>
argocd app sync <app-name>
argocd app refresh <app-name>
argocd app history <app-name>
argocd app rollback <app-name> <ID>
argocd app delete <app-name>
argocd app manifests <app-name>
argocd app diff <app-name>
argocd app resources <app-name>
argocd app logs <app-name>
```

---

# Installing Argo CD on Kubernetes

Create the namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD:

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Check pods:

```bash
kubectl get pods -n argocd
```

Check services:

```bash
kubectl get svc -n argocd
```

---

# Access Argo CD

For a local cluster such as Minikube, port-forwarding is a simple way to access the server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Then open:

```text
https://localhost:8080
```

Depending on your environment, you can also expose the Argo CD service using NodePort, LoadBalancer, or an Ingress.

---

# Get the Initial Admin Password

The initial admin password can be retrieved from the Kubernetes Secret:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo
```

The default username is:

```text
admin
```

After logging in, it is recommended to change the password and use appropriate authentication for real environments.

---

# Argo CD with Helm

Argo CD can deploy Helm charts.

Example Git structure:

```text
repository/
└── helm/
    └── ecommerce/
        ├── Chart.yaml
        ├── values.yaml
        └── templates/
```

Argo CD points to the chart directory:

```yaml
source:
  repoURL: https://github.com/example/ecommerce.git
  targetRevision: main
  path: helm/ecommerce
```

The important difference is:

```text
Helm  → Packages/templates Kubernetes applications
Argo CD → Continuously delivers and reconciles them using GitOps
```

---

# Argo CD with GitHub Actions

A common DevOps architecture is:

```text
Developer
   ↓
GitHub
   ↓
GitHub Actions CI
   ↓
Build + Test
   ↓
Docker Image
   ↓
Docker Hub / Registry
   ↓
Update image tag in Git
   ↓
Argo CD detects Git change
   ↓
Kubernetes
```

CI is responsible for building and testing.

Argo CD is responsible for continuous delivery and synchronization.

---

# Argo CD vs kubectl

| Argo CD | kubectl |
|---|---|
| GitOps CD tool | Kubernetes CLI |
| Uses Git as desired state | Directly communicates with cluster |
| Continuously monitors drift | Does not continuously reconcile Git |
| Supports automatic sync | Commands are manually executed |
| Application-level view | Resource-level operations |
| Supports Git-based rollback workflows | Can apply previous manifests manually |

Example:

```bash
kubectl apply -f deployment.yaml
```

This directly changes the cluster.

With Argo CD:

```text
Change Git
   ↓
Argo CD detects change
   ↓
Argo CD synchronizes cluster
```

---

# Argo CD vs Helm

| Helm | Argo CD |
|---|---|
| Kubernetes package manager | GitOps continuous delivery tool |
| Creates/releases charts | Deploys and reconciles applications |
| Templates Kubernetes manifests | Monitors desired vs actual state |
| Manages Helm releases | Manages Argo CD Applications |
| Supports Helm rollback | Provides GitOps deployment/reconciliation |

They are commonly used together.

```text
Helm Chart
    ↓
Argo CD
    ↓
Kubernetes
```

---

# Application Lifecycle

A typical lifecycle is:

```text
1. Create application
        ↓
2. Argo CD reads Git
        ↓
3. Generate manifests
        ↓
4. Compare desired and actual state
        ↓
5. Application becomes OutOfSync if different
        ↓
6. Sync application
        ↓
7. Kubernetes resources updated
        ↓
8. Application becomes Synced
        ↓
9. Argo CD continues monitoring
```

---

# Sync Waves

Sync waves can control the order in which resources are deployed.

Example:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "0"
```

Another resource can use:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

Lower waves are processed before higher waves.

This can be useful when one resource must exist before another resource is created.

---

# App of Apps Pattern

The App of Apps pattern uses one Argo CD Application to manage other Argo CD Applications.

```text
Root Application
      ↓
 ┌────┼────┐
 ↓    ↓    ↓
App1 App2 App3
 ↓    ↓    ↓
K8s  K8s  K8s
```

This is useful when managing many applications or environments.

---

# Argo CD Project

An Argo CD Project groups applications and controls what they are allowed to access.

Projects can restrict:

- Allowed Git repositories
- Allowed Kubernetes clusters
- Allowed namespaces
- Allowed resource types

This is useful for security and multi-team environments.

---

# Troubleshooting Workflow

When an Argo CD application is not working:

## 1. Check applications

```bash
argocd app list
```

## 2. Check application details

```bash
argocd app get ecommerce
```

## 3. Check differences

```bash
argocd app diff ecommerce
```

## 4. Check Kubernetes resources

```bash
kubectl get all -n ecommerce
```

## 5. Check pods

```bash
kubectl get pods -n ecommerce
```

## 6. Check pod logs

```bash
kubectl logs <pod-name> -n ecommerce
```

## 7. Describe a failing resource

```bash
kubectl describe pod <pod-name> -n ecommerce
```

## 8. Check Argo CD controller logs

```bash
kubectl logs -n argocd deploy/argocd-application-controller
```

---

# Common Problems

## Application is OutOfSync

Possible causes:

- Git changed but sync has not happened.
- Someone manually changed the cluster.
- Generated manifests differ from the live resources.

Check:

```bash
argocd app diff ecommerce
```

Then sync if appropriate:

```bash
argocd app sync ecommerce
```

## Application is Degraded

The configuration may be synchronized, but the Kubernetes resources are unhealthy.

Check:

```bash
kubectl get pods -n ecommerce
kubectl describe pod <pod-name> -n ecommerce
kubectl logs <pod-name> -n ecommerce
```

## Sync fails

Check:

```bash
argocd app get ecommerce
argocd app diff ecommerce
```

Then inspect the Kubernetes event/resource error.

---

# Argo CD Mental Model

Remember these five ideas:

```text
Git
 ↓
Desired State
 ↓
Argo CD
 ↓
Compare + Sync
 ↓
Kubernetes
```

And:

```text
Sync       = Apply desired state
Refresh    = Check for changes
OutOfSync  = Desired != Actual
Synced     = Desired == Actual
Prune      = Remove resources deleted from Git
Self Heal  = Restore manually changed resources
```

---

# Most Important Commands

```bash
# Check version
argocd version

# Login
argocd login <server>

# List applications
argocd app list

# Application details
argocd app get ecommerce

# Refresh
argocd app refresh ecommerce

# Compare differences
argocd app diff ecommerce

# Sync
argocd app sync ecommerce

# History
argocd app history ecommerce

# Rollback
argocd app rollback ecommerce <ID>

# Resources
argocd app resources ecommerce

# Delete application
argocd app delete ecommerce
```

---

# Interview Definition

> **Argo CD is a GitOps continuous delivery tool for Kubernetes that uses Git as the source of truth and continuously compares the desired state in Git with the actual state in the Kubernetes cluster.**

---

# Argo CD Learning Order

### Beginner

- What is Argo CD?
- GitOps
- Desired vs actual state
- Application
- Sync
- Refresh
- Synced vs OutOfSync
- Health status

### Commands

- `argocd login`
- `argocd app list`
- `argocd app get`
- `argocd app sync`
- `argocd app refresh`
- `argocd app diff`
- `argocd app history`
- `argocd app rollback`

### Important Features

- Automated sync
- Self Heal
- Prune
- Projects
- Sync waves
- App of Apps

### DevOps Integration

- Argo CD + Kubernetes
- Argo CD + Helm
- Argo CD + GitHub Actions
- Argo CD + Docker
- Argo CD + GitOps

---

# Final Mental Model

```text
                 GitHub
                   │
                   │ Desired State
                   ↓
              ┌─────────┐
              │ Argo CD │
              └────┬────┘
                   │
          Compare Desired/Actual
                   │
             Sync / Self Heal
                   ↓
          Kubernetes Cluster
                   │
                   ↓
             Application
```

**Git is the source of truth. Argo CD continuously watches and reconciles. Kubernetes runs the application.**
