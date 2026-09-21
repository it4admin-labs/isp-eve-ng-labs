# Конфигурации оборудования

Конфигурации сетевого оборудования для выпуска «MPLS, LDP и MPLS L3 VPN — Выпуск 5».

## Файлы конфигурации

| Файл | Оборудование | Роль |
|---|---|---|
| [`Gogolya15p2-02-ASW1.cfg`](./Gogolya15p2-02-ASW1.cfg) | Huawei CE6800 | Access Switch |
| [`Lenina10p1-01-ASW1.cfg`](./Lenina10p1-01-ASW1.cfg) | Huawei CE6800 | Access Switch |
| [`Lermontova4p5-04-ASW1.cfg`](./Lermontova4p5-04-ASW1.cfg) | Huawei CE6800 | Access Switch |
| [`Pushkina8p1-03-ASW1.cfg`](./Pushkina8p1-03-ASW1.cfg) | Huawei CE6800 | Access Switch |
| [`01-AGG-SW1.cfg`](./01-AGG-SW1.cfg) | Cisco | Aggregation Switch |
| [`02-AGG-SW1.cfg`](./02-AGG-SW1.cfg) | Cisco | Aggregation Switch |
| [`03-AGG-SW1.cfg`](./03-AGG-SW1.cfg) | Cisco | Aggregation Switch |
| [`04-AGG-SW1.cfg`](./04-AGG-SW1.cfg) | Cisco | Aggregation Switch |
| [`AR-1.cfg`](./AR-1.cfg) | Cisco CSR1000v | AR |
| [`NOC-SW-1.cfg`](./NOC-SW-1.cfg) | Cisco | NOC Switch |
| [`PE-1.cfg`](./PE-1.cfg) | Huawei NE40E | PE |
| [`PE-2.cfg`](./PE-2.cfg) | Juniper vMX | PE |
| [`PE-3.cfg`](./PE-3.cfg) | Nokia 7750 VSR | PE |
| [`PE-4.cfg`](./PE-4.cfg) | Cisco IOS XR | PE |

## Назначение

Конфигурации предназначены для практического воспроизведения настроек MPLS, LDP, MPLS L3 VPN, VRF, VPNv4, BGP и Management VRF, показанных в пятом выпуске.

Коммутаторы доступа Huawei CE6800 подключают клиентскую сеть. Коммутаторы с именами вида `01-AGG-SW1` — Cisco-коммутаторы агрегации, через которые клиентский трафик передаётся в сеть оператора. Узлы `PE-1` — `PE-4` выполняют роль Provider Edge и участвуют в построении MPLS L3 VPN.

## Учетные данные лаборатории

В опубликованных конфигурациях используются тестовые учетные данные, предназначенные только для виртуальной лаборатории.

> Не используйте эти учетные данные в производственной сети. Перед использованием конфигураций вне лаборатории замените пароли и другие параметры доступа.
