# Configuraciones Cisco

Exportaciones de los dispositivos del laboratorio GNS3. Cada archivo refleja la configuración observada, con comunidades SNMP y otros secretos sustituidos por marcadores.

| Dispositivo | Archivo | Estado |
|---|---|---|
| R1 · Router IOSv 15.9 | [R1-sanitized.txt](R1-sanitized.txt) | ✅ Exportación revisada |
| SW1 · Switch IOSvL2 | [SW1-sanitized.txt](SW1-sanitized.txt) | ✅ Exportación saneada publicada |
| SW2 · Switch IOSvL2 | [SW2-sanitized.txt](SW2-sanitized.txt) | ✅ Exportación saneada publicada |

## Verificaciones de R1 (5 de noviembre de 2026)

- `GigabitEthernet0/0`: `10.25.13.1`, estado `up/up`.
- `GigabitEthernet0/1–0/3`: apagadas administrativamente.
- Pool DHCP `LAN-ZABBIX`: red `10.25.13.0/24`; exclusiones `.1–.99` y `.200–.254`.
- **Al capturar inicialmente R1, la tabla DHCP tenía 0 concesiones.** Después PC1 ejecutó `ip dhcp` con éxito y obtuvo `10.25.13.101/24`, gateway `.1`, servidores DNS `1.1.1.1` y `8.8.8.8`, y lease de 7 días. R1 confirmó luego `show ip dhcp binding`: `.101` pertenece a la MAC/ID `0100.5079.6668.00` de PC1 y vence el 12 de noviembre de 2026.
- **Solapamiento descartado actualmente:** ZABBIX-SRV tiene `eth0 = 10.25.13.100/24` mientras PC1 utiliza `.101`. Sin embargo, R1 no muestra una concesión `.100`, por lo que sigue pendiente reservar esa IP al servidor o usar una dirección estática fuera del rango DHCP.
- SNMP: comunidad de solo lectura asociada a `ZABBIX-SNMP-ACL`.
- ACL SNMP observada: permitía toda `10.25.13.0/24` (no únicamente el servidor). Endurecimiento recomendado para futuros entornos.
- La configuración tiene `no ip http server` y `no ip http secure-server`.

## Seguridad y procedencia

- Las configuraciones están **sanitizadas**. Marcadores como `[REDACTED_SNMP_COMMUNITY]` no son comandos listos para ejecutar.
- No subir comunidades SNMP reales, contraseñas, hashes, tokens o claves.
- Se excluyeron banners de licencia y texto legal de IOSv, que no forman parte de los parámetros funcionales del laboratorio.
- No se incluyen imágenes ni software propietario de Cisco.
- SNMPv2c es adecuado exclusivamente para este laboratorio aislado; para un entorno real usar SNMPv3, segmentación y permisos mínimos.

## Verificaciones de los switches

- **SW1:** VLAN 1 `10.25.13.2` activa; Gi0/0 y Gi0/1 `up/up`; 90,762 paquetes SNMP recibidos y enviados; ningún error de versión/comunidad en la captura.
- **SW2:** VLAN 1 `10.25.13.3` activa; Gi0/0, Gi0/1 y Gi0/2 `up/up`; 91,139 paquetes SNMP recibidos y enviados; ningún error de versión/comunidad.
- En ambos se omitieron comunidades SNMP y banners propietarios, sin modificar los equipos activos.

Consulta las [validaciones Cisco](../docs/validaciones-cisco.md) para resultados detallados.
