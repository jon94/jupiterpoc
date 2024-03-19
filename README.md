# Jupiter POC Datadog

## Install Datadog into EKS Clusters using Helm

<details>
<summary>Click to toggle for steps</summary>

- **Use values.yaml that can be found in [helm/values.yaml](https://github.com/jon94/jupiterpoc/blob/main/helm/values.yaml)**
- **Obtain Datadog API Key and place in values.yaml**
- **Obtain Datadog APP Key and place in values.yaml**
- **Replace datadog.clusterName with your cluster name.**
  - This serves as metadata for you to identify the cluster name that will show up in the Datadog UI.
  - Lowercase letters, numbers, and hyphens only.
  - Must start with a letter.
  - Must end with a number or a letter.
  - Overall length should not be higher than 80 characters.

- **Create Namespace** 
```
kubectl create ns datadog
```

- **Create Daemonset and necessary resources using helm**
```
helm repo add datadog https://helm.datadoghq.com
helm repo update
helm install datadog datadog/datadog -n datadog -f values.yaml
```
</details>

## Validate that installation is successful

<details>
<summary>Click to toggle for steps</summary>

- **Validate Daemonset number matches your node count.**
  
  - If no, you might have to set [tolerations](https://github.com/DataDog/helm-charts/blob/main/charts/datadog/values.yaml) on the datadog components.
    - See helm values in link above for tolerations.
   
```
kubectl get ds -n datadog

NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
datadog   3         3         3       3            3           kubernetes.io/os=linux   11m
```

- **Validate that the mutatingwebhookconfigurations for datadog is installed**
  - This is required for the APM Library injection to work.
  
```
kubectl get mutatingwebhookconfiguration

NAME                                                       WEBHOOKS  
datadog-webhook                                            3          
```
</details>

## Label and Annotate Deployments for APM Injection

<details>
<summary>Click to toggle for steps</summary>

- Example Manifest
  - Read [here](https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/?tab=kubernetes#containerized-environment) to understand more about how you can set up unified service tagging for easier correlation in Datadog.
  - Example is already shown in example manifest below.

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    tags.datadoghq.com/env: dev  # Add here for unified service tagging
    tags.datadoghq.com/service: adservice # Add here for unified service tagging
    tags.datadoghq.com/version: 1.0.0 # Add here for unified service tagging
  name: adservice
  namespace: default
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: adservice
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      annotations:
        admission.datadoghq.com/java-lib.version: v1.31.2 # Add here for APM lib injection
      labels:
        admission.datadoghq.com/enabled: "true" # Add here for APM lib injection
        app: adservice
        tags.datadoghq.com/env: dev   # Add here for unified service tagging
        tags.datadoghq.com/service: adservice    # Add here for unified service tagging
        tags.datadoghq.com/version: 1.0.0    # Add here for unified service tagging
    spec:
      containers:
      - env:
        - name: PORT
          value: "9555"
        - name: DD_INTEGRATION_KOTLIN_COROUTINE_EXPERIMENTAL_ENABLED    # Add here experimental kotlin coroutine
          value: true        
        image: docker.io/smazzone/adservice:a759553da31c4093c95a54403554188ec2ac229765d6c14405bbc18bce9825ae
        imagePullPolicy: IfNotPresent
        name: server
        ports:
        - containerPort: 9555
          protocol: TCP
        resources:
          limits:
            cpu: 300m
            memory: 300Mi
          requests:
            cpu: 200m
            memory: 180Mi
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - all
          privileged: false
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext:
        fsGroup: 1000
        runAsGroup: 1000
        runAsNonRoot: true
        runAsUser: 1000
      serviceAccount: default
      serviceAccountName: default
      terminationGracePeriodSeconds: 5
```

</details>
