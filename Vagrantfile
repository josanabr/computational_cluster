# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "master" do |m|
  	m.vm.box = "ubuntu/bionic64"
        m.vm.hostname = "master.localhost"
  	m.vm.provider :virtualbox do |vb|
		vb.customize [ 'modifyvm', :id, '--memory', '1500' ]
		vb.customize [ 'modifyvm', :id, '--cpus', '1' ]
		vb.customize [ 'modifyvm', :id, '--name', 'cluster-master' ]
  	end
  end
  (1..2).each do |i|
    config.vm.define "node#{i}" do |m|
  	m.vm.box = "ubuntu/bionic64"
        m.vm.hostname = "node-0#{i}.localhost"
  	m.vm.provider :virtualbox do |vb|
		vb.customize [ 'modifyvm', :id, '--memory', '750' ]
		vb.customize [ 'modifyvm', :id, '--cpus', '1' ]
		vb.customize [ 'modifyvm', :id, '--name', "cluster-node-#{i}" ]
  	end
    end
  end
end
