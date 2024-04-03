Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |v|
    v.memory = 8192
    v.cpus = 4
    v.name = 'tx-microk8s'
  end

  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.network "forwarded_port", guest: 443, host: 443
  config.vm.network "forwarded_port", guest: 5432, host: 5432  
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "forwarded_port", guest: 32000, host: 32000
  config.vm.network "forwarded_port", guest: 32001, host: 32001
  config.vm.network "forwarded_port", guest: 32002, host: 32002
  config.vm.network "forwarded_port", guest: 32003, host: 32003


  config.vm.network "forwarded_port", guest: 16443, host: 16443
  config.vm.network "forwarded_port", guest: 10443, host: 10443

  config.vm.network "private_network", ip: "192.168.56.6"
  config.vm.synced_folder "k8s", "/home/vagrant/k8s", disabled: false, create: true
  config.vm.synced_folder "apps", "/home/vagrant/apps", disabled: false, create: true
  
  config.vm.box = "bento/ubuntu-22.04"

  config.vm.synced_folder "certs/", "/certs"
  config.vm.provision "file", source: "#{File.dirname(__FILE__)}/.bash_aliases", destination: "~/.bash_aliases"
  config.vm.provision :shell, path: "#{File.dirname(__FILE__)}/bin/bootstrap.sh"
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
end
