# 03-sysadmin-01-terminal
# 1)
Установлен VirtualBox-6.1.30-148432-Win.exe.
# =============================================

# 2)
Установлен vagrant_2.2.19_x86_64.msi и vagrant-vmware-utility_1.0.21_x86_64.msi
# =============================================

# 3)
Установлен Windows Terminal.
# =============================================

# 4)
Установлен через vagrant Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)

Дополнительно выполнено:
$ echo $PS1
\[\e]0;\u@\h: 
\w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$

Добавил текущее время в 24-часовом формате:
$ PS1="\A \[\e]0;\u@\h: 
\w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$"
# =============================================

# 5)
RAM 512 Мб, CPU 1 ядро, vHDD 64 Гб, video 4 Мб.
# =============================================

# 6)
Добавил в конфигурационный файл Vagrantfile строчки:
config.vm.provider "virtualbox" do |v|
	v.memory = 1024
	v.cpus = 2
end
# =============================================

# 7)
Сделал.
# =============================================

# 8)
history:
HISTFILESIZE, line 964. Задает максимальное число строк в файле истории.
HISTSIZE, line 982. Задает число запоминаемых командв команде history.
ignoreboth, line 952. Директива ignoreboth объединяет две директивы ignorespace 
и ignoredups. ignorespace не сохраняет в истории записи, начинающиеся с пробела, 
ignoredups не сохраняет записи, совпадающие с имеющимися в истории.
# =============================================

# 9)
{} - зарезервированные системой слова, применяются для выполнения групповых 
команд в текущей среде шелла.
{} применяются в командах bash в качестве сокращения (brace expansion), для 
последовательной подстановки значений из списка или диапазона.
{} применяется в скриптах в операторах, функциях.
# =============================================

# 10)
touch file_{0..100000} - создались

touch file_{0..300000} - не создались
-bash: /usr/bin/touch: Argument list too long

rm -f file_{0..300000} - удалить тоже не получается.
-bash: /usr/bin/rm: Argument list too long

Если увеличить размер стека для пользователя командой ulimit -s 65535, то 
touch file_{0..300000} - выполняется успешно.
# =============================================

# 11)
[[ expression ]] возвращает 0 или 1 после проверки условного выражения между [[ ]]
Выражение -d file - True if file exists and is a directory.
[[ -d /tmp ]] проверяет наличие директории /tmp и возвращает истину, если она существует.
# =============================================

# 12)
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

$ type -a bash
bash is /usr/bin/bash
bash is /bin/bash

$ mkdir /tmp/new_path_directory
$ cp /bin/bash /tmp/new_path_directory
$ PATH=/tmp/new_path_directory:$PATH
$ echo $PATH
/tmp/new_path_directory:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
:/usr/games:/usr/local/games:/snap/bin

$ type -a bash
bash is /tmp/new_path_directory/bash
bash is /usr/bin/bash
bash is /bin/bash
# =============================================

# 13)
man at, man batch
at и batch ставят в очередь, проверяют и удаляют задачи.
at выполняет команды в назначенное время
batch выполняет команды, когда загрузка системы падает ниже 1.5
# =============================================

# 14)
vagrant halt
# =============================================

