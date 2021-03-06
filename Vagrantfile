# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  #########################################################################
  # Configuration settings common to _all_ VMs must be set in this section.
  #########################################################################
  
  # Use our custom base box (generated by packer).
  config.vm.box = "Centos6"
  config.vm.box_url = "CentOS6.box"

  # Set up CPU execution capping and time sync (virtualbox specific).
  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
    v.customize ["guestproperty", "set",
                 :id,
                 "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold",
                 5000]
  end

  # Common provisioners
  config.vm.provision "shell" do |s|
    s.path = "provisioners/vagrant/setup-oracle-jdks"
    s.args = "7u80-b15 8u60-b27"
  end
  
  # Use our custom private key to connect to the VMs
  config.ssh.private_key_path = "unsecure-ssh-keys/unsecure_key"

  ############################################################
  # Configuration of specific VMs must be set in this section.
  ############################################################
  
  # Example server - 10.10.10.3
  config.vm.define "example" do |example|

    # Run this provisioning shell script without params
    #example.vm.provision :shell, path: "provisioners/vagrant-core-setup"

    # Run this provisioning shell script with params
    #example.vm.provision "shell" do |s|
    #  s.path = "provisioners/wildfly/base"
    #  s.args = "8.2.0.Final /opt/oracle-jdk/8-current"
    #end

    # VirtualBox configuration
    example.vm.provider "virtualbox" do |v|
      v.name = "Example - CentOS 6"
      v.memory = 1024
      v.cpus = 2
    end

    example.vm.hostname = "example"
    example.vm.network "private_network", ip: "10.10.10.3"

  end

  # RRR (PHP-MySQL)- 10.10.10.5
  config.vm.define "rrr" do |cfg|

    # Install MySQL
    cfg.vm.provision :shell, path: "provisioners/vagrant/database/setup-mysql-5_6"

    # Install Apache httpd, php and xdebug
    cfg.vm.provision :shell, path: "provisioners/vagrant/php/setup-php-5_6"

    cfg.vm.provision "shell" do |s|
      s.path = "provisioners/guests/rrr/base"
      s.args = "rrr"
    end

    # VirtualBox configuration
    cfg.vm.provider "virtualbox" do |v|
      v.name = "RRR - CentOS 6"
      v.memory = 1024
      v.cpus = 2
    end

    cfg.vm.hostname = "rrr"
    cfg.vm.network "private_network", ip: "10.10.10.5"

  end

end
