### Question 17 — Conceptual — Programmable platform expertise — Kubernetes

What problem do **readiness probes** solve that **liveness probes** do not?

> Readiness probes control whether a pod should receive traffic, while liveness probes determine whether a container should be restarted due to being unhealthy.

### Question 18 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **Kubernetes Pod** snippet that:

* defines both a **readiness** and a **liveness** HTTP probe,
* with different paths and initial delays.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: my-app
      image: image-path
      ports:
        - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
        initialDelaySeconds: 30
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
        initialDelaySeconds: 5
```

### Question 19 — Conceptual — Programmable platform expertise — Kubernetes

Why is **etcd** considered a critical component of a Kubernetes cluster, and what class of failures does it primarily influence?

> etcd is the consensus-backed source of truth for cluster state that the control plane depends on, and failures primarily impact control-plane correctness and availability, including scheduling, leader election, and API reads/writes.

### Question 20 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **HorizontalPodAutoscaler (HPA)** snippet that:

* scales a Deployment based on **CPU utilization**,
* sets a minimum and maximum replica count.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

### Question 21 — Conceptual — Programmable platform expertise — Kubernetes

Why does the **Kubernetes scheduler** rely on **resource requests** rather than **actual usage** when placing pods?

> The scheduler makes placement decisions before pods are running or measurable, so it relies on resource requests as a guaranteed minimum rather than observed usage, which is only available at runtime.

### Question 22 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **NetworkPolicy** that:

* allows ingress traffic **only** from pods with label `app=frontend`,
* to pods with label `app=backend`,
* on TCP port 8080.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

### Question 23 — Conceptual — Programmable platform expertise — Kubernetes

What is the difference between a **Deployment** and a **StatefulSet**, and when would you choose one over the other?

> Deployments manage interchangeable, stateless replicas, while StatefulSets provide stable pod identity, ordering, network identity, and persistent storage for stateful workloads that require those guarantees.

### Question 24 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **PersistentVolumeClaim (PVC)** and a **Pod** that:

* mounts the PVC at `/data`,
* requests 1Gi of storage.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: app
      image: my-image
      volumeMounts:
        - name: data
          mountPath: /data
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: my-pvc
```

### Question 25 — Conceptual — Programmable platform expertise — Kubernetes

What problem does a **PodDisruptionBudget (PDB)** solve, and what kinds of disruptions does it *not* protect against?

> PodDisruptionBudgets limit voluntary disruptions like drains, upgrades, or autoscaling by enforcing minimum availability, but they do not protect against involuntary failures such as crashes, OOMs, or node outages.

### Question 26 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **Ingress** resource that:

* routes HTTP traffic for `example.com`,
* to a Service named `web`,
* on port 80.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
    - host: example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web
                port:
                  number: 80
```

### Question 27 — Conceptual — Programmable platform expertise — Kubernetes

Why can **node autoscaling** (e.g., Cluster Autoscaler or Karpenter) not by itself guarantee that pending pods will be scheduled?

> Node autoscaling can add capacity, but pods may still remain unschedulable due to node selection constraints like labels, taints/tolerations, affinities, or resource requests that no available node can satisfy.

### Question 28 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **Pod** spec that:

* tolerates a taint with key `dedicated`,
* value `gpu`,
* effect `NoSchedule`.

Keep it concise (YAML only).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: app
      image: my-image
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
```

### Question 29 — Conceptual — Programmable platform expertise — Kubernetes

What is the difference between a **taint** and a **toleration**, and how do they interact during scheduling?

> A taint is a node attribute that repels pods, while a toleration is a pod attribute that allows it to be scheduled onto a tainted node if the toleration matches.

### Question 30 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **nodeAffinity** example that:

* requires scheduling onto nodes labeled `node-type=spot`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: node-type
                operator: In
                values:
                  - spot
  containers:
    - name: app
      image: my-image
```

### Question 31 — Conceptual — Programmable platform expertise — Kubernetes

What problem does **pod anti-affinity** solve, and what trade-offs does it introduce?

> Pod anti-affinity prevents certain pods from being co-located to improve availability or fault isolation, but it reduces scheduling flexibility and can easily lead to unschedulable pods if capacity or labels are constrained.

### Question 32 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **Pod anti-affinity** rule that:

* prevents pods with label `app=api` from running on the same node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: api-pod
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - api
          topologyKey: "kubernetes.io/hostname"
  containers:
    - name: app
      image: my-image
```

### Question 33 — Conceptual — Programmable platform expertise — Kubernetes

What is the difference between **requiredDuringSchedulingIgnoredDuringExecution** and **preferredDuringSchedulingIgnoredDuringExecution**, and when would you use each?

> `requiredDuringSchedulingIgnoredDuringExecution` enforces a hard scheduling constraint that must be satisfied, while `preferredDuringSchedulingIgnoredDuringExecution` expresses a soft preference that the scheduler may violate to improve placement flexibility.
