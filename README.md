# Vagrant Development Environment Setup

This repository contains the sources required to build and setup one or
more virtual machines that can be used for development purposes.

**Important:** all commands in this README are intended to be executed
from the root of this repository.

## Build Prerequisites

Next to VirtualBox, you should have installed both packer and vagrant
on your workstation.

Packer can be downloaded from [the packer.io web
site](https://www.packer.io/downloads.html). Vagrant can be obtained
from [the vagrant web site](https://www.vagrantup.com/downloads.html).

## Build Procedure

### Build the Base Box

    $ packer build CentOS6.json

This step will first download the ISO file of CentOS. Based on this
iso file, a virtual machine is created and customised for use with
vagrant. The install procedure is based on kickstart (see
`kickstart/ks.cfg`), with some provisioning scripts thrown in (see
`provisioners/packer-*`).

You shouldn't have to do anything except executing the command
above. Downloading the ISO file and going through the installation
procedure may take more than ten minutes, so this is the moment where
you go and brew a cup of tea.

### Prepare Vagrant for the Use of Shared Folders

The folders you will share between your host and the virtual machine
have a location that is specific to your own workstation. Therefore,
we do not include them in the Vagrant file of this repository.

To make shared folders work in a painless way, create a Vagrantfile in
the `.vagrant.d` directory right below your home directory (mine was
`/Users/standgr/.vagrant.d`) with these contents:

    # User defaults
    Vagrant.configure(2) do |config|
      config.vm.synced_folder "/Users/standgr/repositories/", "/srv/src"
    end

Do replace the first path with the host folder you want to share and
the second path with the path you want to find the directory tree on
your virtual machine. You can add as many shared folders as you like.

**Important:** Use an absolute path to the folder you want to share.

### Launch a Vagrant Instance for the First Time

Upon finishing the build of the base box, you will see a file called
`CentOS6.box` in your repository. That file is the basis
upon which vagrant builds your developer work station(s).

Note: everytime you change something in the box configuration (i.e
every time you ask packer to rebuild the box) you should trigger
vagrant to make use of the newly developed box. The easiest way of
doing this is:

    $ vagrant box list
    Centos6 (virtualbox, 0)
    $ vagrant box remove Centos6
    Removing box 'Centos6' (v0) with provider 'virtualbox'...

TODO: by introducing versioning of the box files, this process could
be improved.

The actual provisioning of the developer work station is run through
shell scripts on the virtual machine itself.

Again, a single command does the job:

    # Start up ALL virtual machines defined in the Vagrantfile
    $ vagrant up

    # OR
    
    # Start up the virtual machine with the given name
    $ vagrant up <vm-name>

To access your brand new development server(s):

    $ vagrant ssh <vm-name>

Port forwarding to required services has been set up in vagrant and
should work out of the box.

**Important:** If the configured ports are still in use on the host
machine, the server will not be able to startup.

## Working with Vagrant

As we saw before, SSH'ing to your new development server is a matter
of launching a single command. Vagrant takes care of the
authentication details (through the use of unsecure SSH keys).

    $ vagrant ssh <vm-name>

Stopping your development servers is as simple as executing

    # Halt all running virtual machines
    $ vagrant halt
    
    # Halt the virtual machine with the given name
    $ vagrant halt <vm-name>

## Maintaining your virtual machines

As the build procedure should be fully automated, there is little
point in manually maintaining the individual vagrant boxes. If you
stumble upon something not already automated, please automate it.

That said, upon upgrading your VirtualBox, the version of the guest
additions on the host will not match with the version inside the
guest. To automagically check for and remedy that condition, you can
use a vagrant plugin `vagrant-vbguest`. Sources are on the [github
vagrant-vbguest project
page](https://github.com/dotless-de/vagrant-vbguest)

To install the plugin:

    $ vagrant plugin install vagrant-vbguest

That command should be executed on the host, not on the guest.

If you want to update or re-install the virtual machines (for example 
because the provisioning scripts were updated), you have two options:

    # Start the server and re-run the provisioning scripts
    # Note: Depending on the scripts, this might not update/reinstall everything
    $ vagrant up <vm-name> --provision

    # Completely delete the virtual machines and reinstall them from scratch
    $ vagrant destroy
    $ vagrant up
