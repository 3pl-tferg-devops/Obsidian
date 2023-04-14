```
echo "# argocd" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/3pl-tferg-devops/argocd.git
git push -u origin main
```

Working with a private github repo in ArgoCD

There are many ways to authenticate to a private github repository, personal access tokens, github apps, but for this we're going to use SSH keys.

To generate the SSH key run:
Elliptic curve
```
ssh-keygen -t ed25519 -C "argocd" -f ~/.ssh/argocd_ed25519
```
RSA
```
ssh-keygen -t rsa -b 4096 -C "argocd" -f ~/.ssh/argocd_rsa4096
```

Upload the public key to the github repository
```
cat ~/.ssh/argocd_ed25519
```

Add the deploy key to the repository
```
gh api repos/3pl-tferg-devops/argocd/keys -f title=argocd -f key="<paste-your-public-key-here" -F read_only=true
```