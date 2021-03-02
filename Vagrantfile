# -*- mode: ruby -*-
# vi: set ft=ruby :

cluster = {
  "k8s-master" => { :ip => "192.168.100.11", :mem => 2048, :cpu => 2 },
  "k8s-node1" => { :ip => "192.168.100.12", :mem => 1024, :cpu => 1 },
  "k8s-node2" => { :ip => "192.168.100.13", :mem => 1024, :cpu => 1 }
}

Vagrant.configure("2") do |config|

  cluster.each_with_index do |(hostname, info), index|

    config.vm.define hostname do |cfg|
      cfg.vm.provider :virtualbox do |vb, override|
        override.vm.box = "centos/8"
        # override.vm.cpus = 2
        override.vm.provision "file", source: "~/.ssh/id_ed25519.pub", destination: "/home/vagrant/id_ed25519.pub"
        override.vm.provision "shell", inline: <<-SHELL
          cat /home/vagrant/id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
          rm /home/vagrant/id_ed25519.pub
        SHELL
        override.vm.network :private_network, ip: "#{info[:ip]}"
        override.vm.hostname = hostname + ".test"
        vb.name = hostname
        vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpu], "--hwvirtex", "on"]
      end # end provider
    end # end config
  end # end cluster

end
