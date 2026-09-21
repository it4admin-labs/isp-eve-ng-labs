# Установка и настройка Snort 3 на Debian 13

Пошаговая инструкция для лаборатории EVE-NG из выпуска 5.1. В инструкции Snort 3 и LibDAQ собираются из исходного кода и устанавливаются в `/usr/local`.

> Команды ниже рассчитаны на чистую лабораторную VM Debian 13 и выполнение от имени `root`. При работе обычным пользователем добавляйте `sudo` там, где это необходимо.

## 1. Установка зависимостей

Сначала обновите список пакетов и установите инструменты сборки и библиотеки, необходимые Snort 3 и LibDAQ.

```bash
apt update

apt install -y \
  build-essential cmake g++ flex bison git pkg-config \
  libpcap-dev libpcre2-dev libluajit-5.1-dev \
  libssl-dev zlib1g-dev liblzma-dev \
  libhwloc-dev libdumbnet-dev \
  libsqlite3-dev libfl-dev \
  libgoogle-perftools-dev \
  ethtool net-tools
```

Snort 3 использует C++17, CMake, LibDAQ, dnet, flex, hwloc, LuaJIT, OpenSSL, libpcap, PCRE2, pkg-config и zlib. Часть установленных пакетов является дополнительной и используется в лаборатории для удобства диагностики и включения дополнительных возможностей сборки. citeturn419675search1turn419675search0

## 2. Сборка и установка LibDAQ

LibDAQ — библиотека, через которую Snort получает доступ к источнику сетевого трафика. Для inline-режима в этой лаборатории используется модуль `afpacket`. citeturn179470search1turn472976search0

```bash
cd /tmp
git clone https://github.com/snort3/libdaq.git
cd libdaq

./bootstrap
./configure
make -j$(nproc)
make install

ldconfig
```

Проверьте, что DAQ-модули видны:

```bash
snort --daq-list
```

В списке должен присутствовать `afpacket`. Если DAQ установлен в нестандартный каталог, его путь необходимо указать через `--daq-dir`. citeturn472976search0turn179470search1

## 3. Сборка и установка Snort 3

Получите исходный код из официального репозитория и соберите Snort с поддержкой tcmalloc.

```bash
cd /tmp
git clone https://github.com/snort3/snort3.git
cd snort3

./configure_cmake.sh --prefix=/usr/local --enable-tcmalloc
cd build

make -j$(nproc)
make install

ldconfig
```

Официальная документация Snort 3 использует `configure_cmake.sh`, после него сборка выполняется из каталога `build`. citeturn419675search0turn179470search1

Проверьте установку:

```bash
/usr/local/bin/snort -V
snort --daq-list
```

## 4. Подготовка каталогов Snort

Создайте каталоги конфигурации и журналов:

```bash
mkdir -p /usr/local/etc/snort
mkdir -p /var/log/snort
```

Скопируйте из каталога `configs/` репозитория три файла:

```text
configs/snort.lua          → /usr/local/etc/snort/snort.lua
configs/local_rules        → /usr/local/etc/snort/local_rules
configs/my_block_page.html → /usr/local/etc/snort/my_block_page.html
```

Например, если файлы находятся в `/root/isp-eve-ng-labs/05.1-snort3/configs/`:

```bash
cp /root/isp-eve-ng-labs/05.1-snort3/configs/snort.lua /usr/local/etc/snort/
cp /root/isp-eve-ng-labs/05.1-snort3/configs/local_rules /usr/local/etc/snort/
cp /root/isp-eve-ng-labs/05.1-snort3/configs/my_block_page.html /usr/local/etc/snort/
```

Проверьте конфигурацию до запуска:

```bash
/usr/local/bin/snort -T -c /usr/local/etc/snort/snort.lua
```

Параметр `-c` задаёт Lua-конфигурацию, а проверка позволяет убедиться, что конфигурация и подключаемые правила корректны до запуска обработки трафика. citeturn179470search4

## 5. Настройка сетевых интерфейсов

В лаборатории Snort устанавливается между двумя интерфейсами Linux:

```text
eth1  ← трафик от клиента/внутренней сети
  |
 Snort 3
  |
eth2  → трафик в сторону маршрутизатора/внешней сети
```

Интерфейсы должны работать как прозрачный L2-пролёт и не использовать IP-адреса для самого Snort.

Для варианта с `/etc/network/interfaces` можно использовать:

```text
auto eth1
allow-hotplug eth1
iface eth1 inet manual
    up ip link set dev eth1 promisc on
    up ip link set dev eth1 mtu 9000
    up ethtool -K eth1 gro off lro off

auto eth2
allow-hotplug eth2
iface eth2 inet manual
    up ip link set dev eth2 promisc on
    up ip link set dev eth2 mtu 9000
    up ethtool -K eth2 gro off lro off
```

После изменения конфигурации проверьте состояние интерфейсов:

```bash
ip link show eth1
ip link show eth2
ethtool -k eth1
ethtool -k eth2
```

Если сеть в конкретном образе Debian управляется не через ifupdown, те же параметры можно применить командами `ip link` и `ethtool` вручную.

## 6. Проверка inline-режима вручную

Snort 3 переходит в inline-режим с параметром `-Q`. Для DAQ `afpacket` необходимо передать пару интерфейсов через двоеточие, например `eth1:eth2`. citeturn472976search0

Перед созданием systemd-службы полезно сначала запустить Snort вручную:

```bash
/usr/local/bin/snort \
  -Q \
  -c /usr/local/etc/snort/snort.lua \
  -l /var/log/snort/ \
  --daq afpacket \
  -i eth1:eth2
```

В другом терминале можно проверить сообщения и трафик:

```bash
tail -f /var/log/snort/alert_fast.txt
```

При успешном запуске Snort должен удерживать inline-пару `eth1:eth2` и обрабатывать проходящий через неё трафик. citeturn472976search0

Остановить ручной запуск:

```text
Ctrl+C
```

## 7. Создание systemd-службы

Создайте файл:

```bash
nano /etc/systemd/system/snort.service
```

Содержимое:

```ini
[Unit]
Description=Snort 3 Inline IPS
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/snort -Q -c /usr/local/etc/snort/snort.lua -l /var/log/snort/ --daq afpacket -i eth1:eth2
ExecStop=/bin/kill -TERM $MAINPID
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Загрузите новую конфигурацию systemd:

```bash
systemctl daemon-reload
systemctl enable snort.service
systemctl start snort.service
```

Проверьте:

```bash
systemctl status snort.service
journalctl -u snort.service -f
```

## 8. Что делает текущая конфигурация

В поставляемом `snort.lua` уже настроены основные элементы выпуска:

```text
HOME_NET
    ↓
HTTP inspection
    ↓
local_rules
    ├── reject ICMP
    └── react HTTP
             ↓
      my_block_page.html
```

Встроенные IPS-правила отключены:

```lua
ips =
{
    enable_builtin_rules = false,
    rules = local_rules,
    variables = default_variables
}
```

Поэтому в этой лаборатории демонстрация построена вокруг собственных правил из файла `local_rules`.

## 9. Проверка правил

### Проверка ICMP

С клиента, находящегося за Snort, выполните ping:

```bash
ping <IP-адрес>
```

Правило:

```text
reject icmp any any -> any any
```

должно срабатывать на ICMP и завершать обработку соответствующим активным ответом. `reject` в Snort 3 предназначен для завершения сессии с TCP reset или ICMP unreachable. citeturn398382search0

### Проверка HTTP

Для демонстрации HTTP-контроля нужен незашифрованный HTTP-запрос с соответствующим значением Host.

Текущее правило ищет имя:

```text
httpforever.com
```

После срабатывания `react` Snort отправляет клиенту указанную страницу-заглушку и завершает HTTP-сессию. `react` относится к активным действиям Snort и предназначен именно для отправки ответа клиенту с завершением сессии. citeturn398382search0

HTTP-правила Snort 3 используют специализированные HTTP-буферы, в том числе `http_uri` и `http_header`, что позволяет проверять отдельные элементы HTTP-запроса. citeturn756083search2turn756083search1

> Важно: это правило не является механизмом блокировки HTTPS-сайтов по имени. При HTTPS HTTP-заголовки находятся внутри зашифрованного TLS-сеанса и не доступны такому правилу без отдельной схемы TLS inspection.

## 10. Полезные команды диагностики

Проверка версии:

```bash
snort -V
```

Проверка DAQ:

```bash
snort --daq-list
```

Проверка конфигурации:

```bash
snort -T -c /usr/local/etc/snort/snort.lua
```

Проверка состояния интерфейсов:

```bash
ip -br link
ethtool eth1
ethtool eth2
```

Проверка журнала systemd:

```bash
journalctl -u snort.service -n 100 --no-pager
```

Просмотр алертов:

```bash
tail -f /var/log/snort/alert_fast.txt
```

## 11. Типовые проблемы

### `afpacket` не отображается

Проверьте:

```bash
snort --daq-list
ldconfig -p | grep -i daq
```

Если LibDAQ установлена не в стандартный путь, используйте `--daq-dir` с каталогом DAQ-модулей. citeturn472976search0

### Snort не запускается с `eth1:eth2`

Проверьте, что оба интерфейса существуют и подняты:

```bash
ip link show eth1
ip link show eth2
```

Проверьте, что их имена совпадают с параметром `-i` в systemd-службе.

### Трафик не проходит через Snort

Сначала остановите systemd-службу и запустите Snort вручную. Затем проверьте интерфейсы и наличие трафика:

```bash
tcpdump -ni eth1
tcpdump -ni eth2
```

Если трафик приходит на один интерфейс, но не появляется на втором, сначала проверяйте inline-конфигурацию, состояние интерфейсов и offload-функции.

### Конфигурация не проходит `-T`

Проверьте пути:

```bash
ls -l /usr/local/etc/snort/
```

Особенно важно наличие:

```text
snort.lua
local_rules
my_block_page.html
```

Запускайте проверку непосредственно перед переводом Snort в inline-режим:

```bash
snort -T -c /usr/local/etc/snort/snort.lua
```

## 12. Итоговая схема файлов

После завершения настройки на Debian VM должны присутствовать:

```text
/usr/local/bin/snort
/usr/local/etc/snort/
├── snort.lua
├── local_rules
└── my_block_page.html

/var/log/snort/
└── alert_fast.txt
```

После этого Snort можно запускать как inline IPS между `eth1` и `eth2`, а конфигурацию и правила из этого каталога использовать как основу для дальнейшего расширения лаборатории.
