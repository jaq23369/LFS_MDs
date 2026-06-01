# LFS 12.4 — Contexto para Claude Code

## INSTRUCCIONES CRÍTICAS PARA CLAUDE CODE

1. **NUNCA ejecutes comandos de forma autónoma** — siempre muestra el comando y espera confirmación explícita del usuario antes de ejecutar.
2. **Siempre muestra el output completo** de cada comando para que el usuario pueda validar.
3. **Ante cualquier error o output inesperado**, detente y explica qué salió mal antes de continuar.
4. **Un comando a la vez** para pasos críticos (compilación, instalación, configuración de sistema).
5. **Nunca borres archivos** sin confirmación explícita.
6. **Si algo se ve raro**, pregunta antes de proceder.
7. **No estás dentro del chroot todavía** — sigue el flujo de entrada descrito abajo.

---

## Setup General

| Campo | Valor |
|---|---|
| Proyecto | Linux From Scratch 12.4 |
| Curso | CC3064 Sistemas Operativos — UVG |
| Host | MacBook Pro M3 Max (Apple Silicon, macOS) |
| Hipervisor | UTM con Hypervisor.framework |
| VM | Ubuntu 22.04.5 LTS aarch64 |
| Arquitectura | **aarch64 (ARM64)** — NO es x86_64 |
| IP de la VM | 192.168.64.4 |
| Usuario VM | jjaquez |

---

## Variables Críticas

```bash
LFS=/mnt/lfs
LFS_TGT=aarch64-lfs-linux-gnu
MAKEFLAGS=-j4
```

---

## Disco LFS

| Campo | Valor |
|---|---|
| Dispositivo | `/dev/mapper/ubuntu--vg-lfs--lv` |
| Tipo | LVM — ext4 |
| Tamaño | 20GB |
| Punto de montaje | `/mnt/lfs` |

---

## Cómo entrar al entorno LFS (hacer esto SIEMPRE al iniciar sesión)

### Paso 1 — Ser root y verificar LFS

```bash
sudo -i
echo $LFS   # debe imprimir /mnt/lfs
# Si no imprime nada:
export LFS=/mnt/lfs
```

### Paso 2 — Verificar si los mounts están activos

```bash
findmnt | grep lfs
```

Si NO aparecen los mounts, ejecutar el Paso 3. Si ya están, saltar al Paso 4.

### Paso 3 — Montar los virtual filesystems (solo si no están montados)

```bash
mount -v --bind /dev $LFS/dev
mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
if [ -h $LFS/dev/shm ]; then
   install -v -d -m 1777 $LFS$(realpath /dev/shm)
else
   mount -vt tmpfs -o nosuid,nodev tmpfs $LFS/dev/shm
fi
```

### Paso 4 — Entrar al chroot

```bash
chroot "$LFS" /usr/bin/env -i \
    HOME=/root \
    TERM="$TERM" \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin \
    MAKEFLAGS="-j4" \
    TESTSUITEFLAGS="-j4" \
    /bin/bash --login
```

El prompt debe verse así: `(lfs chroot) root:/#`

---

## Estado Actual — Lo que ya está completado

### ✅ Capítulos 1–6 — Completados en sesión anterior
- Cross-toolchain y herramientas temporales construidas
- Todos los tests del profesor pasados (test02 falla por ser aarch64, es esperado)
- Backup guardado en `/root/lfs-temp-tools-12.4.tar.xz`

### ✅ Capítulo 7 — Completado
- 7.2: Ownership cambiado de `lfs` a `root`
- 7.3: Virtual filesystems montados (dev, devpts, proc, sysfs, tmpfs, shm)
- 7.4: Entorno chroot funcional
- 7.5: Estructura de directorios completa creada
- 7.6: Archivos esenciales creados (mtab, hosts, passwd, group, logfiles)
  - Prompt cambió de `I have no name!` a `root` ✅
- 7.7: Gettext instalado ✅
- 7.8: Bison instalado ✅
- 7.9: Perl instalado ✅
- 7.10: Python 3.13.7 instalado ✅
- 7.11: Texinfo instalado ✅
- 7.12: Util-linux instalado ✅
- 7.13: Limpieza hecha, backup creado (1.1GB en `/root/`)

### ✅ Capítulo 8 — EN PROGRESO

| Sección | Paquete | Estado |
|---|---|---|
| 8.3 | Man-pages-6.15 | ✅ |
| 8.4 | Iana-Etc-20250807 | ✅ |
| 8.5 | Glibc-2.42 | ✅ (con config de locales, timezone Guatemala, ld.so.conf) |
| 8.6 | Zlib-1.3.1 | ⬅️ **AQUÍ EMPEZAMOS** |

---

## Dónde continuar

**Dentro del chroot**, en `/sources`, instalar desde **8.6 — Zlib** en adelante.

```bash
# Verificar que estás en el lugar correcto
pwd   # debe ser /sources o puedes hacer cd /sources
ls zlib*  # debe mostrar zlib-1.3.1.tar.xz
```

---

## Diferencias aarch64 vs x86_64 (outputs normales en nuestra arquitectura)

| Elemento | x86_64 (libro) | aarch64 (nuestro) |
|---|---|---|
| Dynamic linker | `ld-linux-x86-64.so.2` | `ld-linux-aarch64.so.1` |
| Libstdc++ | `$LFS/usr/lib/` | `$LFS/usr/lib64/` |
| LFS_TGT | `x86_64-lfs-linux-gnu` | `aarch64-lfs-linux-gnu` |

---

## Notas importantes

- **Dentro del chroot no existe `sudo`** — ya eres root directamente
- **`MAKEFLAGS=-j4`** — ya está seteado en el chroot
- **El libro muestra outputs x86_64** — los outputs aarch64 son distintos pero correctos
- **Timezone seteada**: `America/Guatemala`
- **`/usr/lib64` NO debe existir** según FHS — si aparece, es un problema
- **Error de `test-installation.pl` en Glibc** (`libnsl`, `libnss_dns not found`) es un falso positivo conocido en aarch64 — no es error real
- **FAILs conocidos en Glibc `make check`**: `io/tst-lchmod` y `nptl/tst-mutex10` — normales en chroot/VM
- Los directorios compilados en `/sources` se pueden borrar después de `make install` — los tarballs `.tar.xz` NO se borran

---

## Patrón estándar de instalación de paquetes (Cap. 8)

La mayoría de paquetes siguen este patrón:

```bash
cd /sources
tar -xf <paquete>.tar.xz   # o .tar.gz según el caso
cd <paquete>
./configure --prefix=/usr [opciones específicas]
make
make check   # opcional según el libro
make install
cd /sources
rm -rf <paquete>
```

Excepciones notables que tienen pasos extra o distintos:
- **Bzip2** — usa `Makefile-libbz2_so` especial
- **GCC** — el más largo (~10 SBU), múltiples passes y verificaciones
- **Shadow** — configuración de seguridad post-instalación
- **Bash** — hay que reiniciar el shell después
- **Glibc** — ya completado, fue el más complejo

---

## Backup disponible

Si algo sale muy mal, hay un backup completo del sistema hasta antes del Cap. 8:

```bash
# SOLO como último recurso, fuera del chroot como root:
cd $LFS
rm -rf ./*
tar -xpf /root/lfs-temp-tools-12.4.tar.xz
```
