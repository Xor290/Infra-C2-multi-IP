ENV['VAGRANT_DEFAULT_PROVIDER'] = 'docker'

Vagrant.configure("2") do |config|
  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.ssh.username = "cgi"
  config.vm.network :private_network, type: "dhcp", docker_network__internal: true, docker_network: "c2_network"

  
  c2_nodes = [
    { name: "c2-1", host_port: 8081, ssh_port: 2226 },
    { name: "c2-2", host_port: 8082, ssh_port: 2223 }
  ]

  postgres_nodes = [
    { name: "pg-1", pg_port: 5432, data: "pg-1", ssh_port: 2225 },
    { name: "pg-2", pg_port: 5433, data: "pg-2", ssh_port: 2224 }
  ]

  
  inventory_dir = File.join(Dir.pwd, "inventory")
  Dir.mkdir(inventory_dir) unless Dir.exist?(inventory_dir)
  hosts_file = File.join(inventory_dir, "hosts.ini")

  File.open(hosts_file, "w") do |f|
    f.puts "[c2_cluster]"
    c2_nodes.each do |node|
      f.puts "#{node[:name]} ansible_host=127.0.0.1 ansible_port=#{node[:ssh_port]} ansible_user=cgi ansible_ssh_private_key_file=/home/cgi/.vagrant.d/insecure_private_key"
    end

    f.puts "\n[postgres_cluster]"
    postgres_nodes.each do |node|
      f.puts "#{node[:name]} ansible_host=127.0.0.1 ansible_port=#{node[:ssh_port]} ansible_user=cgi ansible_ssh_private_key_file=/home/cgi/.vagrant.d/insecure_private_key"
    end
  end
  puts "✅ Fichier hosts.ini généré : #{hosts_file}"


  c2_nodes.each do |node_info|
    config.vm.define node_info[:name] do |node|
      node.vm.provider :docker do |docker|
        docker.image = "nexus-pic2.support-ent.fr/vagrant/alma9-box"
        docker.name  = node_info[:name]
        docker.ports = [
          "#{node_info[:host_port]}:80",  
          "#{node_info[:ssh_port]}:22"    
        ]
        docker.remains_running = true
        docker.has_ssh = true
        docker.privileged = true
      end
    end
  end

  postgres_nodes.each do |node_info|
    config.vm.define node_info[:name] do |node|
      node.vm.provider :docker do |docker|
        docker.image = "nexus-pic2.support-ent.fr/vagrant/alma9-box"
        docker.name  = node_info[:name]
        docker.ports = [
          "#{node_info[:pg_port]}:5432",
          "#{node_info[:ssh_port]}:22"
        ]
        docker.remains_running = true
        docker.has_ssh = true
        docker.privileged = true
        docker.volumes = [
          "#{Dir.pwd}/data/#{node_info[:data]}:/var/lib/postgresql/data"
        ]
      end
    end
  end


  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.compatibility_mode = "2.0"
    ansible.config_file = "ansible.cfg"
    ansible.verbose = "v"
    ansible.inventory_path = hosts_file
  end
end
