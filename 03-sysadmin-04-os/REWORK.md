# 03-sysadmin-04-os  
  
## Доработка задания 1.  
  
>Предлагаю уточнить как именно в службу будут передаваться дополнительные опции.   
>Замечу, что речь идёт не о переменных окружения, а об опциях (параметрах) запуска службы.  
  
### Смотрим состояние запуска службы. 
   
>vagrant@sandbox:~$ sudo systemctl status node-exporter  
>Warning: The unit file, source configuration file or drop-ins of node-exporter.ser>● node-exporter.service - node_exporter for machine metrics  
>     Loaded: loaded (/etc/systemd/system/node-exporter.service; enabled; vendor pr>     Active: active (running) since Mon 2022-01-31 12:24:40 UTC; 5min ago  
>   Main PID: 827 (node_exporter)  
>      Tasks: 4 (limit: 1071)  
>     Memory: 14.2M  
>     CGroup: /system.slice/node-exporter.service  
>             └─827 /usr/local/bin/node_exporter  

### Меняем unit-файл службы.  
  
>vagrant@sandbox:~$ sudo nano /etc/systemd/system/node-exporter.service  
>>[Unit]  
>>Description=node_exporter for machine metrics  
>>Wants=network-online.target  
>>After=network-online.target  
>>  
>>[Service]  
>>Restart=always  
>>User=node_exporter  
>>Group=node_exporter  
>>Type=simple  
>>EnvironmentFile=/etc/default/node_exporter  
>>ExecStart=/usr/local/bin/node_exporter $EXTRA_OPTS  
>>  
>>[Install]  
>>WantedBy=multi-user.target  
  
### Меняем Environment-файл.  
  
>vagrant@sandbox:~$ sudo nano /etc/default/node_exporter  
>> #EnvironmentFile for node_exporter  
>> EXTRA_OPTS="--log.level=debug"  
  
### Перезапускаем службу, видим, что она запустилась с параметром.   
  
>vagrant@sandbox:~$ sudo systemctl daemon-reload  
>vagrant@sandbox:~$ sudo systemctl restart node-exporter  
>vagrant@sandbox:~$ sudo systemctl status node-exporter  
>● node-exporter.service - node_exporter for machine metrics  
>     Loaded: loaded (/etc/systemd/system/node-exporter.service; enabled; vendor pr>     Active: active (running) since Mon 2022-01-31 12:32:18 UTC; 5s ago  
>   Main PID: 1881 (node_exporter)  
>      Tasks: 4 (limit: 1071)  
>     Memory: 2.3M  
>     CGroup: /system.slice/node-exporter.service  
>             └─1881 /usr/local/bin/node_exporter --log.level=debug
