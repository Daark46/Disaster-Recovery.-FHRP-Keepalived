# Disaster-Recovery.-FHRP-Keepalived

### Задание 1  
Дана схема для Cisco Packet Tracer, рассматриваемая в лекции.  
На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)  
Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).  
Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.  
На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.  

#### Ответ: 
[Packet Tracer схема](https://github.com/Daark46/Disaster-Recovery.-FHRP-Keepalived/blob/main/Disaster1.pkt)

![alt text](https://github.com/Daark46/Disaster-Recovery.-FHRP-Keepalived/blob/main/1.png)
![alt text](https://github.com/Daark46/Disaster-Recovery.-FHRP-Keepalived/blob/main/2.png)

### Задание 2  
Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного файла.  
Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах  
Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.  
Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля  (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script  
На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html  

#### Ответ: 
MASTER CONF   
```
vrrp_track_process check_nginx {
process "nginx"
}
vrrp_instance VI_1 {
        state MASTER
        interface ens18
        virtual_router_id 15
        priority 255
        advert_int 1

        virtual_ipaddress {
              192.168.6.55/24
        }
        track_process {
                check_nginx
        }
        vrrp_script check_site {
            script "/home/sysadmin/check_site.sh"  # путь до скрипта
            interval 3  # Интервал проверки в секундах
            fall 2  # Количество последовательных неудачных попыток
            rise 2  # Количество последовательных успешных попыток
        }
}
```
BACKUP CONF  
```
vrrp_track_process check_nginx {
process "nginx"
}
vrrp_instance VI_1 {
        state BACKUP
        interface ens18
        virtual_router_id 15
        priority 200
        advert_int 1

        virtual_ipaddress {
              192.168.6.55/24
        }
        track_process {
                check_nginx
        }
        vrrp_script check_site {
            script "/home/sysadmin/check_site.sh"  # путь до скрипта
            interval 3  # Интервал проверки в секундах
            fall 2  # Количество последовательных неудачных попыток
            rise 2  # Количество последовательных успешных попыток
        }
}
```
SCRIPT MASTER  
```
#!/bin/bash
# Проверка доступности порта веб-сервера
nc -zv 192.168.6.251 80

# Проверка существования файла index.html в root-директории веб-сервера
if [ -f /var/www/html/index.html ]; then
    exit 0  # Файл существует, возвращаем успешный код
else
    exit 1  # Файл не существует, возвращаем код ошибки
fi
```
SCRIPT BACKUP  
```
#!/bin/bash
# Проверка доступности порта веб-сервера
nc -zv 192.168.6.250 80
# Проверка существования файла index.html в root-директории веб-сервера
if [ -f /var/www/html/index.html ]; then
    exit 0  # Файл существует, возвращаем успешный код
else
    exit 1  # Файл не существует, возвращаем код ошибки
fi
```

![alt text](https://github.com/Daark46/Disaster-Recovery.-FHRP-Keepalived/blob/main/1.1.jpeg)
![alt text](https://github.com/Daark46/Disaster-Recovery.-FHRP-Keepalived/blob/main/1.2.jpeg)

