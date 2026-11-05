# Troubleshooting: consultas SNMP sin respuesta en GNS3/Proxmox

## Síntoma

El contenedor Debian `ZABBIX-SRV` tenía conectividad ICMP hacia R1, SW1 y SW2, pero las consultas `snmpget` por UDP/161 agotaban el tiempo de espera.

## Investigación

1. Se comprobó direccionamiento IPv4, interfaces activas y las ACL SNMP de los dispositivos.
2. Las capturas en el contenedor y en los enlaces de GNS3 evidenciaron solicitudes SNMP GetRequest salientes.
3. Los equipos Cisco no registraban solicitudes en los contadores `show snmp`.
4. Una inspección detallada con `tcpdump -vvv` mostró indicaciones `bad udp cksum` en paquetes SNMP. Estas indicaciones en una captura local pueden deberse a offloading y, por sí solas, no prueban un paquete inválido en la red.
5. Se verificó que la interfaz `eth0` del contenedor tenía `tx-checksumming: on` y `tx-checksum-ip-generic: on`.
6. Tras desactivar temporalmente TX checksum offloading, las consultas SNMP comenzaron a responder correctamente a los tres nodos.

## Corrección aplicada

Ejecutado dentro del contenedor de laboratorio:

```bash
ethtool -K eth0 tx off
ethtool -k eth0 | grep -E 'tx-checksumming|tx-checksum-ip-generic'
```

## Persistencia con systemd

Archivo: `/etc/systemd/system/fix-snmp-offloading.service`

```ini
[Unit]
Description=Disable TX checksum offloading for GNS3 SNMP network
After=network.target
Before=zabbix-server.service

[Service]
Type=oneshot
ExecStart=/usr/sbin/ethtool -K eth0 tx off
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Activación:

```bash
systemctl daemon-reload
systemctl enable --now fix-snmp-offloading.service
systemctl status fix-snmp-offloading.service --no-pager
```

> Esta solución corresponde al entorno virtualizado concreto de las pruebas. No se recomienda deshabilitar offloading de forma general sin diagnosticar primero el problema.

## Verificación sin exponer la comunidad SNMP

```bash
read -rsp 'Comunidad SNMP del laboratorio: ' COMMUNITY
echo
for ip in 10.25.13.1 10.25.13.2 10.25.13.3; do
  snmpget -v2c -c "$COMMUNITY" -t 3 -r 1 "$ip" 1.3.6.1.2.1.1.5.0
done
unset COMMUNITY
```

Resultados observados: `R1`, `SW1` y `SW2`.

## Nota de seguridad

No subir capturas PCAP sin editar: SNMPv2c incluye su cadena de comunidad en texto claro. Las evidencias públicas deben estar saneadas y las comunidades utilizadas en pruebas expuestas deben rotarse.
