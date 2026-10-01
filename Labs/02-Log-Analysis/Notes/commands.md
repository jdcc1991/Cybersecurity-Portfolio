# Commands Used - Log Analysis Lab

Este archivo documenta los principales comandos utilizados durante el laboratorio de análisis de logs en Windows y Linux.

---

## Windows

### Verificar información del sistema

```cmd
winver
```

Permite consultar la versión y compilación de Windows utilizada en el laboratorio.

```cmd
hostname
```

Muestra el nombre del equipo. En este laboratorio se utilizó para identificar el host que generaba los eventos de seguridad.

```cmd
whoami
```

Muestra la cuenta asociada a la sesión actual. Se utilizó para comprobar qué usuario estaba autenticado antes y después de las pruebas.

```cmd
ipconfig
```

Muestra la configuración de red del sistema Windows y permite identificar las interfaces y direcciones IP de la máquina virtual.

---

### Abrir el Visor de eventos

```cmd
eventvwr.msc
```

Abre el Visor de eventos de Windows, utilizado para analizar:

- **Event ID 4624:** inicio de sesión exitoso.
- **Event ID 4625:** intento de inicio de sesión fallido.

---

### Crear usuario de laboratorio

```cmd
net user soclab * /add
```

Crea el usuario local `soclab` solicitando la contraseña de forma interactiva.

La cuenta fue utilizada exclusivamente para generar eventos controlados de autenticación durante el laboratorio.

---

### Probar autenticación con credenciales alternativas

```cmd
runas /user:soclab cmd
```

Intenta iniciar una nueva instancia de `cmd` utilizando las credenciales del usuario `soclab`.

Durante el laboratorio se realizaron dos pruebas:

1. Contraseña incorrecta para generar un **Event ID 4625**.
2. Contraseña correcta para generar un **Event ID 4624**.

Después de la autenticación correcta se utilizó:

```cmd
whoami
```

para confirmar que la nueva terminal estaba ejecutándose como:

```text
desktop-obsogjn\soclab
```

---

## Linux - Kali Linux

En Kali Linux se utilizó `systemd-journald` mediante `journalctl` para consultar y analizar los registros del sistema.

### Consultar eventos recientes

```bash
sudo journalctl --no-pager -n 20
```

Muestra los últimos 20 eventos registrados por el journal. Se utilizó para comprobar que el sistema estaba almacenando correctamente eventos relacionados con `sudo` y PAM.

---

### Generar un intento de autenticación controlado

```bash
sudo -k
sudo whoami
```

`sudo -k` invalida las credenciales almacenadas temporalmente por `sudo`, obligando a solicitar nuevamente la contraseña.

Durante la prueba se introdujo primero una contraseña incorrecta y posteriormente la contraseña correcta para generar eventos de autenticación observables en los logs.

---

### Buscar eventos relacionados con autenticación

```bash
sudo journalctl --since "10 minutes ago" --no-pager | grep -Ei "authentication failure|incorrectpassword|sudo|pam_unix"
```

Filtra eventos recientes relacionados con `sudo`, PAM y fallos de autenticación.

---

### Analizar una secuencia específica de eventos de sudo

```bash
sudo journalctl --since "2026-10-01 12:23:00" --until "2026-10-01 12:24:00" --no-pager | grep -E "sudo\[9095\]"
```

Permite correlacionar los eventos asociados al proceso `sudo` con PID `9095`, utilizado durante la prueba de autenticación.

---

### Filtrar fallos de autenticación

```bash
sudo journalctl --since "2026-10-01 12:20:00" --until "2026-10-01 12:25:00" --no-pager | grep -Ei "authentication failure|password check failed"
```

Reduce los resultados a eventos directamente relacionados con fallos de autenticación.

---

### Filtrar eventos generados por sudo

```bash
sudo journalctl _COMM=sudo --since "2026-10-01 12:20:00" --until "2026-10-01 12:25:00" --no-pager
```

Utiliza el campo `_COMM` del journal para mostrar únicamente eventos cuyo proceso corresponde a `sudo`.

Este filtro puede reducir el ruido, pero también puede excluir eventos relacionados generados por otros procesos, como `unix_chkpwd`.

---

### Filtrar eventos relacionados con el usuario kali

```bash
sudo journalctl --since "2026-10-01 12:20:00" --until "2026-10-01 12:25:00" --no-pager | grep -E "user(=| )(\()?kali|by kali|ruser=kali|uid=1000"
```

Busca referencias al usuario `kali` utilizando campos de contexto en lugar de buscar únicamente la palabra `kali`.

Esto evita parte de los falsos positivos producidos porque `kali` también corresponde al hostname del sistema.

---

### Analizar eventos según su prioridad

```bash
sudo journalctl -p warning --since "2026-10-01 12:20:00" --until "2026-10-01 12:25:00" --no-pager
```

Consulta eventos con prioridad `warning` o superior. Durante el laboratorio no devolvió eventos para el intervalo analizado.

```bash
sudo journalctl -p notice --since "2026-10-01 12:23:00" --until "2026-10-01 12:24:00" --no-pager
```

Permitió visualizar el fallo de autenticación y la ejecución del comando.

```bash
sudo journalctl -p info --since "2026-10-01 12:23:00" --until "2026-10-01 12:24:00" --no-pager
```

Permitió observar una secuencia más completa, incluyendo la apertura y cierre de la sesión.

---

### Examinar un evento en formato detallado

```bash
sudo journalctl _PID=9095 --since "2026-10-01 12:23:00" --until "2026-10-01 12:24:00" -o verbose --no-pager
```

Muestra los campos internos de los eventos asociados al PID `9095`.

Se utilizó para identificar información como:

- `PRIORITY`
- `SYSLOG_IDENTIFIER`
- `_COMM`
- `_EXE`
- `_CMDLINE`
- `MESSAGE`
- `_PID`

---

### Generar actividad controlada sobre un archivo sensible

```bash
sudo cat /etc/shadow > /dev/null
```

Simula una acción privilegiada sobre un archivo sensible del sistema.

La salida se redirigió a `/dev/null`, por lo que el contenido de `/etc/shadow` no fue mostrado ni almacenado.

---

### Buscar el acceso a /etc/shadow

```bash
sudo journalctl _COMM=sudo --since "2026-10-01 14:20:00" --until "2026-10-01 14:21:00" --no-pager | grep "/etc/shadow"
```

Permite identificar en los registros la ejecución del comando privilegiado sobre `/etc/shadow`.

---

### Construir una línea de tiempo de eventos de seguridad

```bash
sudo journalctl --since "2026-10-01 12:23:00" --until "2026-10-01 14:21:00" --no-pager | grep -Ei "authentication failure|password check failed|COMMAND=/usr/bin/whoami|COMMAND=/usr/bin/cat /etc/shadow"
```

Combina varios indicadores para reconstruir una línea de tiempo simplificada:

1. Fallo en la comprobación de contraseña.
2. Fallo de autenticación mediante PAM.
3. Ejecución correcta de un comando privilegiado.
4. Acceso controlado a `/etc/shadow`.

Esta técnica permite correlacionar eventos separados y reconstruir una secuencia de actividad relevante para un análisis SOC.