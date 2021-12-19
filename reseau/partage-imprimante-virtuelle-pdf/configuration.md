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
adduser [nomUtilisateur] lpadmin
# ou usermod -a -G lpadmin [nomUtilisateur]
groups [nomUtilisateur]
```

* Configurer l'imprimante virtuelle PDF

    * Ouvrir l'interface du serveur cups à l'adresse _https://localhost:631_
    * Ouvrir l'onglet _**Administration**_
    * Sélectionner _**Ajouter une imprimante**_
    * Sélectionner _**Imprimantes locales :** CUPS-PDF (Virtual PDF Printer)_
    * Sélectionner _**Continuer**_
    * Configurer : 
        * _**Nom**_ : imprimante_virtuelle_pdf
        * _**Partage**  Partager cette imprimante_ : activé
    * Sélectionner _**Continuer**_
    * Configurer : 
        * _**Marque**_ : Generic
    * Sélectionner _**Continuer**_
    * Configurer :
        * _**Modèle**_ : Generic CUPS-PDF Printer (w/ options) (en)
    * Sélectionner _**Ajouter une imprimante**_
    * Configurer : 
        * _**Page Size**_ : A4
        * _**Output Resolution**_ : 600 DPI
    * Sélectionner _**Définir les options par défaut**_

* Déterminer l'adresse de l'imprimante virtuelle partagée

```bash
ippfind
```

## Configuration du client d'impression (Debian)

* Ouvrir l'interface du serveur cups à l'adresse _https://localhost:631_
* Ouvlir l'onglet _**Administration**_
* Sélectionner _**Ajouter une imprimante**_
* Selectionner _**Autres imprimantes réseau :**  Internet Printing Protocol (ipp)_
* Configurer : 
    * _**Connexion :**_ : adresse de l'imprimante virtuelle partagée
* Sélectionner _**Continuer**_
* Configurer : 
    * _**Nom**_ : imprimante_virtuelle_pdf
* Sélectionner _**Continuer**_
* Configurer : 
    * _**Marque**_ : Generic 
* Sélectionner _**Continuer**_
* Configurer : 
    * _**Modèle**_ : Generic PDF Printer (en)
* Sélectionner _**Ajouter une imprimante**_
* Configurer : 
    * _**Page Size**_ : A4
    * _**Resolution**_ : 600 DPI
* Sélectionner _**Définir les options par défaut**_

## Configuration du client d'impression (Windows)

### Déterminer l'adresse de l'imprimante

* Ouvrir l'interface du serveur cups à l'adresse _https://localhost:631_
* Sélectionner _**Imprimantes**_
* Sélectionner l'imprimante virtuelle partagée _**imprimante_virtuelle_pdf**_
* Noter l'adresse http de l'imprimante virtuelle [1] par exemple _http://[serveur]:631/printers/imprimante_virtuelle_pdf_

### Configurer une imprimante dans Windows

* Ouvrir les _**Paramètres**_
* Sélectionner _**Imprimantes et scanneurs**_
* Sélectionner _**Ajouter une imprimante ou un scanneur**_
* Sélectionner _**Je ne trouve pas l'imprimante recherchée dans la liste**_
* Sélectionner _**Sélectionner une imprimante partagée par nom**_
* Configurer :
    * _**Adresse**_ : [1] par exemple _http://[serveur]:631/printers/imprimante_virtuelle_pdf_
* Sélectionner _**Suivant**_
* Configurer : 
    * _**Fabricant**_ : Microsoft
    * _**Imprimantes**_ : Microsoft PS Class Driver
* Sélectionner _**Ok**_
* Sélectionner _**Suivant**_
* Sélectionner _**Terminer**_