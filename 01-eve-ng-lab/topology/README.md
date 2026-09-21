# Топология EVE-NG

Файл [`ISP_NET.unl`](./ISP_NET.unl) содержит топологию виртуальной лаборатории «Сеть оператора связи» для EVE-NG.

## Скачать топологию

[Скачать `ISP_NET.unl`](./ISP_NET.unl)

## Требуемые образы

Для загрузки топологии в EVE-NG должны быть установлены соответствующие образы оборудования и Linux/Windows-узлов.

### Cisco

- `csr1000vng-17-03-08a`
- `xrv9k-fullk9-x-7.11.1`
- `i86bi_Linux-L2-Adventerprisek9-ms.SSA.high_iron_20190423.bin`

### Huawei

- `huaweine40e-ne40e`
- `huaweice6800-ce6800`

### Juniper

- `vmxvcp-18.4R3-S5.4`
- `vmxvcp-re0-18.4R3-S5.4`
- `vmxvcp-re1-18.4R3-S5.4`
- `vmxvfp-fpc0-18.4R3-S5.4`
- `vmxvfp-fpc1-18.4R3-S5.4`
- `vmxvcp-re0-18.2R1.9`
- `vmxmpc-18.2R1.9`
- `vmx-14.1R4.8-domestic`
- `juniper-vrr-23.2R1.13`

### Nokia

- `timos-ng-21.10.R1`

### Linux / Windows

- `linux-debian-10.3.0`
- `win-tiny10`

## Кастомные шаблоны

Для части узлов используются кастомные шаблоны EVE-NG.

[Открыть каталог кастомных шаблонов →](../custom-templates/)

## Juniper vMX

Для некоторых конфигураций Juniper vMX используются отдельные компоненты vCP/vFP и метафайлы.

[Открыть материалы Juniper vMX →](../software/juniper/)

> Образы сетевого оборудования в репозитории не публикуются. Названия выше должны соответствовать установленным в EVE-NG образам.
