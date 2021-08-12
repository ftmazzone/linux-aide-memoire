# iptables - redirection d' un port

Rediriger les paquets adressés depuis l'extérieur au port 443 vers le port 8080.

```bash
#Reset iptables
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

#Accept packets from 443 and 8080 (internal only)
iptables -I INPUT 1 -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -I INPUT -i pan -p udp --dport 67:68 -j ACCEPT
iptables -A INPUT -i wlan0 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i usb0 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i pan -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -m mark ! --mark 1 -p tcp --dport 8080 -j ACCEPT

#Allow ping
iptables -A INPUT -i wlan0 -p icmp -j ACCEPT
iptables -A OUTPUT -o wlan0 -p icmp -j ACCEPT
iptables -A INPUT -i usb0 -p icmp -j ACCEPT
iptables -A OUTPUT -o usb0 -p icmp -j ACCEPT
iptables -A INPUT -i pan -p icmp -j ACCEPT
iptables -A OUTPUT -o pan -p icmp -j ACCEPT

#Logging
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables paquet rejeté: " --log-level 7
iptables -A INPUT -j DROP
iptables -A FORWARD -j REJECT

#Mark external packets coming from port 8080
#Redirect packets from port 443 to port 8080
iptables -t mangle -A PREROUTING -p tcp -m tcp --dport 8080 -j MARK --set-mark 1
iptables -t nat -A PREROUTING -p tcp -m tcp --dport 443 -j REDIRECT --to-ports 8080

#Save iptables changes
iptables-save > /etc/iptables.up.rules

echo $'#!/bin/sh\n/sbin/iptables-restore < /etc/iptables.up.rules' > /etc/network/if-pre-up.d/iptables
chmod +x /etc/network/if-pre-up.d/iptables

#Check configuration
iptables -nvL
```

