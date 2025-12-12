# Instalación de CMS en arquitectura de 4 capas en alta disponibilidad.
Contiene los ficheros VagrantFile y de aprovisionamiento necesarios para esta tarea.
## Capas
- Capa 1: Capa pública. Balanceador de carga con Nginx.
- Capa 2: Backend (Dos servidores web y un servidor NFS y motor PHP_FPM).
- Capa 3: Balanceador de Base de Datos con HAProxy.
- Capa 4: Base de Datos (Dos servidores de base de datos con MariaDB)
## Índice

1. [Introduccción](#id1)
2. [Instalaciones](#id2)
3. [Scripts](#id3)

# Introducción

En esta práctica se va a realizar el despliegue de una aplicacion web que esta alojada en un repositorio público de GitHub en alta disponibiliadd.

La infraestructura montada en vagrant se monta un balanceador con Nginx, dos servidores de web , un servidor NFS con la aplicacion alojada en una carpeta a la que acceder los servidores para mostrarla en el navegador, después en la misma maquina hay instalado el motor de PHP-FPM (separa el proceso del PHP del servidor web mediante el protocolo FastCGI), seguidamente hay un servidor que actua de balanceador en los servidores de base de datos, se ha usado HAProxy, y por último dos servidores de base de datos donde se aloja la base de datos.

### Direccionamiento

Para esta practica se han usado las diferentes redes:
- Red publica balanceador: 192.168.10.0/24
- Red privada servidores web: 192.168.20.0/24
- Red privada nfs: 192.168.30.0/24
- Red privada base de datos: 192.168.40.0/24
##### Por cada máquina
- Balanceador: Utiliza dos redes.
    - Red pública (redpublica): 192.168.10.0/24 - utiliza - 192.168.10.10
    - Red privada (redweb): 192.168.20.0/24 - utiliza - 192.168.20.14
- Server Web 1: Utiliza dos redes.
    - Red privada (redweb): 192.168.20.0/24 - utiliza - 192.168.20.10
    - Red privada (rednfs): 192.168.30.0/24 - utiliza - 192.168.30.11
- Server Web 2: Utiliza dos redes.
    - Red privada (redweb): 192.168.20.0/24 - utiliza - 192.168.20.11
    - Red privada (rednfs): 192.168.30.0/24 - utiliza - 192.168.30.12
- Server NFS: Utiliza dos redes.
    - Red privada (redweb): 192.168.20.0/24 - utiliza - 192.168.20.13
    - Red privada (rednfs): 192.168.30.0/24 - utiliza - 192.168.30.13
- Server HAProxy: Utiliza dos redes.
    - Red privada (rednfs): 192.168.30.0/24 - utiliza - 192.168.30.10
    - Red privada (reddatabase): 192.168.40.0/24 - utiliza - 192.168.40.10
- Server DB1: Utiliza una unica red.
    - Red privada (reddatabase): 192.168.40.0/24 - utiliza - 192.168.40.11
- Server DB2: Utiliza una unica red.
    - Red privada (reddatabase): 192.168.40.0/24 - utiliza - 192.168.40.12

 ## Scripts

## Orden para el correcto levantamiento de las máquinas
```
vagrant up serverdatos1ManuelSoltero  serverdatos2ManuelSoltero proxyBBDDManuelSoltero serverNFSManuelSoltero  serverweb1ManuelSoltero serverweb2ManuelSoltero balanceadorManuelSoltero
```
 ### Vagrantfile
```
# -- mode: ruby --
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
config.vm.box = "debian/bookworm64"

  config.vm.define "balanceadorManuelSoltero" do |balanceador|
     balanceador.vm.network "private_network", ip:"192.168.10.10", virtualbox__intnet: "redpublica"
     balanceador.vm.network "private_network", ip:"192.168.20.14", virtualbox__intnet: "redweb"
     balanceador.vm.network "forwarded_port", guest: 80, host: 8080
     balanceador.vm.provision "shell", path: "balanceador.sh"
     balanceador.vm.hostname = "balanceadorManuelSoltero"
  end
  
  config.vm.define "serverweb1ManuelSoltero" do |web1|
     web1.vm.network "private_network", ip:"192.168.20.10", virtualbox__intnet: "redweb"
     web1.vm.network "private_network", ip:"192.168.30.11", virtualbox__intnet: "rednfs"
     web1.vm.provision "shell", path: "web1.sh"
     web1.vm.hostname = "serverweb1ManuelSoltero"
  end
  
  config.vm.define "serverweb2ManuelSoltero" do |web2|
     web2.vm.network "private_network", ip:"192.168.20.11", virtualbox__intnet: "redweb"
     web2.vm.network "private_network", ip:"192.168.30.12", virtualbox__intnet: "rednfs"
     web2.vm.provision "shell", path: "web2.sh"
     web2.vm.hostname = "serverweb2ManuelSoltero"
  end
  
  config.vm.define "serverNFSManuelSoltero" do |nfs|
     nfs.vm.network "private_network", ip:"192.168.20.13", virtualbox__intnet: "redweb"
     nfs.vm.network "private_network", ip:"192.168.30.13", virtualbox__intnet: "rednfs"
     nfs.vm.provision "shell", path: "nfs.sh"
     nfs.vm.hostname = "serverNFSManuelSoltero"
  end
  
  config.vm.define "proxyBBDDManuelSoltero" do |proxy|
     proxy.vm.network "private_network", ip:"192.168.30.10", virtualbox__intnet: "rednfs"
     proxy.vm.network "private_network", ip:"192.168.40.10", virtualbox__intnet: "reddatabase"
     proxy.vm.provision "shell", path: "proxy.sh"
     proxy.vm.hostname = "proxyBBDDManuelSoltero"
  end
  
  config.vm.define "serverdatos1ManuelSoltero" do |db1|
     db1.vm.network "private_network", ip:"192.168.40.11", virtualbox__intnet: "reddatabase"
     db1.vm.provision "shell", path: "db1.sh"
     db1.vm.hostname = "serverdatos1ManuelSoltero"
  end
  
  config.vm.define "serverdatos2ManuelSoltero" do |db2|
     db2.vm.network "private_network", ip:"192.168.40.12", virtualbox__intnet: "reddatabase"
     db2.vm.provision "shell", path: "db2.sh"
     db2.vm.hostname = "serverdatos2ManuelSoltero"
  end

  # Para crear las maquinas en orden
  # vagrant up serverdatos1ManuelSoltero  serverdatos2ManuelSoltero proxyBBDDManuelSoltero serverNFSManuelSoltero  serverweb1ManuelSoltero serverweb2ManuelSoltero balanceadorManuelSoltero


  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # vagrant box outdated. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessible to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  # config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

### Balanceador

```

```

### Server Web 1

```
```
### Server Web 2

```
```
### Server NFS + PHP-FPM

```
```
### Server HAProxy

```
```

### Server DB 1
´´´
´´´

### Server DB 2

```
```
