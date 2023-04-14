```yaml
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

[[how-to-deploy-a-kubernetes-manifest]]

