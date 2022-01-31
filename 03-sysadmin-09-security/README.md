# 03-sysadmin-09-security  
  
## 1)  
Зарегистрировался, сохранил пароли. Скриншот приложил.  
  
## 2)  
Установил, настроил. Скриншоты приложил.  
  
## 3)  
>vagrant@sandbox:\~$ sudo apt update  
>vagrant@sandbox:\~$ sudo apt install apache2  
>vagrant@sandbox:\~$ sudo a2enmod ssl  
>vagrant@sandbox:\~$ sudo systemctl restart apache2  
  
>vagrant@sandbox:\~$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \  
> -keyout /etc/ssl/private/apache-selfsigned.key \  
> -out /etc/ssl/certs/apache-selfsigned.crt \  
> -subj "/C=RU/ST=Moscow/L=Moscow/O=Company Name/OU=Org/CN=testapache2.com"  
>Generating a RSA private key  
>...............................................................................+++++  
>........+++++  
>writing new private key to '/etc/ssl/private/apache-selfsigned.key'  
>\-----  
  
>vagrant@sandbox:\~$ sudo nano /etc/apache2/sites-available/testapache2.com.conf  
>\<VirtualHost *:443>  
>   ServerName testapache2.com  
>   DocumentRoot /var/www/testapache2.com  
>  
>   SSLEngine on  
>   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt  
>   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key  
>\</VirtualHost>  
  
>vagrant@sandbox:\~$ sudo mkdir /var/www/testapache2.com  
>vagrant@sandbox:\~$ sudo nano /var/www/testapache2.com/index.html  
>\<h1\>it worked!\</h1\>  
  
>vagrant@sandbox:\~$ sudo a2ensite testapache2.com.conf  
>Enabling site testapache2.com.  
>To activate the new configuration, you need to run:  
>  systemctl reload apache2  
  
>vagrant@sandbox:\~$ sudo apache2ctl configtest  
>AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.2.1. Set the 'ServerName' directive globally to suppress this message  
>Syntax OK  
  
>vagrant@sandbox:\~$ sudo systemctl reload apache2  
  
>C:\Windows\System32\drivers\etc\hosts  
>127.0.0.1 testapache2.com  
  
>Vagrantfile:  
>default.vm.network "forwarded_port", guest: 443, host: 443  
  
Увидел в браузере на хосте `it worked!`, скриншот приложил.  
  
## 4)  
Протестил нетологию, приложил выдержку.  
  
>vagrant@sandbox:\~/testssl.sh$ ./testssl.sh -U --sneaky https://netology.ru  
>  
>Testing all IPv4 addresses (port 443): 104.22.41.171 104.22.40.171 172.67.21.207  
>\------------------------------------------------------  
  
...  
  
>Secure Renegotiation (RFC 5746)           OpenSSL handshake didn't succeed  
>BREACH (CVE-2013-3587)                    potentially NOT ok, "gzip" HTTP compression detected.  
>SWEET32 (CVE-2016-2183, CVE-2016-6329)    VULNERABLE, uses 64 bit block ciphers  
>BEAST (CVE-2011-3389)                     TLS1: ECDHE-RSA-AES128-SHA AES128-SHA  
>                                                 ECDHE-RSA-AES256-SHA AES256-SHA  
>                                                 DES-CBC3-SHA  
>                                           VULNERABLE -- but also supports higher protocols  TLSv1.1 TLSv1.2 (likely mitigated)  
>LUCKY13 (CVE-2013-0169), experimental     potentially VULNERABLE, uses cipher block chaining (CBC) ciphers with TLS. Check patches  
  
...  
  
>\------------------------------------------------------  
>Done testing now all IP addresses (on port 443): 104.22.41.171 104.22.40.171 172.67.21.207  
  
## 5)  
>vagrant@sandbox:\~$ apt install openssh-server  
>vagrant@sandbox:\~$ systemctl start sshd.service  
>vagrant@sandbox:\~$ ssh-keygen  
>Generating public/private rsa key pair.  
>Enter file in which to save the key (/home/vagrant/.ssh/id_rsa):  
>/home/vagrant/.ssh/id_rsa already exists.  
>Overwrite (y/n)? y  
>Enter passphrase (empty for no passphrase):  
>Enter same passphrase again:  
>Your identification has been saved in /home/vagrant/.ssh/id_rsa  
>Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub  
>The key fingerprint is:  
>SHA256:w5+NDs0XxalR3kt+JzmlC0OysQNYlQkUV8wM7A+cV44 vagrant@sandbox  
>The key's randomart image is:  
>+---[RSA 3072]----+  
>|       .==+O. .  |  
>|       o .+ ++.o |  
>|      . .oo.o+=.o|  
>|       . .=*E+++.|  
>|        S ++= =oo|  
>|         = =.+ +o|  
>|        . * o .  |  
>|         o .     |  
>|          .      |  
>+----[SHA256]-----+  
  
>vagrant@sandbox:\~$ cat \~/.ssh/id_rsa.pub  
>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDXrxAAhjHdohhwJ3XqmZWwznBYdTcNdq5eaSGVbMhNX0yhUe8R/ssEZT4kuRCuk2yBTti0bKmpC8fNeYo1aoMREyInKeaYerqGlDoNQHWr5/Vtkqqs9SYn54Cci3+o3yneziJ6HGwTYlKZNhxx/OmBzhmCFDkelFArlsrUc8/DnxSFP3U1+pXoKfHt6Fwrrl9LCPF2SxPTJJSulBRNORqIB0Fg/9XqL2B9qTnHsdh+rqvTNKhEOzdd9XT5+jGRwc7RDsFJ2WvhaFgEVwa6IFf65kmvM3Fi/OIViNwtY+n3XsOQzR9RxR1KqGjr8SOreKPSBEW8lCMCKNd8opUm7K0tj6KTe+Kgx2dQtCNg2I6WQA9YTsBvKJCAfw5PhRKSx7+O3nQT5EtBL0K8+WP0R6mcvirLp2caakhhKj5rNMWxsuO7V8wYDF84bAfbufxd1hq5TrhOUEzYqXJC56MlVOF0gBnULoZy6HDKtpqpwvWd1G3g9j6+aFsCxO8CAARvKGM= vagrant@sandbox  
>  
На этом этапе создал вторую vm в vagrant, добавил в конфигурации обоих машин дополнительные интерфейсы private network c адресами 10.0.0.200 и 10.0.0.201:  

>default.vm.network "private_network", ip: "10.0.0.200"  
>sandbox2.vm.network "private_network", ip: "10.0.0.201"  
  
>vagrant@sandbox:\~$ ping 10.0.0.201  
>PING 10.0.0.201 (10.0.0.201) 56(84) bytes of data.  
>64 bytes from 10.0.0.201: icmp_seq=1 ttl=64 time=0.926 ms  
  
>vagrant@sandbox:\~$ ssh-copy-id vagrant@10.0.0.201  
>/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"  
>The authenticity of host '10.0.0.201 (10.0.0.201)' can't be established.  
>ECDSA key fingerprint is SHA256:RztZ38lZsUpiN3mQrXHa6qtsUgsttBXWJibL2nAiwdQ.  
>Are you sure you want to continue connecting (yes/no/[fingerprint])? y  
>Please type 'yes', 'no' or the fingerprint: yes  
>/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed  
>/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys  
>vagrant@10.0.0.201's password:  
>  
>Number of key(s) added: 1  
>  
>Now try logging into the machine, with:   "ssh 'vagrant@10.0.0.201'"  
>and check to make sure that only the key(s) you wanted were added.  
  
Проверил authorized_keys, ключ на месте:  
  
>vagrant@sandbox2:\~$ cat .ssh/authorized_keys  
>ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDXrxAAhjHdohhwJ3XqmZWwznBYdTcNdq5eaSGVbMhNX0yhUe8R/ssEZT4kuRCuk2yBTti0bKmpC8fNeYo1aoMREyInKeaYerqGlDoNQHWr5/Vtkqqs9SYn54Cci3+o3yneziJ6HGwTYlKZNhxx/OmBzhmCFDkelFArlsrUc8/DnxSFP3U1+pXoKfHt6Fwrrl9LCPF2SxPTJJSulBRNORqIB0Fg/9XqL2B9qTnHsdh+rqvTNKhEOzdd9XT5+jGRwc7RDsFJ2WvhaFgEVwa6IFf65kmvM3Fi/OIViNwtY+n3XsOQzR9RxR1KqGjr8SOreKPSBEW8lCMCKNd8opUm7K0tj6KTe+Kgx2dQtCNg2I6WQA9YTsBvKJCAfw5PhRKSx7+O3nQT5EtBL0K8+WP0R6mcvirLp2caakhhKj5rNMWxsuO7V8wYDF84bAfbufxd1hq5TrhOUEzYqXJC56MlVOF0gBnULoZy6HDKtpqpwvWd1G3g9j6+aFsCxO8CAARvKGM= vagrant@sandbox  
  
Подключился по ssh:  
  
>vagrant@sandbox:\~$ ssh vagrant@10.0.0.201  
>Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-96-generic x86_64)  
>  
> * Documentation:  https://help.ubuntu.com  
> * Management:     https://landscape.canonical.com  
> * Support:        https://ubuntu.com/advantage  
>  
>  System information as of Mon 31 Jan 2022 09:49:32 AM UTC  
>  
>  System load:  0.08               Processes:             119  
>  Usage of /:   12.7% of 30.88GB   Users logged in:       1  
>  Memory usage: 19%                IPv4 address for eth0: 10.0.2.15  
>  Swap usage:   0%                 IPv4 address for eth1: 10.0.0.201  
>  
>  
>This system is built by the Bento project by Chef Software  
>More information can be found at https://github.com/chef/bento  
>Last login: Mon Jan 31 09:35:14 2022 from 10.0.2.2  
>vagrant@sandbox2:\~$  
  
## 6)  
>vagrant@sandbox:\~$ cd .ssh/  
>vagrant@sandbox:\~/.ssh$ mv id_rsa pav_rsa  
>vagrant@sandbox:\~/.ssh$ mv id_rsa.pub pav_rsa.pub  
>vagrant@sandbox:\~/.ssh$ touch \~/.ssh/config && chmod 600 \~/.ssh/config  
>vagrant@sandbox:\~/.ssh$ nano config  
>>Host sandbox2  
>>User vagrant  
>>HostName 10.0.0.201  
>>Port 22  
>>IdentityFile \~/.ssh/pav_rsa  
  
>vagrant@sandbox:\~/.ssh$ ssh sandbox2  
>Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-96-generic x86_64)  
>  
> * Documentation:  https://help.ubuntu.com  
> * Management:     https://landscape.canonical.com  
> * Support:        https://ubuntu.com/advantage  
>  
>  System information as of Mon 31 Jan 2022 10:26:20 AM UTC  
>  
>  System load:  0.0                Processes:             116  
>  Usage of /:   12.7% of 30.88GB   Users logged in:       1  
>  Memory usage: 19%                IPv4 address for eth0: 10.0.2.15  
>  Swap usage:   0%                 IPv4 address for eth1: 10.0.0.201  
>  
>  
>This system is built by the Bento project by Chef Software  
>More information can be found at https://github.com/chef/bento  
>Last login: Mon Jan 31 09:49:32 2022 from 10.0.0.200  
>vagrant@sandbox2:\~$  
  
## 7)  
>vagrant@sandbox:\~$ sudo tcpdump -w 0001.pcap -c 100 -i eth0  
>tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes  
>100 packets captured  
>154 packets received by filter  
>0 packets dropped by kernel  
  
Создал в Windows папку vagrantshare, настроил ее в virtualbox, положил туда файл.  
  
>vagrant@sandbox:\~$ sudo mount -t vboxsf vagrantshare \~/vagrantshare  
>vagrant@sandbox:\~$ cp 0001.pcap \~/vagrantshare  
>vagrant@sandbox:\~$ sudo umount vagrantshare  
  
Установил Wireshark на хост Windows, открыл 0001.pcap. Приложил скриншоты.
