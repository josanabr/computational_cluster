# -*- mode: ruby -*-
# vi: set ft=ruby :

$nnscript = <<-SCRIPT
echo "" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "auto enp0s8" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "iface enp0s8 inet dhcp" >> /etc/network/interfaces.d/50-cloud-init.cfg
sudo /etc/init.d/networking restart
SCRIPT

$nmscript = <<-SCRIPT
echo "" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "auto enp0s8" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "iface enp0s8 inet static" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "address 10.11.13.3" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "gateway 10.11.13.1" >> /etc/network/interfaces.d/50-cloud-init.cfg
echo "netmask 255.255.255.0" >> /etc/network/interfaces.d/50-cloud-init.cfg
sudo /etc/init.d/networking restart ; true
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.define "master" do |m|
  	m.vm.box = "ubuntu/xenial64"
        m.vm.hostname = "master.localhost"
  	m.vm.provider :virtualbox do |vb|
		vb.customize [ 'modifyvm', :id, '--memory', '1500' ]
		vb.customize [ 'modifyvm', :id, '--cpus', '1' ]
		vb.customize [ 'modifyvm', :id, '--name', 'cluster-master' ]
                vb.customize [ "modifyvm", :id, "--nic2", "hostonly" ]
                vb.customize [ "modifyvm", :id, "--hostonlyadapter2", "vboxnet4" ]
  	end
        m.vm.provision "shell", inline: $nmscript
  end
  (1..2).each do |i|
    config.vm.define "node#{i}" do |m|
  	m.vm.box = "ubuntu/xenial64"
        m.vm.hostname = "node-0#{i}.localhost"
  	m.vm.provider :virtualbox do |vb|
		vb.customize [ 'modifyvm', :id, '--memory', '750' ]
		vb.customize [ 'modifyvm', :id, '--cpus', '1' ]
		vb.customize [ 'modifyvm', :id, '--name', "cluster-node-#{i}" ]
                vb.customize [ "modifyvm", :id, "--nic2", "hostonly" ]
                vb.customize [ "modifyvm", :id, "--hostonlyadapter2", "vboxnet4" ]
  	end
        m.vm.provision "shell", inline: $nnscript
    end
  end
end
