Debian Bullseye+ releases auto-install a package called [`openresolv`](https://wiki.archlinux.org/title/Openresolv) with a certain configuration that will cause unexpected behaviour for pihole and unbound.

The effect is that the `unbound-resolvconf.service` instructs `resolvconf` to write `unbound`'s own DNS service at `nameserver 127.0.0.1` , but without the 5335 port, into the file `/etc/resolv.conf`

That `/etc/resolv.conf` file is used by local services/processes to determine DNS servers configured. You need to edit the configuration file and disable the service to work-around the misconfiguration.

Check if the service is active:
```sh
sudo systemctl status unbound-resolvconf.service
```

Disable the service:
```sh
sudo systemctl disable --now unbound-resolvconf.service
```

Disable the file resolvconf_resolvers.conf
```sh
sudo sed -Ei 's/^unbound_conf=/#unbound_conf=/' /etc/resolvconf.conf
sudo rm /etc/unbound/unbound.conf.d/resolvconf_resolvers.conf
```