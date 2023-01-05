# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
    config.vm.box = "centos/7"
  
  #  config.vm.provision "ansible" do |ansible|
  #    ansible.verbose = "vvv"
  #    ansible.playbook = "playbook.yml"
  #    ansible.become = "true"
  #  end
  
    config.vm.provider "virtualbox" do |v|
      v.memory = 512
      v.cpus = 1
    end
  
    #config.vm.define "ansible" do |ansible|
    #  ansible.vm.network "private_network", ip: "192.168.50.10", virtualbox__intnet: "net1"
    #  ansible.vm.hostname = "ansible"
      #nfss.vm.provision "shell", path: "ansible_script.sh"
    #end
  
    config.vm.define "nginx" do |nginx|
      nginx.vm.network "public_network", ip: "192.168.88.11"
      nginx.vm.hostname = "nginx"
      #nfsc.vm.provision "shell", path: "nginx_script.sh"
    end
  
  end
  