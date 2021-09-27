# Ansible setup multi-node Kubernetes cluster in VMs
This repo is used ansible to setup multinodes Kubernetes cluster in virtual machines or bare-metal servers. Tested on Debian / Redhat Family linux server to setup K8S cluster version 1.21.4-0.

## Tested on OS
- Ubuntu 18.04 or higher
- CentOS 7 or higher

## Tools requirement
- Git
- OpenSSH client and SSH Keys for passwordless authentication between ansible host and servers
- Ansible 2.7 or greater
- For local testing in Vagrant or Libvirt-QEMU

## Setup
- Provide username amd nodes DNS/IP address information in ./hosts file.
- Initially set single master to standup the cluster, later join additional master server to cluster
- Single or multiple worker nodes can be added cluster
- Define variable files in ./vars/variables.yaml
    - Kubernetes Version
    - Network interface used for K8S cluster
    - K8S Cluster PODs CIDR Network
    - Non-privilege user to run `kubectl` commands
    - K8S cluster networking flannel or calico
- Run `main-play.yaml` ansible playbook

## Usage
Clone the repo
```sh
git clone https://github.com/m-adnan8080/ansible-k8s-cluster.git
cd ansible-k8s-cluster
```
#### Vagrant with virtualbox
In Virualbox setup two network interface used in VMs. One is vagrant default nat interface and 2nd interface is hostonly interface used for Kubernetes cluster setup. Update the hosts file for IP 192.168.100.0/24 network if using virtualbox driver. Update variables.yaml file for interface name used for kubernetes cluster. Interfaces details are as under:

| OS     | Interface   | Provider   |
|--------|-------------|------------|
| CentOS | eth0        | libvirt    |
| CentOS | eth1        | virtualbox |
| Ubuntu | enp0s3      | libvirt    |
| Ubuntu | enp0s8      | virtualbox |

```sh
cp Vagrantfile-virtualbox Vagrantfile
```

In Vagrant file define the number of nodes, RAM, CPUs and IP address for hostonly interface.
```sh
cluster = {
  "kmaster" => { :ip => "192.168.100.11", :mem => 2048, :cpu => 2 },
  "worker1" => { :ip => "192.168.100.12", :mem => 2048, :cpu => 2 },
  "worker2" => { :ip => "192.168.100.13", :mem => 2048, :cpu => 2 },
}
```
Add your own ssh public-key to `echo` command in Vagrantfile
```
  cat ~/.ssh/id_rsa.pub   #### copy the output and replace ssh-public-key in below line of Vagrantfile ####
  config.vm.provision "shell", inline: "echo 'ssh-public-key' >> /home/vagrant/.ssh/authorized_keys"
```

Start all VMs using command
```sh
vagrant up
```

Check all nodes reacheable to ansible
```sh
ansible -i hosts all -m ping
```
Play `main-play.yaml` playbook to setup Kubernetes cluster
```sh
ansible-playbook -i hosts main-play.yaml
```
After successful run of playbook, it will drop kubeconfig file to the current directory. Export that kubeconfig file and start using `kubectl` commands from your host machine.
```sh
export KUBECONFIG=$PWD/kubeconfig
kubectl get nodes
kubectl get pods -A
```
> Note: kubectl need to installed on local hosts in order to run commands from host machine

#### Vagrant with libvirt-qemu
In Libvirt additional settings are required to setup the environment. Sample command is provided in Vagrantfile to assigne static IP and MAC to VM using `virsh` command. Copy this line as per your cluster node definition and update the information for MAC and IP address for each node respectively. Update the hosts file for IP addresses to 192.168.122.0/24 network if using libvirt driver with default net having IP Address in network 192.168.122.0/24.Update variables.yaml file for interface name used for kubernetes cluster. Interfaces details are as under:

| OS     | Interface   | Provider   |
|--------|-------------|------------|
| CentOS | eth0        | libvirt    |
| CentOS | eth1        | virtualbox |
| Ubuntu | enp0s3      | libvirt    |
| Ubuntu | enp0s8      | virtualbox |

```sh
#system("virsh net-update default add ip-dhcp-host \"<host mac='52:54:00:15:63:c1' ip='192.168.122.11' />\" --live --config")

```
Copy the required vagrantfile.
```sh
cp Vagrantfile-virtualbox Vagrantfile
```

In Vagrant file define the number of nodes, RAM, CPUs and IP address.
```sh
cluster = {
  "kmaster" => { :mgmt_name => "default", :mgmt_nw => "192.168.122.0/24", :mgmt_mac => "52:54:00:15:63:c1", :mem => 2048, :cpu => 2 },
  "worker1" => { :mgmt_name => "default", :mgmt_nw => "192.168.122.0/24", :mgmt_mac => "52:54:00:15:63:c2", :mem => 2048, :cpu => 2 },
  "worker2" => { :mgmt_name => "default", :mgmt_nw => "192.168.122.0/24", :mgmt_mac => "52:54:00:15:63:c3", :mem => 2048, :cpu => 2 },
}

```
> Note: IP Address should match the libvirt default network IP

Add your own ssh public-key to `echo` command in Vagrantfile
```
  cat ~/.ssh/id_rsa.pub   #### copy the output and replace ssh-public-key in below line of Vagrantfile ####
  config.vm.provision "shell", inline: "echo 'ssh-public-key' >> /home/vagrant/.ssh/authorized_keys"
```

Start all VMs using command
```sh
vagrant up
```

Check all nodes reacheable to ansible
```sh
ansible -i hosts all -m ping
```
Play `main-play.yaml` playbook to setup Kubernetes cluster
```sh
ansible-playbook -i hosts main-play.yaml
```
After successful run of playbook, it will drop kubeconfig file to the current directory. Export that kubeconfig file and start using `kubectl` commands from your host machine.
```sh
export KUBECONFIG=$PWD/kubeconfig
kubectl get nodes
kubectl get pods -A
```
> Note: kubectl need to installed on local hosts in order to run commands from host machine
