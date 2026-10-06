# Helm - Kubernetes Package Manager

## What is Helm?

Helm is a package manager for Kubernetes. It helps us package Kubernetes YAML files into reusable **Charts**, configure them using **values**, and manage deployments as **Releases**.

Instead of maintaining many Kubernetes YAML files separately, Helm allows us to use templates and values.

```text
Chart + values.yaml + templates
              ↓
             Helm
              ↓
     Kubernetes resources
              ↓
        Kubernetes Cluster
```

## Why do we use Helm?

Without Helm, an application may have many YAML files:

```text
Deployment
Service
ConfigMap
Secret
Ingress
HPA
```

We may need to change the image tag, replica count, service type, ports, environment variables, etc. for different environments.

Helm allows us to keep the common Kubernetes configuration in templates and change only the values.

Example:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.25"
```

The template can use:

```yaml
replicas: {{ .Values.replicaCount }}
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

---

# Important Helm Concepts

## 1. Chart

A **Chart** is a package containing Kubernetes templates and configuration.

Example:

```text
ecommerce/
├── Chart.yaml
├── values.yaml
└── templates/
```

## 2. Release

A **Release** is an installed instance of a Helm Chart.

```bash
helm install ecommerce ./ecommerce
```

Here `ecommerce` is the release name.

The same chart can be installed multiple times with different release names:

```bash
helm install ecommerce-dev ./ecommerce
helm install ecommerce-prod ./ecommerce
```

## 3. Repository

A Helm repository stores Helm charts.

Example:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## 4. values.yaml

`values.yaml` contains configurable values used by the templates.

Example:

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.25"

service:
  type: ClusterIP
  port: 80
```

## 5. Templates

Templates are Kubernetes YAML files containing Helm expressions.

Example:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}
```

Helm replaces the values and generates normal Kubernetes YAML.

---

# Helm Chart Structure

A typical Helm chart looks like:

```text
ecommerce/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── _helpers.tpl
│   └── NOTES.txt
└── .helmignore
```

## Chart.yaml

Contains chart metadata.

```yaml
apiVersion: v2
name: ecommerce
description: Ecommerce application Helm chart
type: application
version: 0.1.0
appVersion: "1.0.0"
```

Important fields:

- `apiVersion` - Chart API version.
- `name` - Chart name.
- `version` - Version of the Helm chart.
- `appVersion` - Version of the application.

---

# Helm Workflow

A common workflow is:

```text
Create Chart
     ↓
Configure values.yaml
     ↓
Create templates
     ↓
helm lint
     ↓
helm template
     ↓
helm install
     ↓
helm list / helm status
     ↓
helm upgrade
     ↓
helm history
     ↓
helm rollback if required
```

---

# Important Helm Commands

## 1. Check Helm version

```bash
helm version
```

Checks whether Helm is installed and shows its version.

## 2. Create a chart

```bash
helm create myapp
```

Creates a starter Helm chart.

## 3. Add a Helm repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

## 4. List repositories

```bash
helm repo list
```

## 5. Update repositories

```bash
helm repo update
```

Downloads the latest chart information from configured repositories.

## 6. Search a repository

```bash
helm search repo nginx
```

## 7. Search Artifact Hub

```bash
helm search hub nginx
```

---

# Validate and Test a Chart

## helm lint

```bash
helm lint ./myapp
```

Checks a chart for common problems.

## helm template

```bash
helm template myapp ./myapp
```

Renders the Helm templates locally without installing anything.

This is one of the most useful commands for debugging templates.

## Dry run

```bash
helm install myapp ./myapp --dry-run --debug
```

Simulates an installation without actually deploying it.

A useful deployment check is:

```bash
helm upgrade --install myapp ./myapp --dry-run --debug
```

---

# Install a Chart

## Basic installation

```bash
helm install myapp ./myapp
```

## Install into a namespace

```bash
helm install myapp ./myapp -n myapp --create-namespace
```

## Install using a values file

```bash
helm install myapp ./myapp -f values-prod.yaml
```

## Override one value

```bash
helm install myapp ./myapp --set replicaCount=5
```

---

# Helm Upgrade

After changing templates or values:

```bash
helm upgrade myapp ./myapp
```

Using a values file:

```bash
helm upgrade myapp ./myapp -f values-prod.yaml
```

Using `--set`:

```bash
helm upgrade myapp ./myapp --set image.tag=2.0
```

---

# helm upgrade --install

This is one of the most useful commands for CI/CD.

```bash
helm upgrade --install ecommerce ./ecommerce \
  --namespace ecommerce \
  --create-namespace
```

Meaning:

```text
Release does not exist → Install
Release already exists → Upgrade
```

---

# Manage Releases

## List releases

```bash
helm list
```

All namespaces:

```bash
helm list -A
```

Specific namespace:

```bash
helm list -n ecommerce
```

## Check release status

```bash
helm status ecommerce
```

## Get values

```bash
helm get values ecommerce
```

Get all computed values:

```bash
helm get values ecommerce -a
```

## Get deployed Kubernetes manifest

```bash
helm get manifest ecommerce
```

## Get all release information

```bash
helm get all ecommerce
```

---

# Helm History and Rollback

Helm keeps release revisions.

Example:

```text
Revision 1 → Working
Revision 2 → Working
Revision 3 → Broken
```

Check history:

```bash
helm history ecommerce
```

Rollback to revision 2:

```bash
helm rollback ecommerce 2
```

For a namespace:

```bash
helm rollback ecommerce 2 -n ecommerce
```

---

# Uninstall a Release

```bash
helm uninstall ecommerce
```

With a namespace:

```bash
helm uninstall ecommerce -n ecommerce
```

This removes the resources managed by that Helm release.

---

# Helm Dependencies

If a chart depends on other charts:

```bash
helm dependency list
```

Update dependencies:

```bash
helm dependency update
```

Build dependencies:

```bash
helm dependency build
```

---

# Package a Chart

```bash
helm package ./ecommerce
```

Example output:

```text
ecommerce-0.1.0.tgz
```

This creates a distributable Helm chart package.

---

# Inspect a Chart

Show chart metadata:

```bash
helm show chart bitnami/nginx
```

Show default values:

```bash
helm show values bitnami/nginx
```

Show chart README:

```bash
helm show readme bitnami/nginx
```

Show all chart information:

```bash
helm show all bitnami/nginx
```

---

# Environment-Specific Values

A professional project can have different values files:

```text
helm/
└── ecommerce/
    ├── Chart.yaml
    ├── values.yaml
    ├── values-dev.yaml
    ├── values-staging.yaml
    ├── values-prod.yaml
    └── templates/
```

Example `values-dev.yaml`:

```yaml
replicaCount: 1

image:
  tag: "dev"

service:
  type: NodePort
```

Example `values-prod.yaml`:

```yaml
replicaCount: 3

image:
  tag: "1.0.0"

service:
  type: ClusterIP
```

Deploy development:

```bash
helm upgrade --install ecommerce ./ecommerce \
  -f values-dev.yaml \
  -n ecommerce \
  --create-namespace
```

Deploy production:

```bash
helm upgrade --install ecommerce ./ecommerce \
  -f values-prod.yaml \
  -n ecommerce \
  --create-namespace
```

---

# Helm + kubectl

Helm manages the application release, while `kubectl` directly interacts with Kubernetes resources.

Check Helm:

```bash
helm list -n ecommerce
helm status ecommerce -n ecommerce
```

Check Kubernetes:

```bash
kubectl get pods -n ecommerce
kubectl get deployments -n ecommerce
kubectl get services -n ecommerce
kubectl get ingress -n ecommerce
```

---

# Helm + Docker + Kubernetes

Typical DevOps flow:

```text
Source Code
    ↓
Build Application
    ↓
Docker Image
    ↓
Container Registry
    ↓
Helm Chart
    ↓
Kubernetes
```

Example:

```bash
docker build -t ashikdv/ecommerce-backend:1.0 .
docker push ashikdv/ecommerce-backend:1.0

helm upgrade --install ecommerce ./helm/ecommerce \
  --set image.repository=ashikdv/ecommerce-backend \
  --set image.tag=1.0
```

---

# Helm + Argo CD

Helm and Argo CD solve different problems and can work together.

- **Helm** - Packages and templates Kubernetes applications.
- **Argo CD** - Provides GitOps continuous delivery and keeps the cluster synchronized with Git.

Typical architecture:

```text
Developer
   ↓
GitHub
   ↓
CI
   ↓
Docker Image → Container Registry
   ↓
Git repository with Helm chart/values
   ↓
Argo CD
   ↓
Kubernetes
```

Argo CD can use a Helm chart as the application source and continuously reconcile the desired state stored in Git.

---

# Helm Debugging Workflow

When a Helm deployment fails, use this order:

## 1. Lint

```bash
helm lint ./ecommerce
```

## 2. Render templates

```bash
helm template ecommerce ./ecommerce
```

## 3. Dry run

```bash
helm upgrade --install ecommerce ./ecommerce --dry-run --debug
```

## 4. Check Helm status

```bash
helm status ecommerce
```

## 5. Inspect the deployed manifest

```bash
helm get manifest ecommerce
```

## 6. Check Kubernetes resources

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

---

# Most Useful Commands to Remember

```bash
helm version
helm create myapp
helm repo add
helm repo list
helm repo update
helm search repo
helm lint ./myapp
helm template myapp ./myapp
helm install myapp ./myapp
helm upgrade myapp ./myapp
helm upgrade --install myapp ./myapp
helm list
helm status myapp
helm get values myapp
helm get manifest myapp
helm history myapp
helm rollback myapp 1
helm uninstall myapp
helm package ./myapp
helm dependency update
```

---

# Quick Cheat Sheet

| Task | Command |
|---|---|
| Check Helm | `helm version` |
| Create chart | `helm create myapp` |
| Add repo | `helm repo add NAME URL` |
| Update repos | `helm repo update` |
| Search repo | `helm search repo nginx` |
| Validate chart | `helm lint ./myapp` |
| Render templates | `helm template myapp ./myapp` |
| Install | `helm install myapp ./myapp` |
| Install/upgrade | `helm upgrade --install myapp ./myapp` |
| List releases | `helm list` |
| Status | `helm status myapp` |
| Get values | `helm get values myapp` |
| Get manifest | `helm get manifest myapp` |
| Upgrade | `helm upgrade myapp ./myapp` |
| History | `helm history myapp` |
| Rollback | `helm rollback myapp 1` |
| Uninstall | `helm uninstall myapp` |
| Package | `helm package ./myapp` |
| Dependencies | `helm dependency update` |

---

# Helm vs kubectl

| Helm | kubectl |
|---|---|
| Kubernetes package manager | Kubernetes CLI |
| Manages releases | Manages Kubernetes resources |
| Uses charts | Uses Kubernetes manifests |
| Provides templating | Does not use Helm templates |
| Supports release history | Focuses on cluster resources |
| Supports rollback | Resource-level operations |

Example:

```bash
helm install ecommerce ./ecommerce
```

versus:

```bash
kubectl apply -f deployment.yaml
```

---

# Helm vs Docker

Docker packages an application into an image.

Helm packages Kubernetes deployment configuration into a chart.

```text
Application
    ↓
Docker
    ↓
Docker Image
    ↓
Container Registry
    ↓
Helm
    ↓
Kubernetes
```

---

# Interview Definition

> **Helm is a Kubernetes package manager that uses Charts, Templates, and Values to package, configure, deploy, upgrade, and rollback Kubernetes applications.**

---

# Recommended Learning Order

## Beginner

Learn:

- Helm
- Chart
- Release
- Repository
- values.yaml
- templates
- `.Values`

## Commands

Practice:

- `helm create`
- `helm install`
- `helm list`
- `helm status`
- `helm uninstall`

## Configuration

Learn:

- `--set`
- `-f values.yaml`
- Environment-specific values files

## Debugging

Learn:

- `helm lint`
- `helm template`
- `--dry-run`
- `--debug`
- `helm get manifest`

## Release Management

Learn:

- `helm upgrade`
- `helm history`
- `helm rollback`

## Advanced

Learn:

- Named templates
- `_helpers.tpl`
- Conditionals
- Loops
- Functions
- Dependencies
- Subcharts
- Hooks
- Chart packaging and versioning

## DevOps

Learn:

- Helm + Docker
- Helm + GitHub Actions
- Helm + Kubernetes
- Helm + Argo CD
- Helm + GitOps
- Environment-specific deployments

---

# Helm Mental Model

Remember:

```text
Chart
  +
Values
  +
Templates
  ↓
Helm renders Kubernetes manifests
  ↓
Release
  ↓
Kubernetes Cluster
```

In simple words:

- **Chart** = Package
- **values.yaml** = Configuration
- **templates/** = Kubernetes YAML templates
- **Release** = Deployed instance
- **Helm** = Tool that manages the chart and release
