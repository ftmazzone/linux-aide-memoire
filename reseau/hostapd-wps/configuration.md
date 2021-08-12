# hostapd - wps

## Pour activer le protocole [WPS](https://fr.wikipedia.org/wiki/Wi-Fi_Protected_Setup) pour l'application [hostapd](http://w1.fi/hostapd/) :

* créer un fichier "hostapd.psk" vide dans le répertoire "/etc/hostapd"

```bash
touch /etc/hostapd/hostapd.psk
```

* vérifier que WPS est configuré dans "/etc/hostapd/hostapd.conf"

```ini
# /etc/hostapd/hostapd.conf

# interface=wlan0
# driver=nl80211
# ssid=monssid
# hw_mode=g
# ieee80211n=1
# channel=4
# wmm_enabled=0
# macaddr_acl=0
# auth_algs=1
# ignore_broadcast_ssid=0
# wpa=2
# wpa_passphrase=monmotdepasse
# wpa_key_mgmt=WPA-PSK
# wpa_pairwise=TKIP
# rsn_pairwise=CCMP
# wpa_psk_file=/etc/hostapd/hostapd.psk
# ctrl_interface=/var/run/hostapd
# eap_server=1
# wps_state=2
# ap_setup_locked=1
# wps_pin_requests=/var/run/hostapd.pin-req
# config_methods=label display push_button keypad

cat > /etc/hostapd/hostapd.conf <<EOL
interface=wlan0
driver=nl80211
ssid=monssid
hw_mode=g
ieee80211n=1
channel=4
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=monmotdepasse
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
wpa_psk_file=/etc/hostapd/hostapd.psk
ctrl_interface=/var/run/hostapd
eap_server=1
wps_state=2
ap_setup_locked=1
wps_pin_requests=/var/run/hostapd.pin-req
config_methods=label display push_button keypad
EOL
```
* arrêter le service hostapd

```bash
systemctl stop hostapd
```

* démarrer manuellement le service hostapd

```bash
hostapd -d /etc/hostapd/hostapd.conf
```

* activer WPS à l'aide de la commande

```bash
hostapd_cli wps_pbc
```

* connecter le client au point d'accès sans fil

* configurer nftables

```bash
#Réinitialiser la configuration iptables
#apt-get install nftables

nft flush ruleset
nft add table inet filter
nft add table ip nat

nft add chain inet filter INPUT { type filter hook input priority 0\; policy drop\; }
nft add chain inet filter OUTPUT { type filter hook output priority 0\; policy accept\; }
nft add chain inet filter FORWARD { type filter hook forward priority 0\; policy drop\; }

nft add set inet filter adresses_locales_ipv4 { type ipv4_addr\;flags interval\; elements={10.0.0.0/8, 169.254.0.0/16, 172.16.0.0/12, 192.168.0.0/16} \; }

nft add rule inet filter INPUT iifname "lo" counter accept
nft add rule inet filter INPUT counter
nft add rule inet filter INPUT ct state invalid counter drop
nft add rule inet filter INPUT ct state related,established counter accept
nft add rule inet filter INPUT iifname {eth0 , eth1} ip saddr @adresses_locales_ipv4 tcp dport 22 counter accept
nft add rule inet filter INPUT iifname wlan0 udp dport { 53, 67 } counter accept
nft add rule inet filter INPUT ip saddr @adresses_locales_ipv4 ip protocol icmp counter accept
nft add rule inet filter INPUT limit rate over 5/minute burst 5 packets counter log prefix \"inettables paquet rejeté: \" level debug
nft add rule inet filter INPUT counter reject

nft add rule inet filter OUTPUT counter
nft add rule inet filter OUTPUT oifname {eth0 , eth1} ip daddr != @adresses_locales_ipv4 counter meta mark set 0x1

nft add rule inet filter FORWARD ct state related,established counter accept
nft add rule inet filter FORWARD iifname wlan0 ip daddr != @adresses_locales_ipv4 oifname { eth0, eth1 } counter accept
nft add rule inet filter FORWARD iifname {eth0, eth1} ip saddr @adresses_locales_ipv4 oifname {eth0, eth1 } ip daddr @adresses_locales_ipv4 counter accept
nft add rule inet filter FORWARD counter reject

nft add chain ip nat prerouting '{ type filter hook prerouting priority -150; policy accept; }'
nft add rule ip nat prerouting iifname wlan0 counter meta mark set 0x2
nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }
nft add rule ip nat postrouting oifname {eth0 , eth1} counter masquerade

nft list ruleset > /etc/nftables.conf

systemctl restart nftables

nft list ruleset
```

* Configurer [Traffic Control](https://wiki.debian.org/TrafficControl) pour la gestion du traffic du point d'accès.

```bash
tc qdisc del dev eth0 root handle 1: htb
tc filter del dev eth0
tc qdisc add dev eth0 root handle 1: htb default 1
tc class add dev eth0 parent 1: classid 1:1 htb rate 100mbit ceil 100mbit
tc class add dev eth0 parent 1: classid 1:2 htb rate 1.8mbit
tc class add dev eth0 parent 1:2 classid 1:3 htb rate 0.5mbit ceil 1.8mbit
tc class add dev eth0 parent 1:2 classid 1:4 htb rate 0.5mbit ceil 1.5mbit
tc qdisc add dev eth0 parent 1:4 handle 20: sfq perturb 10
tc filter add dev eth0 protocol ip parent 1: prio 0 handle 1 fw flowid 1:3
tc filter add dev eth0 protocol ip parent 1: prio 0 handle 2 fw flowid 1:4
tc -s qdisc show dev eth0
tc -s -g class show dev eth0
tc -s filter show dev eth0
```

* Activer le routage IP.

```bash
# /etc/sysctl.d/routage-point-acces.conf
#
# net.ipv4.ip_forward=1
# net.ipv6.conf.default.forwarding=1
# net.ipv6.conf.all.forwarding=1
#
adresse_fichier=/etc/sysctl.d/routage-point-acces.conf
echo net.ipv4.ip_forward=1 > $adresse_fichier
echo net.ipv6.conf.default.forwarding=1 >> $adresse_fichier
echo net.ipv6.conf.all.forwarding=1 >> $adresse_fichier
```

## Sources:
* [Setting up a Raspberry Pi as a Wireless Access Point](https://www.raspberrypi.org/documentation/computers/configuration.html#setting-up-a-routed-wireless-access-point)
* [How to configure hostapd.conf for wps push button?](https://unix.stackexchange.com/questions/372902/how-to-configure-hostapd-conf-for-wps-push-button)
* [QoS et gestion du trafic avec Traffic Control](https://connect.ed-diamond.com/GNU-Linux-Magazine/GLMF-127/QoS-et-gestion-du-trafic-avec-Traffic-Control)