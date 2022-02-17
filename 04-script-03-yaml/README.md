# 04-script-03-yaml

## Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```

Не хватает запятой и кавычек:

```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            },
            { "name" : "second",
            "type" : "proxy",
            "ip" : "71.78.22.43"
            }
        ]
    }
```

## Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

### Ваш скрипт:
```python
#!/usr/bin/env python3

import socket, time, json, yaml

hdict = { "drive.google.com" : "127.0.0.1", "mail.google.com" : "127.0.0.1", "google.com" : "127.0.0.1" }

while (1==1):
        hlist = []
        for host in hdict:
                ip = socket.gethostbyname(host)
                hlist.append({ host : ip })
                if hdict[host] == ip:
                        print(host, ", old ip: ", hdict[host], ", new ip: ", ip)
                else:
                        print(str("[ERROR] " + host + " IP mismatch: " + hdict[host] + " - " + ip))
                        with open('google.yml', 'w') as ym:
                                ym.write(yaml.dump(hlist, indent=4, explicit_start=True, explicit_end=True))
                        with open('google.json', 'w') as js:
                                js.write(json.dumps(hlist, indent=4))
                        hdict[host] = ip
        time.sleep(3)
```

### Вывод скрипта при запуске при тестировании:
```bash
vagrant@sandbox:~/test$ ./script.py
[ERROR] drive.google.com IP mismatch: 127.0.0.1 - 74.125.131.194
[ERROR] mail.google.com IP mismatch: 127.0.0.1 - 74.125.131.83
[ERROR] google.com IP mismatch: 127.0.0.1 - 64.233.164.113
drive.google.com , old ip:  74.125.131.194 , new ip:  74.125.131.194
mail.google.com , old ip:  74.125.131.83 , new ip:  74.125.131.83
[ERROR] google.com IP mismatch: 64.233.164.113 - 64.233.164.139
drive.google.com , old ip:  74.125.131.194 , new ip:  74.125.131.194
mail.google.com , old ip:  74.125.131.83 , new ip:  74.125.131.83
google.com , old ip:  64.233.164.139 , new ip:  64.233.164.139
drive.google.com , old ip:  74.125.131.194 , new ip:  74.125.131.194
mail.google.com , old ip:  74.125.131.83 , new ip:  74.125.131.83
google.com , old ip:  64.233.164.139 , new ip:  64.233.164.139
drive.google.com , old ip:  74.125.131.194 , new ip:  74.125.131.194
mail.google.com , old ip:  74.125.131.83 , new ip:  74.125.131.83
google.com , old ip:  64.233.164.139 , new ip:  64.233.164.139
KeyboardInterrupt
```

### json-файл(ы), который(е) записал ваш скрипт:
```bash
vagrant@sandbox:~/test$ nano google.json
```
```json
[
    {
        "drive.google.com": "74.125.131.194"
    },
    {
        "mail.google.com": "74.125.131.83"
    },
    {
        "google.com": "64.233.164.139"
    }
]
```

### yml-файл(ы), который(е) записал ваш скрипт:
```bash
vagrant@sandbox:~/test$ nano google.yml
```
```yaml
---
-   drive.google.com: 74.125.131.194
-   mail.google.com: 74.125.131.83
-   google.com: 64.233.164.139
...
```
