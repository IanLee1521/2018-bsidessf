# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.define "dev", primary: true do |dev|
        dev.vm.box = "ubuntu/xenial64"
        dev.vm.network :private_network, ip: "10.0.0.10"

        dev.vm.provision "shell", inline: <<-SHELL
            apt-get update -y
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

    config.vm.define "db" do |db|
        db.vm.box = "ubuntu/xenial64"
        db.vm.hostname = "db"
        db.vm.network :private_network, ip: "10.0.0.20"

        db.vm.provision "shell", inline: <<-SHELL
            echo "mysql-server-5.7 mysql-server/root_password password root" | debconf-set-selections
            echo "mysql-server-5.7 mysql-server/root_password_again password root" | debconf-set-selections

            apt-get update
            apt-get install -y mysql-server
        SHELL
    end

    config.vm.define "web" do |web|
        web.vm.box = "ubuntu/xenial64"
        web.vm.hostname = "web"
        web.vm.network :private_network, ip: "10.0.0.20"

        web.vm.provision "shell", inline: <<-SHELL
            apt-get update -y
            apt-get install -y apache2

            echo "hello world!" > /var/www/html/index.html
        SHELL
    end

    config.vm.define "jenkins" do |jenkins|
        jenkins.vm.box = "ubuntu/xenial64"
        jenkins.vm.hostname = "jenkins"
        jenkins.vm.network :private_network, ip: "10.0.0.30"

        jenkins.vm.provision "shell", inline: <<-SHELL
            wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | apt-key add -
            echo "deb https://pkg.jenkins.io/debian-stable binary/" | tee /etc/apt/sources.list.d/jenkins.list

            apt-get update
            apt-get install -y jenkins
        SHELL
    end
end
