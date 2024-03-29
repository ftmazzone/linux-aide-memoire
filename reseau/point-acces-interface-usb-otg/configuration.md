# Point d'accès internet port usb OTG (raspbian bullseye)

Configurer un point d'accès réseau avec l'interface usb0 en utilisant un raspberry zero. Le client dhcp se connecte aux autres réseaux par l'intermédiaire de l'interface USB OTG. L'interface wlan0 est connectée aux autres réseaux.

Un serveur DHCP attribue une adresse au clients se connectant aux interface usb0 (192.168.5.1/24).

## Configurer le port usb OTG (Raspberry Zero)

```bash
# /boot/config.txt
# dtoverlay=dwc2
#
adresse_fichier=/boot/config.txt
echo dtoverlay=dwc2 >> $adresse_fichier
```

```bash
# /boot/cmdline.txt
# Après "rootwait", ajouter "modules-load=dwc2,g_ether"
#
adresse_fichier=/boot/cmdline.txt
echo -n modules-load=dwc2,g_ether >> $adresse_fichier
```

## Installer le serveur DHCP et le daemon pour le point d'accès réseau sans fil.

```bash
apt-get install dnsmasq
```

## Configurer le serveur DHCP pour l'interface usb0

__Remarque__ Si "NetworkManager" est utilisé à la place de "dhcpcd", le plugin DnsMasq peut être utilisé : il faut alors ajouter le fichier de configuration dans le répertoire "/etc/NetworkManager/dnsmasq.d/" par exemple "/etc/NetworkManager/dnsmasq.d/00-mon-domaine.conf". 

* Attribuer une adresse statique à l'interface dans le fichier "/etc/dhcpcd.conf"

```ini
# /etc/dhcpcd.conf
#
# interface usb0
#   static ip_address=192.168.5.1/24
#
adresse_fichier=/etc/dhcpcd.conf
echo interface usb0 >> $adresse_fichier
echo -e "\tstatic ip_address=192.168.5.1/24" >> $adresse_fichier
```

* Rajouter un sous-réseau dans le fichier "/etc/dnsmasq.conf"

```ini
# /etc/dnsmasq.conf
#
# bind-interfaces
# interface=usb0

# domain-needed
# bogus-priv
# addn-hosts=/etc/dnsmasq_hosts
# no-hosts
# expand-hosts
# server=/mon-domaine/192.168.5.1

# domain=mon-domaine
# dhcp-authoritative
# listen-address=::1,127.0.0.1,192.168.5.1
# dhcp-range=192.168.5.10,192.168.5.100,255.255.255.0,24h
# dhcp-option=3,192.168.5.1
# dhcp-option=6,192.168.5.1
#
cat > /etc/dnsmasq.conf <<EOL
#bind-interfaces
interface=usb0

domain-needed
bogus-priv
addn-hosts=/etc/dnsmasq_hosts
no-hosts
expand-hosts
server=/mon-domaine/192.168.5.1

domain=mon-domaine
dhcp-authoritative
listen-address=::1,127.0.0.1,192.168.5.1
dhcp-range=192.168.5.10,192.168.5.100,255.255.255.0,24h
dhcp-option=3,192.168.5.1
dhcp-option=6,192.168.5.1
EOL
```

* Configurer le nom de domaine local

```bash
# /etc/dnsmasq_hosts
# 192.168.5.1	 <noms d'hôte>
#
echo -e "192.168.5.1\t" $(hostname) $(hostname).mon-domaine > /etc/dnsmasq_hosts
```

## Configurer la redirection et les règles NAT dynamiques

* Activer le routage IP.

```ini
# /etc/sysctl.d/routage-point-acces.conf
#
# net.ipv4.ip_forward=1
# net.ipv6.conf.default.forwarding=1
# net.ipv6.conf.all.forwarding=1
#
cat > /etc/sysctl.d/routage-point-acces.conf <<EOL
net.ipv4.ip_forward=1
net.ipv6.conf.default.forwarding=1
net.ipv6.conf.all.forwarding=1
EOL
```

* Configurer nftables

```bash
#Réinitialiser la configuration iptables
#apt-get install nftables
nft flush ruleset

nft add table inet filter
nft add table ip nat

nft add chain inet filter INPUT { type filter hook input priority 0\; policy drop\; }
nft add chain inet filter OUTPUT { type filter hook output priority 0\; policy accept\; }
nft add chain inet filter FORWARD { type filter hook forward priority 0\; policy drop\; }

nft add rule inet filter INPUT iifname "lo" counter accept
nft add rule inet filter INPUT counter
nft add rule inet filter INPUT ct state invalid counter drop
nft add rule inet filter INPUT ct state related,established counter accept
nft add rule inet filter INPUT ip saddr != { 10.0.0.0/8, 169.254.0.0/16, 172.16.0.0/12, 192.168.0.0/16 } tcp dport 22 limit rate over 1/minute counter drop
nft add rule inet filter INPUT tcp dport 22 counter accept
nft add rule inet filter INPUT iifname "usb0" udp dport { 53, 67 } counter accept
# Accepter le traffic entrant qui a été redirigé vers le port 80 du serveur DHCP
# nft add rule inet filter INPUT iifname "usb0" tcp dport 8080 counter accept
nft add rule inet filter INPUT ip protocol icmp counter accept
nft add rule inet filter INPUT limit rate over 5/minute burst 5 packets counter log prefix \"inettables paquet rejeté: \" level debug
nft add rule inet filter INPUT counter reject

nft add rule inet filter OUTPUT counter

nft add rule inet filter FORWARD ct state related,established counter accept
nft add rule inet filter FORWARD iifname usb0 oifname wlan0 counter accept

nft add chain ip nat prerouting { type nat hook prerouting priority 0 \; }
# Rediriger tout le traffic entrant sur l'interface usb0 et le port 80 vers le port 8080 du serveur DHCP
# nft add rule nat prerouting iif usb0 tcp dport 80 counter dnat to 192.168.5.1:8080
# Refuser le traffic entrant vers le port 8080 
# nft add rule nat prerouting iif usb0 tcp dport 80 counter drop

nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }
nft add rule ip nat postrouting oifname wlan0 counter masquerade

nft list ruleset > /etc/nftables.conf

systemctl restart nftables

nft list ruleset
```

## Configurer le serveur ssh

* N'autoriser l'authentification par mot de passe que pour les clients disposant d'une adresse locale

```ini
# /etc/ssh/sshd_config
#
# PasswordAuthentication no
# # A la fin du fichier, pour n'autoriser l'authentification par mot de passe que pour les clients disposant d'une adresse locale, ajouter :
# Match Address 10.0.0.0/8,169.254.0.0/16,172.16.0.0/12,192.168.0.0/16
#    PasswordAuthentication yes
#
cat > /etc/ssh/sshd_config <<EOL
PasswordAuthentication no
# A la fin du fichier, pour n'autoriser l'authentification par mot de passe que pour les clients disposant d'une adresse locale, ajouter :
Match Address 10.0.0.0/8,169.254.0.0/16,172.16.0.0/12,192.168.0.0/16
    PasswordAuthentication yes
EOL
```

## A partir du client dhcp connecté à l'interface usb0, vérifier que la connexion est activée

```bash
traceroute 1.1.1.1
```
