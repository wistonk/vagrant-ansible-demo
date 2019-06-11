# Vagrant Provisioning with Ansible

## VagrantFile

<pre>
  <code>
  Vagrant.configure("2") do |config|
    config.vm.define "webserver" do |webserver|
      webserver.vm.box = "ubuntu/trusty64"
      webserver.vm.network "private_network", ip: "192.168.5.100"
      #webserver.vm.provision "shell", path: "scripts/install.sh"
      webserver.vm.hostname = "webserver"
    end
    config.vm.define "ansible" do |ansible|
      ansible.vm.box = "ubuntu/trusty64"
      ansible.vm.network "private_network", ip: "192.168.2.200"
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
##### 1. Start the machines
<pre>
<code>
$ vagrant up ansible
$ vagrant up webserver
</code>
</pre>
##### 2. Login to ansible
<pre>
<code>
$ vagrant ssh ansible
vagrant@ansible:~$ sudo apt-get install ansible
</code>
</pre>
##### 3. Generating a new keypair on the ansible machines
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

##### 1. Creating inventory
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

##### 2. Configuring ssh-agent
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

### Ansible Playbooks
<pre>
<code>
$ vagrant ssh ansible
</code>
</pre>

##### Create a _nginx.yml_ file and add below content.
 <pre>
 <code>
 ---
- hosts: webservers
  vars:
    user: www-data
    worker_processes: 2
    pid: /run/nginx.pid
    worker_connections: 768
  tasks:
  - name: install nginx
    apt: name=nginx state=latest update_cache=yes
  - name: ensure nginx is running (and enable it at boot)
    service: name=nginx state=started enabled=yes
  - name: write the nginx config file
    template: src=templates/nginx.conf.j2 dest=/etc/nginx/nginx.conf
    notify:
    - restart nginx
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
 </code>
 </pre>

##### Create a _templates/nginx.conf.j2_ file and add below content.

<pre>
<code>
user {{ user }};
worker_processes {{ worker_processes }};
pid {{ pid }};

events {
	worker_connections {{ worker_connections }} ;
}

http {

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	gzip on;
	gzip_disable "msie6";

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
</code>
</pre>

<pre>
<code>
vagrant@ansible:~$ ls
hosts  nginx.yml  templates
</code>
</pre>

##### Execute the playbook
<pre>
<code>
vagrant@ansible:~$ ansible-playbook -i hosts -u root nginx.yml

PLAY [webservers] *************************************************************

GATHERING FACTS ***************************************************************
ok: [192.168.5.100]

TASK: [install nginx] *********************************************************
changed: [192.168.5.100]

TASK: [ensure nginx is running (and enable it at boot)] ***********************
ok: [192.168.5.100]

TASK: [write the nginx config file] *******************************************
changed: [192.168.5.100]

NOTIFIED: [restart nginx] *****************************************************
changed: [192.168.5.100]

PLAY RECAP ********************************************************************
192.168.5.100              : ok=5    changed=3    unreachable=0    failed=0
</code>
</pre>

##### Test nginx to see if its running
<pre>
<code>
vagrant@ansible:~$ curl http://192.168.5.100
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
  body {
      width: 35em;
      margin: 0 auto;
      font-family: Tahoma, Verdana, Arial, sans-serif;
  }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

</code>
</pre>

<pre>
<code>
vagrant@ansible:~$ exit
logout
Connection to 127.0.0.1 closed.
</code>
</pre>

## NOTE

Its also possible to run ansible as part of vagrant Provisioning. i.e

<pre>
  <code>
  Vagrant.configure("2") do |config|
    config.vm.define "webserver" do |webserver|
      webserver.vm.box = "ubuntu/trusty64"
      config.vm.network "private_network", ip: "192.168.5.100"
      config.vm.hostname = "webserver"
      ansible.playbook = "nginx.yml"
    end
  end
  </code>
</pre>

## Congratulations, We did it !!
