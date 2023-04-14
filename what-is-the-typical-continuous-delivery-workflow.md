Typically you would seperate your application code and your infrastructure code into seperate repositories. 

When you make changes to the application code, something like Jenkins would build a new Docker image and upload it to your registry like Dockerhub. 

This would create a new image tag and Jenkins would then push that change to the infrastructure repo. 

ArgoCD polls the infrastructure repo for changes and when it notices a change, it downloads and applys the changes to the Kubernetes cluster which will handle the rest of the deployment. 

This process is known as continuous delivery, where every change to the source code results in a deployment. 

Typically automated deployments would occur in development and staging environments but production would still require manual intervention before deploying any changes. 