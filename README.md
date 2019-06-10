# Getting Started

## Reference
https://github.com/wardviaene/devops-box

<pre>
  <code>

  Vagrant.configure("2") do |config|

    config.vm.define "webserver" do |webserver|
      webserver.vm.box = "ubuntu/trusty64"
      webserver.vm.network "private_network", ip: "192.168.0.2"
      #webserver.vm.provision "shell", path: "scripts/install.sh"
      webserver.vm.hostname = "webserver"
    end
    config.vm.define "ansible" do |ansible|
      ansible.vm.box = "ubuntu/trusty64"
      ansible.vm.network "private_network", ip: "192.168.0.254"
      ansible.vm.hostname = "ansible"
      # devbox.vm.provider "virtualbox" do |v|
      # 		  v.memory = 4096
      # 		  v.cpus = 2
      # 		end
    end

  end

  </code>
</pre>

## Demo
Provision two machines; _webserver & ansible_

## Start
$ vagrant up ansible
$ vagrant up webserver

## login to ansible
$ vagrant ssh ansible
$ sudo apt-get install ansible

## Generating a new keypair on the ansible machines
$ *ssh-keygen* with no passphrase

This is stored on
_$HOME/.ssh/id_rsa.pub_ --> public key
    - To copy to *.ssh/authorized_keys* on host i.e webserver machine
    - on *host* machine i.e _webserver_, echo 'full content of the public key; id_rsa.pub' > /root/.ssh/authorized_keys
_$HOME/.ssh/id_rsa_ --> private key
    For *ansible* machine
