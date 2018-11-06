# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/bionic64"

    config.vm.define "dev", primary: true do |dev|
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
            echo "10.0.10.50 ad-server.internal" >> /etc/hosts

            # Set up SSH
            cp /vagrant/ssh_config ~vagrant/.ssh/config
            ssh-keyscan -t rsa db web github.com > ~vagrant/.ssh/known_hosts

            ## Scan in SSH host keys from servers
            ssh-keyscan -t rsa github.com gitlab.com bitbucket.org >> ~vagrant/.ssh/known_hosts
            ssh-keyscan -t rsa oslic.llnl.gov  >> ~vagrant/.ssh/known_hosts

            # Configure SSH keypairs for the host
            ssh-keygen -t rsa -N "" -f ~vagrant/.ssh/id_rsa
            ssh-keygen -t rsa -N "" -f ~vagrant/.ssh/production
            # ssh-keygen -t rsa -N "" -f ~vagrant/.ssh/builder
            ssh-keygen -t rsa -N "Sup3rS3cr3t" -f ~vagrant/.ssh/aws

            ## Copy new public keys to host
            mkdir -p /vagrant/ssh_keys/
            cp ~vagrant/.ssh/*.pub /vagrant/ssh_keys/

            # Create Git Projects
            mkdir -p ~vagrant/projects/

            git init ~vagrant/projects/cpython
            git init ~vagrant/projects/django

            git init ~vagrant/projects/super_secret_project
            cd ~vagrant/projects/super_secret_project && git remote add origin https://gitlab.internal/r_and_d/super_secret_project.git && cd

            FILE="~vagrant/projects/super_secret_project/super_secret_code.py"
            echo "#! /usr/bin/env python" > $FILE
            echo "" >> $FILE
            echo "print('all the secrets!')" >> $FILE
            echo "" >> $FILE
            echo "import antigravity  # I can fly!" >> $FILE

            FILE="~vagrant/projects/super_secret_project/secrets.yml"
            echo "- db_creds:" > $FILE
            echo "  - username: admin" >> $FILE
            echo "  - password: admin" >> $FILE

            # Create fake bash_history file
            echo "hostname" > ~vagrant/.bash_history
            echo "id" >> ~vagrant/.bash_history
            echo "mkdir projects" >> ~vagrant/.bash_history
            echo "cd projects" >> ~vagrant/.bash_history
            echo "git clone https://gitlab.internal/r_and_d/super_secret_project.git" >> ~vagrant/.bash_history
            echo "vim secrets.yml" >> ~vagrant/.bash_history
            echo "git commit" >> ~vagrant/.bash_history

            # Set up shell environment tokens
            alias token_gen_sha1="dd bs=1024 if=/dev/urandom count=2 2> /dev/null | sha1sum | awk '{print \$1}'"
            alias token_gen_md5="dd bs=1024 if=/dev/urandom count=2 2> /dev/null | md5sum | awk '{print \$1}'"

            ## AWS
            echo "## AWS" >> ~vagrant/.bashrc
            echo "export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE" >> ~vagrant/.bashrc
            echo "export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" >> ~vagrant/.bashrc
            echo "export AWS_DEFAULT_REGION=us-west-2" >> ~vagrant/.bashrc
            echo "" >> ~vagrant/.bashrc

            ## Azure
            echo "## Azure" >> ~vagrant/.bashrc
            echo "export AZURE_TENANT_ID=$(uuidgen)" >> ~vagrant/.bashrc
            echo "export AZURE_CLIENT_ID=$(uuidgen)" >> ~vagrant/.bashrc
            echo "export AZURE_CLIENT_SECRET=$(token_gen_sha1)" >> ~vagrant/.bashrc
            echo "" >> ~vagrant/.bashrc

            ## GitHub
            echo "## GitHub" >> ~vagrant/.bashrc
            echo "export GITHUB_API_TOKEN=$(token_gen_sha1)" >> ~vagrant/.bashrc
            echo "export JEKYLL_GITHUB_TOKEN=\$GITHUB_API_TOKEN" >> ~vagrant/.bashrc
            echo "export HOMEBREW_GITHUB_API_TOKEN=$(token_gen_sha1)" >> ~vagrant/.bashrc
            echo "" >> ~vagrant/.bashrc

            ## GitLab
            echo "## GitLab" >> ~vagrant/.bashrc
            echo "export GITLAB_API_TOKEN=$(token_gen_sha1 | cut -c 1-20)" >> ~vagrant/.bashrc
            echo "" >> ~vagrant/.bashrc

            # Supplement bash_history with mysql commands
            echo "mysql -h db -u root -p" >> ~vagrant/.bash_history
            echo "mysql -h db -u root -proot" >> ~vagrant/.bash_history
            echo "mysql -h db -u root -proot mysql" >> ~vagrant/.bash_history
            echo "mysqldump -h db -u root -p root -A > /tmp/db.sql.backup" >> ~vagrant/.bash_history

            # Fix any wacky file ownership / permissions
            chown -R vagrant:vagrant ~vagrant
            chmod -R go-rwx ~vagrant/.ssh
        SHELL
    end

    config.vm.define "db" do |db|
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
        jenkins.vm.hostname = "jenkins"
        jenkins.vm.network :private_network, ip: "10.0.0.40"

        jenkins.vm.provision "shell", inline: <<-SHELL
            # wget -q -O - https://pkg.jenkins.io/debian/jenkins-ci.org.key | apt-key add -
            # echo "deb https://pkg.jenkins.io/debian-stable binary/" | tee /etc/apt/sources.list.d/jenkins.list

            # apt-get update
            # apt-get install -y jenkins
            useradd -m -p SecretPassword jenkins
            mkdir -p /home/jenkins/.ssh/ && chmod go-rwx /home/jenkins/.ssh/
            cat /vagrant/ssh_keys/id_rsa.pub >> /home/jenkins/.ssh/authorized_keys
        SHELL
    end


end
