# Easy istall.

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

```shell

```
