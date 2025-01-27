require 'yaml'

config_data = YAML.load_file('config.yml')
db_user = 'put_here_your_user_name'
db_password = config_data['mysql_password']

Vagrant.configure("2") do |config|
  config.vm.define "wordpress" do |wp|
    # wp.vm.box = ENV['BOX'] || "debian/bookworm64"
    # wp.vm.box = ENV['BOX'] || "ubuntu/jammy64"
     wp.vm.box = ENV['BOX'] || "centos/stream9"

    wp.vm.network "private_network", ip: "192.168.66.150"
    wp.vm.synced_folder ".", "/vagrant"
    wp.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end

    wp.vm.provision "shell", inline: <<-SHELL
      set -e
      source /etc/os-release
      DISTRO=$ID
      echo "Detected distribution: $DISTRO"

      if [ "$DISTRO" = "debian" ] || [ "$DISTRO" = "ubuntu" ]; then
        apt-get update
        apt-get install -y mariadb-server apache2 php php-mysql wget tar
        systemctl enable mariadb
        systemctl start mariadb
      elif [ "$DISTRO" = "centos" ]; then
        yum update -y
        yum install -y mariadb-server httpd php php-mysqlnd wget tar
        systemctl enable mariadb
        systemctl start mariadb
      else
        echo "Unsupported distribution: $DISTRO"
        exit 1
      fi

      if ! mysql -e "USE wordpress;" 2>/dev/null; then
        echo "Database wordpress does not exist. Creating..."
        mysql -e "CREATE DATABASE wordpress;"
        mysql -e "CREATE USER '#{db_user}'@'%' IDENTIFIED BY '#{db_password}';"
        mysql -e "GRANT ALL PRIVILEGES ON wordpress.* TO '#{db_user}'@'%';"
        mysql -e "FLUSH PRIVILEGES;"

        if [ -f /vagrant/wordpress_dump.sql ]; then
          echo "Found dump_wordpress.sql, importing..."
          mysql -u #{db_user} -p#{db_password} wordpress < /vagrant/wordpress_dump.sql
        else
          echo "wordpress_dump.sql not found!"
        fi
      fi

       if [ ! -d /var/www/html ]; then
        mkdir -p /var/www/html
      fi

      wget https://wordpress.org/latest.tar.gz -O /tmp/latest.tar.gz
      tar -xzf /tmp/latest.tar.gz -C /tmp

      NEW_VERSION=$(grep "wp_version =" /tmp/wordpress/wp-includes/version.php | awk -F "'" '{print $2}')

      if [ -f /var/www/html/wp-includes/version.php ]; then
        LOCAL_VERSION=$(grep "wp_version =" /var/www/html/wp-includes/version.php | awk -F "'" '{print $2}')
      else
        LOCAL_VERSION="none"
      fi

      echo "Local WordPress version: $LOCAL_VERSION"
      echo "New WordPress version: $NEW_VERSION"

      if [ "$NEW_VERSION" != "$LOCAL_VERSION" ]; then
        echo "New version detected, updating WordPress..."

        tar -czf /vagrant/wordpress_backup_$(date +%F).tar.gz -C /var/www/html .

        cp -r /tmp/wordpress/* /var/www/html/
      else
        echo "No changes detected, WordPress is up to date."
      fi

      if [ "$DISTRO" = "debian" ] || [ "$DISTRO" = "ubuntu" ]; then
        rm -f /var/www/html/index.html
        chown -R www-data:www-data /var/www/html/
      elif [ "$DISTRO" = "centos" ]; then
        chown -R apache:apache /var/www/html/
        setsebool -P httpd_can_network_connect_db 1
      fi

      cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
      sed -i "s/database_name_here/wordpress/" /var/www/html/wp-config.php
      sed -i "s/username_here/#{db_user}/" /var/www/html/wp-config.php
      sed -i "s/password_here/#{db_password}/" /var/www/html/wp-config.php
      sed -i "s/localhost/127.0.0.1/" /var/www/html/wp-config.php

      rm -rf /tmp/latest.tar.gz /tmp/wordpress

      if [ "$DISTRO" = "debian" ] || [ "$DISTRO" = "ubuntu" ]; then
        systemctl restart apache2
      elif [ "$DISTRO" = "centos" ]; then
        systemctl restart httpd
      fi
    SHELL
  end
end