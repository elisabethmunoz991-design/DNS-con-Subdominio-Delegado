 -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Configuración global del proveedor VirtualBox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end

  # =======================================================
  # 1. SERVIDOR UBUNTU - SERVIDOR DNS Y DHCP 
  # =======================================================
  config.vm.define "servidor" do |servidor|
    servidor.vm.box = "ubuntu/jammy64"
    servidor.vm.hostname = "servidor-ubuntu"

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
    # (recibe IP por DHCP del servidor Ubuntu)
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
  # 3. C_ WINDOWS - RED OFICINA
  # =======================================================
  config.vm.define "clienteWindows" do |cliente|
    cliente.vm.box = "ubuntu/jammy64"
    cliente.vm.hostname = "cliente-oficina"

    # Interfaz conectada a la red interna
    # (recibe IP por DHCP del servidor Ubuntu o Windows)
    cliente.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "red_oficina"

    # Quitar la ruta por defecto de la NAT de Vagrant (10.0.2.2)
    cliente.vm.provision "shell",
      run: "always",
      inline: "ip route del default via 10.0.2.2 || true"
  end

  # =======================================================
  # 4. SERVIDOR WINDOWS - RED OFICINA
  # =======================================================
  config.vm.define "servidorWindows" do |servidor|
    servidor.vm.box = "StefanScherer/windows_2019"
    servidor.vm.hostname = "servidorWindows"

    # Indicar a Vagrant que el sistema invitado es Windows
    servidor.vm.guest = :windows

    # Usar WinRM en lugar de SSH para comunicarse con Windows
    servidor.vm.communicator = "winrm"
    servidor.vm.boot_timeout = 600

    # Interfaz conectada a la red interna
    # (con DHCP, recibe IP por MAC desde el servidor Ubuntu)
    servidor.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "red_profesorado"

    servidor.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = 2
      # FIJAMOS LA MAC EN VB: tarjeta 2 (red interna).
      # Debe coincidir con windows_mac del playbook.yml
      vb.customize ["modifyvm", :id, "--macaddress2", "080027000010"]
    end
  end

  # =======================================================
  # 5. CLIENTE(3) - WINDOWS 10 - RED PROFESORADO
  # =======================================================
  config.vm.define "clienteWindows" do |cliente|
    cliente.vm.box = "StefanScherer/windows_10"
    cliente.vm.hostname = "cliente-profesorado"

    # Interfaz conectada a la red interna
    # (con DHCP, puede recibir IP de Ubuntu o de Windows Server)
    cliente.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "red_profesorado"

    cliente.vm.provider "virtualbox" do |vb|
      vb.gui = true
      vb.memory = "2048"
      vb.cpus = 2
    end
  end

end