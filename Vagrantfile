# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"

  config.vm.provision "shell", inline: <<-SHELL
    echo "mysql-server-5.7 mysql-server/root_password password root" | debconf-set-selections
    echo "mysql-server-5.7 mysql-server/root_password_again password root" | debconf-set-selections

    apt-get update
    apt-get install -y screen git zsh mysql-client mysql-server nmap

    # Set up a fake bash history
    cp ~vagrant/.bashrc ~vagrant/.bashrc_orig
    cp /vagrant/bashrc ~vagrant/.bashrc
    cp /vagrant/bash_history ~vagrant/.bash_history

    # Set up SSH
    cp /vagrant/ssh_config ~vagrant/.ssh/config

    ssh-keyscan -t rsa localhost github.com gitlab.com > ~vagrant/.ssh/known_hosts

    # Configure SSH keypairs for the host
    ssh-keygen -t rsa -N "" -f ~vagrant/.ssh/id_rsa
    ssh-keygen -t rsa -N "Sup3rS3cr3t" -f ~vagrant/.ssh/production
    ssh-keygen -t rsa -N "Sup3rS3cr3t" -f ~vagrant/.ssh/builder
    ssh-keygen -t rsa -N "Sup3rS3cr3t" -f ~vagrant/.ssh/aws

    # Fix any wacky file ownership
    chown -R vagrant:vagrant ~vagrant
  SHELL
end
