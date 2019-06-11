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
