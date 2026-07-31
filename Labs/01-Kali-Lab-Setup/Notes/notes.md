# Notes

## Objetivo

Registrar las observaciones, lecciones aprendidas y recomendaciones obtenidas durante la preparación del entorno de trabajo.

---

# Observaciones

- Se decidió utilizar **VMware Workstation Pro** como plataforma de virtualización principal por su estabilidad, facilidad para gestionar snapshots y buen rendimiento con Kali Linux.
- Se configuró la red en modo **NAT**, permitiendo acceso a Internet sin exponer directamente la máquina virtual a la red local.
- Todo el desarrollo y la documentación del portafolio se realizará desde **Kali Linux**, utilizando Windows únicamente como sistema anfitrión (Host).

---

# Problemas encontrados

## 1. Configuración de la resolución de pantalla

Inicialmente, la resolución de Kali Linux no se ajustaba correctamente al tamaño de la ventana de VMware.

**Solución aplicada:**

- Se instalaron y verificaron las VMware Tools.
- Se ajustó automáticamente la resolución de pantalla desde VMware.

---

## 2. Organización del proyecto

Al inicio, el repositorio se creó en Windows. Posteriormente se decidió migrarlo completamente a Kali Linux para mantener un entorno de trabajo más profesional y similar al utilizado en equipos de ciberseguridad.

---

# Lecciones aprendidas

Durante este laboratorio comprendí la importancia de preparar correctamente el entorno antes de comenzar cualquier práctica de ciberseguridad. Una configuración adecuada evita problemas futuros y facilita el desarrollo de los laboratorios.

También reforcé conocimientos sobre:

- Virtualización con VMware.
- Administración básica de Kali Linux.
- Gestión de proyectos con Git.
- Control de versiones mediante GitHub.
- Organización de documentación técnica en Markdown.

---

# Buenas prácticas aplicadas

- Mantener una estructura de carpetas organizada.
- Documentar cada procedimiento realizado.
- Nombrar archivos de forma consistente y descriptiva.
- Crear un Snapshot Base antes de modificar el entorno.
- Versionar todos los cambios mediante Git.

---

# Recomendaciones

Antes de iniciar nuevos laboratorios se recomienda:

- Verificar la conectividad a Internet.
- Confirmar que la máquina virtual funcione correctamente.
- Crear snapshots antes de realizar cambios importantes.
- Mantener el sistema operativo actualizado.
- Documentar todas las actividades y resultados obtenidos.

---

# Conclusión

El Laboratorio 1 permitió construir una base sólida para el desarrollo del portafolio de ciberseguridad. El entorno quedó completamente operativo, documentado y preparado para los siguientes laboratorios, facilitando la reproducibilidad de las prácticas y el seguimiento de la evolución del proyecto.