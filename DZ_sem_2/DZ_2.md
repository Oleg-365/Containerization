lxc-create --name=con1 --template=ubuntu
создать контейнер с именем con1 по шаблону ubuntu
(без лог-файла)

команда:
lxc-create -n con2 -t ubuntu --logfile=HomeWork02/con2.log
создать контейнер с именем con1 по шаблону ubuntu, назначить лог-файл

В файлах конфигурации контейнеров были добавлены ограничение памяти 128М и 256М, а также автозапуск.

Отрывки из конфиг-файла
# additional
lxc.cgroup2.memory.max = 128M
lxc.start.auto = 1

# logging
lxc.log.file = /home/HomeWork02/con1.log
lxc.log.level = 1


lxc-start con2 --logfile=HomeWork02/con2.log --logpriority=NOTICE 
запустить контейнер с именем con2, назначить лог-файл, назначить уровень логирования 

lxc-start con1 --logfile=HomeWork02/con1.log --logpriority=NOTICE
запустить контейнер с именем con1, назначить лог-файл, назначить уровень логирования


На этом этапе оба контейнера подключились в общую сеть на мост lxcbr0. Был отключен dhcp в файлах /etc/netplan/0-lxc.yaml, соответственно, сеть работать перестала.
Далее хост-система была перезагружена, контейнеры запустились автоматически, без сети, с заданными ограничениями памяти.
Изменений в назначенных лог-файлах не возникло. Запись в лог-файлы после перезагрузки стала осуществляться после добавления опций логирования в конфигурационные файлы контейнеров.

После этого постепенно была настроена сеть средствами утилиты ip. 
Затем были изменены файлы конфигурации netplan контейнеров и хост-системы.
Общий результат: 
Контейнеры запускаются автоматически, пингуют друг-друга и хост-систему. Доступа в интернет из контейнеров нет. Логирование настроено на уровень DEBUG, с сохранением в файл на хост-системе.

Содержимое конфигурационного файла netplan контейнера con1:

# cat /var/lib/lxc/con1/rootfs/etc/netplan/10-lxc.yaml 
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0: 
      dhcp4: no
      addresses: [ 10.0.11.2/24 ]
      routes:
      - to: default
        via: 10.0.11.2
      - to: 10.0.11.0/24
        via: 10.0.11.2
      - to: 10.0.12.0/24
        via: 10.0.11.1

Содержимое конфигурационного файла netplan хост-системы:

# cat /etc/netplan/00-installer-config.yaml 
# This is the network config written by 'subiquity'
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true

  bridges:
    lxcbr1:
      dhcp4: false
      addresses: [ 10.0.11.1/24 ]
      routes:
      - to: 10.0.11.0/24
        via: 10.0.11.1
        on-link: true
    lxcbr2:
      dhcp4: false
      addresses: [ 10.0.12.1/24 ]
      routes:
      - to: 10.0.12.0/24
        via: 10.0.12.1
        on-link: true


