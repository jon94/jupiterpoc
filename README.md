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
  
```
kubectl get mutatingwebhookconfiguration

NAME                                                       WEBHOOKS  
datadog-webhook                                            3          
```
