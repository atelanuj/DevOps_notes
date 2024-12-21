## [Network Policy](https://earthly.dev/blog/kubernetes-network-policies/)

- you can only use the `NetworkPolicy` object in k8s when you have deployed a Networking solution inside the cluster.
  - **Antrea**
    - calico
    - cilium
  - **kube-router**
    - Romana
    - Weave-net
- If you want to control traffic flow at the IP address or port level (OSI layer 3 or 4), NetworkPolicies allow you to specify rules for traffic flow within your cluster, and also between Pods and the outside world.
- Your cluster must use a network plugin that supports `NetworkPolicy` enforcement.
- by default any pod in any node within cluster can communicates with any pod.
- With Network policies attached to a pod you can control the ingress(incomming) and egress(Outgoing) traffic from a pod.
  **User Case**
- Suppose we have a 3-tier application
  - front-end
  - back-end
  - Database
- we want only back-end pod can communicates with the database pod, not the front-end pod
- we can achive this with the help of `Network policy` object in K8s.

[Network](https://earthly.dev/blog/assets/images/kubernetes-network-policies/T9DWs2n.png)
![alt text](image-21.png)

**FEATURES OF NETWORK POLICIES**

- **Namespaced resources:** Network policies are defined as Kubernetes objects and are applied to a specific namespace. This allows you to control traffic flow between pods in different namespaces.
- **Additive:** In Kubernetes, network policies are additive. This means that if you create multiple policies that select the same pod, all the rules specified in each policy will be combined and applied to the pod.
- **Label-based traffic selection:** Kubernetes network policies allow you to select traffic based on pod labels. This provides a flexible way to control traffic flow within your cluster.
- **Protocol and port selection:** You can use network policies to specify which protocols (TCP-UDP-SCTP) and ports are allowed for incoming and outgoing traffic. This helps to ensure that only authorized traffic is allowed between pods.
- **Integration with network plugins:** Kubernetes network policies are implemented by network plugins. This allows you to use the network plugin of your choice and take advantage of its features and capabilities.

![alt text](image-22.png)

Create a pod `database` with `labels`

```yml
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
  labels:
    app: database
spec:
  containers:
  -  image: mysql
     name: database-pod
     command: ["sleep", "5000"]
```

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-pod-NP
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: back-end
      ports:
        - protocol: TCP
          port: 3306
```

Now to enable the traffic from another `namespace`, below will allow traffic from `back-end` pod from `prod` namespace to `database` pod

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-pod-NP
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: back-end
        - namespaceSelector:
            matchLabels:
              ns: prod
      ports:
        - protocol: TCP
          port: 3306
```

To control the outgoing traffic `engress` from the `database` pod to the `back-end` pod

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-pod-NP
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: back-end
        - namespaceSelector:
            matchLabels:
              env: prod
      ports:
        - protocol: TCP
          port: 3306

  engress:
    - to:
        - podSelector:
            matchLabels:
              app: back-end
        - namespaceSelector:
            matchLabels:
              ns: prod
      ports:
        - protocol: TCP
          port: 3303
```

to add external server to connect to the database pod, then you need to allow that server IP

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-pod-NP
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16 # add range of Ip address
            except:
              - 172.17.1.0/24
      ports:
        - protocol: TCP
          port: 6875
```

---

### **Kubectx**:

With this tool, you don't have to make use of lengthy “kubectl config” commands to switch between contexts. This tool is particularly useful to switch context between clusters in a multi-cluster environment.

**Installation**:

```
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
```

**Syntax**:

To list all contexts:

```
kubectx
```

To switch to a new context:

```
kubectx <context_name>
```

To switch back to previous context:

```
kubectx -
```

To see current context:

```
kubectx -c
```

---

### **Kubens**:

This tool allows users to switch between namespaces quickly with a simple command.

**Installation**:

```
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

**Syntax**:

To switch to a new namespace:

```
kubens <new_namespace>
```

To switch back to previous namespace:

```
kubens -
```

---

# Networking in kubernetes

## Network Namespace

A **Network Namespace** is a Linux kernel feature that provides isolation of network resources between different processes. It allows multiple containers, applications, or processes to have separate network stacks, making it a key building block for containerized environments like Docker and Kubernetes.

### **How Network Namespaces Work**

When you create a network namespace:

1.  A new virtual network stack is created.
2.  The new namespace is isolated from the host and other namespaces.
3.  Virtual network interfaces (veth pairs) are often used to connect the namespace to the host or other namespaces.

### **Key Features of Network Namespaces**

1.  **Isolation**:

    -   Each network namespace has its own network stack, including:
        -   **Interfaces** (e.g., `eth0`, `lo`)
        -   **Routing Tables**
        -   **Firewall Rules (iptables)**
        -   **Sockets**
2.  **Multiple Instances**:

    -   Multiple network namespaces can exist simultaneously on the same host.
    -   Each namespace can have different network configurations.
3.  **Default Namespace**:

    -   By default, all processes and applications use the host's network namespace.
    -   When a new namespace is created, it starts empty, without any network interfaces.
4.  **Independence**:

    -   Changes made in one namespace (e.g., adding routes or interfaces) do not affect others.

### **Creating and Managing Network Namespaces**

1.  **List Existing Namespaces**:

    `ip netns list`

2.  **Create a New Namespace**:

    `ip netns add my_namespace`

3.  **Run Commands in a Namespace**:

    `ip netns exec my_namespace <command>`

**Example:**

  `ip netns exec my_namespace ip a`

4.  **Delete a Namespace**:

    `ip netns delete my_namespace`


## Docker networking

1. **None**
   - Docker container inside the host connot talk to each other they are complely isolated from each other
2. **Host**
   - Docker container and host has no isolation, if the application running on port 80 for docker then it automatically run on port 80 of the host.
3. **overlay**
   - Overlay networks connect multiple Docker daemons together.
4. **ipvlan**
   - IPvlan networks provide full control over both IPv4 and IPv6 addressing.
5. **macvlan**
   - Assign a MAC address to a container.
6. **User-defined networks**
   - You can create custom, user-defined networks, and connect multiple containers to the same network. Once connected to a user-defined network, containers can communicate with each other using container IP addresses or container names.
7. **Bridge**
   - This is the default network in docker
   - In terms of Docker, a bridge network uses a software bridge which lets containers connected to the same bridge network communicate, while providing isolation from containers that aren't connected to that bridge network.
   - containers on different bridge networks can't communicate directly with each other.
   - User-define bridge networks are `superior` to the default `bridge` network.
     - User-defined bridges provide automatic DNS resolution between containers.
   - `docker0` on the host is the docker `bridge` network
   - Docker network is just a `network namespace` behind the scene
   - while doing the port fowarding in docker from host to container, docker creates the routing table behind the scene

```shell
iptables \
   -t nat \
   -A PREROUTING \
   -k DNAT \
   --dport 8080 \
   --to-destination 80
```

## Container Networking Interface (CNI)

- CNI is a standard for networking in container orchestration systems like Kubernetes.
- The Container Networking Interface (CNI) is a specification and framework for configuring networking in containerized environments. It is widely used in container orchestration systems, such as Kubernetes, to manage how containers communicate with each other, with services, and with external networks.
  - CNI defines how network interfaces should be created, configured, and deleted for containers.
  - It provides a standardized approach, ensuring that any CNI-compliant plugin can integrate seamlessly with a container runtime.

### Example: CNI Workflow in Kubernetes

**Pod Creation:**

- Kubernetes requests the container runtime to create a pod.
- The runtime calls the CNI plugin to configure the pod's network.

**CNI Plugin Execution:**

- The plugin allocates an IP address for the pod.
- It sets up network routes, rules, and interfaces.

**Networking Established:**

- The pod is now part of the cluster's network, with routing to other pods and services.

**Pod Deletion:**

- The CNI plugin removes the network configuration and releases the IP address.

### **Common CNI Plugins**

1.  **Flannel**:

    - Implements a simple overlay network.
    - Often used for Kubernetes cluster networking.

2.  **Calico**:

    - Supports advanced networking policies.
    - Provides Layer 3 networking without overlays for better performance.

3.  **Weave Net**:

    - Provides a simple-to-use network for Kubernetes and Docker clusters.

4.  **Cilium**:

    - Uses eBPF for high-performance and secure networking.

5.  **Multus**:

    - Enables attaching multiple CNI plugins to a single container.

### Cluster Networking:
- The network model is implemented by the container runtime on each node. The most common container runtimes use Container Network Interface (CNI)

## Service Networking
- Services are abstractions that expose a network service to other pods and services in the cluster.
- Service is a clusterwide, it exists accross all nodes in the cluster but whitin a namespace.
- it is not specific to a one node.
- Service has its own IP address
- When a pod tries to reach another pod it first reaches the service then routes to the pod.
### Types of Services
- **ClusterIP**: Internal service discovery within the cluster.
- **NodePort**: Exposes a service on each node's IP address.
- **LoadBalancer**: Uses a cloud provider's API to provsion a load balancer to expose a service.

## DNS in k8s
- Kubernetes uses DNS for service discovery.
