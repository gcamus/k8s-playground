# -*- mode: ruby -*-
# # vi: set ft=ruby :

Vagrant.configure(2) do |config|
  $noproxy=",otto.lotsys.corp"
  (1..3).each do |i|
	$noproxy += ",172.42.42.#{i},k8s#{i}"
  end

  # Configuration du proxy (si le plugin nécessaire est installé)
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http           = "http://#{ENV['USER']}:#{ENV['USER_PWD']}@192.168.240.80:8080"
	config.proxy.https          = "http://#{ENV['USER']}:#{ENV['USER_PWD']}@192.168.240.80:8080"
    config.proxy.no_proxy       = "#{ENV['no_proxy']}#$noproxy"
  end

  (1..3).each do |i|
    config.vm.define "k8s#{i}" do |s|
      s.ssh.forward_agent = true
      s.vm.box = "ubuntu/xenial64"
      s.vm.hostname = "k8s#{i}"
      s.vm.provision :shell, path: "scripts/bootstrap_ansible.sh"
      if i == 1
        s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-master.yml -c local"
      else
        s.vm.provision :shell, inline: "PYTHONUNBUFFERED=1 ansible-playbook /vagrant/ansible/k8s-worker.yml -c local"
		s.vm.network "forwarded_port", guest: 32763, host: "3276#{i}"
      end
      s.vm.network "private_network", ip: "172.42.42.#{i}", netmask: "255.255.255.0",
        auto_config: true,
        virtualbox__intnet: "k8s-net"
      s.vm.provider "virtualbox" do |v|
        v.name = "k8s#{i}"
        v.memory = 2048
		v.cpus = 1
        v.gui = false
      end
    end
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

end
