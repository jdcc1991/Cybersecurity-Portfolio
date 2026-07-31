# Commands Used

Este documento registra los principales comandos utilizados durante la preparación del laboratorio.

---

## Actualización del sistema

```bash
sudo apt update
```

Actualiza la lista de paquetes disponibles desde los repositorios.

```bash
sudo apt full-upgrade -y
```

Instala las versiones más recientes de los paquetes del sistema.

```bash
sudo apt autoremove -y
```

Elimina dependencias y paquetes que ya no son necesarios.

---

## Verificación del sistema

```bash
whoami
```

Muestra el usuario actualmente autenticado.

```bash
hostnamectl
```

Muestra información del sistema operativo y del equipo.

```bash
ip a
```

Muestra la configuración de las interfaces de red.

```bash
ping -c 4 google.com
```

Verifica la conectividad a Internet mediante el envío de cuatro paquetes ICMP.

```bash
cat /etc/os-release
```

Muestra la versión y la información del sistema operativo.

---

## Git

```bash
git init
```

Inicializa el repositorio Git local.

```bash
git status
```

Muestra el estado actual del repositorio.

```bash
git add .
```

Agrega todos los cambios al área de preparación.

```bash
git commit -m "chore: initialize cybersecurity portfolio structure"
```

Crea el primer commit del proyecto.

```bash
git remote add origin https://github.com/jdcc1991/Cybersecurity-Portfolio.git
```

Asocia el repositorio local con GitHub.

```bash
git push -u origin main
```

Publica el proyecto en GitHub y establece la rama remota por defecto.

```bash
git log --oneline
```

Muestra un resumen del historial de commits.

```bash
git remote -v
```

Lista los repositorios remotos configurados.