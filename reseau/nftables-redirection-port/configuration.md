# nftables - redirection d'un port

* Rediriger les connexions externes aux ports 443 et 8443 vers les ports 8080 et 8081.

* Ouvrir le port 22 pour les adresses privées et de liaisons locales autoconfigurées.

* Ouvrir les port udp 67-68 pour l'interface pan

```bash
nft flush ruleset

#nft delete table inet filter
#nft delete table ip nat
#nft delete table ip mangle

nft add table inet filter

nft add chain inet filter INPUT { type filter hook input priority 0\; policy drop\; }
nft add chain inet filter FORWARD { type filter hook forward priority 0\; policy drop\; }
nft add chain inet filter OUTPUT { type filter hook output priority 0\; policy accept\; }

nft add rule inet filter INPUT iifname "lo" counter accept
nft add rule inet filter INPUT counter
nft add rule inet filter INPUT ct state invalid counter drop
nft add rule inet filter INPUT ct state related,established counter accept
nft add rule inet filter INPUT ip saddr { 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 169.254.0.0/16 } tcp dport 22 counter accept
nft add rule inet filter INPUT iifname "pan" udp dport 67-68 counter accept
nft add rule inet filter INPUT mark != 0x1 tcp dport { 8080, 8081 } counter accept
nft add rule inet filter INPUT ip protocol icmp counter accept
nft add rule inet filter INPUT limit rate over 5/minute burst 5 packets counter log prefix \"inettables paquet rejeté: \" level debug
nft add rule inet filter INPUT counter reject

nft add rule inet filter FORWARD counter reject

nft add rule inet filter OUTPUT counter
nft add rule inet filter OUTPUT ip protocol icmp counter accept

nft add table ip nat

nft add chain ip nat PREROUTING '{ type nat hook prerouting priority -100; policy accept; }'
nft add chain ip nat INPUT { type nat hook input priority 100\; policy accept\; }
nft add chain ip nat POSTROUTING { type nat hook postrouting priority 100\; policy accept\; }
nft add chain ip nat OUTPUT '{ type nat hook output priority -100; policy accept; }'

nft add rule ip nat PREROUTING tcp dport 443 counter redirect to :8080
nft add rule ip nat PREROUTING tcp dport 8443 counter redirect to :8081

nft add table ip mangle

nft add chain ip mangle PREROUTING '{ type filter hook prerouting priority -150; policy accept; }'
nft add chain ip mangle INPUT '{ type filter hook input priority -150; policy accept; }'
nft add chain ip mangle FORWARD '{ type filter hook forward priority -150; policy accept; }'
nft add chain ip mangle OUTPUT '{ type route hook output priority -150; policy accept; }'
nft add chain ip mangle POSTROUTING '{ type filter hook postrouting priority -150; policy accept; }'
nft add rule ip mangle PREROUTING tcp dport { 8080, 8443 } counter meta mark set 0x1

nft list ruleset

#nft list ruleset > /etc/nftables.conf

#systemctl restart nftables
```