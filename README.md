# Desplegando un cluster computacional

En este repositorio se mostrará como se puede crear un cluster computacional que tenga las siguientes herramientas:

* [NFS](https://en.wikipedia.org/wiki/Network_File_System): sistema de archivo distribuido
* [Ganglia](http://ganglia.sourceforge.net/): herramienta de monitoreo
* [HAProxy](http://www.haproxy.org/): herramienta para el balanceo de carga
* [OpenMPI](https://www.open-mpi.org/): herramienta para la programación de aplicaciones distribuidas

Para llevar a cabo el despliegue de este sistema se hará uso de la herramienta de virtualización VirtualBox y de Vagrant como herramienta para el aprovisionamiento de sistemas operativos en máquinas virtuales.

[Definición de máquinas virtuales](#definicion-de-maquinas-virtuales)

[NFS](#nfs)


---

# Definicion de maquinas virtuales

Para llevar a cabo esta demostración se crearán tres máquinas virtuales con la siguiente configuración. 

* Maestro: 1 procesador, 1.5 GB de RAM
* Nodo de trabajo (2): 1 procesador, 750 MB

Este despliegue se llevará a cabo en una máquina con 4 núcleos (2.7 GHz Intel Core i5) y 8 GB de RAM.

La definición de estas máquinas se da en el siguiente [Vagrantfile](https://raw.githubusercontent.com/josanabr/computational_cluster/01-DefVMs/Vagrantfile).

```
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

---

# NFS

NFS viene de *Network File System*. 
Este es un servicio de red que permite a un computador (a.k.a. servidor) compartir una carpeta al cual este tenga acceso con otros computadores que lo puedan acceder a través de la red.

Para instalar el servidor de NFS se requiere hacer una instalación en el servidor y otra en el cliente.
A continuación mostraremos como llevar a cabo la instalación en el servidor.

## Servidor NFS

Para llevar a cabo esta instalación usted debe conectarse al servidor. 
Si usted utilizó el `Vagrantfile` de este repositorio usted se podrá conectar al servidor de NFS usando el siguiente comando.

```
vagrant ssh master
```

Estando en este servidor se ejecutarán los siguientes pasos.

### Instalación del servidor de NFS

```
sudo apt-get update && sudo apt-get -y install nfs-kernel-server
```

### Creación del directorio a ser compartido

Para este caso se va a compartir el directorio `/export/shared`.
El directorio `/export/shared` contendrá información general que se quiera compartir en el cluster.

```
sudo mkdir -p /export/shared
sudo mkdir /shared
sudo chmod 777 /{export,shared} && sudo chmod 777 /export/*
```

### Enlazando directorios 

Se asociará el directorio `/shared` con el directorio `/export/shared` del servidor de NFS

```
sudo mount --bind /shared /export/shared
```

### Exportando directorios

Para exportar el directorio `/shared` anteriormente se deben adicionar las siguientes líneas al archivo `/etc/exports`:

```
/export/shared *(rw,fsid=0,insecure,no_subtree_check,async)
```

### Reiniciando el servidor de NFS

```
sudo service nfs-kernel-server restart
```

## Cliente NFS

Para cada uno de los nodos del cluster (`node1` y `node2`) llevar a cabo los siguientes pasos.

### Instalación de software 

```
sudo apt-get update && sudo apt-get install -y nfs-common
```

### Creación de directorio 

```
sudo mkdir /shared
sudo chmod 777 /shared
```


### Configurando el archivo `/etc/fstab`

Editar el archivo `/etc/fstab` y adicionar las siguientes líneas:

```
10.11.13.3:/export/shared  /shared  nfs  auto  0  0
```

### Montando directorios remotos

```
sudo mount -a
```

## Validando instalación

Ingresar al `master` y ejecutar el siguiente comando:

```
touch /shared/demo
```

Salir del `master` e ingresar a `node1`.
Estando allí validar que el archivo existe:

```
ls -l /shared/demo
```

El archivo debería estar disponible también en `node1`.

## Hágalo usted mismo

Desarrollo los scripts que correspondan para que la instalación de NFS se haga a la hora de levantar el cluster.
Es decir, el usuario ejecutará `vagrant up` y deberá poder ejecutar lo que se encuentra en la sección de [validar instalación](#validando-instalacion)
