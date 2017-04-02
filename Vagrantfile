# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "centos/7"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true

    # Customize the amount of memory on the VM:
    vb.memory = "12288"

    # Add Disk
    vb.customize ['createmedium', 'disk', '--filename', "disk/sdb.vdi", '--format', 'VDI', '--size', 200 * 1024]
    vb.customize ['storageattach', :id,'--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', "disk/sdb.vdi"]

    # Add NIC
    vb.customize ["modifyvm", :id, "--nic2", "hostonly"]
    vb.customize ["modifyvm", :id, "--nictype2", "82540EM"]
    vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "file", source: "ifcfg-br-ex", destination: "~/ifcfg-br-ex"
  config.vm.provision "file", source: "ifcfg-eth0", destination: "~/ifcfg-eth0"
  config.vm.provision "shell", inline: <<-SHELL
    systemctl restart network
    yum -y update
    yum -y remove mariadb-libs
    sed -i "/^\[base\]$/a exclude=mariadb*" /etc/yum.repos.d/CentOS-Base.repo
    sed -i "/^\[updates\]$/a exclude=mariadb*" /etc/yum.repos.d/CentOS-Base.repo
    yum install -y https://rdoproject.org/repos/rdo-release.rpm
    yum -y install ntp emacs
    systemctl start ntpd
    systemctl enable ntpd
    sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    setenforce 0
    systemctl stop NetworkManager
    systemctl disable NetworkManager
    pvcreate /dev/sdb
    vgextend VolGroup00 /dev/sdb
    lvextend -l +100%FREE /dev/VolGroup00/LogVol00
    xfs_growfs /dev/VolGroup00/LogVol00
    yum install -y openstack-packstack
    packstack --gen-answer-file=~/answer.txt
    sed -i 's/^CONFIG_MANILA_INSTALL=n$/CONFIG_MANILA_INSTALL=y/' ~/answer.txt
    sed -i 's/^CONFIG_TROVE_INSTALL=n$/CONFIG_TROVE_INSTALL=y/' ~/answer.txt
    sed -i 's/^CONFIG_HEAT_INSTALL=n$/CONFIG_HEAT_INSTALL=y/' ~/answer.txt
    sed -i 's/10.0.2.15/192.168.33.10/' ~/answer.txt
    packstack --answer-file=~/answer.txt
    cp /home/vagrant/ifcfg-br-ex /etc/sysconfig/network-scripts/
    cp /home/vagrant/ifcfg-eth0 /etc/sysconfig/network-scripts/
    systemctl restart network
  SHELL
end
