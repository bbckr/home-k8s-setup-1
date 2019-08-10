# -*- mode: ruby -*-
# # vi: set ft=ruby :

Vagrant.require_version "==2.0.2"
VAGRANT_CONFIGURATION_VERSION = "2"
ENV["LC_ALL"] = "en_US.UTF-8"

require 'yaml'

nodes = YAML.load_file('nodes.yml')
POD_CIDR = '192.168.0.0/16'
KUBETOKEN = ENV['KUBETOKEN'] | 'vucsht.9vg5xomq3lvk0dgc'
MASTER_IP = nodes[0].ip

$vagrantfilecommon = File.expand_path('./Vagrantfile.common', __FILE__)
load $vagrantfilecommon

Vagrant.configure(VAGRANT_CONFIGURATION_VERSION) do |config|
    nodes.each do |node_attr|
        config.vm define node_attr[:name] do |config|
            config.vm.hostname = node_attr[:name]
            config.vm.box = node_attr[:name]
            config.vm.box_version = node_attr[:box_version]
            config.vm.network :private_network, ip: node_attr[:ip]

            config.vm.provider "virtualbox" do |box|
                box.name = node_attr[:name]
                box.customize ["modifyvm", :id, "--cpus", node_attr[:cpu]]
                box.customize ["modifyvm", :id, "--memory", node_attr[:memory]]
            end

            config.vm.provision "shell", inline: $installDependencies
            config.vm.provision "shell", inline: $configurePostInstall

            if node_attr[:master]
                config.vm.provision "shell", inline: $configureMaster
            else
                config.vm.provision "shell", inline: $configureWorker
            end
        end
    end
end
