```sh
kubectl get secrets argocd-initial-admin-secret -o yaml

apiVersion: v1
data:
  password: aktmdGVXU3V3RlRjcnpDRA== # <-- here
kind: Secret
metadata:
  creationTimestamp: "2023-04-02T04:48:17Z"
  name: argocd-initial-admin-secret
  namespace: argocd
  resourceVersion: "84727"
  uid: ade99cf7-0c28-4c32-94b7-c604e9386f90
type: Opaque
```

[[how-to-decode-a-base-64-encoded-string]]

