# [Logging and Monitoring the Kubernetes Components:](https://kubernetes.io/docs/tasks/debug/)

## Monitoring The k8s Cluster

- `Metric Server` is available on each node to gather logs.
- Metric server can be enabled using `minikube addons enable metrics-server` on minikube only
- for Others to deploy **metric server**
  - `git clone https://github.com/kubernetes-incubator/metrics-server.git`
  - `kubectl create -f deploy/1.8+/`
  - `kubectl top node` to view resource consumption of cluster.

```
controlplane kubernetes-metrics-server on  master ➜  kubectl top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
controlplane   258m         0%     1084Mi          0%
node01         118m         0%     268Mi           0%
```

- `kubectl top pod` to view resources consumption of pods.

```
controlplane kubernetes-metrics-server on  master ➜  kubectl top pod
NAME       CPU(cores)   MEMORY(bytes)
elephant   15m          32Mi
lion       1m           18Mi
rabbit     102m         14Mi
```

## Logging in K8s

- to view logs of a pod
  - `kubectl logs <pod-name>`
- to view logs of a pod in a specific container
  - `kubectl logs <pod-name> -c <container-name>`
- to view logs of a pod in a specific container and follow the logs
- `kubectl describe pod` to find the container name

```
Name:         my-pod
Namespace:    default
Priority:     0
Node:         node1/10.0.0.1
Start Time:   Tue, 01 Mar 2023 10:00:00 +0000
Labels:       app=my-app
Annotations:  <none>
Status:       Running
IP:           10.0.0.2
IPs:
  IP:  10.0.0.2
Containers:
  **my-container:**
    Container ID:   docker://abcdefg1234567890
    Image:          my-image:latest
    Image ID:       docker-pullable://my-image@sha256:abcdefg1234567890
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 01 Mar 2023 10:00:01 +0000

```

# Application Lifecycle Management in K8s

## Rollout and Versioning

- Users expect applications to be available all the time, and developers are expected to deploy new versions of them several times a day. In Kubernetes this is done with rolling updates
- A **rollinrg update** allows a Deployment update to take place with **zero downtime**
- It does this by incrementally replacing the current Pods with new ones.
- Kubernetes waits for those new Pods to start before removing the old Pods.
- In Kubernetes, updates are versioned and any Deployment update can be reverted to a previous (stable) version.
- the Service will load-balance the traffic only to available Pods during the update.
- when updates happens the new set of replicas are created one by one and the old set of replicas are down to 0 one by one in **rolling update**. when doing the undo old ones are up and new ones are down to zero one by one.

```
> kubectl get replicasets
NAME DESIRED CURRENT READY AGE
myapp-deployment-67c749c58c 0 0 0 22m
myapp-deployment-7d57dbdb8d 5 5 5 20m
```

**Rolling updates allow the following actions:**

1. Promote an application from one environment to another (via container image updates)
2. Rollback to previous versions
3. Continuous Integration and Continuous Delivery of applications with zero downtime

![alt text](image-1.png)

to update the image

- `kubectl set image deployment/<deployment-name> <container-name>=<new-image>`

To view the status of application upgrade

- `kubectl rollout status deployment/<deployment-name>`

```
> kubectl rollout status deployment/myapp-deployment
Waiting for rollout to finish: 0 of 10 updated replicas are available...
Waiting for rollout to finish: 1 of 10 updated replicas are available...
Waiting for rollout to finish: 2 of 10 updated replicas are available...
Waiting for rollout to finish: 3 of 10 updated replicas are available...
Waiting for rollout to finish: 4 of 10 updated replicas are available...
Waiting for rollout to finish: 5 of 10 updated replicas are available...
Waiting for rollout to finish: 6 of 10 updated replicas are available...
Waiting for rollout to finish: 7 of 10 updated replicas are available...
Waiting for rollout to finish: 8 of 10 updated replicas are available...
Waiting for rollout to finish: 9 of 10 updated replicas are available...
deployment "myapp-deployment" successfully rolled out
```

To view the history of application upgrade

- `kubectl rollout history deployment/<deployment-name>`

```
> kubectl rollout history deployment/myapp-deployment
deployments "myapp-deployment"
REVISION CHANGE-CAUSE
1        <none>
2        kubectl apply --filename=deployment-definition.yml --record=true
```

To rollback to a previous version

- `kubectl rollout undo deployment/<deployment-name>`

## Deployment Strategy

- The Deployment controller supports two different strategies for rolling updates:

1. **Recreate**

   - In this strategy, all existing Pods are killed one at a time, and new ones are created to replace them.
   - In the strategy user will faces the downtime of application during the update.

2. **Rolling Update**
   - In this strategy, new Pods are created and the old ones are killed one at a time.
   - In this strategy, user will not faces the downtime of application during the update.

![alt text](image-2.png)

![alt text](image-3.png) ![alt text](image-4.png)

There are two ways to update the deployment

1. **Manually**

   - The user can manually update the deployment by changing the image version in the deployment manifest. `kubectl edit deployment <deployment-name>`
   - or using commandline `kubectl set image deployment/<deployment-name> <container-name>=<new-image-name>`

2. **Automatically**
   - The user can also update the deployment automatically by using the image update policy.
