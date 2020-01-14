# -*- mode: ruby -*-
# vi: set ft=ruby :
manage_host = true
manage_guest = true
private_net_prefix='192.168.50'
domain="example.com"
hosts={
  master:{
    hostname: "na001.#{domain}",
    aliases: "master.example.com",
    #memory: 1024,
    private_ip: "#{private_net_prefix}.2",
    provision:
      <<-SCRIPT
        sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
        sudo setenforce 0
        sudo yum install wget -y
        sudo yum install git -y
        git clone git://github.com/netauth/netauth
        cd netauth
        git checkout tags/v0.3.2
        go build -o netauthd cmd/netauthd/main.go
        go build -o netauth cmd/netauth/main.go
        sudo cp netauth netauthd /usr/local/sbin/
        #wget https://github.com/netauth/netauth/releases/download/v0.3.2/NetAuth_0.3.2_Linux_x86_64.tar.gz
        #sudo tar zxf NetAuth_0.3.2_Linux_x86_64.tar.gz -C /usr/local/bin/
        sudo useradd netauth
        sudo mkdir /etc/netauth
        sudo mkdir /var/lib/netauth
        sudo chown netauth: /var/lib/netauth
        sudo cp /vagrant/config.toml /etc/netauth/
        sudo cp /vagrant/netauthd.service /etc/systemd/system/
        sudo systemctl daemon-reload

      SCRIPT
  },
  edge:{
    hostname: "edge001.#{domain}",
    #aliases: "kerberos2.example.com",
    #memory: 1024,
    private_ip: "#{private_net_prefix}.11",
    provision:
      <<-SCRIPT
        sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
        sudo setenforce 0
      SCRIPT
  },
}

if Gem::Version.new(::Vagrant::VERSION) < Gem::Version.new('1.5')
  Vagrant.require_plugin('vagrant-hostmanager')
end

Vagrant.configure('2') do |config|

  if ENV.key? 'VAGRANT_BOX'
    config.vm.box = ENV['VAGRANT_BOX']
  else
    config.vm.box = 'centos/7'
  end

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = manage_host
  config.hostmanager.manage_guest = manage_guest

  hosts.each do |name,prop|
    config.vm.define name do |server|
      server.vm.hostname = prop[:hostname] if prop[:hostname]
      if prop[:private_ip]
        server.vm.network :private_network, :ip => prop[:private_ip]
      end
      server.hostmanager.aliases = prop[:aliases] if prop[:aliases]
      if prop[:memory]
        server.vm.provider "virtualbox" do |vb|
          vb.memory = prop[:memory]
        end
      end
      server.vm.provision "shell", inline: prop[:provision] if prop[:provision]
    end
  end
end


