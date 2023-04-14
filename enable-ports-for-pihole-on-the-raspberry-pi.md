To enable DNS for pihole you need to open port 53 tcp/udp
```sh
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
```

To enable DHCP for pihole you need to open port 67 tcp/udp
```sh
sudo ufw allow 67/tcp
sudo ufw allow 67/udp
```

To enable DHCP for pihole over IPv6 you need to open port 546:547 udp
```sh
sudo ufw allow 546:547/udp
```

To enable the Web UI for pihole you need to open port 80 tcp
```sh
sudo ufw allow 80/tcp
```

Note the above commands open ports for both IPv4 and IPv6