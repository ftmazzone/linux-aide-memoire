# Bluetooth pan network - Raspbian (stretch)

## Configuration de l'interface réseau

Installer les outils bluetooth.

```bash
apt-get install bluez-tools
```

Sauvegarder les fichiers suivants :

```ini
# /etc/systemd/network/pan.netdev
#
# [NetDev]
# Name=pan
# Kind=bridge
# ForwardDelaySec=0

cat > /etc/systemd/network/pan.netdev <<EOL
[NetDev]
Name=pan
Kind=bridge
ForwardDelaySec=0
EOL
```

```bash
# /etc/systemd/network/pan.network
#
# [Match] Name=pan  [Network] Address=172.20.1.1/24 DHCPServer=yes

echo [Match] Name=pan  [Network] Address=172.20.1.1/24 DHCPServer=yes > /etc/systemd/network/pan.network
```

Ouvrir les ports UDP 67 & 68 pour la négociation d'adresse IP.

```bash
iptables -I INPUT -i pan -p udp --dport 67:68 -j ACCEPT
```

Si nécessaire, ouvrir le port 22 pour se connecter au serveur ssh.

```bash
iptables -A INPUT -i pan -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i pan -p icmp -j ACCEPT
iptables -A OUTPUT -o pan -p icmp -j ACCEPT
```

## Configuration du service pan

Sauvegarder les fichiers suivants :

```ini
# /etc/systemd/system/pan.service
#
# [Unit]
# Description=Bluetooth Personal Area Network
# After=bluetooth.service systemd-networkd.service
# Requires=systemd-networkd.service
# PartOf=bluetooth.service
# [Service]
# Type=notify
# ExecStart=/usr/bin/bt-network -s nap pan
# Restart=on-failure
# RestartSec=30
# StartLimitInterval=200
# StartLimitBurst=3
# [Install]
# WantedBy=bluetooth.target

cat > /etc/systemd/system/pan.service <<EOL
[Unit]
Description=Bluetooth Personal Area Network
After=bluetooth.service systemd-networkd.service
Requires=systemd-networkd.service
PartOf=bluetooth.service
[Service]
Type=notify
ExecStart=/usr/bin/bt-network -s nap pan
Restart=on-failure
RestartSec=30
StartLimitInterval=200
StartLimitBurst=3
[Install]
WantedBy=bluetooth.target
EOL
```

Activer et démarrer le service pan.

```bash
systemctl daemon-reload
systemctl restart systemd-networkd
systemctl enable pan
systemctl start pan
```

## Connecter le client bluetooth

### Serveur

```bash
power on agent on default-agent
discoverable on trust XX:XX:XX:XX:XX:XX
discoverable off
```

### Client

Utiliser le client bluetooth (par exemple blueman) pour se connecter.

Connection à : &#8594; Point d'accès réseau

## Sources

[fgtk](https://github.com/mk-fg/fgtk/blob/master/bt-pan)

[How can I set up a bluetooth PAN connection with a Raspberry Pi and an iPod?](https://raspberrypi.stackexchange.com/questions/29504/how-can-i-set-up-a-bluetooth-pan-connection-with-a-raspberry-pi-and-an-ipod)