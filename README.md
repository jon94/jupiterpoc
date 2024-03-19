#jupiterpoc

## Install Datadog into EKS Clusters using Helm


<details>
<summary>Click to toggle for `steps`</summary>

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


