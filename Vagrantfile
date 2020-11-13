# -*- mode: ruby -*-
# vi: set ft=ruby :
# Based on https://github.com/UtahDave/salt-vagrant-demo
require 'yaml'

# Use Python 3 by default
# Python 2 based salt packages are not available for all OS, i.e.
# for Debian 10 only Python 3 packages available in official repo
PYTHON = ENV.fetch("PYTHON", "3")
# Load template project by default (i.e. if we in devkit root)
if File.exist?('Vagrantfile')
  BASE = Dir.pwd + "/template"
else
  # load BASE from environment or use current working dir by default
  BASE = ENV.fetch("BASE", Dir.pwd)
end
puts "Using '#{BASE}' as project root"

CONFIG_FILE = "#{BASE}/config.yaml"
puts "Loading config: #{CONFIG_FILE}"
if File.exist?(CONFIG_FILE)
  CONFIG = YAML.load(File.read(CONFIG_FILE))
else
  puts "`#{CONFIG_FILE}` does not exist, no configuration loaded"
  CONFIG = {}
end

Vagrant.configure("2") do |config|
  net_ip = CONFIG.fetch("net_ip", "192.168.150")
  install_type = CONFIG.fetch("install_type", "stable")

  ## Salt Master ##
  config.vm.define :master, primary: true do |master_config|
    cfg = CONFIG.fetch("master", {})
    ip = cfg.fetch("ip", "10")
    # Minimum required synced folders
    synced_folders = [
      ["./provision/etc/salt/master.p", "/etc/salt/master.p"],
      ["./provision/etc/salt/minion.p", "/etc/salt/minion.p"],
      ["./salt/pillar", "/srv/salt/pillar"],
      ["./salt/state",  "/srv/salt/state"],
    ]
    # Additional synced folders from config file
    if cfg.fetch("synced_folders", false)
      synced_folders = synced_folders + cfg.fetch("synced_folders")
    end
    # Salt Master VM properties
    master_config.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 1
      vb.name = "saltmaster"
    end
    master_config.vm.box = cfg.fetch("os", "bento/debian-10")
    master_config.vm.host_name = cfg.fetch("host_name", "saltmaster.local")
    master_config.vm.network "private_network", ip: "#{net_ip}.#{ip}"
    # Sync folders
    synced_folders.each do |src,dst|
      master_config.vm.synced_folder "#{BASE}/#{src}", dst
    end
    # Install salt packages using salt provisioner
    master_config.vm.provision :salt do |salt|
      salt.install_type = install_type
      salt.python_version = PYTHON
      salt.install_master = true
      salt.verbose = true
      salt.colorize = true
      salt.master_config = "#{BASE}/provision/etc/salt/master"
      salt.minion_config = "#{BASE}/provision/etc/salt/minion"
    end
    # Restart salt master after all folders are synced
    master_config.vm.provision "shell",
      run: "always",
      inline: "systemctl restart salt-master; systemctl restart salt-minion"
  end

  ## Salt Minions ##
  minions = CONFIG.fetch("minions", [["minion-debian10", "254", "1024", "bento/debian-10"]])
  minions.each do |vmname,ip,mem,os|
    config.vm.define "#{vmname}" do |minion_config|
      # Minimum required synced folders
      synced_folders = [
        ["./provision/etc/salt/minion.p", "/etc/salt/minion.p"]
      ]
      # Salt Minion VM properties
      minion_config.vm.provider "virtualbox" do |vb|
        vb.memory = "#{mem}"
        vb.cpus = 1
        vb.name = "#{vmname}"
      end
      minion_config.vm.box = "#{os}"
      minion_config.vm.hostname = "#{vmname}"
      minion_config.vm.network "private_network", ip: "#{net_ip}.#{ip}"
      # Sync folders
      synced_folders.each do |src,dst|
        minion_config.vm.synced_folder "#{BASE}/#{src}", dst
      end
      # Install salt packages using salt provisioner
      minion_config.vm.provision :salt do |salt|
        salt.install_type = install_type
        salt.python_version = PYTHON
        salt.verbose = true
        salt.colorize = true
        salt.minion_config = "#{BASE}/provision/etc/salt/minion"
      end
      # Restart salt minions after all folders are synced
      minion_config.vm.provision "shell",
        run: "always",
        inline: "systemctl restart salt-minion"
    end
  end

end
