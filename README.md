# SMB_Share   
[Samba](https://www.samba.org/) — software package that allows you to access network drives and printers on various operating systems using the SMB / CIFS protocol. It has client and server parts. Is free software released under the GPL.    
Starting with the fourth version, which has been under development for almost 10 years, Samba can act as a domain controller and Active Directory service compatible with the implementation of Windows 2000, and is able to serve all versions of Microsoft-supported Windows-clients, including Windows 10.
Samba runs on most Unix-like systems, such as Linux, POSIX-compatible Solaris, and Mac OS X Server, on various BSD variants; OS/2 ported Samba-client, which is a plug-in to the virtual file system NetDrive. Samba is included with almost all Linux distributions.
#### Task   
* Share the directory on the server using SAMBA, with the upload directory subfolder with write permissions		
* Set up auto-mounting on the client		
#### Solution		
Vagrant up lifts 2 machines:    
smbsrv    - 192.168.111.60 -SAMBA server		
smbclient - 192.168.111.61 -samba client		

#### server configuration:   
* Installing prerequisites		
`sudo yum install samba samba-common`   
* Directory preparation		
`sudo mkdir -p /samba/upload`   
give out rights.   
`sudo chmod -R 0775 /samba/upload`    
change owner on behalf of which the recording will take place.   
`sudo chown -R nobody:nobody /samba/upload`   
* Configuration [Firewalld](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Configuring_firewalld.html), allow port 445		
`sudo firewall-cmd --permanent --add-port=445/tcp`
* Configuration [Selinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/managing_confined_services/sect-managing_confined_services-samba-booleans)   
`sudo setsebool -P samba_export_all_ro=1 samba_export_all_rw=1`
* Configuration smb service		
The main configuration file `/ etc / samba / smb.conf` defines global server parameters and parameters of shared directories.		
A detailed description of the syntax and parameters in [man smb.conf](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html).    
In my case, a connection is used with guest authorization `guest ok = yes` and the ability to write` writable = yes`		
```
[global]
	workgroup = WORKGROUP
	security = user
	netbios name = smbsrv
	passdb backend = tdbsam

[samba_share]
	comment = share with upload
	path = /samba
	browsable =yes
	writable = yes
	guest ok = yes
	read only = no
	force user = nobody		
	create mode = 0777
        directory mode = 0777
        force user = nobody		
```   
  #### client configuration.   
  Mounting a directory automatically using fstab, Detailed description [man fstab](http://man7.org/linux/man-pages/man5/fstab.5.html).      
  * Create mount directory		
    `mkdir /mnt/share`    
  * add to `/etc/fstab`   
  `//192.168.111.60/samba_share/ /mnt/share cifs guest,dir_mode=0777,file_mode=0777 0 0 `   
  
  check:    
  `sudo mount -a`   
  `df -h` #output should be something like this:    
  ```
  [vagrant@smbclient upload]$ df -h
Filesystem                     Size  Used Avail Use% Mounted on
/dev/sda1                       40G  3.8G   37G  10% /
devtmpfs                       237M     0  237M   0% /dev
tmpfs                          244M     0  244M   0% /dev/shm
tmpfs                          244M  4.5M  240M   2% /run
tmpfs                          244M     0  244M   0% /sys/fs/cgroup
//192.168.111.60/samba_share/   40G  3.8G   37G  10% /mnt/share   
```   
Vagranfile client setup is written		
`cp /vagrant/FileTestShare /mnt/share/upload` -checking file recording in a network directory.		
