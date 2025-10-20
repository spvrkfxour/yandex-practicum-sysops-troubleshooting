## Загружаем операционную систему
Ошибка:<br>
<img width="767" height="144" alt="image" src="https://github.com/user-attachments/assets/1e879770-0ab5-4e81-9fa5-5fdb2681a4c0" />

Решение:<br>
Ищем все блочные устройства<br>
<img width="442" height="365" alt="image" src="https://github.com/user-attachments/assets/7be84382-6f62-4603-aecc-226ea4391a01" />

sda2 среди них нет, но есть vda<br>
Посмотрим информацию о файловых системах<br>
```console
(initramfs) blkid
/dev/vda2: UUID="ed465c6e-049a-41c6-8e0b-c8da348a3577" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="8c548af8-0682-43e1-9da4-2db0a9f4935b"
/dev/vda3: UUID="crOrKJ-PUDA-2Qgm-zGhS-fZns-d1YX-3hZ2eM" TYPE="LVM2_member" PARTLABEL="trouble" PARTUUID="0cfe7489-6922-4ff2-8b84-468e2e5de46b"
/dev/vda1: PARTUUID="c66f751c-f027-41a2-ba00-3342a74eb0cb"
```
vda2 больше всего похожа на корневую файловую систему
```console
(initramfs) cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-5.15.0-101-generic root=/dev/sda2 ro console=tty0 console=ttyS0,115200n8 rootdelay=10 net.ifnames=0 biosdevname=0 console=ttyS0
```
Перезагрузим командой reboot -n -f и исправим root в конфигурации загрузчика GRUB2 на /dev/vda2<br>
<img width="774" height="305" alt="image" src="https://github.com/user-attachments/assets/3a21b985-dac0-42bc-aa41-3c79dd19d114" />

Сохраним изменения и перезагрузим Ctrl-X

## Первый вход
Ошибка:<br>
<img width="632" height="130" alt="image" src="https://github.com/user-attachments/assets/dbc17b53-c532-4004-ad0a-bfe7d440f7c0" />

Решение:<br>
В конфигурации загрузчика GRUB2 в linux добавляем init=/bin/bash и изменяем ro на rw. Диск будет смонтирован в режиме чтение/запись и вместо systemd запустится bash. Как и в прошлой ошибке нужно будет изменить root<br>
<img width="763" height="305" alt="image" src="https://github.com/user-attachments/assets/1b67ff5d-ec58-46d5-b19a-849a9fc28f4e" />

Сбрасываем пароль root командой passwd root и запустим обычную программу инициализации systemd командой exec /sbin/init<br>
Также после логина с новым паролем исправим конфигурацию загрузчика командой update-grub<br>
<img width="773" height="316" alt="image" src="https://github.com/user-attachments/assets/d3c3661b-3b56-476f-9e43-c88f9078f080" />

## Дисковые разделы
Ошибка:<br>
Нет логического тома sysadmin и ошибка с размером заголовка GPT<br>
<img width="767" height="72" alt="image" src="https://github.com/user-attachments/assets/fe8c79e1-8d31-4bd4-9e41-fa19d8bfe45e" />

Решение:<br>
Посмотрим блочные устройства<br>
<img width="771" height="348" alt="image" src="https://github.com/user-attachments/assets/e30118a5-9f6b-4046-9a41-5b698b8eacda" />

Исправим ошибку с GPT командой parted /dev/vda print и fix<br>
<img width="769" height="297" alt="image" src="https://github.com/user-attachments/assets/244115e1-9df4-4c42-a076-6746ec6acf82" />

Раздел /dev/vda3 является физическим диском группы томов LVM<br>
Посмотрим какие есть логические тома командой lvs<br>
<img width="773" height="97" alt="image" src="https://github.com/user-attachments/assets/8067e3c9-7c7c-4f94-89de-570d352e9b0b" />

Пакет lvm2 не установлен

## Устанавливаем пакеты
Ошибка:<br>
<img width="746" height="87" alt="image" src="https://github.com/user-attachments/assets/93c20ac4-096c-4600-a372-ea10b5f2662a" />

Решение:<br>
Посмотрим содержимое /etc/apt/sources.list<br>
<img width="765" height="96" alt="image" src="https://github.com/user-attachments/assets/40f805ae-3380-4da3-9b89-ad7037b9e0df" />

Тут должен быть список репозиториев<br>
Очистим sources.list и добавим список репозиториев для DISTRIB_CODENAME=jammy<br>
```console
cat << EOF | tee /etc/apt/sources.list
deb http://archive.ubuntu.com/ubuntu/ jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu/ jammy-updates main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu/ jammy-backports main restricted universe multiverse
EOF
```
Проверим<br>
<img width="779" height="148" alt="image" src="https://github.com/user-attachments/assets/db521cb7-0856-4625-abb9-5e501b42edab" />

Теперь другая ошибка

## Ищем корневую проблему установки пакетов
Ошибка:<br>
<img width="777" height="151" alt="image" src="https://github.com/user-attachments/assets/a96ba3b6-723a-4858-b847-5323a47494cb" />

Решение:<br>
Система не может разрешить доменные имена в IP-адреса. Проверим сеть<br>
<img width="775" height="357" alt="image" src="https://github.com/user-attachments/assets/bfff3716-3672-48be-9fd4-d7cc9e13b1d0" />

## Чиним сеть
Ошибка:<br>
<img width="770" height="81" alt="image" src="https://github.com/user-attachments/assets/d689581b-6068-4247-b699-c8988196f680" />

Решение:<br>
Поднимем интерфейс командой ip link set eth0 up. IP-адрес не появился<br>
Запросим IP-адрес у DHCP командой dhclient -v eth0. IP-адрес появился<br>
Посмотрим конфигурацию netplan<br>
<img width="773" height="332" alt="image" src="https://github.com/user-attachments/assets/b4572946-9a59-4b8a-8f3a-2772a3785678" />

MAC-адрес отличается. Исправим MAC-адрес в конфигурации через vim на d0:0d:16:c8:05:5b<br>
Применим изменения командами chmod 600 /etc/netplan/50-cloud-init.yaml и netplan apply<br>
Проверим<br>
<img width="778" height="520" alt="image" src="https://github.com/user-attachments/assets/18297491-5c71-4eef-b2cc-d270fc756172" />

## Продолжаем установку пакетов
Ошибка:<br>
Ошибка с DNS осталась<br>
<img width="775" height="82" alt="image" src="https://github.com/user-attachments/assets/9e92f5e4-8a3c-4330-8f3f-f173cad025d7" />

Решение:<br>
Так как ip пингуется ошибка должна быть в DNS резолвере

## Чиним резолвер
Ошибка:<br>
<img width="764" height="59" alt="image" src="https://github.com/user-attachments/assets/22d53bf7-bc38-4cdd-874c-21045ec28c33" />

Решение:<br>
Команды dig и host находят ip по доменным именам<br>
<img width="762" height="120" alt="image" src="https://github.com/user-attachments/assets/5a21f380-2170-4eeb-985d-da2037ecbd66" /><br>
<img width="778" height="481" alt="image" src="https://github.com/user-attachments/assets/58accdc7-ffca-44db-8852-4c59a45e5aa5" />

Команда getent hosts practicum.yandex.ru не показывает никаких результатов. Значит ошибка в конфигурации nsswitch.conf, так как команды dig и host не смотрят в этот конфиг, а работают напрямую с DNS сервером<br>
<img width="776" height="440" alt="image" src="https://github.com/user-attachments/assets/7fd46c15-6746-44d8-ad1b-a09c49cadbd0" />

В конфиге указано смотреть только в /etc/hosts. Исправляем на hosts: files dns<br>
Проверим<br>
<img width="774" height="89" alt="image" src="https://github.com/user-attachments/assets/a6b98dba-b531-441f-8a36-d574dd1d8f13" /><br>
<img width="779" height="393" alt="image" src="https://github.com/user-attachments/assets/6d06011c-c22a-478d-8a81-71e5f6b53f3a" />

## Продолжаем восстановление LVM
Ошибка:<br>
Нет логического тома sysadmin

Решение:<br>
Установим пакет lvm2 для работы с LVM<br>
В lvm смотрим есть ли созданные pv, vg и lv командами pvdisplay, vgdisplay и lvdisplay соответственно<br>
<img width="768" height="291" alt="image" src="https://github.com/user-attachments/assets/50b16a0a-1b86-4d92-923d-ba69a2a6c1e4" /><br>
<img width="777" height="554" alt="image" src="https://github.com/user-attachments/assets/aa7723f1-cdc8-4f6a-9dbf-a16699b7f956" /><br>
<img width="775" height="317" alt="image" src="https://github.com/user-attachments/assets/9d21da3b-0f8a-43cc-8d82-e736a255d9c7" /><br>
Логический том sysadmin есть, но помечен как не активный<br>
Активируем логический том командой lvchange -ay /dev/opt/sysadmin<br>
<img width="765" height="59" alt="image" src="https://github.com/user-attachments/assets/8e68bba1-9057-45db-8332-4faf8b9d5b7e" />

Том активирован

## Монтируем файловую систему
Ошибка:<br>
Логический том sysadmin не смонтирован

Решение:<br>
Смонтируем том в директорию /opt/sysadmin<br>
<img width="780" height="39" alt="image" src="https://github.com/user-attachments/assets/715dc94b-5a65-4eba-8d17-8c7bc563a126" />

Ошибка говорит о том что ФС тома повреждена

## Восстанавливаем файловую систему
Ошибка:<br>
ФС логического тома sysadmin повреждена

Решение:<br>
Считаем первый сектор тома - первые 512 байт. В первом секторе все метаданные<br>
<img width="774" height="138" alt="image" src="https://github.com/user-attachments/assets/9e9a0547-a62f-4854-ad2f-304333fb39d4" />

Команда file ничего не показала. Суперблок не содержит распознаваемой сигнатуры<br>
Посмотрим hex-дамп<br>
<img width="776" height="126" alt="image" src="https://github.com/user-attachments/assets/299194aa-c2e8-4c21-a9eb-770c2a6a3b0f" />

Установим пакет xfsprogs и попробуем восстановить XFS командой xfs_repair /dev/opt/sysadmin<br>
<img width="777" height="322" alt="image" src="https://github.com/user-attachments/assets/74d29b4f-63da-4b89-b51c-52ccd53bc729" />

Получилось<br>
<img width="772" height="131" alt="image" src="https://github.com/user-attachments/assets/3514f04a-d4ff-47fa-a1da-fd20d733455a" />

Ещё раз смонтируем том в директорию<br>
<img width="771" height="77" alt="image" src="https://github.com/user-attachments/assets/081968aa-a642-4ec3-be36-83a769ff4d22" />

Настроим автоматическое монтирование при загрузке<br>
```console
echo 'UUID=9f4973de-b7f8-46cd-910a-67bd466ec7f2 /opt/sysadmin xfs defaults 0 2' >> /etc/fstab
```

## Запускаем сервис trouble. Часть 1
Ошибка:<br>
При запуске сервиса<br>
<img width="729" height="80" alt="image" src="https://github.com/user-attachments/assets/bc5c7578-0b72-48fd-8b80-257115fbe5ee" />

Решение:<br>
Попробовать запустить сервис не с рута<br>
Создадим пользователя для этого сервиса командой useradd -m -s /bin/bash trouble  и systemd юнит файл для управления сервисом как службой<br>
<img width="768" height="505" alt="image" src="https://github.com/user-attachments/assets/8b72322d-5db9-4976-a99f-089ccb425828" />

И сделаем сервис исполняемым для юзера trouble командами
```console
chown trouble:trouble /opt/sysadmin/bin/trouble
chmod o+x /opt/sysadmin/
```

Перечитаем конфигурацию и запустим сервис
```console
systemctl daemon-reload
systemctl enable trouble
systemctl start trouble
systemctl status trouble --no-pager --full
```

Получаем ошибку при запуске сервиса<br>
<img width="740" height="199" alt="image" src="https://github.com/user-attachments/assets/1ae6c7f4-e19f-43e2-8ebd-d0e530a6dcb1" />

Для удобной отладки подключимся напрямую к VM по SSH

## Настраиваем пользователя и подключаемся к машине по SSH
Ошибка:<br>
Не подключиться напрямую к VM по SSH

Решение:<br>
Устанавливаем пакет openssh-server и запускаем сервис командой systemctl start ssh<br>
Настраиваем для пользователя trouble вход по ssh
```console
passwd trouble
visudo /etc/sudoers.d/trouble
su -s /bin/bash trouble
cd ~
mkdir .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
vim .ssh/authorized_keys
```

И временно даем права рута<br>
<img width="634" height="48" alt="image" src="https://github.com/user-attachments/assets/f7ed844c-dea1-41ff-9a8d-3b35fecef150" />

Пробуем подключиться по SSH напрямую к VM<br>
<img width="769" height="483" alt="image" src="https://github.com/user-attachments/assets/bc9aa8a9-be3c-4aee-91ef-677819d86f68" />

## Запускаем сервис trouble. Часть 2
Ошибка:<br>
<img width="736" height="200" alt="image" src="https://github.com/user-attachments/assets/ffa32927-24f6-4a8f-a4d6-586c59e4317c" />

Решение:<br>
Запустим сервис в интерактивном режиме командой /opt/sysadmin/bin/trouble и посмотрим в режиме отладки командой strace /opt/sysadmin/bin/trouble<br>
<img width="776" height="93" alt="image" src="https://github.com/user-attachments/assets/4ed66735-82ca-4cfe-b1cd-1e38fd9e7203" />

Закончились файловые дескрипторы<br>
Лимит можно узнать командой ulimit -n. Текущий лимит 512<br>
Проверим лимиты на использование ресурсов для юзеров в /etc/security/limits.conf<br>
<img width="526" height="63" alt="image" src="https://github.com/user-attachments/assets/9d4cfa08-d0c2-4c52-8fe1-f2fe3e5604a1" />

Увеличим лимиты до 2048<br>
Теперь другая ошибка при запуске сервиса<br>
<img width="779" height="204" alt="image" src="https://github.com/user-attachments/assets/b4d9b164-3a39-4fe0-90f4-289d223e624e" />

## Запускаем сервис trouble. Часть 3
Ошибка:<br>
<img width="776" height="202" alt="image" src="https://github.com/user-attachments/assets/c86d5a24-fcd0-4fe8-8c7a-4603f4016320" />

Решение:<br>
Не удается заблокировать файл /locks/lockfile.lock при старте сервиса<br>
Проверим заблокировал ли этот файл какой то другой сервис<br>
<img width="776" height="180" alt="image" src="https://github.com/user-attachments/assets/a2ad907c-7e5b-44f2-92fd-b241da8f852e" /><br>
<img width="776" height="220" alt="image" src="https://github.com/user-attachments/assets/d42ab051-004c-4f32-a9fc-cde35f5a93dd" />

Сервис создан специально чтобы заблокировать файл. Попробуем убить этот процесс командой sudo killall watcher и перезапустить сервис trouble<br>
Запретим автозапуск командой sudo systemctl disable locker.service<br>
Новая ошибка<br>
<img width="781" height="90" alt="image" src="https://github.com/user-attachments/assets/02682fb8-7dc7-4e41-b138-9b0f3d2debb5" />

## Запускаем сервис trouble. Часть 4
Ошибка:<br>
<img width="778" height="94" alt="image" src="https://github.com/user-attachments/assets/ef3e5bb2-9514-495e-9192-a1ffadfdcffb" />

Решение:<br>
Порт занят другим сервисом. Проверим командой sudo ss -nltp<br>
<img width="772" height="94" alt="image" src="https://github.com/user-attachments/assets/ee506beb-d68c-45da-8c8c-d5179eacce64" />

Попробуем посмотреть что за сервис и убить<br>
<img width="777" height="236" alt="image" src="https://github.com/user-attachments/assets/4187b072-61ce-41cb-876f-3469c71a209c" />

Сервис снова поднялся. Запретим автозапуск командой sudo systemctl disable echo.service и остановим сервис sudo systemctl stop echo.service<br>
Перезапустим сервис trouble. Можно также убрать службы echo и locker<br>

<img width="774" height="224" alt="image" src="https://github.com/user-attachments/assets/eadc7738-21b2-49ef-b6be-aa197a9aa884" />

Также от рута введем команду sudo loginctl enable-linger trouble чтобы сервис запускался без логина пользователя trouble, так как будет ошибка /run/user/1001/file ENOENT (No such file or directory)

## Завершаем траблшутинг
Ошибка:<br>
Запустить HTTP-сервис trouble на виртуальной машине и получить к нему доступ из браузера

Решение:<br>
Используем nginx чтобы перенаправить трафик на 127.0.0.1:8080<br>
Установим пакет nginx и создадим конфиг /etc/nginx/sites-enabled/trouble для сервиса<br>
<img width="559" height="171" alt="image" src="https://github.com/user-attachments/assets/dec03330-93d3-4b84-8e7f-153b1ddc202c" />

Рестартим nginx<br>
<img width="586" height="247" alt="image" src="https://github.com/user-attachments/assets/39e14937-5337-4238-8aac-44fa8f15ef2f" />
