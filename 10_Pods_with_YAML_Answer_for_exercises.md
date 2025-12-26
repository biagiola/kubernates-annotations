# Pods with YAML. Answer for exercises

This lecture is essentially the **lab solution walkthrough for Pods** on a browser-based learning platform (you mentioned **KodeKloud**, which the instructor uses as a Kubernetes playground). The goal is to get comfortable with Pod inspection, troubleshooting, and basic lifecycle operations, using mostly `kubectl` commands. The instructor answers a series of questions (around ~15) and demonstrates the exact commands to solve each one. Below is the same flow, keeping the intent of each question and pairing it with the CLI used.

---

## Pod lab walkthrough (questions + commands)

### 1) **How many pods exist on the system?**

Use `kubectl get pods`. The output may mention “default namespace” — the instructor says ignore that for now because namespaces are covered later.

```bash
kubectl get pods
```

Result in the lab: **0 pods** (“No resources found”).

---

### 2) **Create a new pod using the nginx image**

The instructor uses the imperative approach with `kubectl run`. Pod name can be anything if not specified, but the **image must be `nginx`**.

```bash
kubectl run nginx --image=nginx

```

(They also mention using help if you forget syntax.)

```bash
kubectl run --help
```

---

### 3) **How many pods are created now?**

Check again with:

```bash
kubectl get pods
```

In the lab, there are additional pods already present (not only the one you created), totaling **4 pods** at that point.

---

### 4) **What image is used to create the new pods?**

Inspect one of the pods in detail with `describe` and look at the **Image** field under Containers.

```bash
kubectl describe pod <pod-name>
```

In the lab example, the image shown is **busybox** (for one of the “new pods” that existed besides nginx).

---

### 5) **Which nodes are these pods placed on?**

Two ways are mentioned:

**A.** Describe each pod and check the “Node” field:

```bash
kubectl describe pod <pod-name>
```

**B.** Easier: use wide output to see node placement for all pods at once:

```bash
kubectl get pods -o wide
```

In the lab, the pods are scheduled on the **control-plane** node.

---

### 6) **How many containers are part of the pod `webapp`?**

Check the READY column in `kubectl get pods` (e.g., `1/2`) or confirm in `describe`.

```bash
kubectl get pods
# or
kubectl describe pod webapp
```

In the lab, `webapp` has **2 containers**.

---

### 7) **What images are used in the `webapp` pod?**

Look at the two containers listed in `describe` and read their image names.

```bash
kubectl describe pod webapp
```

In the lab, the images are **nginx** and **agentx**.

---

### 8) **What is the state of the container `agentx` in the `webapp` pod?**

In `describe`, under the container details you’ll see “State”.

```bash
kubectl describe pod webapp
```

In the lab, `agentx` is in a **Waiting** state (error-type waiting).

---

### 9) **Why is the `agentx` container in an error/waiting state?**

The instructor points to two places:

- The container “Reason” (e.g., `ErrImagePull` / `ImagePullBackOff`)
- The **Events** section at the bottom (failed to pull the image)

Command:

```bash
kubectl describe pod webapp
```

Explanation in the lab: the image **agentx doesn’t exist on Docker Hub**, so Kubernetes can’t pull it.

---

### 10) **What does the READY column in `kubectl get pods` indicate?**

The instructor explains it using examples like `1/1` and `1/2`.

Command shown where READY appears:

```bash
kubectl get pods
```

Meaning: **ready containers / total containers in the pod** (e.g., `1/2` = 1 ready out of 2).

---

### 11) **Delete the `webapp` pod**

```bash
kubectl delete pod webapp
```

---

### 12) **Create a pod named `redis` (or “redis” task) with image `redis123`**

The instructor says the image name is intentionally wrong and recommends creating it via a **YAML definition**, generated from an imperative command using **dry-run**.

First, generate YAML (note: the lecture points out the newer form is `--dry-run=client`):

```bash
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-pod.yaml
```

Then create the pod from the file:

```bash
kubectl create -f redis-pod.yaml
# (apply would also work in many cases)
```

Verify:

```bash
kubectl get pods
```

In the lab, it shows an image pull error state (as expected) because `redis123` is invalid.

---

### 13) **Fix the pod by changing the image to `redis` so it becomes Running**

Two approaches are mentioned:

**A. Edit the live pod**

```bash
kubectl edit pod redis
```

**B. Edit the YAML file and re-apply**

You edit `redis-pod.yaml` and change:

```yaml
image:redis123
```

to:

```yaml
image:redis
```

Then apply:

```bash
kubectl apply -f redis-pod.yaml
```

Finally confirm it’s running:

```bash
kubectl get pods
```

The instructor notes you may see an “error/warning” message during apply that will make more sense later when the course covers **imperative vs declarative** workflows, but for now the goal is simply to verify the pod reaches **Running**.

---

## Key takeaways from the lab

Pods are inspected and troubleshot primarily with `kubectl get pods`, `kubectl describe pod ...`, and `kubectl get pods -o wide`. Image pull failures are diagnosed via the **Events** section in `describe`. The lab also introduces a practical workflow: using `kubectl run ... --dry-run=client -o yaml` to quickly generate a starter YAML, saving it to a file, and then using `kubectl create -f` / `kubectl apply -f` to manage the pod declaratively.

If you paste the exact question list from KodeKloud (sometimes wording differs slightly), I can mirror the numbering 1:1 exactly—but the commands above match what the instructor executed in this walkthrough.

ChatGPT can make mistakes. Check important info.