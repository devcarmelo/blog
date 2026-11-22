---
title: Cómo crear una ISO Live de Debian 13 con XFCE usando live-build
author: Carmelo
pubDatetime: 2026-11-18T13:58:00Z
featured: false
draft: false
tags:
  - Debian
  - Linux
  - tutorial
description: Aprende a preparar un sistema Live de Debian Trixie con XFCE, firmware y aplicaciones, y a generar una ISO híbrida con live-build.
---

En este tutorial vamos a preparar una imagen Live de Debian 13 (Trixie) con el escritorio XFCE, firmware para varios dispositivos y un conjunto de herramientas y aplicaciones. Usaremos `live-build` para configurar el sistema y construir una ISO que se pueda probar en una máquina virtual o grabar en un USB.

El resultado será una imagen de prueba personalizable. En esta configuración, el usuario de la sesión Live será `debian`; para iniciar sesión, usa la contraseña predeterminada de Debian Live: `live`.

## 1. Preparar el entorno

Necesitas un sistema Debian o derivado con acceso a Internet y espacio libre para descargar los paquetes y construir la imagen. Instala las herramientas necesarias:

```bash
sudo apt update
sudo apt install -y live-build xorriso squashfs-tools git wget curl
```

La construcción debe ejecutarse desde el directorio de trabajo del proyecto. Créalo y entra en él:

```bash
mkdir debian-live-trixie
cd debian-live-trixie
```

Los siguientes pasos crean un script de preparación que puedes conservar y volver a ejecutar para reconstruir la configuración.

## 2. Crear el script de preparación

Crea un archivo llamado `prepare-live.sh`:

```bash
vim prepare-live.sh
```

Pega este contenido:

```sh
#!/bin/sh
set -e

echo "========================================================"
echo " [1/3] Configurando live-build (auto/config)..."
echo "========================================================"
mkdir -p auto
cat << 'EOF' > auto/config
#!/bin/sh
set -e

lb config noauto \
    --distribution trixie \
    --architectures amd64 \
    --binary-images iso-hybrid \
    --archive-areas "main contrib non-free non-free-firmware" \
    --apt-recommends true \
    --debian-installer live \
    --mirror-bootstrap http://deb.debian.org/debian/ \
    --mirror-chroot-security http://security.debian.org/debian-security \
    --mode debian \
    --bootappend-live "boot=live components quiet splash locales=es_ES.UTF-8 keyboard-layouts=es timezone=Europe/Madrid username=debian user-fullname='Usuario Live' hostname=debian-trixie"
EOF
chmod +x auto/config
echo "-> Archivo auto/config generado y listo."

echo ""
echo "========================================================"
echo " [2/3] Creando directorio de paquetes..."
echo "========================================================"
mkdir -p config/package-lists
echo "-> Directorio config/package-lists creado."

echo ""
echo "========================================================"
echo " [3/3] Generando listados de paquetes..."
echo "========================================================"

echo "-> Escribiendo entorno gráfico (desktop.list.chroot)..."
cat > config/package-lists/desktop.list.chroot << 'EOF'
task-xfce-desktop
lightdm
lightdm-gtk-greeter
network-manager
network-manager-gnome
pulseaudio
pavucontrol
xfce4-goodies
EOF

echo "  -> Escribiendo drivers y firmware (firmware.list.chroot)..."
cat > config/package-lists/firmware.list.chroot << 'EOF'
firmware-amd-graphics
amd64-microcode
firmware-misc-nonfree
firmware-realtek
firmware-iwlwifi
EOF

echo "  -> Escribiendo herramientas y apps (apps.list.chroot)..."
cat > config/package-lists/apps.list.chroot << 'EOF'
sudo
vim
git
htop
btop
nvtop
fastfetch
gparted
gnome-disk-utility
xfce4-terminal
thunar-archive-plugin
zip
unzip
p7zip-full
vlc
firefox-esr
curl
wget
EOF

echo ""
echo "========================================================"
echo " ¡PROCESO COMPLETADO CON ÉXITO!"
echo "========================================================"
echo "Ya tienes todo listo en tu directorio de trabajo."
echo "Para limpiar cualquier construcción anterior, configurar live-build y construir la ISO, ejecuta:"
echo "  sudo lb clean --purge"
echo "  sudo ./auto/config"
echo "  sudo lb build"
echo "========================================================"
```

Guarda el archivo y hazlo ejecutable:

```bash
chmod +x prepare-live.sh
```

La sesión Live usa el usuario `debian` y la contraseña predeterminada de Debian Live: `live`.

## 3. Generar la configuración del sistema Live

Ejecuta el script desde la carpeta del proyecto:

```bash
./prepare-live.sh
```

Se crearán estos archivos y directorios:

```text
auto/config
config/package-lists/desktop.list.chroot
config/package-lists/firmware.list.chroot
config/package-lists/apps.list.chroot
```

`auto/config` guarda las opciones de `live-build`. Los archivos `.list.chroot` contienen un paquete por línea; durante la construcción, `live-build` instalará esos paquetes dentro del sistema Live.

## 4. Entender las opciones principales

La configuración selecciona Debian Trixie y la arquitectura de PC de 64 bits (`amd64`). `iso-hybrid` genera una imagen que se puede usar tanto en una máquina virtual como en un medio USB. Los repositorios incluyen `contrib`, `non-free` y `non-free-firmware` para poder instalar firmware y paquetes que no están en `main`.

El parámetro `--debian-installer live` incluye el instalador de Debian en el medio. Las opciones de `--bootappend-live` seleccionan el arranque Live, el idioma español, la distribución de teclado, la zona horaria, el nombre de usuario y el nombre del equipo.

Los listados de paquetes separan el contenido por función:

- **`desktop.list.chroot`** instala XFCE, LightDM, NetworkManager y herramientas de audio.
- **`firmware.list.chroot`** añade firmware y microcódigo para algunos componentes AMD, Realtek e Intel Wi-Fi.
- **`apps.list.chroot`** instala utilidades de administración, archivado, multimedia y navegación.

Puedes editar esos listados antes de construir la ISO: elimina los paquetes que no necesites o añade otros disponibles en los repositorios configurados.

## 5. Construir la ISO

Cuando hayas revisado la configuración, limpia los restos de cualquier construcción anterior, vuelve a ejecutar la configuración y construye la ISO:

```bash
sudo lb clean --purge
sudo ./auto/config
sudo lb build
```

El proceso descarga paquetes y puede tardar un tiempo, dependiendo de la conexión y del equipo. Si termina correctamente, encontrarás en el directorio del proyecto un archivo con extensión `.iso`. Conserva también el directorio de trabajo si piensas modificar la configuración y reconstruir la imagen.

Si la construcción falla, revisa el primer error que aparece en la salida. Entre las causas habituales están una conexión interrumpida, falta de espacio, un paquete que ya no existe en los repositorios o una configuración incompatible con la versión de `live-build`.

## 6. Probar y usar la imagen

Antes de grabar la ISO en un USB, pruébala en una máquina virtual. Comprueba que inicia en modo Live, que el escritorio XFCE carga y que funcionan la red, el teclado y los dispositivos que quieras usar.

Para crear un USB de arranque, utiliza una herramienta de grabación de imágenes y selecciona el archivo `.iso` generado. La escritura sobrescribe el contenido del USB: verifica cuidadosamente la unidad seleccionada y guarda antes cualquier archivo que quieras conservar.

## Conclusión

Con `live-build` puedes mantener en archivos de texto la selección de paquetes y las opciones de arranque de un Debian Live. Este ejemplo deja una base con Trixie, XFCE, firmware y aplicaciones habituales; desde aquí puedes adaptarla a tu hardware y a las necesidades de tu proyecto. Antes de publicar o compartir la ISO, revisa especialmente las credenciales y prueba la imagen de principio a fin.
