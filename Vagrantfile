# -*- mode: ruby -*-
# # vi: set ft=ruby :

# Vagrant version and Vagrant API version requirements
Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = "2"

# YAML module for reading box configurations.
require 'yaml'

# Read machine configs from YAML file
machines = YAML.load_file(File.join(File.dirname(__FILE__), 'machines.yml'))

# Create boxes
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Disable updates to keep environment sane.
  config.vm.box_check_update = false

  # Enable shared folder, see https://superuser.com/questions/756758/is-it-possible-to-disable-default-vagrant-synced-folder
  # config.vm.synced_folder 'devsecops/', '/devsecops', type: "nfs", disabled: false
  config.vm.synced_folder 'devsecops/', '/home/vagrant/devsecops', type: 'virtualbox', disabled: false

  # Iterate through entries in YAML file
  machines.each do |machine|
    config.vm.define machine["name"] do |box|
      box.vm.box = machine["box"]
      box.vm.box_version = machine["box_version"]
      box.vm.hostname = machine["name"]
      box.vm.network "private_network", ip: machine["ip"]

      if machine["script"] != nil
        box.vm.provision :shell, :path => machine["script"]
      end

      if machine["ansible"] != nil
        box.vm.provision "ansible" do |ansible|
            ansible.playbook = machine["ansible"]
            ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" }
            ansible.verbose = "vv"
        end
      end

      box.vm.provider :virtualbox do |vb|
        vb.name = machine["name"]
        vb.memory = machine["ram"]

        if machine["gui"] == true
        	vb.gui = true
        end # end of gui

        vb.customize ["modifyvm", :id, "--groups", "/DevSecOps-Studio"]
        vb.customize ["modifyvm", :id, "--vram", "256"]
        vb.customize ["modifyvm", :id, "--graphicscontroller", "vboxsvga"]

      end # end of vb provider
    end # end of box
  end # end of machines loop

  config.vm.provision "shell", inline: <<-SHELL
    DEBIAN_FRONTEND=noninteractive apt-get install -y avahi-daemon libnss-mdns \
    net-tools gdm3 tree wget curl software-properties-common vim
  SHELL

end # end of config
