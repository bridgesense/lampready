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
        PHP_VERSION="7.2" # 7.2 - 7.4

        XDEBUG_PORT=9041
        XDEBUG_FORCE_ERROR_DISPLAY="no"

        INSTALL_DB="no" # yes or no
        DB_IS_MAGENTO="no" # 1, 2 or no 
        DB_IS_WORDPRESS="no" # yes or no
        DB_NAME="user_database"
        DB_USER="user_user"
        DB_PASS='drowssap'
        DB_PREFIX=""
        DB_FILENAME="user_database.sql" # located below document root
        DB_CUSTOM_FUNCTIONS="custom_functions.sql"

        INSTALL_DB2="no" # yes or no
        DB2_IS_MAGENTO="no" # 1, 2 or no
        DB2_IS_WORDPRESS="no" # yes or no
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

        sudo dnf module reset -y php
        sudo dnf module enable -y php:remi-${PHP_VERSION}

        echo "Updating Bind..."
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/hostname
        sudo printf "\n127.0.0.1   #{FULL_DOMAIN}\n" >> /etc/hosts
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/named.conf
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /var/named/custom.site.db
        sudo sed -i "s@.* ; Serial Number.*@            201704800 ;Serial Number@" /var/named/custom.site.db
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /var/named/custom.site.rev
        sudo sed -i "s@.* ; Serial Number.*@            201704800 ;Serial Number@" /var/named/custom.site.rev

        echo "Updating Apache..."
        sudo sed -i "s@DocumentRoot.*@DocumentRoot /var/www/${DOCUMENT_ROOT}@" /etc/httpd/conf.d/default.conf
        sudo sed -i "s@DocumentRoot.*@DocumentRoot /var/www/${DOCUMENT_ROOT}@" /etc/httpd/conf.d/ssl.conf
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/httpd/conf.d/default.conf
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/httpd/conf.d/ssl.conf

        echo "Updating the Xdebug Configuration..."
        sudo sed -i "s@9041@${XDEBUG_PORT}@" /etc/php.d/15-xdebug.ini
        if [ "${XDEBUG_FORCE_ERROR_DISPLAY}" == "yes" ]; then
            sudo printf "xdebug.force_display_errors=2\n" >> /etc/php.d/15-xdebug.ini
            sudo printf "xdebug.scream=1" >> /etc/php.d/15-debug.ini
        fi
        sudo iptables -I INPUT -p tcp -s 0.0.0.0/0 --dport ${XDEBUG_PORT} -j ACCEPT
        sudo iptables -I OUTPUT -p tcp -s 0.0.0.0/0 --dport ${XDEBUG_PORT} -j ACCEPT

        echo "Creating SSL Key..."
        sudo rm -rf /etc/pki/tls/certs/localhost.crt 2>/dev/null
        sudo rm -rf /etc/pki/tls/private/localhost.key 2>/dev/null
        sudo printf "\n[ SAN ]\nsubjectAltName=DNS:#{config.vm.hostname},DNS:*.#{config.vm.hostname}" >> /etc/pki/tls/openssl.cnf
        sudo openssl req -newkey rsa:2048 -x509 -sha256 -days 999999 -nodes \
            -subj "/C=US/ST=Oregon/L=Portland/O=DevTeam/CN=#{config.vm.hostname}" \
            -reqexts SAN \
            -extensions SAN \
            -out /etc/pki/tls/certs/localhost.crt \
            -keyout /etc/pki/tls/private/localhost.key

        echo "Setting up Mail Relay..."
        sudo sed -i "s#vagrant\@lampready\.com#${MAIL_RELAY}#" /etc/postfix/virtual-regexp
        sudo sed -i "s#vagrant\@lampready\.com##{config.vm.hostname}#" /etc/postfix/main.cf
        sudo sed -i "s@lampready\.com@#{config.vm.hostname}@g" /etc/postfix/main.cf
        sudo postmap /etc/postfix/virtual-regexp
        sudo systemctl reload postfix 

        echo "Checking for Database..."
        if [ "${INSTALL_DB}" == "yes" ]; then
            if [ "${DB_NAME}" != "" ]; then
                echo "CREATING DATABASE: ${DB_NAME}..."
                mysql -e "CREATE DATABASE ${DB_NAME}"
            fi
            if [ -f "/var/www/${DB_FILENAME}"  ]; then
                echo "INJECTING ${DB_FILENAME}..."
                mysql -f ${DB_NAME} < /var/www/${DB_FILENAME}
            fi
            if [ -f "/var/www/${DB_CUSTOM_FUNCTIONS}"  ]; then
                echo "INJECTING ${DB_CUSTOM_FUNCTIONS}..."
                mysql -f ${DB_NAME} < /var/www/${DB_CUSTOM_FUNCTIONS}
            fi
            if [ "${DB_USER}" != "" ]; then
                mysql -e "CREATE USER '${DB_USER}'@'localhost' IDENTIFIED BY '$(echo ${DB_PASS})'"
                mysql -e "GRANT ALL PRIVILEGES ON * . * TO '${DB_USER}'@'localhost'"
            fi
            if [ "${DB_IS_MAGENTO}" == "1" ] || [ "${DB_IS_MAGENTO}" == "yes" ]; then
                echo "Setting up Magento..."
                MGQ1=$(mysql -N -e "use ${DB_NAME}; SELECT value FROM ${DB_PREFIX}core_config_data WHERE scope = 'default' AND path = 'web/unsecure/base_url'")
                MGA1=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $MGQ1)
                mysql -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}core_config_data SET value = '$MGA1' WHERE path = 'web/unsecure/base_url'"
                MGQ2=$(mysql -N -e "use ${DB_NAME}; SELECT value FROM ${DB_PREFIX}core_config_data WHERE scope = 'default' AND path = 'web/secure/base_url'")
                MGA2=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $MGQ2)
                mysql -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}core_config_data SET value = '$MGA2' WHERE path = 'web/secure/base_url'"
                mysql -e "USE ${DB_NAME}; TRUNCATE ${DB_PREFIX}core_cache"
                mysql -e "USE ${DB_NAME}; TRUNCATE ${DB_PREFIX}core_cache_tag"
                mysql -e "USE ${DB_NAME}; TRUNCATE ${DB_PREFIX}core_session"
                php -f shell/compiler.php -- disable
                php -f shell/indexer.php reindexall
            fi
            if [ "${DB_IS_MAGENTO}" == "2" ]; then
                php bin/magento setup:store-config:set --base-url="http://${FULL_DOMAIN}/"
                php bin/magento setup:store-config:set --base-url-secure="https://${FULL_DOMAIN}/"
                php bin/magento cache:flush
                php bin/magento indexer:reindex
            fi
            if [ "${DB_IS_WORDPRESS}" == "yes" ]; then
                WPQ1=$(mysql -N -e "use ${DB_NAME}; SELECT option_value FROM ${DB_PREFIX}options WHERE option_name = 'siteurl'")
                WPA1=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}@" <<< $WPQ1)
                mysql -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}options SET option_value = '${WPA1}' WHERE option_name = 'siteurl' OR option_name = 'home'"
                mysql -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}posts SET guid = replace(guid, '${WPQ1}','${WPA1}')"
                mysql -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}posts SET post_content = replace(post_content, '${WPQ1}', '${WPA1}')"
                mysql -e "USE ${DB_NAME}; UPDATE ${DB_PREFIX}postmeta SET meta_value = replace(meta_value,'${WPQ1}','${WPA1}')"
            fi
        fi

        echo "Checking for Second Database..."
        if [ "${INSTALL_DB2}" == "yes" ]; then
            if [ "${DB2_NAME}" != "" ]; then
                echo "CREATING DATABASE: ${DB2_NAME}..."
                mysql -e "CREATE DATABASE ${DB2_NAME}"
            fi
            if [ -f "/var/www/${DB2_FILENAME}"  ]; then
                echo "INJECTING ${DB2_FILENAME}..."
                mysql -f ${DB2_NAME} < /var/www/${DB2_FILENAME}
            fi
            if [ "${DB2_USER}" != "" ]; then
                mysql -e "CREATE USER '${DB2_USER}'@'localhost' IDENTIFIED BY '$(echo ${DB2_PASS})'"
                mysql -e "GRANT ALL PRIVILEGES ON * . * TO '${DB2_USER}'@'localhost'"
            fi
            if [ "${DB2_IS_MAGENTO}" == "1" ] || [ "${DB2_IS_MAGENTO}" == "yes" ]; then
                echo "Setting up Magento..."
                MGQ3=$(mysql -N -e "use ${DB2_NAME}; SELECT value FROM ${DB2_PREFIX}core_config_data WHERE scope = 'default' AND path = 'web/unsecure/base_url'")
                MGA3=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $MGQ3)
                mysql -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}core_config_data SET value = '$MGA3' WHERE path = 'web/unsecure/base_url'"
                MGQ4=$(mysql -N -e "use ${DB2_NAME}; SELECT value FROM ${DB@_PREFIX}core_config_data WHERE scope = 'default' AND path = 'web/secure/base_url'")
                MGA4=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}/@" <<< $MGQ4)
                mysql -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}core_config_data SET value = '$MGA4' WHERE path = 'web/secure/base_url'"
                mysql -e "USE ${DB2_NAME}; TRUNCATE ${DB2_PREFIX}core_cache"
                mysql -e "USE ${DB2_NAME}; TRUNCATE ${DB2_PREFIX}core_cache_tag"
                mysql -e "USE ${DB2_NAME}; TRUNCATE ${DB2_PREFIX}core_session"
                php -f shell/compiler.php -- disable
                php -f shell/indexer.php reindexall
            fi
            if [ "${DB2_IS_MAGENTO}" == "2" ]; then
                php bin/magento setup:store-config:set --base-url="http://${FULL_DOMAIN}/"
                php bin/magento setup:store-config:set --base-url-secure="https://${FULL_DOMAIN}/"
                php bin/magento cache:flush
                php bin/magento indexer:reindex
            fi
            if [ "${DB2_IS_WORDPRESS}" == "yes" ]; then
                WPQ2=$(mysql -N -e "use ${DB2_NAME}; SELECT option_value FROM ${DB2_PREFIX}options WHERE option_name = 'siteurl'")
                WPA2=$(sed -E "s@(:\/\/([a-zA-Z\-]*\.)?([a-zA-Z\-]*)\.([a-zA-Z\-]*))(\.[a-zA-Z\-]*)?\/?@://${FULL_DOMAIN}@" <<< $WPQ2)
                mysql -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}options SET option_value = '${WPA2}' WHERE option_name = 'siteurl' OR option_name = 'home'"
                mysql -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}posts SET guid = replace(guid, '${WPQ2}','${WPA2}')"
                mysql -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}posts SET post_content = replace(post_content, '${WPQ2}', '${WPA2}')"
                mysql -e "USE ${DB2_NAME}; UPDATE ${DB2_PREFIX}postmeta SET meta_value = replace(meta_value,'${WPQ2}','${WPA2}')"
            fi
        fi

        echo "Checking for Third Database..."
        if [ "${INSTALL_DB3}" == "yes" ]; then
            if [ "${DB3_NAME}" != "" ]; then
                echo "CREATING DATABASE: ${DB3_NAME}..."
                mysql -e "CREATE DATABASE ${DB3_NAME}"
            fi
            if [ -f "/var/www/${DB3_FILENAME}"  ]; then
                echo "INJECTING ${DB3_FILENAME}..."
                mysql -f ${DB2_NAME} < /var/www/${DB3_FILENAME}
            fi
            if [ "${DB3_USER}" != "" ]; then
                mysql -e "CREATE USER '${DB3_USER}'@'localhost' IDENTIFIED BY '$(echo ${DB3_PASS})'"
                mysql -e "GRANT ALL PRIVILEGES ON * . * TO '${DB3_USER}'@'localhost'"
            fi
        fi

        echo "Restart Services..."
        sudo systemctl restart mysqld
        sudo systemctl restart named
        sudo systemctl restart httpd
        sudo systemctl restart php-fpm
        

        echo "++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++"
        echo "+                                                          +"
        echo "+  Your LAMP is ready...                                   +"
        echo "+                                                          +"
        echo "+  Please add the following line to your hosts file:       +"
        echo "+  192.168.33.10   ${FULL_DOMAIN}"
        echo "+                                                          +"
        echo "+  Xdebug has been set up on port: ${XDEBUG_PORT}"
        echo "+  Memcached is: localhost 11211                           +"
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
