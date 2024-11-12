# Mettre à l'heure le système d'exploitation avec un système de positionnement et ntp

- Installer les dépendances ntp et gpsd
    ```bash
    apt-get install ntpstat ntp gpsd 
    ```

## sirf3

- Configurer gpsd
    ```ini
    # /etc/default/gpsd

    # Default settings for the gpsd init script and the hotplug wrapper.

    # Start the gpsd daemon automatically at boot time
    START_DAEMON="false"

    # Use USB hotplugging to add new USB devices automatically to the daemon
    USBAUTO="true"

    # Devices gpsd should collect to at boot time.
    # They need to be read/writeable, either by user gpsd or the group dialout.
    DEVICES="/dev/ttyUSB0"

    # Other options you want to pass to gpsd
    GPSD_OPTIONS="-n -G -b"
    GPSD_SOCKET="/var/run/gpsd.sock"
    # end of file gpsd
    ```

- Configurer ntp
    ```ini
    # /etc/ntpsec/ntp.conf
    # Commenter pour pouvoir considérer une heure déterminée localement
    # tos minclock 4 minsane 3

    # Utiliser l'heure déterminée et fournie par gpsd
    server 127.127.28.0 minpoll 4 maxpoll 4 true
    fudge 127.127.28.0 time1 0.100 refid GPS

    # Restreindre les droits des clients du serveur NTP
    restrict default kod nomodify notrap
    ```

## neo-m8m

- Configurer gpsd
    ```ini
    # /etc/default/gpsd
    # Default settings for the gpsd init script and the hotplug wrapper.

    # Start the gpsd daemon automatically at boot time
    START_DAEMON="false"

    # Use USB hotplugging to add new USB devices automatically to the daemon
    USBAUTO="true"

    # Devices gpsd should collect to at boot time.
    # They need to be read/writeable, either by user gpsd or the group dialout.
    DEVICES="/dev/ttyACM0"

    # Other options you want to pass to gpsd
    GPSD_OPTIONS="-n -G -b"
    GPSD_SOCKET="/var/run/gpsd.sock"
    # end of file gpsd
    ```

- Configurer ntp
    ```ini
    # /etc/ntpsec/ntp.conf
    # Commenter pour pouvoir considérer une heure déterminée localement
    # tos minclock 4 minsane 3

    # Utiliser l'heure déterminée et fournie par gpsd
    server 127.127.28.0 minpoll 4 maxpoll 4 true
    fudge 127.127.28.0 time1 0.100 refid PPS0
    ```

## Vérifier que la source utilisée est GPS ou PPS
```bash
ntpq -p
```