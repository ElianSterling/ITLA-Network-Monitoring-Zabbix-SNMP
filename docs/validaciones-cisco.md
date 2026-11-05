# Validaciones Cisco IOSv / IOSvL2

Resultados observados en las consolas del laboratorio el **5 de noviembre de 2026**. Los contadores corresponden al momento de la captura y pueden variar con el tiempo o reinicios.

## R1 — Router principal

| Comprobación | Resultado |
|---|---|
| IP Gi0/0 | `10.25.13.1`, **up/up** |
| Pool DHCP | `LAN-ZABBIX`, segmento `10.25.13.0/24` |
| Concesiones permitidas | `10.25.13.100–10.25.13.199` |
| Concesión de PC1 | `10.25.13.101`, identificador `0100.5079.6668.00` |
| Vencimiento de la concesión de PC1 | 12 de noviembre de 2026 |
| ACL SNMP | `ZABBIX-SNMP-ACL`: permite `10.25.13.0/24` |

**Prueba ICMP desde PC1:** `ping 10.25.13.1` recibió **5/5 respuestas**, entre **6.719 y 11.692 ms**. La concesión en VPCS mostraba `10.25.13.101/24`, DNS `1.1.1.1` y `8.8.8.8`, y lease de 7 días. [Evidencia textual completa](../evidencias/dhcp/07-dhcp-pc1-y-conectividad.md).

El comando `show ip dhcp binding` registró la concesión de PC1. El servidor ZABBIX-SRV usa actualmente `10.25.13.100/24` en `eth0`, verificado con `ip -4 -br addr show dev eth0`; esta dirección todavía no está reservada formalmente en R1.

## SW1 — Switch principal

| Comprobación | Resultado |
|---|---|
| Gi0/0 (a R1) | **up/up** |
| Gi0/1 (a SW2) | **up/up** |
| VLAN 1 | `10.25.13.2/24`, **up/up** |
| Gateway | `10.25.13.1` |
| VLAN 1 | Activa con puertos Gi0/0–Gi1/3 |
| SNMP paquetes entrantes / salientes | 90,762 / 90,762 |
| SNMP GET / GET-NEXT / SET | 26,668 / 1 / 0 |
| Errores de versión, comunidad, codificación o cola | 0 |

## SW2 — Switch de acceso

| Comprobación | Resultado |
|---|---|
| Gi0/0 (a SW1) | **up/up** |
| Gi0/1 (a PC1) | **up/up** |
| Gi0/2 (a Cloud1/ZABBIX-SRV) | **up/up** |
| VLAN 1 | `10.25.13.3/24`, **up/up** |
| Gateway | `10.25.13.1` |
| SNMP paquetes entrantes / salientes | 91,139 / 91,139 |
| SNMP GET / GET-NEXT / SET | 26,783 / 1 / 0 |
| Errores de versión, comunidad, codificación o cola | 0 |

En ambos switches la configuración SNMP usa una comunidad de **solo lectura** ligada a la ACL `ZABBIX-SNMP-ACL`, que permite consultas desde `10.25.13.0/24`. Las comunidades reales no se publican.

## Mejoras de seguridad pendientes

1. Reservar `10.25.13.100` para el contenedor de monitoreo o asignarle una dirección estática fuera del pool DHCP, tras evaluar los efectos sobre el servicio.
2. Restringir la ACL SNMP al servidor en lugar de toda la subred, comprobando previamente su conectividad.
3. Para despliegues fuera del laboratorio: preferir **SNMPv3**, segmentación de administración y transporte cifrado para la interfaz web.

## Evidencia de fallos y recuperación

Se realizó un apagado temporal de `Gi0/0` de SW2. Por la topología de monitoreo, la caída aisló a R1 y SW1 de ZABBIX-SRV, mientras SW2 continuó accesible desde Cloud1. Zabbix mostró alertas de ausencia de ping y recuperó la disponibilidad a 3/3 después de ejecutar `no shutdown`. Las capturas están en [evidencias/alertas](../evidencias/alertas).
