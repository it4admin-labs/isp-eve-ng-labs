# FreeRADIUS

Конфигурация FreeRADIUS, использованная в выпуске 7 для авторизации IPoE-абонентов на BRAS Cisco IOS XR, выдачи параметров пользовательских профилей и управления активными сессиями через CoA.

## Установка

В лаборатории используется пакетная установка FreeRADIUS:

```bash
apt update
apt install freeradius freeradius-utils
```

Для этого сценария EAP и внутренний virtual server `inner-tunnel` не используются:

```bash
rm /etc/freeradius/3.0/sites-enabled/inner-tunnel
rm /etc/freeradius/3.0/mods-enabled/eap
```

После копирования конфигурационных файлов рекомендуется сначала проверить конфигурацию:

```bash
freeradius -XC
```

Для отладки RADIUS в foreground:

```bash
systemctl stop freeradius
freeradius -X
```

После проверки:

```bash
systemctl restart freeradius
systemctl status freeradius
```

## Сетевая настройка сервера

Пример `/etc/network/interfaces`, использованный в лаборатории:

```text
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
iface eth0 inet dhcp

allow-hotplug eth1
iface eth1 inet static
        address 172.31.0.100/24
        up ip route add 198.18.0.3/32 via 172.31.0.1
```

Интерфейс `eth1` используется для связи с BRAS. Сервер имеет адрес `172.31.0.100/24`, а отдельный host route направляет трафик к RADIUS source address BRAS `198.18.0.3/32` через `172.31.0.1`.

## Файлы конфигурации

| Файл | Куда установить | Назначение |
|---|---|---|
| [`radiusd.conf`](./radiusd.conf) | `/etc/freeradius/3.0/radiusd.conf` | Основная конфигурация FreeRADIUS |
| [`clients.conf`](./clients.conf) | `/etc/freeradius/3.0/clients.conf` | BRAS как RADIUS client |
| [`users`](./users) | `/etc/freeradius/3.0/users` | Абонентские профили и RADIUS-атрибуты |
| [`sites-enabled/default`](./sites-enabled/default) | `/etc/freeradius/3.0/sites-enabled/default` | RADIUS listeners и обработка запросов |

## Основные параметры

### RADIUS client

В `clients.conf` BRAS описан как RADIUS client:

- адрес BRAS: `198.18.0.3/32`;
- shared secret: `cisco`.

Shared secret должен совпадать с ключом, настроенным на BRAS.

### Authentication и Accounting

В `sites-enabled/default` сервер слушает:

- `172.31.0.100:1812/UDP` — authentication;
- `172.31.0.100:1813/UDP` — accounting.

Для авторизации используются `files` и PAP. Accounting записывается модулем `detail`.

### Пользовательские профили

В файле `users` задаются параметры абонента. В показанном примере используются наиболее важные атрибуты:

- `Cleartext-Password` — пароль абонента;
- `Cisco-AVPair = subscriber:sa=...` — активация сервиса на BRAS;
- `ipv4-unnumbered` — выбор интерфейса/шаблона для IPoE;
- `Framed-IP-Address` и `Framed-IP-Netmask` — параметры публичного IP-адреса;
- `dhcp-class=PUBLIC` — выбор DHCP-класса на BRAS.

Идентификатор пользователя в лаборатории формируется BRAS из DHCP Circuit-ID и Remote-ID.

> В лабораторном файле присутствует `DEFAULT Auth-Type := Accept`. Это упрощение для стенда и не должно бездумно переноситься в производственную конфигурацию.

### Логирование

В `radiusd.conf` включено подробное логирование authentication, в том числе успешных и неуспешных паролей. Это удобно при демонстрации и отладке, но для production такую настройку следует пересмотреть.

## CoA — изменение активной сессии

На BRAS CoA принимается на UDP-порту `1700`.

В примерах используется:

- BRAS: `198.18.0.3:1700`;
- shared secret: `cisco`;
- идентификатор активной сессии: `Acct-Session-Id=04000001`.

### Смена тарифного профиля

Сначала снимаем текущий полисер:

```bash
echo "Acct-Session-Id=04000001,cisco-avpair +='subscriber:sd=POLICER_64K'" | radclient -x 198.18.0.3:1700 coa cisco
```

Затем активируем новый:

```bash
echo "Acct-Session-Id=04000001,cisco-avpair +='subscriber:sa=POLICER_128K'" | radclient -x 198.18.0.3:1700 coa cisco
```

Где:

- `subscriber:sd` — деактивация сервиса;
- `subscriber:sa` — активация сервиса.

### Финансовая блокировка и redirect

Применить профиль `SERVICE_NOPAY`:

```bash
echo "Acct-Session-Id=04000001,cisco-avpair +='subscriber:sa=SERVICE_NOPAY'" | radclient -x 198.18.0.3:1700 coa cisco
```

Снять профиль финансовой блокировки:

```bash
echo "Acct-Session-Id=04000001,cisco-avpair +='subscriber:sd=SERVICE_NOPAY'" | radclient -x 198.18.0.3:1700 coa cisco
```

Профиль `SERVICE_NOPAY` определён на BRAS. Он разрешает доступ только к заданным ресурсам, HTTP-трафик перенаправляет на страницу уведомления, а остальной трафик блокирует.

## Важно

Конфигурация предназначена для лаборатории EVE-NG.

Перед использованием вне стенда обязательно замените:

- RADIUS shared secrets;
- пароли пользователей;
- IP-адреса;
- пользовательские профили;
- параметры логирования.

Не используйте опубликованные тестовые значения в производственной сети.
