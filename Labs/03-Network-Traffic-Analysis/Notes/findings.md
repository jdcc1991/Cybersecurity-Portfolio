# Findings - Network Traffic Analysis

Este documento registra los principales hallazgos obtenidos durante el Laboratorio 3 de análisis de tráfico de red con Wireshark.

El objetivo fue diferenciar tráfico legítimo de red, comprender el comportamiento de distintos protocolos y reconocer patrones compatibles con actividad sospechosa desde una perspectiva SOC.

---

## Finding 01 - Tráfico ICMP legítimo

### Información

- Origen: `192.168.14.128`
- Destino: `8.8.8.8`
- Protocolo: ICMP
- Tipo de tráfico: Echo Request / Echo Reply

### Observación

Se generaron cuatro solicitudes ICMP Echo Request desde Kali Linux hacia `8.8.8.8`.

Wireshark permitió identificar cuatro solicitudes y sus correspondientes respuestas Echo Reply.

### Evidencia observada

```text
192.168.14.128 → 8.8.8.8
ICMP Echo Request

8.8.8.8 → 192.168.14.128
ICMP Echo Reply
```

### Análisis SOC

El comportamiento observado corresponde a una prueba legítima de conectividad mediante `ping`.

ICMP es utilizado habitualmente para diagnóstico de red. Sin embargo, múltiples solicitudes ICMP dirigidas hacia numerosos sistemas podrían utilizarse durante actividades de reconocimiento.

### Clasificación

```text
Actividad legítima
```

### Evidencia

```text
Screenshots/01-icmp-analysis.png
Captures/icmp-normal-traffic.pcapng
```

---

## Finding 02 - Resolución ARP

### Información

- Host origen: `192.168.14.128`
- Host consultado: `192.168.14.2`
- Dirección MAC identificada: `00:50:56:f2:ca:c6`
- Protocolo: ARP

### Observación

Kali Linux necesitó identificar qué dirección MAC correspondía a la dirección IP `192.168.14.2`.

Wireshark registró una solicitud:

```text
Who has 192.168.14.2? Tell 192.168.14.128
```

Posteriormente se observó la respuesta:

```text
192.168.14.2 is at 00:50:56:f2:ca:c6
```

La asociación también pudo verificarse mediante la tabla de vecinos del sistema.

### Análisis SOC

El tráfico corresponde al funcionamiento normal del protocolo ARP dentro de una red local.

Durante esta captura no se observaron evidencias suficientes para indicar ARP spoofing o ARP poisoning.

En un entorno real, cambios inesperados en la dirección MAC asociada a una misma IP podrían requerir investigación adicional.

### Clasificación

```text
Actividad legítima
```

### Evidencia

```text
Screenshots/02-arp-analysis.png
Captures/arp-normal-traffic.pcapng
```

---

## Finding 03 - Resolución DNS

### Información

- Cliente: `192.168.14.128`
- Servidor DNS: `192.168.14.2`
- Dominio consultado: `example.com`
- Tipo de consulta: `A`
- Protocolo de transporte: UDP/53

### Observación

Se realizó una consulta DNS tipo A para determinar las direcciones IPv4 asociadas a `example.com`.

Wireshark permitió identificar la consulta y su correspondiente respuesta.

Durante la prueba fueron observadas las siguientes direcciones:

```text
104.20.23.154
172.66.147.243
```

También se detectaron consultas DNS generadas en segundo plano por otras aplicaciones.

### Análisis SOC

DNS proporciona información de alto valor durante una investigación, ya que permite identificar qué dominios están siendo consultados por un sistema.

La presencia de tráfico DNS adicional demostró la necesidad de correlacionar correctamente los paquetes antes de atribuir una comunicación a una actividad específica.

En esta captura no se identificaron indicadores suficientes de actividad maliciosa.

### Clasificación

```text
Actividad legítima
```

### Evidencia

```text
Screenshots/03-dns-analysis.png
Captures/dns-normal-traffic.pcapng
```

---

## Finding 04 - Información visible mediante HTTP

### Información

- Cliente: Windows 10
- IP cliente: `192.168.14.129`
- Servidor: Kali Linux
- IP servidor: `192.168.14.128`
- Puerto: TCP/80
- Protocolo: HTTP

### Observación

Se levantó un servidor HTTP en Kali Linux y se accedió desde Windows 10.

Wireshark permitió identificar directamente una solicitud:

```text
GET / HTTP/1.1
```

y la respuesta:

```text
HTTP/1.0 200 OK
```

Mediante `Follow HTTP Stream` también fue posible observar información como:

```text
Host: 192.168.14.128
User-Agent: ...
```

y el contenido transmitido:

```text
Laboratorio SOC - Analisis HTTP con Wireshark
```

### Análisis SOC

La prueba demuestra que HTTP no proporciona cifrado al contenido intercambiado.

Un analista o actor con capacidad para capturar este tráfico puede reconstruir solicitudes, cabeceras y contenido transmitido.

En redes donde se transmite información sensible, HTTP representa un riesgo de exposición frente a protocolos cifrados como HTTPS.

### Clasificación

```text
Actividad legítima con exposición de información en texto claro
```

### Evidencia

```text
Screenshots/04-http-analysis.png
Captures/http-normal-traffic.pcapng
```

---

## Finding 05 - Comunicación protegida mediante HTTPS/TLS

### Información

- Cliente: `192.168.14.128`
- Destino observado: `172.66.147.243`
- Puerto: TCP/443
- Protocolo: TLS
- SNI observado: `example.com`

### Observación

Se realizó una conexión HTTPS hacia `example.com`.

Wireshark permitió identificar elementos del establecimiento de la sesión TLS, entre ellos:

```text
Client Hello
Server Hello
Application Data
```

Durante el Client Hello también pudo observarse:

```text
SNI = example.com
```

El cliente anunció soporte para versiones modernas de TLS.

### Análisis SOC

A diferencia de HTTP, el contenido HTTP transportado mediante TLS no fue visible directamente en texto claro dentro de la captura.

Sin embargo, el cifrado no elimina todos los metadatos disponibles para análisis.

Todavía fue posible observar información como:

- IP origen.
- IP destino.
- Puerto TCP.
- tiempos de comunicación.
- tamaños de paquetes.
- establecimiento de la sesión TLS.
- determinados elementos del handshake.

Esto demuestra que el tráfico cifrado continúa proporcionando información útil para análisis de red.

### Clasificación

```text
Actividad legítima cifrada
```

### Evidencia

```text
Screenshots/05-https-tls-analysis.png
Captures/https-tls-traffic.pcapng
```

---

## Finding 06 - Sesión SSH cifrada

### Información

- Cliente: Windows 10
- IP cliente: `192.168.14.129`
- Servidor: Kali Linux
- IP servidor: `192.168.14.128`
- Puerto: TCP/22
- Protocolo: SSHv2

### Observación

Se estableció una sesión SSH desde Windows 10 hacia Kali Linux.

Wireshark permitió identificar el establecimiento TCP y los banners SSH intercambiados.

Se observaron elementos como:

```text
Client: Protocol
Server: Protocol
Key Exchange Init
Encrypted packet
```

El cliente fue identificado como una implementación de OpenSSH para Windows y el servidor como OpenSSH sobre Kali/Debian.

Dentro de la sesión se ejecutaron comandos como:

```text
whoami
hostname
```

pero estos no fueron visibles directamente en la captura.

### Análisis SOC

SSH protege mediante cifrado las credenciales, comandos y contenido de la sesión.

Sin embargo, un analista todavía puede identificar:

- IP cliente.
- IP servidor.
- puerto utilizado.
- dirección de la conexión.
- versión o implementación SSH.
- establecimiento del intercambio de claves.
- duración y volumen aproximado de la sesión.

### Clasificación

```text
Actividad legítima cifrada
```

### Evidencia

```text
Screenshots/06-ssh-analysis.png
Captures/ssh-traffic.pcapng
```

---

## Finding 07 - Acceso a recurso mediante SMB2

### Información

- Cliente: Windows 10
- IP cliente: `192.168.14.129`
- Servidor: Kali Linux
- IP servidor: `192.168.14.128`
- Puerto: TCP/445
- Protocolo: SMB2
- Recurso: `SOC-Lab`
- Archivo: `evidencia-soc.txt`

### Observación

Windows 10 accedió a un recurso compartido alojado en Kali Linux mediante Samba.

Wireshark permitió identificar operaciones SMB2 relacionadas con:

```text
evidencia-soc.txt
```

Entre las operaciones observadas estuvieron:

```text
Create Request
Read Request
Read Response
GetInfo Request
Close Request
```

La captura permitió identificar además el recurso:

```text
\\192.168.14.128\SOC-Lab
```

### Análisis SOC

SMB es un protocolo relevante para análisis defensivo porque se utiliza ampliamente para compartir archivos y recursos dentro de redes Windows.

La identificación de nombres de archivos, recursos y operaciones puede aportar contexto durante investigaciones relacionadas con:

- movimiento lateral,
- acceso no autorizado,
- transferencia de archivos,
- ejecución remota,
- propagación de malware.

En esta prueba la actividad fue generada de forma controlada y corresponde a un acceso autorizado.

### Clasificación

```text
Actividad legítima controlada
```

### Evidencia

```text
Screenshots/07-smb-analysis.png
Captures/smb-traffic.pcapng
```

---

# Finding 08 - Actividad de reconocimiento mediante TCP SYN Port Scan

### Información

- IP origen: `192.168.14.128`
- Sistema origen: Kali Linux
- IP destino: `192.168.14.129`
- Sistema destino: Windows 10
- Técnica observada: TCP SYN Port Scan

### Observación

Se identificó una gran cantidad de paquetes TCP SYN enviados desde un único sistema origen hacia múltiples puertos del mismo host destino durante un intervalo reducido.

Entre los puertos observados estuvieron:

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

El patrón general observado fue:

```text
192.168.14.128 → 192.168.14.129:135  SYN
192.168.14.128 → 192.168.14.129:139  SYN
192.168.14.128 → 192.168.14.129:445  SYN
192.168.14.128 → 192.168.14.129:554  SYN
192.168.14.128 → 192.168.14.129:3306 SYN
192.168.14.128 → 192.168.14.129:5900 SYN
192.168.14.128 → 192.168.14.129:8080 SYN
```

### Análisis mediante Wireshark

Se utilizó un filtro para identificar paquetes SYN iniciales:

```text
ip.src == 192.168.14.128 && ip.dst == 192.168.14.129 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Posteriormente se revisó:

```text
Statistics → Conversations → TCP
```

La vista de conversaciones permitió observar que un único host estaba intentando iniciar conexiones hacia numerosos puertos del mismo sistema destino.

Este comportamiento es consistente con un proceso de enumeración de servicios.

### Búsqueda de respuestas SYN/ACK

Se buscó tráfico correspondiente a posibles puertos abiertos:

```text
ip.src == 192.168.14.129 && ip.dst == 192.168.14.128 && tcp.flags.syn == 1 && tcp.flags.ack == 1
```

Resultado:

```text
0 paquetes
```

### Búsqueda de respuestas RST

Se buscaron respuestas asociadas normalmente a puertos cerrados:

```text
ip.src == 192.168.14.129 && ip.dst == 192.168.14.128 && tcp.flags.reset == 1
```

Resultado:

```text
0 paquetes
```

### Correlación

Durante la prueba controlada, Nmap clasificó los puertos analizados como:

```text
filtered (no-response)
```

Wireshark permitió corroborar que se enviaron múltiples SYN pero no se observaron respuestas SYN/ACK ni RST provenientes del sistema destino.

También se observaron retransmisiones de solicitudes SYN.

### Interpretación SOC

La combinación de los siguientes indicadores permite formular una hipótesis de reconocimiento:

- Un único host origen.
- Un único sistema objetivo.
- Numerosos puertos destino.
- Gran cantidad de paquetes SYN.
- Intervalo temporal reducido.
- Ausencia de establecimiento normal de múltiples sesiones TCP.
- Retransmisiones de solicitudes SYN.

El patrón observado es compatible con:

```text
TCP SYN Port Scanning
```

La captura por sí sola no permite afirmar intención maliciosa. En un entorno corporativo sería necesario correlacionar el evento con otros datos antes de clasificarlo como ataque confirmado.

### Clasificación

```text
Actividad sospechosa - Reconocimiento
```

### Severidad de laboratorio

```text
Media
```

La severidad indicada corresponde únicamente al contexto didáctico del laboratorio y no representa una clasificación universal para entornos productivos.

### Acción recomendada en un entorno real

Un Analista SOC podría:

- identificar el sistema origen,
- validar si el escaneo fue autorizado,
- revisar logs del firewall,
- analizar eventos del host destino,
- verificar intentos posteriores de conexión,
- revisar alertas del IDS/IPS,
- determinar si existen otros hosts escaneados,
- correlacionar el evento con inteligencia de amenazas cuando corresponda.

### Evidencia

```text
Screenshots/08-suspicious-port-scan.png
Screenshots/09-port-scan-conversations.png
Captures/suspicious-port-scan.pcapng
```

---

# Correlación general de hallazgos

| ID | Protocolo / Actividad | Resultado |
|---|---|---|
| F01 | ICMP | Comunicación legítima |
| F02 | ARP | Resolución IP-MAC legítima |
| F03 | DNS | Resolución de dominio legítima |
| F04 | HTTP | Contenido visible en texto claro |
| F05 | HTTPS/TLS | Contenido protegido mediante cifrado |
| F06 | SSH | Sesión remota cifrada |
| F07 | SMB2 | Acceso autorizado a recurso compartido |
| F08 | TCP SYN Scan | Patrón compatible con reconocimiento |

---

# Indicadores identificados durante el laboratorio

## Indicadores de tráfico normal

```text
ICMP Echo Request / Echo Reply
ARP Request / Reply
DNS Query / Response
HTTP GET / 200 OK
TLS Client Hello / Server Hello
SSH Key Exchange
SMB2 Create / Read / Close
```

## Indicadores que pueden requerir investigación

```text
Múltiples SYN desde una única IP.
Numerosos puertos destino.
Actividad concentrada en un intervalo corto.
Retransmisiones SYN.
Ausencia de establecimiento normal de las conexiones.
```

---

# Diferencia entre tráfico normal y actividad sospechosa

Uno de los principales aprendizajes del laboratorio fue que un paquete aislado no suele ser suficiente para determinar si existe actividad maliciosa.

Por ejemplo:

```text
Un paquete SYN → comportamiento normal.
```

Mientras que:

```text
Cientos de SYN
+
Una misma IP origen
+
Múltiples puertos destino
+
Intervalo corto
```

pueden indicar una actividad de reconocimiento que requiere investigación.

El contexto, la frecuencia, la dirección del tráfico y la correlación entre eventos son fundamentales para realizar un análisis adecuado.

---

# Conclusión

El laboratorio permitió analizar diferentes protocolos desde una perspectiva defensiva y comprender qué información puede extraerse mediante inspección de tráfico de red.

Wireshark permitió diferenciar comunicaciones legítimas de un patrón compatible con reconocimiento mediante TCP SYN port scanning.

Las pruebas también demostraron las diferencias entre protocolos sin cifrado, como HTTP, y protocolos que protegen el contenido mediante cifrado, como HTTPS/TLS y SSH.

El análisis confirmó que incluso cuando el contenido está cifrado, los metadatos de red continúan siendo útiles para identificar sistemas, servicios, patrones de comunicación y posibles anomalías.

La identificación del TCP SYN Scan permitió aplicar un flujo de trabajo similar al utilizado en operaciones SOC:

```text
Captura
   ↓
Filtrado
   ↓
Identificación del patrón
   ↓
Correlación
   ↓
Análisis
   ↓
Clasificación
   ↓
Documentación
```

Este enfoque permite transformar una captura de paquetes en información útil para una investigación de seguridad.