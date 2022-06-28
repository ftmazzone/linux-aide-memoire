# Bluetooth pan network - Raspbian (buster)

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
#
cat > /etc/systemd/network/pan.netdev <<EOL
[NetDev]
Name=pan
Kind=bridge
EOL
```

```ini
# /etc/systemd/network/pan.network
#
# [Match]
# Name=pan
#
# [Network]
# Address=192.168.6.1/24 
# DHCPServer=yes
# DNS=1.1.1.1
#
cat > /etc/systemd/network/pan.network <<EOL
[Match]
Name=pan

[Network]
Address=192.168.6.1/24 
DHCPServer=yes
DNS=1.1.1.1
EOL
```

Ouvrir le port UDP 67 pour la négociation d'adresse IP.

```bash
#Réinitialiser la configuration iptables
#apt-get install nftables
nft flush ruleset

nft add table inet filter
nft add table ip nat

nft add set inet filter adresses_locales_ipv4 { type ipv4_addr\;flags interval\; elements={10.0.0.0/8, 169.254.0.0/16, 172.16.0.0/12, 192.168.0.0/16} \; }
nft add set inet filter adresses_locales_ipv6 { type ipv6_addr\; flags interval\; elements={fe80::/10 } \; }
nft add set inet filter adresses_multicast_ipv6 { type ipv6_addr\; flags interval\; elements={ ff00::/8 } \; }
nft add set inet filter adresses_unicast_ipv6 { type ipv6_addr\; flags interval\; elements={2000::/3} \; }
nft add	set inet filter sous_reseaux_autorises_ipv6 { type ipv6_addr\; flags dynamic, timeout\; timeout 5m\;}

nft add chain inet filter INPUT { type filter hook input priority 0\; policy drop\; }
nft add chain inet filter OUTPUT { type filter hook output priority 0\; policy accept\; }
nft add chain inet filter FORWARD { type filter hook forward priority 0\; policy drop\; }

nft add rule inet filter INPUT iifname "lo" counter accept
nft add rule inet filter INPUT counter
nft add rule inet filter INPUT ct state invalid counter drop
nft add rule inet filter INPUT ct state related,established counter accept
nft add rule inet filter INPUT ip saddr @adresses_locales_ipv4 tcp dport 22 counter limit rate 1/minute accept
nft add rule inet filter INPUT iifname pan udp dport { 67 } counter accept
nft add rule inet filter INPUT ip protocol icmp counter accept
nft add rule inet filter INPUT ip6 saddr @adresses_locales_ipv6 limit rate 5/minute ip6 nexthdr icmpv6 counter accept
nft add rule inet filter INPUT ip6 daddr @adresses_multicast_ipv6 limit rate 5/minute icmpv6 type { nd-neighbor-solicit, nd-router-advert, nd-neighbor-advert } accept
# nft add rule inet filter INPUT nftrace set 1
nft add rule inet filter INPUT ip6 saddr \& ffff\:ffff\:ffff\:ffff\:\: != @sous_reseaux_autorises_ipv6 ip6 daddr @adresses_unicast_ipv6 limit rate 1/minute add @sous_reseaux_autorises_ipv6 { ip6 daddr \& ffff\:ffff\:ffff\:ffff\:\:} counter
nft add rule inet filter INPUT ip6 saddr \& ffff\:ffff\:ffff\:ffff\:\: @sous_reseaux_autorises_ipv6 ip6 nexthdr ipv6-icmp limit rate 5/minute counter accept
nft add rule inet filter INPUT ip6 saddr \& ffff\:ffff\:ffff\:ffff\:\: @sous_reseaux_autorises_ipv6 tcp dport 22 limit rate 1/minute counter accept
nft add rule inet filter INPUT limit rate over 5/minute burst 5 packets counter log prefix \"inettables paquet rejeté: \" level debug
nft add rule inet filter INPUT counter reject

nft add rule inet filter OUTPUT counter

nft add rule inet filter FORWARD ct state related,established counter accept
nft add rule inet filter FORWARD iifname pan oifname wlan0 counter accept

nft add chain ip nat prerouting { type nat hook prerouting priority 0 \; }
nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }
nft add rule ip nat postrouting oifname wlan0 counter masquerade

nft list ruleset > /etc/nftables.conf

systemctl restart nftables

nft list ruleset
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
systemctl restart pan
```

## Connecter le client bluetooth

### Serveur

```bash
bluetoothctl
power on 
agent on 
default-agent
discoverable on 
trust XX:XX:XX:XX:XX:XX
discoverable off
```

### Client

Utiliser le client bluetooth (par exemple blueman) pour se connecter.

Connection à : &#8594; Point d'accès réseau

## Sources

[fgtk](https://github.com/mk-fg/fgtk/blob/master/bt-pan)

[How can I set up a bluetooth PAN connection with a Raspberry Pi and an iPod?](https://raspberrypi.stackexchange.com/questions/29504/how-can-i-set-up-a-bluetooth-pan-connection-with-a-raspberry-pi-and-an-ipod)
