# Getting Started

## VagrantFile

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

## Prerequisite
Provision 2 machines; _webserver & ansible_

### Getting Started
#### 1. Start the machines
<pre>
<code>
$ vagrant up ansible
$ vagrant up webserver
</code>
</pre>
#### 2. Login to ansible
<pre>
<code>
$ vagrant ssh ansible
$ sudo apt-get install ansible
</code>
</pre>
#### 3. Generating a new keypair on the ansible machines
<pre>
<code>
$ ssh-keygen
</code>
</pre>

This is stored in
_$HOME/.ssh/id_rsa.pub_ --> public key

Copy the public key to _ssh/authorized_keys_ on the _webserver_. On _webserver_, echo 'full content of the public key; id_rsa.pub' > /root/.ssh/authorized_keys

_$HOME/.ssh/id_rsa_ --> is private key for *ansible* machine
