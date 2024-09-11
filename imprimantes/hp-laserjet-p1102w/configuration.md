# Configuration de l'imprimante HP LaserJet P1102w

## Réinitialisation de l'imprimante

- Appuyer sur le bouton de mise sous tension
- Appuyer sur le bouton `annuler` puis `réseau sans fil` et continuer à les maintenir appuyés jusqu'à ce qu'une page de diagnostique soit imprimée

## Connection au réseau "Ad hoc" de l'imprimante avec NetworkManager

- Dans la rubrique `Network Information` de la page de diagnostique, vérifier avec la propriété `Communication Mode` que le mode du réseau est bien `Ad Hoc`
- Dans la rubrique `Network Information` de la page de diagnostique, déterminer avec la propriété `Network Name (SSID)` le nom du réseau Ad hoc 
- Utiliser la configuration suivante pour configurer la connexion avec `NetworkManager`

```bash
# /etc/NetworkManager/system-connections/HP012345.nmconnection
[connection]
id=HP012345
uuid=11111111-2222-3333-4444-555555555555
type=wifi
interface-name=wlan0

[wifi]
mode=adhoc
ssid=HP012345

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

- Appuyer sur le bouton `annuler` et continuer à le maintenir appuyé jusqu'à ce qu'une page de diagnostique soit imprimée
- Dans la rubrique `Network Information` de la page de diagnostique, lire l'adresse de la propriété `IPv4 Address`
- Se connecter à l'interface d'administration avec un navigateur avec l'adresse `http://[IPv4 Address lue précédemment]`

