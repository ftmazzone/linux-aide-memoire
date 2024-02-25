# Configuration Raspberry Pi

## Configurer l'utilisateur

```bash
mkdir -p "/tmp/rpi_boot"
read -p "Saisissez le nom de l'utilisateur: " nom_utilisateur
echo "$nom_utilisateur:$(openssl passwd -6)" > /tmp/rpi_boot/userconf.txt
```

## Configurer ssh

```bash
mkdir -p "/tmp/rpi_boot"
echo "" > /tmp/rpi_boot/ssh
```

## Configurer le réseau sans fil

```bash
mkdir -p "/tmp/rpi_boot"
read -p "Saisissez le pays du réseau sans fil: " pays_reseau_sans_fil
read -p "Saisissez le nom du réseau sans fil: " nom_reseau_sans_fil
read -s -p "Saisissez le mot de passe du réseau sans fil: " mot_de_passe_reseau_sans_fil

cat > /tmp/rpi_boot/wpa_supplicant.conf <<EOL
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
country=$pays_reseau_sans_fil
update_config=1

network={
 ssid="${nom_reseau_sans_fil}"
 psk="${mot_de_passe_reseau_sans_fil}"
}
EOL
```

## Configurer le réseau

```bash
#Réinitialiser la configuration iptables
#apt-get install nftables
nft flush ruleset

nft add table inet filter

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

nft list ruleset > /etc/nftables.conf

systemctl restart nftables

nft list ruleset
```

## Désactiver l'espace d'échange (swap)

```bash
systemctl disable dphys-swapfile.service
```