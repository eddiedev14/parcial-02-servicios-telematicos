Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04"

  # SERVIDOR 2: vsftpd
  config.vm.define "servidor2" do |srv2|
    srv2.vm.hostname = "srv2-2235060"
    srv2.vm.network "private_network", ip: "192.168.50.2", virtualbox__intnet: "red-interna"
    srv2.vm.provider "virtualbox" do |vb|
      vb.name = "parcial2-servidor2"
      vb.memory = 1024
    end
  end

  # SERVIDOR 1: Firewall UFW (IP pública + red interna)
  config.vm.define "servidor1" do |srv1|
    srv1.vm.hostname = "srv1-2235060"

    # Interfaz PÚBLICA (bridge): IP fija (debe estar libre)
    srv1.vm.network "public_network", ip: "192.168.1.150"

    # Interfaz INTERNA: conecta con el Servidor 2
    srv1.vm.network "private_network", ip: "192.168.50.3", virtualbox__intnet: "red-interna"

    srv1.vm.provider "virtualbox" do |vb|
      vb.name = "parcial2-servidor1"
      vb.memory = 1024
    end
  end
end
