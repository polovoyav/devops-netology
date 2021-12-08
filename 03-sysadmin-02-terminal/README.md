# 03-sysadmin-02-terminal

## 1)

>vagrant@vagrant:~$ type cd  
>cd is a shell builtin

Встроенные команды являются частью оболочки. cd должен менять  
каталог для всей оболочки, а если использовать cd как внешнюю  
команду, то он изменит каталог только для текущего процесса  
внутри своего окружения, а шелл останется в том же каталоге,  
в котором он был все это время. Тогда после смены каталога  
придется вызвать терминал из этого нового катлога и мы получим  
новую оболочку.

## 2)

>vagrant@vagrant:~$ cd test/  
>vagrant@vagrant:~/test$ nano test_grep  
>Hello  
>world  
>hi everybody  
>wonderful world  
>  
>vagrant@vagrant:~/test$ grep world test_grep | wc -l  
>2  
>vagrant@vagrant:~/test$ grep world test_grep -c  
>2  

## 3)

#### Ответ - процесс systemd  
>vagrant@vagrant:~$ pstree -p  
>systemd(1)─┬─VBoxService(795)─┬─{VBoxService}(797)  
>           │                  ├─{VBoxService}(798)  
>           │                  ├─{VBoxService}(799)  
>           │                  ├─{VBoxService}(800)  
>           │                  ├─{VBoxService}(801)  
>           │                  ├─{VBoxService}(802)  
>           │                  ├─{VBoxService}(804)  
>           │                  └─{VBoxService}(805)  
>           ├─accounts-daemon(565)─┬─{accounts-daemon}(586)  
>           │                      └─{accounts-daemon}(618)  
>  

## 4)

#### Запускаем второй терминал:  
>vagrant@vagrant:~/test$ who  
>vagrant  pts/0        2021-12-06 16:00 (10.0.2.2)  
>vagrant  pts/1        2021-12-06 17:58 (10.0.2.2)  

#### Вывод stdout в первый терминал /dev/pts/0:  
>vagrant@vagrant:~/test$ ls -l /proc/$$/fd/{0,1,2} 2>/dev/pts/1  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/0 -> /dev/pts/0  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/1 -> /dev/pts/0  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/2 -> /dev/pts/0  

#### Выполняем в /dev/pts/0  
>vagrant@vagrant:~/test$ ls -l /proc/$/fd/{0,1,2} 2>/dev/pts/1  

#### Вывод stderr во второй терминал /dev/pts/1:  
>vagrant@vagrant:~$ ls: cannot access `/proc/$/fd/0`: No such file or directory  
>ls: cannot access `/proc/$/fd/1`: No such file or directory  
>ls: cannot access `/proc/$/fd/2`: No such file or directory  

## 5)

#### Попробуем с cat:  
>vagrant@vagrant:~/test$ ls -l /proc/$$/fd/{0,1,2} > test_std  
>  
>vagrant@vagrant:~/test$ cat test_std  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/0 -> /dev/pts/0  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/1 -> /dev/pts/0  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/2 -> /dev/pts/0  
>  
>vagrant@vagrant:~/test$ cat <test_std >test_std_1  
>  
>vagrant@vagrant:~/test$ cat test_std_1  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/0 -> /dev/pts/0  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/1 -> /dev/pts/0  
>lrwx------ 1 vagrant vagrant 64 Dec  6 16:00 /proc/1593/fd/2 -> /dev/pts/0  

#### Получилось.  

## 6)

#### Сможем. Для tty запустим консоль виртуальной машины в VirtualBox.  
#### В консоли VB определяем tty:  
>vagrant@vagrant:~$ tty  
>/dev/tty2  

#### В ssh терминале пишем:  
>vagrant@vagrant:~/test$ tty  
>/dev/pts/0  
>vagrant@vagrant:~/test$ echo Hello world from /dev/pts/0 > /dev/tty2  

#### В консоли видим:  
>vagrant@vagrant:~$ Hello world from /dev/pts/0  

#### В консоли пишем:  
>vagrant@vagrant:~$ echo Hello world from /dev/tty2 > /dev/pts/0  

#### В терминале видим:  
>vagrant@vagrant:~/test$ Hello world from /dev/tty2  

## 7)

bash `5>&1` - мы создали файловый дескриптор `5` для нашей оболочки и вывели его в stdout:  
>vagrant@vagrant:~$ ll /proc/$$/fd/5  
>lrwx------ 1 vagrant vagrant 64 Dec  7 18:47 /proc/2349/fd/5 -> /dev/pts/0  

Выведем `netology` в файл с дескриптором `5`, т.е. в терминал:  
>vagrant@vagrant:~$ echo netology > /proc/$$/fd/5  
>netology  

## 8)

Посмотрим, какие файловые дескрипторы у нас есть:  
>vagrant@vagrant:~$ ls -l /proc/$$/fd  
>total 0  
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 0 -> /dev/pts/1  
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 1 -> /dev/pts/1  
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 2 -> /dev/pts/1  
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 255 -> /dev/pts/1  

Создадим команду, которая выводит и ошибку и отклик в терминал:  
>vagrant@vagrant:~$ ls -l /proc/$$/fd/{0,1,2,10}  
>ls: cannot access `/proc/2664/fd/10`: No such file or directory  
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 /proc/2664/fd/0 -> /dev/pts/1  
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 /proc/2664/fd/1 -> /dev/pts/1  
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 /proc/2664/fd/2 -> /dev/pts/1  

По умолчанию оба файловых дескриптора 1 `stdout` и 2 `stderr`  
попадают в терминал `/dev/pts/1`. Поменяем файловые дескрипторы  
`stdout` и `stderr` местами и отдадим новый `stdout` через pipe следующей  
команде. Новый `stderr` будет выведен в шелл.  
>vagrant@vagrant:~$ ls -l /proc/$$/fd/{0,1,2,10} 3>&1 1>&2 2>&3 | grep cannot >f_stderr
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 /proc/2664/fd/0 -> /dev/pts/1
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 /proc/2664/fd/1 -> /dev/pts/1
>lrwx------ 1 vagrant vagrant 64 Dec  7 21:02 /proc/2664/fd/2 -> /dev/pts/1

Прочитаем содержимое файла `f_stderr`:  
>vagrant@vagrant:~$ cat f_stderr  
>ls: cannot access `/proc/2664/fd/10`: No such file or directory  

## 9)

Выводится список переменных окружения текущей оболочки и их значения.  
То же самое можно получить построчно командами `env`, `printenv`, `set`  
без параметров.

## 10)

`/proc/[pid]/cmdline` - строка 257, это файл только для чтения, который  
содержит полную командную строку для процесса. Если процесс является  
зомби-процессом, то этот файл не имеет содержимого.

>vagrant@vagrant:~$ cat /proc/2834/cmdline  
>-bash  

`/proc/[pid]/exe` - строка 316, это символическая ссылка, содержащая  
фактический путь к выполняемой команде (программе). Можно ввести `/proc/[pid]/exe`  
чтобы запустить еще одну копию программы.

>vagrant@vagrant:~$ ls -lt /proc/2834/exe  
>lrwxrwxrwx 1 vagrant vagrant 0 Dec  7 22:29 /proc/2834/exe -> /usr/bin/bash  

## 11)

#### Ответ - sse4_2  
>vagrant@vagrant:~/test$ grep sse /proc/cpuinfo  
>  
>flags   : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36  
>clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl  
>xtopology nonstop_tsc cpuid tsc_known_freq pni ssse3 cx16 pcid sse4_1 sse4_2  
>hypervisor lahf_lm pti fsgsbase md_clear flush_l1d arch_capabilities

## 12)

По умолчанию при выполнении команд через ssh на удаленной машине для удаленной  
сессии `tty` не выделяется. Можно использовать ключ `-t` для принудительного создания  
псевдотерминала.

>vagrant@vagrant:~$ ssh localhost 'tty'  
>vagrant@localhost`s password:  
>not a tty  
>  
>vagrant@vagrant:~$ ssh -t localhost 'tty' 
>vagrant@localhost`s password:  
>/dev/pts/2  
>Connection to localhost closed.  

>vagrant@vagrant:~$  

## 13)

#### Воспользовался https://github.com/nelhage/reptyr#readme  
>vagrant@vagrant:~$ sudo apt install reptyr  
>  
>vagrant@vagrant:~$ reptyr 3294  
>Unable to attach to pid 3294: Operation not permitted  
>The kernel denied permission while attaching. If your uid matches  
>the target`s, check the value of /proc/sys/kernel/yama/ptrace_scope.  
>For more information, see /etc/sysctl.d/10-ptrace.conf  
>  
>vagrant@vagrant:~$ sudo nano /etc/sysctl.d/10-ptrace.conf  

Установил `kernel.yama.ptrace_scope = 0`  

>vagrant@vagrant:~$ sudo reboot  
>  
>vagrant@vagrant:~$ top  

Отправил в бэкграунд процесс с помощью `CTRL-Z`  
>[1]+  Stopped                 top  
>  
>vagrant@vagrant:~$ bg  
>[1]+ top &  
>  
>vagrant@vagrant:~$ jobs -l  
>[1]+   894 Stopped (signal)        top  
>  
>vagrant@vagrant:~$ disown top  
>-bash: warning: deleting stopped job 1 with process group 894  
>vagrant@vagrant:~$ jobs -l  
>vagrant@vagrant:~$ ps -a  
>    PID TTY          TIME CMD  
>    894 pts/0    00:00:00 top  
>    897 pts/0    00:00:00 ps  
>vagrant@vagrant:~$ reptyr 894  

Получаем:  
>top - 17:16:53 up 8 min,  2 users,  load average: 0.00, 0.05, 0.05  
>Tasks: 103 total,   2 running, 100 sleeping,   0 stopped,   1 zombie  
>%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st  
>MiB Mem :    981.0 total,    622.4 free,    106.4 used,    252.3 buff/cache  
>MiB Swap:    980.0 total,    980.0 free,      0.0 used.    725.5 avail Mem  
>  
>    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND  
>    907 vagrant   20   0    2592   1916   1812 S   3.0   0.2   0:08.96 reptyr  

Отсоединяем `screen` через `CTRL-a d`  
>detached from 899.pts-0.vagrant  
>
>vagrant@vagrant:~$ ps -a  
>    PID TTY          TIME CMD  
>    907 pts/1    00:00:17 reptyr  
>    908 pts/0    00:00:00 top <defunct>  
>    920 pts/0    00:00:00 ps  
>
>vagrant@vagrant:~$ exit  
>logout  
>Connection to 127.0.0.1 closed.  
>  
>PS C:\Users\andrew\VagrantVM> vagrant ssh  
>
>Reconnect ssh, attach screen  
>vagrant@vagrant:~$ screen -r  

Видим `top`:  
>top - 17:26:12 up 17 min,  2 users,  load average: 0.31, 0.19, 0.11  
>Tasks: 105 total,   1 running, 103 sleeping,   0 stopped,   1 zombie  
>%Cpu(s):  0.0 us,  0.2 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.2 si,  0.0 st  
>MiB Mem :    981.0 total,    619.1 free,    109.1 used,    252.8 buff/cache  
>MiB Swap:    980.0 total,    980.0 free,      0.0 used.    722.7 avail Mem  
>  
>    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND  
>    907 vagrant   20   0    2592   1916   1812 S   1.7   0.2   0:36.90 reptyr  
>    894 vagrant   20   0   11848   4008   3348 R   0.3   0.4   0:00.69 top  

## 14)

>vagrant@vagrant:~$ echo string | sudo tee /root/new_file  
>string  

Команда `tee` читает из стандартного ввода `stdin` и записывает в стандартный  
вывод `stdout` и файл или несколько файлов одновременно, указанных в качестве  
параметров. Здесь `tee` получает через пайп string от `stdout` команды `echo`,  
а так как `tee` запущена под суперпользователем, то может создать файл `/root/new_file`  
и записать в него string.  
