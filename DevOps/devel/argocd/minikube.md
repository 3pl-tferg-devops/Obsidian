[[argo-cd]]
[[docker]]
[[helm]]

Minikube installs a kubernetes cluster, locally on youre computer with a single node. It's lightweight and useful for testing new software. 

To install minikube on MacOS, you can simply use brew and run

```sh
brew install minikube
```

To uninstall minikube, you can run the below commands

```sh
brew uninstall minikube # this should be enough
brew uninstall --force minikube
brew cleanup
```

If you want to reinstall minikube make sure you unlink and link it again.

```sh
brew install minikube
brew unlink minikube
brew link minikube
```

Once you've finished installing it,  you can start it with 

```sh
minikube start
```

minikube relies on a container driver like [[docker]] so make the daemon is running before you start it.

If you want, you can specify a particular version of kubernetes to deploy 

```sh
minikube start --kubernetes-version=v1.26.1
```

You can also specify the driver you want to use as well.

```sh
minikube start --driver=docker
```

If you don't want to specify the docker version each time you launch the cluster, you can set the config like so

```sh
minikube config set driver docker
```

## Port forward the service
```
kubectl port-forward svc/argocd-server -n argocd 8080:80
```

## Create Secret
```
apiVersion: v1
kind: Secret
metadata:
	name: agrocd-ssh-private-key
	namespace: argocd
	labels:
		argocd.argoproj.io/secret-type: repository
	stringData:
		url: https://github.com/3pl-tferg-devops/argocd.git
		sshPrivateKey: |
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACBRicDOYOU8+5fNZIHvgOZslF7S00xg9aiAw7dV+0iXsAAAAJDWNu/c1jbv
3AAAAAtzc2gtZWQyNTUxOQAAACBRicDOYOU8+5fNZIHvgOZslF7S00xg9aiAw7dV+0iXsA
AAAECaxkvkPPG1zNNGy7rIFcimeJdQfPk7WuYfqALrkDXrnFGJwM5g5Tz7l81kge+A5myU
XtLTTGD1qIDDt1X7SJewAAAABmFyZ29jZAECAwQFBgc=
-----END OPENSSH PRIVATE KEY-----
		insecure: "false"
		enableLfs: "true"
```
Make sure to create the secret in the cluster first manually otherwise you're going to get the same error.