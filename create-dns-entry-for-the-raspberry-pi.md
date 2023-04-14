Instead of using the IP address of the host, we use the DNS address.

Edit the hosts file located here: `/etc/hosts` to have the IP address and the host name

```sh
sudo vim /etc/hosts
...
192.168.20.9 raspberry-pi.local
```

Then flush the dns cache

```sh
sudo dscacheutil -flushcache
```