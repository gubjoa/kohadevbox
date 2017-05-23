# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'fileutils'

Vagrant.configure(2) do |config|

  # http://fgrehm.viewdocs.io/vagrant-cachier
  if Vagrant.has_plugin?("vagrant-cachier")
    puts "INFO: vagrant-cachier is installed"
    config.cache.scope = :box
    config.cache.synced_folder_opts = {
      type: :nfs,
      # The nolock option can be useful for an NFSv3 client that wants to avoid the
      # NLM sideband protocol. Without this option, apt-get might hang if it tries
      # to lock files needed for /var/cache/* operations. All of this can be avoided
      # by using NFSv4 everywhere. Please note that the tcp option is not the default.
      mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    }
  else
    puts "INFO: vagrant-cachier is not installed"
    puts "Run 'vagrant plugin install vagrant-cachier' to speed up provisioning."
  end

  # Default to host's ansible
  provisioner = :ansible

  config.vm.hostname = "kohadev-deploylab"

  config.vm.define "jessie", primary: true do |jessie|
    jessie.vm.box = "debian/jessie64"
  end

  config.vm.network :forwarded_port, guest: 6001, host: 6008, auto_correct: true  # SIP2
  config.vm.network :forwarded_port, guest: 80,   host: 8008, auto_correct: true  # OPAC
  config.vm.network :forwarded_port, guest: 8080, host: 8608, auto_correct: true  # INTRA
  config.vm.network :forwarded_port, guest: 9200, host: 9208, auto_correct: true  # ES
  config.vm.network "private_network", ip: "192.168.208.18"

  # config.vm.provider :virtualbox do |vb|
  #   if ENV['KOHA_ELASTICSEARCH']
  #     vb.customize ["modifyvm", :id, "--memory", "4096"]
  #   else
  #     vb.customize ["modifyvm", :id, "--memory", "2048"]
  #   end
  # end
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
  end

  config.vm.synced_folder "~/Code/Koha/deploylab-koha", "/home/vagrant/kohaclone", type: "nfs"

  if ENV['PLUGIN_REPO']
    if OS.windows?
      unless Vagrant.has_plugin?("vagrant-vbguest")
        raise 'The vagrant-vbguest plugin is not present, and is mandatory for PLUGIN_REPO on Windows! See README.md'
      end

      config.vm.synced_folder ENV['PLUGIN_REPO'], "/home/vagrant/koha_plugin", type: "virtualbox"

    else
      # We should safely rely on NFS
      config.vm.synced_folder ENV['PLUGIN_REPO'], "/home/vagrant/koha_plugin", type: "nfs"
    end
  end

  config.vm.provision provisioner do |ansible|
    ansible.extra_vars = { ansible_ssh_user: "vagrant", testing: true }
    ansible.extra_vars.merge!({ sync_repo: true });
    ansible.extra_vars.merge!({ elasticsearch: true });
    ansible.playbook = "site.yml"
  end

  config.vm.post_up_message = "Welcome to the kohadev-deploylab box!"

end
