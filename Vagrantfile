require 'yaml'
#----- -- ------ --- ---- --- ----- --- -- - - --------- -- -- 

config_data = YAML.load_file('config.yml.example')
db_user = 'ByteConqueror'

Vagrant.configure("2") do |config|
  config.vm.define "db" do |db|
    db.vm.box = "generic/debian12"
    db.vm.network "private_network", ip: "192.168.77.100"
    db.vm.synced_folder ".", "/vagrant"
    db.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y mariadb-server
      systemctl enable mariadb
      systemctl start mariadb
      mysql -e "CREATE DATABASE wordpress;"
      mysql -e "CREATE USER '#{db_user}'@'%' IDENTIFIED BY '#{config_data['mysql_password']}';"
      mysql -e "GRANT ALL PRIVILEGES ON wordpress.* TO '#{db_user}'@'%';"
      mysql -e "FLUSH PRIVILEGES;"
      sed -i "s/bind-address.*/bind-address = 0.0.0.0/" /etc/mysql/mariadb.conf.d/50-server.cnf
      systemctl restart mariadb
      if [ -f /vagrant/dump_wordpress.sql ]; then
        echo "Found dump_wordpress.sql, importing..."
        mysql -u #{db_user} -p#{config_data['mysql_password']} wordpress < /vagrant/dump_wordpress.sql
      else
        echo "dump_wordpress.sql not found!"
      fi
    SHELL
   end

config.vm.define "wordpress" do |wp|
    wp.vm.box = "generic/debian12"
    wp.vm.network "private_network", ip: "192.168.77.200"
    wp.vm.synced_folder ".", "/vagrant"
    wp.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y apache2 php php-mysql wget mariadb-client
      wget https://wordpress.org/latest.tar.gz
      tar -xzf latest.tar.gz
      mv wordpress/* /var/www/html/
      rm /var/www/html/index.html
      chown -R www-data:www-data /var/www/html/
      cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
      sed -i "s/database_name_here/wordpress/" /var/www/html/wp-config.php
      sed -i "s/username_here/#{db_user}/" /var/www/html/wp-config.php
      sed -i "s/password_here/#{config_data['mysql_password']}/" /var/www/html/wp-config.php
      sed -i "s/localhost/192.168.77.100/" /var/www/html/wp-config.php
    SHELL
  end
end