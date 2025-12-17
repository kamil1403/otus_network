# -*- mode: ruby -*-
# vi: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "inetRouter",
        :net => [
                   ["192.168.255.1", 2, "255.255.255.252", "router-net"],
                   ["192.168.50.10", 8, "255.255.255.0"],
                ]
  },
  :centralRouter => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "centralRouter",
        :net => [
                   ["192.168.255.2",  2, "255.255.255.252", "router-net"],
                   ["192.168.0.1",    3, "255.255.255.240", "dir-net"],
                   ["192.168.255.9",  6, "255.255.255.252", "office1-central"],
                   ["192.168.255.5",  7, "255.255.255.252", "office2-central"],
                   ["192.168.50.11",  8, "255.255.255.0"],
                ]
  },
  :office1Router => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office1Router",
        :net => [
                   ["192.168.255.10", 2, "255.255.255.252", "office1-central"],
                   ["192.168.2.129",  3, "255.255.255.192", "office1-net"],
                   ["192.168.50.20",  8, "255.255.255.0"],
                ]
  },
  :office2Router => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office2Router",
        :net => [
                   ["192.168.255.6",  2, "255.255.255.252", "office2-central"],
                   ["192.168.1.1",    3, "255.255.255.128", "office2-net"],
                   ["192.168.50.30",  8, "255.255.255.0"],
                ]
  },
  :centralServer => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "centralServer",
        :net => [
                   ["192.168.0.2",    2, "255.255.255.240", "dir-net"],
                   ["192.168.50.12",  8, "255.255.255.0"],
                ]
  },
  :office1Server => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office1Server",
        :net => [
                   ["192.168.2.130",  2, "255.255.255.192", "office1-net"],
                   ["192.168.50.21",  8, "255.255.255.0"],
                ]
  },
  :office2Server => {
        :box_name => "ubuntu/jammy64",
        :vm_name => "office2Server",
        :net => [
                   ["192.168.1.2",    2, "255.255.255.128", "office2-net"],
                   ["192.168.50.31",  8, "255.255.255.0"],
                ]
  }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.hostname = boxconfig[:vm_name]

      boxconfig[:net].each do |ipconf|
        box.vm.network "private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3]
      end

      box.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
        v.name = boxconfig[:vm_name]
        v.customize ["modifyvm", :id, "--ioapic", "on"]
      end
    end
  end
  
  # Provision runs only once after all machines are up
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/provision.yml"
    ansible.inventory_path = "ansible/hosts"
    ansible.host_key_checking = false
    ansible.limit = "all"
  end
end
