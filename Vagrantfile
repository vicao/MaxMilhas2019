# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  #config.vm.network "public_network"

  config.vm.provider "virtualbox" do |vb|
    config.vm.hostname = "zabbix.lab.local"
	vb.memory = 2048
  end

  config.vm.provision "shell", inline <<-SHELL
    sudo apt-get install -y apache2
  SHELL

  config.vm.provision "shell", path: "build.sh", privileged: true
end
