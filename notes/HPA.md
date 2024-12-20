# **HorizontalPodAutoscaler**

- Horizontal scaling means that the response to increased load is to **deploy more [Pods](https://kubernetes.io/docs/concepts/workloads/pods/).** This is different from *vertical* scaling, which for Kubernetes would mean assigning more resources (for example: memory or CPU) to the Pods that are already running for the workload.
- If the load decreases, and the number of Pods is above the configured minimum, the **HorizontalPodAutoscaler** instructs the workload resource (the Deployment, StatefulSet, or other similar resource) to **scale back down.**
- HPA needs a [Metrics Server](https://github.com/kubernetes-sigs/metrics-server#readme) deployed
- If you are running [Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/), run the following command to enable metrics-server:

```bash
minikube addons enable metrics-server

helm install stable/metrics-server

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## **Create the HorizontalPodAutoscaler**

- create the autoscaler using `kubectl`. The [`kubectl autoscale`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#autoscale)
- Roughly speaking, the HPA [controller](https://kubernetes.io/docs/concepts/architecture/controller/) will increase and decrease the number of replicas (by updating the Deployment) to maintain an average CPU utilization across all Pods of 50%
- Create the HorizontalPodAutoscaler:

```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10

kubectl get hpa
```

- The output is similar to:

```bash
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   0% / 50%  1         10        1          18s

```

- when you increase the load in the pods

```bash
NAME         REFERENCE                     TARGET      MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   305% / 50%  1         10        1          3m
```

- You should see the replica count matching the figure from the HorizontalPodAutoscaler

```bash
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   7/7      7           7           19m
```

- **Stop generating load [](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#stop-load)**

```bash
NAME         REFERENCE                     TARGET       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache/scale   0% / 50%     1         10        1          11m

kubectl get deployment php-apache

NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   1/1     1            1           27m
```

- Once CPU utilization dropped to 0, the HPA automatically scaled the number of replicas back down to 1
- When an HPA reports “Unknown” targets, **it indicates that the HPA controller is unable to retrieve the current resource utilization metrics from the pods**.
    1. **Missing or incorrect metrics server**:
    
    ```bash
    kubectl get deployments -n kube-system | grep metrics-server.
    ```
    
    1. Install metric server if missing
    
    ```bash
    helm install stable/metrics-server
    ```
    
    1. Make sure the metric server pods are running
- get YAML file HPA

```bash
kubectl get hpa php-apache -o yaml > /tmp/hpa-v2.yaml
```

- output: in case of resource is CPU Utilization

```yaml
apiVersion: autoscaling/v2  # Specifies the API version for HPA (Horizontal Pod Autoscaler)
kind: HorizontalPodAutoscaler  # Defines the kind as HPA
metadata:
  name: php-apache  # Name of the HPA
spec:
  scaleTargetRef:  # Specifies the target resource (e.g., Deployment) to scale
    apiVersion: apps/v1  # API version of the target resource
    kind: Deployment  # Kind of the target resource (Deployment in this case)
    name: php-apache  # Name of the target Deployment
  minReplicas: 1  # Minimum number of replicas to maintain
  maxReplicas: 10  # Maximum number of replicas to scale up to
  metrics:  # Metrics to monitor for scaling
  - type: Resource  # Metric type is a Kubernetes resource
    resource:
      name: cpu  # Monitors CPU utilization
      target:
        type: Utilization  # Target type is CPU utilization percentage
        averageUtilization: 50  # Add/remove pods when CPU utilization crosses 50%
status:
  observedGeneration: 1  # Tracks the last observed generation of the HPA
  lastScaleTime: <some-time>  # Timestamp of the last scaling action
  currentReplicas: 1  # Current number of replicas
  desiredReplicas: 1  # Desired number of replicas based on metrics
  currentMetrics:  # Current metric values
  - type: Resource  # Metric type is a Kubernetes resource
    resource:
      name: cpu  # Metric being monitored is CPU
      current:
        averageUtilization: 0  # Current CPU utilization percentage
        averageValue: 0  # Current CPU resource value

```

- In case of resource in memory Utilization

```yaml
apiVersion: autoscaling/v2  # Specifies the API version for HPA (Horizontal Pod Autoscaler)
kind: HorizontalPodAutoscaler  # Defines the kind as HPA
metadata:
  name: php-apache-memory  # Name of the HPA
spec:
  scaleTargetRef:  # Specifies the target resource (e.g., Deployment) to scale
    apiVersion: apps/v1  # API version of the target resource
    kind: Deployment  # Kind of the target resource (Deployment in this case)
    name: php-apache  # Name of the target Deployment
  minReplicas: 1  # Minimum number of replicas to maintain
  maxReplicas: 10  # Maximum number of replicas to scale up to
  metrics:  # Metrics to monitor for scaling
  - type: Resource  # Metric type is a Kubernetes resource
    resource:
      name: memory  # Monitors memory utilization
      target:
        type: Utilization  # Target type is memory utilization percentage
        averageUtilization: 70  # Add/remove pods when memory utilization crosses 70%
status:
  observedGeneration: 1  # Tracks the last observed generation of the HPA
  lastScaleTime: <some-time>  # Timestamp of the last scaling action
  currentReplicas: 1  # Current number of replicas
  desiredReplicas: 1  # Desired number of replicas based on metrics
  currentMetrics:  # Current metric values
  - type: Resource  # Metric type is a Kubernetes resource
    resource:
      name: memory  # Metric being monitored is memory
      current:
        averageUtilization: 0  # Current memory utilization percentage
        averageValue: 0  # Current memory resource value
```

---

### Metric Types:

1. Resource
    1. **Supported Resources**: `cpu`, `memory`, and custom resources if defined in the cluster.
2. Pods
    1. This monitors a custom metric collected from each `pod` in the target deployment or stateful set.
    2. **Use Case**: Useful when you want to scale based on application-level metrics, like requests per second.
3. Object
    1. This monitors metrics on a specific Kubernetes object (e.g., Ingress, Service).
    2. **Use Case**: Useful for scaling based on metrics tied to other Kubernetes objects, such as the number of HTTP requests hitting an Ingress.
4. External
    1. This monitors metrics from an external monitoring system, like Prometheus or custom APIs.
    2. **Use Case**: Enables scaling based on external systems' data, such as message queues or cloud service metrics.

---

- **Custom Metrics**:
    - To use **Pods**, **Object**, or **External**, you need a custom metrics adapter (e.g., Prometheus Adapter).
- **Multiple Metrics**:
    - You can combine different metric types in the `metrics` list, and HPA will use the one that requires the most scaling action.
    - 
