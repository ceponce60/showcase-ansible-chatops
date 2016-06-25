# -*- mode: ruby -*-
# vi: set ft=ruby :

# Set Hubot Slack Token here
HUBOT_SLACK_TOKEN = ENV['HUBOT_SLACK_TOKEN'] ? ENV['HUBOT_SLACK_TOKEN'] : ''
ST2VER = ENV['ST2VER'] ? ENV['ST2VER'] : 'stable'
HUBOT_NAME = ENV['HUBOT_NAME'] ? ENV['HUBOT_NAME'] : 'hubot'

VIRTUAL_MACHINES = {
  :web => {
    :ip => '192.168.90.61',
    :hostname => 'web',
  },
  :db => {
    :ip => '192.168.90.62',
    :hostname => 'db',
  },
  :chatops => {
    :ip => '192.168.90.60',
    :hostname => 'chatops',
  },
}

# Autoinstall for vagrant-hostmanager plugin
# See https://github.com/smdahlen/vagrant-hostmanager
unless Vagrant.has_plugin?('vagrant-hostmanager')
  system('vagrant plugin install vagrant-hostmanager') || exit!
  exit system('vagrant', *ARGV)
end

unless HUBOT_SLACK_TOKEN.start_with?('xoxb-')
  puts "Error! HUBOT_SLACK_TOKEN is required."
  puts "Please specify it in your environment, e.g.:"
  puts "export HUBOT_SLACK_TOKEN=xoxb-5187818172-I7wLh4oqzhAScwXZtPcHyxCu"
  exit
end

Vagrant.configure(2) do |config|
  # Global configuration for all boxes
  config.hostmanager.enabled = false
  config.hostmanager.manage_host = true
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

  # VM specific configurations
  VIRTUAL_MACHINES.each do |name, cfg|
    config.vm.define name do |vm_config|
      vm_config.vm.box = 'ubuntu/trusty64'
      vm_config.vm.hostname = cfg[:hostname]
      vm_config.hostmanager.aliases = "www.#{cfg[:hostname]}"
      vm_config.vm.network :private_network, ip: cfg[:ip]

      vm_config.vm.provider :virtualbox do |vb|
        vb.name = cfg[:hostname]
      end

      if Vagrant.has_plugin?('vagrant-cachier')
        vm_config.cache.scope = :box
      end

      vm_config.vm.provision :hostmanager

      if name == :web
        vm_config.vm.provision :shell, :path => "web.sh"
      end

      if name == :db
        vm_config.vm.provision :shell, :path => "db.sh"
      end

      # Additional rules for chatops server
      if name == :chatops
        vm_config.vm.provider :virtualbox do |vb|
          vb.memory = 2048
        end
        # Start shell provisioning for chatops server
        vm_config.vm.provision :shell, :inline => "curl -sSL https://stackstorm.com/packages/install.sh | bash -s -- --user=testu --password=testp"
        vm_config.vm.provision :shell, :inline => "bash '/vagrant/validate.sh'"
        vm_config.vm.provision :shell, :path => "ansible.sh"
        vm_config.vm.provision :shell, :inline => "HUBOT_SLACK_TOKEN=#{HUBOT_SLACK_TOKEN} HUBOT_NAME=#{HUBOT_NAME} bash -c '/vagrant/chatops.sh'"
      end
    end
  end
end
