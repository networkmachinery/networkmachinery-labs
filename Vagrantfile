# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version '>= 1.6.0'
VAGRANTFILE_API_VERSION = '2'

# Require 'yaml' module
require 'yaml'

# Read YAML file with VM details (box, CPU, RAM, IP addresses)
# Edit machines.yml to change VM configuration details
machines = YAML.load_file(File.join(File.dirname(__FILE__), 'devConfig.yml'))

# Create and configure the VMs
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Iterate through entries in YAML file to create VMs
  machines.each do |machine|
    # Configure the VMs per details in devConfig.yml
    config.vm.define machine['name'] do |srv|
      srv.vm.box_check_update = false
      srv.vm.hostname = machine['name']
      srv.vm.box = machine['box']['vb']

      # Configure default synced folder (disable by default)
      if machine['sync_disabled'] != nil
        srv.vm.synced_folder '.', '/vagrant', disabled: machine['sync_disabled']
      else
        srv.vm.synced_folder '.', '/vagrant', disabled: true
      end #if machine['sync_disabled']

      # Iterate through networks as per settings in machines.yml
      machine['nics'].each do |net|
        if net['ip_addr'] == 'dhcp'
          srv.vm.network net['type'], type: net['ip_addr']
        else
          srv.vm.network net['type'], ip: net['ip_addr']
        end # if net['ip_addr']
      end # machine['nics'].each

      # Configure CPU & RAM per settings in machines.yml (VirtualBox)
      srv.vm.provider 'virtualbox' do |vb, override|
        vb.customize ['modifyvm', :id, '--cableconnected1', 'on']
        vb.memory = machine['ram']
        vb.cpus = machine['vcpu']
        override.vm.box = machine['box']['vb']
      end # srv.vm.provider 'virtualbox'
    end # config.vm.define
  end # machines.each

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "./ansible/provision.yml"
      ansible.config_file = "./ansible/ansible.cfg"
      ansible.inventory_path="./ansible/inventory"
  end
end # Vagrant.configure
