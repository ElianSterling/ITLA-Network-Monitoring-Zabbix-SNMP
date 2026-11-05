# Evidencia 07 — DHCP y conectividad de PC1

**Fecha de verificación:** 5 de noviembre de 2026  
**Entorno:** GNS3 — VPCS `PC1` y router Cisco IOSv `R1`  
**Red:** `10.25.13.0/24`

## 1. Dirección IP obtenida por DHCP

Salida de consola de PC1:

```text
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.25.13.101/24
GATEWAY     : 10.25.13.1
DNS         : 1.1.1.1  8.8.8.8
DHCP SERVER : 10.25.13.1
DHCP LEASE  : 603931, 604800/302400/529200
MAC         : 00:50:79:66:68:00
MTU         : 1500
```

Los campos locales de la consola VPCS (`LPORT` y `RHOST:PORT`) se omiten porque no son necesarios para la evidencia de configuración IP.

**Validación:** PC1 ha recibido automáticamente una dirección dentro del rango DHCP, un gateway, servidores DNS y una concesión de siete días.

## 2. Conectividad ICMP con el gateway R1

```text
PC1> ping 10.25.13.1

84 bytes from 10.25.13.1 icmp_seq=1 ttl=255 time=7.380 ms
84 bytes from 10.25.13.1 icmp_seq=2 ttl=255 time=6.719 ms
84 bytes from 10.25.13.1 icmp_seq=3 ttl=255 time=11.387 ms
84 bytes from 10.25.13.1 icmp_seq=4 ttl=255 time=8.792 ms
84 bytes from 10.25.13.1 icmp_seq=5 ttl=255 time=11.692 ms
```

**Validación:** se recibieron las cinco respuestas ICMP de `10.25.13.1` (**5/5 exitosas**). Esto confirma conectividad IP entre PC1 y la interfaz LAN de R1 en el momento de la prueba.

## 3. Concesión registrada en el router R1

```text
R1# show ip dhcp binding

Bindings from all pools not associated with VRF:
IP address          Client-ID/             Lease expiration          Type
10.25.13.101        0100.5079.6668.00     Nov 12 2026 03:56 PM      Automatic
```

**Validación:** R1 registró la concesión de `10.25.13.101` al cliente VPCS PC1, cuyo identificador está asociado a `00:50:79:66:68:00`.

## Resultado

| Prueba | Resultado |
|---|---|
| Asignación de IPv4 por DHCP | ✅ Correcta |
| Máscara de red | ✅ `/24` |
| Gateway y DNS | ✅ Entregados |
| Ping PC1 → R1 | ✅ 5/5 respuestas |
| Concesión en `show ip dhcp binding` | ✅ Confirmada |

> **Nota:** La dirección de ZABBIX-SRV `10.25.13.100/24` se comprobó por separado. Aun sin colisión actual con PC1, falta asegurar una reserva DHCP o una dirección fija apropiada para que el servidor no cambie accidentalmente de IP.

> **Evidencia de consola:** estas salidas fueron proporcionadas como texto. La captura PNG de la consola GNS3 sigue pendiente y no se representa aquí como si ya se hubiera tomado.
