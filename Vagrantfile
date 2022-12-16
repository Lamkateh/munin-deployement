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
			define('DB_HOST', '10.0.0.11');
			define('SECRET_KEY', 'pfIZHYz5q0cnfPdrnw60BLijZualZ8pcxbxEGhBmt4grp5Fm2k');
			define('WP_CONTENT_DIR', '/var/lib/wordpress/wp-content');
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

		subconfig.vm.provision "shell", inline: <<-SHELL
			apt-get install munin munin-plugins-extra apache2 -y
			ln -s /usr/share/munin/plugins/mysql_ /etc/munin/plugins/mysql_
			ln -s /usr/share/munin/plugins/mysql_bytes /etc/munin/plugins/mysql_bytes
			ln -s /usr/share/munin/plugins/mysql_queries /etc/munin/plugins/mysql_queries
			ln -s /usr/share/munin/plugins/mysql_slowqueries /etc/munin/plugins/mysql_slowqueries
			ln -s /usr/share/munin/plugins/mysql_threads /etc/munin/plugins/mysql_threads

			cat <<-EOF > /etc/wordpress/config-default.php
			dbdir  /var/lib/munin
			htmldir /var/cache/munin/www
			logdir /var/log/munin
			rundir  /var/run/munin
			tmpldir /etc/munin/templates

			[localhost.localdomain]
				address 127.0.0.1
				use_node_name yes

			[db.localdomain]
				address 10.0.0.11
				use_node_name yes

			[web.localdomain]
				address 10.0.0.10
				use_node_name yes
			EOF

			cat <<-EOF > /etc/munin/apache24.conf

			ScriptAlias /munin-cgi/munin-cgi-graph /usr/lib/munin/cgi/munin-cgi-graph
			Alias /munin/static/ /var/cache/munin/www/static/

			<Directory /var/cache/munin/www>
				Require all granted
				Options None
			</Directory>

			<Directory /usr/lib/munin/cgi>
				Require local
				<IfModule mod_fcgid.c>
					SetHandler fcgid-script
				</IfModule>
				<IfModule !mod_fcgid.c>
					SetHandler cgi-script
				</IfModule>
			</Directory>

			Alias /munin /var/cache/munin/www
			EOF

			sudo cp -p /etc/munin/apache24.conf /etc/apache2/sites-available/munin.conf
			sudo a2ensite munin
			sudo systemctl reload apache2

			systemctl restart apache2
			systemctl restart munin
		SHELL
	end

	config.vm.provision "shell", inline: <<-SHELL
		# Update the system
		apt-get update
		apt-get upgrade -y
		apt-get install -y munin-node munin-plugins-extra 
		echo "allow 10.0.0.12" >> /etc/munin/munin-node.conf
		systemctl restart munin-node
	SHELL
end
