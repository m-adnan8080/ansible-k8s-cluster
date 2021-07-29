# -*- mode: ruby -*-
# vi: set ft=ruby :

cluster = {
  "kmaster" => { :ip => "192.168.100.11", :mem => 2048, :cpu => 2 },
  "knode1" => { :ip => "192.168.100.12", :mem => 1024, :cpu => 1 },
  "knode2" => { :ip => "192.168.100.13", :mem => 1024, :cpu => 1 }
}

Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"
  config.vm.provision "file", source: "~/.ssh/id_ed25519.pub", destination: "/home/vagrant/id_ed25519.pub"
  config.vm.provision "shell", inline: <<-SHELL
    cat /home/vagrant/id_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
    rm /home/vagrant/id_ed25519.pub
  SHELL

  cluster.each_with_index do |(hostname, info), index|
    config.vm.define hostname do |cfg|
      cfg.vm.provider :virtualbox do |vb, override|
        override.vm.network :private_network, ip: "#{info[:ip]}"
        override.vm.hostname = hostname + ".test"
        vb.name = hostname
        vb.customize ["modifyvm", :id, "--memory", info[:mem], "--cpus", info[:cpu], "--hwvirtex", "on"]
      end # end provider
    end # end config
  end # end cluster

end
