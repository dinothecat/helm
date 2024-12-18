### What is Helm?

Package manager for Kubernetes. 
Define, install, and upgrade Kubernetes apps using command-line interface.

### What Does Helm Do?

Helm simplifies the deployment and management of applications on Kubernetes by using "charts." 
A Helm chart is a collection of files that describe a set of Kubernetes resources.

- **Package** Kubernetes resources into a single chart.
- **Install** applications on Kubernetes clusters easily.
- **Upgrade** applications with new versions.
- **Roll back** to previous versions if needed.

### Why is Helm Used?

Helm is used for several reasons:

- **Simplification**: It abstracts the complexity of Kubernetes resource management.
- **Reusability**: Charts can be reused across different projects and teams.
- **Version Control**: Manage application versions and rollbacks.
- **Configuration Management**: Manage configurations through values files.

### Benefits of Helm for Managing Kubernetes

1. **Efficiency**: Reduces the time and effort to deploy apps.
2. **Consistency**: Consistent deployments across different environments.
3. **Collaboration**: Share and collaborate on Helm charts.
4. **Ecosystem**: Pre-built charts are available in repos like Artifact Hub.

### Example Helm Chart Structure
```
my-helm-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
```

### Chart.yaml: 
Contains metadata about the chart: name, version, and description...
Track release/versioning when you install or upgrade a chart
```
apiVersion: v2
name: my-helm-chart
description: A simple Helm chart for a web application
version: 0.1.0
appVersion: "1.0"
```

### values.yaml: 
Contains the **default configuration** values for the chart. 
Users can override these values when installing the chart.
These values can be referenced in the `templates/deployment.yaml` and `templates/service.yaml`.
```
replicaCount: 1

image:
  repository: nginx
  tag: stable
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  # Annotations for the ingress controller
  annotations: {}
  # Hosts for the ingress
  hosts:
    - host: chart-example.local
      paths:
        - /
```

### templates/: 
This directory contains Kubernetes manifest templates. 
The templates will be rendered into valid Kubernetes YAML files when the chart is installed. 
The templates can use values from values.yaml.

**deployment.yaml**: A template for a kubernetes deployment
This file uses values from `values.yaml` to configure the deployment. 
Replaces the placeholders with values from `values.yaml` or any overridden values to create the kubernetes manifest
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
  labels:
    app: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Release.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80

```

### service.yaml: 
A template for a Kubernetes Service.
Similar to `deployment.yaml`, this file uses values from `values.yaml` to configure the service.
For example it might use `.Values.service.type` to determine the type of service (eg. `ClusterIP`, `NodePort`)
```
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
  selector:
    app: {{ .Release.Name }}
```

### `.Release` Notes:
The `.Release` object is a built-in context variable provided by Helm during rendering of the templates. 

It has release `.name`, `.namespace`, `version`, etc. 

`.Release.Name` is specified when you install the chart. For example `helm install my-release my-helm-chart` (my-release is hte .Name)

> `Release.Namespace`: Kubernetes namespace the release is installed in

> `Release.IsInstall`: boolean indicated if it's an install operation

> `Release.IsUpgrade`: boolean indicated if it's an upgrade operation

> `Release.Time`: time when the release was created or upgraded

Example template to see how `.Release.Name` is used to dynamically name the Kubernetes resource
```
<> yaml
metadata: 
  name: {{ .Release.Name }}-deployment
```
This will create a deployment with a name based on the release name specifified during installation
```
<> bash
helm install my-wordpress bitnami/wordpress
```
The deployment will be named `my-wordpress-deployment`. 

### Some basic helm commands

**Initialize Helm (for Helm 2.x only)**:

   > ```helm init```
   
**Add a Chart Repository**:

  >  ```helm repo add <repo-name> <repo-url>```
   
  >  ```helm repo add bitnami https://charts.bitnami.com/bitnami```

**Update Chart Repositories**:

  >  ```helm repo update```

**Search for Charts**:

   > ```helm search repo <chart-name>```
   
   > ```helm search repo wordpress```

**Install a Chart**:

   > ```helm install <release-name> <chart-name>```
   
   > ```helm install my-wordpress bitnami/wordpress```

**List Installed Releases**:

   > ```helm list```

**Get Information About a Release**:

   > ```helm get all <release-name>```
   
   > ```helm get all my-wordpress```

**Upgrade a Release**:

   > ```helm upgrade <release-name> <chart-name> [--set key=value]```
   
   > ```helm upgrade my-wordpress bitnami/wordpress --set wordpressPassword=newpassword```

**Rollback a Release**:

   > ```helm rollback <release-name> <revision>```
   
   > ```helm rollback my-wordpress 1```

**Uninstall a Release**:

   > ```helm uninstall <release-name>```
   
   > ```helm uninstall my-wordpress```

**Show Chart Information**:

   > ```helm show chart <chart-name>```
   
   > ```helm show chart bitnami/wordpress```

**Show Values for a Chart**:

   > ```helm show values <chart-name>```
   
   > ```helm show values bitnami/wordpress```

**Package a Chart**:

   > ```helm package <chart-directory>```
   
   > ```helm package my-helm-chart```

**Lint a Chart**:

   > ```helm lint <chart-directory>```
   
   > ```helm lint my-helm-chart```

### Summary
When you package this chart and install it using Helm, the templates will be rendered with the values specified in `values.yaml`, resulting in valid Kubernetes manifests that can be applied to your cluster. You can customize the chart by modifying the `values.yaml` file or by providing your own values during installation.
