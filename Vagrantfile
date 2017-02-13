# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    config.vm.box = "bridgesense/xdebugbox"

    config.vm.hostname = "mywebsite.local"
    config.vm.network "private_network", ip: "192.168.33.10"
    config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=755", "fmode=644"]

    config.ssh.username = "vagrant"
    config.ssh.password = "vagrant"


    config.vm.provision "shell", inline: <<-SHELL

        # # User Defined Settings # #
        # # # # # # # # # # # # # # #

        DOCUMENT_ROOT="public_html"
        XDEBUG_PORT=9041
        MAIL_RELAY="vagrant@#{config.vm.hostname}"

        INSTALL_DB="no"
        DB_FILENAME="user_database.sql"
        DB_NAME="user_database"
        DB_USER="user_user"
        DB_PASS="drowssap"

        INSTALL_DB2="no"
        DB2_FILENAME="user_database2.sql"
        DB2_NAME="user_database2"
        DB2_USER="user_user"
        DB2_PASS="drowssap"

        # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # # #

        echo "Updating Bind..."
        sudo sed -i "s@mywebsite\.local@#{config.vm.hostname}@g" /etc/bind/named.conf.local
        sudo sed -i "s@mywebsite\.local@#{config.vm.hostname}@g" /etc/bind/zones/db.default.com
        sudo sed -i "s@.* ; Serial Number.*@            201704769 ;Serial Number@" /etc/bind/zones/db.default.com
        sudo sed -i "s@mywebsite\.local@#{config.vm.hostname}@g" /etc/bind/zones/db.10.33.168.192
        sudo sed -i "s@.* ; Serial Number.*@            201704769 ;Serial Number@" /etc/bind/zones/db.10.33.168.192

        echo "Updating Apache..."
        sudo sed -i "s@ServerName.*@ServerName #{config.vm.hostname}@" /etc/apache2/apache2.conf
        sudo sed -i "s@DocumentRoot.*@DocumentRoot /var/www/${DOCUMENT_ROOT}@" /etc/apache2/sites-available/000-default.conf
        sudo sed -i "s@DocumentRoot.*@DocumentRoot /var/www/${DOCUMENT_ROOT}@" /etc/apache2/sites-available/default-ssl.conf
        sudo sed -i "s@mywebsite\.local@#{config.vm.hostname}@g" /etc/apache2/sites-available/000-default.conf
        sudo sed -i "s@mywebsite\.local@#{config.vm.hostname}@g" /etc/apache2/sites-available/default-ssl.conf

        echo "Updating the Xdebug Port..."
        sudo sed -i "s@9041@${XDEBUG_PORT}@" /etc/php5/mods-available/xdebug.ini
        sudo service iptables-persistent restart
        sudo iptables -I INPUT -p tcp -s 0.0.0.0/0 --dport ${XDEBUG_PORT} -j ACCEPT
        sudo iptables -I OUTPUT -p tcp -s 0.0.0.0/0 --dport ${XDEBUG_PORT} -j ACCEPT

        echo "Creating SSL Key..."
        sudo mkdir /etc/apache2/ssl 2>/dev/null
        sudo openssl req -newkey rsa:2048 -x509 -sha256 -days 999999 -nodes \
            -subj "/C=US/ST=Oregon/L=Portland/O=DevTeam/CN=#{config.vm.hostname}" \
            -out /etc/apache2/ssl/apache.pem \
            -keyout /etc/apache2/ssl/apache.key
        sudo sed -i "s@SSLCertificateFile.*@SSLCertificateFile /etc/apache2/ssl/apache.pem@g" /etc/apache2/sites-available/default-ssl.conf
        sudo sed -i "s@SSLCertificateKeyFile.*@SSLCertificateKeyFile /etc/apache2/ssl/apache.key@g" /etc/apache2/sites-available/default-ssl.conf
        sudo a2ensite default-ssl.conf

        echo "Setting up Mail Relay..."
        sudo sed -i "s#vagrant\@mywebsite\.local#${MAIL_RELAY}#" /etc/postfix/virtual-regexp
        sudo sed -i "s#vagrant\@mywebsite\.local##{config.vm.hostname}#" /etc/postfix/main.cf
        sudo sed -i "s@mywebsite\.local@#{config.vm.hostname}@g" /etc/postfix/main.cf
        sudo service postfix reload

        echo "Installing Database..."
        if [ "${INSTALL_DB}" == "yes" ]; then
            echo "CREATING DATABASE: ${DB_NAME}..."
            if [ -f "/var/www/${DB_FILENAME}"  ]; then
                echo "INJECTING ${DB_NAME}.sql..."
                mysql -u root -p'root' -e "CREATE DATABASE ${DB_NAME}"
                mysql -u root -p'root' -f ${DB_NAME} < /var/www/${DB_FILENAME}
                mysql -u root -p'root' -e "CREATE USER '${DB_USER}'@'localhost' IDENTIFIED BY '$(echo ${DB_PASS})'"
                mysql -u root -p'root' -e "GRANT ALL PRIVILEGES ON * . * TO '${DB_USER}'@'localhost'"
            fi
        fi

        echo "Installing Second Database..."
        if [ "${INSTALL_DB2}" == "yes" ]; then
            echo "CREATING DATABASE: ${DB2_NAME}..."
            if [ -f "/var/www/${DB2_FILENAME}"  ]; then
                echo "INJECTING ${DB2_NAME}.sql..."
                mysql -u root -p'root' -e "CREATE DATABASE ${DB2_NAME}"
                mysql -u root -p'root' -f ${DB2_NAME} < /var/www/${DB2_FILENAME}
                mysql -u root -p'root' -e "CREATE USER '${DB2_USER}'@'localhost' IDENTIFIED BY '$(echo ${DB2_PASS})'"
                mysql -u root -p'root' -e "GRANT ALL PRIVILEGES ON * . * TO '${DB2_USER}'@'localhost'"
            fi
        fi

        echo "Updating YadrLite"
        sudo -H -u vagrant bash -c '~/.yadrlite/setup update'

        echo "Restart Services..."
        sudo service mysql restart
        sudo service bind9 restart
        sudo service apache2 restart

        echo "++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++"
        echo "+  That's all folks!                                       +"
        echo "+  You may add the following line to your hosts file:      +"
        echo "+  #{config.vm.hostname}   192.168.33.10"
        echo "+                                                          +"
        echo "+  Xdebug has been set up on port: ${XDEBUG_PORT}"
        echo "+                                                          +"
        echo "+  All mail will be routed to the following email:         +"
        echo "+  ${MAIL_RELAY}"
        echo "+                                                          +"
        echo "+  If the mail relay is to vagrant@#{config.vm.hostname}"
        echo "+  url: https://192.168.33.10/webmail                      +"
        echo "+  user: vagrant                                           +"
        echo "+  password: vagrant                                       +"
        echo "++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++"


    SHELL
end
