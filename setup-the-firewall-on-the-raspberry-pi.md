The preinstalled firewall software on Ubuntu server is `ufw`. 

Check the status of `ufw` service with `systemctl`
```sh
sudo systemctl status ufw
```

Check the status of `ufw`
```sh
sudo ufw status
```

Allow ssh connections
```sh
sudo ufw allow ssh
```

Enable the firewall
```sh
sudo ufw enable 
```

[[enable-ports-for-pihole-on-the-raspberry-pi]]

