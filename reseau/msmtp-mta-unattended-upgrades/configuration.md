# msmtp-mta & unattended-upgrades

* Installer msmtp-mta & bsd-mailx & unattended-upgrades

```bash
apt-get install bsd-mailx msmtp-mta unattended-upgrades
```

* Créer le fichier /etc/msmtprc

```ini
# /etc/msmtprc

# account default
# aliases /etc/aliases
# host [adresseServeur]
# tls  on
# port 587
# auth on
# user [nomUtilisateur]
# password [motDePasse]
# from [adresseExpediteur]
# syslog on

cat > /etc/msmtprc <<EOL
account default
aliases /etc/aliases
host [adresseServeur]
tls  on
port 587
auth on
user [nomUtilisateur]
password [motDePasse]
from [adresseExpediteur]
syslog on
EOL
```

```bash
chown root:msmtp /etc/msmtprc
chmod 640 /etc/msmtprc
```

* Tester
```
echo "Ceci est un test" | mail -s "Envoyé par mailx" $adresse_destinataire
```

* Configurer /etc/apt/apt.conf.d/50unattended-upgrades

```bash
fichier="/etc/apt/apt.conf.d/50unattended-upgrades"
adresse_destinataire="destinataire@localhost"
adresse_expediteur="expediteur@localhost"

# Unattended-Upgrade::Remove-Unused-Dependencies "true";
sed -E -i "s/(\/\/){0,1}( )*(Unattended-Upgrade::Remove-Unused-Dependencies \")(.*)(\";)/\3true\5/g" $fichier

# Unattended-Upgrade::Mail "[adresse_destinataire]";
sed -E -i "s/(\/\/){0,1}( )*(Unattended-Upgrade::Mail \")(.*)(\";)/\3$adresse_destinataire\5/g" $fichier

# Unattended-Upgrade::Sender "[adresse_expediteur]";
sed -E -i "s/(\/\/){0,1}( )*(Unattended-Upgrade::Sender \")(.*)(\";)/\3$adresse_expediteur\5/g" $fichier
occurence=$(grep -o '^Unattended-Upgrade::Sender' $fichier | wc -l)
if [ $occurence -eq "0" ];then
	echo -e "\nUnattended-Upgrade::Sender \""$adresse_expediteur"\";" >> $fichier
fi

# "origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";
ligneInsertion=$(sed -n '/"origin=Debian,codename=${distro_codename}-security,label=Debian-Security";/=' $fichier)
ligneInsertion=$((ligneInsertion+1))
occurence=$(grep -o '"origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";' $fichier | wc -l)
if [ $occurence -eq "0" ];then
	sed -i $ligneInsertion'i \\n\t"origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";\n' $fichier
fi
```

* Tester configurant le serveur ssh pour envoyer une notification à chaque connexion.

* Créer le fichier /etc/ssh/sshrc

```bash
# /etc/ssh/sshrc

# ip_client=$(echo $SSH_CONNECTION | cut -d " " -f 1)
# nom_serveur=$(hostname -f)
# nom_fichier=$(echo -n "$USER-$(date +%F)" | sha256sum | cut -d' ' -f1)
# fichier="/tmp/ssh-connexions-$nom_fichier.log"

# occurence=$(awk "/$ip_client\t/" $fichier 2>/dev/null)

# if [ $? -eq 2 ] || [ -z "$occurence" ];then
#     nombre_connexions_client=1
#     echo "$ip_client\t$nombre_connexions_client" >> $fichier
#     echo "L'utilisateur $USER s'est connecté à partir de l'ip $ip_client." | mail -s "Connexion ssh [$nom_serveur]" adresse_destinataire
# else
#     nombre_connexions_client=$(sed -n "s/\($ip_client\t\)\(.*\)/\2/p" $fichier 2>/dev/null)
#     nombre_connexions_client=$(($nombre_connexions_client+1))
#     sed -i "s/\($ip_client\t\)\(.*\)/\1$nombre_connexions_client/" $fichier
# fi
# # logger -t ssh-wrapper "L'utilisateur $USER s'est connecté à partir de l'ip $ip_client. Nombre de connexions aujourd'hui : $nombre_connexions_client"


adresse_destinataire="destinataire@localhost.com"
cat > /etc/ssh/sshrc <<EOL
ip_client=\$(echo \$SSH_CONNECTION | cut -d " " -f 1)
nom_serveur=\$(hostname -f)
nom_fichier=\$(echo -n "\$USER-\$(date +%F)" | sha256sum | cut -d' ' -f1)
fichier="/tmp/ssh-connexions-\$nom_fichier.log"

occurence=\$(awk "/\$ip_client\t/" \$fichier 2>/dev/null)

if [ \$? -eq 2 ] || [ -z "\$occurence" ];then
    nombre_connexions_client=1
    echo "\$ip_client\t\$nombre_connexions_client" >> \$fichier
    echo "L'utilisateur \$USER s'est connecté à partir de l'ip \$ip_client." | mail -s "Connexion ssh [\$nom_serveur]" $adresse_destinataire
else 
    nombre_connexions_client=\$(sed -n "s/\(\$ip_client\t\)\(.*\)/\2/p" \$fichier 2>/dev/null) 
    nombre_connexions_client=\$((\$nombre_connexions_client+1))
    sed -i "s/\(\$ip_client\t\)\(.*\)/\1\$nombre_connexions_client/" \$fichier
fi

# logger -t ssh-wrapper "L'utilisateur \$USER s'est connecté à partir de l'ip \$ip_client. Nombre de connexions aujourd'hui : \$nombre_connexions_client"
EOL
```