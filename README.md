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

### Creating machines
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
vagrant@ansible:~$ sudo apt-get install ansible
</code>
</pre>
#### 3. Generating a new keypair on the ansible machines
<pre>
<code>
vagrant@ansible:~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
f0:b1:7d:77:13:90:5a:0b:e8:d4:56:d7:11:65:8e:73 vagrant@ansible
The key's randomart image is:
+--[ RSA 2048]----+
|          o .o.+B|
|         o + oo+.|
|      . + . + +.E|
|       o = . . o.|
|        S . . ...|
|           . . ..|
|                 |
|                 |
|                 |
+-----------------+
</code>
</pre>

<pre>
<code>
vagrant@ansible:~$ cat .ssh/id_rsa.pub
</code>
</pre>
Copy the public key.

<pre>
<code>
vagrant@ansible:~$ exit

logout
Connection to 127.0.0.1 closed.
</code>
</pre>

<pre>
<code>
$ vagrant ssh webserver

vagrant@webserver:~$ sudo -s

root@webserver:~# echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDt5Uvt/b32ID8gxCGr+q418Af4Ma83ii1E8bYGqTT3YTo1qMO1YzA/Ah0RzAZdQ39d360m1ygv8+0VzAGyvCI51yyUKC/NILKn0axNEJoX/grkaZ1t5SOke4XelqTC0cu+gau2/bqNNfHuWEpt6rqb//XhcfMsfAXbs1Nj3oIVSPikJGyEeP5E3aCUmUdAU77b5sZDXBwolJCTJk+uQ0k4qDjxEwfgk4Iwex6Y1PoLUKi8BZdQ4CerVWPKwGNJdTNS9vTpsErdCdkQeekCXEnTXPsNen2oL8ytN0GgdyBWOBKnQyV9i05IK94AbCFDSoJk2SQD8UfxmAZiMj5QBMhh vagrant@ansible' > /root/.ssh/authorized_keys
</code>
</pre>

<pre>
<code>
root@webserver:~# exit
exit
vagrant@webserver:~$ exit
logout
Connection to 127.0.0.1 closed.
</code>
</pre>

### Ansible Machine Configuration
<pre>
<code>
$ vagrant ssh ansible
</code>
</pre>

#### 1. Creating inventory
<pre>
<code>
vagrant@ansible:~$ echo '[webservers]'> hosts
vagrant@ansible:~$ echo '192.168.5.100'>> hosts
vagrant@ansible:~$ ls
hosts
vagrant@ansible:~$ cat hosts
[webservers]
192.168.5.100
</code>
</pre>

#### 2. Configuring ssh-agent
_ssh-agent_ sends our _id_rsa_ key (private key) automatically. so we don't have to put it as an argument.

<pre>
<code>
vagrant@ansible:~$ ssh-agent bash
vagrant@ansible:~$ ssh-add .ssh/id_rsa
Identity added: .ssh/id_rsa (.ssh/id_rsa)
</code>
</pre>

You need to execute this for every new session or add it to *.bash_profile*(So that it can be executed whenever you login).
Ansible ping all tests whether all our hosts in our inventory file are reachable.

<pre>
<code>
vagrant@ansible:~$ ansible -i hosts -u root -m ping all
previous known host file not found
The authenticity of host '192.168.5.100 (192.168.5.100)' can't be established.
ECDSA key fingerprint is 36:d5:7a:5f:cd:56:fd:70:fe:01:2d:72:12:8c:cd:81.
Are you sure you want to continue connecting (yes/no)? yes
192.168.5.100 | success >> {
    "changed": false,
    "ping": "pong"
}
</code>
</pre>


<pre>
<code>
vagrant@ansible:~$ exit
exit
vagrant@ansible:~$ exit
logout
Connection to 127.0.0.1 closed.
</code>
</pre>
## Congratulations, We did it !!
