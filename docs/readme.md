[README от разработчика](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/docs/readme.txt)



Настройка zapret от bol-van на Keentic
Инструкция от @nik_pushistov. 07.08.2024

[Установка системы пакетов репозитория Entware на Keenetic в USB-накопитель](https://help.keenetic.com/hc/ru/articles/360021214160-%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B-%D0%BF%D0%B0%D0%BA%D0%B5%D1%82%D0%BE%D0%B2-%D1%80%D0%B5%D0%BF%D0%BE%D0%B7%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D1%8F-Entware-%D0%BD%D0%B0-USB-%D0%BD%D0%B0%D0%BA%D0%BE%D0%BF%D0%B8%D1%82%D0%B5%D0%BB%D1%8C)

Подключение по SSH к роутеру через "putty", 192.168.1.1:22.

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

Переходим в каталог zapret и выполняем скрипт:
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

Теперь сделаем, что бы запрет стартовал при запуски Keenetic.
```
ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S90-zapret
```

Правим стартовый скрипт
```
nano /opt/zapret/init.d/sysv/zapret
```

Добавляем PATH и WS_USER под словами END INIT INFO
```
PATH=/opt/sbin:/opt/bin:/opt/usr/sbin:/opt/usr/bin:/usr/sbin:/usr/bin:/sbin:/bin
WS_USER=nobody
```

Создаем небольшой скрипт, чтобы Keenetic не забывал правила
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

Ну вот и все. Перезагружаем и проверяем.
```
reboot
```

P.S.:
Поправить конфиг zapret'а:
```
nano /opt/zapret/config
```

Заменяем весь текст если будут проблемы
```
MODE_HTTP=1
MODE_HTTP_KEEPALIVE=0
MODE_HTTPS=1
MODE_QUIC=1
MODE_FILTER=hostlist
NFQWS_OPT_DESYNC="--dpi-desync=fake,disorder2 --dpi-desync-ttl=3 --dpi-desync-fooling=badsum"
NFQWS_OPT_DESYNC_QUIC="--dpi-desync=fake"
```

Обход замедления ютуб:
Хост лист (zapret-hosts-user.txt)
```
nano /opt/zapret/ipset/zapret-hosts-user.txt
```

Автоконфиг:
```
nano /opt/zapret/ipset/zapret-hosts-auto.txt
```

Вставить домены ниже
```
www.youtube.com
youtu.be
googlevideo.com
ytimg.com
nhacmp3youtube.com
```

Соответственно, после добавления нужно запустить следующие скрипты:
```
/opt/zapret/ipset/clear_lists.sh
```

```
/opt/zapret/ipset/get_user.sh
/opt/zapret/ipset/get_config.sh
```

И можно перезапустить Zapret
```
/opt/zapret/init.d/sysv/zapret restart_daemons
```

Или есть еще такая команда
```
/opt/etc/init.d/S90-zapret restart
```

