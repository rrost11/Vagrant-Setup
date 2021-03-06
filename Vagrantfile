# -*- mode: ruby -*-
# vi: set ft=ruby :

#############################
# Development Server Setup
#############################

box           = 'ubuntu/trusty64'
vmware_box    = 'boxcutter/ubuntu1404'
parallels_box = 'parallels/ubuntu-14.04'
hostname      = 'development-vm'
ram           = '1024'
num_cpus      = '1'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define hostname do |v|
    v.vm.box = box

    v.vm.hostname = hostname

    v.vm.synced_folder ".", "/vagrant", id: "vagrant-root"

    v.vm.provider :virtualbox do |vb, override|
      vb.name = hostname

      vb.memory = ram
      vb.cpus   = num_cpus

      vb.customize ["modifyvm",             :id, "--natdnshostresolver1", "on"]
      vb.customize ["setextradata",         :id, "--VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
      vb.customize ["guestproperty", "set", :id, "--timesync-threshold",  "1000"]

      override.vm.synced_folder "./", "/vagrant", {
        type: "nfs",
        mount_options: ["nolock,vers=3,tcp,noatime,actimeo=1"]
      }
    end

    # Configuration options for the VMware Fusion provider.
    v.vm.provider :vmware_fusion do |vmw, override|
      vmw.vmx["memsize"]  = ram
      vmw.vmx["numvcpus"] = num_cpus

      override.vm.box = vmware_box

      override.vm.synced_folder ".", "/vagrant", {
        owner: "vagrant",
        group: "www-data"
      }
    end

    # Configuration options for the Parallels provider.
    v.vm.provider :parallels do |p, override|
      p.name = hostname

      p.check_guest_tools  = true
      p.update_guest_tools = true

      p.memory = ram
      p.cpus   = num_cpus

      p.customize ["set", :id, "--longer-battery-life", "off"]

      override.vm.box = parallels_box

      override.vm.synced_folder ".", "/vagrant", {
        owner: "vagrant",
        group: "www-data",
        mount_options: ["dmode=775,fmode=664"]
      }
    end

    v.vm.provision :ansible do |ansible|
      ansible.playbook = "ansible/playbooks/vagrant/init.yml"

      ansible.galaxy_role_file  = "ansible/roles.yml"
      ansible.galaxy_roles_path = "ansible/roles"
    end

    if Vagrant.has_plugin? "vagrant-reload"
      v.vm.provision :reload
    end
  end

  # Plugin specific options. Helpful for development but most likely not necessary for class
  if Vagrant.has_plugin? "vagrant-dnsmasq"
    config.dnsmasq.domain = ".test"

    # overwrite default location for /etc/dnsmasq.conf
    brew_prefix = `/usr/local/bin/brew --prefix`.strip
    config.dnsmasq.dnsmasqconf = brew_prefix + '/etc/dnsmasq.conf'

    # command for reloading dnsmasq after config changes
    config.dnsmasq.reload_command = 'sudo launchctl unload /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist; sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.dnsmasq.plist'
  end

  if Vagrant.has_plugin? "vagrant-cachier"
    config.cache.scope = :box

    config.cache.enable :apt
    config.cache.enable :apt_lists
    config.cache.enable :apt_cacher
    config.cache.enable :composer
    config.cache.enable :bower
    config.cache.enable :npm
    config.cache.enable :gem
  end
end
