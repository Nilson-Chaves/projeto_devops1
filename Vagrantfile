Vagrant.configure("2") do |config|
  config.vbguest.auto_update = false
  config.vm.box = "debian/bookworm64"
  config.vm.synced_folder "ansible", "/vagrant/ansible"
  config.ssh.insert_key = false

  # Definições das máquinas virtuais
  {
    "arq" => {
      hostname: "arq.nilson.wellington.devops",
      ip: "192.168.56.102",
      ram: 512,
      cpus: 1,
      disks: 3,
      playbooks: ["comum-playbook.yml", "arq-playbook.yml"]
    },
    "db" => {
      hostname: "db.nilson.wellington.devops",
      ip: nil,
      ram: 512,
      cpus: 1,
      playbooks: ["comum-playbook.yml", "db-playbook.yml"]
    },
    "app" => {
      hostname: "app.nilson.wellington.devops",
      ip: nil,
      ram: 512,
      cpus: 1,
      playbooks: ["comum-playbook.yml", "app-playbook.yml"]
    },
    "cli" => {
      hostname: "cli.nilson.wellington.devops",
      ip: nil,
      ram: 1024,
      cpus: 1,
      playbooks: ["comum-playbook.yml", "cli-playbook.yml"]
    }
  }.each do |name, opts|
    config.vm.define name do |node_config|
      node_config.vm.hostname = opts[:hostname]

      # Configuração de rede: IP fixo ou DHCP
      if opts[:ip]
        node_config.vm.network "private_network", ip: opts[:ip]
      else
        node_config.vm.network "private_network", type: "dhcp"
      end

      # Recursos da VM (RAM e CPU)
      node_config.vm.provider "virtualbox" do |vb|
        vb.memory = opts[:ram]
        vb.cpus = opts[:cpus]
        vb.linked_clone = true
      end

      # Discos adicionais para a VM arq (armazenados em .vagrant/)
      if name == "arq" && opts[:disks]
        (1..opts[:disks]).each do |i|
          disk_path = ".vagrant/arq_disk#{i}.vdi"
          unless File.exist?(disk_path)
            system("VBoxManage createmedium disk --filename #{disk_path} --size 10240 --format VDI --variant Standard")
          end
          node_config.vm.provider "virtualbox" do |vb|
            vb.customize [
              "storageattach", :id,
              "--storagectl", "SATA Controller",
              "--port", i,
              "--device", 0,
              "--type", "hdd",
              "--medium", File.expand_path(disk_path)
            ]
          end
        end
      end

      # Provisionamento com Ansible Local (um playbook por máquina)
      opts[:playbooks].each do |playbook|
        node_config.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "/vagrant/ansible/#{playbook}"
        end
      end
    end
  end
end


