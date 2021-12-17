# 03-sysadmin-04-os  

## 1)  
Примечания:  
>/usr/lib/systemd/system/ – юниты из установленных пакетов.  
>/run/systemd/system/ — юниты, созданные в рантайме.  
>/etc/systemd/system/ — юниты, созданные системным администратором.  
Файлы юнитов загружаются из целого ряда мест, команда `systemctl show --property=UnitPath`  
выведет полный список.  

>vagrant@sandbox:\~$ sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter  
>vagrant@sandbox:\~$ mkdir Downloads  
>vagrant@sandbox:\~$ cd Downloads/  
>vagrant@sandbox:\~/Downloads$ wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz  
>vagrant@sandbox:\~/Downloads$ tar xzf node_exporter-1.3.1.linux-amd64.tar.gz  
>  
>vagrant@sandbox:\~$ sudo mv -v node_exporter /usr/local/bin  
>renamed 'node_exporter' -> '/usr/local/bin/node_exporter'  
>
>vagrant@sandbox:\~$ sudo chown root:root /usr/local/bin/node_exporter  
>
>vagrant@sandbox:\~$ node_exporter --version  
>node_exporter, version 1.3.1 (branch: HEAD, revision: a2321e7b940ddcff26873612bccdf7cd4c42b6b6)  

>vagrant@sandbox:\~$ sudo nano /etc/systemd/system/node-exporter.service  
>
> >[Unit]  
> >Description=node_exporter for machine metrics  
> >
> >[Service]  
> >Restart=always  
> >User=node_exporter  
> >ExecStart=/usr/local/bin/node_exporter  
> >ExecReload=/bin/kill -HUP $MAINPID  
> >TimeoutStopSec=20s  
> >SendSIGKILL=no  
> >
> >[Install]  
> >WantedBy=multi-user.target  

>vagrant@sandbox:\~$ sudo systemctl daemon-reload  
>vagrant@sandbox:\~$ sudo systemctl start node-exporter.service  
>vagrant@sandbox:\~$ sudo systemctl enable node-exporter  
>Created symlink /etc/systemd/system/multi-user.target.wants/node-exporter.service → /etc/systemd/system/node-exporter.service.  

>vagrant@sandbox:~$ sudo systemctl status node-exporter  
>● node-exporter.service - node_exporter for machine metrics  
>     Loaded: loaded (/etc/systemd/system/node-exporter.service; enabled; vendor preset: e>  
>     Active: active (running) since Thu 2021-12-16 12:09:01 UTC; 2min 1s ago  
>   Main PID: 1261 (node_exporter)  
>      Tasks: 4 (limit: 1071)  
>     Memory: 2.6M  
>     CGroup: /system.slice/node-exporter.service  
>             └─1261 /usr/local/bin/node_exporter  

Проверяем порт 9100:  
>vagrant@sandbox:~$ sudo ss -pnltu | grep 9100  
>tcp     LISTEN   0        4096                    *:9100                *:*      users:(("node_exporter",pid=1261,fd=3))  

Порт проброшен на хост, на http://localhost:9100/metrics есть ответ:  
![node](//~/devops-netology/03-sysadmin-04-os/node_exporter_9100.png "node_exporter_9100.png")

Смотрим на виртуалке:  
>vagrant@sandbox:~$ curl http://localhost:9100/metrics  
>\# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.  
>\# TYPE go_gc_duration_seconds summary  
>go_gc_duration_seconds{quantile="0"} 3.068e-05  
>go_gc_duration_seconds{quantile="0.25"} 3.2839e-05  
>go_gc_duration_seconds{quantile="0.5"} 3.9345e-05  
>...

Проверяем работу процесса:  
>vagrant@sandbox:\~$ ps -e | grep node_exporter  
>   1261 ?        00:00:00 node_exporter  
>  
>vagrant@sandbox:\~$ sudo systemctl stop node-exporter  
>vagrant@sandbox:\~$ ps -e | grep node_exporter  
>  
>vagrant@sandbox:\~$ sudo systemctl start node-exporter  
>vagrant@sandbox:\~$ ps -e | grep node_exporter  
>   1415 ?        00:00:00 node_exporter  
>  
>vagrant@sandbox:\~$ sudo reboot  
>vagrant@sandbox:\~$ ps -e | grep node_exporter  
>    595 ?        00:00:00 node_exporter  
>  
>vagrant@sandbox:\~$ sudo systemctl status node-exporter  
>● node-exporter.service - node_exporter for machine metrics  
>     Loaded: loaded (/etc/systemd/system/node-exporter.service; enabled; vendor preset: e>  
>     Active: active (running) since Thu 2021-12-16 13:05:03 UTC; 33s ago  
>   Main PID: 595 (node_exporter)  
>      Tasks: 5 (limit: 1071)  
>     Memory: 14.0M  
>     CGroup: /system.slice/node-exporter.service  
>             └─595 /usr/local/bin/node_exporter  

Добавляем возможность запуска node_exporter c опциями через внешний файл:  
>vagrant@sandbox:/etc/default$ sudo nano /etc/systemd/system/node-exporter.service  
>  
> >EnvironmentFile=/etc/default/node_exporter  
> >ExecStart=/usr/local/bin/node_exporter $EXTRA_OPTS  
>  
>vagrant@sandbox:\~$ cd /etc/default  
>vagrant@sandbox:\~$ sudo nano /etc/default/node_exporter  
>
>\# This is environmentFile for node_exporter  
>EXTRA_OPTS=""  

## 2)  
Для просмотра опций вводим `node_exporter --help`. Можно также посмотреть в `http://localhost:9100/metrics`  

CPU:  
node_cpu_seconds_total{cpu="0",mode="idle"} 2241.23  
node_cpu_seconds_total{cpu="0",mode="system"} 8.41  
node_cpu_seconds_total{cpu="0",mode="user"} 2.32  
node_cpu_seconds_total{cpu="1",mode="idle"} 2234.84  
node_cpu_seconds_total{cpu="1",mode="system"} 6.18  
node_cpu_seconds_total{cpu="1",mode="user"} 1.34  

Mem:  
node_memory_MemAvailable_bytes 7.54868224e+08  
node_memory_MemFree_bytes 6.40970752e+08  
node_memory_MemTotal_bytes 1.028694016e+09 

HDD:  
node_filesystem_avail_bytes{device="/dev/mapper/vgvagrant-root",fstype="ext4",mountpoint="/"} 6.0652433408e+10  
node_filesystem_avail_bytes{device="/dev/sda1",fstype="vfat",mountpoint="/boot/efi"} 5.35801856e+08  

LAN:  
node_network_receive_bytes_total{device="eth0"} 199554  
node_network_receive_bytes_total{device="lo"} 776  
node_network_transmit_bytes_total{device="eth0"} 192640  
node_network_transmit_bytes_total{device="lo"} 776  

## 3)  
>vagrant@sandbox:~$ sudo apt install -y netdata  

Проверяем на виртуальной машине:  
>vagrant@sandbox:~$ sudo lsof -i TCP:19999  
>COMMAND PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME  
>netdata 624 netdata    4u  IPv4  23396      0t0  TCP *:19999 (LISTEN)  
>netdata 624 netdata   31u  IPv4  29801      0t0  TCP sandbox:19999->_gateway:62875 (ESTABLISHED)  

В браузере на хосте видим метрики: `Used Swap, Disk Read, Disk Write, CPU, Net Inbound, Net Outbound, Used RAM`.  
![netdata](//~/devops-netology/03-sysadmin-04-os/netdata.png "netdata.png")  

## 4)  
Сообщения из лога событий загрузки ядра можно прочитать командой:  
>vagrant@sandbox:~$ cat /var/log/dmesg  

Да, ОС понимает, что система виртуализована, даже знает, что это `oracle virtualbox`.  
>vagrant@sandbox:~$ cat /var/log/dmesg | grep virt*  
>[    0.004456] kernel: CPU MTRRs all blank - virtualized system.  
>[    0.047843] kernel: Booting paravirtualized kernel on KVM  
>[   10.239020] systemd[1]: Detected virtualization oracle.  

То же самое, с помощью `dmesg`:  
>vagrant@sandbox:~$ dmesg -H | grep virt*  
>[  +0.000006] CPU MTRRs all blank - virtualized system.  
>[  +0.000001] Booting paravirtualized kernel on KVM  
>[  +0.000039] systemd[1]: Detected virtualization oracle.  

Дополнение: команда `dmesg` показывает не только сообщения от ядра, но и другие сообщения от системных служб, например, от системы инициализации `systemd`.  

## 5)  
Утилита `sysctl` позволяет настроить параметры ядра системы во время выполнения. Все настраиваемые параметры  
можно вывести командой `sysctl -a`.  

Выведем значение нужного параметра `fs.nr_open`:  
>vagrant@sandbox:~$ /sbin/sysctl -n fs.nr_open  
>1048576  

Это максимальное количество файловых дескрипторов (открытых файлов), которое может открыть процесс.  
То же самое:  
>vagrant@sandbox:~$ cat /proc/sys/fs/nr_open  
>1048576  

Для интереса, максимальное количество открытых файлов в системе:  
>vagrant@sandbox:\~$ sysctl fs.file-max  
>fs.file-max = 9223372036854775807  
>vagrant@sandbox:\~$ cat /proc/sys/fs/file-max  
>9223372036854775807  

`ulimit` позволяет посмотреть и изменить настройки лимитов ресурсов в системе.  
Предела в `1048576` не позволит достичь мягкое ограничение в `1024` файла (можно изменить):  
>vagrant@sandbox:\~$ ulimit -a  
>open files                      (-n) 1024  
>vagrant@sandbox:\~$ ulimit -n  
>1024  

Жесткое ограничение равно `1048576`:  
>vagrant@sandbox:\~$ ulimit -aH  
>open files                      (-n) 1048576  
>vagrant@sandbox:~$ ulimit -Hn  
>1048576  

## 6)  
>vagrant@sandbox:\~$ sudo unshare --fork --pid --mount-proc sleep 1h&  
>[1] 1429  
>vagrant@sandbox:\~$ ps -e | grep sleep  
>   1432 pts/0    00:00:00 sleep  
>vagrant@sandbox:\~$ sudo nsenter --target 1432 --pid --mount  
>root@sandbox:/#  ps -e | grep sleep  
>      1 pts/0    00:00:00 sleep  

## 7)  
`:(){ :|:& };:` - это forkbomb.  
С разрывами строк:  
>:()  
>{  
>    :|:&  
>};  
>:  
Определяется функция `:()` с именем `:`, которая вызывает опять себя и передает данные в себя `: | :` в  
фоновом режиме `&`. После `;` определение функции выполнено, и функция `:` запускается. Таким образом, 
каждый экземпляр функции `:` стартует две новых функции `:` и так далее, пока система не повиснет.  

>vagrant@sandbox:\~$ :(){ :|:& };:  
>-bash: fork: retry: Resource temporarily unavailable  
>...  
>  
>vagrant@sandbox:\~$ dmesg | grep fork  
>[  154.558659]  ret_from_fork+0x35/0x40  
>[ 1224.783013] cgroup: fork rejected by pids controller in /user.slice/user-1000.slice/session-3.scope  

Файла самого не нашел. Видимо, там ограничивается количество процессов, которое можно запустить в  
пространстве пользователя. Если использовать `ulimit -u 100` для ограничения максимального количества  
процессов пользователя 100 штуками, то система стабилизируется быстрее.  

>vagrant@sandbox:\~$ ulimit -u 100  
>vagrant@sandbox:\~$ :(){ :|:& };:  
>...  
>-bash: fork: Resource temporarily unavailable 
>  
>[2]+  Done                    : | :  
>vagrant@sandbox:\~$  
