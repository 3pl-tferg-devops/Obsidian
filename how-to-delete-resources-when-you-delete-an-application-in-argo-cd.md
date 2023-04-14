```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app
	namespace: argocd
	finalizers:
		- resources-finalizer.argocd.argoproj.io
spec:
	project: default
	source:
		repoURL: https://<your-public-repo-url-here>.git
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

[[what-order-are-resources-deleted-with-a-finalizer-set-on-an-application-in-argo-cd]]
