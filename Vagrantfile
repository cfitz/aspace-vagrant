# -*- mode: ruby -*-
# vi: set ft=ruby :

git_username = `git config user.name`.chomp
git_email = `git config user.email`.chomp

Vagrant.configure("2") do |config|

  config.vm.define 'solr' do |solr|
    solr.vm.box = 'aspace_solr'
    solr.vm.box_url = "file://dist/solr.box"

    solr.vm.hostname = 'aspacesolrlocal'
    solr.vm.network "private_network", ip: "192.168.40.101"
    solr.vm.synced_folder "dist", "/apps/dist"
    solr.vm.synced_folder '/apps/git/archivesspace-core', '/apps/git/archivesspace-core'

    # start Solr (without SSL)
    solr.vm.provision "shell", privileged: false, inline: <<-SHELL
      /apps/solr/scripts/core.sh /apps/git/archivesspace-core archivesspace
      cd /apps/solr/solr && ./control startnossl
    SHELL
  end

  config.vm.define 'aspace' do |aspace|
    aspace.vm.box = "puppetlabs/centos-7.0-64-puppet"
    aspace.vm.box_version = "1.0.1"

    aspace.vm.hostname = 'aspacelocal'
    aspace.vm.network "private_network", ip: "192.168.40.100"

    aspace.vm.synced_folder "dist", "/apps/dist"
    aspace.vm.synced_folder "/apps/git/aspace-env", "/apps/git/aspace-env"

    aspace.vm.provider "virtualbox" do |vb|
      # Customize the amount of memory on the VM
      vb.memory = "1024"
    end

    aspace.vm.provision 'shell', inline: 'puppet module install puppetlabs-mysql'

    # system packages
    aspace.vm.provision 'puppet', manifest_file: 'aspace.pp'

    # firewall rules
    aspace.vm.provision 'shell', path: 'scripts/openports.sh', args: [8080, 8081, 8089, 8090]

    # configure Git
    aspace.vm.provision 'shell', path: 'scripts/git.sh', args: [git_username, git_email], privileged: false
    # install runtime env
    aspace.vm.provision "shell", path: "scripts/env.sh"

    # Oracle JDK
    aspace.vm.provision 'shell', path: 'scripts/jdk.sh'

    # ArchivesSpace application
    aspace.vm.provision 'shell', path: 'scripts/aspace.sh'

    # server-specific values
    aspace.vm.provision 'file', source: 'files/env', destination: '/apps/aspace/config/env'

    # set up MySQL database
    aspace.vm.provision 'shell', path: 'scripts/database.sh', privileged: false

    # start the service
    aspace.vm.provision 'shell', inline: 'cd /apps/aspace && ./control start', privileged: false
  end
end
