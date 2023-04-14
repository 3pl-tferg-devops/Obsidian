```terraform
resource "helm_release" "argocd" {
	name             = "argocd"
	repository       = "https://argoproj.github.io/argo-helm"
	chart            = "argo-cd"
	namespace        = "argocd"
	create_namespace = true
	version          = "3.35.4"
	values           = [file("values/argocd.yaml")]
}
```

[[how-to-find-the-current-argo-cd-chart-version-in-helm]]

[[how-to-create-the-values-for-the-argo-cd-helm-release-in-yaml]]

