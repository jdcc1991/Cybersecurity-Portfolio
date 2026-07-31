# 🖥️ Lab 01 - Kali Linux Lab Setup

## 📌 Descripción

En este laboratorio se documenta la preparación del entorno de trabajo que servirá como base para el desarrollo de los siguientes laboratorios de ciberseguridad. Se realizó la instalación y configuración de Kali Linux sobre VMware Workstation Pro, la actualización del sistema operativo, la configuración de Git y Visual Studio Code, así como la creación del primer snapshot para disponer de un punto de restauración estable.

Este laboratorio constituye la base del entorno de prácticas para actividades de reconocimiento, análisis de vulnerabilidades, monitoreo y respuesta a incidentes.

---

# 🎯 Objetivos

## Objetivo general

Preparar un laboratorio de ciberseguridad funcional, estable y documentado que sirva como plataforma para el desarrollo de prácticas orientadas al rol de Analista SOC.

## Objetivos específicos

- Instalar VMware Workstation Pro.
- Implementar Kali Linux como máquina virtual.
- Configurar la red utilizando NAT.
- Actualizar completamente el sistema operativo.
- Configurar Git para el control de versiones.
- Instalar Visual Studio Code como editor principal.
- Crear un Snapshot Base para recuperación rápida.
- Organizar la estructura inicial del portafolio.

---

# 🏗️ Arquitectura del laboratorio

```text
                    Internet
                        │
                 Router del Host
                        │
                 Windows 11 Host
                        │
                VMware Workstation Pro
                        │
                ┌─────────────────┐
                │ Kali Linux 2026 │
                └─────────────────┘
                        │
                      NAT
```

---

# 💻 Especificaciones del equipo

| Componente | Valor |
|------------|-------|
| Sistema Operativo Host | Windows 11 Pro |
| Virtualizador | VMware Workstation Pro 26H1 |
| Sistema Invitado | Kali Linux 2026.2 |
| Procesador | AMD Ryzen 5 3400G |
| Memoria RAM | 16 GB |
| RAM asignada a la VM | 4 GB |
| Procesadores virtuales | 4 |
| Disco virtual | 80 GB |
| Tipo de red | NAT |

---

# 🛠️ Herramientas utilizadas

- VMware Workstation Pro
- Kali Linux
- Git
- GitHub
- Visual Studio Code
- Terminal Bash

---

# ⚙️ Procedimiento realizado

## 1. Instalación de VMware

Se instaló VMware Workstation Pro como plataforma de virtualización para alojar el laboratorio de ciberseguridad.

---

## 2. Implementación de Kali Linux

Se importó la máquina virtual oficial de Kali Linux y se configuraron los recursos de hardware necesarios para su funcionamiento.

---

## 3. Configuración de la máquina virtual

Se configuró la máquina virtual con:

- 4 GB de memoria RAM.
- 4 procesadores virtuales.
- Disco virtual de 80 GB.
- Adaptador de red NAT.

---

## 4. Actualización del sistema

Se ejecutaron las tareas de actualización del sistema operativo para instalar las últimas versiones disponibles de los paquetes.

---

## 5. Verificación de conectividad

Se comprobó el funcionamiento de la red mediante pruebas de conectividad y consulta de la configuración IP.

---

## 6. Configuración de Git

Se configuró Git para el control de versiones del portafolio y se vinculó el repositorio local con GitHub.

---

## 7. Instalación de Visual Studio Code

Se instaló Visual Studio Code como entorno de desarrollo para la documentación y automatización de los laboratorios.

---

## 8. Snapshot Base

Se creó un Snapshot Base con el objetivo de disponer de un punto de restauración estable antes de comenzar los siguientes laboratorios.

---

# 📸 Evidencias

Las capturas del laboratorio se encuentran en:

```text
Screenshots/
```

Entre ellas:

- kali-first-desktop.png
- apt-update-output.png
- apt-full-upgrade-output.png
- apt-autoremove-output.png
- ip-address-output.png
- ping-google-output.png
- whoami-output.png
- cat-release-output.png
- vmware-base-snapshot.png

---

# 📊 Resultados

Al finalizar el laboratorio se obtuvo un entorno completamente funcional con acceso a Internet, actualizado, documentado y preparado para el desarrollo de prácticas de ciberseguridad.

---

# 🔍 Hallazgos

- La configuración NAT permitió acceso estable a Internet.
- VMware reconoció correctamente el hardware virtual.
- Git quedó integrado con GitHub.
- Visual Studio Code quedó configurado como editor principal.
- El Snapshot Base permite restaurar rápidamente el entorno.

---

# 📚 Lecciones aprendidas

- La preparación adecuada del laboratorio reduce problemas durante el desarrollo de futuras prácticas.
- La documentación técnica facilita la reproducibilidad del entorno.
- El uso de Git permite mantener un historial completo de cambios.
- La creación de snapshots evita reinstalaciones innecesarias.

---

# 🚀 Próximos pasos

El siguiente laboratorio estará enfocado en el reconocimiento de redes mediante Nmap y la identificación de servicios expuestos.

---

# 📖 Referencias

- https://www.kali.org/
- https://www.vmware.com/
- https://git-scm.com/
- https://code.visualstudio.com/