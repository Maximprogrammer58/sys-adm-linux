# Отчет по лабораторной работе

**Студент:** Шаменков Максим Александрович 6413

## Среда выполнения
| Параметр | Значение |
|----------|----------|
| **Гипервизор** | VMware |
| **Образ Ubuntu Server** | 24.04.4 LTS |
| **Клиентская ОС** | Ubuntu Live Server |
| **Имя пользователя** | shamenkov |

---

## Задание 1. Развёртывание ВМ и подключение дисков

![alt text](img/image-1.png)

## Задание 2. Настройка ssh по ключу

### Уже настроено в лабе 1
![alt text](img/image-2.png)

## Задание 3. Настройка сети через netplan
![alt text](img/image-3.png)

![alt text](img/image-4.png)

![alt text](img/image-6.png)

![alt text](img/image-5.png)

## Задание 4. Изучение конфигурации дискового пространства виртуальной машины

![alt text](img/image-7.png)

![alt text](img/image-8.png)

## Задание 5. Объединение свободные неразмеченные диски ВМ в RAID-массив 5-го уровня

### Очистка дисков
![alt text](img/image-9.png)

### Создание RAID 5
![alt text](img/image-10.png)

### Проверка 
![alt text](img/image-11.png)

### Сохранение RAID
![alt text](img/image-12.png)

## Задание 6. LVM и постоянное монтирование в /raid/0
### Создать Physical Volume (PV)
![alt text](img/image-13.png)

### Создать Volume Group (VG)
![alt text](img/image-14.png)

### Создать Logical Volume (LV)
![alt text](img/image-15.png)

### Создать файловую систему
![alt text](img/image-16.png)

### Создать точку монтирования и смонтировать
![alt text](img/image-17.png)

### Делаем постоянно монтирование

#### Узнаем uuid
![alt text](img/image-19.png)

#### Сделали запис в файле sudo nano /etc/fstab
![alt text](img/image-18.png)

#### Монтируем и проверяем
![alt text](img/image-20.png)


## Задание 7. Средствами менеджера виртуальных машин добавить к ВМ новый диск; добавить диск в RAID-массив и в LVM

### Состояние перед добавлением нового диска
![alt text](img/image-21.png)

### Добавляем новый виртуальный диск
![alt text](img/image-22.png)
![alt text](img/image-23.png)
![alt text](img/image-24.png)

![alt text](img/image-25.png)

### Очищаем диск
```bash
sudo wipefs -a /dev/sde
```

### Добавляем в raid
```bash
sudo mdadm --add /dev/md0 /dev/sde
```
### Увеличиваем Raid
```bash
sudo mdadm --grow /dev/md0 --raid-devices=4
```

![alt text](img/image-26.png)

### Расширяем LVM

#### Обновляем PV
```bash
sudo pvresize /dev/md0
```

#### Расширяем LV
```bash
sudo lvextend -l +100%FREE /dev/shamenkov/lv0
```

#### Расширяем файловую систему
sudo resize2fs /dev/shamenkov/lv0

![alt text](img/image-27.png)


## Задание 9 — NFS

### Создать папку
sudo mkdir -p /raid/0/backup

### Настраиваем экспорт
sudo nano /etc/exports
![alt text](img/image-28.png)

### Применяем конфиг
sudo exportfs -rav

### Запускаем NFS + проверка
sudo systemctl enable --now nfs-kernel-server

![alt text](img/image-29.png)

### Создаем точку монтирования
sudo mkdir -p /mnt/nfs_test

### Смонтируем
sudo mount -t nfs 127.0.0.1:/raid/0/backup /mnt/nfs_test
![alt text](img/image-31.png)


### Проверка 
![alt text](img/image-30.png)

все работает в df -h 
127.0.0.1:/raid/0/backup → /mnt/nfs_test

## Задание 10 — Настроить резервное копирование содержимого домашнего каталога пользователя (/home/<user>) в каталог backup на сервере раз в 2 минуты, используя cron.

### Цель: каждые 2 минуты создается архив
/home/shamenkov → /raid/0/backup

### Создаем скрипт
sudo nano /usr/local/bin/backup.sh

![alt text](img/image-32.png)

### Делаем скрипт исполняемым
sudo chmod +x /usr/local/bin/backup.sh

### Проверяем сначала вручную
sudo chmod +x /usr/local/bin/backup.sh
ls -lh /raid/0/backup

### Отлично появился файл backup_2026-04-05_15-18-07.tar.gz

![alt text](img/image-33.png)

### Настраиваем cron

#### добавляем строку
*/2 * * * * /usr/local/bin/backup.sh

![alt text](img/image-34.png)

### Спустя 2 минуты проверяем результат. 
### Итог: проблема. пользователь shamenkov НЕ может писать в /raid/0/backup

### Исправляем владельца
sudo chown -R shamenkov:shamenkov /raid/0/backup

### Теперь ждем 2 минуты и проверяем 
#### Состояние до
![alt text](img/image-35.png)

#### спустя 2 минуты

![alt text](img/image-36.png)


## Задание 11 — iptables

### Смотрим правила в iptables

sudo iptables -L -n -v

![alt text](img/image-38.png)

### Запрещаем icmp трафик
sudo iptables -A INPUT -p icmp -j DROP

### Проверка ping не идет 
![alt text](img/image-39.png)


### Узнаем  ip-адрес хоста
![alt text](img/image-40.png)

### Блокировка выбранного IP=192.168.50.124

sudo iptables -A INPUT -s 192.168.50.124 -j DROP

### Результат:
Текущая сессия ssh закрылась

### Далее заблокируем подключаения по ssh
sudo iptables -A INPUT -p tcp --dport 22 -j DROP

### Проверка
![alt text](img/image-41.png)


## Задание 12 — удаление правил

sudo iptables -F

### Проверка 
![alt text](img/image-42.png)

### Проверка ssh 
![alt text](img/image-43.png)

### Проверка ping
![alt text](img/image-44.png)

## Задание 13 — Включить брандмауэр ufw и создать те же правила блокировки.

### Включаем UFW
sudo ufw enable

![alt text](img/image-45.png)


### Блокируем SSH
![alt text](img/image-46.png)

### Блокировка выбранного IP=192.168.50.124
![alt text](img/image-47.png)

### Блокируем ping

sudo nano /etc/ufw/before.rules

#### Находим строку 
-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT

#### Меняем на 
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP

![alt text](img/image-48.png)

### Применим изменения
sudo ufw reload

### Проверка
sudo ufw status numbered

![alt text](img/image-49.png)

## Задание 14 — Мониторинг и диагностика: использовать ss и nc для просмотра портов и проверки доступности.

### Проверка портов через ss
ss -tuln
![alt text](img/image-50.png)

### Проверка портов через nc
![alt text](img/image-51.png)


## Задание 15 — Использовать tcpdump для анализа трафика:
### Просмотр трафика
![alt text](img/image-52.png)

### Фильтр по порту (например SSH)
sudo tcpdump -i ens33 port 22
![alt text](img/image-54.png)

### Фильтр по ICMP (ping)
sudo tcpdump -i ens33 icmp
![alt text](img/image-55.png)

### Запись в файл
sudo tcpdump -i ens33 -w capture.pcap
![alt text](img/image-53.png)
![alt text](img/image-56.png)

## Задание 16 — iptraf-ng

### Установка
sudo apt install iptraf-ng -y

### Запуск
sudo iptraf-ng
![alt text](img/image-57.png)

![alt text](img/image-58.png)

### Мониторинг
![alt text](img/image-59.png)

## Задание 17 — На сервере настроить дополнительный сетевой интерфейс через netplan, статический IP 172.23.0.7/24.

![alt text](img/image-60.png)

## Задание 18 — В ufw открыть порты 139 и 445. Проверить доступность портов извне виртуальной машины с помощью nc

### Открываем порты 139 и 445

sudo ufw allow 139
sudo ufw allow 445

![alt text](img/image-61.png)

### Проерим через nc
![alt text](img/image-62.png)

## Задание 19 — openVPN

### Установка
sudo apt install openvpn easy-rsa


### Создали инфраструктуру сертификатов (PKI)
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
./easyrsa init-pki
./easyrsa build-ca

### Сгенерировали сертификаты на сервере
./easyrsa gen-req server nopass
./easyrsa sign-req server server

### Сгенерировали сертификаты на клиенте
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1

### Генерация параметров Диффи-Хеллмана
./easyrsa gen-dh

### Копирование файлов на сервер 
sudo cp pki/ca.crt /etc/openvpn/
sudo cp pki/private/server.key /etc/openvpn/
sudo cp pki/issued/server.crt /etc/openvpn/
sudo cp pki/dh.pem /etc/openvpn/

### Настройка конфиг файл server.conf
/etc/openvpn/server.conf

![alt text](img/image-76.png)

### Включение машрутизацию
net.ipv4.ip_forward=1

### Запуск сервер и провери работу
![alt text](img/image-77.png)

### Создание клиентский конфиг
![alt text](img/image-78.png)

### Передача файлов на клиент 
scp shamenkov@192.168.50.240:~/openvpn-ca/pki/ca.crt .
scp shamenkov@192.168.50.240:~/openvpn-ca/pki/issued/client1.crt .
scp shamenkov@192.168.50.240:~/openvpn-ca/pki/private/client1.key .
scp shamenkov@192.168.50.240:~/openvpn-ca/client1.ovpn .

### Передачи ovpn на клиент и запуск
sudo openvpn --config client1.ovpn
![alt text](img/image-79.png)



## Задание 20 — openVPN
### Установка
sudo apt install samba -y

### Создаем папку, которую будем расшаривать
sudo mkdir -p /raid/0/share

### Создаем тестовый файлик
echo "test file" | sudo tee /raid/0/share/test.txt

### Настроить права на папку
sudo chmod 777 /raid/0/share

![alt text](img/image-85.png)


### Отредактируем конфиг samba
sudo nano /etc/samba/smb.conf

### Добавим следующий раздел
[share]
   path = /raid/0/share
   browseable = yes
   writable = yes
   guest ok = yes
   read only = no

![alt text](img/image-86.png)


### Перезапускаем и проверяем состояние
![alt text](img/image-87.png)

### Проверяем, что порты слушаются
![alt text](img/image-88.png)

### Проверка работы smb
smbclient -L //172.23.0.7 -N
![alt text](img/image-89.png)

## Задание 21 — Настроить проброс портов 139 и 445 из сети 172.23.0.0/24 в VPN-сеть 10.8.0.0/24.

### Делаем проброс портов

### Проброс входящих соединений

sudo iptables -t nat -A PREROUTING -i ens33 -p tcp --dport 139 -j DNAT --to-destination 10.8.0.2:139
sudo iptables -t nat -A PREROUTING -i ens33 -p tcp --dport 445 -j DNAT --to-destination 10.8.0.2:445

### Разрешаем пересылку
sudo iptables -A FORWARD -p tcp -d 10.8.0.2 --dport 139 -j ACCEPT
sudo iptables -A FORWARD -p tcp -d 10.8.0.2 --dport 445 -j ACCEPT

### Включаем NAT
sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE

![alt text](img/image-90.png)

### Проверим на хосте
nc -zv 192.168.50.240 139
nc -zv 192.168.50.240 445

![alt text](img/image-91.png)


### Вывод

В ходе лабораторной работы была выполнена настройка различных сетевых сервисов в Linux. Были изучены и реализованы:

- технологии **RAID** и **LVM**;
- сетевой файловый доступ с использованием **NFS** и **Samba**;
- резервное копирование с помощью **cron**.

Дополнительно был настроен **VPN-сервер** на базе **OpenVPN**, обеспечивающий защищённое соединение между клиентом и сервером. На заключительном этапе был реализован **проброс портов** с использованием **iptables**, что позволило обеспечить доступ к сервисам внутри VPN-сети.

### Возникшие трудности

В процессе выполнения работы возникали трудности с настройкой сетевого взаимодействия, в частности:

- маршрутизация;
- работа firewall;
- корректная настройка проброса портов.

Также потребовалось разобраться с расположением сервисов (Samba на клиенте или сервере) и особенностями их взаимодействия через VPN.

### Полученные результаты

В результате были получены практические навыки:

- настройки сетевых служб;
- администрирования Linux-систем;
- диагностики сетевых проблем.