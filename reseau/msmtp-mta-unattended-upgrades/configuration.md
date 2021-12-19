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

* Configurer /etc/apt/apt.conf.d/50unattended-upgrades

```bash
fichier="50unattended-upgrades"
adresse_destinataire="destinataire@localhost.com"
adresse_expediteur="expediteur@localhost.com"

# Unattended-Upgrade::Mail "[adresse_destinataire]";
sed -E -i "s/(\/\/){0,1}( )*(Unattended-Upgrade::Mail \")(.*)(\";)/\3$adresse_destinataire\5/g" $fichier

# Unattended-Upgrade::Sender "[adresse_expediteur]";
sed -E -i "s/(\/\/){0,1}( )*(Unattended-Upgrade::Sender \")(.*)(\";)/\3$adresse_expediteur\5/g" $fichier
occurence=$(grep -o '^Unattended-Upgrade::Sender' $fichier | wc -l)
if [ $occurence -eq "0" ];then
	echo -e "\nUnattended-Upgrade::Sender \""$adresse_expediteur"\";" >> $fichier
fi

# "origin=Raspbian,codename=${distro_codename},label=Raspbian";
ligneInsertion=$(sed -n '/"origin=Debian,codename=${distro_codename}-security,label=Debian-Security";/=' $fichier)
ligneInsertion=$((ligneInsertion+1))
occurence=$(grep -o '"origin=Raspbian,codename=${distro_codename},label=Raspbian";' $fichier | wc -l)
if [ $occurence -eq "0" ];then
	sed -i $ligneInsertion'i \\n\t"origin=Raspbian,codename=${distro_codename},label=Raspbian";' $fichier
fi

# "origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";
ligneInsertion=$(sed -n '/"origin=Raspbian,codename=${distro_codename},label=Raspbian";/=' $fichier)
ligneInsertion=$((ligneInsertion+1))
occurence=$(grep -o '"origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";' $fichier | wc -l)
if [ $occurence -eq "0" ];then
	sed -i $ligneInsertion'i \\t"origin=Raspberry Pi Foundation,codename=${distro_codename},label=Raspberry Pi Foundation";\n' $fichier
fi
```

* Tester configurant le serveur ssh pour envoyer une notification à chaque connexion.

* Créer le fichier /etc/ssh/sshrc

```bash
# /etc/ssh/sshrc

# ipClient=$(echo $SSH_CONNECTION | cut -d " " -f 1)
# nomServeur=$(hostname -f)
# # logger -t ssh-wrapper "L'utilisateur $USER s'est connecté à partir de l'ip $ipClient."
# echo "L'utilisateur $USER s'est connecté à partir de l'ip $ipClient." | mail -s "Connexion ssh [$nomServeur]" [adresse]

cat > /etc/ssh/sshrc <<EOL
ipClient=\$(echo \$SSH_CONNECTION | cut -d " " -f 1)
nomServeur=\$(hostname -f)
# logger -t ssh-wrapper "L'utilisateur \$USER s'est connecté à partir de l'ip \$ipClient."
echo "L'utilisateur \$USER s'est connecté à partir de l'ip \$ipClient." | mail -s "Connexion ssh [\$nomServeur]" [adresse]
EOL
```