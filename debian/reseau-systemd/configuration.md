# Activer l'interface réseau (Debian Bookworm)

```bash
echo auto l0 > /etc/network/interfaces
echo iface lo inet loopback >> /etc/network/interfaces
```

## Interfaces

### Réseau filaire

```bash
echo auto eth0 >> /etc/network/interfaces
echo iface eth0 inet static >> /etc/network/interfaces
echo "    address 192.168.4.2" >> /etc/network/interfaces
echo "    gateway 192.168.4.1" >> /etc/network/interfaces
```

### Réseau sans fil

#### Installer wpasupplicant à partir de l'installeur Debian

```bash
mkdir /tmp/clef
mount /dev/sdb1 /tmp/clef
echo deb [allow-insecure=yes] file:/tmp/clef bookworm main > /etc/apt/sources.list
apt install wpasupplicant
systemctl restart networking
```

#### Configurer le réseau sans fil

```bash
echo allow-hotplug wlan0 >> /etc/network/interfaces
echo iface wlan0 inet dhcp >> /etc/network/interfaces
echo "	wpa-ssid nom_reseau" >> /etc/network/interfaces
echo "	wpa-psk mot_de_passe_reseau" >> /etc/network/interfaces
```

### Réseau par câble usb

```bash
echo auto usb0 >> /etc/network/interfaces
echo allow-hotplug usb0 >> /etc/network/interfaces
echo iface usb0 inet dhcp >> /etc/network/interfaces
```

## Rédémarrer le service réseau

```bash
systemctl restart networking
```