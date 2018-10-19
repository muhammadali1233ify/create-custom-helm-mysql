# create-custom-helm-for-mysql
## Description
some description
<br/>
## Procedure<br/>
1. The deployment, service and ingress files are written in the manifest folder(templates). The command `helm install ./` will install helm chart in the current directory that has a dependency over values.yaml, chart.yaml and templates. Let's discuss the purpose of each of these file in depth.<br/>

### Values.yaml<br/>
This file creates a hierarchical addressing system for files inside the template. For the value related fields, the addressing system begins with **.Value** and all the naming fields with **.Chart**. For e.g. `port: {{.Values.accountDatabase.image.port.Mysqlconport}}` means that port key inside the **service.yaml** file will have a value that is assigned to it via the **values.yaml**. The file inside values.yaml file is written as;
```
accountDatabase:
  image:
    port:
      Mysqlconport: 3306
```

### Chart.yaml
All the naming related files are written here. for example, this file defines the name of the container. If the container is for **mysql** and you want to name it as **wesahelmworkflow** then the chart will list the name as follows;
```
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for Kubernetes
name: wesahelmworkflow
.
.
.
```

and the deployment file will be as follows;

```
apiVersion: extensions/v1beta1
kind: Deployment
.
.
.
spec:
  selector:
    matchLabels:
      app: {{ .Values.accountDatabase.dblabelapp }}
      tier: {{ .Values.accountDatabase.dbtier }}
  strategy:
    type: {{ .Values.accountDatabase.stype }}
  template:
    metadata:
      labels:
        app: {{ .Values.accountDatabase.dblabelapp }}
        tier: {{ .Values.accountDatabase.dbtier }}
    spec:
      containers:
        - image: "{{ .Values.accountDatabase.image.repository }}:{{ .Values.accountDatabase.image.tag }}"
          name: {{ .Chart.Name }}
.
.
.
```

2. The deployment file inside of the templates folder contains a basic mqsql pod implementation. We'll commit all the potential changeable value inside this fill to the values.yaml file in the root. This will allow us to upgrade and rollback changes where ever we feel necessary. For example, if we want to work with **mysql:5.6** instead **mysql:5.0** then we'll simply apply helm upgrade command to version control the entire deployment in seconds. This is the power of helm. All the pods and their dependency on services and other pods will be upgraded simultaneously as well. The command is as below;

```
helm upgrade --set scale=3,tag=5.6 <deployement_name> ./
```

The number of pods for version 5.6 will be scaled to a total of 3 and the previous one pod will be destroyed.<br/>
3. To avoid issues with helm install, make sure that the syntax of all the yaml files are strictly followed.<br/>
4. To avoid issues with labels(it's app name and tier) make sure that addressing is done right and it is specified as a selector under spec key in your deployment file.

```
.
.
.
spec:
  selector:
    matchLabels:
      app: {{ .Values.accountDatabase.dblabelapp }}
      tier: {{ .Values.accountDatabase.dbtier }}
.
.
.
```

5. To avoid issues with container name, specify the name inside of **chart.yaml** files instead of the **values.yaml**.
