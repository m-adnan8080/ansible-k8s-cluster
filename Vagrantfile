ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

cluster = {
  "kmaster" => { :mgmt_name => "default", :mgmt_nw => "192.168.122.0/24", :mgmt_mac => "52:54:00:15:63:c1", :mem => 2048, :cpu => 2 },
  "worker1" => { :mgmt_name => "default", :mgmt_nw => "192.168.122.0/24", :mgmt_mac => "52:54:00:15:63:c2", :mem => 2048, :cpu => 2 },
  "worker2" => { :mgmt_name => "default", :mgmt_nw => "192.168.122.0/24", :mgmt_mac => "52:54:00:15:63:c3", :mem => 2048, :cpu => 2 },
}

#system("virsh net-update default add ip-dhcp-host \"<host mac='52:54:00:15:63:c1' ip='192.168.122.11' />\" --live --config")

Vagrant.configure("2") do |config|

  config.vm.provision "shell", inline: "echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKBYP8WLbAudznHNukAktWyDL1I0N5Wn8hus/re3ncI8' >> /home/vagrant/.ssh/authorized_keys"
  config.vm.synced_folder ".", "/vagrant", disabled: true

  cluster.each_with_index do |(node, info), index|

    if "#{node}" == "kmaster" then
      config.vm.define node do |cfg|
        cfg.vm.box = "centos/7"
        cfg.vm.hostname = node + '.local.test'
        cfg.vm.provider :libvirt do |vm|
          vm.management_network_name = "#{info[:mgmt_name]}"
          vm.management_network_address = "#{info[:mgmt_nw]}"
          vm.management_network_mac = "#{info[:mgmt_mac]}"
          vm.memory = info[:mem]
          vm.cpus = info[:cpu]
        end # end provider
      end # end config
    else
      config.vm.define node do |cfg|
        cfg.vm.box = "centos/7"
        cfg.vm.hostname = node + '.local.test'
        cfg.vm.provider :libvirt do |vm|
          vm.management_network_name = "default"
          vm.management_network_address = "#{info[:mgmt_nw]}"
          vm.management_network_mac = "#{info[:mgmt_mac]}"
          vm.memory = info[:mem]
          vm.cpus = info[:cpu]
        end # end provider
      end # end config
    end # end if
  end # end cluster

end
