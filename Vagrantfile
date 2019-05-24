# -*- mode: ruby -*-
# vi: set ft=ruby :

extras = "
sudo apt update && \
sudo apt install -y python-pip sshpass libkrb5-dev && \
sudo pip install --upgrade pip setuptools wheel && \
sudo pip install \
		ansible==2.7.6 \
    botocore==1.12.86 \
		boto==2.49.0 \
		boto3==1.9.86 \
		awscli==1.16.96 \
    pywinrm[kerberos]==0.3.0 \
    hvac \
    pretty_json \
    twine
cat <<EOF >> /etc/hosts
127.0.0.100     localhost.localdomain
EOF
"

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "ansible", primary: true do |ansible|
    ansible.vm.provision "shell", inline: extras
    ansible.vm.synced_folder "./", "/home/vagrant/src/"
    ansible.vm.provider "docker" do |d|
      d.image = "gbevan/vagrant-ubuntu-dev:bionic"
      d.has_ssh = true
      d.volumes = [
        "/etc/localtime:/etc/localtime:ro",
        "/etc/timezone:/etc/timezone:ro"
      ]
    end
  end
end
