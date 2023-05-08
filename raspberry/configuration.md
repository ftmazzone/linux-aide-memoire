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

## Désactiver l'espace d'échange (swap)

```bash
systemctl disable dphys-swapfile.service
```