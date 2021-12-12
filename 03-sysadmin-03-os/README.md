# 03-sysadmin-03-os

## 1)  
`cd` делает системный вызов `chdir`  
>vagrant@sandbox:~$ strace -e chdir /bin/bash -c 'cd /tmp'  
>chdir("/tmp")                           = 0  
>+++ exited with 0 +++  

Действительно, `strace` выводит все системные вызовы программы, их параметры и результат  
выполнения в стандартный поток ошибок.  
>vagrant@sandbox:\~$ strace -e chdir /bin/bash -c 'cd /tmp' 2>strace.log  
>vagrant@sandbox:\~$ cat strace.log  
>chdir("/tmp")                           = 0  
>+++ exited with 0 +++  

## 2)  
Команда `file` выполняет определение типов переданных элементов файловой системы  
(файлов, директорий, ссылок, именованных каналов и сокетов). Поищем файлы, которые  
она открывает, т.е. использует системный вызов `openat`.  

>vagrant@sandbox:\~$ strace -e trace=openat file /dev/tty  
>...  
>vagrant@sandbox:\~$ strace -e trace=openat file /dev/sda  
>...  
>vagrant@sandbox:~$ strace -e trace=openat file /bin/bash  
>openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3  
>openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3  
>openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3  
>openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3  
>openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3  
>openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3  
>openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3  
>openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3  
>openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)  
>openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3  
>openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3  
>openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3  

Во всех трех случаях список обращений одинаковый. Поиск в интернетах дал:  
The file command identifies the type of a file using, among other tests, a test for  
whether the file begins with a certain magic number. The file `/usr/share/misc/magic`  
specifies what magic numbers are to be tested for, what message to print if a particular  
magic number is found, and additional information to extract from the file.  

База - это файл `/usr/share/misc/magic.mgc`, `/etc/magic.mgc` - не существует, `/etc/magic` - пустой.

## 3)  
>vagrant@sandbox:\~$ ping localhost > f_stderr &  
>[2] 1700  
>  
>vagrant@sandbox:\~$ ps -a  
>    PID TTY          TIME CMD  
>   1451 pts/0    00:00:00 man  
>   1461 pts/0    00:00:00 pager  
>   1700 pts/0    00:00:00 ping  
>   1704 pts/0    00:00:00 ps  
>     
>vagrant@sandbox:~$ ps aux | grep 1700  
>vagrant     1700  0.0  0.2   9772  2832 pts/0    S    21:39   0:00 ping localhost  
>vagrant     1707  0.0  0.0   8900   740 pts/0    R+   21:40   0:00 grep --color=auto 1700  
>  
>vagrant@sandbox:~$ lsof -p 1700  
>COMMAND  PID    USER   FD      TYPE DEVICE SIZE/OFF NODE NAME  
>ping    1700 vagrant  cwd   unknown                      /proc/1700/cwd (readlink: Permission denied)  
>ping    1700 vagrant  rtd   unknown                      /proc/1700/root (readlink: Permission denied)  
>ping    1700 vagrant  txt   unknown                      /proc/1700/exe (readlink: Permission denied)  
>ping    1700 vagrant NOFD                                /proc/1700/fd (opendir: Permission denied)  
>  
>vagrant@sandbox:~$ sudo lsof -p 1700 | grep f_stderr  
>ping    1700 vagrant    1w   REG  253,0    11443 131092 /home/vagrant/f_stderr  
>
>vagrant@sandbox:~$ cat f_stderr  
>PING localhost(localhost (::1)) 56 data bytes  
>64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.021 ms  
>64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.034 ms  
>  
>vagrant@sandbox:\~$ rm f_stderr  
>vagrant@sandbox:\~$ cat f_stderr  
>cat: f_stderr: No such file or directory  
>vagrant@sandbox:~$ sudo lsof -p 1700 | grep f_stderr  
>ping    1700 vagrant    1w   REG  253,0    15603 131092 /home/vagrant/f_stderr (deleted)  
>  
>vagrant@sandbox:~$ sudo cat /proc/1700/fd/1  
>PING localhost(localhost (::1)) 56 data bytes  
>64 bytes from localhost (::1): icmp_seq=1 ttl=64 time=0.021 ms  
>64 bytes from localhost (::1): icmp_seq=2 ttl=64 time=0.034 ms  
>  
>vagrant@sandbox:\~$ sudo -i  
>root@sandbox:\~# echo "" >/proc/1700/fd/1  
>root@sandbox:~# cat /proc/1700/fd/1  
>  
>64 bytes from localhost (::1): icmp_seq=749 ttl=64 time=0.037 ms  

Файл очистился, но продолжает заполняться, так как процесс работает.  

>root@sandbox:\~# kill 1700  
>root@sandbox:\~# exit  
>logout  
>[2]-  Terminated              ping localhost > f_stderr  

## 4)  
Когда процесс завершается, его ресурсы освобождаются операционной системой. Однако его  
запись в таблице процессов остается, пока родитель не вызовет wait(). Такой процесс  
становится зомби. Зомби процессы не забирают ни память, ни CPU, но занимают место в таблице  
процессов. Таблица процессов - это конечный ресурс, и когда избыточные зомби заполняют ее,  
никакие другие процессы не могут запускаться. Если родительский процесс выполняется от имени  
суперпользователя, для освобождения записей (перезапуска процесса) может потребоваться перезагрузка.  

## 5)  
>sudo apt-get update -y  
>sudo apt-get install -y bpfcc-tools  
>  
>vagrant@sandbox:~$ dpkg -L bpfcc-tools | grep sbin/opensnoop  
>/usr/sbin/opensnoop-bpfcc  
>  
>vagrant@sandbox:~$ /usr/sbin/opensnoop-bpfcc  
>modprobe: ERROR: could not insert 'kheaders': Operation not permitted  
>Unable to find kernel headers. Try rebuilding kernel with CONFIG_IKHEADERS=m (module)  
>chdir(/lib/modules/5.4.0-80-generic/build): No such file or directory  
>Traceback (most recent call last):  
>  File "/usr/sbin/opensnoop-bpfcc", line 180, in <module>  
>    b = BPF(text=bpf_text)  
>  File "/usr/lib/python3/dist-packages/bcc/__init__.py", line 347, in __init__  
>    raise Exception("Failed to compile BPF module %s" % (src_file or "<text>"))  
>Exception: Failed to compile BPF module <text>  
>  
>vagrant@sandbox:\~$ sudo -i  
>root@sandbox:\~# /usr/sbin/opensnoop-bpfcc -d 1  
>PID    COMM               FD ERR PATH  
>835    vminfo              4   0 /var/run/utmp  
>566    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services  
>566    dbus-daemon        17   0 /usr/share/dbus-1/system-services  
>566    dbus-daemon        -1   2 /lib/dbus-1/system-services  
>566    dbus-daemon        17   0 /var/lib/snapd/dbus-1/system-services/  

## 6)  
>vagrant@sandbox:~$ strace uname -a  
>...  
>uname({sysname="Linux", nodename="sandbox", ...}) = 0  

`uname` использует системный вызов `uname()`  

>vagrant@sandbox:\~$ sudo apt install manpages-dev  
>vagrant@sandbox:\~$ man 2 uname  
>63        Part of the utsname information is also accessible via /proc/sys/kernel/{ostype,  
>64        hostname, osrelease, version, domainname}.  
>  
>vagrant@sandbox:~$ cat /proc/sys/kernel/ostype  
>Linux  

## 7)  
Прочитал  man bash раздел Lists:  
В cmd1 && cmd2 cmd2 выполняется только в том случае , если cmd1 завершится успешно (возвращает 0).  
В cmd1 ; cmd2 cmd2 выполняется в любом случае.  
В cmd1 || cmd2 cmd2 выполняется только в том случае , если cmd1 завершится со статусом, не равным 0.  

`test -d FILE` - проверяет FILE существует и является директорией.  

`set -e` указывает оболочке выйти, если команда дает ненулевой статус выхода. Проще говоря, прекратит  
выполнение после первой ошибки в цепочке команд и оболочка завершит работу. `set -e` часто ставится  
вверху скрипта, чтобы убедиться, что скрипт будет остановлен при первой ошибке.  

>vagrant@sandbox:\~$ set -e  
>vagrant@sandbox:\~$ test -d /tmp/some_dir; echo Hi  
>Connection to 127.0.0.1 closed.  
>PS C:\Users\andrew\VagrantVM>  
>  
>vagrant@sandbox:\~$ set -e  
>vagrant@sandbox:\~$ test -d /tmp/some_dir && echo Hi  
>vagrant@sandbox:~$  

Вывод: `&&` использовать c `set -e` видимо имеет смысл, если нужно избежать вылета оболочки при  
неуспешном завершении работы команды `test`. При этом вторая команда не будет исполнена.  

## 8)  
`set -euxo pipefail`  
* -e указывает оболочке выйти, если команда дает ненулевой статус выхода.  
* -u обрабатывает не установленные или не определенные переменные как ошибки во время раскрытия  
параметра, за исключением специальных параметров, таких как знаки `*` или `@`.  
* -x: печатает команды и их аргументы во время выполнения.  
* -o pipefail возвращает значение конвейера команд как состояние последней команды, которая должна  
выйти с ненулевым статусом, или как ноль, если все команды выполнены успешно.  

`set -euxo pipefail` удобно применять для отладки скриптов или цепочек команд, покажет вывод ошибки,  
при этом завершит обработку скрипта на этой команде, если ошибка не была в последней команде.

## 9)  
`ps -o stat`
>PROCESS STATE CODES  
>       Here are the different values that the s, stat and state output specifiers  
>       (header "STAT" or "S") will display to describe the state of a process:  
>  
>               D    uninterruptible sleep (usually IO)  
>               I    Idle kernel thread  
>               R    running or runnable (on run queue)  
>               S    interruptible sleep (waiting for an event to complete)  
>               T    stopped by job control signal  
>               t    stopped by debugger during the tracing  
>               W    paging (not valid since the 2.6.xx kernel)  
>               X    dead (should never be seen)  
>               Z    defunct ("zombie") process, terminated but not reaped by its parent  
>  
>       For BSD formats and when the stat keyword is used, additional characters may be displayed:  
>  
>               <    high-priority (not nice to other users)  
>               N    low-priority (nice to other users)  
>               L    has pages locked into memory (for real-time and custom IO)  
>               s    is a session leader  
>               l    is multi-threaded (using CLONE_THREAD, like NPTL pthreads do)  
>               +    is in the foreground process group  

Наиболее часто встречающийся статус у процессов S (спящие, ожидающие события завершения процесса) и  
I (бездействующие процессы ядра).  

>vagrant@sandbox:\~$ ps -eo stat | wc -l  
>99  
>vagrant@sandbox:\~$ ps -eo stat | grep -c S  
>51  
>vagrant@sandbox:\~$ ps -eo stat | grep -c I  
>47  
>vagrant@sandbox:\~$ ps -eo stat | grep -c R  
>1  
