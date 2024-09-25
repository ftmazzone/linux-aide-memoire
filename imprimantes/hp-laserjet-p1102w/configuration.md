# Configuration de l'imprimante HP LaserJet P1102w

## Réinitialisation de l'imprimante

- Appuyer sur le bouton de mise sous tension
- Appuyer sur le bouton `annuler` puis le bouton `réseau sans fil` et maintenir les deux appuyés jusqu'à ce qu'une page de diagnostique soit imprimée

## Connection au réseau "Ad hoc" de l'imprimante avec NetworkManager

- Dans la rubrique `Network Information` de la page de diagnostique, vérifier avec la propriété `Communication Mode` que le mode du réseau est bien `Ad Hoc`
- Dans la rubrique `Network Information` de la page de diagnostique, déterminer avec la propriété `Network Name (SSID)` le nom du réseau Ad hoc 
- Utiliser la configuration suivante pour configurer la connexion avec `NetworkManager`

```bash
# /etc/NetworkManager/system-connections/HP012345.nmconnection
[connection]
id=HP012345 # Remplacer le nom du réseau
uuid=11111111-2222-3333-4444-555555555555
type=wifi
interface-name=wlan0

[wifi]
mode=adhoc
ssid=HP012345 # Remplacer le nom du réseau

[ipv4]
method=link-local

[ipv6]
addr-gen-mode=stable-privacy
method=disabled

[proxy]
```

- En cas d'erreur, consulter les journaux des services réseaux
```bash
journalctl -f -u wpa_supplicant -u NetworkManager -u systemd-networkd
```

## Ouvir la page d'administration de l'imprimante en utilisant le réseau Ad hoc de l'imprimante

Il y a au moins deux solutions pour déterminer l'adresse IP de l'interface d'administration de l'imprimante.

### Solution 1 : impression d'une page de diagnostique

- Appuyer sur le bouton `annuler` et continuer à le maintenir appuyé jusqu'à ce qu'une page de diagnostique soit imprimée
- Dans la rubrique `Network Information` de la page de diagnostique, lire l'adresse de la propriété `IPv4 Address`
- Se connecter à l'interface d'administration avec un navigateur avec l'adresse `http://[IPv4 adresse lue précédemment]`

### Solution 2 : déterminer l'adresse avec ping

```bash
for ip3 in {0..255} 
do
	for ip4 in {1..254} 
	do
	    ping -c 1 -W 0.1 169.254.$ip3.$ip4 | grep "bytes from"
	done
done
```

## Raspberry Pi

La configuration du réseau `Ad hoc` ne fonctionne pas et `NetworkManager` n'arrive pas à établir la connexion. Voir [Add option to disable p2p](https://gitlab.freedesktop.org/NetworkManager/NetworkManager/-/issues/537).  
`wpa_supplicant` est mal configuré par `NetworkManager` et échoue à se connecter au réseau avec l'erreur suivante `nl80211: Failed to set interface into IBSS mode`.

Une alternative est d'arrêter `NetworkManager` et d'utiliser `iw` pour se connecter.

```bash
# Arrêter NetworkManager
systemctl stop NetworkManager

# Configurer une adresse IPv4 lien local
ip link set dev wlan0 up
ip address add dev wlan0 scope link 169.254.10.10/16

ip addr

# Se connecter au réseau Ad hoc avec iw
iw wlan0 set type ibss
iw wlan0 ibss join HP012345 2412 # Remplacer le nom du réseau

iw dev
iw dev wlan0 link
iw dev wlan0 scan | grep -B8 -A5 "SSID: HP012345" # Remplacer le nom du réseau
```

## Sources

* [How to setup an unprotected Ad Hoc (IBSS) Network and if possible with WPA encryption?](https://raspberrypi.stackexchange.com/a/94048)
* [HP Laserjet P1102W Wireless Connection Trouble](https://h30434.www3.hp.com/t5/Printers-Archive-Read-Only/HP-Laserjet-P1102W-Wireless-Connection-Trouble/m-p/2422555/highlight/true#M128700)

