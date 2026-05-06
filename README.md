# 🛡️ PROJET — PROTOTYPE SERVEUR INTRANET / EXTRANET SÉCURISÉ
**Environnement imposé :**
- Hyperviseur : VirtualBox  
- OS : Debian 12 (installation minimale, sans interface graphique)  
- 2 interfaces réseau (LAN + WAN simulé)  
- Aucun DNS à mettre en place (non demandé)

# 🖥️ ÉTAPE 1 — INSTALLATION ET CONFIGURATION DU SERVEUR

## 1.1 Création de la machine virtuelle

    Dans VirtualBox :

    - Type : Linux
    - Version : Debian 12 (64-bit)
    - CPU : 1
    - RAM : 1024 à 2048 Mo
    - Disque : 10 Go (VDI dynamique)


## 1.2 Configuration réseau (AVANT installation)

    | Carte | Mode | Nom | Rôle | IP |
    |-------|------|------|------|------|
    | Carte 1 | Réseau interne | intnet | LAN | 192.168.10.1 |
    | Carte 2 | Réseau interne | extnet | WAN simulé | 150.10.0.1 |
    | Carte 3 (temp.) | NAT | — | Internet (installation) | DHCP |


## 1.3 Installation Debian minimale

    Pendant l’installation :

    - ❌ Décochez **Environnement de bureau**
    - ❌ Décochez **GNOME / KDE**
    - ✅ Laissez uniquement **Serveur SSH** et **Utilitaires usuels**

## Se connecter avec la machine hôte en ssh

    Paramètres VM → Network → NAT → Port forwarding

    ssh nom_utilisateur_vm@ip_vm -p 2222 # port

    # si prob de port, le configurer

        C:\Users\TON_USER\.ssh\
        # Et supprime le fichier concerné


## 1.4 Mise à jour système

    apt update
    apt upgrade -y
    apt install sudo curl wget vim unzip net-tools -y

## 1.5 Configuration IP statique

    nano /etc/network/interfaces
        auto lo
        iface lo inet loopback

        # LAN interne
        auto enp0s3
        iface enp0s3 inet static
            address 192.168.10.1
            netmask 255.255.255.0

        # WAN simulé
        auto enp0s8
        iface enp0s8 inet static
            address 150.10.0.1
            netmask 255.255.0.0

        # NAT temporaire (supprimer plus tard)
        auto enp0s9
        iface enp0s9 inet dhcp
            dns-nameservers 8.8.8.8 1.1.1.1

    systemctl restart networking
    ip a
    

## 1.6 Suppression de la carte NAT (IMPORTANT)

    Quand tout fonctionne :

        - Éteindre la VM

        - Supprimer la carte NAT dans VirtualBox

        - Redémarrer

        - Modifier la configuration, en enlevant toute la zone NAT
    
# 🌐 ÉTAPE 2 — SERVICE WEB

## Serveur Web : Apache HTTP Server

## 2.1 Installation

    apt install apache2 -y
    systemctl enable apache2
    systemctl start apache2
    systemctl status apache2
    curl http://192.168.10.1
    nano ~/.bashrc
        export PATH=$PATH:/usr/sbin
    source ~/.bashrc

## 2.2 Sécurisation des modules

    a2dismod status autoindex -f
    a2enmod ssl headers rewrite
    systemctl reload apache2

## 2.3 Durcissement Apache

    nano /etc/apache2/conf-enabled/security.conf

    Ajouter :

        ServerTokens Prod
        ServerSignature Off
        TraceEnable Off

        Header always append X-Frame-Options SAMEORIGIN
        Header always set X-XSS-Protection "1; mode=block"
        Header always set X-Content-Type-Options nosniff

## 2.4 Arborescences des vhosts

    # importer les fichiers demandé 
        # Sur la machine hôte

        scp -r -P 2222 C:\chemin\vers\dossier Nom_Hôte@127.0.0.1:\trajet\vers\fichier\a\recuperer

        # exemple

        scp -r -P 2222 "C:\Users\gazor\OneDrive\Bureau\Openclassroom\Administrateur systèmes, réseaux et cybersécurité\4. Mettez en place des infrastructures et services Web sécurisés\Eléments de mission\ASRC-P4-main\intranet" picpic@127.0.0.1:/home/picpic/

        scp -r -P 2222 "C:\Users\gazor\OneDrive\Bureau\Openclassroom\Administrateur systèmes, réseaux et cybersécurité\4. Mettez en place des infrastructures et services Web sécurisés\Eléments de mission\ASRC-P4-main\extranet" picpic@127.0.0.1:/home/picpic/
        

    

    ## Copier les fichiers du projet dans les bons dossiers :

        # Extranet
            mv /home/picpic/extranet /var/www/
        # Intranet
            mv /home/picpic/intranet /var/www/

        # Vérifier la présence des fichiers :

            ls /var/www/extranet/
            ls /var/www/intranet/

    # Créer tous le répertoire nécessaire :

        # Répertoire web sur la VM
    
            mkdir -p /var/www/extranet/pdf

## 2.5 Gestion des droits

    # Groupes et utilisateurs

        groupadd dev
        groupadd graph

        useradd -m -G dev testdev
        useradd -m -G graph testgraph
        passwd testdev
        passwd testgraph

    # Mini test

        ls -l /home/
        # Il doit y avoir les 2 nouveaux utilisateurs et celui que tu utilise.

    # Droits généraux pour Apache
        
        chown -R www-data:dev /var/www/extranet
        chown -R www-data:dev /var/www/intranet
        chmod -R 750 /var/www/extranet
        chmod -R 750 /var/www/intranet

    # Dossier PDF (écriture Apache uniquement)

        chown www-data:www-data /var/www/extranet/pdf
        chmod 750 /var/www/extranet/pdf

    # Dossier images (graphistes)

        chown -R www-data:graph /var/www/extranet/images
        chown -R www-data:graph /var/www/intranet/images
        chmod -R 750 /var/www/extranet/images
        chmod -R 750 /var/www/intranet/images
   
    chmod 755 /var
    chmod 755 /var/www
    
    
    
    # Utilisation de ACL pour un accés restraint

        # si elle ne marche pas 

            apt install acl
        
        # On autorise uniquement l'exécution (x) sur le dossier extranet.

            setfacl -m g:graph:--x /var/www/extranet
            setfacl -m g:graph:--x /var/www/intranet

        # Acces lecture pour dev
            setfacl -m u:testdev:rx /var/www/extranet/pdf

            # Cela permet à graph :

            #    ✔ traverser
            #    ❌ lire le dossier
            #    ❌ voir les fichiers

        # Test

            su - testgraph
            cd /var/www/extranet/images
            ls
            cd /var/www/intranet/images
            ls
            # Tous doit marcher

## 2.6 Certificat wildcard auto-signé

    # Création du dossier ssl
        mkdir /etc/apache2/ssl

    # Création de la clé privée
        openssl genrsa -out /etc/apache2/ssl/valserac.key 4096

    # Création du CSR (Certificate Signing Request)
        openssl req -new \
        -key /etc/apache2/ssl/valserac.key \
        -out /etc/apache2/ssl/valserac.csr

        # Informations :

            Country: FR
            State: ILE DE FRANCE
            City: PARIS
            Organization: CONSULAT
            OU: DIRECTION INFRASTRUCTURE ET LOGISTIQUE
            Common Name: *.valserac.com
            Email: admin@valserac.com

    # Création du certificat auto-signé
        openssl x509 -req -days 365 \
        -in /etc/apache2/ssl/valserac.csr \
        -signkey /etc/apache2/ssl/valserac.key \
        -out /etc/apache2/ssl/valserac.crt



## 2.7 VirtualHost EXTRANET (IP publique uniquement)

    nano /etc/apache2/sites-available/extranet.valserac.local.conf

        # Forcer l'utilisation de HTTPS
            <VirtualHost 150.10.0.1:80>
                ServerName extranet.valserac.com
                DocumentRoot /var/www/extranet
                RewriteEngine On
                RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
            </VirtualHost>
        # Configuration SSL
            <VirtualHost 150.10.0.1:443>

                ServerName extranet.valserac.com
                DocumentRoot /var/www/extranet

                # Configuration SSL sécurisée
                SSLEngine on
                SSLCertificateFile /etc/apache2/ssl/valserac.crt
                SSLCertificateKeyFile /etc/apache2/ssl/valserac.key
                # Protocoles SSL sécurisés
                SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
                SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
                SSLHonorCipherOrder on
                # En-têtes de sécurité
                Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
                # Désactiver le listage des répertoires
                <Directory /var/www/extranet>
                    Options +Indexes +FollowSymLinks
                    AllowOverride None
                    Require all granted
                </Directory>
#           J'ai changé les "-" en "+" pour activer le listage et pouvoir avoir un répertoire lors de l'accès au site
#           Sinon il fallait créer un fichier index avec tout le repertoire, faisable à faible échelle mais très complex à grande   échelle.

                ErrorLog ${APACHE_LOG_DIR}/extranet_error.log
                Customlog ${APACHE_LOG_DIR}/extranet_access.log combined
            </VirtualHost>


## 2.8 VirtualHost INTRANET (IP privée + ports 5501 / 5502)

    nano /etc/apache2/sites-available/intranet.valserac.local.conf

        <VirtualHost 192.168.10.1:5501>
            # Configuration HTTP
            ServerName intranet.valserac.com            
            DocumentRoot /var/www/intranet
            RewriteEngine On
            # Rediriger uniquement HTTP 5501 vers HTTPS 5502
            RewriteCond %{SERVER_PORT} =5501
            RewriteRule ^/?(.*)$ https://%{SERVER_NAME}:5502/$1 [R=301,L]
        </VirtualHost>

        <VirtualHost 192.168.10.1:5502>
            # Configuration HTTPS
            ServerName intranet.valserac.com
            DocumentRoot /var/www/intranet

            SSLEngine on
            SSLCertificateFile /etc/apache2/ssl/valserac.crt
            SSLCertificateKeyFile /etc/apache2/ssl/valserac.key

            <Directory /var/www/intranet>
                Options +Indexes +FollowSymLinks
                AllowOverride None
                Require all granted
            </Directory>
#           J'ai changé les "-" en "+" pour activer le listage et pouvoir avoir un répertoire lors de l'accès au site
#           Sinon il fallait créer un fichier index avec tout le repertoire, faisable à faible échelle mais très complex à grande   échelle.

            ErrorLog ${APACHE_LOG_DIR}/intranet_error.log
            CustomLog ${APACHE_LOG_DIR}/intranet_access.log combined
        </VirtualHost>




## 2.9 Ports d’écoute

    nano /etc/apache2/ports.conf

        Listen 80
        Listen 443
        Listen 5501
        Listen 5502

## 3.0 Modification de l'ip du serveur

    nano /etc/apache2/apache2.conf

        # Ajoute en haut du fichier :

        ServerName extranet.valserac.com
    
    a2enconf servername

    a2enmod ssl
    a2dissite 000-default.conf
    a2dissite default-ssl.conf
    a2ensite extranet.valserac.local.conf
    a2ensite intranet.valserac.local.conf
    systemctl restart apache2
    systemctl reload apache2

    

# 📂 ÉTAPE 3 — SERVICE FTP

    # Serveur FTP : vsftpd

        ## 3.1 Installation
apt install vsftpd -y
            
            # Si prob d'installation 
                nano /etc/apt/sources.list
                    # Remplace TOUT !!! par 
                    deb http://deb.debian.org/debian bullseye main contrib non-free
                    deb http://deb.debian.org/debian bullseye-updates main contrib non-free
                    deb http://security.debian.org/debian-security bullseye-security main contrib non-free
                apt update
                apt install vsftpd -y


            apt install ftp -y
            cp /etc/vsftpd.conf /etc/vsftpd.conf.orig

        ## 3.2 Configuration sécurisée

            nano /etc/vsftpd.conf

                listen=YES
                listen_address=192.168.10.1
                anonymous_enable=NO
                local_enable=YES
                write_enable=YES

                chroot_local_user=YES
                allow_writeable_chroot=YES
                local_root=/var/www

                pasv_enable=YES
                pasv_min_port=40000
                pasv_max_port=40100
                pasv_address=192.168.10.1

                log_ftp_protocol=YES
                xferlog_enable=YES
                xferlog_std_format=YES
                xferlog_file=/var/log/vsftpd.log

            systemctl restart vsftpd
            systemctl enable vsftpd

        ## 3.3 Tester FTP

            # Connexion :

                ftp 192.168.10.1

                # Test compte dev :

                    # Login :

                        Name: testdev
                        Password: ********

                        ls
                        cd extranet
                        ls
                        cd css # oui
                        ls
                        cd ..
                        cd images # non (droit graph)
                        cd css 
                        cd..
                        ls
                        cd intranet
                        ls
                        cd css # oui
                        ls
                        cd ..
                        cd images # non (droit graph)
                        cd css 

                # Test compte graph :

                    # Login :

                        Name: testgraph
                        Password: ********

                        ls
                        cd extranet
                        ls
                        cd images # oui                 
                        ls
                        cd ..
                        cd css # non (droit dev)
                        cd..
                        ls
                        cd intranet
                        ls
                        cd images # oui                 
                        ls
                        cd ..
                        cd css # non (droit dev)  
                        cd css # non (droit www-data)


# 🔥 ÉTAPE 4 — FILTRAGE NETFILTER (UFW)

    # Pare-feu : Uncomplicated Firewall

        apt install ufw -y

        # ufw --force reset
        ufw default deny incoming
        ufw default allow outgoing
        # Autorisations minimales :
            # SSH interne uniquement
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 22 proto tcp

            # HTTP/HTTPS public
            ufw allow in on enp0s8 to any port 80 proto tcp
            ufw allow in on enp0s8 to any port 443 proto tcp

            # Intranet
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 5501 proto tcp
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 5502 proto tcp

            # FTP interne
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 21 proto tcp
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 40000:40100 proto tcp

            # NAT Temporaire
            ufw allow in on enp0s9 to any port 22 proto tcp # ip de la carte réseau
            # A enlevé avant d'enlevé la carte réseau !!!

            ufw enable

# 🛡️ ÉTAPE 5 — PRÉVENTION SÉCURITÉ
    ## 5.1 Module anti-DDoS Apache

        # Module : mod_evasive

            apt install libapache2-mod-evasive -y
            nano /etc/apache2/mods-available/evasive.conf
                <IfModule mod_evasive20.c>
                    DOSHashTableSize 3097
                    DOSPageCount 2               # nombre de requêtes par page autorisées
                    DOSSiteCount 50              # nombre total de requêtes sur le site
                    DOSPageInterval     1        # intervalle en secondes
                    DOSSiteInterval     1        # intervalle en secondes
                    DOSBlockingPeriod 10       # durée du blocage en secondes
                    DOSEmailNotify      admin@valserac.com
                    DOSSystemCommand    "su - someuser -c '/sbin/... %s ...'"
                    DOSLogDir           "/var/log/mod_evasive"
                </IfModule>

    ## 5.2 Protection EXTRANET avec CrowdSec

        # Solution IDS : CrowdSec

            apt install crowdsec -y
            apt install libapache2-mod-evasive

            # crowdsec apache bouncer
            curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec-apache/script.deb.sh | sudo bash
            apt-get install crowdsec-apache2-bouncer
            apt update
            a2enmod evasive
            a2enmod crowdsec
            cscli bouncers add apache-bouncer
            systemctl restart apache2

            sudo cscli bouncers add apache-bouncer

                API key for 'apache-bouncer':
                xxxxxxxxxxxxxxxx

                # Copie-la soigneusement !!!

            sudo nano /etc/crowdsec/bouncers/crowdsec-apache-bouncer.conf

                api_url: http://127.0.0.1:8080/
                api_key: TA_CLE_API_ICI

                cache_expiration: 1m
                log_level: info
                log_dir: /var/log/

            # Donner les bonnes permissions
                sudo chown root:www-data /etc/crowdsec/bouncers/crowdsec-apache-bouncer.conf
                sudo chmod 640 /etc/crowdsec/bouncers/crowdsec-apache-bouncer.conf

            sudo systemctl restart crowdsec
            sudo systemctl restart apache2

            # Si l'on souhaite installer crowdsec-firewall-bouncer à la place

            # a2enmod evasive
            # mkdir /var/log/mod_evasive
            # chown -R www-data:www-data /var/log/mod_evasive
            # apt install crowdsec-firewall-bouncer -y
            # systemctl enable crowdsec-firewall-bouncer
            # systemctl start crowdsec-firewall-bouncer
            # systemctl restart apache2
            # cscli bouncers list

        # Configurer surveillance :

            nano /etc/crowdsec/acquis.d/apache.yaml
                source: file
                filenames:
                    - /var/log/apache2/access.log
                    - /var/log/apache2/error.log
                labels:
                    type: apache

            nano /etc/crowdsec/acquis.d/ssh.yaml
                source: file
                filenames:
                    - /var/log/auth.log
                labels:
                    type: syslog

        nano /etc/crowdsec/config.yaml

# ATTENTION à L'INDENTATION !!!
common:
  daemonize: true
  log_media: file
  log_level: info
  log_dir: /var/log/
  log_max_size: 20
  compress_logs: true
  log_max_files: 10
config_paths:
  config_dir: /etc/crowdsec/
  data_dir: /var/lib/crowdsec/data/
  simulation_path: /etc/crowdsec/simulation.yaml
  hub_dir: /etc/crowdsec/hub/
  index_path: /etc/crowdsec/hub/.index.json
  notification_dir: /etc/crowdsec/notifications/
  plugin_dir: /usr/lib/crowdsec/plugins/
crowdsec_service:
  #console_context_path: /etc/crowdsec/console/context.yaml
  acquisition_path: /etc/crowdsec/acquis.yaml
  acquisition_dir: /etc/crowdsec/acquis.d
  parser_routines: 1
cscli:
  output: human
  color: auto
db_config:
  log_level: info
  type: sqlite
  db_path: /var/lib/crowdsec/data/crowdsec.db
  #max_open_conns: 100
  #user:
  #password:
  #db_name:
  #host:
  #port:
  flush:
    max_items: 5000
    max_age: 7d
plugin_config:
  user: nobody # plugin process would be ran on behalf of this user
  group: nogroup # plugin process would be ran on behalf of this group
api:
  server:
    log_level: info
    listen_uri: 127.0.0.1:8080
    profiles_path: /etc/crowdsec/profiles.yaml
    console_path: /etc/crowdsec/console.yaml
    trusted_ips:
      - 127.0.0.1
      - ::1

  client:
    credentials_path: /etc/crowdsec/local_api_credentials.yaml
    insecure_skip_verify: false
prometheus:
  enabled: true
  level: full
  listen_addr: 127.0.0.1
  listen_port: 6060

        # Création du fichier log pour apache

            touch /var/log/apache2/access.log
            touch /var/log/apache2/error.log
            chown www-data:adm /var/log/apache2/*.log
            chmod 644 /var/log/apache2/*.log
            touch /var/log/auth.log
            chmod 644 /var/log/auth.log
            chown root:adm /var/log/auth.log

        
            nano /etc/crowdsec/config.yaml


            # Cherchez la section api et modifiez-la pour désactiver la synchronisation :

            api:
            server:
                # Désactive la synchronisation avec l'API centrale
                online_client: false


            # a2enmod crowdsec
            systemctl restart apache2
            systemctl restart crowdsec
            systemctl status crowdsec

    ## 5.3 Test de sécurité

        1 vm graphique : 2 cartes réseau, 2 interne : 1 nom = intnet, 1 nom = extnet

        ## ATTENTION !!! Avoir le meme nom de réseau sur les cartes réseaux qui ont besoin de communiquer entre elles dans le même réseau



            # Modifier le fichier hosts du client (Windows/Linux) :

                150.10.0.1   extranet.valserac.com
            # Test connection

                # Pour intranet :

                    https://intranet.valserac.com:5502
                    http://intranet.valserac.com:5501

                # Ensuite accéder via navigateur :

                    https://extranet.valserac.com/
                    http://extranet.valserac.com/
                
            # Test attaque :

                for i in {1..200}; do
                    curl -k -H "Host: extranet.valserac.local" https://150.10.0.1/file$i.php &
                done

            ### L'ATTAQUE DOIS ÊTRE FORTE ( ET VARIER LES CIBLES ) !!!

        # Vérification bannissement (sur VM Serv)
            cscli alerts list
            cscli decisions list

        #Suppression ban (sur VM Serv)
            cscli decisions delete --ip 150.10.0.2







## Récupérer les fichiers de la VM Debian 12 vers la machine hôte

    # 1. Création des fichiers pour l'exportation 
        # Créer le ZIP des fichiers web
            apt install zip -y
            
        # Si il y a deja un dossier comme celui là, on le supprime
            rm -rf /home/picpic/config_services_web

        # Création du dossier config du service web + ajout des fichiers
            mkdir /home/picpic/config_services_web
            cp /etc/apache2/apache2.conf /home/picpic/config_services_web/
            cp /etc/apache2/sites-available/*.conf /home/picpic/config_services_web/
            cp /etc/apache2/mods-available/*.conf /home/picpic/config_services_web/
            cp -r /etc/apache2/ssl /home/picpic/config_services_web/ 2>/dev/null


            cd /home/picpic
            zip -r config_services_web.zip config_services_web
        # Vérifier :

            ls -lh /home/picpic/config_services_web.zip
            ## unzip -l /home/picpic/config_services_web.zip (dézip)
        
        # Exporter les configs importantes

            # FTP
            cp /etc/vsftpd.conf /home/picpic/config_ftp.txt

            # Netfilter (UFW)
            ufw status verbose > /home/picpic/config_netfilter.txt

            # CrowdSec
            cscli metrics > /home/picpic/config_crowdsec.txt

        # Vérifier avant SCP
            ls -lh /home/picpic/

            # Tu dois voir :

                config_services_web.zip
                config_ftp.txt
                config_netfilter.txt
                config_crowdsec.txt


    # 2.1 Récupération via SCP (de la VM vers l’hôte)

        # Depuis Windows (PowerShell)
        # Récupérer les fichiers demandé dans /home/picpic/

        # 💡 Remarque : Si ton SSH écoute sur un port différent (ex: 2222), ajoute -P 2222 :

            scp -P 2222 picpic@127.0.0.1:/home/picpic/config_services_web.zip C:\Users\gazor\Downloads\
            scp -P 2222 picpic@127.0.0.1:/home/picpic/config_ftp.txt C:\Users\gazor\Downloads\
            scp -P 2222 picpic@127.0.0.1:/home/picpic/config_netfilter.txt C:\Users\gazor\Downloads\
            scp -P 2222 picpic@127.0.0.1:/home/picpic/config_crowdsec.txt C:\Users\gazor\Downloads\
    
    
    # 2.1' Récupération via dossier partagé VirtualBox

        ***Dans VirtualBox → VM → Configuration → Dossiers partagés.

        # Créer le dossier partagé sur Windows (hôte)

        # Sur ton PC Windows, crée un dossier qui servira de point de transfert, par exemple :

        D:\VM_Shared

        # Vérifie qu’il est accessible et que tu peux y créer des fichiers.

        2️⃣ Ajouter le dossier dans VirtualBox

        Ouvre VirtualBox → Sélectionne ta VM → Configuration → Dossiers partagés

        Clique sur Ajouter un nouveau dossier partagé

        Sélectionne le chemin de ton dossier Windows (D:\VM_Shared)

        Configure les options :

        Automount → coche cette option pour que VirtualBox monte le dossier automatiquement au démarrage de la VM

        Rendre permanent → coche cette option pour que le partage reste actif à chaque démarrage

        VirtualBox montera le dossier dans la VM par défaut dans /media/sf_<nom_du_dossier>.

    3️⃣ Installer les outils nécessaires dans la VM

        Pour que la VM Linux puisse accéder au dossier partagé, installe les Guest Additions et le paquet virtualbox-guest-utils :

            sudo apt update
            sudo apt install virtualbox-guest-utils -y

        # Depuis la VM, copier les fichiers vers le dossier partagé :

            # Exemple
            cp /home/testdev/config_services_web.zip /media/sf_shared/
            cp /home/testdev/config_ftp.txt /media/sf_shared/
            cp /home/testdev/config_netfilter.txt /media/sf_shared/
            cp /home/testdev/config_crowdsec.txt /media/sf_shared/

        # Les fichiers apparaîtront dans le dossier partagé sur ta machine hôte.

## Il est maintenant temps d'enlever la carte réseau enp0s9 (NAT)

    ufw --force reset
        ufw default deny incoming
        ufw default allow outgoing
        # Autorisations minimales :
        # Autorisations minimales :
            # SSH interne uniquement
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 22 proto tcp

            # HTTP/HTTPS public
            ufw allow in on enp0s8 to any port 80 proto tcp
            ufw allow in on enp0s8 to any port 443 proto tcp

            # Intranet
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 5501 proto tcp
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 5502 proto tcp

            # FTP interne
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 21 proto tcp
            ufw allow in on enp0s3 from 192.168.10.0/24 to any port 40000:40100 proto tcp

            ufw enable

    nano /etc/network/interfaces
        auto lo
        iface lo inet loopback

        # LAN interne
        auto enp0s3
        iface enp0s3 inet static
            address 192.168.10.1
            netmask 255.255.255.0

        # WAN simulé
        auto enp0s8
        iface enp0s8 inet static
            address 150.10.0.1
            netmask 255.255.0.0

    systemctl restart networking
    ip a
