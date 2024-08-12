## [README принципа работы от разработчика](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/docs/readme.txt)

## Инструкция в процессе доработки..

Настройка Zapret от bol-van на Keenetic от [@nik](https://t.me/nik_pushistov)

## Краткое описание

Существуют режимы обхода TPWS и NFQWS. Режим NFQWS имеет ряд преимуществ пред режимом TPWS – больше параметров модификации сетевых пакетов, а также возможность модификации трафика по протоколу QUIC. Это не VPN и не прокси, это минисофт для обмана (подмены) пакетов, повышается анонимность, а также не ломает то, что и так хорошо работало.

### Подготовка

(обязательно) Проверить, что установленны все пакеты под категорией OPKG в наборах компонентов в настройках.

(обязательно) Необходимо заменить в настройках роутера DNS-серверы провайдера на публичные. Прописываем IP адреса DNS - 8.8.8.8 и 8.8.4.4, и не забываем поставить галочку "игнорировать DNS предлагаемые провайдером интернета".

(обязательно) Установить entware на маршрутизатор по инструкции [Установка системы пакетов репозитория Entware на USB-накопитель](https://help.keenetic.com/hc/ru/articles/360021214160) или не очень хороший метод [Установка OPKG Entware на встроенную память роутера](https://help.keenetic.com/hc/ru/articles/360021888880).

(опционально) Можно настроить, это позволит паралельно обходить блокировки от США, для таких сервисов как xbox, ps network, если находитесь в России [Прокси-серверы DNS-over-TLS и DNS-over-HTTPS для шифрования DNS-запросов](https://help.keenetic.com/hc/ru/articles/360007687159).

Все дальнейшие команды выполняются не в cli роутера, а **в среде entware**.

Подключение по SSH к роутеру через "putty", 192.168.1.1:22.

Далее левой кнопкой мыши нажимая на квадратики копируем код, а правой вставляем в putty.

## Установка

Логин: 
```
root
```

Пароль: 
```
keenetic
```

Обновляем opkg пакеты:
```
opkg update
```

Устанавливаем пакеты:

```
opkg install coreutils-sort curl git-http grep gzip ipset iptables kmod_ndms nano xtables-addons_legacy
```

Переходим в tmp и скачиваем Zapret:
```
cd /opt/tmp
git clone --depth=1 https://github.com/bol-van/zapret.git
```

<details>
  <summary>Если выдает ошибку -  fatal: could not create leading directories of 'zapret/.git', откройте этот раскрывающий список</summary>

  1. Переходим https://github.com/bol-van/zapret.git, скачайте zip архив нажатием на code далее download zip
  2. Далее выгружаем архив через интерфейс роутера по пути: Приложения>флэшка>папка opkg>папка tmp>вверху "загрузить файл в выбранную папку", также это можно сдлеать через SMB протокол, если вы активировали его заранее
  3. Открываем putty, авторизуемся
  4. Вводим
     ```
     cd /opt/tmp
     unzip zapret-master.zip
     ```
     ```
     cd zapret-master
     ```
     ```
     sh install_easy.sh
     ```
  5. Пошел процесс, далее по инструкции..

</details>

Для начала узнаем имя внешнего сетевого интерфейса (WAN) на роутере. Его можно узнать воспользовавшись командой ifconfig, которая выведет все сетевые интерфейсы в системе. Просто находим тот интерфейс, у которого будет ваш внешний IP адрес. В моем случае – это ppp0, в вашем же, если у вам провайдер выдает адрес по статике или DHCP, eth3.
```
ifconfig
```

Переходим в каталог Zapret и выполняем скрипт:
```
cd zapret
./install_easy.sh
```

Далее (будут спрашивать, 3 раза отвечаем Y затем enter, после каждого раза):
```
y
```

Далее "select firewall type:" (Выбираем 1)
```
1
```

Далее "enable ipv6 support?:" (Y)
```
y
```

Далее "select mode:" (Выбираем 3)
```
3
```

Далее нажимаем N, затем enter, конфиг поправим позже:
```
n
```

Далее выбираем "wan interface:" (Например на keenetic giga kn-1011 с провайдером Ростелеком это ppp0, то есть если при настройки использовались учетные данные pppoe логин и пароль, в ином случае eth3)
```
18
```

Дальше жмем Enter 3 раза

Далее "select filtering:" (Выбираем 4 вариант затем enter. Автохостлист, пригодится для автопополнения запрещенных доменов РКН)
```
4
```

Далее press enter to continue (Нажимаем)

Удаляем ненужное.
```
rm -rf /opt/tmp/zapret
rm -rf /opt/tmp/zapret-master
rm -rf /opt/tmp/zapret-master.zip
```

Теперь автозапуск Zapret при включении роутера.
```
ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S90-zapret
```

Правим стартовый скрипт
```
nano /opt/zapret/init.d/sysv/zapret
```

Добавляем PATH и WS_USER под строкой END INIT INFO (После как добавили нажимаем CTRL+X, затем Y, затем enter и так везде где требуется вставка)
```

PATH=/opt/sbin:/opt/bin:/opt/usr/sbin:/opt/usr/bin:/usr/sbin:/usr/bin:/sbin:/bin
WS_USER=nobody

```

Создаем небольшой скрипт, чтобы роутер не забывал правила
```
nano /opt/etc/ndm/netfilter.d/000-zapret.sh
```

Вставляем весь текст
```
#!/bin/sh
[ "$type" == "ip6tables" ] && exit 0
[ "$table" != "mangle" ] && exit 0
/opt/zapret/init.d/sysv/zapret restart-fw
```

Исполняем
```
chmod +x /opt/etc/ndm/netfilter.d/000-zapret.sh
```

Переводим net.netfilter.nf_conntrack_checksum в 0.
```
nano /opt/etc/init.d/S00fix
```

Вставляем весь текст
```
#!/bin/sh
start() {
    echo 0 > /proc/sys/net/netfilter/nf_conntrack_checksum
}
stop() {
    echo 1 > /proc/sys/net/netfilter/nf_conntrack_checksum
}
case "$1" in
    'start')
        start
        ;;
    'stop')
        stop
        ;;
    *)
        stop
        start
        ;;
esac
exit 0
```

Исполняем
```
chmod +x /opt/etc/init.d/S00fix
```

Правим конфиг Zapret.
```
nano /opt/zapret/config
```

Очищаем весь код удерживая CTRL+K, пока не станет пусто и вставляем код ниже, пока не сохраняем, читаем дальше (Конфиг тестировался на Ростелеком, Краснодарский край, вам может не подойти, поэтому ниже расскажу про подбор параметров)
```
# this file is included from init scripts
# change values here

# can help in case /tmp has not enough space
#TMPDIR=/opt/zapret/tmp

# redefine user for zapret daemons. required on Keenetic
WS_USER=nobody

# override firewall type : iptables,nftables,ipfw
FWTYPE=iptables

# options for ipsets
# maximum number of elements in sets. also used for nft sets
SET_MAXELEM=522288
# too low hashsize can cause memory allocation errors on low RAM systems , even if RAM is enough
# too large hashsize will waste lots of RAM
IPSET_OPT="hashsize 262144 maxelem $SET_MAXELEM"
# dynamically generate additional ip. $1 = ipset/nfset/table name
#IPSET_HOOK="/etc/zapret.ipset.hook"

# options for ip2net. "-4" or "-6" auto added by ipset create script
IP2NET_OPT4="--prefix-length=22-30 --v4-threshold=3/4"
IP2NET_OPT6="--prefix-length=56-64 --v6-threshold=5"
# options for auto hostlist
AUTOHOSTLIST_RETRANS_THRESHOLD=3
AUTOHOSTLIST_FAIL_THRESHOLD=3
AUTOHOSTLIST_FAIL_TIME=60
# 1 = debug autohostlist positives to ipset/zapret-hosts-auto-debug.log
AUTOHOSTLIST_DEBUGLOG=0

# number of parallel threads for domain list resolves
MDIG_THREADS=30

# ipset/*.sh can compress large lists
GZIP_LISTS=1
# command to reload ip/host lists after update
# comment or leave empty for auto backend selection : ipset or ipfw if present
# on BSD systems with PF no auto reloading happens. you must provide your own command
# set to "-" to disable reload
#LISTS_RELOAD="pfctl -f /etc/pf.conf"

# override ports
#HTTP_PORTS=80-81,85
#HTTPS_PORTS=443,500-501
#QUIC_PORTS=443,444

# CHOOSE OPERATION MODE
# MODE : nfqws,tpws,tpws-socks,filter,custom
# nfqws : nfqws for dpi desync
# tpws : tpws transparent mode
# tpws-socks : tpws socks mode
# filter : no daemon, just create ipset or download hostlist
# custom : custom mode. should modify custom init script and add your own code
MODE=nfqws
# apply fooling to http
MODE_HTTP=1
# for nfqws only. support http keep alives. enable only if DPI checks for http request in any outgoing packet
MODE_HTTP_KEEPALIVE=0
# apply fooling to https
MODE_HTTPS=1
# apply fooling to quic
MODE_QUIC=1
# none,ipset,hostlist,autohostlist
MODE_FILTER=autohostlist

# CHOOSE NFQWS DAEMON OPTIONS for DPI desync mode. run "nfq/nfqws --help" for option list
DESYNC_MARK=0x40000000
DESYNC_MARK_POSTNAT=0x20000000
NFQWS_OPT_DESYNC="--dpi-desync=fake,disorder2 --dpi-desync-split-pos=1 --dpi-desync-ttl=0 --dpi-desync-fooling=md5sig,badsum --dpi-desync-repeats=6 --dpi-desync-any-protocol --dpi-desync-cutoff=d4" 
#NFQWS_OPT_DESYNC_HTTP=
#NFQWS_OPT_DESYNC_HTTPS=
#NFQWS_OPT_DESYNC_HTTP6=
#NFQWS_OPT_DESYNC_HTTPS6=
NFQWS_OPT_DESYNC_QUIC="--dpi-desync=fake,disorder2 --dpi-desync-repeats=6 --dpi-desync-ttl=0  --dpi-desync-any-protocol --dpi-desync-cutoff=d4 --dpi-desync-fooling=md5sig,badsum"
#NFQWS_OPT_DESYNC_QUIC6=

# CHOOSE TPWS DAEMON OPTIONS. run "tpws/tpws --help" for option list
TPWS_OPT="--hostspell=HOST --tlsrec=sni --split-pos=1 --oob --disorder"

# openwrt only : donttouch,none,software,hardware
FLOWOFFLOAD=donttouch

# openwrt: specify networks to be treated as LAN. default is "lan"
#OPENWRT_LAN="lan lan2 lan3"
# openwrt: specify networks to be treated as WAN. default wans are interfaces with default route
#OPENWRT_WAN4="wan vpn"
#OPENWRT_WAN6="wan6 vpn6"

# for routers based on desktop linux and macos. has no effect in openwrt.
# CHOOSE LAN and optinally WAN/WAN6 NETWORK INTERFACES
# or leave them commented if its not router
# it's possible to specify multiple interfaces like this : IFACE_LAN="eth0 eth1 eth2"
# if IFACE_WAN6 is not defined it take the value of IFACE_WAN
#IFACE_LAN=eth0
IFACE_WAN=ppp0
#IFACE_WAN6="ipsec0 wireguard0 he_net"

# should start/stop command of init scripts apply firewall rules ?
# not applicable to openwrt with firewall3+iptables
INIT_APPLY_FW=1
# firewall apply hooks
#INIT_FW_PRE_UP_HOOK="/etc/firewall.zapret.hook.pre_up"
#INIT_FW_POST_UP_HOOK="/etc/firewall.zapret.hook.post_up"
#INIT_FW_PRE_DOWN_HOOK="/etc/firewall.zapret.hook.pre_down"
#INIT_FW_POST_DOWN_HOOK="/etc/firewall.zapret.hook.post_down"

# do not work with ipv4
#DISABLE_IPV4=1
# do not work with ipv6
DISABLE_IPV6=0

# select which init script will be used to get ip or host list
# possible values : get_user.sh get_antizapret.sh get_combined.sh get_reestr.sh get_hostlist.sh
# comment if not required
GETLIST=

```

Так как вы вставили мой конфиг, вы также перенесли одну индивидуальную настройку, а именно параметр:
```
IFACE_WAN=ppp0
```

Прописываем после IFACE_WAN=ваш интерфейс, например
```
IFACE_WAN=eth3
```

Далее выбор за вами, если необходимо обойти только ютуб то переходим к (zapret-hosts-user.txt) в ином случае, переходим к альтернативному способу
```
nano /opt/zapret/ipset/zapret-hosts-user.txt
```

Заменяем строку на то, что ниже и сохраняем.
```
www.youtube.com
youtube.com
youtu.be
googlevideo.com
ytimg.com
nhacmp3youtube.com
```

На этом настройка Zapret роутере завершена и можно перезагружать:
```
reboot
```

Проверяем ютуб на каком нибудь 8K ролике! Между прочим даже с таким способом будет автоматический обход и других сайтов, но немного иначе.

Альтенативный способ: (В данный момент находимся на этом этапе, файл закрываем CTRL+X N)
```
nano /opt/zapret/ipset/zapret-hosts-user.txt
```

Качаем файл по ссылке ниже с доменами или открываем в RAW и копируем по пути на флэшку файл целиком, либо открываем файл и вставляем с заменой все строки opkg\zapret\ipset\zapret-hosts-user.txt
[zapret-hosts-user.txt](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/docs/zapret-hosts-user.txt)

Перезагружаемся командой reboot.
```
reboot
```

Проверяем инстаграм на телефоне, торренты и прочую запрещенку)



Если не заработало, открываем снова конфиг
```
nano /opt/zapret/config
```

Обращаем внимание на строку:
```
NFQWS_OPT_DESYNC="--dpi-desync=fake,disorder2 --dpi-desync-split-pos=1 --dpi-desync-ttl=0 --dpi-desync-fooling=md5sig,badsum --dpi-desync-repeats=6 --dpi-desync-any-protocol --dpi-desync-cutoff=d4" 
```

Можно попробовать менять значение ttl от 0 до 12 или же сменить значения dpi-desync с split2 на disorber2 ниже несколько примеров:
```
NFQWS_OPT_DESYNC="--dpi-desync=split2"
```
```
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=2 --dpi-desync-fooling=badsum"
```
```
NFQWS_OPT_DESYNC="--dpi-desync=fake,disorder2 --dpi-desync-ttl=3 --dpi-desync-fooling=badsum"
```
```
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=6 --dpi-desync-fooling=badsum"
```
```
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=3 --dpi-desync-fooling=badsum"
```
```
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=6 --dpi-desync-fooling=md5sig"
```
```
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=6 --dpi-desync-ttl6=2 --dpi-desync-split-pos=1 --wssize 1:6 --dpi-desync-fooling=md5sig"
```

После этого исполняем все что ниже и так по кругу, пока не добьетесь результата

Очистка таблицы ip адресов
```
/opt/zapret/ipset/clear_lists.sh
```

Получить новые данные списка хостов
```
/opt/zapret/ipset/get_user.sh
```
Получить новые данные конфига
```
/opt/zapret/ipset/get_config.sh
```
Перезагрузить Zapret
```
/opt/zapret/init.d/sysv/zapret restart
```

### Также можно воспользоваться автоподбором. Автоподбор параметров, у каждого могут быть индивидуальными
Следует прогнать blockcheck по нескольким заблокированным сайтам и выявить общий характер блокировок.
Разные сайты могут быть заблокированы по-разному, нужно искать такую технику, которая работает на большинстве.
Чтобы записать вывод blockcheck.sh в файл, выполните : ./blockcheck.sh | tee /tmp/blockcheck.txt
```
/opt/zapret/blockcheck.sh | tee /opt/zapret/blockcheck.txt
```

Проанализируйте какие методы дурения DPI работают, в соответствии с ними настройте /opt/zapret/config.

Имейте в виду, что у провайдеров может быть несколько DPI или запросы могут идти через разные каналы
по методу балансировки нагрузки. Балансировка может означать, что на разных ветках разные DPI или
они находятся на разных хопах. Такая ситуация может выражаться в нестабильности работы обхода.
Дернули несколько раз curl. То работает, то connection reset или редирект. blockcheck.sh выдает
странноватые результаты. То split работает на 2-м. хопе, то на 4-м. Достоверность результата вызывает сомнения.
В этом случае задайте несколько повторов одного и того же теста. Тест будет считаться успешным только,
если все попытки пройдут успешно.

При использовании autottl следует протестировать как можно больше разных доменов. Эта техника
может на одних провайдерах работать стабильно, на других потребуется выяснить при каких параметрах
она стабильна, на третьих полный хаос, и проще отказаться.

Blockcheck имеет 3 уровня сканирования.
Цель режима quick - максимально быстро найти хоть что-то работающее.
standard дает возможность провести исследование как и на что реагирует DPI в плане методов обхода.
force дает максимум проверок даже в случаях, когда ресурс работает без обхода или с более простыми стратегиями.

## P.S.:

Автохостлист находится по адресу (Сюда идет пополнение доменов на те к которым вы пытаетесь достучаться, например рутор, чуть позже он автоматически туда попадает):
```
nano /opt/zapret/ipset/zapret-hosts-auto.txt
```

Удаленное подключение по ssh к Keenetic если вы под admin, в консоли вводим:
```
exec sh
```

Запустить NFQWS и проверять работоспособность с помощью команды:
```
/opt/zapret/init.d/sysv/zapret start
```

Для перезагрузки NFQWS использовать команду:
```
/opt/zapret/init.d/sysv/zapret restart
```

Для остановки NFQWS использовать команду:
```
/opt/zapret/init.d/sysv/zapret stop
```

#zapret #bol-van #keenetic #youtube #обход #замедление #РКН
