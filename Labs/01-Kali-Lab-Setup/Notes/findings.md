# Findings

## Resumen

Durante la preparación del entorno de trabajo se verificó que todos los componentes necesarios para el desarrollo de los laboratorios de ciberseguridad quedaron correctamente instalados y configurados.

---

# Hallazgos Técnicos

## 1. Virtualización

- VMware Workstation Pro se instaló correctamente.
- La máquina virtual de Kali Linux se importó sin errores.
- Los recursos de hardware asignados fueron suficientes para un funcionamiento estable.

---

## 2. Sistema Operativo

- Kali Linux inició correctamente.
- El sistema fue actualizado a la versión más reciente disponible mediante APT.
- No se presentaron errores durante el proceso de actualización.

---

## 3. Conectividad

- La configuración de red NAT permitió acceso estable a Internet.
- La prueba de conectividad mediante `ping` fue satisfactoria.
- La interfaz de red obtuvo una dirección IP correctamente.

---

## 4. Herramientas

Las siguientes herramientas quedaron disponibles y listas para los próximos laboratorios:

- Git
- Visual Studio Code
- Terminal Bash

---

## 5. Control de Versiones

- Se inicializó un repositorio Git local.
- Se realizó el primer commit del proyecto.
- El repositorio fue conectado correctamente con GitHub.
- El primer `push` se realizó sin errores.

---

## 6. Recuperación del Entorno

Se creó un **Snapshot Base** que permitirá restaurar rápidamente el laboratorio en caso de fallos o configuraciones incorrectas durante futuras prácticas.

---

# Conclusión

El laboratorio permitió establecer un entorno de trabajo estable, documentado y reproducible, preparado para el desarrollo de los siguientes laboratorios del portafolio de ciberseguridad.