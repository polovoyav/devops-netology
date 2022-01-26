# 03-sysadmin-06-net1  

## 1)  
>vagrant@sandbox:~/test$ telnet stackoverflow.com 80  
>Trying 151.101.193.69...  
>Connected to stackoverflow.com.  
>Escape character is '^]'.  
>GET /questions HTTP/1.0  
>HOST: stackoverflow.com  
>  
>HTTP/1.1 301 Moved Permanently  
>cache-control: no-cache, no-store, must-revalidate  
>location: https://stackoverflow.com/questions  
>x-request-guid: c367c869-8e0e-4069-9624-cf72420c323d  

...  


Код перенаправления "301 Moved Permanently" протокола HTTP показывает, что запрошенный ресурс был окончательно  
перемещён в URL, указанный в заголовке location, в нашем случае на https://stackoverflow.com/questions.  

## 2)  
Код 307.  

>Request URL: http://stackoverflow.com/  
>Request Method: GET  
>Status Code: 307 Internal Redirect  

Дольше всего обрабатывался второй запрос, на https://stackoverflow.com/, 433 ms. Скриншот приложил.  

## 3)  
Данные с https://whoer.net/ru: 79.126.61.2  

Еще вариант:  

>vagrant@sandbox:~/test$ wget -qO- eth0.me  
>79.126.61.2  
  
## 4)  
>vagrant@sandbox:~/test$ whois 79.126.61.2  
>role:           OJSC Rostelecom, Nizhny Novgorod  

...  

>route:          79.126.32.0/19  
>descr:          NMTS Autonomous System  
>origin:         AS25405  
>mnt-by:         NMTS-MNT  
>created:        2009-02-06T07:49:54Z  
>last-modified:  2009-03-11T07:53:11Z  
>source:         RIPE  

...

## 5)  
>vagrant@sandbox:~/test$ traceroute -AIn 8.8.8.8  
>traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets  
> 1  10.0.2.2 [*]  0.408 ms  0.325 ms  0.262 ms  
> 2  192.168.1.1 [*]  2.601 ms  2.542 ms  2.968 ms  
> 3  188.254.2.28 [AS12389]  28.599 ms  28.542 ms  28.484 ms  
> 4  * * *  
> 5  * * *  
> 6  * * *  
> 7  108.170.250.83 [AS15169]  13.758 ms  13.109 ms *  
> 8  * * *  
> 9  216.239.57.222 [AS15169]  29.831 ms  29.903 ms  29.820 ms  
>10  216.239.63.27 [AS15169]  29.010 ms  28.952 ms  28.895 ms  
>11  * * *  
>12  * * *  
>13  * * *  
>14  * * *  
>15  * * *  
>16  * * *  
>17  * * *  
>18  * * *  
>19  * * *  
>20  * 8.8.8.8 [AS15169]  29.323 ms  30.050 ms  

## 6)  
>vagrant@sandbox:~/test$ mtr -zn 8.8.8.8  
>                              My traceroute  [v0.93]  
>sandbox (10.0.2.15)                                       2022-01-26T19:53:03+0000  
>Keys:  Help   Display mode   Restart statistics   Order of fields   quit  
>                                          Packets               Pings  
>  Host                                   Loss%   Snt   Last   Avg  Best  Wrst StDev  
> 1. AS???    10.0.2.2                    0.0%   141    1.5   0.8   0.5   4.1   0.4  
> 2. AS???    192.168.1.1                 0.0%   141    2.3   2.9   2.0  14.9   1.7  
> 3. AS12389  188.254.2.28                0.0%   141    5.4  23.1   3.3 428.8  47.1  
> 4. AS12389  188.254.2.29               44.6%   140    3.2   4.3   2.9  18.5   2.6  
> 5. (waiting for reply)  
> 6. (waiting for reply)  
> 7. AS15169  108.170.250.83             44.6%   140   12.4  17.7  11.9  62.2  10.1  
> 8. AS15169  209.85.249.158             62.1%   140   31.9  31.3  29.2  46.6   3.4  
> 9. AS15169  216.239.57.222              0.0%   140   28.4  28.5  26.5  47.7   2.9  
>10. AS15169  216.239.63.27               0.0%   140   30.0  29.9  28.5  37.5   1.5  
>11. (waiting for reply)  
>12. (waiting for reply)  
>13. (waiting for reply)  
>14. (waiting for reply)  
>15. (waiting for reply)  
>16. (waiting for reply)  
>17. (waiting for reply)  
>18. (waiting for reply)  
>19. (waiting for reply)  
>20. AS15169  8.8.8.8                    18.7%   140   28.9  30.2  26.4  50.0   2.7  


Максимальное время задержки на 3-м узле `AS12389`, `188.254.2.28`, `428.8 ms`.

## 7)  
>vagrant@sandbox:~/test$ dig +short @8.8.8.8 -q dns.google -t NS  
>ns4.zdns.google.  
>ns1.zdns.google.  
>ns3.zdns.google.  
>ns2.zdns.google.  
>  
>vagrant@sandbox:~/test$ dig +short @8.8.8.8 dns.google A  
>8.8.8.8  
>8.8.4.4  

## 8)  
>vagrant@sandbox:~/test$ dig +short -x 8.8.8.8  
>dns.google.  
>  
>vagrant@sandbox:~/test$ dig +short -x 8.8.4.4  
>dns.google.  

