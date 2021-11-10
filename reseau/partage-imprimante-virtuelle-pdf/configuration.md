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
    * Sélectionner sur _Ajouter une imprimante_
    * Sélectionner _**Imprimantes locales :** CUPS-PDF (Virtual PDF Printer)_
    * Sélectionner _Continuer_
    * Configurer : 
        * _**Nom**_ : imprimante_virtuelle_pdf
        * _**Partage**  Partager cette imprimante_ : activé
    * Sélectionner _Continuer_
    * Configurer : 
        * _**Marque**_ : Generic
    * Sélectionner _Continuer_
    * Configurer :
        * _**Modèle**_ : Generic CUPS-PDF Printer (w/ options) (en)
    * Sélectionner _Ajouter une imprimante_
    * Configurer : 
        * _**Page Size**_ : A4
        * _**Output Resolution**_ : 600 DPI
    * Sélectionner _Définir les options par défaut_

* Déterminer l'adresse de l'imprimante virtuelle partagée

```bash
ippfind
```

## Configuration du client d'impression (Debian)

* Ouvrir l'interface du serveur cups à l'adresse _https://localhost:631_
* Ouvlir l'onglet _Administration_
* Sélectionner _Ajouter une imprimante_
* Selectionner _**Autres imprimantes réseau :**  Internet Printing Protocol (ipp)_
* Configurer : 
    * _**Connexion :**_ : adresse de l'imprimante virtuelle partagée
* Sélectionner _Continuer_
* Configurer : 
    * _**Nom**_ : imprimante_virtuelle_pdf
* Sélectionner _Continuer_
* Configurer : 
    * _**Marque**_ : Generic 
* Sélectionner _Continuer_
* Configurer : 
    * _**Modèle**_ : Generic PDF Printer (en)
* Sélectionner _Ajouter une imprimante_
* Configurer : 
    * _**Page Size**_ : A4
    * _**Resolution**_ : 600 DPI
* Sélectionner _Définir les options par défaut_

## Configuration du client d'impression (Windows)

* Déterminer l'adresse de l'imprimante

    * Ouvrir l'interface du serveur cups à l'adresse _https://localhost:631_
    * Sélectionner _Imprimantes_
    * Sélectionner l'imprimante virtuelle partagée _imprimante_virtuelle_pdf_
    * Noter l'adresse http de l'imprimante virtuelle [1] par exemple _http://[serveur]:631/printers/imprimante_virtuelle_pdf_

* Configurer une imprimante dans Windows

    * Ouvir les_Paramètres_
    * Sélectionner _Imprimantes et scanneurs_
    * Sélectionner _Ajouter une imprimante ou un scanneur_
    * Sélectionner _Je ne trouve pas l'imprimante recherchée dans la liste_
    * Sélectionner _Sélectionner une imprimante partagée par nom_
    * Configurer :
        * _**Adresse**_ : [1] par exemple _http://[serveur]:631/printers/imprimante_virtuelle_pdf_
    * Selectionner _Suivant_
    * Configurer : 
        * _**Fabricant**_ : Microsoft
        * _**Imprimantes**_ : Microsoft PS Class Driver
    * Sélectionner _Ok_
    * Sélectionner _Suivant_
    * Sélectionner _Terminer_