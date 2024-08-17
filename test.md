## [README принципа работы от разработчика>>](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/docs/readme.txt)

# Подробная обновляемая [@nik](https://t.me/nik_pushistov) инструкция настройки репозитория [Zapret](https://github.com/bol-van/zapret) от [bol-van](https://github.com/bol-van) на Keenetic.

## Краткое описание

Автономное, без задействования сторонних серверов, средство противодействия DPI.
Может помочь обойти замедления сайтов http(s), сигнатурный анализ tcp и udp протоколов,
например с целью замедления YouTube.

Существуют режимы обхода TPWS и NFQWS. Режим NFQWS имеет ряд преимуществ пред режимом TPWS. Больше параметров модификации TCP соединения на уровне пакетов. Реализуется через обработчик очереди NFQUEUE и raw сокеты, а также возможность модификации трафика по протоколу QUIC.

### P.S. Это не VPN и не прокси, это средство для обмана (подмены) пакетов tcp и udp протокола, не режет скорость, повышается анонимность, а также не ломает то, что и так хорошо работало.

## Технические требования

(рекомендуется) Keenetic Viva kn-1912 с 256мб ОЗУ и дороже с наличием USB порта.

(минимально) Keenetic Extra, Viva с менее 256мб ОЗУ лучше использовать ограниченный свежий [хостлист](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/YFIT/zapret-hosts-user), где будут только Youtube, F, I, T(X), Ror, Rer - это позволит значительно снизить нагрузку с ОЗУ.

## Подготовка

(обязательно) Проверить, что установленны все пакеты под категорией OPKG в наборах компонентов в настройках, также протокол IPv6 и Модули ядра подсистемы Netfilter (он появится в списке пакетов только после установки пакета "Протокол IPv6").

(обязательно) Необходимо заменить в настройках роутера DNS-серверы провайдера на публичные. Прописываем IP адреса DNS - 8.8.8.8 и 77.88.8.8, и не забываем поставить галочку "игнорировать DNS предлагаемые провайдером интернета".

(обязательно) [Установка системы пакетов репозитория Entware на USB-накопитель](https://help.keenetic.com/hc/ru/articles/360021214160). 
Или не очень хороший метод [Установка OPKG Entware на встроенную память роутера](https://help.keenetic.com/hc/ru/articles/360021888880).

(опционально) Можно настроить, но не обязательно, если совсем ничего не поможет. [Прокси-серверы DNS-over-TLS и DNS-over-HTTPS для шифрования DNS-запросов](https://help.keenetic.com/hc/ru/articles/360007687159).

Все дальнейшие команды выполняются не в cli роутера, а **в среде entware**.

### Подключение по SSH к роутеру через "putty", например если адрес вашего роутера 192.168.1.1 а порт 22 или 222 если активировали сервер SSH

Далее левой кнопкой мыши нажимая на квадратики копируя код, а правой вставляем в putty.

## Установка

### Логин: 
```shell
root
```

### Пароль: 
```shell
keenetic
```

### Обновляем opkg пакеты:
```shell
opkg update
```

### Устанавливаем пакеты:

```shell
opkg install coreutils-sort curl git-http grep gzip ipset iptables kmod_ndms nano xtables-addons_legacy
```

### Для начала узнаем имя внешнего сетевого интерфейса (WAN) на роутере. Его можно узнать воспользовавшись командой ifconfig, которая выведет все сетевые интерфейсы в системе. Просто находим тот интерфейс, у которого будет ваш внешний IP адрес. В моем случае – это ppp0, в вашем же, если у вам провайдер выдает адрес по статике или DHCP, eth3. Запоминаем.
```shell
ifconfig
```

### Переходим в tmp и скачиваем Zapret:
```shell
cd /opt/tmp
git clone --depth=1 https://github.com/bol-van/zapret.git
```

<details>
    <summary>Если выдает ошибку -  fatal: could not create leading directories of 'zapret/.git', откройте этот раскрывающий список, в ином случае, следуйте инструкции ниже</summary>

  1. Переходим https://github.com/bol-van/zapret.git, скачайте zip архив нажатием на code далее download zip
  2. Далее выгружаем архив через интерфейс роутера по пути: Приложения>флэшка>папка opkg>папка tmp>вверху "загрузить файл в выбранную папку", также это можно сдлеать через SMB протокол, если вы активировали его заранее
  3. Открываем putty, авторизуемся
  4. Вводим
     ```shell
     cd /opt/tmp
     unzip zapret-master.zip
     ```
     ```shell
     cd zapret-master
     ```
     ```shell
     sh install_easy.sh
     ```
  5. Пошел процесс, далее по инструкции..

</details>

### Переходим в каталог Zapret и выполняем скрипт (если у вас до этого вышла ошибка, пункт пропускаем, вы уже это сделали, смотрим наже):
```shell
cd zapret
./install_easy.sh
```

### Далее (будут спрашивать, 3 раза отвечаем Y затем enter, после каждого раза):
```shell
y
```

### Далее "select firewall type:" (Выбираем 1)
```shell
1
```

### Далее "enable ipv6 support?:" (Y)
```shell
y
```

### Далее "select mode:" (Выбираем 3)
```shell
3
```

### Далее нажимаем N, затем enter, конфиг поправим позже:
```shell
n
```

### Далее выбираем "wan interface:" (Например на keenetic giga kn-1011 с провайдером Ростелеком это ppp0, то есть если при настройки использовались учетные данные pppoe логин и пароль, в ином случае eth3)
```shell
18
```

### Дальше жмем Enter 3 раза

### Далее "select filtering:" (Выбираем 4 вариант затем enter. Автохостлист, пригодится для автопополнения "замедленных" доменов)
```shell
4
```

### Далее press enter to continue (Нажимаем)

### Удаляем ненужное.
```shell
rm -rf /opt/tmp/zapret
rm -rf /opt/tmp/zapret-master
rm -rf /opt/tmp/zapret-master.zip
```

### Теперь автозапуск Zapret при включении роутера.
```shell
ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S90-zapret
```

### Загружаем готовый стартовый скрипт с dnsmasq внутри.
```shell
cd /opt/zapret/init.d/sysv
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/zapret/init.d/sysv/zapret
```

### Загружаем готовый скрипт, чтобы роутер не забывал правила.
```shell
cd /opt/etc/ndm/netfilter.d
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/etc/ndm/netfilter.d/000-zapret.sh
```

### Исполняем
```shell
chmod +x /opt/etc/ndm/netfilter.d/000-zapret.sh
```

### Загружаем готовый скрипт для перевода net.netfilter.nf_conntrack_checksum в 0.
```shell
cd /opt/etc/init.d
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/etc/init.d/S00fix
```

### Исполняем.
```shell
chmod +x /opt/etc/init.d/S00fix
```

### Загружаем готовый конфиг Zapret, подходит для большинста провайдеров (Тестировался на Ростелеком, ЮФО).
```shell
cd /opt/zapret
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/zapret/config
```

### Так как вы вставили мой конфиг, вы также перенесли несколько настроек а именно:
```bash
IFACE_WAN=ppp0
MODE_FILTER=autohostlist
```

### Если у вас роутер с ОЗУ менее 256мб или авторизация у провайдера не pppoe, а динамика или статика по динамике, то правим конфиг Zapret, в ином случае, пропускаем шаги.
```shell
nano /opt/zapret/config
```

#### Если у вас роутер с ОЗУ менее 256мб, ищем строку MODE_FILTER=autohostlist и на:
```bash
MODE_FILTER=hostlist
```

#### Прописываем интерфейс провайдера в строке IFACE_WAN=ваш интерфейс, например:
```bash
IFACE_WAN=eth3
```

#### Также можно прописать несколько используемых WAN интерфейсов через пробел, например:
```bash
IFACE_WAN="eth3 usb0"
```

#### (опционально, надо проверить) Если необходимо направить трафик не на всю сеть, а например только гостевую сеть, vlan, опеределенный порт или l2tp, уберите решетку в #IFACE_LAN=eth0 и укажите 1 или несколько интерфейсов (узнать какой интерфейс за что отвечает, можно все той же командой ifconfig)
```bash
IFACE_LAN=br0
```
#### Или br0 - это бридж основной сети, br1 - vlan гостевой сети, ezcfg0 - частный VPN сервер IKEv2/IPsec и т.д.
```bash
IFACE_LAN="br0 br1 ezcfg0"
```

Если закончили править, сохраняем CTRL+S, затем CTRL+X для выхода.

### Если необходимо ускорить только youtube, загружаем.
```shell
/opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/youtube/zapret-hosts-user.txt
```

### Если необходимо ускорить youtube и соц. сети f, y, t(x).
```shell
/opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/yfit/zapret-hosts-user.txt
```

### Если необходимо ускорить все что можно, то.
```shell
/opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/blacklist-russia/zapret-hosts-user.txt
```

### Перезагружаемся командой reboot.
```shell
reboot
```

## Готово, проверяем.



## Если не заработало, открываем снова конфиг
```shell
nano /opt/zapret/config
```

### Обращаем внимание на строку:
```bash
NFQWS_OPT_DESYNC="--dpi-desync=fake,disorder2 --dpi-desync-split-pos=1 --dpi-desync-ttl=0 --dpi-desync-fooling=md5sig,badsum --dpi-desync-repeats=6 --dpi-desync-any-protocol --dpi-desync-cutoff=d4" 
```

### Можно попробовать менять значение ttl от 0 до 12 или же сменить значения dpi-desync с split2 на disorber2 ниже несколько примеров:
```bash
NFQWS_OPT_DESYNC="--dpi-desync=split2"
```
```bash
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=2 --dpi-desync-fooling=badsum"
```
```bash
NFQWS_OPT_DESYNC="--dpi-desync=fake,disorder2 --dpi-desync-ttl=3 --dpi-desync-fooling=badsum"
```
```bash
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=6 --dpi-desync-fooling=badsum"
```
```bash
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=3 --dpi-desync-fooling=badsum"
```
```bash
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=6 --dpi-desync-fooling=md5sig"
```
```bash
NFQWS_OPT_DESYNC="--dpi-desync=fake,split2 --dpi-desync-ttl=6 --dpi-desync-ttl6=2 --dpi-desync-split-pos=1 --wssize 1:6 --dpi-desync-fooling=md5sig"
```

### После этого исполняем все что ниже и так по кругу, пока не добьетесь результата

### Очистка таблицы ip адресов
```shell
/opt/zapret/ipset/clear_lists.sh
```

### Получить новые данные списка хостов
```shell
/opt/zapret/ipset/get_user.sh
```
### Получить новые данные конфига
```shell
/opt/zapret/ipset/get_config.sh
```
### Перезагрузить Zapret
```shell
/opt/zapret/init.d/sysv/zapret restart
```

### Также можно воспользоваться автоподбором. Автоподбор параметров, у каждого могут быть индивидуальными
Следует прогнать blockcheck по нескольким замедленным сайтам и выявить общий характер замедленний.
Разные сайты могут быть замедленны по-разному, нужно искать такую технику, которая работает на большинстве.
Чтобы записать вывод blockcheck.sh в файл, выполните : ./blockcheck.sh | tee /tmp/blockcheck.txt
```shell
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

### Автохостлист находится по адресу (Сюда идет пополнение доменов на те к которым вы пытаетесь достучаться, например рутор, чуть позже он автоматически туда попадает):
```shell
nano /opt/zapret/ipset/zapret-hosts-auto.txt
```

### Удаленное подключение по ssh к Keenetic если вы под admin, в консоли вводим:
```shell
exec sh
```

### Запустить NFQWS и проверять работоспособность с помощью команды:
```shell
/opt/zapret/init.d/sysv/zapret start
```

### Для перезагрузки NFQWS использовать команду:
```shell
/opt/zapret/init.d/sysv/zapret restart
```

### Для остановки NFQWS использовать команду:
```shell
/opt/zapret/init.d/sysv/zapret stop
```

#zapret #bol-van #keenetic #youtube #ускорение #ростелеком
