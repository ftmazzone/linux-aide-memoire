# apt-get proxy socks 5 via un serveur ssh

Dans l'exemple suivant, le serveur SOCKS est un raspberry Pi avec accès à internet et qui est connecté au client SOCKS via l'interface USB OTG.

Activer le transfert dynamique des ports et démarrage du serveur SOCKS.
```bash
ssh -D 127.0.0.1:1234 pi@serveur.local
```

Configurer apt-get pour utiliser le serveur SOCKS précédent pour télécharger les mises à jour.
```bash
# temporairement
apt-get -o Acquire::http::proxy="socks5h://127.0.0.1:1234" update

# ou globalement
# /etc/apt/apt.conf.d/20proxy
# Acquire::http::proxy=true;
# Acquire::http::proxy="socks5h://127.0.0.1:1234";

adresse_fichier=/etc/apt/apt.conf.d/20proxy
echo $'Acquire::http::proxy=true;\nAcquire::http::proxy="socks5h://127.0.0.1:1234";' > $adresse_fichier
```

### Remarque

* Si l'adresse IP du serveur est inconnue, par example si il n'y a pas de serveur DHCP disponible, on peut déterminer l'adresse du serveur à l'aide de la commande fping.

    ```bash
    fping -c 1 -g 169.254.255.255/16 2> /dev/null
    ```