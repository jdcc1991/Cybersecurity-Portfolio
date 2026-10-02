# Technical Notes - Windows & Linux Log Analysis

## 1. Propósito de estas notas

Este documento reúne los principales conceptos técnicos aprendidos durante el laboratorio de análisis de logs en Windows y Linux.

El objetivo es conservar una referencia rápida sobre eventos de autenticación, registros de Linux, filtrado de logs y correlación de eventos desde una perspectiva SOC.

---

# Windows Log Analysis

## 2. Windows Security Log

Windows registra eventos relacionados con autenticación y seguridad dentro del registro:

Security

Estos eventos pueden analizarse mediante:

Event Viewer → Windows Logs → Security

Durante el laboratorio se analizaron principalmente los eventos:

- Event ID 4624
- Event ID 4625

---

## 3. Event ID 4624 - Successful Logon

El Event ID 4624 indica que se creó correctamente una sesión de inicio de sesión.

Sin embargo, un evento 4624 no significa necesariamente que una persona haya iniciado sesión manualmente.

Para interpretar correctamente el evento es necesario revisar campos como:

- TargetUserName
- TargetDomainName
- LogonType
- LogonProcessName
- AuthenticationPackageName
- WorkstationName
- IpAddress
- ProcessName
- ElevatedToken

Durante el laboratorio se generó una autenticación controlada utilizando:

runas /user:soclab cmd

La autenticación correcta produjo un Event ID 4624 asociado al usuario `soclab`.

### Lección aprendida

Un analista SOC no debe interpretar un Event ID únicamente por su número.

Es necesario analizar los campos internos del evento para determinar qué ocurrió realmente.

---

## 4. Event ID 4625 - Failed Logon

El Event ID 4625 representa un intento de autenticación fallido.

Durante el laboratorio se generó intencionalmente un intento fallido utilizando el usuario:

soclab

Los códigos observados fueron:

Status: 0xC000006D  
SubStatus: 0xC000006A

Interpretación:

- `0xC000006D` → fallo de autenticación.
- `0xC000006A` → contraseña incorrecta para una cuenta existente.

También se observó:

Logon Type: 2

El Logon Type 2 representa un inicio de sesión interactivo.

### Lección aprendida

Un único evento 4625 no demuestra por sí mismo la existencia de un ataque.

Para determinar si existe actividad sospechosa deben analizarse factores como:

- cantidad de intentos
- frecuencia
- usuario objetivo
- origen
- dirección IP
- tipo de inicio de sesión
- eventos anteriores y posteriores

Una secuencia repetitiva de eventos 4625 podría ser relevante para investigar posibles intentos de fuerza bruta o password spraying, dependiendo del contexto.

---

## 5. Correlación entre 4625 y 4624

Durante el laboratorio se observó la siguiente secuencia:

Intento de autenticación
        ↓
Contraseña incorrecta
        ↓
Event ID 4625
        ↓
Contraseña correcta
        ↓
Event ID 4624

Ambos eventos estaban relacionados con la cuenta:

soclab

Esto permitió reconstruir la secuencia de autenticación.

### Lección aprendida

La correlación temporal permite comprender mejor un incidente que analizar eventos aislados.

Un analista SOC debe buscar relaciones entre eventos utilizando elementos como:

- timestamp
- usuario
- equipo
- dirección IP
- Logon Type
- proceso
- actividad posterior

---

## 6. Smart Card Authentication Failure

Durante el análisis apareció un Event ID 4625 relacionado con autenticación mediante smart card.

El evento presentaba:

Status: 0xC000006D  
SubStatus: 0xC0000380

Windows interpretaba el evento como un problema relacionado con el PIN de la smart card.

Sin embargo, durante el laboratorio se determinó que el PIN no era realmente incorrecto.

La causa identificada en el entorno fue una interferencia producida por la virtualización y la redirección del dispositivo criptográfico entre Host y Guest.

La interacción entre los servicios de smart card de ambos sistemas podía provocar problemas de comunicación con el dispositivo.

### Lección aprendida

El mensaje mostrado por un sistema operativo no siempre representa la causa raíz del problema.

Un analista debe diferenciar entre:

- lo que registra el log
- la interpretación inicial del sistema
- la causa raíz confirmada mediante investigación

---

# Linux Log Analysis

## 7. systemd-journald

En la máquina Kali Linux utilizada durante el laboratorio no estaba disponible:

/var/log/auth.log

El sistema utilizaba `systemd-journald` para almacenar y consultar eventos.

Los registros fueron analizados mediante:

journalctl

### Lección aprendida

La ubicación y mecanismo de almacenamiento de logs puede variar entre distribuciones Linux.

No se debe asumir que `/var/log/auth.log` estará disponible en todos los sistemas.

---

## 8. sudo y PAM

Durante el laboratorio se generó intencionalmente un fallo de autenticación utilizando `sudo`.

Se observó un evento similar a:

pam_unix(sudo:auth): authentication failure

También apareció:

password check failed for user (kali)

Posteriormente, al ingresar la contraseña correcta, `sudo` ejecutó el comando solicitado como root.

Esto permitió observar la secuencia:

Fallo de autenticación
        ↓
Autenticación correcta
        ↓
Ejecución privilegiada
        ↓
Sesión sudo
        ↓
Cierre de sesión

---

## 9. Correlación mediante PID

Durante el análisis de `sudo` se observó el PID:

9095

El mismo PID apareció relacionado con:

- fallo de autenticación
- ejecución del comando
- apertura de sesión
- cierre de sesión

### Lección aprendida

El PID puede utilizarse para relacionar diferentes eventos generados por el mismo proceso.

Esto permite reconstruir una actividad con mayor precisión.

---

## 10. Filtrado de logs

Durante el laboratorio se utilizaron herramientas como:

- journalctl
- grep

Los filtros permitieron reducir el volumen de eventos y localizar información relacionada con:

- fallos de autenticación
- sudo
- PAM
- usuarios
- PID
- comandos ejecutados
- intervalos de tiempo

### Lección aprendida

Un filtro demasiado amplio puede generar ruido.

Por ejemplo, buscar únicamente:

kali

puede devolver eventos donde `kali` representa el hostname y no necesariamente el usuario.

Los filtros deben construirse utilizando contexto adicional.

---

## 11. Prioridad del evento

Durante el análisis se comprobó que el fallo de autenticación de `sudo` tenía:

PRIORITY=5

correspondiente a nivel `notice`.

Los eventos de apertura y cierre de sesión aparecieron con:

PRIORITY=6

correspondiente a nivel `info`.

### Lección aprendida

La importancia de seguridad de un evento no siempre coincide con su prioridad de syslog.

Filtrar únicamente eventos `warning` o superiores puede ocultar información relevante para una investigación.

---

## 12. Actividad privilegiada sobre archivos sensibles

Se generó de forma controlada la ejecución:

sudo cat /etc/shadow > /dev/null

El objetivo fue generar evidencia de acceso privilegiado a un archivo sensible sin mostrar ni almacenar su contenido.

El journal registró la ejecución del comando mediante `sudo`.

### Lección aprendida

El acceso a archivos sensibles como `/etc/shadow` puede ser relevante durante una investigación.

Sin embargo, la presencia del comando en un log no demuestra por sí sola intención maliciosa.

Debe analizarse:

- usuario
- contexto
- momento
- proceso
- actividad previa
- actividad posterior

---

# SOC Analysis

## 13. Construcción de una línea de tiempo

Los eventos relevantes de Linux fueron organizados cronológicamente.

La secuencia observada fue:

12:23:15 → Password check failed  
12:23:15 → sudo authentication failure  
12:23:25 → sudo ejecuta whoami como root  
14:20:47 → sudo ejecuta cat /etc/shadow

La línea de tiempo permite observar cómo diferentes eventos pueden formar parte de una misma investigación.

---

## 14. Principios aprendidos durante el laboratorio

Los principales aprendizajes fueron:

1. No analizar eventos de forma aislada.
2. Correlacionar eventos mediante tiempo, usuario, PID, origen y proceso.
3. Diferenciar autenticaciones exitosas y fallidas.
4. Comprender que un evento exitoso no siempre representa una acción humana.
5. Evitar asumir que un fallo de autenticación representa automáticamente un ataque.
6. Mejorar progresivamente los filtros para reducir falsos positivos y ruido.
7. No depender únicamente de la severidad o prioridad del sistema.
8. Analizar actividad privilegiada dentro de su contexto.
9. Diferenciar el mensaje registrado de la causa raíz confirmada.
10. Construir líneas de tiempo para reconstruir actividades de seguridad.

---

## 15. Perspectiva SOC

Este laboratorio permitió practicar un flujo básico de trabajo similar al utilizado durante un análisis SOC:

Generación del evento
        ↓
Registro del evento
        ↓
Recolección
        ↓
Filtrado
        ↓
Análisis
        ↓
Correlación
        ↓
Construcción de timeline
        ↓
Documentación de hallazgos

El objetivo principal no fue únicamente encontrar eventos, sino aprender a interpretarlos dentro de su contexto y documentar la evidencia de manera reproducible.