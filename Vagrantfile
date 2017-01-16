# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

#Vagrant.require_version ">= 1.8.0"
if `vagrant --version` < 'Vagrant 1.8.0'
    abort('Your Vagrant is too old. Please install at least 1.8.0')
end

[
  { :name => "vagrant-hostsupdater", :version => ">= 1.0.2" },
  { :name => "vagrant-triggers", :version => ">= 0.5.3" },
  { :name => "vagrant-vbguest", :version => ">= 0.13.0"}
].each do |plugin|

  if not Vagrant.has_plugin?(plugin[:name], plugin[:version])
    raise "#{plugin[:name]} #{plugin[:version]} is required. Please run `vagrant plugin install #{plugin[:name]}`"
  end
end


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  #config.vm.box = "precise32"
  #config.vm.box_url = "http://files.vagrantup.com/precise32.box"
  # Lets use a better starting box...
  config.vm.box = "bento/ubuntu-12.04-i386"

  config.vm.hostname = 'vip.local'
  config.vm.network :private_network, ip: "10.86.73.80"

  # Ensure SSH port forwarding gets setup right
  # @see https://realguess.net/2015/10/06/overriding-the-default-forwarded-ssh-port-in-vagrant/
  config.vm.network :forwarded_port, guest: 22, host: 2222, id: "ssh", disabled: true
  config.vm.network :forwarded_port, guest: 22, host: 2299, auto_correct: true

  # Not sure why this was ever an issue - must have been vagrant/virtualbox version related.
  #config.ssh.insert_key = false #see https://github.com/Automattic/vip-quickstart/issues/502
  #config.ssh.username = "vagrant"
  #config.ssh.password = "vagrant"

  # Virtualbox overrides
  config.vm.provider "virtualbox" do |v|
    # Use 1GB of memory
    v.memory = 1024

    # Use 2 CPUs
    v.cpus = 2

    # Lets not peg host cpu
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
    # Speed up disk io
    v.customize ["storagectl", :id, "--name", "SATA Controller", "--hostiocache", "on"]

    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
    v.customize ["modifyvm", :id, "--nictype1", "Am79C973"]
  end

  # VMWare Fusion overrides
  config.vm.provider "vmware_fusion" do |v|
    # Use 1GB of memory in vmware_fusion
    v.memory = 1024

    v.vm.box = "precise64-vmware"
    v.vm.box_url = "http://files.vagrantup.com/precise64_vmware.box"
  end

  config.vm.synced_folder ".", "/srv", owner: 'www-data', group: 'www-data', mount_options: ["dmode=775", "fmode=664"]

  # Address a bug in an older version of Puppet
  # See http://stackoverflow.com/questions/10894661/augeas-support-on-my-vagrant-machine
  config.vm.provision :shell, :inline => "if ! dpkg -s puppet > /dev/null; then sudo apt-get update --quiet --yes && sudo apt-get install puppet --quiet --yes; fi"

  # Provision everything we need with Puppet
  config.vm.provision :puppet do |puppet|
    puppet.module_path = "puppet/modules"
    puppet.manifests_path = "puppet/manifests"
    puppet.manifest_file  = "init.pp"
    puppet.options = ['--templatedir', '/srv/puppet/files']
    puppet.facter = {
      "quickstart_domain" => 'vip.local',
    }
  end

end
