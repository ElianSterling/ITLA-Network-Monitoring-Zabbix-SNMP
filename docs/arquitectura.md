# Arquitectura de red y direccionamiento

## Diseño

El laboratorio combina un router IOSv, dos switches IOSvL2, un cliente VPCS y un enlace Cloud de GNS3 hacia el contenedor Zabbix alojado en Proxmox.

| Enlace | Interfaz origen | Interfaz destino |
|---|---|---|
| R1–SW1 | R1 Gi0/0 | SW1 Gi0/0 |
| SW1–SW2 | SW1 Gi0/1 | SW2 Gi0/0 |
| SW2–PC1 | SW2 Gi0/1 | PC1 Ethernet0 |
| SW2–Cloud1 | SW2 Gi0/2 | Cloud1 enp6s19 |

La red de monitoreo usa `10.25.13.0/24`, con R1 `10.25.13.1`, SW1 `10.25.13.2`, SW2 `10.25.13.3`. Las interfaces de administración de SW1 y SW2 están en VLAN 1, con puerta de enlace `10.25.13.1`.

## DHCP

R1 proporciona un pool denominado `LAN-ZABBIX` con red `10.25.13.0/24`, puerta de enlace `10.25.13.1`, DNS `1.1.1.1` y `8.8.8.8`, y concesión de siete días.

Exclusiones: `10.25.13.1–10.25.13.99` y `10.25.13.200–10.25.13.254`. El rango asignable queda entre `10.25.13.100` y `10.25.13.199`. En pruebas anteriores PC1 recibió `10.25.13.100` y ZABBIX-SRV utilizó `10.25.13.101`. **En la última prueba VPCS PC1 recibió `10.25.13.101/24`, gateway `10.25.13.1`, DNS `1.1.1.1` y `8.8.8.8`, con concesión de 7 días.** Dado que `.101` fue utilizado por ZABBIX-SRV anteriormente, su dirección actual debe comprobarse antes de descartar un conflicto de IP. Estas asignaciones no son reservas DHCP permanentes.

## Alojamiento

- Proxmox VE 8.4 aloja la VM 100 `GNS3-Controller`, con 8 GB RAM y 4 vCPU.
- GNS3 utiliza una interfaz conectada a la LAN (`vmbr0`) y otra conectada a una red de laboratorio aislada (`vmbr1`).
- El contenedor LXC 120 `ZABBIX-SRV` ejecuta Debian 12 con 4 GB RAM, 2 vCPU y disco de 32 GB.
- El contenedor usa `eth0` para el segmento `10.25.13.0/24` y `eth1` para administración e instalación desde la red doméstica.

## Alcance de seguridad

SNMPv2c de solo lectura está habilitado en R1/SW1/SW2, con una ACL de administración de laboratorio. La versión final publicada omite comunidades y contraseñas. No exponer el servicio Zabbix HTTP ni SNMPv2c directamente a Internet.

## Validación

Las pruebas de conectividad (ICMP), DHCP y SNMP confirmaron el funcionamiento. El apagado temporal del enlace Gi0/0 de SW2 aisló R1 y SW1 del contenedor de monitoreo, permitiendo demostrar alertas de indisponibilidad y recuperación.

> Evidencias GNS3/Zabbix y configuración saneada de R1 incorporadas; pendientes las configuraciones de SW1 y SW2 y la verificación de la IP actual del servidor.
