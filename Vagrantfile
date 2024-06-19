# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

BOX = "generic/ubuntu1804"
BOX_VERSION = "4.3.12"

node_count = 3
control_plane_count = 1
subnet = "192.168.56"
name = "devlab"

ansible_groups = {
  "master" => [],
  "worker" => [],
  "cluster:children" => ["master", "worker"]
}

Vagrant.configure("2") do |config|
  config.vm.box = BOX
  config.vm.box_version = BOX_VERSION
  config.vm.box_check_update = false

  config.vm.provider :libvirt do |v|
    v.cpus = 2
    v.memory = 2048
    v.nested = true
  end

  (1..node_count).each do |i|
    if i <= control_plane_count
      config.vm.define "#{name}-control-plane-#{i}" do |config|
        config.vm.hostname = "#{name}-control-plane-#{i}"
        config.vm.network "private_network", ip: "#{subnet}.#{100 + i}"
        ansible_groups["master"] += ["#{config.vm.hostname}"]
      end
    else
      j = i - control_plane_count
      config.vm.define "#{name}-worker-#{j}" do |config|
        config.vm.hostname = "#{name}-worker-#{j}"
        config.vm.network "private_network", ip: "#{subnet}.#{200 + j}"
        ansible_groups["worker"] += ["#{config.vm.hostname}"]
      end
    end

    if i == node_count
      puts i, node_count
      config.vm.provision :ansible do |a|
        a.compatibility_mode = "2.0"
        a.verbose = "true"
        a.playbook = "site.yml"
        a.groups = ansible_groups
        a.limit = "all"
      end
      break
    end
  end
end
