# FreeRADIUS — выпуск 8

Настройки FreeRADIUS, используемые совместно с BRAS Juniper в выпуске 8.

Полная инструкция по установке и базовой настройке FreeRADIUS приведена в материалах выпуска 7:

[FreeRADIUS — установка, основные параметры и CoA](../../../07-freeradius-bras/configs/freeradius/)

В выпуске 8 базовую установку сервера не повторяем — здесь находятся только файлы, относящиеся к интеграции FreeRADIUS с Juniper BRAS.

## Файлы

| Файл | Назначение |
|---|---|
| [`clients.conf`](./clients.conf) | RADIUS clients: BRAS Cisco и BRAS Juniper |
| [`users`](./users) | Абонентские профили для Juniper BRAS |
| [`CoA.example`](./CoA.example) | Примеры RADIUS CoA для управления сервисами |

## clients.conf

В конфигурации присутствуют два BRAS:

- `BRAS-1` — `198.18.0.3/32`, shared secret `cisco`;
- `BRAS-2` — `198.18.0.4/32`, shared secret `juniper`.

Для выпуска 8 используется `BRAS-2`.

## users

В файле `users` приведены примеры абонентских профилей для Juniper.

В качестве username используется строка, сформированная BRAS из DHCP Option 82:

```text
Circuit-ID:Remote-ID
```

Основные RADIUS-атрибуты:

- `Cleartext-Password` — пароль для авторизации;
- `Framed-Pool` — адресный пул на BRAS;
- `Framed-IP-Address` — постоянный публичный IP-адрес абонента;
- `ERX-Service-Activate` — активация динамического сервиса Juniper.

Пример назначения скоростного профиля:

```text
ERX-Service-Activate:1 = "RATELIMIT(10000000)"
```

Значение в скобках передаётся в динамический профиль `RATELIMIT` как скорость policer.

> В лабораторном файле присутствует `DEFAULT Auth-Type := Accept`. Это упрощение стенда и не должно бездумно переноситься в production.

## CoA

Файл [`CoA.example`](./CoA.example) содержит готовые запросы CoA к Juniper BRAS `198.18.0.4`.

В примерах используется `Acct-Session-Id="104"`.

### Применение финансовой блокировки

```bash
echo Acct-Session-Id="104", ERX-Service-Activate:1="REDIRECT_NOPAY" | radclient -d /usr/share/freeradius/ -sx 198.18.0.4 coa juniper
```

### Снятие финансовой блокировки

```bash
echo Acct-Session-Id="104", ERX-Service-Deactivate="REDIRECT_NOPAY" | radclient -d /usr/share/freeradius/ -sx 198.18.0.4 coa juniper
```

### Активация скоростного профиля

```bash
echo Acct-Session-Id="104", 'ERX-Service-Activate:1 = "RATELIMIT(10000000)"' | radclient -d /usr/share/freeradius/ -sx 198.18.0.4 coa juniper
```

### Снятие скоростного профиля

```bash
echo Acct-Session-Id="104", ERX-Service-Deactivate = 'RATELIMIT' | radclient -d /usr/share/freeradius/ -sx 198.18.0.4 coa juniper
```

### Смена тарифной скорости одним CoA

В одном запросе сначала снимается текущий сервис `RATELIMIT`, затем активируется тот же профиль с новым значением скорости:

```bash
echo 'Acct-Session-Id = "104",
ERX-Service-Deactivate = "RATELIMIT",
ERX-Service-Activate:1 = "RATELIMIT(20000000)"' \
| radclient -d /usr/share/freeradius/ -x 198.18.0.4 coa juniper
```

Так тарифный профиль изменяется на активной абонентской сессии без её полного переподключения.

## Важно

Все адреса, пароли и shared secrets относятся к лаборатории EVE-NG.

Перед использованием вне стенда замените:

- shared secrets;
- пароли пользователей;
- IP-адреса;
- значения сервисных профилей;
- параметры CoA.
