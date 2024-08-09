[README принципа работы от разработчика](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/docs/readme.txt)

Инструкция в процессе доработки..

Настройка zapret от bol-van на Keenetic
Инструкция от [@nik](https://t.me/nik_pushistov). 07.08.2024

[Установка системы пакетов репозитория Entware на Keenetic в USB-накопитель](https://help.keenetic.com/hc/ru/articles/360021214160-%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B-%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%BE%D0%B2-%D1%80%D0%B5%D0%BF%D0%BE%D0%B7%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D1%8F-Entware-%D0%BD%D0%B0-USB-%D0%BD%D0%B0%D0%BA%D0%BE%D0%BF%D0%B8%D1%82%D0%B5%D0%BB%D1%8C)

Подключение по SSH к роутеру через "putty", 192.168.1.1:22.

Далее левой кнопкой мыши нажимая на квадратики копируем код, а правой вставляем в putty.

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

Далее "enable ipv6 support?:" (Без разницы, либо N)
```
n
```

Далее "select mode:" (Выбираем 3)
```
3
```

Далее нажимаем N, затем enter, конфиг поправим позже:
```
n
```

Далее выбираем "wan interface:" (Например на keenetic giga kn-1011 это 18:ppp0)
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

Очищаем весь код удерживая CTRL+K, пока не станет пусто и вставляем код ниже. (Конфиг тестировался на Ростелеком, Краснодарский край, вам может не подойти, поэтому ниже расскажу про подбор параметров)
```
FWTYPE=iptables
SET_MAXELEM=522288
IPSET_OPT="hashsize 262144 maxelem $SET_MAXELEM"
IP2NET_OPT4="--prefix-length=22-30 --v4-threshold=3/4"
IP2NET_OPT6="--prefix-length=56-64 --v6-threshold=5"
AUTOHOSTLIST_RETRANS_THRESHOLD=3
AUTOHOSTLIST_FAIL_THRESHOLD=3
AUTOHOSTLIST_FAIL_TIME=60
AUTOHOSTLIST_DEBUGLOG=0
MDIG_THREADS=30
GZIP_LISTS=1
MODE=nfqws
MODE_HTTP=1
MODE_HTTP_KEEPALIVE=0
MODE_HTTPS=1
MODE_QUIC=1
MODE_FILTER=autohostlist
DESYNC_MARK=0x40000000
DESYNC_MARK_POSTNAT=0x20000000
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=6 --dpi-desync-ttl6=2 --dpi-desync-split-pos=1 --wssize 1:6 --dpi-desync-fooling=md5sig"
NFQWS_OPT_DESYNC_HTTP=""
NFQWS_OPT_DESYNC_HTTPS=""
NFQWS_OPT_DESYNC_HTTP6=""
NFQWS_OPT_DESYNC_HTTPS6=""
NFQWS_OPT_DESYNC_QUIC="--dpi-desync=fake --dpi-desync-repeats=6"
NFQWS_OPT_DESYNC_QUIC6=""
TPWS_OPT="--hostspell=HOST --split-http-req=method --split-pos=3 --oob"
FLOWOFFLOAD=donttouch
IFACE_WAN=ppp0
INIT_APPLY_FW=1
DISABLE_IPV6=1

```

Далее выбор за вами, если необходимо обойти только ютуб то переходим к (zapret-hosts-user.txt) в ином случае, переходим к альтернативному способу
```
nano /opt/zapret/ipset/zapret-hosts-user.txt
```

Заменяем строку на 5 доменов ютуба и сохраняем.
```
www.youtube.com
youtu.be
googlevideo.com
ytimg.com
nhacmp3youtube.com
```

Перезагружаемся командой reboot.
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

Цена вопроса 0 рублей.

Если все такие не заработало, открываем снова конфиг


Автоподбор параметров, у каждого могут быть индивидуальными
```
/opt/zapret/blockcheck.sh | tee /opt/zapret/blockcheck.txt
```

P.S.:

Автоконфиг находится по адресу (Сюда идет пополнение доменов на те к которым вы пытаетесь достучаться, например рутор, чуть позже он автоматически туда попадает):
```
nano /opt/zapret/ipset/zapret-hosts-auto.txt
```

Zapret можно перезапустить
```
/opt/zapret/init.d/sysv/zapret restart_daemons
```

Или есть еще такая команда
```
/opt/etc/init.d/S90-zapret restart
```

Но лучше reboot, если вносили изменения в config.

Перед настройкой нужно провести исследование какую бяку устроил вам ваш провайдер.

Нужно выяснить не подменяет ли он DNS и какой метод обхода DPI работает.
В этом вам поможет скрипт blockcheck.sh.
```
/opt/zapret/blockcheck.sh
```

Если DNS подменяется, но провайдер не перехватывает обращения к сторонним DNS, поменяйте DNS на публичный.
Например : 8.8.8.8, 8.8.4.4, 1.1.1.1, 1.0.0.1, 9.9.9.9
Если DNS подменяется и провайдер перехватывает обращения к сторонним DNS, настройте dnscrypt.
Еще один эффективный вариант - использовать ресолвер от yandex 77.88.8.88 на нестандартном порту 1253.
Многие провайдеры не анализируют обращения к DNS на нестандартных портах.

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

#zapret #bol-van #keenetic #youtube #обход #замедление #РКН
