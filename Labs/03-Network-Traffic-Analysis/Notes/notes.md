# Notes - Network Traffic Analysis

Este documento contiene los principales conceptos técnicos aprendidos durante el Laboratorio 3 de análisis de tráfico de red con Wireshark.

El laboratorio se desarrolló utilizando un entorno virtualizado compuesto principalmente por Kali Linux y Windows 10.

---

# 1. Entorno del laboratorio

## Kali Linux

```text
IP: 192.168.14.128
Interfaz: eth0
```

Kali Linux fue utilizado principalmente como:

- estación de análisis,
- sistema donde se ejecutó Wireshark,
- servidor HTTP,
- servidor SSH,
- servidor SMB mediante Samba,
- origen del escaneo de puertos controlado.

## Windows 10

```text
IP: 192.168.14.129
```

Windows 10 fue utilizado como:

- cliente HTTP,
- cliente SSH,
- cliente SMB,
- sistema objetivo durante el escaneo TCP SYN.

## Virtualización

```text
VMware Workstation
```

Ambos sistemas se comunicaron a través de una red virtual.

---

# 2. TCP/IP

TCP/IP es el conjunto de protocolos utilizado para permitir la comunicación entre sistemas dentro de redes locales e Internet.

Durante el laboratorio se analizaron principalmente protocolos pertenecientes a diferentes niveles de la comunicación:

```text
Aplicación
├── HTTP
├── HTTPS
├── DNS
├── SSH
└── SMB

Transporte
├── TCP
└── UDP

Internet
├── IPv4
└── ICMP

Acceso a red
└── ARP
```

Cada protocolo aporta información diferente durante una investigación de seguridad.

---

# 3. Direcciones IP

Las direcciones IP permiten identificar sistemas dentro de una red.

Durante el laboratorio se utilizaron principalmente:

```text
192.168.14.128 → Kali Linux
192.168.14.129 → Windows 10
192.168.14.2   → componente/servicio de la red virtual
```

Durante un análisis SOC es importante identificar:

```text
Source IP
Destination IP
```

Estos valores permiten determinar quién inició una comunicación y hacia qué sistema fue dirigida.

---

# 4. Puertos TCP y UDP

Los puertos permiten identificar los servicios utilizados durante una comunicación.

Algunos de los observados durante el laboratorio fueron:

| Servicio | Puerto |
|---|---:|
| DNS | UDP/53 |
| HTTP | TCP/80 |
| HTTPS | TCP/443 |
| SSH | TCP/22 |
| SMB | TCP/445 |

Los puertos permiten a un Analista SOC realizar una primera aproximación al tipo de servicio involucrado.

Sin embargo, el número de puerto por sí solo no garantiza que realmente se esté utilizando el protocolo esperado.

---

# 5. TCP Three-Way Handshake

Antes de establecer una conexión TCP normal, cliente y servidor realizan normalmente un proceso de tres pasos:

```text
Cliente                  Servidor

   SYN  -------------------->

        <---------------- SYN/ACK

   ACK  -------------------->
```

Este proceso se conoce como:

```text
Three-Way Handshake
```

## SYN

Indica que un sistema desea iniciar una conexión.

## SYN/ACK

Indica que el sistema destino recibió la solicitud y acepta iniciar la comunicación.

## ACK

Confirma el establecimiento de la conexión.

Una vez completado este proceso puede comenzar el intercambio de datos.

---

# 6. RST

El flag:

```text
RST
```

significa:

```text
Reset
```

Puede utilizarse para terminar o rechazar una conexión TCP.

En un escenario de escaneo de puertos, una respuesta RST puede ser consistente con un puerto cerrado.

Ejemplo:

```text
Scanner → SYN → Puerto destino

Destino → RST
```

Sin embargo, siempre debe analizarse el contexto completo antes de interpretar un paquete individual.

---

# 7. ICMP

ICMP significa:

```text
Internet Control Message Protocol
```

Es utilizado principalmente para diagnóstico y control de comunicaciones IP.

Durante el laboratorio se utilizó:

```bash
ping -c 4 8.8.8.8
```

Esto generó:

```text
Echo Request
Echo Reply
```

Flujo observado:

```text
192.168.14.128
      |
      | Echo Request
      v
   8.8.8.8
      |
      | Echo Reply
      v
192.168.14.128
```

## Aplicación defensiva

ICMP puede utilizarse legítimamente para comprobar conectividad.

Sin embargo, múltiples solicitudes hacia numerosos sistemas también pueden aparecer durante actividades de reconocimiento.

Por esta razón, un paquete ICMP aislado no debe considerarse malicioso automáticamente.

---

# 8. ARP

ARP significa:

```text
Address Resolution Protocol
```

Se utiliza dentro de redes locales para descubrir qué dirección MAC corresponde a una determinada dirección IP.

Durante el laboratorio se observó:

```text
Who has 192.168.14.2?
Tell 192.168.14.128
```

seguido de:

```text
192.168.14.2 is at 00:50:56:f2:ca:c6
```

El proceso puede representarse como:

```text
IP
192.168.14.2
      |
      v
ARP Resolution
      |
      v
MAC
00:50:56:f2:ca:c6
```

## Aplicación defensiva

ARP es un protocolo normal y necesario dentro de una LAN.

Sin embargo, alteraciones inesperadas en las asociaciones:

```text
IP ↔ MAC
```

pueden ser relevantes durante investigaciones de:

```text
ARP Spoofing
ARP Poisoning
Man-in-the-Middle
```

---

# 9. DNS

DNS significa:

```text
Domain Name System
```

Su función principal es traducir nombres de dominio a direcciones IP.

Ejemplo:

```text
example.com
      |
      | DNS Query
      v
Servidor DNS
      |
      | DNS Response
      v
Dirección IP
```

Durante el laboratorio se ejecutó:

```bash
dig example.com
```

La consulta analizada fue de tipo:

```text
A
```

que se utiliza para obtener direcciones IPv4.

## Aplicación defensiva

DNS es una fuente de información importante para un SOC porque permite identificar dominios consultados por un sistema.

Puede ayudar durante investigaciones relacionadas con:

- phishing,
- malware,
- command and control,
- dominios sospechosos,
- comunicaciones periódicas,
- DNS tunneling.

Sin embargo, una consulta DNS por sí sola no demuestra actividad maliciosa.

Debe correlacionarse con otros indicadores.

---

# 10. HTTP

HTTP significa:

```text
Hypertext Transfer Protocol
```

Durante el laboratorio se ejecutó un servidor web en Kali Linux:

```text
192.168.14.128:80
```

Windows realizó una solicitud:

```text
GET / HTTP/1.1
```

El servidor respondió:

```text
HTTP/1.0 200 OK
```

Wireshark permitió reconstruir la comunicación mediante:

```text
Follow HTTP Stream
```

y observar directamente:

```text
Host
User-Agent
Método GET
Respuesta HTTP
Contenido HTML
```

## Riesgo principal

HTTP no cifra el contenido de la comunicación.

Por esta razón, un sistema con capacidad de capturar el tráfico puede potencialmente leer la información transmitida.

---

# 11. HTTPS y TLS

HTTPS utiliza HTTP protegido mediante TLS.

Durante el laboratorio se analizó tráfico hacia:

```text
https://example.com
```

Wireshark permitió observar:

```text
Client Hello
Server Hello
Application Data
```

También se identificó en la captura:

```text
SNI = example.com
```

## Diferencia principal frente a HTTP

En HTTP observamos:

```text
GET /
200 OK
Contenido HTML
```

directamente en la captura.

En HTTPS observamos principalmente:

```text
TLS Handshake
Application Data
```

sin poder leer directamente el contenido HTTP transportado.

## Información que puede continuar visible

Aunque el contenido esté cifrado, un analista todavía puede observar información como:

- dirección IP origen,
- dirección IP destino,
- puertos,
- tamaño de los paquetes,
- tiempos,
- volumen de tráfico,
- establecimiento de conexiones,
- determinados metadatos del handshake TLS.

Esto demuestra que:

```text
Tráfico cifrado ≠ tráfico invisible
```

---

# 12. TLS 1.3 y legacy_version

Durante el análisis del Client Hello se observó:

```text
Version: TLS 1.2 (0x0303)
```

aunque Wireshark identificó la negociación como TLS 1.3.

Esto no representa necesariamente un error.

TLS 1.3 mantiene el valor histórico:

```text
0x0303
```

en el campo `legacy_version` por motivos de compatibilidad.

Las versiones realmente soportadas pueden anunciarse mediante:

```text
supported_versions
```

Durante la captura se observaron versiones modernas de TLS disponibles para la negociación.

---

# 13. SSH

SSH significa:

```text
Secure Shell
```

Se utiliza para establecer sesiones remotas cifradas.

Durante el laboratorio se realizó:

```text
Windows 10
192.168.14.129
      |
      | TCP/22
      v
Kali Linux
192.168.14.128
```

Wireshark permitió observar elementos como:

```text
Client Protocol
Server Protocol
Key Exchange Init
Encrypted packet
```

También se pudieron identificar los banners de las implementaciones SSH.

Sin embargo, después del proceso criptográfico no fue posible observar directamente comandos como:

```text
whoami
hostname
```

## Aplicación defensiva

Incluso cuando SSH está cifrado, un SOC puede estudiar:

- quién inició la conexión,
- qué servidor recibió la conexión,
- puerto,
- duración,
- volumen,
- frecuencia,
- implementación SSH,
- comportamiento temporal.

---

# 14. SMB

SMB significa:

```text
Server Message Block
```

Es ampliamente utilizado para compartir archivos y recursos en redes.

Durante el laboratorio Windows accedió al recurso:

```text
\\192.168.14.128\SOC-Lab
```

y posteriormente al archivo:

```text
evidencia-soc.txt
```

Wireshark identificó operaciones SMB2 como:

```text
Create Request
Read Request
Read Response
GetInfo Request
Close Request
```

## Importancia para un SOC

SMB es especialmente relevante porque puede estar presente durante:

- acceso a archivos compartidos,
- administración de sistemas,
- transferencia de archivos,
- movimiento lateral,
- propagación de malware,
- actividad de ransomware.

El contexto determina si una comunicación SMB es legítima o requiere investigación.

---

# 15. Diferencia entre protocolo y comportamiento

Uno de los principales aprendizajes del laboratorio es que:

```text
Protocolo ≠ amenaza
```

Por ejemplo:

```text
ICMP
DNS
SSH
SMB
```

son protocolos legítimos.

Lo que puede convertir una actividad en sospechosa es su comportamiento.

Ejemplo:

```text
Un SYN hacia TCP/443
```

es completamente normal.

Pero:

```text
100 SYN
+
100 puertos diferentes
+
mismo origen
+
mismo destino
+
intervalo corto
```

puede indicar reconocimiento.

---

# 16. TCP SYN Port Scan

Durante el laboratorio se generó un escaneo TCP SYN controlado.

El comportamiento observado fue:

```text
192.168.14.128
      |
      | SYN → 135
      | SYN → 139
      | SYN → 445
      | SYN → 554
      | SYN → 3306
      | SYN → 5900
      | SYN → 8080
      | ...
      v
192.168.14.129
```

La característica principal fue:

```text
Una IP origen
      +
Un host destino
      +
Muchos puertos
      +
Muchos SYN
      +
Poco tiempo
```

Esto constituye un patrón compatible con reconocimiento mediante escaneo de puertos.

---

# 17. Estados de puertos durante un SYN Scan

De forma simplificada, durante un TCP SYN Scan pueden observarse comportamientos como:

## Puerto abierto

```text
Scanner → SYN

Destino → SYN/ACK
```

## Puerto cerrado

```text
Scanner → SYN

Destino → RST
```

## Puerto filtrado o sin respuesta observable

```text
Scanner → SYN

Destino → sin respuesta
```

Durante la prueba realizada no se observaron:

```text
SYN/ACK
RST
```

provenientes del sistema Windows para los puertos analizados.

El resultado fue consistente con:

```text
filtered (no-response)
```

durante la prueba controlada.

---

# 18. Retransmisiones TCP

Durante el escaneo se observaron solicitudes SYN repetidas.

Cuando un sistema envía un paquete y no recibe la respuesta esperada, puede realizar una retransmisión.

Ejemplo:

```text
SYN →
      sin respuesta

SYN →
      sin respuesta
```

La presencia de retransmisiones ayuda a entender por qué se observaron más paquetes que puertos inicialmente analizados.

---

# 19. Statistics → Conversations

Wireshark incluye diferentes herramientas estadísticas.

Durante el laboratorio se utilizó:

```text
Statistics
    ↓
Conversations
    ↓
TCP
```

Esta vista permitió observar múltiples conversaciones asociadas a:

```text
192.168.14.128
        ↓
192.168.14.129
```

utilizando numerosos puertos destino.

Esta forma de análisis permite identificar patrones que pueden ser difíciles de reconocer revisando paquetes individualmente.

---

# 20. Display Filters

Los filtros de visualización permiten reducir el ruido de una captura.

Ejemplos utilizados:

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

También pueden combinarse condiciones:

```text
ip.src == 192.168.14.128 &&
ip.dst == 192.168.14.129 &&
tcp.flags.syn == 1 &&
tcp.flags.ack == 0
```

El filtrado es fundamental para un Analista SOC porque una captura puede contener miles o millones de paquetes.

---

# 21. TCP Streams

Wireshark asigna identificadores a conversaciones TCP.

Ejemplo:

```text
tcp.stream eq 50
```

permite mostrar únicamente una conversación específica.

Durante el laboratorio esta técnica fue utilizada para analizar sesiones:

- HTTP,
- HTTPS/TLS,
- SSH,
- SMB.

Esto permitió eliminar tráfico no relacionado con la investigación.

---

# 22. Follow Stream

La función:

```text
Follow TCP Stream
```

o:

```text
Follow HTTP Stream
```

permite reconstruir una conversación entre cliente y servidor.

Fue especialmente útil durante el análisis HTTP porque permitió visualizar directamente:

```text
Request
Response
Headers
Contenido
```

Cuando el protocolo utiliza cifrado, la misma técnica no permite necesariamente visualizar el contenido de aplicación en texto claro.

---

# 23. PCAPNG

Wireshark almacena capturas utilizando formatos como:

```text
.pcap
.pcapng
```

En este laboratorio se utilizó principalmente:

```text
.pcapng
```

Guardar los archivos de captura permite:

- repetir el análisis,
- aplicar nuevos filtros,
- compartir evidencia,
- validar hallazgos,
- realizar investigaciones posteriores.

Por esta razón se conservaron los PCAPNG relevantes dentro de:

```text
Captures/
```

---

# 24. tshark

`tshark` es la versión de línea de comandos de Wireshark.

Durante el laboratorio fue utilizado para procesar capturas existentes.

Por ejemplo:

```bash
tshark -r captura.pcapng
```

permite analizar una captura desde la terminal.

También puede utilizar filtros:

```bash
tshark -r captura.pcapng -Y http
```

y generar nuevas capturas filtradas.

Esto permitió reducir archivos grandes y conservar únicamente las conversaciones relevantes para el portafolio.

---

# 25. Diferencia entre captura completa y evidencia filtrada

Una captura completa puede contener:

```text
tráfico legítimo,
servicios del sistema,
actualizaciones,
DNS,
broadcast,
conexiones de aplicaciones,
ruido de red.
```

Para una investigación puede ser necesario conservar la captura completa.

Sin embargo, para documentar el laboratorio se generaron también capturas filtradas que contienen únicamente las conversaciones relevantes.

Esto facilita:

- revisión,
- almacenamiento,
- reproducción,
- publicación en GitHub.

---

# 26. Correlación

Un principio importante del análisis SOC es evitar depender de una sola fuente.

Durante el laboratorio se correlacionaron datos procedentes de:

```text
Wireshark
Nmap
Terminal Linux
Servicios del sistema
Windows
tshark
```

Ejemplo del escaneo:

```text
Nmap
  ↓
Reporta puertos filtered
  ↓
Wireshark
  ↓
Observa SYN
  ↓
No observa SYN/ACK ni RST
```

La correlación permite obtener conclusiones más sólidas.

---

# 27. Falsos positivos y contexto

Una actividad técnicamente sospechosa no siempre representa un ataque.

Por ejemplo:

```text
Port Scan
```

puede ser realizado por:

- un atacante,
- un administrador,
- un vulnerability scanner,
- un sistema de monitoreo,
- una auditoría autorizada,
- un laboratorio de seguridad.

Por eso un Analista SOC debe evitar conclusiones precipitadas.

Una descripción adecuada sería:

```text
"Actividad compatible con reconocimiento mediante TCP SYN scanning."
```

en lugar de afirmar automáticamente:

```text
"El sistema está siendo atacado."
```

---

# 28. Flujo básico de análisis SOC aplicado

Durante este laboratorio se aplicó un flujo similar al siguiente:

```text
1. Capturar tráfico
        ↓
2. Identificar sistemas
        ↓
3. Filtrar protocolos
        ↓
4. Analizar conversaciones
        ↓
5. Identificar patrones
        ↓
6. Correlacionar información
        ↓
7. Clasificar actividad
        ↓
8. Documentar evidencia
```

Este proceso puede aplicarse en investigaciones de red más complejas.

---

# 29. Preguntas útiles durante un análisis de red

Durante una investigación un Analista SOC puede preguntarse:

```text
¿Quién inició la comunicación?

¿Cuál es la IP origen?

¿Cuál es la IP destino?

¿Qué protocolo está siendo utilizado?

¿Qué puerto está involucrado?

¿La conexión fue establecida?

¿Existe cifrado?

¿Se observa información en texto claro?

¿Cuántas conexiones existen?

¿La frecuencia es normal?

¿Se están utilizando múltiples puertos?

¿Hay retransmisiones?

¿Existen dominios sospechosos?

¿Se están accediendo recursos SMB?

¿El comportamiento coincide con reconocimiento?

¿Hay otras fuentes que permitan correlacionar el evento?
```

Estas preguntas ayudan a pasar de observar paquetes a realizar análisis de seguridad.

---

# 30. Principales aprendizajes

Durante el laboratorio se reforzaron los siguientes conceptos:

- Captura de tráfico mediante Wireshark.
- Uso de filtros de visualización.
- Identificación de IP origen y destino.
- Interpretación básica de TCP.
- Análisis del Three-Way Handshake.
- Análisis de ICMP.
- Resolución ARP.
- Resolución DNS.
- Diferencias entre HTTP y HTTPS.
- Análisis de TLS.
- Identificación de sesiones SSH.
- Análisis de tráfico SMB2.
- Uso de TCP Streams.
- Uso de Follow Stream.
- Uso de estadísticas de conversaciones.
- Procesamiento de PCAPNG mediante tshark.
- Identificación de múltiples paquetes SYN.
- Reconocimiento de patrones de port scanning.
- Importancia de la correlación.
- Diferencia entre actividad sospechosa y actividad maliciosa confirmada.

---

# Conclusión

El análisis de tráfico de red no consiste únicamente en observar paquetes individuales.

Un Analista SOC debe comprender:

```text
protocolos
+
direcciones IP
+
puertos
+
flags
+
frecuencia
+
dirección del tráfico
+
contexto
+
correlación
```

para determinar qué está ocurriendo dentro de una comunicación.

Durante este laboratorio se analizaron tanto comportamientos normales como un escenario controlado de reconocimiento.

El principal aprendizaje fue que Wireshark permite transformar tráfico de red aparentemente complejo en información útil para responder preguntas como:

```text
Quién se comunicó.
Con quién.
Por qué protocolo.
Por qué puerto.
Qué información fue visible.
Si la comunicación estaba cifrada.
Qué operaciones se realizaron.
Si existe un patrón anómalo.
```

Estas capacidades forman parte de las habilidades fundamentales para el análisis de eventos de seguridad en un entorno SOC.