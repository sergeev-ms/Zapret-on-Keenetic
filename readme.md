## [README принципа работы от разработчика>>](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/docs/readme.txt)

# Подробная обновляемая инструкция настройки репозитория [Zapret](https://github.com/bol-van/zapret) от [bol-van](https://github.com/bol-van) на Keenetic.

[Дискуссия по инструкции](https://github.com/nikrays/Zapret-on-Keenetic/discussions/3)

## Краткое описание.

Автономное, без задействования сторонних серверов, средство противодействия DPI.
Может помочь обойти замедления сайтов http(s), сигнатурный анализ tcp и udp протоколов,
например с целью замедления YouTube.

Существуют режимы обхода TPWS и NFQWS. Режим NFQWS имеет ряд преимуществ пред режимом TPWS. Больше параметров модификации TCP соединения на уровне пакетов. Реализуется через обработчик очереди NFQUEUE и raw сокеты, а также возможность модификации трафика по протоколу QUIC.

### P.S. Это не VPN и не прокси, это средство для обмана (подмены) пакетов tcp и udp протокола, не режет скорость, повышается анонимность, а также не ломает то, что и так хорошо работало.

### По необходимости улучшаются: [config](https://github.com/nikrays/Zapret-on-Keenetic?tab=readme-ov-file#14-%D0%BE%D0%B1%D0%BD%D0%BE%D0%B2%D0%BB%D1%8F%D0%B5%D0%BC%D1%8B%D0%B9-%D0%BF%D1%83%D0%BD%D0%BA%D1%82-%D0%B7%D0%B0%D0%B3%D1%80%D1%83%D0%B6%D0%B0%D0%B5%D0%BC-%D0%B3%D0%BE%D1%82%D0%BE%D0%B2%D1%8B%D0%B9-%D0%BA%D0%BE%D0%BD%D1%84%D0%B8%D0%B3-zapret-%D0%BF%D0%BE%D0%B4%D1%85%D0%BE%D0%B4%D0%B8%D1%82-%D0%B4%D0%BB%D1%8F-%D0%B1%D0%BE%D0%BB%D1%8C%D1%88%D0%B8%D0%BD%D1%81%D1%82%D0%B0-%D0%BF%D1%80%D0%BE%D0%B2%D0%B0%D0%B9%D0%B4%D0%B5%D1%80%D0%BE%D0%B2-%D1%81-pppoe-%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BB%D1%81%D1%8F-%D0%BD%D0%B0-%D1%80%D0%BE%D1%81%D1%82%D0%B5%D0%BB%D0%B5%D0%BA%D0%BE%D0%BC-%D0%B4%D0%BE%D0%BC%D1%80%D1%83), [hostlists](https://github.com/nikrays/Zapret-on-Keenetic#15-%D0%BE%D0%B1%D0%BD%D0%BE%D0%B2%D0%BB%D1%8F%D0%B5%D0%BC%D1%8B%D0%B9-%D0%BF%D1%83%D0%BD%D0%BA%D1%82-%D0%B4%D0%B0%D0%BB%D0%B5%D0%B5-%D0%B2%D1%8B%D0%B1%D0%B8%D1%80%D0%B0%D0%B5%D0%BC-%D1%87%D1%82%D0%BE-%D0%B1%D1%83%D0%B4%D0%B5%D0%BC-%D1%83%D1%81%D0%BA%D0%BE%D1%80%D1%8F%D1%82%D1%8C), [zapret](https://github.com/nikrays/Zapret-on-Keenetic?tab=readme-ov-file#%D0%BE%D0%B1%D0%BD%D0%BE%D0%B2%D0%B8%D1%82%D1%8C-%D1%80%D0%B5%D0%BF%D0%BE%D0%B7%D0%B8%D1%82%D0%BE%D1%80%D0%B8%D0%B9-zapret-%D0%BC%D0%BE%D0%B6%D0%BD%D0%BE-%D0%B2%D1%8B%D0%BF%D0%BE%D0%BB%D0%BD%D0%B8%D0%B2-%D1%8D%D1%82%D1%83-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%83-%D1%80%D0%B0%D0%B7%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%87%D0%B8%D0%BA-%D0%B2-%D0%BF%D0%BE%D1%81%D0%BB%D0%B5%D0%B4%D0%BD%D0%B5%D0%B5-%D0%B2%D1%80%D0%B5%D0%BC%D1%8F-%D0%B0%D0%BA%D1%82%D0%B8%D0%B2%D0%B8%D0%B7%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BB%D1%81%D1%8F-%D0%B8-%D1%80%D0%B5%D0%B3%D1%83%D0%BB%D1%8F%D1%80%D0%BD%D0%BE-%D0%BF%D1%80%D0%B0%D0%B2%D0%B8%D1%82-%D0%BA%D0%BE%D0%B4). После обновления хотя бы одного из них, обязательно [перезагружаем entware(openwrt)](https://github.com/nikrays/Zapret-on-Keenetic?tab=readme-ov-file#16-%D0%BF%D0%B5%D1%80%D0%B5%D0%B7%D0%B0%D0%B3%D1%80%D1%83%D0%B6%D0%B0%D0%B5%D0%BC-entwareopenwrt-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D0%BE%D0%B9-%D0%BD%D0%B8%D0%B6%D0%B5), а также в конце инструкции вас ждут полезные команды.

## Технические требования.

(рекомендуемые) Любой Keenetic равным или более 128мб ОЗУ и наличием USB порта. Но пока есть проблемы, а именно, [утечка памяти в модуле ядра Netfilter](https://github.com/bol-van/zapret/issues/311) и [случайные перезагрузки роутера](https://github.com/bol-van/zapret/issues/306), разработчики
Keenetic в курсе о [проблеме](https://forum.keenetic.com/topic/18656-%D1%83%D1%82%D0%B5%D1%87%D0%BA%D0%B0-%D0%BF%D0%B0%D0%BC%D1%8F%D1%82%D0%B8-kn-1811-417/?do=findComment&comment=187392) и решат ее в следующем редизе 4.2 beta 3.
Есть временное решение, а именно, отключение [сетевого ускорителя](https://help.keenetic.com/hc/ru/articles/214470905-%D0%A1%D0%B5%D1%82%D0%B5%D0%B2%D0%BE%D0%B9-%D1%83%D1%81%D0%BA%D0%BE%D1%80%D0%B8%D1%82%D0%B5%D0%BB%D1%8C) в компонентах.

## Подготовка.

(обязательно) Необходимо настроить DоT вне зависимости от провайдера, согласно инструкции [Прокси-серверы DNS-over-TLS и DNS-over-HTTPS для шифрования DNS-запросов](https://help.keenetic.com/hc/ru/articles/360007687159).
<details>
    <summary>Обязательно привести настройки DNS к виду как на скриншоте</summary>
    
![DoT](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/images/DoT.png)
    
</details>

(обязательно) [Установка системы пакетов репозитория Entware на USB-накопитель](https://help.keenetic.com/hc/ru/articles/360021214160).

(обязательно) Проверить, что установленны все пакеты под категорией OPKG в наборах компонентов в настройках, также протокол IPv6 и Модули ядра подсистемы Netfilter (он появится в списке пакетов только после установки пакета "Протокол IPv6").

### Подключение по SSH к роутеру через "putty", например если адрес вашего роутера 192.168.1.1 а порт 22 или 222 если активировали сервер SSH

### Логин: 
```shell
root
```

### Пароль: 
```shell
keenetic
```

### Если увидите "(config)>" значит вошли под admin, чтобы продолжить, вводим:
```shell
exec sh
```

### Если видим приветствие BusyBox built-in shell (ash), переходим к установке, левой кнопкой мыши нажимая на квадратики копируем код, а правой вставляем в putty.

## [Эксперементальный упрощенный способ установки.](https://github.com/nikrays/Zapret-on-Keenetic/blob/master/readme_ez.md)

## Классическая установка.

### 1. Обновляем пакеты opkg.
```shell
opkg update
```

### 2. Устанавливаем пакеты ipset:

```shell
opkg install coreutils-sort curl git-http grep gzip ipset iptables kmod_ndms nano xtables-addons_legacy
```

### 3. Загружаем репозиторий Zapret:
```shell
cd /opt/tmp
git clone --depth=1 https://github.com/bol-van/zapret.git
```

### 4. Начнинаем устновку, выполняем скрипт:
```shell
cd zapret
./install_easy.sh
```

### 5. Далее (будут предупреждать и спрашивать, продолжать ли установку, отвечаем Y, затем enter и так 3 раза).

### 6. Далее (необходимо выбирать варианты для построения вашего конфига, но это бессмысленно, так как на 14 шаге вы все равно загружаете мой конфиг, который при желании конечно, можно будет отведактировать позже, поэтому везде нажимаем ENTER, пока не увидим надпись "press enter to continue", а затем снова жмем ENTER).

### 7. Удаляем ненужное из временной папки:
```shell
rm -rf /opt/tmp/*
```

### 8. Создадим правило для автозапуска Zapret при включении роутера:
```shell
ln -fs /opt/zapret/init.d/sysv/zapret /opt/etc/init.d/S90-zapret
```

### 9. Загружаем готовый стартовый скрипт с dnsmasq внутри:
```shell
cd /opt/zapret/init.d/sysv
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/zapret/init.d/sysv/zapret
```

### 10. Загружаем готовый скрипт, чтобы роутер не забывал правила:
```shell
cd /opt/etc/ndm/netfilter.d
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/etc/ndm/netfilter.d/000-zapret.sh
```

### 11. Исполняем:
```shell
chmod +x /opt/etc/ndm/netfilter.d/000-zapret.sh
```

### 12. Загружаем готовый скрипт для перевода net.netfilter.nf_conntrack_checksum в 0:
```shell
cd /opt/etc/init.d
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/opt/etc/init.d/S00fix
```

### 13. Исполняем:
```shell
chmod +x /opt/etc/init.d/S00fix
```

### 14. (обновляемый пункт) Загружаем один из готовых конфигов Zapret, подходит для большинста провайдеров с pppoe (Ростелеком, ИСС, Annex.PRO, МТС, Дом.ру), а также с dhcp и через модемы:
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

### 15. (обновляемый пункт) Далее выбираем, что будем ускорять:

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

#### Если необходимо ускорить более 100к сайтов:
```shell
cd /opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/blacklist-russia/zapret-hosts-user.txt
```

#### Загружаем список исключений с доменами, которые могут работать некорректно (по желанию, например у кого проблемы с входом в сбербанк):
```shell
cd /opt/zapret/ipset
curl -O https://raw.githubusercontent.com/nikrays/Zapret-on-Keenetic/master/hostlists/zapret-hosts-user-exclude.txt
```

### 16. Перезагружаем entware(openwrt) командой ниже:
```shell
/opt/etc/init.d/rc.unslung restart
```

## Готово, проверяем.

### Запустить тест скорости Youtube:
```shell
curl --connect-to ::speedtest.selectel.ru https://manifest.googlevideo.com/100MB -k -o/dev/null
```

### Проверить ресурс:
```shell
curl https://play.google.com -I
```

## P.S.:

#### Чтобы узнать имя внешнего сетевого интерфейса (WAN) на роутере, воспользуйтесь командой ifconfig, которая выведет все сетевые интерфейсы в системе.
```shell
ifconfig
```

#### Настроить конфиг Zapret можно тут.
```shell
nano /opt/zapret/config
```

#### Внешний интерфейс в конфиге прописан в этой строке:
```bash
IFACE_WAN=ppp0
```

Если закончили править, сохраняем CTRL+S, затем CTRL+X для выхода.

#### После всех манипуляций перезагружаем entware(openwrt).
```shell
/opt/etc/init.d/rc.unslung restart
```

### Также можно воспользоваться автоподбором. Автоподбор параметров, у каждого могут быть индивидуальными
Следует прогнать blockcheck по нескольким замедленным сайтам и выявить общий характер замедленний.
Разные сайты могут быть замедленны по-разному, нужно искать такую технику, которая работает на большинстве.
Чтобы записать вывод blockcheck.sh в файл, выполните : ./blockcheck.sh | tee /tmp/blockcheck.txt
```shell
/opt/zapret/blockcheck.sh | tee /opt/zapret/blockcheck.txt
```

<details>
    <summary>Проанализируйте какие методы дурения DPI работают, в соответствии с ними настройте /opt/zapret/config.</summary>

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
</details>

## Полезные команды.

#### Настроить конфиг Zapret.
```shell
nano /opt/zapret/config
```

#### Автоматический список доменов:
```shell
nano /opt/zapret/ipset/zapret-hosts-auto.txt
```

#### Ручной список доменов:
```shell
nano /opt/zapret/ipset/zapret-hosts-user.txt
```

#### Запустить Zapret:
```shell
/opt/zapret/init.d/sysv/zapret start
```

#### Перезагрузить Zapret:
```shell
/opt/zapret/init.d/sysv/zapret restart
```

#### Остановить Zapret:
```shell
/opt/zapret/init.d/sysv/zapret stop
```

### Сделать backup entware(openwrt) с Zapret:
```shell
cd /opt
tar cvzf /opt/backup-`date -I`.tar.gz *
```

### Для очистки содержимого накопителя, чтобы например, попробовать все заново:
```shell
rm -rf /opt/*
```

### Чтобы восстановить backup entware(openwrt) с Zapret, достаточно снова создать папку install и загрузить в нее ранее созданный backup, после чего переопределить накопитель в менеджере OPKG выбрав "не выбран" затем снова выбрать "накопитель", ждем когда закончится процесс развертывания. Далее вставляем скрипт ниже в параметр "Сценарий initrc" в менеджере OPKG, после перезагружаем роутер.
```shell
/opt/etc/init.d/rc.unslung
```

### Обновить репозиторий Zapret можно выполнив эту команду (разработчик в последнее время активизировался и регулярно правит код):
```shell
cd /opt/zapret
git reset --hard HEAD
git pull --rebase --autostash
/opt/etc/init.d/rc.unslung restart

```

#### Точечная фильтрация сайтов для определенных устройств настраивается по этой инструкции:
[Установка AdGuard Home на роутеры Keenetic](https://dartraiden.github.io/AdGuard-Home-Keenetic/)

#### Получить доступ к ИИ-сервисам, включая (Microsoft Copilot, ChatGPT, Google Gemini и другие), а также выполнять установку антивирусов и их обновлений, инсайдерских сборок и обновлений Windows без использования VPN ниже:
[Как настроить Comss.one DNS (DNS-over-HTTPS) на роутерах Keenetic](https://www.comss.ru/page.php?id=13072)

#zapret #bol-van #keenetic #youtube #ускорение #ростелеком #исс #annex.pro #мтс #дом.ру
