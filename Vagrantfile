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

      export DB_USER="#{db_user}"
      export DB_PASSWORD="#{db_password}"

      if [ "$DISTRO" = "debian" ] || [ "$DISTRO" = "ubuntu" ]; then
        apt-get update
        apt-get install -y mariadb-server apache2 php php-mysql wget tar jq
        systemctl enable mariadb
        systemctl start mariadb
      elif [ "$DISTRO" = "centos" ]; then
        yum update -y
        yum install -y mariadb-server httpd php php-mysqlnd wget tar jq
        systemctl enable mariadb
        systemctl start mariadb
      else
        echo "Unsupported distribution: $DISTRO"
        exit 1
      fi

      if ! mysql -e "USE wordpress;" 2>/dev/null; then
        echo "Database wordpress does not exist. Creating..."
        mysql -e "CREATE DATABASE wordpress;"
        mysql -e "CREATE USER '$DB_USER'@'%' IDENTIFIED BY '$DB_PASSWORD';"
        mysql -e "GRANT ALL PRIVILEGES ON wordpress.* TO '$DB_USER'@'%';"
        mysql -e "FLUSH PRIVILEGES;"

        if [ -f /vagrant/wordpress_dump.sql ]; then
          echo "Found wordpress_dump.sql, importing..."
          mysql -u "$DB_USER" -p"$DB_PASSWORD" wordpress < /vagrant/wordpress_dump.sql
        else
          echo "wordpress_dump.sql not found!"
        fi
      fi

      if [ ! -d /var/www/html ]; then
        mkdir -p /var/www/html
      fi

      if [ -f /var/www/html/wp-includes/version.php ]; then
        LOCAL_VERSION=$(grep "wp_version =" /var/www/html/wp-includes/version.php | awk -F "'" '{print $2}')
      else
        LOCAL_VERSION="none"
      fi

      echo "Local WordPress version: $LOCAL_VERSION"

      NEW_VERSION=$(curl -s https://api.wordpress.org/core/version-check/1.7/ | jq -r '.offers[0].version')
      echo "Latest WordPress version available: $NEW_VERSION"

      if [ "$LOCAL_VERSION" = "none" ] || [ "$LOCAL_VERSION" != "$NEW_VERSION" ]; then
        echo "New version detected, updating WordPress..."
        wget https://wordpress.org/latest.tar.gz -O /tmp/latest.tar.gz
        tar -xzf /tmp/latest.tar.gz -C /tmp
        tar -czf /vagrant/wordpress_backup_$(date +%F).tar.gz -C /var/www/html .
        cp -r /tmp/wordpress/* /var/www/html/
        rm -rf /tmp/latest.tar.gz /tmp/wordpress
      else
        echo "No changes detected, WordPress is up to date."
      fi

      if [ "$DISTRO" = "debian" ] || [ "$DISTRO" = "ubuntu" ]; then
        rm -f /var/www/html/index.html
        chown -R www-data:www-data /var/www/html/
        systemctl restart apache2
      elif [ "$DISTRO" = "centos" ]; then
        chown -R apache:apache /var/www/html/
        setsebool -P httpd_can_network_connect_db 1
        systemctl restart httpd
      fi

      cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
      sed -i "s/database_name_here/wordpress/" /var/www/html/wp-config.php
      sed -i "s/username_here/$DB_USER/" /var/www/html/wp-config.php
      sed -i "s/password_here/$DB_PASSWORD/" /var/www/html/wp-config.php
      sed -i "s/localhost/127.0.0.1/" /var/www/html/wp-config.php
    SHELL
  end
end