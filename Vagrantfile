
Vagrant.configure("2") do |config|

	config.vm.provider :virtualbox do |vb|
	  vb.gui = true
	end

	config.vm.provision "shell", inline: "echo Hello"
  
	config.vm.define "web" do |web|
		web.vm.box = "debian/bullseye64"
		web.vm.hostname = "web"
		web.vm.network :private_network, ip: "10.0.0.10"
	end
  
	config.vm.define "db" do |db|
		db.vm.box = "debian/bullseye64"
		db.vm.hostname = "db"
		db.vm.network :private_network, ip: "10.0.0.11"
	end
  
	config.vm.define "munin" do |munin|
		munin.vm.box = "debian/bullseye64"
		munin.vm.hostname = "munin"
		munin.vm.network :private_network, ip: "10.0.0.12"
	end
end
