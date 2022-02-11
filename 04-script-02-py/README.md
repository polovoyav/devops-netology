# 04-script-02-py

## Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | Никакое, а - целочисленная, b - строковая переменная, их нельзя сложить  |
| Как получить для переменной `c` значение 12?  | c = int(str(a)+b)  |
| Как получить для переменной `c` значение 3?  | c = a + int(b)  |

## Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

### Ваш скрипт:
```python
#!/usr/bin/env python3

import os

bash_command = ["cd /home/vagrant/test", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
#is_change = False - переменная не используется
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
#        break - прерывает цикл после первого выполнения
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@sandbox:~/test$ ./script.py
bash_script.sh
folder/index.html
index.html
ipcheck.log
script.py

vagrant@sandbox:~/test$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   bash_script.sh
        modified:   folder/index.html
        modified:   index.html
        modified:   ipcheck.log
        modified:   script.py

no changes added to commit (use "git add" and/or "git commit -a")
```

## Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import sys, os

pathto = sys.argv[1]
print ("Path to git repo: " + pathto)
bash_command = ["cd " + pathto, "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@sandbox:~/test$ cp ~/test/script.py /tmp

vagrant@sandbox:/tmp$ /tmp/script.py ~/test
Path to git repo: /home/vagrant/test
bash_script.sh
folder/index.html
index.html
ipcheck.log
script.py
```

## Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import socket, time

hdict = {"drive.google.com": "127.0.0.1",
         "mail.google.com": "127.0.0.1",
         "google.com": "127.0.0.1"
        }

while (1==1):
        for host, ip in hdict.items():
                ipnew = socket.gethostbyname(host)
                if ipnew == ip:
                        print(host, ", old ip: ", ip, ", new ip: ", ipnew)
                else:
                        message = str("\n[ERROR] " + host + " IP mismatch: " + ip + " - " + ipnew)
                        print(message)
                        with open ("/home/vagrant/test/google.log", "a") as logfile:
                                logfile.write(message)
                        hdict[host] = ipnew
        time.sleep(3)	
```

### Вывод скрипта при запуске при тестировании:
```
vagrant@sandbox:~/test$ ./script.py

[ERROR] drive.google.com IP mismatch: 127.0.0.1 - 173.194.220.194

[ERROR] mail.google.com IP mismatch: 127.0.0.1 - 64.233.162.83

[ERROR] google.com IP mismatch: 127.0.0.1 - 74.125.131.101

drive.google.com , old ip:  173.194.220.194 , new ip:  173.194.220.194

[ERROR] mail.google.com IP mismatch: 64.233.162.83 - 64.233.162.19

[ERROR] google.com IP mismatch: 74.125.131.101 - 74.125.131.138

drive.google.com , old ip:  173.194.220.194 , new ip:  173.194.220.194
mail.google.com , old ip:  64.233.162.19 , new ip:  64.233.162.19
google.com , old ip:  74.125.131.138 , new ip:  74.125.131.138
drive.google.com , old ip:  173.194.220.194 , new ip:  173.194.220.194
mail.google.com , old ip:  64.233.162.19 , new ip:  64.233.162.19
google.com , old ip:  74.125.131.138 , new ip:  74.125.131.138
drive.google.com , old ip:  173.194.220.194 , new ip:  173.194.220.194
mail.google.com , old ip:  64.233.162.19 , new ip:  64.233.162.19
google.com , old ip:  74.125.131.138 , new ip:  74.125.131.138


vagrant@sandbox:~/test$ cat google.log

[ERROR] drive.google.com IP mismatch: 127.0.0.1 - 173.194.220.194

[ERROR] mail.google.com IP mismatch: 127.0.0.1 - 64.233.162.83

[ERROR] google.com IP mismatch: 127.0.0.1 - 74.125.131.101

[ERROR] mail.google.com IP mismatch: 64.233.162.83 - 64.233.162.19

[ERROR] google.com IP mismatch: 74.125.131.101 - 74.125.131.138
```
