# [Helm Chart](https://helm.sh/docs/)

- **Helm** is a package manager for Kubernetes that simplifies the process of installing, upgrading, and managing applications (referred to as “charts”) in a Kubernetes cluster. It provides a consistent and repeatable way to deploy applications, making it easier to manage complex deployments and reduce errors.
- to create a helm chart use command
    
    `helm create testchart`
    
- Output

```bash
-- testchart
    |-- Chart.yaml
    |-- charts
    |-- templates
    |   |-- NOTES.txt
    |   |-- _helpers.tpl
    |   |-- deployment.yaml
    |   |-- hpa.yaml
    |   |-- ingress.yaml
    |   |-- service.yaml
    |   |-- serviceaccount.yaml
    |   `-- tests
    |       `-- test-connection.yaml
    `-- values.yaml
	  `-- values-dev.yaml
```

- Contents of the `Charts.yaml`

```yaml
apiVersion: v2
name: testchart
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"
```

- contents of `values.yml`
1. `values.yaml`

```yaml
# Deployment Values
appName: myapp
replica: 3
appimage: myapp

# ConfigMap Values
configmap:
	name: my_configmap
	data:
		ui_properties_file_name: "my_vlaue"

# Application Image values
image:
	name: docker.hub/myapp
	tag: "v1"
```

1. `values-dev.yaml` 

```yaml
# env: Dev

# ConfigMap Values
configmap:
	name: my_configmap_dev
	data:
		ui_properties_file_name: "my_vlaue_dev"
	
# Deployment Values
appName: myapp_dev
replica: 2
appimage: myapp_dev
```

- To install the helm chart use `helm install <application_release> <chart_name>`
    - `-f` to use custom values.yml `helm install <my-release> <chart_name> -f values-dev.yaml`
    - `-n` to deploy the chart in a k8s namespace
- In Templates you have configuration files such as `deployment.yml` `service.yml` `configmap.yml` etc.
1. `deployment.yml`

```yaml
apiVersion: apps/v1
kind: Deployment 
metadata:
# values comming from values.yml
  name: {{ .Values.appName }} 
spec:
  replicas: {{ .Values.replicas }}
  selector:
    matchLabels:
    # values comming from values.yml
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
      # values comming from values.yml
        app: {{ .Values.appName }}
    spec:
      containers:
      - name: myapp-container
        image: {{ .Values.image.name}}:{{ .Values.image.tag }}  
        ports:
        - containerPort: 80
```

1. `configmap.yml`

```yaml
apiVersion: v1
kind: {{ .Values.configmap.name }}
metadata:
  name: game-demo
data:
  # property-like keys; each key maps to a simple value
  player_initial_lives: "3"
  ui_properties_file_name: {{ .Values.configmap.data.ui_properties_file_name }} 
```

- `helm ls` lists all releases (deployments) in the current namespace. If no namespace is specified, it defaults to the current namespace.
    - `--all-namespaces`

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/17521e93-a4c2-4d7a-a7cb-2d4bc636c987/e9562e81-cd23-449a-8519-146f993f0f2c/image.png)

```bash
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
```

- `helm upgrade <application_release> <helm_chart> --values <helm_chart>/values.yml`
    - This command upgrades a release to a new version of a chart. The upgrade arguments must be a release and chart. The chart argument can be either: a chart reference('example/mariadb'), a path to a chart directory, a packaged chart, or a fully qualified URL. For chart references, the latest version will be specified unless the '--version' flag is set.
    - To override values in a chart, use either the '`--values`' flag and pass in a file or use the '`--set`' flag and pass configuration from the command line, to force string values, use '--set-string'. You can use '`--set-file`' to set individual values from a file when the value itself is too long for the command line or is dynamically generated. You can also use '--set-json' to set json values (scalars/objects/arrays) from the command line.
    - The old pods get terminated and new pods are created.
- `NOTES.txt` is used to store metadata or information, specific command like how to access the application.
- Check the rendered templates with:

```yaml
helm template ./my-chart -f values-dev.yml
```

- Use the `helm repo add` command to add a repository. For example, to add the official Helm stable repository:

```yaml
helm repo add <repo-name> <repo-url>
```

- **List Repositories**

```yaml
helm repo list
```

- Search Charts in the Repository

```yaml
helm search repo <repo-name>
```

- Remove a Repository (Optional)

```yaml
helm repo remove <repo-name>
```