# Configuraciones Cisco

Exportaciones de los dispositivos del laboratorio GNS3. Cada archivo refleja la configuración observada, con comunidades SNMP y otros secretos sustituidos por marcadores.

| Dispositivo | Archivo | Estado |
|---|---|---|
| R1 · Router IOSv 15.9 | [R1-sanitized.txt](R1-sanitized.txt) | ✅ Exportación revisada |
| SW1 · Switch IOSvL2 | `SW1-sanitized.txt` | ⏳ Pendiente de recopilar |
| SW2 · Switch IOSvL2 | `SW2-sanitized.txt` | ⏳ Pendiente de recopilar |

## Verificaciones de R1 (5 de noviembre de 2026)

- `GigabitEthernet0/0`: `10.25.13.1`, estado `up/up`.
- `GigabitEthernet0/1–0/3`: apagadas administrativamente.
- Pool DHCP `LAN-ZABBIX`: red `10.25.13.0/24`; exclusiones `.1–.99` y `.200–.254`.
- **Concesiones DHCP activas en el momento de la captura: 0.** Se requiere una prueba nueva desde PC1; el pool existente no garantiza que haya un cliente con concesión vigente.
- SNMP: comunidad de solo lectura asociada a `ZABBIX-SNMP-ACL`.
- ACL SNMP observada: permitía toda `10.25.13.0/24` (no únicamente el servidor). Endurecimiento recomendado para futuros entornos.
- La configuración tiene `no ip http server` y `no ip http secure-server`.

## Seguridad y procedencia

- Las configuraciones están **sanitizadas**. Marcadores como `[REDACTED_SNMP_COMMUNITY]` no son comandos listos para ejecutar.
- No subir comunidades SNMP reales, contraseñas, hashes, tokens o claves.
- Se excluyeron banners de licencia y texto legal de IOSv, que no forman parte de los parámetros funcionales del laboratorio.
- No se incluyen imágenes ni software propietario de Cisco.
- SNMPv2c es adecuado exclusivamente para este laboratorio aislado; para un entorno real usar SNMPv3, segmentación y permisos mínimos.
