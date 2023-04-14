[[argo-cd]]
[[docker]]
[[minikube]]
[[terraform]]

# Add a repo
```sh
helm repo add argo https://argoproj.github.io/argo-helm
```

Whenever you add a new repo you'll want to update the index
```sh
helm repo update
```

# Search for a chart
```sh
helm search repo argocd
```

Most of the time we want to override at least a few default variables in a chart. To do this you want to output the variables to a yaml file

```
helm show values argo/argo-cd --version 3.35.4 > argocd-defaults.yaml
```

# Get failed charts with 
```sh
helm list --pending -A
```
