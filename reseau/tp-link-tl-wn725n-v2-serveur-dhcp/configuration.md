# TP-LINK TL-WN725N v2 et serveur DHCP (raspbian buster)

Utiliser l'adaptateur wifi TP-LINK TL-WN725N v2 ([Realtek RTL8188EUS](https://www.realtek.com/en/products/communications-network-ics/item/rtl8188eus)) avec raspbian buster et un Raspberry Pi 1 Model B. Cette carte peut être configurée à l'aide de network-manager sans installer de pilotes supplémentaires.

Cette solution est une alternative à la [solution proposée](https://www.raspberrypi.org/forums/viewtopic.php?f=28&t=62371) sur le forum raspbian.

## Interface réseau sans fil

Installer network-manager.
```bash
apt-get install network-manager
```

Scanner les réseaux disponibles.
```bash
nmcli device wifi list
```

Se connecter au réseau wifi.
```bash
nmcli device wifi connect <monSsid> password <monMotDePasse>
```

Activer le service.
```bash
systemctl enable network-manager
systemctl disable dhcpcd
```

## Interface filaire du serveur DHCP

Déterminer l'identifiant de l'interface "eth0". Le nom est dans la colonne "CONNECTION".
```bash
nmcli dev status
```

Configurer l'adresse IP statique de l'interface "eth0".
```bash
nmcli con mod "Connexion filaire 1" ipv4.method manual
nmcli con mod "Connexion filaire 1" ipv4.addresses 192.168.4.1/24
```

## Extinction automatique

```bash
# /etc/NetworkManager/dispatcher.d/10-extinction-automatique
#
# #!/bin/sh
# export LC_ALL=C
# interface="$1"
# etat="$2"
# echo "Route '${interface}' '${etat}' "
# secondesDepuisDemarrage=$(cat /proc/uptime | cut -d' ' -f1 | cut -d'.' -f1)
# case $interface in
#      eth0 | wlan0)
#      if [ $etat = 'down' ] && [ $secondesDepuisDemarrage -gt "300" ]
#      then     
#          shutdown -h +1
#          echo "Extinction automatique: interface '${interface}' raison '${etat}' '${secondesDepuisDemarrage}' secondes depuis le démarrage "
#      fi
#      
#      if [ $etat = 'up' ]
#      then
#           shutdown -c
#           echo "Extinction automatique annulée : interface '${interface}' raison '${etat}'"
#      fi
#;;
#esac

adresse_fichier=/etc/NetworkManager/dispatcher.d/10-extinction-automatique
echo '#!/bin/sh' > $adresse_fichier
echo 'export LC_ALL=C' >> $adresse_fichier
echo 'interface="$1"' >> $adresse_fichier
echo 'etat="$2"' >> $adresse_fichier
echo 'echo "Route '\''${interface}'\'' '\''${etat}'\'' "' >> $adresse_fichier
echo 'secondesDepuisDemarrage=$(cat /proc/uptime | cut -d'\'' '\'' -f1 | cut -d'\''.'\'' -f1)' >> $adresse_fichier
echo 'case $interface in' >> $adresse_fichier
echo '      eth0 | wlan0)' >> $adresse_fichier
echo '      if [ $etat = '\''down'\'' ] && [ $secondesDepuisDemarrage -gt "300" ]'  >> $adresse_fichier
echo '      then     '  >> $adresse_fichier
echo '          shutdown -h +1'  >> $adresse_fichier
echo '          echo "Extinction automatique: interface '\''${interface}'\'' raison '\''${etat}'\'' '\''${secondesDepuisDemarrage}'\'' secondes depuis le démarrage "'  >> $adresse_fichier
echo '      fi' >> $adresse_fichier
echo '      '  >> $adresse_fichier
echo '      if [ $etat = '\''up'\'' ]' >> $adresse_fichier
echo '      then' >> $adresse_fichier
echo '           shutdown -c' >> $adresse_fichier
echo '           echo "Extinction automatique annulée : interface '\''${interface}'\'' raison '\''${etat}'\''"' >> $adresse_fichier
echo '      fi' >> $adresse_fichier
echo '	;;' >> $adresse_fichier
echo 'esac' >> $adresse_fichier
chmod +x "$adresse_fichier"
```


## Serveur DHCP utilisant l'interface eth0 et [Dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html)

Activer le plugin dnsMasq.
```bash
# /etc/NetworkManager/conf.d/00-activer-dnsmasq.conf
# 
# [main]
# dns=dnsmasq
#
adresse_fichier=/etc/NetworkManager/conf.d/00-activer-dnsmasq.conf
echo  [main]> $adresse_fichier
echo dns=dnsmasq >> $adresse_fichier
```

Ajouter un fichier "/etc/NetworkManager/dnsmasq.d/configuration.conf".
```bash
# /etc/NetworkManager/dnsmasq.d/configuration.conf
# 
# interface=eth0
# dhcp-range=192.168.4.10,192.168.4.254,255.255.255.0,24h
# domain=reseau.local
# server=1.1.1.1
#
adresse_fichier=/etc/NetworkManager/dnsmasq.d/configuration.conf
echo interface=eth0 > $adresse_fichier
echo dhcp-range=192.168.4.10,192.168.4.254,255.255.255.0,24h >> $adresse_fichier
echo domain=reseau.local >> $adresse_fichier
echo server=1.1.1.1 >> $adresse_fichier
```

Désinstaller dnsmasq et redémarrer network-manager.
```bash
apt-get remove dnsmasq
systemctl restart NetworkManager
```

Vérifier que le plugin dnsmasq est démarré, utilise le nouveau fichier de configuration et qu'aucune erreur n'est journalisée.
```bash
ps aux | grep dnsmasq
#/usr/sbin/dnsmasq [...] --conf-dir=/etc/NetworkManager/dnsmasq.d [...]
cat /var/log/syslog | grep dnsmasq
#dnsmasq[880]: demarré, version 2.80 (taille de cache 400)
```

Activer le routage IP.
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

## Configuration du pare-feu

```bash
#Réinitialiser la configuration iptables
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
nft add rule inet filter INPUT iifname "eth0" tcp dport 22 counter drop
nft add rule inet filter INPUT ip saddr @adresses_locales_ipv4 tcp dport 22 counter limit rate 1/minute accept
nft add rule inet filter INPUT iifname "eth0" udp dport { 53, 67 } counter accept
nft add rule inet filter INPUT ip saddr @adresses_locales_ipv4 ip protocol icmp counter limit rate 5/minute accept
nft add rule inet filter INPUT ip6 saddr @adresses_locales_ipv6 limit rate 5/minute ip6 nexthdr icmpv6 counter accept
nft add rule inet filter INPUT ip6 daddr @adresses_multicast_ipv6 limit rate 5/minute icmpv6 type { nd-neighbor-solicit, nd-router-advert, nd-neighbor-advert } accept
# nft add rule inet filter INPUT tcp dport 22 nftrace set 1
nft add rule inet filter INPUT ip6 saddr \& ffff\:ffff\:ffff\:ffff\:\: != @sous_reseaux_autorises_ipv6 ip6 daddr @adresses_unicast_ipv6 limit rate 1/minute add @sous_reseaux_autorises_ipv6 { ip6 daddr \& ffff\:ffff\:ffff\:ffff\:\:} counter
nft add rule inet filter INPUT ip6 saddr \& ffff\:ffff\:ffff\:ffff\:\: @sous_reseaux_autorises_ipv6 ip6 nexthdr ipv6-icmp limit rate 5/minute counter accept
nft add rule inet filter INPUT ip6 saddr \& ffff\:ffff\:ffff\:ffff\:\: @sous_reseaux_autorises_ipv6 tcp dport 22 limit rate 1/minute counter accept
nft add rule inet filter INPUT limit rate over 6/minute counter reject
nft add rule inet filter INPUT limit rate over 5/minute counter log prefix \"inettables paquet rejeté: \" level debug
nft add rule inet filter INPUT counter reject

nft add rule inet filter OUTPUT counter

nft add rule inet filter FORWARD ct state related,established counter accept
nft add rule inet filter FORWARD iifname eth0 oifname wlan0 counter accept

nft add chain ip nat prerouting { type nat hook prerouting priority 0 \; }
nft add chain ip nat postrouting { type nat hook postrouting priority 100 \; }
nft add rule ip nat postrouting oifname wlan0 counter masquerade

nft list ruleset > /etc/nftables.conf
systemctl enable nftables
nft list ruleset
```