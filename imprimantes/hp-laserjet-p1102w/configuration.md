# Configuration de l'imprimante HP LaserJet P1102w

## Réinitialisation de l'imprimante

- Appuyer sur le bouton de mise sous tension
- Appuyer sur le bouton "annuler" puis "reseau sans fil" et continuer à appuyer jusqu'à ce qu'une page de diagnostique soit imprimée

## Connection au réseau "Ad hoc" de l'imprimante avec NetworkManager

- Dans la rubrique `Network Information`, vérifier avec la propriété `Communication Mode` que le mode du réseau est bien `Ad Hoc`
- Dans la rubrique `Network Information`, déterminer avec la propriété `Network Name (SSID)` le nom du réseau Ad hoc 
- Utiliser la configuration suivante

```bash

```



## Ouvir la page d'administration de l'imprimante en utilisant le réseau Ad hoc de l'imprimante

- Appuyer sur le bouton "annuler" et continuer à appuyer jusqu'à ce qu'une page de diagnostique soit imprimée
- Dans la rubrique `Network Information` de la page de diagnostique, lire l'adresse de la propriété `IPv4 Address`
- Se connecter à l'interface d'administration avec un navigateur avec l'adresse `http://[IPv4 Address lue précédemment]`

