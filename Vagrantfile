# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define :lvs1 do |lvs1|
    lvs1.vm.box = "CentOS66"
    lvs1.vm.network :private_network, ip: "192.168.33.11"
    lvs1.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2"]
    end
  end
  config.vm.define :lvs2 do |lvs2|
    lvs2.vm.box = "CentOS66"
    lvs2.vm.network :private_network, ip: "192.168.33.12"
    lvs2.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024", "--cpus", "2"]
    end
  end
  config.vm.define :web1 do |web1|
    web1.vm.box = "CentOS66"
    web1.vm.network :private_network, ip: "192.168.33.101"
    web1.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512", "--cpus", "2"]
    end
  end
  config.vm.define :web2 do |web2|
    web2.vm.box = "CentOS66"
    web2.vm.network :private_network, ip: "192.168.33.102"
    web2.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512", "--cpus", "2"]
    end
  end
end