# Easy istall.
Эксперементальный ускоренный способ установки.

После вставки кода, ждем очереди на исполнение, после жмем ENTER

## 1. Установка.
```shell
opkg update
opkg install coreutils-sort curl git-http grep gzip ipset iptables kmod_ndms nano xtables-addons_legacy
cd /opt/tmp
git clone --depth=1 https://github.com/bol-van/zapret.git
cd zapret
./install_easy.sh
```
Далее будут предупреждать и спрашивать, продолжать ли установку, отвечаем Y, затем enter и так 3 раза.

Далее необходимо выбирать варианты для построения вашего конфига, но это бессмысленно, так как на 14 шаге вы все равно загружаете мой конфиг, который при желании конечно, можно будет отведактировать позже, поэтому везде нажимаем ENTER, пока не увидим надпись "press enter to continue", а затем снова жмем ENTER

## 2. Настройка.

```shell
ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S90-zapret
cd /opt/zapret/init.d/sysv
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/zapret/init.d/sysv/zapret
cd /opt/etc/ndm/netfilter.d
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/etc/ndm/netfilter.d/000-zapret.sh
chmod +x /opt/etc/ndm/netfilter.d/000-zapret.sh
cd /opt/etc/init.d
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/etc/init.d/S00fix
chmod +x /opt/etc/init.d/S00fix
rm -rf /opt/tmp/*
```

## 3. Настройка пользователя.

### (обновляемый пункт) Загружаем один из готовых конфигов Zapret, подходит для большинста провайдеров с pppoe (Тестировался на Ростелеком, Дом.ру, ИСС, Annex.PRO, МТС):
#### Основной
```shell
cd /opt/zapret
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/zapret/config
```

#### Альтернативный
```shell
cd /opt/zapret
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/zapret_alt/config
```

#### Упрощенный
```shell
cd /opt/zapret
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/zapret_lite/config
```

#### Тестовый
```shell
cd /opt/zapret
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/zapret_test/config
```

### (обновляемый пункт) Далее выбираем, что будем ускорять:

#### Если необходимо ускорить только youtube, загружаем:
```shell
cd /opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/youtube/zapret-hosts-user.txt
```

#### Если необходимо ускорить youtube и соц. сети f, i, t(x):
```shell
cd /opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/yfit/zapret-hosts-user.txt
```

#### Если необходимо ускорить более 160к доменов:
```shell
cd /opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/blacklist-russia/zapret-hosts-user.txt
```

#### Загружаем список исключений с доменами, которые могут работать некорректно (по желанию, например у кого проблемы с входом в сбербанк):
```shell
cd /opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/zapret-hosts-user-exclude.txt
```

## 4. Завершение и тестирование.
```shell
/opt/etc/init.d/rc.unslung restart
curl --connect-to ::speedtest.selectel.ru https://manifest.googlevideo.com/100MB -k -o/dev/null
```
