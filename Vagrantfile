# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'erb'

class TemplateRenderer
  def self.empty_binding
    binding
  end

  def self.render(template_content, locals = {})
    b = empty_binding
    locals.each { |k, v| b.local_variable_set(k, v) }
    ERB.new(template_content).result(b)
  end
end

def get_cygpath(path)
  return path.gsub("\\","/").gsub(/(.):/, "/cygdrive/\\1")
end

# Cross-platform way of finding an executable in the $PATH.
#
#   which('ruby') #=> /usr/bin/ruby
def which(cmd)
  exts = ENV['PATHEXT'] ? ENV['PATHEXT'].split(';') : ['']
  ENV['PATH'].split(File::PATH_SEPARATOR).each do |path|
    exts.each { |ext|
      exe = File.join(path, "#{cmd}#{ext}")
      return exe if File.executable?(exe) && !File.directory?(exe)
    }
  end
  return nil
end

def create_ansible_config(hook_options)
  File.write(hook_options[:ansible][:ansible_config_path], TemplateRenderer.render(File.read(hook_options[:ansible][:ansible_config_template_path]), hook_options))
end

def create_ansible_inventory(hook_options)
  File.write(hook_options[:ansible][:ansible_inventory_path], TemplateRenderer.render(File.read(hook_options[:ansible][:ansible_inventory_template_path]), hook_options))
end

def before(hook_options)
  create_ansible_config(hook_options)
  create_ansible_inventory(hook_options)
end

def after(hook_options)
  
end

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
  vagrant_man_network = ENV['vagrant_man_network'] || 'Port1'
  vagrant_lan_network = ENV['vagrant_lan_network'] || 'virtual'

  vagrant_proxy_http = ENV['vagrant_proxy_http'] || 'http://192.168.0.2:3128/'
  vagrant_proxy_https = ENV['vagrant_proxy_https'] || 'http://192.168.0.2:3128/'
  vagrant_no_proxy = ENV['vagrant_no_proxy'] || 'localhost,127.0.0.1'

  empty_box_name = "hyperv_empty"
  empty_box_url = "hyperv_null.box"
  ubuntu_box_name = "ubuntu"
  ubuntu_box_url = "hyperv_ubuntu-18.04_baremetal.box"

  case "#{provider}"
  when 'vsphere'
    empty_box_name = "vsphere_empty"
    empty_box_url = "vsphere_empty.box"
    ubuntu_box_name = "vsphere_dummy"
    ubuntu_box_url = "./vsphere_dummy.box"
  when 'vmware'
    empty_box_name = "vmware_empty"
    empty_box_url = "vmware_empty.box"   
    ubuntu_box_name = "vmware_ubuntu-18.04_baremetal"
    ubuntu_box_url = "./vmware_ubuntu-18.04_baremetal.box"
  when 'virtualbox'
    empty_box_name = "virtualbox_empty"
    empty_box_url = "virtualbox_empty.box" 
    ubuntu_box_name = "virtualbox_ubuntu-18.04_baremetal"
    ubuntu_box_url = "./virtualbox_ubuntu-18.04_baremetal.box"
  when 'hyperv'
    empty_box_name = "hyperv_empty"
    empty_box_url = "hyperv_null.box"
    ubuntu_box_name = "ubuntu"
    ubuntu_box_url = "./hyperv_ubuntu-18.04_baremetal.box"
  else
    abort("Unknown provider: #{provider}")
  end
  
  ansible_environment_state_path = "ansible/"
  if ENV["VAGRANT_DOTFILE_PATH"]
    ansible_environment_state_path = "#{ENV["VAGRANT_DOTFILE_PATH"]}/provisioners/ansible/".gsub("\\","/")
  end
  dir = File.dirname(ansible_environment_state_path)
  FileUtils.mkdir_p(dir) unless File.directory?(dir)  
  
  ansible_inventory_path = "#{ansible_environment_state_path}inventory/vagrant_ansible_inventory"
  dir = File.dirname(ansible_inventory_path)
  FileUtils.mkdir_p(dir) unless File.directory?(dir)
  
  ansible_inventory_template_path = "ansible/vagrant_ansible_inventory.erb"
  ansible_config_template_path = "ansible/ansible.cfg.erb"
  ansible_playbook_path = "ansible/playbook.yml"
  ansible_config_path = "ansible/ansible.cfg"
  ansible_galaxy_role_path = "ansible/requirements.yml"
  ansible_galaxy_roles_path = "ansible/roles"

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

  # unless Vagrant.has_plugin?('vagrant-windows-domain')
  #   system('vagrant plugin install vagrant-windows-domain') || exit!
  #   exit system('vagrant', *ARGV)
  # end

  # unless Vagrant.has_plugin?('vagrant-openssh-passwd')
  #   system('vagrant plugin install vagrant-openssh-passwd') || exit!
  #   exit system('vagrant', *ARGV)
  # end

  # unless Vagrant.has_plugin?('vagrant-proxyconf')
  #   system('vagrant plugin install vagrant-proxyconf') || exit!
  #   exit system('vagrant', *ARGV)
  # end

  # unless Vagrant.has_plugin?('vagrant-cachier')
  #   system('vagrant plugin install vagrant-cachier') || exit!
  #   exit system('vagrant', *ARGV)
  # end
  
  raise "rsync not in path" if which("rsync").nil?
  raise "ansible not in path" if which("ansible").nil?

  #Allow Rsync to work on windows
  ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"

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

  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.ftp      = vagrant_proxy_ftp
    config.proxy.http     = vagrant_proxy_http
    config.proxy.https    = vagrant_proxy_https
    config.proxy.no_proxy = vagrant_no_proxy
  end

  hook_options = {
    :config => config,
    :ansible => {
      :ansible_inventory_path => ansible_inventory_path,
      :ansible_inventory_template_path => ansible_inventory_template_path,
      :ansible_playbook_path => ansible_playbook_path,
      :ansible_config_path => ansible_config_path,
      :ansible_config_template_path => ansible_config_template_path,
      :ansible_galaxy_role_path => ansible_galaxy_role_path,
      :ansible_galaxy_roles_path => ansible_galaxy_roles_path,
      :ansible_cygpath_private_key_path => get_cygpath(Array(config.ssh.private_key_path)[0]),
      :ansible_cygpath_ansible_environment_state_path => get_cygpath(ansible_environment_state_path),
      :ansible_provision_api_user => vagrant_username,
      :ansible_provision_api_password => vagrant_password,
    }
  }

  before(hook_options)

  config.vm.define "digitalrebar" do |digital_rebar|
    digital_rebar.vm.hostname = "digitalrebar"
    digital_rebar.vm.guest = :ubuntu
    digital_rebar.vm.box = ubuntu_box_name
    digital_rebar.vm.box_url = ubuntu_box_url
    digital_rebar.vm.boot_timeout = 600

    digital_rebar.vm.allowed_synced_folder_types = allowed_synced_folder_types
    digital_rebar.vm.synced_folder ".", "/vagrant", type: "rsync", owner: vagrant_username, rsync__exclude: [".git", "*.box", "output-*", "set*.bat", "dr-provision", ".vagrant"]

    digital_rebar.vm.communicator = 'ssh'

    #MAN network - where digital rebar provisioner gui is accessable https://host_ip:8092
    digital_rebar.vm.network :public_network, bridge: vagrant_man_network
    
    #Virtual network - where digital rebar provisioner will manage network booting
    digital_rebar.vm.network :public_network, bridge: vagrant_lan_network, ip: "172.16.0.1", netmask: "24"

    digital_rebar.vm.provider "hyperv" do |hyperv|
      hyperv.vmname = digital_rebar.vm.hostname
      hyperv.cpus = 4
      hyperv.memory = 8192
      hyperv.enable_virtualization_extensions = true
      hyperv.auto_start_action = "StartIfRunning"
      hyperv.auto_stop_action = "Save"
      hyperv.vm_integration_services = {
        guest_service_interface: true,
        heartbeat: true,
        key_value_pair_exchange: true,
        shutdown: true,
        time_synchronization: true,
        vss: true
      }
    end

    #Update DHCP server with new hostname
    digital_rebar.vm.provision "shell", inline: "apt-get -y install python-minimal", run: "always"

    digital_rebar.vm.provision :ansible do |ansible|	  
      #ansible.config_file = ansible_config_path
      ansible.galaxy_role_file = ansible_galaxy_role_path
      ansible.galaxy_roles_path = ansible_galaxy_roles_path
      ansible.inventory_path = ansible_inventory_path
      ansible.playbook = ansible_playbook_path
      ansible.verbose = "vvv"
      ansible.become = true
      ansible.raw_arguments  = ["--user='#{vagrant_username}'", "--private-key='#{config.ssh.private_key_path}'", "--extra-vars='pipelining=True'"]
    end
  end

  config.vm.define "worker" do |worker|
    worker.vm.hostname = :nil
    worker.vm.guest = :linux
    worker.vm.box = empty_box_name
    worker.vm.box_url = empty_box_url
    worker.vm.boot_timeout = 600
    worker.vm.communicator = 'ssh'
    worker.ssh.insert_key = 'false'
    worker.vm.synced_folder ".", "/vagrant", disabled: true

    worker.vm.network :public_network, bridge: vagrant_lan_network, auto_config: false

    worker.vm.provider "hyperv" do |v|
      hyperv.vmname = worker.vm.hostname
      hyperv.cpus = 4
      hyperv.memory = 8192
      hyperv.enable_virtualization_extensions = true
      hyperv.auto_start_action = "StartIfRunning"
      hyperv.auto_stop_action = "Save"
      hyperv.vm_integration_services = {
        guest_service_interface: true,
        heartbeat: true,
        key_value_pair_exchange: true,
        shutdown: true,
        time_synchronization: true,
        vss: true
      }
    end      
  end

  after(hook_options)
end
