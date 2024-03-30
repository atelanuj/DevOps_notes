# Kubernetes Commands:
    for kind in `kubectl api-resources | tail +2 | awk ' { print $1 }'`; do kubectl explain $kind; done | grep -e "KIND:" -e "VERSION:"

or 
    
    kubectl api-resources

give the list nodes connected to the master plane including master

    Kubectl get nodes

gives detail information about the node
    Kubectl describe node <Private_IP_of_node/node_name>

    Kubectl create -f <pod.yml>
