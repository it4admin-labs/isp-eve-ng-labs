# Конфигурации оборудования

Конфигурации сетевого оборудования для выпуска «MPLS L2 VPN и QinQ — Выпуск 6».

## Файлы конфигурации

| Файл | Оборудование | Роль |
|---|---|---|
| [`CR-1.cfg`](./CR-1.cfg) | Juniper MX | CR |
| [`CR-2.cfg`](./CR-2.cfg) | Juniper MX | CR |
| [`ASBR.cfg`](./ASBR.cfg) | Juniper MX | ASBR |
| [`PE-1.cfg`](./PE-1.cfg) | Huawei NE40E | PE |
| [`PE-2.cfg`](./PE-2.cfg) | Juniper vMX | PE |
| [`PE-3.cfg`](./PE-3.cfg) | Nokia 7750 VSR | PE |
| [`PE-4.cfg`](./PE-4.cfg) | Cisco IOS XR | PE |
| [`BRAS-1.cfg`](./BRAS-1.cfg) | Cisco IOS XR | BRAS |
| [`BRAS-2.cfg`](./BRAS-2.cfg) | Juniper vMX | BRAS |
| [`AR-1.cfg`](./AR-1.cfg) | Cisco CSR1000v | AR |
| [`NAT-1.cfg`](./NAT-1.cfg) | Juniper vMX | NAT |
| [`VRR-1.cfg`](./VRR-1.cfg) | Juniper vRR | Route Reflector |
| [`VRR-2.cfg`](./VRR-2.cfg) | Juniper vRR | Route Reflector |
| [`01-AGG-SW1.cfg`](./01-AGG-SW1.cfg) | Cisco | Aggregation Switch |
| [`02-AGG-SW1.cfg`](./02-AGG-SW1.cfg) | Cisco | Aggregation Switch |
| [`03-AGG-SW1.cfg`](./03-AGG-SW1.cfg) | Cisco | Aggregation Switch |
| [`04-AGG-SW1.cfg`](./04-AGG-SW1.cfg) | Cisco | Aggregation Switch |
| [`Lenina10p1-01-ASW1.cfg`](./Lenina10p1-01-ASW1.cfg) | Huawei CE6800 | Access Switch |
| [`Gogolya15p2-02-ASW1.cfg`](./Gogolya15p2-02-ASW1.cfg) | Huawei CE6800 | Access Switch |
| [`Pushkina8p1-03-ASW1.cfg`](./Pushkina8p1-03-ASW1.cfg) | Huawei CE6800 | Access Switch |
| [`Lermontova4p5-04-ASW1.cfg`](./Lermontova4p5-04-ASW1.cfg) | Huawei CE6800 | Access Switch |
| [`NOC-SW-1.cfg`](./NOC-SW-1.cfg) | Cisco | NOC Switch |

## Назначение

Конфигурации предназначены для практического воспроизведения настроек MPLS L2 VPN, QinQ, VLAN-тегирования и связанных с ними сервисов, показанных в шестом выпуске.

Коммутаторы доступа Huawei CE6800 подключают клиентские сети. Коммутаторы с именами вида `01-AGG-SW1` — Cisco-коммутаторы агрегации, через которые клиентские VLAN передаются в сеть оператора.

Узлы `CR-1` и `CR-2` выполняют роль маршрутизаторов ядра MPLS-сети. `ASBR` используется на внешней границе сети. Узлы `PE-1` — `PE-4` выполняют роль Provider Edge и участвуют в построении L2-сервисов. `BRAS-1` и `BRAS-2` представлены Cisco IOS XR и Juniper vMX соответственно.

## Лабораторный сценарий

Основной сценарий выпуска — передача клиентского VLAN от коммутатора доступа через коммутаторы агрегации, MPLS-сеть и L2 VPN до BRAS.

QinQ используется для переноса клиентского VLAN через сеть оператора с добавлением внешнего VLAN-тега. На стороне PE выполняется необходимая обработка VLAN-тегов перед передачей трафика в L2 VPN.

## Учетные данные лаборатории

В опубликованных конфигурациях используются тестовые учетные данные, предназначенные только для виртуальной лаборатории.

> Не используйте эти учетные данные в производственной сети. Перед использованием конфигураций вне лаборатории замените пароли и другие параметры доступа.
