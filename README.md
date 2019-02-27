# Desplegando un cluster computacional

En este repositorio se mostrará como se puede crear un cluster computacional que tenga las siguientes herramientas:

* [NFS](https://en.wikipedia.org/wiki/Network_File_System): sistema de archivo distribuido
* [Ganglia](http://ganglia.sourceforge.net/): herramienta de monitoreo
* [HAProxy](http://www.haproxy.org/): herramienta para el balanceo de carga
* [OpenMPI](https://www.open-mpi.org/): herramienta para la programación de aplicaciones distribuidas

Para llevar a cabo el despliegue de este sistema se hará uso de la herramienta de virtualización VirtualBox y de Vagrant como herramienta para el aprovisionamiento de sistemas operativos en máquinas virtuales.

[Definición de máquinas virtuales](#definicion-de-maquinas-virtuales)

---

# Definicion de maquinas virtuales

Para llevar a cabo esta demostración se crearán tres máquinas virtuales con la siguiente configuración. 

* Maestro: 1 procesador, 1.5 GB de RAM
* Nodo de trabajo (2): 1 procesador, 750 MB

Este despliegue se llevará a cabo en una máquina con 4 núcleos (2.7 GHz Intel Core i5) y 8 GB de RAM.

La definición de estas máquinas se da en el siguiente Vagrantfile.

```
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
```

Para desplegar las máquinas se puede hacer con el comando:

```
vagrant up
```

O se pueden gestionar cada una de las máquinas por sus nombres:

* `master`
* `node1`
* `node2`

Para crear una máquina en particular `vagrant up master`.
Para acceder a una máquina en especial via `ssh`, `vagrant ssh node1`.
Para destruir una máquina `vagrant destroy node2 -f`.
