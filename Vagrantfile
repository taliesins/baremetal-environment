# -*- mode: ruby -*-
# vi: set ft=ruby :


# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|  
  if ARGV[1] and (ARGV[1].split('=')[0] == "--provider")
    provider = (ARGV[1].split('=')[1]).to_sym
  elsif ARGV[2] and (ARGV[2].split('=')[0] == "--provider")
    provider = (ARGV[2].split('=')[1]).to_sym
  elsif ARGV[3] and (ARGV[3].split('=')[0] == "--provider")
    provider = (ARGV[3].split('=')[1]).to_sym  
  elsif ARGV[4] and (ARGV[4].split('=')[0] == "--provider")
    provider = (ARGV[4].split('=')[1]).to_sym
  else
    provider = (ENV['VAGRANT_DEFAULT_PROVIDER'] || 'hyperv').to_sym
  end

  # Vagrant environment settings 
  vagrant_username = ENV['vagrant_username'] || 'vagrant'
  vagrant_password = ENV['vagrant_password'] || 'vagrant'
  vagrant_network = ENV['vagrant_network'] || 'Port1'

  ubuntuBoxName = "ubuntu"
  ubuntuBoxUrl = "hyperv_ubuntu-16.04_chef.box"

  case "#{provider}"
  when 'vsphere'
    ubuntuBoxName = "vsphere_dummy"
    ubuntuBoxUrl = "./vsphere_dummy.box"
  when 'vmware'
    ubuntuBoxName = "vmware_ubuntu-16.04_chef"
    ubuntuBoxUrl = "./vmware_ubuntu-16.04_chef.box"
  when 'virtualbox'
    windowsBoxName = "virtualbox_ubuntu-16.04_chef"
    windowsBoxUrl = "./virtualbox_ubuntu-16.04_chef.box"
  when 'hyperv'
    windowsBoxName = "ubuntu"
    windowsBoxUrl = "./hyperv_ubuntu-16.04_chef.box"    
  else
    abort("Unknown provider: #{provider}")
  end

  berkshelfPath = "chef/Berksfile"
  cookBookPath = "chef/cookbooks"
  cookBookRolePath = "chef/roles"
  cookBookDataBagsPath = "chef/data_bags"

  # Set the default synced_folder implementation priority
  allowed_synced_folder_types =  ['rsync']
  Vagrant.plugin("2").manager.registered.each do |plugin|
    plugin.components.synced_folders.each do |key, data|
      case "#{key}"
      when 'docker'
        data[1] = 11  
      when 'virtualbox'
        data[1] = 10
      when 'rsync'
        data[1] = 9
      when 'nfs'
        data[1] = 8
      when 'smb'
        data[1] = 7
      else
        data[1] = 5
      end
    end
  end

  unless Vagrant.has_plugin?('vagrant-berkshelf')
    system('vagrant plugin install vagrant-berkshelf') || exit!
    exit system('vagrant', *ARGV)
  end

  # unless Vagrant.has_plugin?('vagrant-windows-domain')
  #   system('vagrant plugin install vagrant-windows-domain') || exit!
  #   exit system('vagrant', *ARGV)
  # end

  # unless Vagrant.has_plugin?('vagrant-openssh-passwd')
  #   system('vagrant plugin install vagrant-openssh-passwd') || exit!
  #   exit system('vagrant', *ARGV)
  # end    

  #Allow Rsync to work on windows
  ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"

  config.berkshelf.enabled = true
  config.berkshelf.berksfile_path = berkshelfPath

  config.winrm.username = vagrant_username
  config.winrm.password = vagrant_password

  config.ssh.username = vagrant_username
  config.ssh.password = vagrant_password
  config.ssh.private_key_path = [File.expand_path('vagrant_rsa')]
  config.ssh.insert_key = false

  config.ssh.default.username = vagrant_username
  config.ssh.default.password = vagrant_password
  config.ssh.default.private_key_path = [File.expand_path('vagrant_rsa')]
  config.ssh.default.insert_key = false

  config.winrm.retry_limit = 6
  config.winrm.timeout = 120

  config.vm.define "digitalrebar" do |digital_rebar|
    digital_rebar.vm.hostname = "digital-rebar"
    digital_rebar.vm.guest = :ubuntu
    digital_rebar.vm.box = ubuntuBoxName
    digital_rebar.vm.box_url = ubuntuBoxUrl
    digital_rebar.vm.boot_timeout = 600

    digital_rebar.vm.allowed_synced_folder_types = allowed_synced_folder_types
    digital_rebar.vm.synced_folder ".", "/vagrant", type: "rsync", owner: vagrant_username, rsync__exclude: [".git", "*.box", "output-*", "set*.bat"]

    digital_rebar.vm.communicator = 'ssh'

    # Create a public network, which generally matched to bridged network.
    # Bridged networks make the machine appear as another physical device on
    # your network.
    digital_rebar.vm.network "public_network", bridge: vagrant_network

    digital_rebar.vm.provider "hyperv" do |hyperv|
      hyperv.vmname = digital_rebar.vm.hostname
      hyperv.cpus = 4
      hyperv.memory = 2048
      hyperv.enable_virtualization_extensions = true
      hyperv.auto_start_action = :StartIfRunning
      hyperv.auto_stop_action = :Save
      hyperv.vm_integration_services = {
        guest_service_interface: true,
        heartbeat: true,
        key_value_pair_exchange: true,
        shutdown: true,
        time_synchronization: true,
        vss: true
      }
    end

    digital_rebar.vm.provision :chef_solo do |chef|
      chef.node_name = digital_rebar.vm.hostname
      chef.cookbooks_path = cookBookPath
      chef.roles_path = cookBookRolePath
      chef.data_bags_path = cookBookDataBagsPath
      chef.add_recipe 'role-digitalrebar'
      chef.json = {  
      }
    end
  end
end
