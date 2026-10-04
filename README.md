# Parcial 2 · Servicios Telemáticos

**Estudiantes:** Eddie Santiago Delgado Campo, Sebastián Leiton Goyes, Christian David Home
**Universidad Autónoma de Occidente · Facultad de Ingeniería**

Implementación de tres servicios de transferencia y resolución segura, protegidos por un firewall UFW sobre máquinas virtuales con Vagrant y VirtualBox:

1. **FTPS** (FTP sobre TLS explícito) con vsftpd, detrás de un firewall con reenvío de puertos.
2. **DNS sobre TLS (DoT)** con `systemd-resolved`.
3. **SFTP** sobre OpenSSH con usuario enjaulado (chroot), detrás del mismo firewall.

## Topología

| Máquina | Hostname | Red pública (bridge) | Red interna | Rol |
| --- | --- | --- | --- | --- |
| Cliente (host Windows) | – | su IP en la red | – | FileZilla, `sftp`, Wireshark |
| Servidor 1 | `srv1-<codigo_estudiantil>` | `192.168.1.150` | `192.168.50.3` | Firewall UFW (única puerta de entrada) |
| Servidor 2 | `srv2-<codigo_estudiantil>` | – | `192.168.50.2` | vsftpd (FTPS), OpenSSH (SFTP) y cliente DoT |

El Servidor 2 solo está en una red interna aislada: el cliente únicamente lo alcanza a través del Servidor 1.

## Archivos de configuración

| Archivo | Servidor | Ruta original | Qué configura |
| --- | --- | --- | --- |
| [`before.rules`](before.rules) | Servidor 1 | `/etc/ufw/before.rules` | Reglas NAT: DNAT del puerto 21, del rango pasivo `50000:50010` y de `2222 → 22`, más `MASQUERADE` |
| [`vsftpd.conf`](vsftpd.conf) | Servidor 2 | `/etc/vsftpd.conf` | FTPS con TLS explícito obligatorio, SSLv2/SSLv3 deshabilitados y rango pasivo |
| [`resolved.conf`](resolved.conf) | Servidor 2 | `/etc/systemd/resolved.conf` | DNS sobre TLS estricto con Cloudflare y Google |
| [`sshd_config`](sshd_config) | Servidor 2 | `/etc/ssh/sshd_config` | Usuario SFTP con `Match User`, `ChrootDirectory` y `ForceCommand internal-sftp` |
| [`Vagrantfile`](Vagrantfile) | – | – | Definición de las máquinas virtuales y las redes |

## Resumen por parte

### Parte 1 · FTPS protegido por UFW
- UFW en el Servidor 1 con política `deny` en tráfico entrante y enrutado, y reenvío IP habilitado.
- Reglas permitidas: `22/tcp` (administración), y con `ufw route` el `21/tcp` y el rango pasivo `50000:50010/tcp` hacia el Servidor 2.
- vsftpd con `force_local_logins_ssl=YES` y `force_local_data_ssl=YES`, certificado firmado por una CA propia, `pasv_min_port=50000`, `pasv_max_port=50010` y `pasv_address` con la IP pública.

### Parte 2 · DNS sobre TLS
- `DNS=` con `1.1.1.1#cloudflare-dns.com` y `8.8.8.8#dns.google`, `DNSOverTLS=yes` y `FallbackDNS=` coherente.
- Verificación con `resolvectl status` y capturas en Wireshark: `tcp.port == 853` (cifrado) frente a `udp.port == 53` (sin cifrar).

### Parte 3 · SFTP protegido por UFW
- Usuario `sftp_<codigo_estudiantil>` enjaulado en `/srv/sftp/<usuario>`, sin shell.
- Puerto externo `2222` del Servidor 1 reenviado al `22` del Servidor 2, con su regla `ufw route`.
- Comparación FTP, FTPS y SFTP con capturas de Wireshark.

## Reproducción rápida

```bash
vagrant up
vagrant ssh servidor1   # Firewall: aplicar before.rules y las reglas UFW
vagrant ssh servidor2   # Servicios: vsftpd, resolved.conf y sshd_config
```

Copiar cada archivo a su ruta original, validar la sintaxis (`sudo ufw reload`, `sudo sshd -t`) y reiniciar el servicio correspondiente.

## Nota de seguridad

Este repositorio **no incluye** claves privadas (`ca.key`, `servidor.key`), certificados, contraseñas ni capturas de red. Esos archivos se generan localmente.
