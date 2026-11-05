# Zabbix 7.0 LTS: instalación, monitoreo y pruebas

## Servicios utilizados

- Debian 12 (contenedor LXC 120)
- Zabbix Server 7.0 LTS y Zabbix Agent
- MariaDB 10.11
- Nginx y PHP 8.2-FPM
- SNMPv2c (UDP/161) y comprobaciones ICMP

El servidor se comprobó escuchando en TCP/10051, y el frontend HTTP del laboratorio en el puerto 8080. La base de datos `zabbix` se inicializó con el esquema oficial, y se verificaron sus tablas antes de iniciar el servicio.

## Equipos monitorizados

| Equipo | IP SNMP | Plantilla |
|---|---|---|
| R1 — Router Principal | `10.25.13.1` | `Cisco IOS by SNMP` |
| SW1 — Switch Principal | `10.25.13.2` | `Cisco IOS by SNMP` |
| SW2 — Switch de Acceso | `10.25.13.3` | `Cisco IOS by SNMP` |

Las interfaces usan SNMPv2c y una macro local de comunidad secreta en Zabbix. **El valor real no se publica.**

## Dashboard

**ITLA | Network Infrastructure & SNMP Monitoring**

1. Disponibilidad SNMP de los tres dispositivos.
2. Tráfico entrante y saliente de la interfaz Gi0/0 de R1, SW1 y SW2.
3. CPU de R1 obtenida desde la plantilla.
4. Estado operativo Gi0/0 de SW1 y SW2.
5. Widget de problemas filtrado al grupo `Network devices`.

Los switches IOSvL2 no devolvieron objetos dentro de la rama de CPU `1.3.6.1.4.1.9.9.109.1.1.1`, por lo que no se presentan porcentajes de CPU para ellos.

## Prueba controlada de fallo y recuperación

Se aplicó `shutdown` temporalmente a la interfaz `Gi0/0` de SW2 en GNS3 (sin guardar la configuración). El servidor Zabbix perdió ICMP/SNMP hacia R1 y SW1 mientras SW2 permaneció accesible por Cloud1.

Zabbix registró eventos `Cisco IOS: Unavailable by ICMP ping` para R1 y SW1, así como un evento de interfaz de SW2. Tras ejecutar `no shutdown`, la disponibilidad volvió a 3/3 y se observaron eventos resueltos.

**Precaución:** no realizar este ejercicio en una red productiva. Un mismo enlace puede ser punto único de fallo para varias rutas de monitoreo.

## Evidencias pendientes

- Captura del dashboard normal y de cada gráfico.
- Captura de los problemas disparados durante el fallo.
- Captura de las recuperaciones y del estado 3/3.
- Comandos de verificación de SNMP y los servicios (con secretos ocultos).
