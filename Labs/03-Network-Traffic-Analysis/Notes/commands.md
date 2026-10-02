# Commands - Network Traffic Analysis

Este documento registra los principales comandos, filtros y herramientas utilizados durante el Laboratorio 3 de análisis de tráfico de red con Wireshark.

---

## 1. Verificación del entorno

### Versión de Wireshark

```bash
wireshark --version
```

Versión utilizada:

```text
Wireshark 4.6.6
```

### Identificación de interfaces de red

```bash
ip a
```

Interfaz utilizada:

```text
eth0
```

Dirección IPv4 de Kali Linux:

```text
192.168.14.128/24
```

Dirección IPv4 de Windows 10:

```text
192.168.14.129
```

---

## 2. ICMP

Se generaron cuatro solicitudes ICMP Echo Request desde Kali Linux hacia `8.8.8.8`.

```bash
ping -c 4 8.8.8.8
```

Filtro utilizado en Wireshark:

```text
icmp
```

Durante la captura se observaron paquetes:

```text
Echo (ping) request
Echo (ping) reply
```

---

## 3. ARP

### Visualización de vecinos conocidos

```bash
ip neigh
```

### Limpieza de la caché de vecinos

```bash
sudo ip neigh flush dev eth0
```

### Generación de tráfico ARP

```bash
ping -c 1 192.168.14.2
```

Filtro utilizado en Wireshark:

```text
arp
```

Durante la captura se observó el proceso de resolución:

```text
Who has 192.168.14.2? Tell 192.168.14.128
```

seguido de la respuesta:

```text
192.168.14.2 is at 00:50:56:f2:ca:c6
```

---

## 4. DNS

Se realizó una consulta DNS tipo A para el dominio `example.com`.

```bash
dig example.com
```

Filtro general:

```text
dns
```

Filtro utilizado para aislar la consulta y la respuesta analizadas:

```text
frame.number == 183 || frame.number == 184
```

Durante la prueba se observaron las direcciones IPv4 asociadas a `example.com`.

---

## 5. HTTP

### Creación del directorio de prueba

```bash
mkdir -p ~/http-lab
```

### Acceso al directorio

```bash
cd ~/http-lab
```

### Creación del archivo HTML

```bash
echo "Laboratorio SOC - Analisis HTTP con Wireshark" > index.html
```

### Inicio del servidor HTTP

```bash
sudo python3 -m http.server 80 --bind 0.0.0.0
```

Desde Windows 10 se accedió mediante navegador a:

```text
http://192.168.14.128
```

Filtro inicial utilizado en Wireshark:

```text
http
```

Filtro utilizado para aislar la comunicación entre Windows 10 y Kali Linux:

```text
ip.addr == 192.168.14.129 && ip.addr == 192.168.14.128 && tcp.port == 80
```

Stream TCP analizado:

```text
tcp.stream eq 406
```

Mediante `Follow HTTP Stream` fue posible observar información en texto claro como:

```text
GET / HTTP/1.1
HTTP/1.0 200 OK
```

y el contenido:

```text
Laboratorio SOC - Analisis HTTP con Wireshark
```

---

## 6. HTTPS / TLS

Se generó una conexión HTTPS hacia `example.com`.

```bash
curl --http1.1 https://example.com
```

Filtro inicial utilizado:

```text
tls
```

Filtro utilizado para aislar la comunicación HTTPS:

```text
ip.addr == 192.168.14.128 && ip.addr == 172.66.147.243 && tcp.port == 443
```

Stream TCP analizado:

```text
tcp.stream eq 8
```

Durante la captura se observaron elementos como:

```text
Client Hello
Server Hello
Application Data
```

También fue posible identificar:

```text
SNI = example.com
```

A diferencia de HTTP, el contenido de aplicación no pudo observarse directamente en texto claro dentro de la captura.

---

## 7. SSH

### Verificación del servicio SSH

```bash
sudo systemctl status ssh --no-pager
```

Inicialmente el servicio se encontraba detenido.

### Inicio del servidor SSH

```bash
sudo systemctl start ssh
```

### Verificación del estado

```bash
sudo systemctl status ssh --no-pager
```

### Verificación del puerto TCP/22

```bash
sudo ss -tlnp | grep :22
```

Desde Windows 10 se realizó una conexión hacia Kali Linux:

```powershell
ssh kali@192.168.14.128
```

Dentro de la sesión SSH se ejecutaron:

```bash
whoami
hostname
exit
```

Filtro inicial utilizado en Wireshark:

```text
tcp.port == 22
```

Stream TCP analizado:

```text
tcp.stream eq 50
```

Durante la captura se observaron elementos como:

```text
Client: Protocol
Server: Protocol
Key Exchange Init
Encrypted packet
```

Wireshark permitió identificar el establecimiento de la sesión SSH, pero los comandos y credenciales no fueron visibles directamente debido al cifrado.

---

## 8. SMB

### Verificación de Samba

```bash
smbd --version
```

Versión utilizada:

```text
Version 4.24.6-Debian-4.24.6+dfsg-1
```

### Verificación del cliente SMB

```bash
smbclient --version
```

### Creación del directorio compartido

```bash
mkdir -p ~/smb-lab
```

### Creación del archivo de prueba

```bash
echo "Archivo de prueba para analisis SMB con Wireshark" > ~/smb-lab/evidencia-soc.txt
```

### Verificación del archivo

```bash
ls -l ~/smb-lab
```

### Copia de seguridad de la configuración de Samba

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
```

### Edición de la configuración

```bash
sudo nano /etc/samba/smb.conf
```

Configuración agregada:

```ini
[SOC-Lab]
   path = /home/kali/smb-lab
   browseable = yes
   read only = yes
   guest ok = no
```

### Validación de la configuración

```bash
testparm
```

### Configuración del usuario SMB

```bash
sudo smbpasswd -a kali
```

### Reinicio del servicio Samba

```bash
sudo systemctl restart smbd
```

### Verificación del estado

```bash
sudo systemctl status smbd --no-pager
```

### Verificación del puerto TCP/445

```bash
sudo ss -tlnp | grep :445
```

Desde Windows 10 se accedió al recurso:

```text
\\192.168.14.128\SOC-Lab
```

Filtro inicial utilizado en Wireshark:

```text
tcp.port == 445
```

Filtro utilizado para identificar operaciones relacionadas con el archivo:

```text
smb2.filename contains "evidencia-soc.txt"
```

Durante el análisis se observaron operaciones SMB2 como:

```text
Create Request
Read Request
Read Response
GetInfo Request
Close Request
```

Stream TCP identificado:

```text
tcp.stream eq 154
```

---

## 9. TCP SYN Port Scan

Se generó de forma controlada un escaneo TCP SYN desde Kali Linux hacia la máquina Windows 10 del laboratorio.

```bash
sudo nmap -sS -Pn -T3 --top-ports 100 192.168.14.129
```

Opciones utilizadas:

```text
-sS            TCP SYN Scan
-Pn            No realizar descubrimiento previo mediante ping
-T3            Velocidad moderada
--top-ports    Analizar los 100 puertos TCP más comunes
```

Filtro general utilizado para observar tráfico TCP relacionado con Windows:

```text
ip.addr == 192.168.14.129 && tcp
```

Filtro utilizado para identificar paquetes SYN enviados desde Kali Linux:

```text
ip.src == 192.168.14.128 && ip.dst == 192.168.14.129 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Se observaron múltiples solicitudes SYN dirigidas desde un único host hacia numerosos puertos del mismo destino.

Algunos de los puertos observados fueron:

```text
135
139
445
554
587
3306
5900
8080
```

### Búsqueda de respuestas SYN/ACK

```text
ip.src == 192.168.14.129 && ip.dst == 192.168.14.128 && tcp.flags.syn == 1 && tcp.flags.ack == 1
```

No se encontraron paquetes.

### Búsqueda de respuestas RST

```text
ip.src == 192.168.14.129 && ip.dst == 192.168.14.128 && tcp.flags.reset == 1
```

No se encontraron paquetes.

El comportamiento observado fue consistente con puertos filtrados por ausencia de respuesta.

---

## 10. Análisis de conversaciones TCP

Para complementar el análisis del escaneo se utilizó:

```text
Statistics → Conversations → TCP
```

Se observó una única dirección IP origen realizando intentos de conexión hacia numerosos puertos del mismo host destino en un intervalo corto.

Este patrón es compatible con actividad de reconocimiento mediante TCP SYN port scanning.

---

## 11. Procesamiento de capturas PCAPNG

### Extracción del stream HTTP

El archivo original de tráfico HTTP tenía un tamaño elevado, por lo que se extrajo únicamente el stream utilizado durante el análisis.

```bash
tshark -r ~/Downloads/http-normal-traffic.pcapng \
-Y "tcp.stream eq 406" \
-w ~/Downloads/http-traffic-filtered.pcapng
```

### Verificación de la captura HTTP filtrada

```bash
tshark -r ~/Downloads/http-traffic-filtered.pcapng -Y http
```

La captura filtrada conservó:

```text
GET / HTTP/1.1
HTTP/1.0 200 OK
```

---

## 12. Procesamiento de la captura SMB

Se identificó primero el stream TCP relacionado con `evidencia-soc.txt`.

```bash
tshark -r ~/Downloads/smb-traffic.pcapng \
-Y 'smb2.filename contains "evidencia-soc.txt"' \
-T fields -e tcp.stream | sort -n | uniq
```

Stream identificado:

```text
154
```

Posteriormente se extrajo la conversación TCP completa:

```bash
tshark -r ~/Downloads/smb-traffic.pcapng \
-Y "tcp.stream eq 154" \
-w ~/Downloads/smb-traffic-filtered.pcapng
```

Esto permitió conservar el contexto TCP de la sesión SMB utilizada durante el laboratorio.

---

## 13. Capturas PCAPNG conservadas

Los archivos finales utilizados como evidencia fueron:

```text
icmp-normal-traffic.pcapng
arp-normal-traffic.pcapng
dns-normal-traffic.pcapng
http-normal-traffic.pcapng
https-tls-traffic.pcapng
ssh-traffic.pcapng
smb-traffic.pcapng
suspicious-port-scan.pcapng
```

---

## 14. Principales filtros de Wireshark

```text
icmp
```

```text
arp
```

```text
dns
```

```text
http
```

```text
tls
```

```text
tcp.port == 22
```

```text
tcp.port == 445
```

```text
smb2.filename contains "evidencia-soc.txt"
```

```text
tcp.stream eq 8
```

```text
tcp.stream eq 50
```

```text
tcp.stream eq 154
```

```text
tcp.stream eq 406
```

```text
ip.src == 192.168.14.128 && ip.dst == 192.168.14.129 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

```text
ip.src == 192.168.14.129 && ip.dst == 192.168.14.128 && tcp.flags.syn == 1 && tcp.flags.ack == 1
```

```text
ip.src == 192.168.14.129 && ip.dst == 192.168.14.128 && tcp.flags.reset == 1
```

---

## 15. Resumen de protocolos analizados

| Protocolo | Puerto / Tipo | Herramienta o método |
|---|---|---|
| ICMP | Echo Request / Echo Reply | ping |
| ARP | Resolución IP-MAC | ip neigh / ping |
| DNS | UDP/53 | dig |
| HTTP | TCP/80 | Python HTTP Server |
| HTTPS/TLS | TCP/443 | curl |
| SSH | TCP/22 | OpenSSH |
| SMB2 | TCP/445 | Samba |
| TCP SYN Scan | Múltiples puertos | Nmap |
| Análisis PCAPNG | Capturas de red | Wireshark / tshark |

---

## 16. Entorno del laboratorio

```text
Kali Linux
IP: 192.168.14.128
Interfaz: eth0
Wireshark: 4.6.6

Windows 10
IP: 192.168.14.129

Virtualización:
VMware Workstation
```

---

## Conclusión

Durante el laboratorio se utilizaron herramientas de línea de comandos y filtros de Wireshark para generar, capturar y analizar diferentes tipos de tráfico de red.

Las pruebas permitieron observar tráfico ICMP, ARP, DNS, HTTP, HTTPS/TLS, SSH y SMB, además de generar de forma controlada un escaneo TCP SYN para analizar un patrón compatible con actividad de reconocimiento.

El uso combinado de Wireshark, tshark, Nmap, OpenSSH, Samba y herramientas estándar de Linux permitió correlacionar acciones realizadas en los sistemas con los paquetes observados en la red.