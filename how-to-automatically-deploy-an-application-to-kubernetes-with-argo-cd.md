```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-public-app
	namespace: argocd
spec:
	project: default
	source:
		repoURL: https://github.com/3pl-tferg-devops/my-public-app.git
		targetRevision: HEAD
		path: my-app
	destination:
		server: https://kubernetes.default.svc
	syncPolicy:
		automated:
			prune: true
			selfHeal: true
			allowEmpty: false
	syncOptions:
		- Validate=true
		- CreateNamespace=false
		- PrunePropagationPolicy=foreground
		- PruneLast=true
```

[[how-to-define-an-argo-cd-application-in-a-kubernetes-manifest]]

