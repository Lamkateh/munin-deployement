Vagrant.configure("2") do |config|
  
	config.vm.define "db" do |subconfig|
		subconfig.vm.box = "debian/bullseye64"
		subconfig.vm.hostname = "db"
		subconfig.vm.network :private_network, ip: "10.0.0.11"

		subconfig.vm.provision "shell", inline: <<-SHELL
			apt-get -y install default-mysql-server
			sed -i 's/^bind-address.*/bind-address = */' /etc/mysql/mariadb.conf.d/50-server.cnf
			# Drop database if it exists to avoid error
			echo 'DROP DATABASE IF EXISTS wordpress' | mysql
			echo 'CREATE DATABASE wordpress' | mysql
			echo "GRANT ALL PRIVILEGES ON wordpress.* TO 'joe'@'%' IDENTIFIED BY '12345';" | mysql
			echo "FLUSH PRIVILEGES;" | mysql
			systemctl restart mariadb		
		SHELL
	end

	config.vm.define "web" do |subconfig|
		subconfig.vm.box = "debian/bullseye64"
		subconfig.vm.hostname = "web"
		subconfig.vm.network :private_network, ip: "10.0.0.10"

		subconfig.vm.provision "shell", inline: <<-SHELL
			apt-get -y install wordpress
			cat  <<-EOF > /etc/apache2/sites-enabled/000-default.conf
			NameVirtualHost *:80
			
			<VirtualHost *:80>
			UseCanonicalName Off
			VirtualDocumentRoot /usr/share/wordpress
			Options All
			</VirtualHost>
			EOF
				a2enmod rewrite
				a2enmod vhost_alias
				systemctl restart apache2
			
				cat <<-EOF > /etc/wordpress/config-default.php
			<?php
			# Created by setup-mysql
			define('DB_NAME', 'wordpress');
			define('DB_USER', 'joe');
			define('DB_PASSWORD', '12345');
			define('DB_HOST', 'db.local');
			define('SECRET_KEY', 'pfIZHYz5q0cnfPdrnw60BLijZualZ8pcxbxEGhBmt4grp5Fm2k');
			define('WP_CONTENT_DIR', '/var/lib/wordpress/wp-content');
			?>
			EOF
			chmod 640 /etc/wordpress/config-default.php
			chown root:www-data /etc/wordpress/config-default.php
			
			apt-get -y install apache2-utils w3m

		SHELL
	end
  
	config.vm.define "munin" do |subconfig|
		subconfig.vm.box = "debian/bullseye64"
		subconfig.vm.hostname = "munin"
		subconfig.vm.network :private_network, ip: "10.0.0.12"
	end

	config.vm.provision "shell", inline: <<-SHELL
		# Update the system
		apt-get update
		apt-get upgrade -y
	SHELL
end
