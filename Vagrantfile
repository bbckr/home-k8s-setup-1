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
    nodes.each_with_index do |node_attr, node_index|
        config.vm.define node_attr['name'] do |node_config|
            node_config.vm.hostname = node_attr['name']
            node_config.vm.box = node_attr['box']
            node_config.vm.box_version = node_attr['box_version']
            node_config.vm.network :private_network, ip: node_attr['ip']

            node_config.vm.provider "virtualbox" do |box|
                box.name = node_attr[:name]
                box.customize ["modifyvm", :id, "--cpus", node_attr['cpu']]
                box.customize ["modifyvm", :id, "--memory", node_attr['memory']]

                if node_attr['storage']
                    if not File.directory?("#{DEFAULT_STORAGE_PATH}#{node_attr['name']}")
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
            end

            node_config.vm.provision "shell", inline: $installDependencies
            node_config.vm.provision "shell", inline: $configurePostInstall

            if node_attr['master']
                node_config.vm.provision "shell", inline: $configureMaster
            else
                node_config.vm.provision "shell", inline: $configureWorker
            end

            node_config.trigger.after :destroy do |trigger|
                trigger.run = { inline: "rm -rf #{DEFAULT_STORAGE_PATH}#{nodes[node_index]['name']}" }
            end
        end
    end
end
