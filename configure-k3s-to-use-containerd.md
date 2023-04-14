Edit the k3s configuration file `/etc/rancher/k3s/config.yaml` and set the `kubelet-config` option to the following

```sh
kubelet-config:
  "container-runtime": "containerd"
```

Restart the k3s service

```sh
sudo systemctl restart k3s
```

Verify k3s is using containerd

```sh
sudo journalctl -u k3s | grep kubelet | grep containerd
```

[[install-k3s-on-raspberry-pi]]

