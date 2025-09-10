

# **Kubernetes Hands-on Guide (Beginner → Intermediate) — Detailed Version**

## **About**

This repository is a **hands-on Kubernetes guide** for beginners to intermediate users.
It covers everything from **Pods → ReplicaSets → Deployments → Services → Namespaces → ConfigMaps → Secrets → Volumes**, with **detailed explanations, examples, and commands**.

You’ll understand not just *how to write YAML*, but **what happens when you deploy it**, and what each field means.

---

## **Kubernetes Resource Structure (4 Main Fields)**

Every Kubernetes YAML typically has **4 main sections**:

1. **apiVersion** – specifies the API version of Kubernetes you are using.

   * Example: `v1` for Pods, `apps/v1` for Deployments or ReplicaSets.
   * Determines which features and fields are available.

2. **kind** – what type of resource you are creating.

   * Example: `Pod`, `ReplicaSet`, `Deployment`, `Service`.

3. **metadata** – information to identify the resource.

   * `name`: unique name of the resource
   * `labels`: key-value pairs to organize and select resources
   * `namespace`: which namespace this resource belongs to (default is `default`)

4. **spec** – the **specification** or “blueprint” of what Kubernetes should create.

   * For **Pod** → defines containers, ports, volumes, env variables
   * For **ReplicaSet/Deployment** → defines `replicas`, `selector`, and `template` (which defines the Pod spec)
   * For **Service** → defines type (ClusterIP/NodePort), selector, and ports

> Think of `spec` as the **“recipe”**: it tells Kubernetes exactly what you want, how many, and how it should behave.

---

## **Stage 1: Basics**

### **1. Pod**

Pods are the **smallest deployable unit** in Kubernetes. They can contain **one or more containers**, share **network and storage**, and run on a Node.

**nginx-pod.yaml**

```yaml
apiVersion: v1            # API version for Pod
kind: Pod                 # Type of resource
metadata:
  name: my-nginx          # Unique name for this Pod
  labels:                 # Labels used for selecting / grouping
    app: nginx
spec:                     # Blueprint of the Pod
  containers:             # List of containers
  - name: nginx-container # Name of the container
    image: nginx:latest   # Docker image to run
    ports:                # Ports exposed by container
    - containerPort: 80
```

**What happens in `spec`**:

* Kubernetes will **schedule the Pod** onto a Node.
* It creates a **container** using `nginx:latest`.
* Exposes **port 80** inside the Pod (other Pods can talk to it via this port).

Commands:

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod my-nginx
kubectl port-forward pod/my-nginx 8080:80
```

Visit [http://localhost:8080](http://localhost:8080) → You see nginx running.

---

### **2. ReplicaSet (Self-healing)**

ReplicaSet ensures that **a fixed number of Pods** are running at all times.
If a Pod dies or is deleted, ReplicaSet automatically creates a replacement.

**nginx-rs.yaml**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3               # Number of Pods desired
  selector:
    matchLabels:
      app: nginx            # Matches Pods with label app=nginx
  template:                 # Pod template (blueprint)
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
```

**What happens in `spec`**:

* `replicas: 3` → Kubernetes ensures **exactly 3 Pods** are running.
* `selector` → tells the ReplicaSet which Pods it manages (must match `template.metadata.labels`).
* `template` → defines **what each Pod should look like** (like your Pod YAML).

Commands:

```bash
kubectl apply -f nginx-rs.yaml
kubectl get rs
kubectl get pods
kubectl delete pod <pod-name>  # ReplicaSet recreates the Pod
kubectl get pods
```

---

### **3. Deployment (Rolling Updates)**

Deployment is a **higher-level controller** that manages ReplicaSets.
It provides **rolling updates, rollback, and scaling**.

**nginx-deploy.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 3                    # Desired number of Pods
  selector:
    matchLabels:
      app: nginx
  template:                       # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.21        # Older version
```

**Rolling Update Example**:

```bash
kubectl apply -f nginx-deploy.yaml
kubectl get deployments
kubectl set image deployment/nginx-deploy nginx-container=nginx:latest
kubectl rollout status deployment/nginx-deploy
```

**What happens in `spec`**:

* Deployment creates a **ReplicaSet** with `replicas=3`.
* Each Pod is created from the `template`.
* When we update the image, Kubernetes does a **rolling update**: gradually replaces old Pods with new ones without downtime.

---

### **4. Service (Expose Pods)**

Services provide a **stable endpoint** to access Pods.
Without a Service, Pod IPs change when they restart → unstable.

**nginx-svc.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort                # Exposes service on cluster node port
  selector:
    app: nginx                  # Selects Pods with label app=nginx
  ports:
    - port: 80                  # Service port inside cluster
      targetPort: 80            # Pod port
      nodePort: 30007           # Port accessible on Node (host)
```

Commands:

```bash
kubectl apply -f nginx-svc.yaml
kubectl get svc
minikube service nginx-service
```

**What happens in `spec`**:

* `selector` → decides which Pods this service routes traffic to
* `ports` → maps NodePort → ServicePort → PodPort
* Kubernetes sets up a **stable IP & port**, even if Pods die or restart

---

## **Stage 2: Intermediate**

### **5. Namespaces**

Logical grouping of resources.

```bash
kubectl create namespace dev
kubectl get ns
kubectl run test-pod --image=nginx -n dev
kubectl get pods -n dev
```

### **6. ConfigMaps**

Store **non-sensitive configuration**.

**configmap.yaml**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: "development"
  APP_PORT: "8080"
```

**Pod using ConfigMap**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env"]
    envFrom:
    - configMapRef:
        name: app-config
```

---

### **7. Secrets**

Store **sensitive data (base64 encoded)**.

**db-secret.yaml**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

**Pod using Secret**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo $DB_USER && echo $DB_PASS"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

---

### **8. Volumes**

Store **persistent or temporary data** inside Pods.

**volume-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo hello > /data/test.txt && sleep 3600"]
    volumeMounts:
    - mountPath: /data
      name: my-volume
  volumes:
  - name: my-volume
    emptyDir: {}   # temporary storage, deleted when Pod dies
```

---

## **Summary**

* **Pods** → smallest unit, runs containers
* **ReplicaSets** → ensures number of Pods, self-healing
* **Deployments** → manage ReplicaSets, rolling updates, scaling
* **Services** → stable network access to Pods
* **Namespaces** → logical separation of environments
* **ConfigMaps / Secrets** → configuration and sensitive data management
* **Volumes** → persistent or temporary storage

---


