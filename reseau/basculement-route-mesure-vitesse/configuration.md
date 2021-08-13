# Basculement route par défaut & Mesure de vitesse

## Configurer les routes pour chaque interface

* Créer une table de routage pour les interfaces secondaires.

```bash
interface="eth1"
adresse=$(ip addr show $interface | sed -n "s/inet \(.*\)\/.*/\1/p") #192.168.2.10
adresse_passerelle=$(ip route | sed -n "s/default via \(.*\)dev ${interface}.* src \(.*\) metric.*/\1/p") #192.168.2.1
bloc=$(ip route | sed -n "/default via/! s/\(.*\)dev ${interface}.* src \(.*\) metric.*/\1/p") #192.168.2.0/24

#echo 2 secondaire >> /etc/iproute2/rt_tables
ip rule add from $bloc table secondaire
ip route add default via $adresse_passerelle dev $interface table secondaire
```

* Si dhcpcd est utilisé, un déclencheur peut être utilisé en créant le fichier "/etc/dhcpcd.exit-hook".
Si les événements tel que "BOUND" ne sont jamais déclenchés après le "fork" du processus : dans le fichier "/etc/systemd/system/dhcpcd5.service", remplacer la ligne "ExecStart=/usr/lib/dhcpcd5/dhcpcd -q -b" par "ExecStart=/usr/lib/dhcpcd5/dhcpcd -q -B".

```ini
# /etc/dhcpcd.exit-hook
#
# if [ $interface = 'eth1' ];then
#
# case $reason in
#   	BOUND | RENEW | REBIND | REBOOT)
#
#		Exemples : adresse : 192.168.2.10, adresse_passerelle : 192.168.2.1, bloc : 192.168.2.0/24
#		adresse=$(ip addr show $interface | sed -n "s/inet \(.*\)\/.*/\1/p")
#               adresse_passerelle=$(ip route | sed -n "s/default via \(.*\)dev ${interface}.* src \(.*\) metric.*/\1/p")
#               bloc=$(ip route | sed -n "/default via/! s/\(.*\)dev ${interface}.* src \(.*\) metric.*/\1/p")
#               table_secondaire_existante=$(ip rule | grep -c "lookup secondaire")
#
#               if [ $table_secondaire_existante -lt "1" ];then
#			        #echo 2 secondaire >> /etc/iproute2/rt_tables
#        			ip rule add from $bloc table secondaire
#        			ip route add default via $adresse_passerelle dev $interface table secondaire
#        			echo "Route pour '${interface}' raison '${reason}' configurée'"
#               fi
#        ;;
#esac
#fi
#
cat > /etc/ssh/sshd_config <<EOL
if [ \$interface = 'eth1' ];then
case \$reason in
	BOUND | RENEW | REBIND | REBOOT)
#       Exemples : adresse : 192.168.2.10, adresse_passerelle : 192.168.2.1, bloc : 192.168.2.0/24
        adresse=\$(ip addr show \$interface | sed -n "\s/inet \(.*\)\/.*/\1/p")
        adresse_passerelle=\$(ip route | sed -n "s/default via \(.*\)dev \${interface}.* src \(.*\) metric.*/\1/p")
        bloc=\$(ip route | sed -n "/default via/! s/\(.*\)dev \${interface}.* src \(.*\) metric.*/\1/p")
        table_secondaire_existante=\$(ip rule | grep -c "lookup secondaire")

        if [ \$table_secondaire_existante -lt "1" ];then
			#echo 2 secondaire >> /etc/iproute2/rt_tables
			ip rule add from \$bloc table secondaire
			ip route add default via \$adresse_passerelle dev \$interface table secondaire
			echo "Route pour '\${interface}' raison '\${reason}' configurée'"
        fi
    ;;
esac
fi
EOL
```

* Vérifier que les bonnes passerelles sont utilisées. Les résultats doivent être différents selon l'adresse source.

```bash
ip rule list
ip route show table secondaire
ip route get 1.1.1.1
ip route get 1.1.1.1 from $adresse
```

## Script pour mesurer la vitesse de la connexion internet par interface

```ini
# verifier_vitesse.sh
# #!/bin/bash
# interfaces=(eth0 eth1)
# for interface in ${interfaces[*]}
# do
#	adresse_fichier=$(dirname "$(realpath $0)")"/vitesse.csv"
#	adresse=$(ip addr show $interface | sed -n "s/inet \(.*\)\/.*/\1/p") #192.168.2.10
#	if ! test -f "$adresse_fichier"; then
#    		echo "Interface,"$(speedtest --csv-header) > $adresse_fichier
#	fi
#
#	echo "$interface,"$(speedtest --csv --source $adresse) >> $adresse_fichier
# done

cat > ~/verifier_vitesse.sh <<EOL
#!/bin/bash
interfaces=(eth0 eth1)
for interface in \${interfaces[*]}
do
	adresse_fichier=\$(dirname "\$(realpath \$0)")"/vitesse.csv"
	adresse=\$(ip addr show \$interface | sed -n "s/inet \(.*\)\/.*/\1/p") #192.168.2.10
	if ! test -f "\$adresse_fichier"; then
    		echo "Interface,"\$(speedtest --csv-header) > \$adresse_fichier
	fi

	echo "\$interface,"\$(speedtest --csv --source \$adresse) >> \$adresse_fichier
done
EOL
```

## Mesurer toutes les 30 minutes

* Configurer le planificateur de tâches

```bash
crontab -e
```

* Mesurer toutes les 30 minutes

```ini
SHELL=/bin/bash
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin
*/30 * * * * ~/verifier_vitesse.sh -v 2>&1 | logger -i --tag "verifier-vitesse" &
```

## Basculer automatiquement les routes par défaut en cas de panne

* Créer le script déterminant la meilleure route à utiliser.

```ini
cat > /usr/local/sbin/basculer-route-defaut.sh <<EOL
#!/bin/bash
interface_priviligiee="eth0"
route_actuelle=""
routes_disponibles=""
routes_indisponibles=""
routes=""
periode_verification_secondes=5

delai_reutilisation_interface_degradee=600
declare -A interfaces_degradees

lister_routes_triees(){
        local index_interface_privilegiee=-1
        local route_privilegiee=""
        readarray -t routes <<< \$(ip route show default)

        for index_interface in "\${!routes[@]}"
        do      
                local route=\$(echo \${routes[\$index_interface]} | sed -e 's/[[:space:]]*\$//')
                routes[\$index_interface]=\$route
                local interface=\$(echo \$route | sed "s/default via .*\ dev \(.*\) proto.*/\1/g")

                if [ "\$interface_priviligiee" == "\$interface" ]
                then
                        index_interface_privilegiee=\$index_interface
                fi

        done

        if [ \$index_interface_privilegiee -gt 0 ]
        then
                route_privilegiee=\${routes[\$index_interface_privilegiee]}

                routes[\$index_interface_privilegiee]=\${routes[0]}
                routes[0]=\$route_privilegiee
        fi
}

verifier_connectivite(){
	local interface=\$1
	local resultat_ping=\$(ping -c 1 -w 5 -I \$interface 1.1.1.1 2>/dev/null | awk -F '/' 'END {print \$5}')
	local delai_ping=\$(cut -d'.' -f1 <<< \$resultat_ping)
	local delai_ping=\${delai_ping:-"-1"}
	echo "\$delai_ping" 
}

ajouter_interface_degradee(){
	interface=\$1
	if [ ! -v interfaces_degradees[\$interface] ]
	then
		echo "Interface '\$interface' ajoutée à la liste des interfaces dégradées."
	fi
	interfaces_degradees[\$interface]=\$(date +"%s")
}

nettoyer_interfaces_degradees(){
	for interface in "\${!interfaces_degradees[@]}"
	do
		if [ -v interfaces_degradees[\$interface] ] && [ \$((\$(date  +'%s') - \${interfaces_degradees[\$interface]:-0})) -gt  \$delai_reutilisation_interface_degradee ]
		then
			unset -v 'interfaces_degradees[\$interface]'
			echo "Interface '\$interface' retirée de la liste des interfaces dégradées."
		fi
	done
}

commuter_reseau(){
	metrique=100

	for route_a_supprimer in "\${routes[@]}"
	do	
		#echo "ip route delete default"
		ip route delete default
	done

	for route_a_ajouter in "\${routes[@]}"
	do
		nouvelle_route=\$(echo \$route_a_ajouter | sed -n "s/\(default via.*proto.*metric\).*/\1 \$metrique/p")
		#echo "ip route add \$nouvelle_route"
		ip route add \$nouvelle_route
		if [ \$metrique == 100 ] 
		then
			route_actuelle=\$nouvelle_route
		fi	
		metrique=\$((100+1))
	done
}

trier_routes_par_connectivite(){
	routes_disponibles=()
	routes_indisponibles=()
	routes_degradees=()

	for route in "\${routes[@]}"
	do
		interface=\$(echo \$route | sed "s/default via .*\ dev \(.*\) proto.*/\1/g")
		delai_ping=\$(verifier_connectivite \$interface)
		if [ \$delai_ping -lt 0 ]
		then
			routes_indisponibles+=("\$route")
			ajouter_interface_degradee \$interface
		elif [ -v interfaces_degradees[\$interface] ]
		then
			routes_degradees+=("\$route")
			nettoyer_interfaces_degradees
		else
			routes_disponibles+=("\$route")
		fi
	done

	routes=( "\${routes_disponibles[@]}" "\${routes_degradees[@]}" "\${routes_indisponibles[@]}" )
	route_disponible=\${routes[0]}
}

while sleep \$periode_verification_secondes
do
	lister_routes_triees
	trier_routes_par_connectivite

	if [ "\$route_disponible" != "\$route_actuelle" ]
	then 
		echo "Basculement interface débuté."
		echo "Route actuelle '\$route_actuelle'."
		echo "Route disponible '\$route_disponible'."

		commuter_reseau

		echo "Basculement interface terminé."
		echo "Route actuelle '\$route_actuelle'."
		sleep 60
	fi
done
EOL
```

* Créer un service.

```ini
# /etc/systemd/system/basculer-route-defaut.service
# [Unit]
# Description=Basculer automatiquement les routes par défaut en cas de panne
# Documentation=https://127.0.0.1
# After=network-online.target
# Requires=network-online.target
#
# [Service]
# #Environment=
# Type=simple
# #User=
# ExecStart=/usr/local/sbin/basculer-route-defaut.sh
# Restart=on-failure
# RestartSec=60
# StartLimitInterval=200
# StartLimitBurst=3
# SyslogIdentifier=basculer-route-defaut
#
# [Install]
# WantedBy=multi-user.target
#
cat > /etc/systemd/system/basculer-route-defaut.service <<EOL
#/etc/systemd/system/basculer-route-defaut.service
[Unit]
Description=Basculer automatiquement les routes par défaut en cas de panne
Documentation=https://127.0.0.1
After=network-online.target
Requires=network-online.target

[Service]
#Environment=
Type=simple
#User=
ExecStart=/usr/local/sbin/basculer-route-defaut.sh
Restart=on-failure
RestartSec=60
StartLimitInterval=200
StartLimitBurst=3
SyslogIdentifier=basculer-route-defaut

[Install]
WantedBy=multi-user.target
EOL
```

* Activer le service.

```bash
chmod u+x /usr/local/sbin/basculer-route-defaut.sh
systemctl enable basculer-route-defaut.service
systemctl start basculer-route-defaut.service
```

## Sources

* [speedtest-cli](https://github.com/sivel/speedtest-cli)
* [SourceBasedRouting](http://wiki.wlug.org.nz/SourceBasedRouting)
* [dhcpcd-run-hooks](https://manpages.debian.org/unstable/dhcpcd5/dhcpcd-run-hooks.8.en.html)