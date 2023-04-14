[[what-is-argo-cd]]

[[what-is-the-workflow-for-argocd]]

[[setup-argo-cd-on-kubernetes]] 

Before installing [[minikube]] we're going to install [[kubectl]] which we need to interact with [[Kubernetes]]. We're going to use [[asdf]] to install this:
```zsh
asdf install kubectl 1.26.3
```

You can download and install [[minikube]] using [[homebrew]] with:
```zsh
brew install minikube
```

You'll also need to make sure that you have [[docker]] installed and running. 
```zsh
brew install docker
```

If you're on MacOS you may also need to install something like [[colima]] to get everything working.
```zsh
brew install colima
```

If you installed [[colima]] then you also need to start it with:
```zsh
colima start
```

You can check to make sure it's running correctly by running:
```zsh
docker images
```

You can set the default `kubernetes-version` and `driver`  so you don't have to specify it each time:
```zsh
minikube config set kubernetes-version v1.26.3
minikube config set driver docker
```

Once that has finished installing, we can bootstrap the cluster with:
```zsh
minikube start --kubernetes-version=v1.26.3 --driver=docker
```

You can check to make sure the cluster is running correctly by running
```zsh
kubectl get nodes
```

The most common way to install open-source software to your kubernetes cluster is with [[helm]]. 

You can install helm using:
```zsh
brew install helm
```

Now we're going to install ArgoCD to our [[Kubernetes]] cluster using a [[helm]] chart. 

To do this we first have to add the ArgoCD repo:
```zsh
helm repo add argo https://argoproj.github.io/argo-helm
```

Every time you add a new repo make sure you update the index:
```zsh
helm repo update 
```

Now we can search for the ArgoCD chart:
```zsh
helm search repo argocd
...
argo/argo-cd # look for this one and make note of the version
```

We should [[get-the-default-variables-from-the-helm-chart]] and output it to a file. 

Now we're going to deploy this [[helm]] chart to our [[minikube]] cluster using [[terraform]]. 

We want to isolate the [[terraform]] config from the application source code, so create and enter a new folder:
```zsh
mkdir terraform
cd $_
```

First we're going to define the provider which will allow us to authenticate with a cluster:
```hcl
# ./terraform/0-provider.tf

terraform {
  required_providers {
    helm = {
      source = "hashicorp/helm"
      version = "2.9.0"
    }
  }
}

provider "helm" {
	kubernetes {
		config_path = "~/.kube/config" # or wherever you placed it
	}
}
```

In the next file we'll create the configuration for deploying the ArgoCD [[helm]] chart. 
```terraform
# ./terraform/1-argocd.tf
resource "helm_release" "argocd" {
	name             = "argocd"
	repository       = "https://argoproj.github.io/argo-helm"
	chart            = "argo-cd"
	namespace        = "argocd"
	create_namespace = true
	version          = "3.35.4"

	values = [file("values/argocd.yaml")]
}
```

`repository` should be the repo of the helm chart we added in an earlier step. 
`chart` is the name of the chart we identified with the search command earlier
`namespace` is the default namespace for the helm chart, if you want to change this you need to udpate the default values.
`create_namespace` set to true in case the namespace doesn't already exist, this will create it for you
`version` this is the version of the helm chart we saw when we searched the repo earlier
`values` we're going to use a file with values we want to override the defaults of.

To do this we'll create a file and simply use the contents of it here.
```yaml
# ./terraform/values/argocd.yaml
---
global:
	image:
		tag: "v2.6.6"

server:
	extraArgs:
	- --insecure
```

`global.image.tag` is set so we can use the latest ArgoCD verison
`sever.extraArgs` here we're setting the --insecure flag because we don't want ArgoCD to generate a self-cert and redirect http traffic to https. 

This isn't how most people secure their endpoints. If you want to expose ArgoCD outside, you would use ingress and terminate https on the ingress level and then route plain http to ArgoCD. 

Initialize the [[terraform]] module with:
```terraform
terraform init
```

Then to deploy the [[helm]] chart we will run:
```terraform
terraform apply
```

If you want to check on the progress with [[helm]] you can run
```helm
helm status argocd -n argocd
```

You can also try and get failed charts with:
```helm
helm list --pending -A
```

To verify that ArgoCD has been successfully installed run:
```kubectl
kubectl get pods -n argocd
```
Make sure all the pods have a running status.

By default this helm chart will generate an Admin password and store it in a Kubernetes secret. You can view this with
```kubectl
kubeclt get secrets -n argocd
...
argocd-initial-admin-secret
```
This is used only once during startup, feel free to change it.

To get the password, we are going to run:
```kubectl
kubectl get secrets argocd-initial-admin-secret -o yaml -n argocd
```

The password is encoded in base64 which we will have to decode with:
```zsh
echo "<secret>" | base64 -d
```

If you notice a `%` being highlighted, ignore it. It indicates the end of a string and is a default behaviour of [[zsh]] 

Now to access the ArgoCD UI we can use the `port-forward` command:
```kubectl
kubectl port-forward svc/argocd-server -n argocd 8080:80
```

Then navigate to in your browser and login using `admin` and the password you got earlier. 

# Simulate Pipeline
Now we're ready to create our first CD pipeline with a public repo & image. 

In this part, we're going to create a public [[github]] repo and [[dockerhub]] image.

To create a new [[github]] repo, you can either do it manually through the UI or you can do it via [[terraform]].

To do it through [[terraform]] we'll modify the provider.tf file we created earlier and add the github provider like so:
```sh
# ./terraform/0-provider.tf

terraform {
  required_providers {
    helm = {
      source = "hashicorp/helm"
      version = "2.9.0"
    }
    github = { # <-- Here
      source = "integrations/github"
      version = "5.18.3"
    }
  }
}

provider "helm" {
	kubernetes {
		config_path = "~/.kube/config"
	}
}

provider "github" {} # <-- Here
```

Now we'll add the configuration for a new repository:
```
./terraform/2-github-public-repo.tf

resource "github_repository" "my-app-public" {
	name      = "my-app-public"
	private   = false
	auto_init = true
}

outputs "clone_url" {
	values = github_repository.my-app-public.http_clone_url
}
```

This repo will contain the [[Kubernetes]] manifests and [[helm]] charts for the application. By modifying the content of this repo, we'll operate our Kubernetes cluster. 

This is known as [[GitOps]]. 

Clone this repo to your local system
```zsh
git clone https://<whatever-the-terraform-output-is>
```

Now we need a [[dockerhub]] account to host the images we're making. 

You can host as many public images as you want with the free account, but only 1 private image. 

Once you make an account, if you navigate to here:
```
https://hub.docker.com/repositories/tomferguson3pl
```
You can create a new repository. Otherwise it's a little tricky to find #imo

To simulate some sort of a continuous delivery pipeline we need an image to use as a base. 

We're going to use the [[nginx]] image for this.

We're going to have to authenticate with [[dockerhub]] by using the built in [[docker]] command:
```
docker login --username <your-username-here>
```

Now that we're authenticated we can pull down the [[nginx]] image
```
docker pull nginx:1.23.3
```

To simulate the CD pipeline, we will increment image tags to deploy new versions. 
```
docker tag nginx:1.23.3 tomferguson3pl/nginx:v0.1.0
```

Use the below command to upload the image to a public repo in our account:
```
docker push tomferguson3pl/nginx:v0.1.0
```

Next we need to create a new [[Kubernetes]] deployment for that new [[docker]] image. 

In the repository we created earlier create a new folder for the application andcreate a new file for the namespace 
```
./my-app-public/my-app/0-namespace.yaml

---
apiVersion: v1
kind: Namespace
metadata:
	name: prod
```

Create the deployment using the image we uploaded to [[dockerhub]].
```
./my-app-public/my-app/1-deployment.yaml

---
apiVersion: apps/v1
kind: Deployment
metadata:
	name: nginx
	namespace: prod
	labels:
		app: nginx
spec:
	replicas: 1
	selector: 
		matchLabels:
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
			  image: tomferguson3pl/nginx:v0.1.0
			  port:
			    - containerPort: 80
```

Now commit these changes to your repository:
```
git add .
git commit -m 'deploy nginx'
git push origin main
```


# Configuring ArgoCD

Now we need to get ArgoCD to watch this repository.

If we head back to the original workspace where we deployed ArgoCD and we'll create the first example folder, then create an `Application` 
```yaml
# ./1-example/application.yaml

---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app
	namespace: argocd # not the target namespace
spec:
	project: default
	source:
		repoURL: https://<your-public-repo-url-here>.git
		targetRevision: HEAD
		path: my-app
	destination:
		server: https://kubernetes.default.svc
```

Theses CRDs are created along with the rest of the [[helm]] chart when you deploy ArgoCD. 

A project is a logical grouping of applications.

The repoURL it doesn't matter right now whether it's using SSH or HTTPS. It's recommended to use the .git suffix since ArgoCD doesn't follow redirects.

The targetRevision: HEAD points to your main branch's latest commit. You can use branches, tags or even regex expressions. In the case of [[helm]] we point to the chart verison. 

The path defines the path inside your repo to track . 

The destination points to the cluster you want to deploy to. In our case we're deploying our app to the same Kubernetes cluster where ArgoCD is running, so we need to provide a path to the local Kubernetes API server. 

Apply the changes using:
```
kubectl apply -f 1-example/application.yaml
```

By default ArgoCD will automatically refresh and compare the state of Kubernetes and git every 5 minutes. It will not apply the changes automatically by default.

Deploy the changes manually, by clicking  `Sync` and the application will deploy.

# Release a new version

Now we're going to simulate our CI/CD pipeline and release a new version of our app. 

We'll tag the [[nginx]] image with a new version number and push it to the repo to simulate a version upgrade
```
docker tag nginx:1.23.3 tomferguson3pl/nginx:v0.1.1
docker push tomferguson3pl/nginx:v0.1.1
```

Increment the image tag in the deployment manifest
```
./my-app-public/my-app/1-deployment.yaml

---
apiVersion: apps/v1
kind: Deployment
metadata:
	name: nginx
	namespace: prod
	labels:
		app: nginx
spec:
	replicas: 1
	selector: 
		matchLabels:
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
			  image: <your-username-here>/nginx:v0.1.1 <--- Here
			  port:
			    - containerPort: 80
```

Commit and push the changes to the repo:
```
git add .
git commit -m 'upgrade nginx to v0.1.1'
```

Without setting up a webhook, it can take ArgoCD up to 5min to detect a change. In the case of Github, if you reduce the refresh limit to 1min or less, you will get rate limited and your API calls get rejected. To setup a webhook means exposing ArgoCD to the internet which is not feasible for most companies. We can also just click refresh and ArgoCD will show that it's out of sync. 

Now we're going to automate this step.

Add the below to the application configuration
```yaml
# ./1-example/application.yaml

---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app
	namespace: argocd
spec:
	project: default
	source:
		repoURL: https://<your-public-repo-url-here>.git
		targetRevision: HEAD
		path: my-app
	destination:
		server: https://kubernetes.default.svc
	syncPolicy: # <--- from here
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

selfHeal: will allow synchronizing state if the Kubernetes state was changed, for example if someone makes a manual change on the cluster

allowEmpty: disables deleting all application resources during automatic syncing

syncOptions: 
Validate=true enables Kubernetes validation; to make sure your manifests don't contain any errors.
CreateNamespace=false; disables the creation of namespaces since we prefer to declare them manually

Apply the changes we made to the application
```
kubectl apply -f 1-example/application.yaml
```

To verify the automatic sync works, increment the tag of the docker image and push it to the repo
```
docker tag nginx:1.23.3 tomferguson3pl/nginx:v0.1.2
docker push tomferguson3pl/nginx:v0.1.2
```

Update the app deployment
```yaml
./my-app-public/my-app/1-deployment.yaml

---
apiVersion: apps/v1
kind: Deployment
metadata:
	name: nginx
	namespace: prod
	labels:
		app: nginx
spec:
	replicas: 1
	selector: 
		matchLabels:
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
			  image: tomferguson3pl/nginx:v0.1.2 <--- Here
			  port:
			    - containerPort: 80
```

Commit and push the changes to github
```
git add .
git commit -m 'upgrade nginx to v0.1.2'
```

ArgoCD will automatically deploy the new image to the cluster

The typical workflow for CI/CD pipeline is to have the build agent build the app and release a new docker image, then modifies the deployments in the repo with the new version.

Here is a script you can use on the build agent to do this
```bash
./upgrade.sh
#!/bin/bash

# exit when any command fails
set -e

new_ver=$1

echo "new version: $new_ver"

# Simulate release of new docker images
docker tag nginx:1.23.3 tomferguson3pl/nginx:$new_ver

# Push the new verison to Dockerhub
docker push tomferguson3pl/nginx:$new_Ver

# Create a temporary directory
tmp_dir=$(mktemp -d)
echo $tmp_dir

# Clone the repo to the temporary directory
git clone git@github.com:3pl-tferg-devops/my-app-public.git $tmp_dir

# Update the image tag
sed -i '' -e "s/tomferguson3pl\/nginx:.*/tomferguson3pl\/nginx:$new_ver/g" $tmp_dir/my-app/1-deployment.yaml

# Commit changes and push
cd $tmp_dir
git add .
git commit -m "Update image to $new_ver"
git push

# Cleanup temp folder
rm -rf $tmp_dir

```

Make the script executable:
```
chmod +x upgrade.sh
```

Execute the script, and provide a new tag:
```
./upgrade.sh v0.1.3
```

# App Of Apps Pattern

Now we'll see what happens when you delete the application resource.
```
kubectl delete -f 1-example/application.yaml
```

In ArgoCD the application is now deleted but if you check in Kubernetes you'll notice it's still running. 

To also delete the application from Kubernetes when we delete the resource we need to add a finalizer to the metadata 
```yaml
# ./1-example/application.yaml

---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app
	namespace: argocd
	finalizers: # <--- from here
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

The finalizers will remove the deployment from Kubernetes first and then remove it from the ArgoCD state. 

Delete the application to test that it's working.
```
kubectl delete -f 1-example/application.yaml
```

The application is being deleted in Kubernetes first, then in ArgoCD which is what we want most of the time. 

When you have a lot of applications, you don't want to create them all manually. 

A common approach to overcome this is the App of Apps pattern which centralizes the applications into a single app. 

Rename the Namespace from prod to foo
```yaml
./my-app-public/my-app/0-namespace.yaml

---
apiVersion: v1
kind: Namespace
metadata:
	name: foo # <--- here
```

Update the deployment with the new namespace
```yaml
./my-app-public/my-app/1-deployment.yaml

---
apiVersion: apps/v1
kind: Deployment
metadata:
	name: nginx
	namespace: foo # <--- here
	labels:
		app: nginx
spec:
	replicas: 1
	selector: 
		matchLabels:
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
			  image: tomferguson3pl/nginx:v0.1.2 
			  port:
			    - containerPort: 80
```

Clone the directory and rename it to `second-app` and then change the namespace from `foo` to `bar`

Create a folder structure to allow us to create multiple apps
```
environments/staging
	/apps
	/my-app
	/second-app
```

Put all the application manifests in the apps folder 
```yaml
# ./environments/staging/apps/my-app.yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app-foo # <--- here
	namespace: argocd
	finalizers:
		- resources-finalizer.argocd.argoproj.io
spec:
	project: default
	source:
		repoURL: https://<your-public-repo-url-here>.git
		targetRevision: HEAD
		path: environments/staging/my-app # <--- here
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

Create another application resource for the second app
```yaml
# ./environments/staging/apps/second-app.yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: second-app-bar # <--- here
	namespace: argocd
	finalizers:
		- resources-finalizer.argocd.argoproj.io
spec:
	project: default
	source:
		repoURL: https://<your-public-repo-url-here>.git
		targetRevision: HEAD
		path: environments/staging/second-app # <--- here
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

Your folder structure should look like this:
```
./environments/staging
	/apps
		my-app.yaml
		second-app.yaml
	/my-app
		0-namespace.yaml
		1-deployment.yaml
	/second-app
		0-namespace.yaml
		1-deployment.yaml
```

The workflow for [[helm]] and [[kustomize]] are similar, except that you target [[helm]] charts. 

Commit the changes to the repo
```sh
git add .
git commit -m 'create app of apps pattern'
git push origin main -f 
```

Create a second example folder and create a new application
```yaml
# ./2-example/application.yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: apps-staging # <--- here
	namespace: argocd
	finalizers:
		- resources-finalizer.argocd.argoproj.io
spec:
	project: default
	source:
		repoURL: https://<your-public-repo-url-here>.git
		targetRevision: HEAD
		path: environments/staging/apps # <--- here
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

This application resource will target the `environments/staging/apps` path and apply all of the application manifests within it. 

Apply the application
```
kubectl apply -f 2-example/application.yaml
```

ArgoCD will deploy the `apps-staging` application first, then `my-app-foo` and `second-app-bar`

Verify the pods are running in `foo` and `bar` namespaces
```
kubectl get pod -n foo
kubectl get pod -n bar
```

Delete the apps with
```
kubectl delete -f 2-example/application.yaml
```

This pattern also works well or [[helm]] charts and [[kustomize]] deploys as well.

Now we'll look at how to authenticate with a private [[github]] and  [[docker]] repo.

Create a new repository in [[dockerhub]] called `nginx-private` and set it to private. 

Make a new private repository in [[github]] using [[terraform]]
```hcl
./terraform/3-github-private-repo.tf

resource "github_repository" "my-app-private" {
	name      = "my-app-private"
	owner     = "3pl-tferg-devops"
	private   = true # <--- Here
	auto_init = true
}

resource "github_branch_default" "my-app-public" {
	repository = github_repository.my-app-public.name
	branch     = "main"
	rename     = true
}

outputs "clone_url" {
	values = github_repository.my-app-public.http_clone_url
}
```

Release a new [[docker]] image and push it to the new private repository
```
docker tag nginx:1.23.3 tomferguson3pl/nginx-private:v0.1.0
docker login --username tomferguson3pl
docker push tomferguson3pl/nginx-private:v0.1.0
```

The process is the same as with public images, except you've created the repository before hand and set it to private.

Clone the private [[github]] repo and create the file structure as you did previously. 
```yaml
# ./my-app
	# 0-namespace.yaml
	---
	apiVersion: v1
	kind: Namespace
	metadata:
		name: foo
	# 1-deployment.yaml
	---
	apiVersion: apps/v1
	kind: Deployment
	metadata:
		name: nginx
		namespace: foo
		labels:
			app: nginx
	spec:
		replicas: 1
		selector: 
			matchLabels:
				app: nginx
		template:
			metadata:
				labels:
					app: nginx
			spec:
				containers:
				- name: nginx
				  image: tomferguson3pl/nginx-private:v0.1.0 # <--- Here
				  port:
				    - containerPort: 80
```

Commit and push the changes
```
git add .
git commit -m 'deploy private nginx'
git push origin main
```

Create a new example folder and define the new application resource using the private repo
```yaml
# 3-example/applicaton.yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app-private
	namespace: argocd
	finalizers:
		- resources-finalizer.argocd.argoproj.io
spec:
	project: default
	source:
		repoURL: git@github.com:3pl-tferg-devops/my-app-private.git # <-- Here
		targetRevision: HEAD
		path: environments/staging/apps
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

We're not going to use HTTPS anymore, instead use SSH to clone the repo.

Now we'll authenticate ArgoCD with the private repo.

We're going to generate a new SSH key
```
ssh-keygen -t ed25519 -C "argocd" -f ~/.ssh/argocd_ed25519
```
This uses the elliptic curve to generate the key. 

On older machines you may have to use `rsa`
```
ssh-keygen -t rsa -b 4096 -c "argocd" -f ~/.ssh/argocd_rsa4096
```

Upload the public ssh key to the repository. We're going to do this with [[terraform]]

Get the public key
```
cat ~/.ssh/argocd_ed25519.pub
```

Create the new resource in [[terraform]]
```hcl
# ./terraform/03-github-private-repo.tf
resource "github_repository_deploy_key" "my-app-private" {
  title      = "argocd"
  repository = "my-app-private"
  key        = "<your-public-key>"
  read_only  = "true"
}
```

Create a secret in Kubernetes to link it with the private repo.
```yaml
# 3-example/git-repo-secret.yaml
---
apiVersion:  v1
kind: Secret
metadata:
	name: my-app-private
	namespace: argocd
	labels:
		argocd.argoproj.io/secret-type: repository
	stringData:
		url: git@github.com:3pl-tferg-devops/my-app-private.git
		sshPrivateKey: |
			<your-private-key>
		insecure: "false"
		enableLfs: true
```

Specify the same repo you used in the application resource. Note: It's not good to store secrets in version control. 

Get the private key:
```sh
cat ~/.ssh/argocd_ed25519 # no .pub extention
```

Create the secret:
```
kubectl apply -f 3-example/git-repo-secret.yaml
```

Apply the application:
```
kubectl apply -f 3-example/application.yaml
```

ArgoCD can now access the private repository 

No we'll authenticate with the private repository of the image. 

Generate a read-only token in our [[dockerhub]] account to allow access to our Kubernetes cluster. 
```
https://hub.docker.com/settings/security
```

Create a new secret of type docker-registry in the same namespace
```
kubectl create secret docker-registry dockerconfigjson -n foo \
	--docker-server="https://index.docker.io/v1/" \
	--docker-username=tomferguson3pl \
	--docker-password=<insert-token-here> \
	--docker-email=<your-email-address>
```

Update the deployment to use the secret
```yaml
# ./myapp/1-deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
	name: nginx
	namespace: foo
	labels:
		app: nginx
spec:
	replicas: 1
	selector: 
		matchLabels:
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
			  image: tomferguson3pl/nginx-private:v0.1.0 
			  port:
				- containerPort: 80
			imagePullSecrets: # <--- Here
			- name: dockerconfigjson
```

Commit and push the changes to the private repo.
```
git add .
git commit -m 'update deployment with secret'
git push origin main
```

Now we're going to look at deploying [[helm]] charts. We'll deploy the  [[metrics-server]] chart.

Typically used for horizontal pod autoscaling and can be used directly by running
```
kubectl top
```

Add the [[metrics-server]] repo
```
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
```

Search for `metrics-server`. Note of the version
```sh
helm search repo metrics-server
...
metrics-server/metrics-server  3.8.4 # <-- here
```

Save the default values
```
helm show values metrics-server/metrics-server --version 3.8.4 > metrics-server-defaults.yaml
```

Create a new example folder with an application resource.
```yaml
# ./4-example/application.yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: metrics-server
	namespace: argocd
	finalizers:
		- resources-finalizer.argocd.argoproj.io
spec:
	project: default
	source: 
		repoURL: https://kubernetes-sigs.github.io/metrics-server/
		targetRevision: 3.8.4
		chart: metrics-server
		helm: # <-- here
			version: v1
			releaseName: my-metric
			passCredentials: false
			parameters:
				- name: "image.tag"
				  value: v0.6.2
			values: |
			defaultArgs:
			- --cert-dir=/tmp
			- --kubelet-preferred-adress-types=InteralIP,ExternalIP,Hostname
			- --kubelet-use-node-status-port
			- --metric-resolution=15s
			- --kubelet-insecure-tls # <-- required for minikube
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

Passing individual parameters can be difficult to read when you start to get complex. It's better to provide a yaml file containing the values. 

Apply this resource
```
kubectl apply -f 4-example/application.yaml
```

Verify the metrics server is running
```
kubectl top pods -n kube-system
```

Now we're going to deploy using [[kustomize]]. It helps keep code [[DRY]] and allows you to override variables specific to the environment.

Create a new folder for the base configuration: 
```
mkdir my-app-base
```

Create the namespace
```yaml
# /my-app-base/namespace.yaml
---
apiVersion: v1
kind: Namespace
metadata:
	name: default # <--- Will be overwritten in each environment
```

Create the deployment
```yaml
# /my-app-base/deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
	name: nginx
spec:
	replicas: 1
	selector: 
		matchLabels:
			app: nginx
	template:
		metadata:
			labels:
				app: nginx
		spec:
			containers:
			- name: nginx
			  imagePullPolicy: Always # <-- here
			  image: tomferguson3pl/nginx # <-- no tag
			  port:
				- containerPort: 80

```

There is no label in the deployment metadata, there is no tag on the image and there is a new field for `imagePullPolicy: Always`

You also need to include a kustomization.yaml file here too

```yaml
# /my-app-base/kustomization.yaml
---
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
	name: arbitrary
resources:
	- deployment.yaml
	- namespace.yaml
```

Now for the environments, we're going to make a new folder structure:
```yaml
# ./environments/staging/my-app/kustomization.yaml
---
namespace: staging
images:
  - name: tomferguson3pl/nginx
    newTag: v0.1.0
resources:
	- ../../../my-app-base
```

So here we only have to define the overrides we're going to make for this environment. The namespace and images. We also need to set where the base resrouces can be found for the environment. 

Commit and push these changes to the repo.

Now create a new folder with the application resource
```yaml
# ./5-example/git-repo-secret.yaml
---
apiVersion:  v1
kind: Secret
metadata:
	name: my-app-private
	namespace: argocd
	labels:
		argocd.argoproj.io/secret-type: repository
	stringData:
		url: git@github.com:3pl-tferg-devops/my-app-private.git
		sshPrivateKey: |
			<paste-your-private-key-here>
		insecure: "false"
		enableLfs: true
# ./5-example/application.yaml
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
	name: my-app-private
	namespace: argocd
	finalizers:
		- resources-finalizer.argocd.argoproj.io
spec:
	project: default
	source:
		repoURL: git@github.com:3pl-tferg-devops/my-app-private.git
		targetRevision: HEAD
		path: environments/staging/my-app-private
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

From the application perspective, it looks similar to how we deploy plain yaml files. You could kustomize some of the parts from here, but it's recommended to use the GitOps repo instead. 

Then apply the changes: 
```
kubectl apply -f 5-example/application.yaml
```