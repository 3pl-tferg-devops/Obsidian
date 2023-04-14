

Login as root

```
su -
```

Then run

```sh
sudo echo 'cgroup_enable=memory cgroup_memory=1' >> /boot/firmware/cmdline.txt
```

