# -*- mode: ruby -*-
# vi: set ft=ruby :

thedr_userid = "2001"
thedr_groupid = "2001"

Vagrant.configure("2") do |config|
  #config.vm.box = "galaxy3/kamino"
  #config.vm.box_version = "2021.01.05-0000"
  config.vm.box = "kalilinux/rolling"
  config.vm.hostname = "kamino-build"
  config.vm.box_version = "2020.4.0"
  # config.disksize.size = '75GB'

  config.vm.network "private_network", ip: "10.55.55.4",
  	virtualbox__intnet: "g3main"
  config.vm.network "private_network", ip: "10.55.56.4",
  	virtualbox__intnet: "metasploitable3"
  config.vm.network "private_network", ip: "10.55.56.10",
  	virtualbox__intnet: "metasploitable3"
  #config.vm.network "private_network", ip: "10.55.55.4"

  config.vbguest.auto_update = false

  config.ssh.insert_key = false
  config.ssh.connect_timeout = 20

#  config.vm.synced_folder	"../../bind",	"/bind", owner: thedr_userid, group: thedr_groupid, create: true
#  config.vm.synced_folder	"../../",	"/vagrant", owner: thedr_userid, group: thedr_groupid
#  config.vm.synced_folder "../../repos", "/repos", owner: thedr_userid, group: thedr_groupid, create: true
#  config.vm.synced_folder "../../Downloads", "/Downloads", owner: thedr_userid, group: thedr_groupid, create: true

  config.vm.network "forwarded_port", guest: 22, host: 2200, id: "ssh", disabled: true
  config.vm.network "forwarded_port", guest: 5901, host: 24901, host_ip: "127.0.0.1", auto_correct: true
  config.vm.network "forwarded_port", guest: 5902, host: 24902, host_ip: "127.0.0.1", auto_correct: true
  config.vm.network "forwarded_port", guest: 3389, host: 24389, host_ip: "0.0.0.0", auto_correct: true
  config.vm.network "forwarded_port", guest: 22, host: 24022, host_ip: "0.0.0.0", auto_correct: true
  config.vm.network "forwarded_port", guest: 80, host: 24080, host_ip: "127.0.0.1", auto_correct: true
  config.vm.network "forwarded_port", guest: 8080, host: 24880, host_ip: "127.0.0.1", auto_correct: true

  config.vm.provision "file", source: "playbook.yml", destination: "playbook.yml"
  config.vm.provision "file", source: "../../functions", destination: "functions/bin"
  config.vm.provision "file", source: "hosts", destination: "hosts"
  config.vm.provision "file", source: "requirements.yml", destination: "requirements.yml"
  config.vm.provision "file", source: "ansible", destination: "roles/ansible"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "Kamino (Kali) Build"
    vb.gui = false
    vb.cpus = "4"
    vb.memory = "8192"

    disk01 = './disk01.vdi'
    disk02 = './disk02.vdi'

    unless File.exist?(disk01)
      vb.customize ['createhd', '--filename', disk01, '--variant', 'Standard', '--size', 500 * 1024]
    end

    unless File.exist?(disk02)
      vb.customize ['createhd', '--filename', disk02, '--variant', 'Standard', '--size', 500 * 1024]
    end

    vb.customize ['storageattach', :id, '--storagectl', 'IDE Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', disk01]
    vb.customize ['storageattach', :id, '--storagectl', 'IDE Controller', '--port', 1, '--device', 1, '--type', 'hdd', '--medium', disk02]

  end
  config.vm.provision "shell", inline: <<-SHELL
     sudo blkid | egrep '/dev/sdb' | egrep ext4 || mkfs.ext4 /dev/sdb
     sudo blkid | egrep '/dev/sdc' | egrep ext4 || mkfs.ext4 /dev/sdc

     egrep '/dev/sdb' /etc/fstab || echo '/dev/sdb /opt	ext4    errors=remount-ro 0       1' >> /etc/fstab
     egrep '/dev/sdc' /etc/fstab || echo '/dev/sdc /usr/share	ext4    errors=remount-ro 0       1' >> /etc/fstab

     #
     mount /opt

     mkdir -p /tmp/usr/share
     mount /dev/sdc /tmp/usr/share
     (cd /usr/share && find . -print | cpio -pdm /tmp/usr/share)
     umount /tmp/usr/share
     mv /usr/share /usr/share.old
     mkdir -p /usr/share
     mount /usr/share
     rm -rf /usr/share.old

     ls -l /home/vagrant
SHELL
#  config.vm.provision "ansible_local" do |ansible|
#    ansible.playbook = "/home/vagrant/playbook.yml"
#    ansible.galaxy_role_file = "/home/vagrant/requirements.yml"
#    inventory_path = "/home/vagrant/hosts"
#  end
  config.vm.provision "shell", inline: <<-SHELL
    bash <(curl -s https://raw.githubusercontent.com/galaxy3-net/dvwa/dvwa/Deploy)
	apt-get autoremove -y
	apt-get clean
	dd if=/dev/zero of=/dummy bs=1M || rm /dummy
SHELL
end
