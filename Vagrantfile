# -*- mode: ruby -*-
# # vi: set ft=ruby :

VAGRANT_CONFIGURATION_VERSION = "2"
ENV["LC_ALL"] = "en_US.UTF-8"

require 'yaml'

nodes = YAML.load_file('nodes.yml')
POD_CIDR = '192.168.0.0/16'
KUBETOKEN = ENV['KUBETOKEN'] || 'vucsht.9vg5xomq3lvk0dgc'
MASTER_IP = nodes[0]['ip']
DOCKER_VERSION = '18.09.0'
DEFAULT_STORAGE_PATH = '/var/lib/vdi/home-k8s-setup-1/'

$vagrantfilecommon = File.expand_path('../Vagrantfile.common', __FILE__)
load $vagrantfilecommon

Vagrant.configure(VAGRANT_CONFIGURATION_VERSION) do |config|
    nodes.each do |node_attr|
        config.vm.define node_attr['name'] do |config|
            config.vm.hostname = node_attr['name']
            config.vm.box = node_attr['box']
            config.vm.box_version = node_attr['box_version']
            config.vm.network :private_network, ip: node_attr['ip']

            config.vm.provider "virtualbox" do |box|
                box.name = node_attr[:name]
                box.customize ["modifyvm", :id, "--cpus", node_attr['cpu']]
                box.customize ["modifyvm", :id, "--memory", node_attr['memory']]

                if node_attr['storage']
                    if not File.directory?(DEFAULT_STORAGE_PATH/node_attr['name'])
                        box.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata", "--portcount", node_attr['storage'].size]
                    end
                    node_attr['storage'].each_with_index do |storage_attr, index|
                        filename = "#{DEFAULT_STORAGE_PATH}/#{node_attr['name']}/#{storage_attr['name']}"
                        if not File.exists?(filename)
                            box.customize ["createmedium", storage_attr['type'], "--filename", filename, "--size", storage_attr['size'], "--format", storage_attr['format'], "--variant", storage_attr['variant']]
                        end
                        box.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", index, "--device", 0, "--type", "hdd", "--medium", filename]
                    end
                end 

                box.trigger.after :destroy, :halt do |trigger|
                    trigger.run = {inline: "rm -rf \"#{DEFAULT_STORAGE_PATH}/*\""}
                end
            end

            config.vm.provision "shell", inline: $installDependencies
            config.vm.provision "shell", inline: $configurePostInstall

            if node_attr['master']
                config.vm.provision "shell", inline: $configureMaster
            else
                config.vm.provision "shell", inline: $configureWorker
            end
        end
    end
end
