# Jupiter POC Datadog

## Install Datadog into EKS Clusters using Helm

<details>
<summary>Click to toggle for `steps`</summary>

- Use values.yaml that can be found in [helm/values.yaml](https://github.com/jon94/jupiterpoc/blob/main/helm/values.yaml)
- Obtain Datadog API Key and place in values.yaml
- Obtain Datadog APP Key and place in values.yaml
- Replace datadog.clusterName with your cluster name.
  - This serves as metadata for you to identify the cluster name that will show up in the Datadog UI.
  - Lowercase letters, numbers, and hyphens only.
  - Must start with a letter.
  - Must end with a number or a letter.
  - Overall length should not be higher than 80 characters.

- Create Namespace 
```
kubectl create ns datadog
```

- Create Daemonset and necessary resources using helm
```
helm repo add datadog https://helm.datadoghq.com
helm repo update
helm install datadog datadog/datadog -n datadog -f values.yaml
```

</details>


