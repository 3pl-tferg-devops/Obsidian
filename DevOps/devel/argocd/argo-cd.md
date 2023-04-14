After deploying the helm chart using terraform we can get the login password by running:
```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

To access the server UI we have to port-forward to the service
```
kubectl port-forward service/argocd-server -n argocd 8080:80 
```
Since we're not using https we're changing the port from 443 to 80

Navigate to: http://localhost:8080

Login using the username: Admin and the password you get from the above command. 

Now we need to tell ArgoCD to watch this repository and the my-app path for any changes

We'll make a new folder in the root dir of the repo for the application
```
argocd/
	1-example/ < -- this
		application.yaml
	terraform/
	my-app/
```

There are multiple ways to deploy apps, we'll start with the most basic custom resource CRD

```yaml
# application.yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app
	namespace: argocd
spec: 
	project: default
	source:
		repoURL: https://github.com/3pl-tferg-devops/argocd.git
		targetRevision: HEAD
		path: my-app
	destination:
		server: https://kubernetes.default.svc
```

`namespace`: Must be argocd. Not to be confused with the target namespace
`project`: Projects provide a logical grouping of applications which is handy when you're working with teams.
`repoURL`: Make sure to include the .git suffix as argocd doesn't follow redirects
`targetRevision`: you can use git branches, git tags or even primitive regex expressions. In the case of [[helm]] it should point to the chart version.
`path`: The path in the repo of the application deployment you want to track.
`destination`: The `server` is the location of the kubernetes api server. In this case we're using the local [[minikube]] cluster.

Now to get ArgoCD to track our repo, we need to manually apply the application for now. 

```sh
kubectl apply -f 1-example/application.yaml
```

You should now see the application show up in ArgoCD web console, but you'll notice it has not been automatically deployed. 

This is the default behaviour of ArgoCD 