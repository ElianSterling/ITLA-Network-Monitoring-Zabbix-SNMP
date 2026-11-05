<div align="center">

# 🛰️ ITLA | Network Monitoring with Zabbix & SNMP

### Monitoreo de infraestructura Cisco virtualizada con GNS3, Proxmox VE y Zabbix 7.0 LTS

**Cisco IOSv · IOSvL2 · SNMPv2c · DHCP · Debian 12 · MariaDB · Nginx**

![Estado](https://img.shields.io/badge/Estado-Laboratorio%20validado-2ea44f)
![Zabbix](https://img.shields.io/badge/Zabbix-7.0%20LTS-d40000)
![Red](https://img.shields.io/badge/Red-10.25.13.0%2F24-0969da)
![Entorno](https://img.shields.io/badge/Entorno-GNS3%20%2B%20Proxmox-555555)

</div>

---

## 📖 Descripción

Laboratorio académico de **monitoreo de red** que integra equipos Cisco virtuales desplegados en **GNS3**, un servidor **Zabbix 7.0 LTS** alojado en un contenedor Debian 12 sobre **Proxmox VE**, y recopilación de métricas mediante **SNMPv2c**.

El proyecto demuestra configuración de direccionamiento IPv4, DHCP para clientes, administración de dispositivos por SNMP de solo lectura, visualización de tráfico y disponibilidad, y detección y recuperación de incidentes de conectividad.

> **Alcance:** laboratorio aislado, con direcciones privadas y configuraciones de ejemplo. No está diseñado para exposición directa a Internet.

## 🎯 Objetivos

- Implementar una LAN funcional con un router, dos switches y un cliente DHCP.
- Configurar direccionamiento IPv4 y servicio DHCP en Cisco IOSv.
- Habilitar SNMP de **solo lectura** en los tres dispositivos.
- Instalar Zabbix Server y sus servicios dependientes.
- Agregar dispositivos mediante la plantilla `Cisco IOS by SNMP`.
- Monitorear tráfico, interfaces, disponibilidad y métricas compatibles con cada imagen.
- Validar la generación y recuperación de alertas ante una interrupción controlada.

## 🧰 Tecnologías

| Componente | Tecnología | Función |
|---|---|---|
| Hipervisor | Proxmox VE 8.4 | Alojamiento del laboratorio |
| Emulador de redes | GNS3 | Topología y dispositivos virtuales |
| Router | Cisco IOSv 15.9 | Enrutamiento, DHCP y SNMP |
| Switches | Cisco IOSvL2 15.2 | Conmutación de capa 2 y SNMP |
| Servidor de monitoreo | Zabbix 7.0 LTS | Recolección, métricas y alertas |
| Sistema operativo | Debian 12 (LXC) | Servidor Zabbix |
| Base de datos | MariaDB 10.11 | Persistencia de Zabbix |
| Interfaz web | Nginx + PHP-FPM | Dashboard y administración |
| Protocolos | IPv4, DHCP, ICMP, SNMPv2c | Operación y monitoreo |

## 🗺️ Arquitectura

```mermaid
flowchart TB
  R1["R1 · Router IOSv<br/>10.25.13.1/24<br/>DHCP + SNMP RO"]
  SW1["SW1 · IOSvL2<br/>10.25.13.2/24"]
  SW2["SW2 · IOSvL2<br/>10.25.13.3/24"]
  PC1["PC1 · VPCS<br/>Cliente DHCP"]
  CLOUD["Cloud1 · GNS3<br/>enp6s19"]
  BRIDGE["Proxmox · vmbr1"]
  ZBX["ZABBIX-SRV · Debian 12<br/>eth0: 10.25.13.100/24 (observado)<br/>Zabbix 7.0 LTS"]
  R1 ---|"Gi0/0 ↔ Gi0/0"| SW1
  SW1 ---|"Gi0/1 ↔ Gi0/0"| SW2
  SW2 ---|"Gi0/1"| PC1
  SW2 ---|"Gi0/2"| CLOUD
  CLOUD --- BRIDGE
  BRIDGE --- ZBX
```

La conexión a Cloud1 enlaza el dominio de capa 2 de GNS3 con el contenedor de monitoreo mediante `vmbr1`. El contenedor dispone además de una interfaz independiente para acceso web en la LAN doméstica, empleada durante la instalación.

### Direccionamiento

| Equipo | Dirección | Asignación |
|---|---|---|
| R1 | `10.25.13.1/24` | Estática |
| SW1 (VLAN 1) | `10.25.13.2/24` | Estática |
| SW2 (VLAN 1) | `10.25.13.3/24` | Estática |
| PC1 | `10.25.13.101/24` | DHCP (última prueba; antes obtuvo `.100`) |
| ZABBIX-SRV (eth0) | `10.25.13.100/24` | Dirección activa observada; reserva DHCP aún no confirmada |
| Red del laboratorio | `10.25.13.0/24` | Privada |

**Verificación completada:** ZABBIX-SRV tiene actualmente `10.25.13.100/24` y PC1 `10.25.13.101/24`, por lo que no hay duplicación activa. R1 registra la concesión `.101` de PC1 con vencimiento el 12 de noviembre de 2026, pero no muestra una concesión `.100`. **Pendiente de endurecimiento:** reservar `.100` para ZABBIX-SRV o asignarle una IP estática fuera del rango DHCP antes de incorporar otros clientes.

**DHCP en R1:** pool `LAN-ZABBIX`; rango utilizable mediante exclusiones: `10.25.13.100–10.25.13.199`; gateway `10.25.13.1`; DNS `1.1.1.1` y `8.8.8.8`; concesión de 7 días.

## 📊 Monitoreo implementado

El tablero **ITLA | Network Infrastructure & SNMP Monitoring** incluye:

- Disponibilidad SNMP de **R1, SW1 y SW2**.
- Gráficos de tráfico recibido y enviado en `Gi0/0` de los tres equipos.
- Utilización de CPU de R1 mediante una métrica disponible.
- Estado operativo `up (1)` / `down (2)` de `Gi0/0` de SW1 y SW2.
- Widget de problemas e historial reciente de alertas.

> Las imágenes IOSvL2 no expusieron la rama `CISCO-PROCESS-MIB` consultada para CPU. Se muestran estados reales de interfaces en lugar de inventar porcentajes de CPU.

## 🧪 Validaciones realizadas

| Prueba | Resultado observado |
|---|---|
| DHCP a PC1 | VPCS completó DHCP (`DDORA`), IP `10.25.13.101/24`, gateway `.1`, DNS `1.1.1.1` y `8.8.8.8`, lease de 7 días; concesión verificada en R1 (expira el 12 de noviembre de 2026) |
| Conectividad en la LAN | PC1 → R1: 5 de 5 respuestas ICMP, tiempos de 6.719 a 11.692 ms |
| Consultas SNMPv2c | R1, SW1 y SW2 responden al OID `sysName.0` |
| Disponibilidad | 3/3 equipos disponibles en operación normal |
| Gráficos de tráfico | Métricas RX/TX visibles en Zabbix |
| Incidente simulado | `shutdown` temporal en `Gi0/0` de SW2 |
| Detección | Alertas ICMP de indisponibilidad en R1 y SW1; evento de interfaz en SW2 |
| Recuperación | `no shutdown`, 3/3 dispositivos disponibles y eventos recuperados |

**Nota técnica:** apagar el enlace SW1–SW2 interrumpió el acceso de Zabbix a R1 y SW1; SW2 siguió accesible desde Cloud1. Esta dependencia explica las alertas de conectividad observadas.

## 🛠️ Troubleshooting destacado

Durante las primeras pruebas, las solicitudes SNMP salían de `ZABBIX-SRV` pero no recibían respuesta. La inspección de capturas evidenció **checksums UDP incorrectos** en paquetes transmitidos a los dispositivos emulados. Deshabilitar el **TX checksum offloading** de la interfaz del contenedor resolvió el problema.

```bash
ethtool -K eth0 tx off
```

La corrección se configuró de forma persistente mediante un servicio `systemd` de tipo `oneshot`. Consulta [Troubleshooting](docs/troubleshooting.md) para diagnóstico y verificaciones.

## 🎬 Video demostrativo — Práctica 5

**[▶️ Ver la demostración en YouTube](https://youtu.be/V94S0YOxVjs)**

Video compartido por el autor, con recorrido del proyecto y una demostración guiada de falla y recuperación de una interfaz de SW2. Consulta [la ficha del video](video/enlace-video.md). *El contenido exacto del enlace no ha sido verificado de forma independiente.*

## 📸 Evidencias reales

### Topología implementada en GNS3

![Topología GNS3 R1 SW1 SW2 PC1 Cloud1](evidencias/topologia/01-topologia-gns3.png)

### Dashboard centralizado

![Dashboard Zabbix con disponibilidad, tráfico, CPU y estado de interfaces](evidencias/zabbix/02-dashboard-zabbix.png)

### Detección y recuperación de incidentes

| Incidencia de conectividad detectada | Recuperación de R1 y SW1 |
|---|---|
| ![Incidente ICMP de R1 y SW1](evidencias/alertas/03-incidencia-icmp.png) | ![Recuperación registrada por Zabbix](evidencias/alertas/04-recuperacion-incidentes.png) |

### SNMP: métricas y equipos registrados

| Métricas de Gi0/0 | Equipos monitorizados |
|---|---|
| ![Métricas de Gi0/0](evidencias/snmp/05-metricas-r1-gi0-0.png) | ![Equipos SNMP en Zabbix](evidencias/snmp/06-equipos-snmp.png) |

### DHCP y conectividad de PC1

[Ver evidencia 07: dirección DHCP, ping al gateway y concesión del router](evidencias/dhcp/07-dhcp-pc1-y-conectividad.md). La evidencia es una transcripción de consola; la captura PNG queda pendiente.

Consulta el [catálogo de evidencias con explicación técnica](evidencias/README.md) para más detalles.

## 📂 Organización

```text
.
├── README.md
├── docs/
│   ├── arquitectura.md
│   ├── monitoreo-zabbix.md
│   └── troubleshooting.md
├── configs/         # Configuraciones saneadas de R1, SW1 y SW2
├── evidencias/
│   ├── README.md
│   ├── topologia/    # Captura GNS3
│   ├── zabbix/      # Dashboard
│   ├── dhcp/        # Evidencia textual de DHCP, ICMP y concesión
│   ├── snmp/        # Métricas y equipos
│   └── alertas/     # Incidencia y recuperación
└── video/           # Enlace del video demostrativo de la práctica
```

Las capturas de evidencia ya están publicadas en sus carpetas; las configuraciones saneadas de R1, SW1 y SW2 ya están publicadas; también se añadió la evidencia textual de DHCP. El enlace del video ya se añadió. Las comprobaciones opcionales y mejoras de seguridad se documentan como trabajo futuro.

## 🔐 Seguridad

- SNMPv2c utiliza comunidades transmitidas en texto claro: **solo para laboratorio aislado**.
- En GitHub se publicarán configuraciones **sanitizadas**, nunca comunidades SNMP reales, credenciales, secretos ni capturas que los revelen.
- No se publicarán imágenes propietarias de Cisco ni discos de máquinas virtuales.
- Para entornos productivos se recomienda **SNMPv3**, segmentación de administración y cifrado de la interfaz web.

## 📚 Documentación

- [Arquitectura y direccionamiento](docs/arquitectura.md)
- [Instalación y monitoreo Zabbix](docs/monitoreo-zabbix.md)
- [Solución de problemas SNMP](docs/troubleshooting.md)
- [Configuraciones Cisco de R1, SW1 y SW2](configs/README.md)
- [Validaciones Cisco: DHCP, interfaces y SNMP](docs/validaciones-cisco.md)
- [Video demostrativo de la Práctica 5](video/enlace-video.md)

## 👨‍💻 Autor

**Elian Sterling Severino Mejía**  
Instituto Tecnológico de Las Américas (ITLA) · Seguridad Informática  
Matrícula: **2025-1331**

---

<div align="center"><sub>Proyecto académico de infraestructura, monitoreo de redes y respuesta a incidentes.</sub></div>
