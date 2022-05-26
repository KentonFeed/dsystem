# dsystem
создать 3 виртуальные машины
1) ald (2 GB ОЗУ, 20 GB )
2) dlp (8 GB ОЗУ 150 GB ) 1 ядро 4 прцессора
3) cl  (2 GB ОЗУ, 20 GB )

установка драйверов


ald сервер
остальные клиенты (dlp, cl)

для обновления до 2й версии на dlp
и cl нужно установить диски
1) smolensk-1.6 в disk1
2) devel-smolensk-1.6 в disk2

создать диск
1.sudo mkdir -p /disk1
также с остальными.

установка диска
sudo cp -a /media/cdrom0/* /disk1
также с остальными.

после установки дисков, нужно исзменить 2 строчки

переходим
sudo nano /etc/apt/sources.list

меняем на:
deb file:///disk1/ smolensk main non-free contrib
deb file:///disk2/ smolensk main non-free contrib
все остальные строчки удалить)

монтируем диски по очереди
к dlp и cl
1. 20190222se16
каждый диск делаем по очереди
sudo mount /mnt/20190222se16.iso /media/cdrom
sudo apt-cdrom -m add

sudo umount /media/cdrom (эта команда не нужна, но была в мануале):)
далее будет вопрос о названии имени диска
20190222se16

далее вводим эти
sudo apt update
sudo apt dist-upgrade
sudo apt -f install

после этого подключаем
2. repository-update-dev
проделываем те же команды)

sudo cat /etc/astra_update_version
при норм раскладе, будет
update 2
bulletin 20190222se16

далее
Скопировать содержимое дисков
1) 20190222se16 в disk3
2) repository-update-dev в disk4
команды выше(установка и создание дисков)

переходим в sources.list
находится команда выше
меняем на
deb file:///disk3/ smolensk contrib main non-free
deb file:///disk4/ smolensk contrib main non-free
deb file:///disk1/ smolensk main non-free contrib
deb file:///disk2/ smolensk main non-free contrib

команда перезагрузки sudo reboot

на dlp
Создаем директорию distr и копируем в неё установщики(диск Astra.iso)
sudo mkdir -p /distr
sudo cp -a /media/cdrom0/* /distr

далее в директории с установщиков вводим команду
cd /distr
ls
зеленая строчка iwdm.6.11.5...... это хорошо
sudo bash iwtm-installer-6.11.5.1156 (нажмите клавишу tab)


По окончании установки перезагрузить ВМ и с любой другой машины проверить работоспособность подключением через Web-интерфейс
Вводом в адресную строку браузера IP адреса ВМ dlp
(надеюсь все прописали ip adress)

какой-то из ip ваших машин dlp или cl
вводим в браузере на ald переходим
будет сайт infowatch
логин officer
пароль xxXX1234


После успешной инсталляции TM переходим на машину cl
и повторяем пункт 3 (обновление системы до версии 2)
и пункт 4 (создание репозитория /distr).
Создание репозитория /disk3 и /disk4 не выполняем.

Переходим в директорию с установщиком IWTM
вводим команду распаковки архива:

sudo tar -xvzf iwdm.6.11.5 (нажмите клавишу tab)

и установки:
sudo ./install.sh dmserver:15101
 
где dmserver – имя ВМ с ОС WinServ2019, 15101 – порт
Продукт будет установлен в директорию /opt/iw/dmagent.

Проверить статус сервисов можно командой:
sudo systemctl status iwdm*

Редактируем файл hosts
-Добавляем запись Х.Х.Х.135 dmserver
(dmserver -  ВМ с ОС WinServer 2019 – 4-я ВМ)


После установки IWTM переходим на ВМ ALD и поднимаем службу NTP
sudo systemctl status ntp

запускаем службу
sudo systemctl enable ntp

проверить статус
sudo apt install fly-admin-ntp

редактировать
sudo nano /etc/ntp.conf
добавляем строки предварительно удалив 4е имеющиеся
#pick a different set every time it starts up
#pool: <http://www.pool.ntp.org/join.html>
server 127.127.1.0
fudge 127.127.1.0 stratum 10

прописываем третий октет

#Clients from this (example!) subnet have unlimited access. but only if
/////
restrict 192.168.126.0 mask 255.255.0 notrust

проверяем источник
ntpq -p

на клиентах вкл tp
sudo systemctl status ntp
sudo systemctl start ntp

изменяем файл
sudo nano /etc/ntp.conf
#pool: <http://www.pool.ntp.org/join.html>
server ald.demo.lab iburst

sudo service ntp restart
ntpq -p

После установки службы ntp через браузер
набираем в адресной строке IP-адрес dlp
для входа в веб-консоль используем

пользователя officer
пароль xxXX1234

Переходим по пути управление/лицензии
подгружаем лицензию Lic_IWTM_6_10.iso


После устанавливаем LDAP-синхронизацию

сервера demo.lab
сервера Astra Linux Directory
автоматическая
ежеминутно 15мин

demo.lab
вкл
389 порт
dc=demo, dc=lab
откл



Установка ssh соединения
systemctl enable ssh
systemctl start ssh


Настройка пакета

Проверить состояние сервиса:
systemctl status ssh

Конфигурация сервера хранится в файле /etc/ssh/sshd_config.

Чтобы изменения вступили в силу необходимо перезапустить сервис:
systemctl restart sshd

для входа на удаленную
ssh user1@ald

Где user1 – это пользователь под, которым заходим
ald – имя удаленной машины
8.2 Для изменения порта для ssh отредактировать файл

sudo nano /etc/ssh/sshd_config
Раскоментировать строку #Port 22 убрать #  и изменить 22 на нужный портщ
Перезапустить systemctl restart sshd

Что бы проверить вступили ли
в силу изменения выполнить команду
sudo netstat -turpan | grep ssh

Для запрета доступа всем ip адресам
кроме своих машин необходимо отредактировать файл
sudo nano /etc/hosts.allow

Добавить ip-адреса доверенных машин
sshd: 192.168.35.131, 192.168.35.132

также внести изменения в
sudo nano /etc/hosts.deny

добавить sshd: ALL
sshd: ALL

Повторить на всех ВМ с учетом адреса, выше приведен пример для ald
т.е. для dlp в hosts.allow указываем 192.168.Х.130 и 192.168.х.132
Для client 192.168.х.130 и 192.168.х.131


ВМ WinServer 2019
Изменить имя ВМ на dmserver (винда)

Добавить запись в файл hosts путь локал диск C
windows
system31
drivers
etc

там прописываем
192.168.35.131 dlp.demo.lab
10 подключение доступа к сети интернет
Для ВМ ALD добавляем второй сетевой адаптер
(переходим в оборудовыание и сетевой адаптер)
И настраиваем другое – NAT

В самой ОС AstraLinux ни каких изменений для адаптера не вносим
Переходим для редактирования 
sudo nano /etc/sysctl.conf

Ищем там строку #net.ipv4.ip_forward=1
и убираем знак комментария

Также необходимо узнать названия наших сетевых интерфейсов и их IP адреса.
Для этого введем следующую команду: sudo ifconfig -a
sudo ifconfig -a

Как мы видим адаптер eth1
имеет отличный от нашей сети IP-адрес – это и есть адаптер
cмотрящий в интернет.

Для перенаправление tcp порта 2222
на tcp порт 2223 другой машины
необходимо добавить следующие правила iptables:
sudo iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT

Это правило разрешает прохождение входящих пакетов внутрь сети.
Теперь опишем форвардинг (forwarding) пакетов:

Создание сертификата для https
Создаем директорию для хранения 
sudo mkdir -p /ssl1
cd /ssl1

создаем корневой ключ
openssl genrsa -out rootCA.key 2048

создаем корневой сертификат
openssl req -x509 -new -key rootCA.key -days 365 -out rootCA.crt
ru
khv
khv
demo
lab
192.168.35.130
demo@demo.com

Создаем сертификат подписаный нашим СА
openssl genrsa -out dlp.demo.lab.key 2048

Создаем запрос на сертификат.
Тут важно указать имя сервера: домен или IP
openssl req -new -key dlp.demo.lab.key -out dlp.demo.lab.csr

ru
khv
khv
demo
lab
192.168.35.131
demo@demo.com
xxXX1234
demo


и подписать запрос
на сертификат нашим корневым сертификатом
openssl x509 -req -in dlp.demo.lab.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out dlp.demo.lab crt -days 365

По заданию все сертификаты
должны быть представлены единым пакетом PKCS (.p12), для этого:

openssl pkcs12 -export -in rootCA.crt -inkey rootCA.key -out cert.p12

страница 14
