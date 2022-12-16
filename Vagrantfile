Vagrant.configure("2") do |config|
  
	config.vm.define "web" do |subconfig|
		subconfig.vm.box = "debian/bullseye64"
		subconfig.vm.hostname = "web"
		subconfig.vm.network :private_network, ip: "10.0.0.10"
	end
  
	config.vm.define "db" do |subconfig|
		subconfig.vm.box = "debian/bullseye64"
		subconfig.vm.hostname = "db"
		subconfig.vm.network :private_network, ip: "10.0.0.11"
	end
  
	config.vm.define "munin" do |subconfig|
		subconfig.vm.box = "debian/bullseye64"
		subconfig.vm.hostname = "munin"
		subconfig.vm.network :private_network, ip: "10.0.0.12"
	end
end
