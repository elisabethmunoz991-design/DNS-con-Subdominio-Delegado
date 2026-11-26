# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Configuración global del proveedor VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end

  # =======================================================
  # 1. SERVIDOR UBUNTU - ROUTER, DHCP Y DNS
  # =======================================================
  config.vm.define "servidor" do |servidor|
    servidor.vm.box = "ubuntu/jammy64"
    servidor.vm.hostname = "dns.practica2.izv"

    # Interfaz 1 enp0s3: NAT
    # Se configura por defecto para gestión SSH y salida a Internet

    # Interfaz 2 enp0s8: Red Oficina
    servidor.vm.network "private_network",
      ip: "192.168.200.1",
      netmask: "255.255.255.0",
      virtualbox__intnet: "red_oficina"

    # Interfaz 3 enp0s9: Red Clientes
    servidor.vm.network "private_network",
      ip: "192.168.222.1",
      netmask: "255.255.255.0",
      virtualbox__intnet: "red_clientes"

    # Aprovisionamiento mediante Ansible Local (se ejecuta dentro del servidor)
    servidor.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "playbook.yml"
      ansible.install_mode = "pip"
      ansible.verbose = "v"
    end
  end

  # =======================================================
  # 2. C_UBUNTU - RED  CLIENTES
  # =======================================================
  config.vm.define "clienteUbuntu" do |cliente|
    cliente.vm.box = "ubuntu/jammy64"
    cliente.vm.hostname = "cliente-ubuntu"

    # Interfaz conectada a la red interna
    # recibe IP por DHCP del servidor Ubuntu
    cliente.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "red_clientes"

    # Quitar la ruta por defecto de la NAT de Vagrant (10.0.2.2)
    # para que el cliente salga a Internet a través del servidor
    cliente.vm.provision "shell",
      run: "always",
      inline: "ip route del default via 10.0.2.2 || true"
  end

  # =======================================================
  # 3. SERVIDOR WINDOWS - RED OFICINA - DNS SUBDOMINIO
  # =======================================================
  config.vm.define "servidorWindows" do |servidor|
    servidor.vm.box = "StefanScherer/windows_2019"
    servidor.vm.hostname = "servidorWindows"

    # Indicar a Vagrant que el sistema invitado es Windows
    servidor.vm.guest = :windows

    # Usar WinRM en lugar de SSH para comunicarse con Windows
    servidor.vm.communicator = "winrm"
    servidor.vm.boot_timeout = 600

    # Interfaz conectada a la red interna oficina
    # IP fija configurada por Vagrant
    servidor.vm.network "private_network",
      # type: "dhcp",
      ip: "192.168.200.2",
      netmask: "255.255.255.0",
      virtualbox__intnet: "red_oficina"

    servidor.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = 2
      
    end
  end

  # =======================================================
  # 4. C_WINDOWS10 - RED OFICINA
  # =======================================================
  config.vm.define "clienteWindows" do |cliente|
    cliente.vm.box = "StefanScherer/windows_10"
    cliente.vm.hostname = "cliente-oficina"

    # Interfaz conectada a la red interna oficina
    # Recibe DHCP de Ubuntu Server
    cliente.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "red_oficina"

    cliente.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = 2
    end
  end

end