# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'libvirt'

BOX = "generic/ubuntu1804"
BOX_VERSION = "4.3.12"

REQUIRED_PLUGINS = %w(vagrant-libvirt)

$node_count = 3
$control_plane_count = 1
$subnet = "192.168.122"
$name = "devlab"

$ansible_groups = {
  "master" => [],
  "worker" => [],
  "cluster:children" => ["master", "worker"]
}

exit unless REQUIRED_PLUGINS.all? do |plugin|
  Vagrant.has_plugin?(plugin) || (
    puts "The #{plugin} plugin is required. Please install it with:"
    puts "$ vagrant plugin install #{plugin}"
    false
  )
end

Vagrant.configure("2") do |config|
  config.vm.box = BOX
  config.vm.box_version = BOX_VERSION
  config.vm.box_check_update = false

  config.vm.provider :libvirt do |v|
    v.cpus = 2
    v.memory = 2048
    v.nested = true
    v.cpu_mode = 'host-model'
    v.graphics_type = 'none'
    v.mgmt_attach = false
  end

  (1..$node_count).each do |i|
    if i <= $control_plane_count
      config.vm.define "#{$name}-control-plane-#{i}" do |node|
        node.vm.hostname = "#{$name}-control-plane-#{i}"
        node.vm.network "private_network",
          :ip => "#{$subnet}.#{100 + i}"
        $ansible_groups["master"] += ["#{node.vm.hostname}"]
      end
    else
      j = i - $control_plane_count
      config.vm.define "#{$name}-worker-#{j}" do |node|
        node.vm.hostname = "#{$name}-worker-#{j}"
        node.vm.network "private_network",
          :ip => "#{$subnet}.#{200 + j}"
        $ansible_groups["worker"] += ["#{node.vm.hostname}"]

        if i == $node_count
          node.vm.provision :ansible do |a|
            a.compatibility_mode = "2.0"
            a.verbose = "true"
            a.playbook = "site.yml"
            a.groups = $ansible_groups
            a.limit = "all,localhost"
            a.raw_arguments = ["--forks=#{$node_count}", "--flush-cache", "-e ansible_become_pass=vagrant"]
          end
        end
      end
    end
  end
end
