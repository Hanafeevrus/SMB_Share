# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  
  config.vm.define "smbsrv", primary: true do |s|
    s.vm.box = "centos/7"
    s.vm.provider :virtualbox do |sb|
        sb.customize ["modifyvm", :id, "--memory", "1024"]
        sb.customize ["modifyvm", :id, "--cpus", "1"]
    end
    s.vm.hostname = 'smbsrv'
    s.vm.network "private_network", ip: "192.168.111.60"
    s.vm.provision "shell", inline: <<-SHELL
    	sudo yum update
    	sudo yum install samba samba-common cups-libs samba-client -y
	sudo mkdir -p /samba/upload
	sudo chmod -R 0775 /samba/upload
        sudo chown -R nobody:nobody /samba/upload
	sudo systemctl start firewalld
	sudo systemctl enable firewalld
	sudo firewall-cmd --permanent --add-port=445/tcp
	sudo systemctl restart firewalld
	sudo setsebool -P samba_export_all_ro=1 samba_export_all_rw=1
	sudo cp /vagrant/smb.conf /etc/samba/smb.conf
	sudo systemctl start smb
	sudo systemctl enable smb
    	SHELL
  end

  config.vm.define "smbclient" do |c|
    c.vm.box = "centos/7"
    c.vm.provider :virtualbox do |cb|
        cb.customize ["modifyvm", :id, "--memory", "512"]
        cb.customize ["modifyvm", :id, "--cpus", "1"]
    end
    c.vm.hostname = 'smbclient'
    c.vm.network "private_network", ip: "192.168.111.61"
    c.vm.provision "shell", inline: <<-SHELL
    	sudo systemctl start firewalld
	sudo systemctl enable firewalld
	sudo yum install samba-client -y
	sudo mkdir /mnt/share
	echo //192.168.111.60/samba_share/ /mnt/share cifs guest, 0 0 >> /etc/fstab
	sudo mount -a
        sudo cp /vagrant/FileTestShare /mnt/share/upload
	sudo reboot
    SHELL
    end
  end
