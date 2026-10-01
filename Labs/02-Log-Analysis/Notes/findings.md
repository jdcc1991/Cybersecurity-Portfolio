# Findings - Log Analysis Lab

Este documento presenta los principales hallazgos obtenidos durante el análisis de eventos de seguridad en Windows y Linux. Los eventos fueron generados y analizados dentro de un entorno de laboratorio controlado.

---

## Finding 1 - Failed Windows Authentication

### Descripción

Se generó de forma controlada un intento de autenticación fallido utilizando el usuario local `soclab`.

La prueba se realizó mediante:

`runas /user:soclab cmd`

introduciendo intencionalmente una contraseña incorrecta. Windows registró el intento como un evento de seguridad **Event ID 4625 - An account failed to log on**.

### Datos relevantes

| Campo | Valor |
|---|---|
| Event ID | 4625 |
| Usuario objetivo | `soclab` |
| Dominio | `DESKTOP-OBSOGJN` |
| Status | `0xC000006D` |
| SubStatus | `0xC000006A` |
| Logon Type | `2` |
| Logon Process | `seclogo` |
| Authentication Package | `Negotiate` |
| Workstation | `DESKTOP-OBSOGJN` |
| Source Address | `::1` |

### Análisis

El código `0xC000006D` indica que la autenticación no fue válida, mientras que el SubStatus `0xC000006A` permite determinar que el fallo estuvo relacionado con una contraseña incorrecta para una cuenta existente.

El **Logon Type 2** corresponde a un inicio de sesión interactivo.

El proceso de inicio de sesión `seclogo` es consistente con el uso de credenciales alternativas mediante `runas`.

La dirección `::1` corresponde a la dirección IPv6 de loopback, lo que indica que la actividad se originó localmente en el mismo equipo y no desde un host remoto.

### Conclusión

El evento corresponde al intento de autenticación fallido generado intencionalmente durante el laboratorio contra la cuenta `soclab`.

Desde una perspectiva SOC, un único Event ID 4625 no es suficiente para determinar que existe un ataque. Para evaluar su relevancia sería necesario correlacionarlo con factores como frecuencia de intentos, cuentas afectadas, origen de la actividad y eventos posteriores.

### Evidencia

`windows-event-4625-failed-logon.png`

---

## Finding 2 - Successful Windows Authentication

### Descripción

Después del intento fallido documentado anteriormente, se realizó una segunda autenticación utilizando:

`runas /user:soclab cmd`

En esta ocasión se introdujo la contraseña correcta del usuario `soclab`. Windows registró la autenticación mediante un **Event ID 4624 - An account was successfully logged on**.

Posteriormente, el comando `whoami` confirmó que la nueva terminal estaba ejecutándose bajo la cuenta:

`desktop-obsogjn\soclab`

### Datos relevantes

| Campo | Valor |
|---|---|
| Event ID | 4624 |
| Usuario que inició la operación | `jdcc1` |
| Usuario autenticado | `soclab` |
| Dominio | `DESKTOP-OBSOGJN` |
| Logon Type | `2` |
| Logon Process | `seclogo` |
| Authentication Package | `Negotiate` |
| Workstation | `DESKTOP-OBSOGJN` |
| Source Address | `::1` |
| Elevated Token | `No` |

### Análisis

El **Event ID 4624** indica que Windows completó correctamente una autenticación.

El usuario objetivo fue `soclab`, mientras que `jdcc1` fue la cuenta desde la cual se inició la operación mediante `runas`.

El **Logon Type 2** identifica una autenticación interactiva. El proceso `seclogo` es consistente con el uso de credenciales alternativas.

La dirección `::1` corresponde al loopback IPv6, indicando que la autenticación se originó localmente.

Además, `Elevated Token: No` indica que la nueva sesión no recibió un token elevado.

### Correlación con el Finding 1

Los eventos permiten reconstruir la siguiente secuencia:

`jdcc1` → `runas /user:soclab cmd` → contraseña incorrecta → **Event ID 4625** → contraseña correcta → **Event ID 4624** → `whoami` confirma `soclab`

La correlación demuestra cómo varios eventos pueden utilizarse conjuntamente para comprender una secuencia de autenticación en lugar de analizar cada registro de forma aislada.

### Conclusión

El Event ID 4624 corresponde a la autenticación exitosa generada de forma controlada con la cuenta `soclab`.

Desde una perspectiva SOC, un Event ID 4624 tampoco debe interpretarse automáticamente como actividad legítima únicamente por representar una autenticación exitosa. Su contexto, usuario, tipo de inicio de sesión, origen y relación con otros eventos son necesarios para determinar su relevancia.

### Evidencia

`windows-event-4624-successful-logon.png`

---

## Finding 3 - Smart Card Authentication Failure

### Descripción

Durante el análisis del registro de seguridad de Windows se identificó un segundo **Event ID 4625** diferente al intento controlado realizado contra la cuenta `soclab`.

El evento presentó información relacionada con un proceso de autenticación mediante Smart Card y fue analizado por separado para evitar confundirlo con el fallo de contraseña generado durante la prueba anterior.

### Datos relevantes

| Campo | Valor |
|---|---|
| Event ID | 4625 |
| Subject User | `DESKTOP-OBSOGJN$` |
| Target User | `-` |
| Status | `0xC000006D` |
| SubStatus | `0xC0000380` |
| Logon Type | `2` |
| Logon Process | `User32` |
| Authentication Package | `Negotiate` |
| Process | `C:\Windows\System32\svchost.exe` |
| Source Address | `127.0.0.1` |
| Source Port | `0` |

### Análisis

A diferencia del Finding 1, este evento no identifica a `soclab` como usuario objetivo.

El registro mostró el SubStatus `0xC0000380`, asociado en el evento con un fallo relacionado con el PIN de Smart Card.

Sin embargo, durante la revisión del entorno se determinó que el evento no correspondía a un PIN introducido incorrectamente por el usuario.

La causa identificada en este laboratorio fue un conflicto dentro del entorno virtualizado. El Host y la máquina virtual Guest utilizaban configuraciones de sistema operativo equivalentes y la redirección del dispositivo criptográfico, mediante mecanismos como USB passthrough o redirección RDP, provocaba interferencia entre los servicios `SCardSvr` del Host y del Guest.

Esta condición afectaba la comunicación con la Smart Card y hacía que Windows registrara el fallo como si se tratara de un PIN incorrecto.

### Conclusión

Este caso demuestra que el mensaje o código observado en un log no siempre identifica por sí solo la causa raíz del incidente.

El registro permitió observar el fallo de autenticación, mientras que la causa fue determinada mediante el análisis adicional del entorno virtualizado.

Desde una perspectiva SOC, es importante diferenciar entre:

- **Evento observado:** fallo de autenticación registrado por Windows.
- **Interpretación inicial:** problema relacionado con el PIN de Smart Card.
- **Causa identificada en el laboratorio:** conflicto de redirección/comunicación de Smart Card entre Host y Guest.

Esto evita concluir automáticamente que un usuario introdujo un PIN incorrecto únicamente a partir del evento registrado.

### Evidencia

`windows-event-4625-smartcard-failure.png`

---
---

## Finding 4 - Failed sudo Authentication

### Descripción

Durante el análisis de logs en Kali Linux se generó de forma controlada un fallo de autenticación mediante `sudo`.

Primero se invalidaron las credenciales almacenadas temporalmente:

```bash
sudo -k
```

Posteriormente se ejecutó:

```bash
sudo whoami
```

Se introdujo intencionalmente una contraseña incorrecta y, posteriormente, la contraseña correcta.

Los eventos fueron registrados por `systemd-journald` y analizados mediante `journalctl`.

### Eventos relevantes

```text
Oct 01 12:23:15 kali unix_chkpwd[9161]: password check failed for user (kali)
Oct 01 12:23:15 kali sudo[9095]: pam_unix(sudo:auth): authentication failure; logname=kali uid=1000 euid=0 tty=/dev/pts/0 ruser=kali rhost= user=kali
Oct 01 12:23:25 kali sudo[9095]: kali : TTY=pts/0 ; PWD=/home/kali ; USER=root ; COMMAND=/usr/bin/whoami
Oct 01 12:23:25 kali sudo[9095]: pam_unix(sudo:session): session opened for user root(uid=0) by kali(uid=1000)
```

### Análisis

La primera entrada muestra que `unix_chkpwd` detectó un fallo durante la comprobación de la contraseña del usuario `kali`.

Posteriormente, PAM registró:

```text
pam_unix(sudo:auth): authentication failure
```

confirmando el fallo de autenticación asociado al uso de `sudo`.

Los campos del evento permiten obtener contexto adicional:

| Campo | Valor |
|---|---|
| Usuario | `kali` |
| UID | `1000` |
| Terminal | `/dev/pts/0` |
| Proceso | `sudo` |
| PID | `9095` |
| Usuario objetivo posterior | `root` |
| Comando | `/usr/bin/whoami` |

Diez segundos después del fallo aparece una ejecución exitosa de:

```text
COMMAND=/usr/bin/whoami
```

seguida por:

```text
session opened for user root(uid=0) by kali(uid=1000)
```

Esto indica que posteriormente se produjo una autenticación correcta y `sudo` abrió una sesión privilegiada para ejecutar el comando.

### Correlación mediante PID

El PID `9095` permitió relacionar varios eventos generados durante la misma operación de `sudo`.

La secuencia observada fue:

**`sudo whoami` → contraseña incorrecta → fallo de autenticación PAM → contraseña correcta → ejecución como root → apertura de sesión → cierre de sesión**

Esta correlación permite reconstruir la actividad con mayor precisión que analizando cada entrada del journal de forma independiente.

### Análisis de prioridad

Al consultar los eventos en formato verbose se observó que el fallo de autenticación tenía:

```text
PRIORITY=5
```

correspondiente al nivel `notice`.

Por esta razón, una búsqueda limitada únicamente a eventos `warning` o superiores no mostró este fallo de autenticación.

Al ampliar la consulta a `notice` e `info` fue posible recuperar más elementos de la secuencia.

Esto demuestra que la relevancia de seguridad de un evento no depende exclusivamente de su prioridad dentro del sistema de logging.

### Conclusión

Los registros permitieron identificar y correlacionar un fallo de autenticación seguido de una ejecución privilegiada exitosa.

En un entorno SOC, esta secuencia debería analizarse considerando factores adicionales como frecuencia, usuario, comandos ejecutados, horario y actividad posterior.

En este laboratorio la actividad fue generada intencionalmente con fines de aprendizaje y no representa una intrusión real.

### Evidencia

- `linux-sudo-authentication-analysis.png`
- `linux-authentication-failure-filter.png`

---

## Finding 5 - Privileged Access to a Sensitive File

### Descripción

Durante el laboratorio se generó de forma controlada una actividad privilegiada sobre el archivo `/etc/shadow`.

Se ejecutó:

```bash
sudo cat /etc/shadow > /dev/null
```

El objetivo fue generar un evento observable en los logs sin mostrar ni almacenar el contenido del archivo.

### Evento relevante

El análisis mediante `journalctl` permitió identificar:

```text
Oct 01 14:20:47 kali sudo[50234]: kali : TTY=pts/0 ; PWD=/home/kali ; USER=root ; COMMAND=/usr/bin/cat /etc/shadow
```

### Datos relevantes

| Campo | Valor |
|---|---|
| Usuario | `kali` |
| Terminal | `pts/0` |
| Directorio de trabajo | `/home/kali` |
| Usuario objetivo | `root` |
| Comando | `/usr/bin/cat /etc/shadow` |
| PID | `50234` |
| Archivo accedido | `/etc/shadow` |

### Análisis

El registro muestra que el usuario `kali` utilizó `sudo` para ejecutar `/usr/bin/cat /etc/shadow` con privilegios de `root`.

`/etc/shadow` es un archivo sensible del sistema Linux relacionado con la información utilizada para la autenticación de cuentas.

Durante la prueba, la salida del comando fue redirigida a:

```text
/dev/null
```

por lo que el contenido de `/etc/shadow` no fue mostrado ni almacenado.

El evento registrado por `sudo` permite identificar quién ejecutó la acción, desde qué terminal, con qué usuario privilegiado y qué comando fue utilizado.

### Perspectiva SOC

La ejecución de un comando sobre un archivo sensible puede ser relevante durante una investigación de seguridad.

Sin embargo, la presencia del comando en los logs no demuestra por sí sola actividad maliciosa.

Para determinar su relevancia sería necesario correlacionarlo con información adicional, como:

- Usuario que realizó la acción.
- Actividad previa y posterior.
- Comandos relacionados.
- Momento de ejecución.
- Comportamiento habitual del usuario o sistema.
- Existencia de otros indicadores de compromiso.

En este laboratorio, la actividad fue generada intencionalmente y de forma controlada.

### Conclusión

El análisis permitió identificar una acción privilegiada sobre `/etc/shadow` y recuperar información suficiente para reconstruir quién ejecutó el comando y bajo qué contexto.

Este ejercicio demuestra cómo los registros de `sudo` pueden utilizarse para investigar actividad privilegiada sobre recursos sensibles sin asumir automáticamente que dicha actividad representa un ataque.

### Evidencia

`linux-sudo-sensitive-file-access.png`

---

## Finding 6 - Security Event Timeline and Correlation

### Descripción

Después de analizar individualmente los eventos de autenticación y actividad privilegiada, se construyó una línea de tiempo utilizando `journalctl` y filtros con `grep`.

El objetivo fue correlacionar diferentes eventos de seguridad registrados durante el laboratorio y reconstruir la secuencia de actividad observada.

### Consulta utilizada

```bash
sudo journalctl --since "2026-10-01 12:23:00" --until "2026-10-01 14:21:00" --no-pager | grep -Ei "authentication failure|password check failed|COMMAND=/usr/bin/whoami|COMMAND=/usr/bin/cat /etc/shadow"
```

### Línea de tiempo

| Hora | Evento | Interpretación |
|---|---|---|
| 12:23:15 | `password check failed for user (kali)` | Fallo en la comprobación de contraseña |
| 12:23:15 | `pam_unix(sudo:auth): authentication failure` | PAM registra el fallo de autenticación |
| 12:23:25 | `COMMAND=/usr/bin/whoami` | Ejecución posterior del comando mediante `sudo` |
| 14:20:47 | `COMMAND=/usr/bin/cat /etc/shadow` | Actividad privilegiada sobre un archivo sensible |

### Análisis

Los dos primeros eventos ocurrieron a las `12:23:15` y corresponden al intento de autenticación fallido generado intencionalmente durante la prueba con `sudo`.

Diez segundos después, a las `12:23:25`, aparece la ejecución de:

```text
COMMAND=/usr/bin/whoami
```

Los eventos analizados previamente permitieron confirmar que esta ejecución estuvo acompañada por la apertura de una sesión para `root`, indicando que posteriormente se produjo una autenticación correcta.

Más tarde, a las `14:20:47`, se registró:

```text
COMMAND=/usr/bin/cat /etc/shadow
```

correspondiente a la actividad controlada sobre el archivo `/etc/shadow`.

### Secuencia reconstruida

La actividad observada puede resumirse como:

**Fallo de contraseña → fallo de autenticación PAM → autenticación posterior exitosa con sudo → ejecución privilegiada de `whoami` → actividad posterior sobre `/etc/shadow`**

Esta secuencia fue generada intencionalmente durante diferentes etapas del laboratorio y no representa la actividad de un atacante real.

### Perspectiva SOC

El análisis demuestra que un evento aislado proporciona información limitada.

La correlación temporal permite comprender mejor el contexto y responder preguntas como:

- ¿Qué ocurrió primero?
- ¿Qué usuario estuvo involucrado?
- ¿La autenticación fallida fue seguida por una autenticación exitosa?
- ¿Qué comandos se ejecutaron posteriormente?
- ¿Se accedió a recursos sensibles?

En un entorno real, una secuencia similar podría justificar una investigación adicional, especialmente si los eventos fueran inesperados o estuvieran asociados con otros indicadores de compromiso.

### Conclusión

La construcción de una línea de tiempo permitió transformar varios registros independientes en una secuencia comprensible de actividad.

Este ejercicio demuestra la importancia de la correlación de eventos durante el análisis SOC y la necesidad de interpretar los logs dentro de su contexto antes de determinar si una actividad es legítima o potencialmente sospechosa.

### Evidencia

`linux-security-event-timeline.png`