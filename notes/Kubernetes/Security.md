# Security

- controlling the access to `kube-apiserver` you can control the authentication
  - **who can acess** and **what can they do**

**Who can access?**
`Authentication mechanism`

- Username and passwords in a file
- Username and Token in a file
- Certificates
- LDAP
- Service Accounts

**Authorization in K8s**

1. RBAC Authorization (Role based access control)
2. ABAC Authorization (Attibute based access control)
3. Node Authorization
4. Webhook Mode

All communication between varius master components are secure using the `TLS certificate`
![alt text](image-10.png)

you can restricthe the communication between pods with `network policies`

## User Authentication in k8s cluster

- To Create a user in Kubernetes you need to create a `User.crt` and `User.key` and add it in the Kubernetes `Kubeconfig` file
- `User.crt` file must be signed by Kubernetes `CA authority`.
- User access the k8s cluster:
  - Admins
  - Developers
  - End users
  - bots
- K8s connot manage the Users natively
  - hence you cannot create or list user in k8s
- k8s relies on external user managment service like `LDAP`
- you can only manage the k8s service account
- all user access is managed by the `kube-apiserver`
- ![alt text](image-11.png)
- Auth Mechanisam for `kube-apiserver`

  - Username and passwords in a csv file
    `    password123,username1,user_id1
    password456,username2,user_id2
    password789,username3,user_id3
   `

        - `--basic-auth-file=user-details.csv`

    ![alt text](image-12.png)

- Username and Token in a csv file
- Certificates
- LDAP
- Service Accounts

**Communication between Pods:**

- `Pods` are isolated from each other
- by default pod on any node can talk with any pod within a cluster.
- you can restrict this by setting up a network policy.

## TLS Certificate Authentication: (https://)

- **TLS Certification**is a cryptographic protocol designed to provide secure communication over a computer network. It ensures data privacy, integrity, and authentication between clients and servers.
- with the help of `https` there is no intervention between sender and recevier.
- `https` is an extension of `http`
- `RSA` is no longer support as a method for key exchange, it is replaced by [Diffie-Hellman algorithm](https://www.techtarget.com/searchsecurity/definition/Diffie-Hellman-key-exchange) for more secure key exchange method.

### Process

1. **Handshake Process:** `TLS Handshake`
   - **Client Hello:** The client sends a "Client Hello" message to the server, which includes the TLS version, cipher suites, and a randomly generated number.
   - **Server Hello:** The server responds with a "Server Hello" message, selecting the TLS version and cipher suite from the client's list, and provides a randomly generated number.
2. **Server Authentication and Pre-Master Secret:**
   - **Server Certificate:** The server sends its digital certificate to the client, which includes the server’s public key and is issued by a trusted `Certificate Authority (CA).`
   - **Pre-Master Secret:** The client generates a pre-master secret _encryption session key_, _encrypts it with the server’s public key_, and sends it to the server.
3. **Session Keys Creation:**
   - Both the client and the server use the`pre-master secret` along with the previously exchanged random numbers to generate the same session keys, which are symmetric keys used for the session’s encryption and decryption.
4. **Secure Communication:**
   - **Change Cipher Spec:** Both client and server send a message to indicate that future messages will be encrypted using the session keys.
   - **Encrypted Messages:** The client and server communicate securely using symmetric encryption.
     ![alt text](image-13.png)

> The combination of TLS and Asystemtec provides a `robust` security framework, where TLS secures the communication channel and Asystemtec verifies user identities and manages sessions.

- **Symmentric Encryption**

  1.  it is a secure way of encryption
  2.  In symmetric key encryption, the same key is used for both encryption and decryption of data.
  3.  encrypted data and key travells through same network
  4.  hacker can intersept that key while sending and decrypt it.
  5.  **faster** way of authentication

- **Asymmtric Encryption**
  1.  Asymmetric encryption uses two mathematically connected keys: a **public** key and a **private** key.
      1. `ssh-keygen` to generate the private and public keys
  2.  The **public** key is used for encryption of user data, while the **private** key is used for decryption of user data of the same server itself.
  3.  Server send the public key to client
  4.  client uses the public key to encrypt the private key of client
  5.  client sends the encrypted private key to server
  6.  server decrypts the encrypted key with its private key
  7.  client encrypts the data with the clients public key
  8.  client sends the encrypted data to server
  9.  server decrypts the data with the clients private key which server recived earlier.
  10. Reads the data.
  11. Hacker can place its own proxy server in place of actual server
  12. then client communicates with that hacker server and hacks the data.s
- **TLS Handshake**
  1.  `SSL/TLS` certs are created by CA authorities.
  2.  Application server sends its public key to CA authority.
  3.  CA authority issues a TLS certificate
  4.  CA authority enclose the application server key init and give **signature** with CA authority server public key.
      1. `SIGNATURE = Application server public key + CA authority server public key`
      2. signature is then added in the certificate.
  5.  Now that certificate from CA will be transfered to the application server with application server public key init.
  6.  now the application server sends that signed certificate to the client server
  7.  client server then recives the certificate from application server with application server public key init
  8.  client server then goes to the CA authority of the application server and brings its public key.
  9.  client then verify the SIGNATURE with public key if application server and Public key of CA authority.
  10. if the SIGNATURE matches then Client establish the connection with that application server, if not then connection is not been established.
  11. client uses that public key to encrypt the private key of client
  12. client send that encrypted private key of the client server to application server which was been encrypted by the public key of the application server.
  13. application server then drypt that client private key with application server private key, and stores that Client private key for that session.
  14. client encrypts the data with its public key and send to the application server
  15. application server recives that ecrypted data and drypt it with clients private key and reads the data.

## TLS certificate in Kubernetes Cluster

### Certificate and Key extensions

![alt text](image-14.png)

- TLS certificates are used to establish secure connection between the Master Node and Workers nodes or a System Admins is accessing the `kube-apiserver` with the help of `Kubectl`.
- kube-apiserver is itself act as server so need to be secure with `https` connection by the help of `TLS certificate` so it has its own `apiserver.crt` and `apiserver.key` .
- similarly `etcd-server` also has `etcdserver.crt` and `etcdserver.key`.
- Admin also requires the `admin.crt` and `admin.key` to access the `kube-apiserver`.
- similarly the `kube-scheduler` also talks with the `kube-apiserver` to schedule the pods on the worker nodes. So it also requires the TLS certificate and the key `scheduler.crt` and `scheduler.key`
- Even the Kube-apiserver requires the `apiserver.crt` and `apiserver.key` to talk to `etcd-server`
  ![alt text](image-15.png)

### How to generate the TLS certificate for k8s cluster

1. **Cluster Certificates**
   - **Generate the Private Key**
     - command = `openssl genrsa -out ca.key 2048`
   - **Generate the Certificate Signing Request (CSR)**
     - command = `openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr`
   - **Sign Certificates** - command = `openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt` - `ca.csr` is the cert with no sign - `ca.key` is used to sign the cert
     ![Cerificate](image-16.png)
2. **Client Certificates**
   - **Generate the Private Key**
     - command = `openssl genrsa -out admin.key 2048`
   - **Generate the Certificate Signing Request (CSR)**
     - command = `openssl req -new -key admin.key -subj "/CN=KUBE-admin" -out admin.csr`
   - **Sign Certificates**
     - command = `openssl x509 -req -in admin.csr -signkey admin.key -out admin.crt`
     - `admin.csr` is the cert with no sign
     - `admin.key` is used to sign the cert
     - `admin.crt` is the final signed certificate

**To view all the certificates**

```
cat /etc/kubernetes/manifests/apiserver.yaml
```

## [KubeConfig](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

- To configure the user with the kubeAPI server you need to use the KubeAPI server.
- Kubeconfig is a configuration file that contains the information about the **cluster**, **user** and the **authentication information (contexts)**.
- It is located inside the `~/.kube/config`

**Clusters**

- Clusters have information about the different environments.
  - Development
  - UAT
  - Production

**Users**

- Users have information about the different users.
- Which user has access to the which cluster
  - Admin user
  - Dev user
  - Prod user
- These user may have different privileges to different cluster.

**Creating User in Kubernetes**

- Kubernetes does not have a concept of users, instead it relies on certificates and would only trust certificates signed by its own CA.
- To get the CA certificates for our cluster, easiest way is to access the master node.
- Because we run on kind, our master node is a docker container.
- The CA certificates exists in the `/etc/kubernetes/pki` folder by default.

```
docker exec -it rbac-control-plane bash

ls -l /etc/kubernetes/pki
total 60
-rw-r--r-- 1 root root 1135 Sep 10 01:38 apiserver-etcd-client.crt
-rw------- 1 root root 1675 Sep 10 01:38 apiserver-etcd-client.key
-rw-r--r-- 1 root root 1143 Sep 10 01:38 apiserver-kubelet-client.crt
-rw------- 1 root root 1679 Sep 10 01:38 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1306 Sep 10 01:38 apiserver.crt
-rw------- 1 root root 1675 Sep 10 01:38 apiserver.key
-rw-r--r-- 1 root root 1066 Sep 10 01:38 ca.crt
-rw------- 1 root root 1675 Sep 10 01:38 ca.key
drwxr-xr-x 2 root root 4096 Sep 10 01:38 etcd
-rw-r--r-- 1 root root 1078 Sep 10 01:38 front-proxy-ca.crt
-rw------- 1 root root 1679 Sep 10 01:38 front-proxy-ca.key
-rw-r--r-- 1 root root 1103 Sep 10 01:38 front-proxy-client.crt
-rw------- 1 root root 1675 Sep 10 01:38 front-proxy-client.key
-rw------- 1 root root 1679 Sep 10 01:38 sa.key
-rw------- 1 root root  451 Sep 10 01:38 sa.pub

exit the container
```

```
cd kubernetes/rbac
docker cp rbac-control-plane:/etc/kubernetes/pki/ca.crt ca.crt
docker cp rbac-control-plane:/etc/kubernetes/pki/ca.key ca.key
```

- As mentioned before, Kubernetes has no concept of users, it trusts certificates that is signed by its CA.
  This allows a lot of flexibility as Kubernetes lets you bring your own auth mechanisms, such as [OpenID](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens) Connect or OAuth.
- First thing we need to do is create a certificate signed by our Kubernetes CA.
- Easy way to create a cert is use openssl and the easiest way to get openssl is to simply run a container:

  ```
  docker run -it -v ${PWD}:/work -w /work -v ${HOME}:/root/ --net host alpine sh

  apk add openssl
  ```

- Let's create a certificate for `Bob Smith`:
  ```
  #start with a private key
  openssl genrsa -out bob.key 2048
  ```
  Now we have a key, we need a certificate signing request (CSR).
  We also need to specify the groups that Bob belongs to.
  Let's pretend Bob is part of the Shopping team and will be developing applications for the Shopping

```
openssl req -new -key bob.key -out bob.csr -subj "/CN=Bob Smith/O=Shopping"
```

Use the CA to generate our certificate by signing our CSR.
We may set an expiry on our certificate as well

```
openssl x509 -req -in bob.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out bob.crt -days 1
```

**Building a kube config**
Let's install kubectl in our container to make things easier:

```
apk add curl nano
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
```

We'll be trying to avoid messing with our current kubernetes config.
So lets tell kubectl to look at a new config that does not yet exists

```
export KUBECONFIG=~/.kube/new-config
```

Create a cluster entry which points to the cluster and contains the details of the CA certificate:

```
kubectl config set-cluster dev-cluster --server=https://127.0.0.1:52807 \
--certificate-authority=ca.crt \
--embed-certs=true

#see changes
nano ~/.kube/new-config
```

```
kubectl config set-credentials bob --client-certificate=bob.crt --client-key=bob.key --embed-certs=true\
kubectl config set-context dev --cluster=dev-cluster --namespace=shopping --user=bob
kubectl config use-context dev
```

**Contexts**

- Contexts are use to link the user and cluster together.
- It is used to switch between different cluster and user.
- For example, you can have a context for the development cluster and another for the production cluster
- use need to set the current context in the config file which tells the kubectl to which user to user for which cluster.
- you can also set the working namespace for a context

**To view the Kubeconfig**

- To view the config file
  - `kubectl config view`
- To use a custom config file
  - `kubectl config view --kubeconfig=my-custom-config`
- To change the current context
  - `kubectl config use-context <user@cluster>`
- To see only the configuration information associated with the current context, use the `--minify` flag.
  - `kubectl config --kubeconfig=config-demo view --minify`

**Example config file**

```yml
apiVersion: v1
kind: Config
current-context: user@cluster

clusters:
- name: development
  cluster:
      certificate-authority: ca.crt
      # to add ca.crt file content in base64 encoded format directly in config file
      # certificate-authority-data:
      server: 0.0.0.0
- name: test
  cluster:
      certificate-authority: ca.crt
      server: 0.0.0.0

users:
- name: developer
  user:
      client-certificate: admin.crt
      client-key: admin.key
- name: experimenter
  user:
      client-certificate; admin.crt
      client-key: admin.key

contexts:
- name: developer@development
  context:
      cluster: development
      user: developer
      namespace: frontend
- name: experimenter@test
  context:
      cluster: test
      user: experimenter
      namespace: storage
```

**Add context details to your configuration file:**

```
kubectl config --kubeconfig=config-demo set-context dev-frontend --cluster=development --namespace=frontend --user=developer
kubectl config --kubeconfig=config-demo set-context dev-storage --cluster=development --namespace=storage --user=developer
kubectl config --kubeconfig=config-demo set-context exp-test --cluster=test --namespace=default --user=experimenter
```

> **Note**:
> A file that is used to configure access to a cluster is sometimes called a _kubeconfig file_. This is a generic way of referring to configuration files. It does not mean that there is a file named `kubeconfig`.

**Set the KUBECONFIG environment variable**

- Linux
  - `export KUBECONFIG_SAVED="$KUBECONFIG"`
- Append $HOME/.kube/config to your KUBECONFIG environment variable
  - `export KUBECONFIG="${KUBECONFIG}:${HOME}/.kube/config`"
- Windows PowerShell
  - `$Env:KUBECONFIG_SAVED=$ENV:KUBECONFIG`
  - `$Env:KUBECONFIG="$Env:KUBECONFIG;$HOME\.kube\config"`

## Kube API groups

- In Kubernetes, API groups are a way to organize different kinds of API resources, helping to make the API more modular and scalable. This approach allows the introduction of new APIs and versions without disturbing the core API, improving Kubernetes' extensibility. API groups essentially group related resources and provide versioning for these resources

**Structure of API Groups:**

`/apis/{group}/{version}/{resource}`

for Example:

`/apis/apps/v1/deployments`

- `apps` is the API group.
- `v1` is the API version.
- `deployments` is the resource being accessed.

### Types of API groups:

1. **Core Group (Legacy or No Group)** **(api)**

- Also known as the core or legacy API, this group doesn't have a name in the URL.
- The resources here are some of the core Kubernetes components, like **Pods**, **Services**, **Namespaces**, and **Nodes**.
- `/api/v1/pods`

---

2. **Named Groups** **(apis)**

- These are groups with names and are organized based on the functionality they serve.
- Each named group can have different versions (e.g., `v1`, `v2beta`, etc.), allowing backward compatibility and smooth transitions to newer versions.
- Examples:
- `apps`: This API group contains resources like **Deployments**, **DaemonSets**, and **StatefulSets**.
  - `/apis/apps/v1/deployments`
- `batch`: Handles jobs and cron jobs.

  - `/apis/batch/v1/jobs`

- `networking.k8s.io`: Deals with network-related resources like **Ingress** and **NetworkPolicies**.

  - `/apis/networking.k8s.io/v1/ingresses`

- `rbac.authorization.k8s.io`: Manages Role-Based Access Control (RBAC) resources like **Roles** and **RoleBindings**.
  - `/apis/rbac.authorization.k8s.io/v1/roles`

---

3. **Custom Resource Definitions (CRDs)**

- Kubernetes allows users to define their own APIs by creating `Custom Resource Definitions` (CRDs), which belong to custom groups and provide an extension to Kubernetes' built-in functionality.
- CRDs can be used to define resources in an organization-specific group.
- `/apis/custom.group/v1/myresource`