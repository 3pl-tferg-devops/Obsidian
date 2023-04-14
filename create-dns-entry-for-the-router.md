[[create-dns-entry-for-the-raspberry-pi]]

Instead of the IP address of the raspberry-pi use the one for the router
```sh
sudo vim /etc/hosts
...
192.168.20.1 router.local
```

Then flush the dns cache

```sh
sudo dscacheutil -flushcache
```

