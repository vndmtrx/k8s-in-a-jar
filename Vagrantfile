# -*- mode: ruby -*-
# vi: set ft=ruby :

# Para executar no LibVirt sem precisar de senha de root:
# - sudo usermod -a -G libvirt $(whoami)
# - sudo usermod -a -G kvm $(whoami)

ENV["VAGRANT_DEFAULT_PROVIDER"] = "libvirt"

# Definição dos nodes com seus IPs e recursos
nodes = {
  "k8s-lb" => { "ip" => "172.24.0.11", "memory" => 512, "cpus" => 1 },
  "k8s-manager1" => { "ip" => "172.24.0.21", "memory" => 2048, "cpus" => 2 },
  "k8s-manager2" => { "ip" => "172.24.0.22", "memory" => 2048, "cpus" => 2 },
  "k8s-manager3" => { "ip" => "172.24.0.23", "memory" => 2048, "cpus" => 2 },
  "k8s-worker1" => { "ip" => "172.24.0.31", "memory" => 1536, "cpus" => 1 },
  "k8s-worker2" => { "ip" => "172.24.0.32", "memory" => 1536, "cpus" => 1 }
}

Vagrant.configure("2") do |config|
  # Imagem a ser utilizada
  config.vm.box = "debian/bookworm64"
  config.ssh.insert_key = false
  config.vm.synced_folder "./", "/vagrant", type: "virtiofs"

  # Configuração comum para todas as VMs (LibVirt)
  config.vm.provider :libvirt do |libvirt|
    libvirt.driver = "kvm"
    libvirt.memorybacking :source, :type => "memfd"
    libvirt.memorybacking :access, :mode => "shared"

    libvirt.cpu_mode = "host-model"
    libvirt.nested = true
  end

  nodes.each do |node_name, specs|
    config.vm.define node_name do |node|
      node.vm.hostname = node_name
      node.vm.network "private_network", ip: specs["ip"]

      # Sobrescreve as configurações de memória e CPU para cada VM
      node.vm.provider :libvirt do |libvirt|
        libvirt.memory = specs["memory"]
        libvirt.cpus = specs["cpus"]
      end

      # Adiciona a chave pública se ela não existir
      node.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("./id_ed25519.pub").first.strip
        s.inline = <<-SHELL
          if ! grep -q "#{ssh_pub_key}" /home/vagrant/.ssh/authorized_keys; then
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
            echo "Chave SSH adicionada."
          else
            echo "A chave SSH já existe no arquivo authorized_keys."
          fi
        SHELL
      end
    end
  end
end
