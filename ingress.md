# Kubernetes Ingress and Ingress Controller

## 1. What is Ingress?

**Ingress** is a Kubernetes resource used to define rules for routing **external HTTP/HTTPS traffic** to different Kubernetes Services.

For example, suppose we have:

```text
/api/home       → home-service
/api/dashboard  → dashboard-service
/api/products   → product-service
```

Ingress allows us to define these routing rules in one place.

### Basic traffic flow

```text
User / Internet
       |
       v
Ingress Controller
       |
       v
     Ingress
   Routing Rules
    /    |    \
   /     |     \
  v      v      v
Home   Dashboard Products
Service  Service  Service
  |        |        |
 Pods     Pods     Pods
```

---

# 2. Why do we need Ingress?

Without Ingress, we may need to expose multiple applications separately using Services such as `NodePort` or `LoadBalancer`.

For example:

```text
Frontend → NodePort
Backend  → NodePort
Admin    → NodePort
```

With Ingress, we can have a single HTTP/HTTPS entry point and route requests based on the URL.

```text
                    Internet
                       |
                       v
                Ingress Controller
                       |
             +---------+---------+
             |         |         |
             v         v         v
         /api/home /api/admin /api/products
             |         |         |
             v         v         v
         Home Svc   Admin Svc  Product Svc
```

---

# 3. What is an Ingress Controller?

An **Ingress Controller** is the actual software component that receives HTTP/HTTPS traffic and implements the routing rules defined by Ingress resources.

### Important distinction

```text
Ingress
   |
   | Defines the rules
   v
"/api/home"      → home-service
"/api/dashboard" → dashboard-service
```

The **Ingress Controller** reads these rules from Kubernetes and actually handles the incoming traffic.

```text
Internet
   |
   v
Ingress Controller
   |
   +---- /api/home ------> home-service
   |
   +---- /api/dashboard -> dashboard-service
```

### Common Ingress Controllers

Examples include:

* NGINX Ingress Controller
* Traefik
* HAProxy
* Kong
* Cloud-provider-specific controllers

---

# 4. Ingress vs Ingress Controller

| Ingress                        | Ingress Controller                    |
| ------------------------------ | ------------------------------------- |
| Kubernetes resource            | Software/component                    |
| Defines routing rules          | Implements routing rules              |
| Created using YAML             | Installed/running in the cluster      |
| Says where traffic should go   | Actually processes and routes traffic |
| Does not itself handle traffic | Handles incoming HTTP/HTTPS traffic   |

Simple way to remember:

> **Ingress = routing rules**

> **Ingress Controller = executes those routing rules**

---

# 5. Example

Suppose we have three Services:

```text
home-service
dashboard-service
product-service
```

We want:

```text
/api/home       → home-service
/api/dashboard  → dashboard-service
/api/products   → product-service
```

We can create an `ingress.yaml` file.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: my-ingress

spec:
  rules:
    - http:

        paths:

          - path: /api/home
            pathType: Prefix
            backend:
              service:
                name: home-service
                port:
                  number: 80

          - path: /api/dashboard
            pathType: Prefix
            backend:
              service:
                name: dashboard-service
                port:
                  number: 80

          - path: /api/products
            pathType: Prefix
            backend:
              service:
                name: product-service
                port:
                  number: 80
```

---

# 6. Understanding the Example

This section:

```yaml
- path: /api/home
  pathType: Prefix
  backend:
    service:
      name: home-service
      port:
        number: 80
```

means:

```text
/api/home
     |
     v
home-service
     |
     v
Home Pods
```

Similarly:

```text
/api/dashboard
       |
       v
dashboard-service
       |
       v
Dashboard Pods
```

And:

```text
/api/products
       |
       v
product-service
       |
       v
Product Pods
```

---

# 7. What is `pathType: Prefix`?

`Prefix` means that the path and paths underneath it will match.

For example:

```yaml
path: /api/products
pathType: Prefix
```

can match:

```text
/api/products
/api/products/1
/api/products/10
/api/products/search
```

So:

```text
/api/products/*
       |
       v
product-service
```

---

# 8. Applying the Ingress

After creating `ingress.yaml`, apply it using:

```bash
kubectl apply -f ingress.yaml
```

Check the Ingress:

```bash
kubectl get ingress
```

For more details:

```bash
kubectl describe ingress my-ingress
```

---

# 9. Ingress Controller in Minikube

If you are using **Minikube**, you can enable the built-in Ingress addon:

```bash
minikube addons enable ingress
```

Then check whether the Ingress Controller is running:

```bash
kubectl get pods -n ingress-nginx
```

You should see the NGINX Ingress Controller running.

For example:

```text
NAME                                       READY   STATUS
ingress-nginx-controller-xxxxx             1/1     Running
```

Then apply your Ingress:

```bash
kubectl apply -f ingress.yaml
```

---

# 10. Complete Request Flow

Suppose the user requests:

```text
http://myapp.com/api/products
```

The request flows approximately like this:

```text
                     User
                      |
                      | HTTP Request
                      v
              Ingress Controller
                      |
                      | checks routing rules
                      v
                   Ingress
                      |
                      | /api/products
                      v
               product-service
                      |
                      v
                 Product Pod
                      |
                      v
               Application
```

The important point is that the **Ingress Controller watches Ingress resources and uses their rules to route traffic**.

---

# 11. Ingress and Kubernetes Service

Ingress does **not normally route directly to Pods**.

The usual flow is:

```text
Ingress
   |
   v
Service
   |
   v
Pods
```

For example:

```text
/api/products
      |
      v
product-service
      |
   +--+--+
   |  |  |
   v  v  v
 Pod Pod Pod
```

The Service then provides networking/load balancing to the matching Pods.

---

# 12. Ingress and ASP.NET Core

If you are using ASP.NET Core, there are actually **two levels of routing**.

### Kubernetes routing

```text
/api/products
       |
       v
Ingress
       |
       v
backend-service
       |
       v
Backend Pod
```

### ASP.NET Core routing

Once the request reaches your application:

```text
Backend Pod
     |
     v
ASP.NET Core
     |
     v
Controller
```

For example:

```csharp
[Route("api/products")]
public class ProductController : ControllerBase
{
    [HttpGet]
    public IActionResult GetProducts()
    {
        return Ok();
    }
}
```

So the complete flow can be:

```text
Internet
   |
   v
Ingress Controller
   |
   | /api/products
   v
Backend Service
   |
   v
Backend Pod
   |
   v
ASP.NET Core
   |
   v
ProductController
```

This is an important distinction:

> **Ingress routes traffic to the Kubernetes Service.**

> **ASP.NET Core routing routes the request to the Controller/Action inside your application.**

---

# 13. Path-Based Routing

Ingress can route based on URL paths.

Example:

```text
myapp.com/api/users
        |
        v
user-service

myapp.com/api/products
        |
        v
product-service

myapp.com/api/orders
        |
        v
order-service
```

Example configuration:

```yaml
rules:
  - http:
      paths:

        - path: /api/users
          pathType: Prefix
          backend:
            service:
              name: user-service
              port:
                number: 80

        - path: /api/products
          pathType: Prefix
          backend:
            service:
              name: product-service
              port:
                number: 80
```

---

# 14. Host-Based Routing

Ingress can also route based on the domain/host.

For example:

```text
api.myapp.com
      |
      v
backend-service

admin.myapp.com
      |
      v
admin-service

shop.myapp.com
      |
      v
frontend-service
```

Example:

```yaml
spec:
  rules:

    - host: api.myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 80

    - host: admin.myapp.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: admin-service
                port:
                  number: 80
```

---

# 15. Ingress + Service + Deployment

A typical Kubernetes application can look like this:

```text
                 Internet
                    |
                    v
          Ingress Controller
                    |
                    v
                 Ingress
             /           \
            /             \
           v               v
    frontend-service   backend-service
           |               |
           v               v
      Deployment       Deployment
           |               |
       +---+---+       +---+---+
       |   |   |       |   |   |
       v   v   v       v   v   v
      Pod Pod Pod     Pod Pod Pod
```

The responsibilities are different:

### Deployment

Manages the desired number of Pods.

```text
Deployment
    |
 ReplicaSet
    |
  Pods
```

### Service

Provides stable networking and distributes traffic to matching Pods.

```text
Service
   |
   +--- Pod
   +--- Pod
   +--- Pod
```

### Ingress

Defines external HTTP/HTTPS routing.

```text
Ingress
   |
   +--- /api/users → user-service
   |
   +--- /api/products → product-service
```

### Ingress Controller

Actually processes the external HTTP/HTTPS traffic according to those Ingress rules.

---

# 16. Simple Mental Model

Remember this:

```text
                 INTERNET
                    |
                    v
            INGRESS CONTROLLER
                    |
                    v
                 INGRESS
              Routing Rules
             /             \
            v               v
       Service A         Service B
          |                 |
          v                 v
        Pods              Pods
```

And remember:

```text
Deployment
    ↓
Manages Pods

Service
    ↓
Provides stable access + distributes traffic to Pods

Ingress
    ↓
Defines external HTTP/HTTPS routing rules

Ingress Controller
    ↓
Actually implements those routing rules
```

## Final Definition

> **Ingress is a Kubernetes resource that defines rules for routing external HTTP/HTTPS traffic to Services.**

> **Ingress Controller is the software that watches those Ingress rules and actually handles and routes the incoming traffic.**
