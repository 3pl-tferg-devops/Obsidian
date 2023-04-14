/etc/unbound/unbound.conf.d/pi-hole.conf

```
server:
    logfile: "/var/log/unbound/unbound.log"
    verbosity: 0
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    do-ip6: no 
    prefer-ip6: no
    harden-glue: yes
    harden-dnssec-stripped: yes
    use-caps-for-id: no
    edns-buffer-size: 1232
    prefetch: yes
    num-threads: 1
    so-rcvbuf: 1m
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```

[[unbound-conf-do-ip6]]
[[unbound-conf-prefer-ip6]]
[[unbound-conf-root-hints]]
[[unbound-conf-harden-glue]]
[[unbound-conf-harden-dnssec-stripped]]
[[unbound-conf-use-caps-for-id]]
[[unbound-conf-edns-buffer-size]]
[[unbound-config-prefetch]]
[[unbound-conf-num-threads]]
[[unbound-conf-so-rcvbuf]]
[[unbound-conf-private-address]]

[[unbound-conf-disable-resolve-conf]]

[[unbound-service-reset]]
