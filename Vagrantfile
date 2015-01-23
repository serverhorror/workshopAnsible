# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "Sgoettschkes/debian7-ansible"

  config.vm.define "web1" do |web1|
    web1.vm.network "private_network", ip: "192.168.33.150"
  end
  config.vm.define "web2" do |web2|
    web2.vm.network "private_network", ip: "192.168.33.151"
  end
  config.vm.define "db" do |db|
    db.vm.network "private_network", ip: "192.168.33.152"
  end
end
