This lecture introduces **Deployments** as the Kubernetes object you typically use to manage real production releases. The instructor asks you to momentarily ignore Pods and ReplicaSets and think like someone deploying a web server in production. In a production environment you usually need **many instances** of the same web server for reliability and capacity, and you also need a clean way to **upgrade** those instances when new application builds (new container images) are published to a registry. But upgrades must be done carefully: you usually don’t want to update every instance at once because that can disrupt users, so you prefer **rolling updates**, where instances are upgraded gradually (one after another).

Deployments also cover operational safety and control. If a rollout introduces a problem, you want the ability to **roll back** to a previous working version. And if you plan to make multiple changes—such as upgrading the app image, scaling the number of instances, and adjusting resource allocations—you may not want each change to take effect immediately as soon as you run a command. Instead, you might want to **pause** changes, make updates, and then **resume** so they roll out together. The instructor’s point is that these production-style capabilities (rolling update, rollback, pause/resume) are what Deployments add on top of basic replication.

TODO: if we have multiple upgrade to our system like it was mentioned, is it better to make “micro” roll out, like a step by step upgrade instead of do it everything at once.?

Then the lecture connects Deployments to the Kubernetes object hierarchy you’ve been building so far. A **Pod** runs a single instance of your application container. A **ReplicaSet** (or older ReplicationController) runs and maintains multiple Pods. A **Deployment** sits “higher” and manages ReplicaSets, giving you the extra release-management features described earlier.

Creating a Deployment looks almost identical to creating a ReplicaSet. You write a YAML definition file with the same overall structure: `apiVersion`, `kind`, `metadata`, and `spec`. The key difference is that **`kind` becomes `Deployment`** (instead of `ReplicaSet`). The file still includes `spec.template` (the Pod template), `spec.replicas` (desired count), and `spec.selector` (label selector). Once the file is ready, you create it with `kubectl create -f ...`.

After creation, the instructor explains what Kubernetes builds automatically underneath. When you create a Deployment, it **automatically creates a ReplicaSet**, and that ReplicaSet creates the Pods. That’s why, when you list resources, you’ll see the Deployment, then a ReplicaSet with a related name, and then the Pods created by that ReplicaSet.

TODO: So, if Deployment look like the file of a replicaSet, it also create a replicaSet by default, it’s not necessary to use replicaSet anymore?

Commands shown/mentioned in this lecture are:

```bash
kubectl create -f deployment-definition.yaml
kubectl get deployments
kubectl get replicaset
kubectl get pods
kubectl get all
```

Finally, the instructor notes that at first glance Deployments can look similar to ReplicaSets because both result in Pods being created. The real value of Deployments becomes clear in upcoming lectures, where they will demonstrate **rolling updates, rollback, and pause/resume** behavior.

```yaml
# TODO: Check if this yml files is correct
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: myapp-replicaset
	labels:
		app: myapp
spec:
	selector:
		matchLabels:
			app: myapp
	replicas: 3
	template:
		metadata:
			name: nginx-2
			labels:
				app: myapp
		spec:
			containers:
				- name: nginx-container
				  image: nginx
```
