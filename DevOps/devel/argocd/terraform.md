[[helm]]
[[argo-cd]]

Installing a helm chart using terraform

Create a new folder in your project folder for the terraform configuration:

Add a providers.tf file to connect to the kube cluster

```hcl
provider "helm" {
  kubernetes {
    config_path = "~/.kube/config"
  }
}
```

If you're using [[minikube]] you can simply point to the kube config file. 

I'm also going to include a required providers block even though I don't belive it's necessary here. 

Now in the main.tf you'll want to add the `helm_release` resource:
```hcl
resource "helm_release" "argocd" {
  name = "argocd"

  repository       = "https://argoproj.github.io/argo-helm"
  chart            = "argo-cd"
  namespace        = "argocd"
  create_namespace = "true"
  version          = "3.35.4"
}
```

There are a couple of ways to override variables. One way is to use individual set statements and to target individual values. This can be cumbersome when you have to setup tolerations and affinity as it turns out to be a lot of set statements. 

Or just create a `values/argocd.yaml` file and specify the path to that file. This is the preferred method.

```hcl
resource "helm_release" "argocd" {
  name = "argocd"

  repository       = "https://argoproj.github.io/argo-helm"
  chart            = "argo-cd"
  namespace        = "argocd"
  create_namespace = "true"
  version          = "3.35.4"

  values = [file("values/argocd.yaml")]
}
```