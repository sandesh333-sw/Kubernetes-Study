Perfect bro 🔥 — I’ll make a **full README.md** for you that you can use directly in GitHub. I’ll include:

* Introduction / About
* Stage 1: Basics (Pods, ReplicaSets, Deployments, Services)
* Stage 2: Intermediate (Namespaces, ConfigMaps, Secrets, Volumes)
* Examples / YAMLs / Commands


# **Kubernetes Hands-on Guide (Beginner → Intermediate)**

## **About**

This repository contains **hands-on exercises** to practice Kubernetes from **beginner to intermediate level**.
You’ll learn how to deploy applications, manage Pods, scale them, configure them, expose them with Services, and work with ConfigMaps, Secrets, and Volumes.

This guide is fully practical, with **YAML examples and commands** to run locally using Minikube or any Kubernetes cluster.

---

## **Stage 1: Basics**

### **1. Pod**

Pods are the **smallest deployable unit** in Kubernetes.

**nginx-pod.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

Commands:

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pods
kubectl describe pod my-nginx
kubectl port-forward pod/my-nginx 8080:80
```

Visit: [http://localhost:8080](http://localhost:8080)

---

### **2. ReplicaSet (Self-healing)**

ReplicaSet ensures **a set number of Pods are always running**.

**nginx-rs.yaml**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
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
      - name: nginx-container
        image: nginx:latest
```

Commands:

```bash
kubectl apply -f nginx-rs.yaml
kubectl get rs
kubectl get pods
kubectl delete pod <pod-name>  # Observe self-healing
kubectl get pods
```

---

### **3. Deployment (Rolling Updates)**

Deployment manages ReplicaSets and supports **rolling updates, scaling, and rollback**.

**nginx-deploy.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
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
      - name: nginx-container
        image: nginx:1.21
```

Commands:

```bash
kubectl apply -f nginx-deploy.yaml
kubectl get deployments
kubectl get pods

# Rolling update
kubectl set image deployment/nginx-deploy nginx-container=nginx:latest
kubectl rollout status deployment/nginx-deploy
```

---

### **4. Service (Expose Pods)**

Services provide a **stable network endpoint** to access Pods.

**nginx-svc.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80        # Cluster port
      targetPort: 80  # Pod port
      nodePort: 30007 # Node/host port
```

Commands:

```bash
kubectl apply -f nginx-svc.yaml
kubectl get svc
minikube service nginx-service
```

---

## **Stage 2: Intermediate**

### **5. Namespaces**

Namespaces provide **logical separation of resources**.

```bash
kubectl create namespace dev
kubectl get ns
kubectl run test-pod --image=nginx -n dev
kubectl get pods -n dev
```

---

### **6. ConfigMaps**

ConfigMaps store **non-sensitive configuration data**.

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

Commands:

```bash
kubectl apply -f configmap.yaml
kubectl apply -f pod-using-config.yaml
kubectl logs config-pod
```

---

### **7. Secrets**

Secrets store **sensitive data (base64 encoded)**.

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

Volumes store **persistent or temporary data** inside Pods.

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
    emptyDir: {}
```

---

## **Summary**

* **Pods** → smallest unit
* **ReplicaSets** → self-healing Pods
* **Deployments** → manage ReplicaSets, rolling updates, scaling
* **Services** → stable networking, expose Pods
* **Namespaces** → logical separation
* **ConfigMaps / Secrets** → configuration management
* **Volumes** → persistent or temporary storage

---

