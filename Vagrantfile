# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

    ## Ajout de la clés SSH indiquer dans le fichier de configuration
    config.vm.provision "shell", inline: <<-SHELL
        echo #{File.readlines("/home/adrien/.ssh/id_rsa.pub").first.strip} >> /home/vagrant/.ssh/authorized_keys
    SHELL

    config.vm.define "prom-graf" do |machine|
        machine.vm.box = "debian/buster64"
        machine.vm.hostname = "prom-graf"
        machine.vm.box_url = "debian/buster64"
        machine.vm.network :private_network, ip: "192.168.61.2"
        machine.vm.provider :virtualbox do |v|
            v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
            v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
            v.customize ["modifyvm", :id, "--memory", 2048]
            v.customize ["modifyvm", :id, "--name", "prom-graf"]
            v.customize ["modifyvm", :id, "--cpus", "2"]
        end
    end

    config.vm.define "serv1" do |machine|
        machine.vm.box = "debian/buster64"
        machine.vm.hostname = "serv1"
        machine.vm.box_url = "debian/buster64"
        machine.vm.network :private_network, ip: "192.168.61.3"
        machine.vm.provider :virtualbox do |v|
            v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
            v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
            v.customize ["modifyvm", :id, "--memory", 512]
            v.customize ["modifyvm", :id, "--name", "serv1"]
            v.customize ["modifyvm", :id, "--cpus", "1"]
        end
    end

end
