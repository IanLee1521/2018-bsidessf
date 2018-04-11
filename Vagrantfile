# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.define "dev", primary: true do |dev|
        dev.vm.box = "ubuntu/xenial64"
        dev.vm.network :private_network, ip: "10.0.0.10"

        dev.vm.provision "shell", inline: <<-SHELL
            apt-get update
            apt-get install -y mysql-client

            # Set up a fake bash history
            cp ~vagrant/.bashrc ~vagrant/.bashrc_orig
            cp /vagrant/bashrc ~vagrant/.bashrc
            cp /vagrant/bash_history ~vagrant/.bash_history

            # Set up /etc/hosts
            echo "10.0.0.20 db" >> /etc/hosts
            echo "10.0.0.30 web" >> /etc/hosts
            echo "10.0.0.40 jenkins" >> /etc/hosts

            # Set up SSH
            cp /vagrant/ssh_config ~vagrant/.ssh/config
            ssh-keyscan -t rsa db web github.com > ~vagrant/.ssh/known_hosts

            # Configure SSH keypairs for the host
            ssh-keygen -t rsa -N "" -f ~vagrant/.ssh/id_rsa
            ssh-keygen -t rsa -N "" -f ~vagrant/.ssh/production
            # ssh-keygen -t rsa -N "" -f ~vagrant/.ssh/builder
            ssh-keygen -t rsa -N "Sup3rS3cr3t" -f ~vagrant/.ssh/aws

            mkdir -p /vagrant/ssh_keys/
            cp ~vagrant/.ssh/*.pub /vagrant/ssh_keys/

            # Create Git Projects
            mkdir -p ~vagrant/projs/
            git clone https://github.com/django/django.git ~vagrant/projs/django
            git clone https://github.com/python/cpython.git ~vagrant/projs/python
            git init ~vagrant/projs/special_project
            FILE="~vagrant/projs/special_project/super_secret_code.py"
            echo "#! /usr/bin/env python" > $FILE
            echo "" >> $FILE
            echo "print('all the secrets!')" >> $FILE
            echo "" >> $FILE
            echo "import antigravity  # I can fly!" >> $FILE

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

            cat /vagrant/ssh_keys/production.pub >> /root/.ssh/authorized_keys
        SHELL
    end

    config.vm.define "web" do |web|
        web.vm.box = "ubuntu/xenial64"
        web.vm.hostname = "web"
        web.vm.network :private_network, ip: "10.0.0.30"

        web.vm.provision "shell", inline: <<-SHELL
            apt-get update
            apt-get install -y apache2

            echo "hello world!" > /var/www/html/index.html
            cat /vagrant/ssh_keys/production.pub >> /root/.ssh/authorized_keys
        SHELL
    end

    config.vm.define "jenkins" do |jenkins|
        jenkins.vm.box = "ubuntu/xenial64"
        jenkins.vm.hostname = "jenkins"
        jenkins.vm.network :private_network, ip: "10.0.0.40"

        jenkins.vm.provision "shell", inline: <<-SHELL
        #     wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | apt-key add -
        #     echo "deb https://pkg.jenkins.io/debian-stable binary/" | tee /etc/apt/sources.list.d/jenkins.list
        #
        #     apt-get update
        #     apt-get install -y jenkins
            useradd -m -p SecretPassword jenkins
            mkdir -p /home/jenkins/.ssh/ && chmod go-rwx /home/jenkins/.ssh/
            cat /vagrant/ssh_keys/id_rsa.pub >> /home/jenkins/.ssh/authorized_keys
        SHELL
    end


end
