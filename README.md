# SMB_Share   
[Samba](https://www.samba.org/) — пакет программ, которые позволяют обращаться к сетевым дискам и принтерам на различных операционных системах по протоколу SMB/CIFS. Имеет клиентскую и серверную части. Является свободным программным обеспечением, выпущена под лицензией GPL.    
Начиная с четвёртой версии, разработка которой велась почти 10 лет, Samba может выступать в роли контроллера домена и сервиса Active Directory, совместимого с реализацией Windows 2000, и способна обслуживать все поддерживаемые Microsoft версии Windows-клиентов, в том числе Windows 10.
Samba работает на большинстве Unix-подобных систем, таких как Linux, POSIX-совместимых Solaris и Mac OS X Server, на различных вариантах BSD; в OS/2 портирован Samba-клиент, являющийся плагином к виртуальной файловой системе NetDrive. Samba включена практически во все дистрибутивы Linux.
#### Задача   
* Расшарить директорию на сервере средствами SAMBA, с вложенной директорией upload с правами на запись    
* Настроить автомонтирование на клиенте   
#### Реализация   
Vagrant up поднимает 2 машины:    
smbsrv    - 192.168.111.60 -сервер SAMBA   
smbclient - 192.168.111.61 -клиент    

#### Настройка сервера:   
* Установка необходимых компонентов   
`sudo yum install samba samba-common`   
* Подготовка директории   
`sudo mkdir -p /samba/upload`   
выдать права.   
`sudo chmod -R 0775 /samba/upload`    
сменить владельца от имени которого будет происходить запись.   
`sudo chown -R nobody:nobody /samba/upload`   
* Настройка [Firewalld](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Configuring_firewalld.html), открытие порта 445    
`sudo firewall-cmd --permanent --add-port=445/tcp`
* Настройка [Selinux](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/managing_confined_services/sect-managing_confined_services-samba-booleans)   
`sudo setsebool -P samba_export_all_ro=1 samba_export_all_rw=1`
* Настройка сервиса smb   
Основной конфигурационный файл `/etc/samba/smb.conf` определяет глобальные параметры сервера и параметры расшариваемых директорий.    
Подробное описание синтаксиса и параметров в [man smb.conf](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html).    
В моем случае используется подключение с гостевой авторизацией `guest ok = yes` и возможностью записи `writable = yes`    
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
  #### Настройка клиента.   
  Автоматическое монтирование директории с помощью fstab, Подробное описание [man fstab](http://man7.org/linux/man-pages/man5/fstab.5.html).      
  * Создать директорию монтирование   
  `mkdir /mnt/share`    
  * Добавить в `/etc/fstab`   
  `//192.168.111.60/samba_share/ /mnt/share cifs guest,dir_mode=0777,file_mode=0777 0 0 `   
  
  Проверить можно:    
  `sudo mount -a`   
  `df -h` #вывод должен быть примерно таким:    
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
В настройка клиента Vagranfile прописано    
`cp /vagrant/FileTestShare /mnt/share/upload` - проверка записи файла в сетевую директорию.   
