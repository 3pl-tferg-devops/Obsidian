```yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app
	namespace: argocd
spec:
	project: default
	source:
		repoURL: https://github.com/3pl-tferg-devops/my-public-app.git
		targetRevision: HEAD
		path: my-app
	destination:
		server: https://kubernetes.default.svc
```

[[how-to-deploy-a-kubernetes-manifest]]

[[how-to-automatically-deploy-an-application-to-kubernetes-with-argo-cd]]

