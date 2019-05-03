# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    config.vm.box = "bridgesense/lampready"

    config.vm.hostname = "lampready.com"
    config.vm.network "private_network", ip: "192.168.33.10"
    config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=755", "fmode=644"]

    config.vm.provision "shell", inline: <<-SHELL

        # # User Defined Settings # #
        # # # # # # # # # # # # # # #

        SUB_DOMAIN="dev"
        DOCUMENT_ROOT="public_html"
        MAIL_RELAY="vagrant@#{config.vm.hostname}"
        PHP_VERSION="5.6" # 5.6 - 7.3

        XDEBUG_PORT=9041
        XDEBUG_FORCE_ERROR_DISPLAY="no"

        INSTALL_DB="no"
        DB_IS_MAGENTO="no"
        DB_IS_WORDPRESS="no"
        DB_NAME="user_database"
        DB_USER="user_user"
        DB_PASS='drowssap'
        DB_PREFIX=""
        DB_FILENAME="user_database.sql" # located below document root
        DB_CUSTOM_FUNCTIONS="custom_functions.sql"

        INSTALL_DB2="no"
        DB2_IS_MAGENTO="no"
        DB2_IS_WORDPRESS="no"
        DB2_NAME=""
        DB2_USER=""
        DB2_PASS=''
        DB2_PREFIX=""
        DB2_FILENAME=""

        INSTALL_DB3="no"
        DB3_NAME=""
        DB3_USER=""
        DB3_PASS=''
        DB3_FILENAME=""

        # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

        if [ "${SUB_DOMAIN}" != "" ]; then
           FULL_DOMAIN=${SUB_DOMAIN}.#{config.vm.hostname}
        else
           FULL_DOMAIN="#{config.vm.hostname}"
        fi

        cd "/var/www/${DOCUMENT_ROOT}/"

        echo "Choosing PHP Version"

        sudo a2dismod php5.6
        sudo a2enmod php${PHP_VERSION}
        sudo update-alternatives --set php /usr/bin/php${PHP_VERSION}

        echo "Updating Bind..."
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/bind/named.conf.local
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/bind/zones/db.default.com
        sudo sed -i "s@.* ; Serial Number.*@            201704800 ;Serial Number@" /etc/bind/zones/db.default.com
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/bind/zones/db.10.33.168.192
        sudo sed -i "s@.* ; Serial Number.*@            201704800 ;Serial Number@" /etc/bind/zones/db.10.33.168.192

        echo "Updating Apache..."
        sudo sed -i "s@ServerName.*@ServerName #{config.vm.hostname}@" /etc/apache2/apache2.conf
        sudo sed -i "s@DocumentRoot.*@DocumentRoot /var/www/${DOCUMENT_ROOT}@" /etc/apache2/sites-available/000-default.conf
        sudo sed -i "s@DocumentRoot.*@DocumentRoot /var/www/${DOCUMENT_ROOT}@" /etc/apache2/sites-available/default-ssl.conf
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/apache2/sites-available/000-default.conf
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/apache2/sites-available/default-ssl.conf

        echo "Updating the Xdebug Configuration..."
        sudo sed -i "s@9041@${XDEBUG_PORT}@" /etc/php/5.6/mods-available/xdebug.ini
        sudo sed -i "s@9041@${XDEBUG_PORT}@" /etc/php/7.2/mods-available/xdebug.ini
        if [ "${XDEBUG_FORCE_ERROR_DISPLAY}" == "yes" ]; then
            sudo printf "xdebug.force_display_errors=1\r" >> /etc/php/5.6/mods-available/xdebug.ini
            sudo printf "xdebug.scream=1" >> /etc/php/5.6/mods-available/xdebug.ini

            sudo printf "xdebug.force_display_errors=1\r" >> /etc/php/7.0/mods-available/xdebug.ini
            sudo printf "xdebug.scream=1" >> /etc/php/7.0/mods-available/xdebug.ini

            sudo printf "xdebug.force_display_errors=1\r" >> /etc/php/7.1/mods-available/xdebug.ini
            sudo printf "xdebug.scream=1" >> /etc/php/7.1/mods-available/xdebug.ini

            sudo printf "xdebug.force_display_errors=1\r" >> /etc/php/7.2/mods-available/xdebug.ini
            sudo printf "xdebug.scream=1" >> /etc/php/7.2/mods-available/xdebug.ini
        fi
        sudo iptables -I INPUT -p tcp -s 0.0.0.0/0 --dport ${XDEBUG_PORT} -j ACCEPT
        sudo iptables -I OUTPUT -p tcp -s 0.0.0.0/0 --dport ${XDEBUG_PORT} -j ACCEPT

        echo "Creating SSL Key..."
        sudo rm -rf /etc/apache2/ssl 2>/dev/null
        sudo mkdir /etc/apache2/ssl 2>/dev/null
        sudo printf "[ SAN ]\nsubjectAltName=DNS:#{config.vm.hostname},DNS:*.#{config.vm.hostname}" >> /etc/ssl/openssl.cnf
        sudo openssl req -newkey rsa:2048 -x509 -sha256 -days 999999 -nodes \
            -subj "/C=US/ST=Oregon/L=Portland/O=DevTeam/CN=#{config.vm.hostname}" \
            -reqexts SAN \
            -extensions SAN \
            -out /etc/apache2/ssl/apache.pem \
            -keyout /etc/apache2/ssl/apache.key
        sudo sed -i "s@SSLCertificateFile.*@SSLCertificateFile /etc/apache2/ssl/apache.pem@g" /etc/apache2/sites-available/default-ssl.conf
        sudo sed -i "s@SSLCertificateKeyFile.*@SSLCertificateKeyFile /etc/apache2/ssl/apache.key@g" /etc/apache2/sites-available/default-ssl.conf
        sudo a2ensite default-ssl.conf 2>/dev/null

        echo "Setting up Mail Relay..."
        sudo sed -i "s#vagrant\@lampready\.com#${MAIL_RELAY}#" /etc/postfix/virtual-regexp
        sudo sed -i "s#vagrant\@lampready\.com##{config.vm.hostname}#" /etc/postfix/main.cf
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/postfix/main.cf
        sudo service postfix reload

        echo "Checking for Database..."
        if [ "${INSTALL_DB}" == "yes" ]; then
            if [ "${DB_NAME}" != "" ]; then
                echo "CREATING DATABASE: ${DB_NAME}..."
                mysql -u root -p'root' -e "CREATE DATABASE ${DB_NAME}"
            fi
            if [ -f "/var/www/${DB_FILENAME}"  ]; then
                echo "INJECTING ${DB_FILENAME}..."
                mysql -u root -p'root' -f ${DB_NAME} < /var/www/${DB_FILENAME}
            fi
            if [ -f "/var/www/${DB_CUSTOM_FUNCTIONS}"  ]; then
                echo "INJECTING ${DB_CUSTOM_FUNCTIONS}..."
                mysql -u root -p'root' -f ${DB_NAME} < /var/www/${DB_CUSTOM_FUNCTIONS}
            fi
            if [ "${DB_USER}" != "" ]; then
                mysql -u root -p'root' -e "CREATE USER '${DB_USER}'@'localhost' IDENTIFIED BY '$(echo ${DB_PASS})'"
                mysql -u root -p'root' -e "GRANT ALL PRIVILEGES ON * . * TO '${DB_USER}'@'localhost'"
            fi
            if [ "${DB_IS_MAGENTO}" == "yes" ]; then
                echo "Setting up Magento..."
                MGQ1=$(mysql -N -u root -proot -e "use ${DB_NAME}; SELECT value FROM ${DB_PREFIX}core_config_data WHERE scope = 'default' AND path = 'web/unsecure/base_url'")
                MGA1=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $MGQ1)
                mysql -u root -p'root' -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}core_config_data SET value = '$MGA1' WHERE path = 'web/unsecure/base_url'"
                MGQ2=$(mysql -N -u root -proot -e "use ${DB_NAME}; SELECT value FROM ${DB_PREFIX}core_config_data WHERE scope = 'default' AND path = 'web/secure/base_url'")
                MGA2=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $MGQ2)
                mysql -u root -p'root' -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}core_config_data SET value = '$MGA2' WHERE path = 'web/secure/base_url'"
                mysql -u root -p'root' -e "USE ${DB_NAME}; TRUNCATE ${DB_PREFIX}core_cache"
                mysql -u root -p'root' -e "USE ${DB_NAME}; TRUNCATE ${DB_PREFIX}core_cache_tag"
                mysql -u root -p'root' -e "USE ${DB_NAME}; TRUNCATE ${DB_PREFIX}core_session"
                php -f shell/compiler.php -- disable
                php -f shell/indexer.php reindexall
            fi
            if [ "${DB_IS_WORDPRESS}" == "yes" ]; then
                WPQ1=$(mysql -N -u root -proot -e "use ${DB_NAME}; SELECT option_value FROM ${DB_PREFIX}options WHERE option_name = 'siteurl'")
                WPA1=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $WPQ1)
                mysql -u root -p'root' -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}options SET option_value = '${WPA1}' WHERE option_name = 'siteurl' OR option_name = 'home'"
            fi
        fi

        echo "Checking for Second Database..."
        if [ "${INSTALL_DB2}" == "yes" ]; then
            if [ "${DB2_NAME}" != "" ]; then
                echo "CREATING DATABASE: ${DB2_NAME}..."
                mysql -u root -p'root' -e "CREATE DATABASE ${DB2_NAME}"
            fi
            if [ -f "/var/www/${DB2_FILENAME}"  ]; then
                echo "INJECTING ${DB2_FILENAME}..."
                mysql -u root -p'root' -f ${DB2_NAME} < /var/www/${DB2_FILENAME}
            fi
            if [ "${DB2_USER}" != "" ]; then
                mysql -u root -p'root' -e "CREATE USER '${DB2_USER}'@'localhost' IDENTIFIED BY '$(echo ${DB2_PASS})'"
                mysql -u root -p'root' -e "GRANT ALL PRIVILEGES ON * . * TO '${DB2_USER}'@'localhost'"
            fi
            if [ "${DB2_IS_MAGENTO}" == "yes" ]; then
                echo "Setting up Magento..."
                MGQ3=$(mysql -N -u root -proot -e "use ${DB2_NAME}; SELECT value FROM ${DB2_PREFIX}core_config_data WHERE scope = 'default' AND path = 'web/unsecure/base_url'")
                MGA3=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $MGQ3)
                mysql -u root -p'root' -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}core_config_data SET value = '$MGA3' WHERE path = 'web/unsecure/base_url'"
                MGQ4=$(mysql -N -u root -proot -e "use ${DB2_NAME}; SELECT value FROM ${DB@_PREFIX}core_config_data WHERE scope = 'default' AND path = 'web/secure/base_url'")
                MGA4=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $MGQ4)
                mysql -u root -p'root' -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}core_config_data SET value = '$MGA4' WHERE path = 'web/secure/base_url'"
                mysql -u root -p'root' -e "USE ${DB2_NAME}; TRUNCATE ${DB2_PREFIX}core_cache"
                mysql -u root -p'root' -e "USE ${DB2_NAME}; TRUNCATE ${DB2_PREFIX}core_cache_tag"
                mysql -u root -p'root' -e "USE ${DB2_NAME}; TRUNCATE ${DB2_PREFIX}core_session"
                php -f shell/compiler.php -- disable
                php -f shell/indexer.php reindexall
            fi
            if [ "${DB2_IS_WORDPRESS}" == "yes" ]; then
                WPQ2=$(mysql -N -u root -proot -e "use ${DB2_NAME}; SELECT option_value FROM ${DB2_PREFIX}options WHERE option_name = 'siteurl'")
                WPA2=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $WPQ2)
                mysql -u root -p'root' -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}options SET option_value = '${WPA2}' WHERE option_name = 'siteurl' OR option_name = 'home'"
            fi
        fi

        echo "Checking for Third Database..."
        if [ "${INSTALL_DB3}" == "yes" ]; then
            if [ "${DB3_NAME}" != "" ]; then
                echo "CREATING DATABASE: ${DB3_NAME}..."
                mysql -u root -p'root' -e "CREATE DATABASE ${DB3_NAME}"
            fi
            if [ -f "/var/www/${DB3_FILENAME}"  ]; then
                echo "INJECTING ${DB3_FILENAME}..."
                mysql -u root -p'root' -f ${DB2_NAME} < /var/www/${DB3_FILENAME}
            fi
            if [ "${DB3_USER}" != "" ]; then
                mysql -u root -p'root' -e "CREATE USER '${DB3_USER}'@'localhost' IDENTIFIED BY '$(echo ${DB3_PASS})'"
                mysql -u root -p'root' -e "GRANT ALL PRIVILEGES ON * . * TO '${DB3_USER}'@'localhost'"
            fi
        fi

        echo "Restart Services..."
        sudo service mysql restart
        sudo service bind9 restart
        sudo service apache2 restart

        echo "++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++"
        echo "+                                                          +"
        echo "+  Your LAMP is ready...                                   +"
        echo "+                                                          +"
        echo "+  Please add the following line to your hosts file:       +"
        echo "+  192.168.33.10   ${FULL_DOMAIN}"
        echo "+                                                          +"
        echo "+  Xdebug has been set up on port: ${XDEBUG_PORT}"
        echo "+                                                          +"
        echo "+  All mail will be routed to the following email:         +"
        echo "+  ${MAIL_RELAY}"
        echo "+                                                          +"
        echo "+  Outbound email can be accessed at:                      +"
        echo "+  url: https://${FULL_DOMAIN}/webmail"
        echo "+  user: vagrant                                           +"
        echo "+  password: vagrant                                       +"
        echo "+                                                          +"
        echo "+  See LAMPready.com for more information.                 +"
        echo "+                                                          +"
        echo "++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++"


    SHELL

end
