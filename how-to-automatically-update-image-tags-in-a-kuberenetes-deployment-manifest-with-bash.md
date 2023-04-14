```sh
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

[[how-to-make-a-script-executable-with-bash]]

[[how-to-execute-the-upgrade-sh-script]]

