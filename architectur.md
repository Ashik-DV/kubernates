Architecture 

CONTROL PLANE
────────────────────────
API Server     → Receives
etcd           → Stores
Scheduler      → Places
Controller     → Maintains


WORKER NODE
────────────────────────
kubelet        → Manages the pods
Runtime        → Runs
kube-proxy     → Networks
Pod            → Contains containers




              Kubernetes Cluster
                     │
          ┌──────────┴──────────┐
          │                     │
    Control Plane           Worker Node
      (Brain)                 (Runs Apps)
          │                     │
    ┌─────┼─────┐          ┌────┼─────┐
    │     │     │          │    │     │
 API   etcd  Scheduler   kubelet  Runtime
Server        Controller             │
              Manager                ▼
                                  Pod
                                    │
                                Container