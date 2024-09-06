# Configuration GRUB (Debian Bookworm)

## Activer le mode texte pour la console GRUB

```bash
fichier="/etc/default/grub"
sed -E -i "s/(#){0,1}(GRUB_TERMINAL=console)/\2/g" $fichier
update-grub
```