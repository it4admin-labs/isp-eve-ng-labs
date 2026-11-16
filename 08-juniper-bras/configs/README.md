# Конфигурации оборудования

Конфигурации для выпуска «BRAS Juniper и FreeRADIUS — Выпуск 8».

## Файлы конфигурации

| Файл | Оборудование | Роль |
|---|---|---|
| [`BRAS2.cfg`](./BRAS2.cfg) | Juniper / Junos 18.4R3-S5.4 | BRAS / BNG |
| [`NAT-1.cfg`](./NAT-1.cfg) | Juniper | L2/L3 транспорт к BRAS |
| [`freeradius/`](./freeradius/) | FreeRADIUS | AAA / профили абонентов / CoA |

## BRAS-2

Конфигурация `BRAS2.cfg` содержит основные настройки Juniper BRAS для IPoE-абонентов.

Ключевые элементы:

- `subscriber-management enable`;
- DHCP Local Server с группой `DHCP-SUBSCRIBERS`;
- формирование RADIUS username из DHCP Option 82: Circuit-ID + Remote-ID;
- динамические demux-интерфейсы для QinQ;
- отдельный VRF `RADIUS-IPoE` для связи с RADIUS-сервером;
- RADIUS authentication и accounting на сервере `172.31.0.100`;
- source address BRAS для RADIUS — `198.18.0.4`;
- accounting update interval 20 секунд;
- локальный публичный пул `IPoE_198_51_100_pool`;
- динамический профиль `RATELIMIT` для ограничения скорости;
- динамический профиль `REDIRECT_NOPAY` для финансовой блокировки;
- Captive Portal Content Delivery для HTTP Redirect;
- CoA Dynamic Variable Validation для изменения активных сервисов.

### Скоростной профиль

Динамический профиль `RATELIMIT` получает скорость через переменную `var-bw` и создаёт входной и выходной firewall filter с policer.

Burst рассчитывается динамически:

```text
max(round($var-bw/1600),20000)
```

Таким образом один профиль используется для разных тарифных скоростей, передаваемых с RADIUS.

### Финансовая блокировка

Профиль `REDIRECT_NOPAY`:

- разрешает DNS;
- разрешает доступ к порталу `198.19.0.100`;
- HTTP-трафик направляет на redirect;
- остальной пользовательский трафик блокируется фильтром.

## NAT-1

В `NAT-1.cfg` для выпуска 8 изменена L2-связность в сторону Juniper BRAS-2.

L2-сервисы к BRAS-2 заведены на агрегированный интерфейс `ae5`:

- `ae5.3201` → VC 3201;
- `ae5.3202` → VC 3202;
- `ae5.3203` → VC 3203;
- `ae5.3204` → VC 3204.

Интерфейс `ae5` построен на `xe-0/0/4` и `xe-0/0/5` и подписан как `to BRAS-2 | L2 Subscribers`.

То есть для сценария выпуска L2C абонентов переведены с направления Cisco BRAS на отдельный LAG в сторону BRAS-2 Juniper.

## FreeRADIUS

Каталог [`freeradius/`](./freeradius/) содержит:

- `clients.conf` — описание BRAS Cisco и Juniper как RADIUS clients;
- `users` — профили абонентов для Juniper BRAS;
- `CoA.example` — готовые примеры CoA-запросов для Juniper.

Подробное описание находится в [README FreeRADIUS](./freeradius/README.md).

## Учетные данные лаборатории

В опубликованных конфигурациях используются тестовые учётные данные и shared secrets, предназначенные только для виртуальной лаборатории.

> Не используйте их в производственной сети. Перед применением конфигураций замените пароли, RADIUS shared secrets, IP-адреса и другие параметры доступа.
