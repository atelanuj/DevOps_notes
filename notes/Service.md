# Service

- Expose an application running in your cluster behind a single outward-facing endpoint, even when the workload is split across multiple backends.
- The Service API, part of Kubernetes, is an abstraction to help you expose groups of Pods over a network
- The set of Pods targeted by a Service is usually determined by a [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) that you define.
- Ingress is not a Service type, but it acts as the entry point for your cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
	# name of the service
  name: my-service
spec:
  selector:
	  # to which pod this service applies
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
	    # Service Port
      port: 80
      # at pod level
      targetPort: 9376
```

- Applying this manifest creates a new Service named "my-service" with the default ClusterIP [service type](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types).
- pods are not accessible from outside they can be access from inside the cluster only.

## ClusterIP

- This default Service type assigns an IP address from a pool of IP addresses that your cluster has reserved for that purpose.

![image]https://miro.medium.com/v2/resize:fit:4800/format:webp/1*dLlC4L2qpImyZS6gOntUjg.png

![image]https://i.octopus.com/blog/2022-11/difference-clusterip-nodeport-loadbalancer-kubernetes/clusterip.png

## NodePort

- the Kubernetes control plane allocates a port from a range **(default: 30000-32767)**
- Using a NodePort gives you the freedom to set up your own load balancing solution, to configure environments that are not fully supported by Kubernetes, or even to expose one or more nodes' IP addresses directly

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      # you can either specify the nodeport of k8s allocates automatically.
      nodePort: 30007
```

![image]https://i.octopus.com/blog/2022-11/difference-clusterip-nodeport-loadbalancer-kubernetes/nodeport.png

## LoadBalancer

- `LoadBalancer` services create external network infrastructure to direct network requests to pods in the cluster. On cloud platforms, like Azure, AWS, and GCP, the external load balancer is typically provided by one of the cloud provider's existing load balancer services. For example, an [**EKS cluster**](https://aws.amazon.com/eks/) on AWS may create an [**Elastic Load Balancer (ELB)**](https://aws.amazon.com/elasticloadbalancing/) to expose pods to public network traffic.
- The downside to `LoadBalancer` services is that they usually incur additional costs. For example, an ELB instance costs roughly $16 per month when left running 24 hours a day, and that's before any costs associated with network traffic.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

![image]https://i.octopus.com/blog/2022-11/difference-clusterip-nodeport-loadbalancer-kubernetes/loadbalancer.png