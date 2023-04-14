We'll use this as our registry for now, later we'll figure out how to use ECR #aws #ecr

Go to https://hub.docker.com

A free docker hub account allows you to store a single private image and an unlimited amount of free images.

To authenticate [[docker]] with dockerhub you need to run
```
docker login --username tomferguson3pl
```

Then to download an image do
```
docker pull nginx:1.23.3
```

To simulate CD pipeline, we would need to increment image tags to deploy new versions. Run the below to tag the open-source image using your personal username from the dockerhub account
```
docker tag nginx:1.23.3 tomferguson3pl/nginx:v0.1.0
```

You'll notice if you run docker images that you'll find the two images have the same ids
```
❯ docker images
REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE
nginx                         1.23.3    25d0f5e5d6da   6 days ago     135MB
tomferguson3pl/nginx          v0.1.0    25d0f5e5d6da   6 days ago     135MB
```

Use the docker push command to upload the image to your docker hub account:
```
docker push tomferguson3pl/nginx:v0.1.0
```

If you go to your docker hub profile, you should see that you have a repository now with a single image and the tag of `v0.1.0`

Now we need to make a new [[kubernetes]] deployment for that new docker image. 