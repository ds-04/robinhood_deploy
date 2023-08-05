# Credit to Milosz Galazka  - https://github.com/milosz - this Vagrantfile is based on:

# https://github.com/milosz/vagrant-multiple-disks

# see https://sleeplessbeastie.eu/2021/05/10/how-to-define-multiple-disks-inside-vagrant-using-virtualbox-provider/

#------------------------------------------------------------------------

VIMAGE = "almalinux/8"
#VIMAGE = "centos/7"
#VIMAGE = "opensuse/Leap-15.4.x86_64"
#VIMAGE = "opensuse/Tumbleweed.x86_64"

# Won't work without more work
#VIMAGE = "debian/bookworm64"
#VIMAGE = "generic/ubuntu2204"

NODES = 1

#------------------------------------------------------------------------

require 'fileutils'

# file operations needs to be relative to this file
VAGRANT_ROOT = File.dirname(File.expand_path(__FILE__))

# directory that will contain VDI files
VAGRANT_DISKS_DIRECTORY = "disks"

# controller definition
VAGRANT_CONTROLLER_NAME = "Virtual I/O Device SCSI controller"
VAGRANT_CONTROLLER_TYPE = "virtio-scsi"

# define disks
# The format is filename, size (GB), port (see controller docs)
local_disks = [
  { :filename => "disk1", :size => 10, :port => 10 },
]

Vagrant.configure("2") do |config|

  config.vm.box = VIMAGE

  disks_directory = File.join(VAGRANT_ROOT, VAGRANT_DISKS_DIRECTORY)

  # create disks before "up" action
  config.trigger.before :up do |trigger|
    trigger.name = "Create disks"
    trigger.ruby do
      unless File.directory?(disks_directory)
        FileUtils.mkdir_p(disks_directory)
      end
      local_disks.each do |local_disk|
        local_disk_filename = File.join(disks_directory, "#{local_disk[:filename]}.vdi")
        unless File.exist?(local_disk_filename)
          puts "Creating \"#{local_disk[:filename]}\" disk"
          system("vboxmanage createmedium --filename #{local_disk_filename} --size #{local_disk[:size] * 1024} --format VDI")
        end
      end
    end
  end

  # create storage controller on first run
  unless File.directory?(disks_directory)
    config.vm.provider "virtualbox" do |storage_provider|
      storage_provider.customize ["storagectl", :id, "--name", VAGRANT_CONTROLLER_NAME, "--add", VAGRANT_CONTROLLER_TYPE, '--hostiocache', 'off']
    end
  end

  # attach storage devices
  config.vm.provider "virtualbox" do |storage_provider|
    local_disks.each do |local_disk|
      local_disk_filename = File.join(disks_directory, "#{local_disk[:filename]}.vdi")
      unless File.exist?(local_disk_filename)
        storage_provider.customize ['storageattach', :id, '--storagectl', VAGRANT_CONTROLLER_NAME, '--port', local_disk[:port], '--device', 0, '--type', 'hdd', '--medium', local_disk_filename]
      end
    end
  end


  (1..NODES).each do |machine_id|
    config.vm.define "robinhood#{machine_id}" do |machine|
      machine.vm.hostname = "robinhood.localdomain.example.net"
      machine.vm.network :private_network, ip: "192.168.56.250"
      config.ssh.insert_key = false
        #Run provisioning on all machines AFTERWARDS
        if machine_id == NODES
          machine.vm.provision :ansible do |ansible|
          ansible.verbose = "v"
          #ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python3" } #if forcing python3
          ansible.extra_vars = { ansible_python_interpreter: "auto" }
          ansible.playbook = "playbook.yml"
          ansible.limit = "robinhood_policy_test"
          ansible.inventory_path  = "hosts.ini"   
        end
      end
    end
  end

  # cleanup after "destroy" action
  config.trigger.after :destroy do |trigger|
    trigger.name = "Cleanup operation"
    trigger.ruby do
      # the following loop is now obsolete as these files will be removed automatically as machine dependency
      local_disks.each do |local_disk|
        local_disk_filename = File.join(disks_directory, "#{local_disk[:filename]}.vdi")
        if File.exist?(local_disk_filename)
          puts "Deleting \"#{local_disk[:filename]}\" disk"
          system("vboxmanage closemedium disk #{local_disk_filename} --delete")
        end
      end
      if File.exist?(disks_directory)
        FileUtils.rmdir(disks_directory)
      end
    end
  end
end
