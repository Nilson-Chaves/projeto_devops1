Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true
  end

  config.ssh.insert_key = false
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vbguest.auto_update = false if Vagrant.has_plugin?("vagrant-vbguest")

  nodes = [
    { name: "arq", ip: "192.168.56.102", ram: 512 },
    { name: "db",  ip: nil,            ram: 512 },
    { name: "app", ip: nil,            ram: 512 },
    { name: "cli", ip: nil,            ram: 1024 }
  ]

  nodes.each do |node|
    config.vm.define node[:name] do |node_config|
      node_config.vm.hostname = "#{node[:name]}.nilson.wellington.devops"
      node_config.vm.provider "virtualbox" do |vb|
        vb.memory = node[:ram]

        # Adiciona discos somente para "arq"
        if node[:name] == "arq"
          (1..3).each do |i|
            disk_path = File.expand_path("../../discos/disk#{i}.vdi", __FILE__)
            unless File.exist?(disk_path)
              system("VBoxManage createhd --filename #{disk_path} --size 10240")
            end
            vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", i, "--device", 0, "--type", "hdd", "--medium", disk_path]
          end
        end
      end

      # Define IP fixo para arq, os demais pegam por DHCP
      if node[:ip]
        node_config.vm.network "private_network", ip: node[:ip]
      else
        node_config.vm.network "private_network", type: "dhcp"
      end
    end
  end
end

