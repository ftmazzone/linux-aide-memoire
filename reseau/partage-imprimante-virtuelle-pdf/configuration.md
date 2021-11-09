# Partage d'une imprimante virtuelle PDF avec cups (debian bullseye)

## Configuration du serveur d'impression

* Installer cups et l'imprimante virtuelle PDF 

```bash
 apt-get install cups printer-driver-cups-pdf
 ```

* Ouvrir le port 631 du pare-feu

* Autoriser le serveur à recevoir les requêtes du même sous-réseau

```ini
# /etc/cups/cupsd.conf
#
# Only listen for connections from the local machine.
#Listen localhost:631
BrowseAdress @LOCAL
Port 631

# Restrict access to the server...
<Location />
# Order allow,deny
  Allow @LOCAL
</Location>

```

* Autoriser l'utilisateur à administrer les imprimantes

```bash
usermod -a -G lpadmin [nomUtilisateur]
```

* Configurer l'imprimante virtuelle PDF

    * Ouvrir l'interface du serveur cups à l'adresse _https://localhost:631_
    * Ouvlir l'onglet _Administration_
    * Cliquer sur _Ajouter une imprimante_
    * Sélectionner _**Imprimantes locales :** CUPS-PDF (Virtual PDF Printer)_
    * Cliquer sur _Continuer_
    * Paramétrer : 
        * _**Nom**_ : imprimante_virtuelle_pdf
        * _**Partage**  Partager cette imprimante_ : activé
    * Cliquer sur _Continuer_
    * Paramétrer : 
        * _**Marque**_ : Generic
    * Cliquer sur _Continuer_
    * Paramétrer :
        * _**Modèle**_ : Generic CUPS-PDF Printer (w/ options) (en)
    * Cliquer sur _Ajouter une imprimante_
    * Paramétrer : 
        * _**Page Size**_ : A4
        * _**Output Resolution**_ : 600 DPI
    * Cliquer sur _Définir les options par défaut_

* Déterminer l'adresse de l'imprimante virtuelle partagée

```bash
ippfind
```

## Configuration du client d'impression (debian)

* Ouvrir l'interface du serveur cups à l'adresse _https://localhost:631_
* Ouvlir l'onglet _Administration_
* Cliquer sur _Ajouter une imprimante_
* Selectionner _**Autres imprimantes réseau :**  Internet Printing Protocol (ipp)_
* Paramétrer : 
    * _**Connexion :**_ : adresse de l'imprimante virtuelle partagée
* Cliquer sur _Continuer_
* Paramétrer : 
    * _**Nom**_ : imprimante_virtuelle_pdf
* Cliquer sur _Continuer_
* Paramétrer : 
    * _**Marque**_ : Generic 
* Cliquer sur _Continuer_
* Paramétrer : 
    * _**Modèle**_ : Generic PDF Printer (en)
* Cliquer sur _Ajouter une imprimante_
* Paramétrer : 
    * _**Page Size**_ : A4
    * _**Resolution**_ : 600 DPI
* Cliquer sur _Définir les options par défaut_
