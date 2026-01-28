# Deployments Update and Rollbacks

This lecture explains how **Deployments** handle **updates** and **rollbacks**, and introduces the idea of **rollouts** and **deployment revisions** so you can track changes over time and safely undo them if something goes wrong.

When you create a Deployment for the first time, Kubernetes triggers a **rollout**. That rollout creates a **ReplicaSet** under the hood, and Kubernetes records that as **Revision 1** of the Deployment. Later, when you upgrade the application—typically by changing the container image version—another rollout happens, Kubernetes creates a *new* ReplicaSet, and the Deployment history records this as **Revision 2**, and so on. This revision history is what enables reliable rollbacks, because Kubernetes can return the Deployment to an earlier revision if the newer rollout causes problems.

To observe rollout progress and history, the instructor highlights two key commands. You can watch the rollout status with:

```bash
kubectl rollout status deployment/<deployment-name>
```

and you can inspect the revision history with:

```bash
kubectl rollout history deployment/<deployment-name>
```

These help you confirm whether an update is complete and which revisions exist.

### Deployment strategies: Recreate vs RollingUpdate

The lecture then describes two strategies Kubernetes can use when updating a Deployment with multiple replicas (for example, 5 replicas of a web application).

With the **Recreate** strategy, Kubernetes first destroys all existing instances (all old Pods) and only then creates the new version. The downside is obvious: there’s a window where the old ones are gone and the new ones are not yet up, so the application is **down** and users can’t access it.

With the **RollingUpdate** strategy, Kubernetes replaces instances **gradually**, bringing up new Pods while taking down old Pods **one by one**, so the application stays available throughout the upgrade. The instructor emphasizes that RollingUpdate is the **default strategy** if you don’t explicitly specify one in your Deployment manifest.

The difference becomes visible if you inspect the Deployment with:

```bash
kubectl describe deployment <deployment-name>
```

In a Recreate update, the events show the old ReplicaSet scaling down to 0 first, then the new ReplicaSet scaling up to the full number. In a RollingUpdate, events show the old ReplicaSet scaling down *incrementally* while the new ReplicaSet scales up at the same time.

### How updates are applied

“Update” can mean several kinds of changes: switching the container image version, changing labels, changing replica count, etc. The recommended workflow when you already have a YAML file is to edit that file and apply it:

```bash
kubectl apply -f deployment-definition.yaml
```

This triggers a new rollout and creates a new revision.

There’s also an imperative alternative for changing the container image specifically:

```bash
kubectlset image deployment/<deployment-name> <container-name>=<image:tag>
```

The instructor warns that this approach can cause your live cluster configuration to drift from what’s written in your Deployment YAML file, so you need to be careful later if you keep using the same file as the “source of truth.”

### What happens under the hood during an upgrade

A Deployment initially creates one ReplicaSet, which creates the Pods needed to reach the desired replica count. When you upgrade the Deployment, Kubernetes creates a **new ReplicaSet** for the new version and begins creating Pods there, while simultaneously reducing Pods in the old ReplicaSet according to the strategy (rolling by default). That’s why listing ReplicaSets after an upgrade typically shows **two** ReplicaSets: an older one (often scaled down to 0 Pods after rollout completion) and the newer one running the active Pods:

```bash
kubectl get rs
```

### Rollback (undo)

If you realize the new version is broken or undesirable after an upgrade, Deployments let you roll back to a previous revision using:

```bash
kubectl rollout undo deployment/<deployment-name>
```

Kubernetes will scale down the new ReplicaSet (destroying those Pods) and scale the previous ReplicaSet back up, effectively returning the application to the older working version. You can confirm what happened by comparing ReplicaSets before and after the rollback: before undo, the “old” ReplicaSet may be at 0 and the “new” at 5; after undo, that is reversed.

### Commands summarized in the lecture

The instructor closes by recapping the commands you’ll use most for deployments, updates, and rollbacks:

```bash
kubectl create -f deployment-definition.yaml
kubectl get deployments
kubectl apply -f deployment-definition.yaml
kubectlset image deployment/<name> <container>=<image:tag>
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
```