# -*- mode: ruby -*-
# vi: set ft=ruby :

# load configs
require 'yaml'
current_dir    = File.dirname(File.expand_path(__FILE__))
configs        = YAML.load_file("#{current_dir}/config.yml")
vagrant_config = configs['configs']

# require necessary plugins
# for sshfs on windows, follow:
# https://docs.microsoft.com/de-de/windows-server/administration/openssh/openssh_install_firstuse
required_plugins = %w( vagrant-vbguest )
required_plugins.each do |plugin|
    exec "vagrant plugin install #{plugin};vagrant #{ARGV.join(" ")}" unless Vagrant.has_plugin? plugin || ARGV[0] == 'plugin'
end
# required_plugins = %w( vagrant-vbguest vagrant-sshfs )
# required_plugins.each do |plugin|
#   system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
# end

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/focal64"
  #config.vm.box = "ubuntu/bionic64"

  # set memory to 2048m
  config.vm.provider "virtualbox" do |vb|
    vb.memory = vagrant_config['memory']
    vb.cpus = vagrant_config['cpus']
    vb.customize ["modifyvm", :id, "--audio", "none"]
  end

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "ansible_vagrant", "/vagrant/ansible_vagrant", create: true, owner: "vagrant", group: "vagrant", mount_options: ["dmode=775,fmode=775"]
  config.vm.synced_folder "cdk", "/home/vagrant/cdk", create: true, owner: "vagrant", group: "vagrant", mount_options: ["dmode=775,fmode=775"]
  config.vm.synced_folder "terraform", "/home/vagrant/terraform", create: true, owner: "vagrant", group: "vagrant", mount_options: ["dmode=775,fmode=775"]
  config.vm.synced_folder "testing", "/home/vagrant/testing", create: true, owner: "vagrant", group: "vagrant", mount_options: ["dmode=775,fmode=775"]

  # auto update guest additions
  config.vbguest.auto_update = true

  # provision the vagrant machine using ansible
  # install ansible from its default repo
  config.vm.provision "shell", inline: "test -f .ssh/authorized_keys && cp --preserve=all .ssh/authorized_keys /tmp/authorized_keys" # preserve authorized_keys for `vagrant ssh` in case it already exists
  config.vm.provision "file", source: "~/.ssh", destination: ".ssh" # copy any key from the host; it is not possible to copy them as id_*, so other files might get overriden
  config.vm.provision "shell", inline: "> .ssh/known_hosts" # clean known_hosts file
  config.vm.provision "shell", inline: "mv /tmp/authorized_keys .ssh/" # restore any prior file

  # --- on focal currently no ppa ansible packages exists
  if config.vm.box == "ubuntu/focal64"
    # install ansible from default repo
    config.vm.provision "shell", inline: "apt update && apt install ansible -y"
    config.vm.provision "ansible_local" do |ansible|
      ansible.install = false
      ansible.playbook = "ansible_vagrant/playbook.yml"
      ansible.galaxy_role_file = "ansible_vagrant/requirements.yml"
      ansible.extra_vars = {
        ansible_python_interpreter:"/usr/bin/python3"
      }
    end
  else
    config.vm.provision "ansible_local" do |ansible|
      ansible.install = true
      ansible.install_mode = :default
      ansible.playbook = "ansible_vagrant/playbook.yml"
      ansible.galaxy_role_file = "ansible_vagrant/requirements.yml"
      ansible.extra_vars = {
        ansible_python_interpreter:"/usr/bin/python3"
      }
    end
  end

  config.ssh.forward_agent = true
  config.ssh.forward_x11 = true
end
