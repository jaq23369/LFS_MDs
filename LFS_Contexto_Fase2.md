# LFS 12.4 — Contexto del Proyecto para Fase 2 (Cap. 7–11)

## Información General

| Campo | Valor |
|---|---|
| Proyecto | Linux From Scratch 12.4 |
| Curso | CC3064 Sistemas Operativos — UVG |
| Hardware host | MacBook Pro M3 Max (Apple Silicon, macOS) |
| Hipervisor | UTM con Hypervisor.framework |
| VM | Ubuntu 22.04.5 LTS aarch64 |
| Arquitectura | **aarch64 (ARM64)** — NO es x86_64 |
| IP de la VM | 192.168.64.4 |

---

## Conexión a la VM

```bash
# Desde la terminal del Mac
ssh jjaquez@192.168.64.4
# Contraseña: la del usuario jjaquez en Ubuntu
```

---

## Credenciales y Usuarios

| Usuario | Contraseña | Propósito |
|---|---|---|
| `jjaquez` | (contraseña Ubuntu normal) | Usuario principal del host |
| `lfs` | `lfs123` | Usuario para construir LFS (Cap. 5-6) |
| `root` | via sudo | Para operaciones privilegiadas |

---

## Variables de Ambiente Críticas

```bash
LFS=/mnt/lfs
LFS_TGT=aarch64-lfs-linux-gnu
MAKEFLAGS=-j4
```

Estas variables están configuradas en:
- `/home/jjaquez/.bash_profile`
- `/home/lfs/.bash_profile` y `/home/lfs/.bashrc`

---

## Disco y Partición LFS

| Campo | Valor |
|---|---|
| Dispositivo | `/dev/mapper/ubuntu--vg-lfs--lv` |
| Tipo | LVM Logical Volume |
| Tamaño | 20GB |
| Sistema de archivos | ext4 |
| Punto de montaje | `/mnt/lfs` |
| Automount | Sí, configurado en `/etc/fstab` |

Verificar que esté montado:
```bash
echo $LFS && mount | grep lfs
```

---

## Estado Actual — Capítulos Completados

### ✅ Capítulo 1 — Introducción
- Lectura y comprensión del libro LFS 12.4

### ✅ Capítulo 2 — Preparación del Host
- `/bin/sh` redirigido a bash (`dpkg-reconfigure dash` → No)
- version-check.sh pasó todos los requisitos
- Partición LFS creada como LVM de 20GB, ext4, montada en `/mnt/lfs`
- `$LFS=/mnt/lfs` y `umask 022` en `.bash_profile` de jjaquez y root
- `/etc/fstab` actualizado para automount

### ✅ Capítulo 3 — Paquetes y Parches
- `$LFS/sources` creado con sticky bit (drwxrwxrwt)
- 87 paquetes + 7 parches descargados
- ncurses descargado desde mirror OSUOSL (URL oficial falló)
- `md5sum -c md5sums` → limpio sin FAILED
- Owner actual: `lfs:root` en `$LFS/sources`

### ✅ Capítulo 4 — Preparaciones Finales
Estructura de directorios creada:
```
$LFS/
├── etc/
├── var/
├── usr/
│   ├── bin/  (symlink ← $LFS/bin)
│   ├── lib/  (symlink ← $LFS/lib)
│   └── sbin/ (symlink ← $LFS/sbin)
├── tools/
└── sources/
```
- Usuario `lfs` creado (grupo `lfs`, home `/home/lfs`)
- Ownership de `$LFS/tools` y `$LFS/usr` dado a `lfs:root`
- `/etc/bash.bashrc` movido a `.NOUSE` (por eso el prompt es `-bash-5.1$`)
- `.bash_profile` y `.bashrc` del usuario `lfs` configurados

### ✅ Capítulo 5 — Cross-Toolchain (como usuario lfs)

| Sección | Paquete | Estado |
|---|---|---|
| 5.2 | Binutils-2.45 Pass 1 | ✅ en `$LFS/tools` |
| 5.3 | GCC-15.2.0 Pass 1 | ✅ en `$LFS/tools` |
| 5.4 | Linux-6.16.1 API Headers | ✅ en `$LFS/usr/include` |
| 5.5 | Glibc-2.42 | ✅ sanity checks pasados |
| 5.6 | Libstdc++ | ✅ en `$LFS/usr/lib64/` |

**Nota aarch64:** Libstdc++ quedó en `lib64` en lugar de `lib` — normal en ARM64.

Sanity checks de Glibc pasados:
```
[Requesting program interpreter: /lib/ld-linux-aarch64.so.1]
found ld-linux-aarch64.so.1 at /mnt/lfs/usr/lib/ld-linux-aarch64.so.1
libc.so.6 → /mnt/lfs/usr/lib/libc.so.6
```

### ✅ Capítulo 6 — Herramientas Temporales Cruzadas (como usuario lfs)

| Sección | Paquete | Estado |
|---|---|---|
| 6.2 | M4-1.4.20 | ✅ |
| 6.3 | Ncurses-6.5-20250809 | ✅ |
| 6.4 | Bash-5.3 | ✅ |
| 6.5 | Coreutils-9.7 | ✅ |
| 6.6 | Diffutils-3.12 | ✅ |
| 6.7 | File-5.46 | ✅ |
| 6.8 | Findutils-4.10.0 | ✅ |
| 6.9 | Gawk-5.3.2 | ✅ |
| 6.10 | Grep-3.12 | ✅ |
| 6.11 | Gzip-1.14 | ✅ |
| 6.12 | Make-4.4.1 | ✅ |
| 6.13 | Patch-2.8 | ✅ |
| 6.14 | Sed-4.9 | ✅ |
| 6.15 | Tar-1.35 | ✅ |
| 6.16 | Xz-5.8.1 | ✅ |
| 6.17 | Binutils-2.45 Pass 2 | ✅ |
| 6.18 | GCC-15.2.0 Pass 2 | ✅ |

---

## Diferencias Importantes: aarch64 vs x86_64

Estas diferencias aparecen en los outputs y son **correctas** para nuestra arquitectura:

| Elemento | x86_64 (libro) | aarch64 (nuestro) |
|---|---|---|
| Dynamic linker | `ld-linux-x86-64.so.2` | `ld-linux-aarch64.so.1` |
| Libstdc++ | `$LFS/usr/lib/` | `$LFS/usr/lib64/` |
| LFS_TGT | `x86_64-lfs-linux-gnu` | `aarch64-lfs-linux-gnu` |
| test02 del profesor | FALLA (hardcodeado x86_64) | Esperado — no es error nuestro |

---

## Tests del Profesor — Resultados

| Test | Resultado | Nota |
|---|---|---|
| test01_environment.sh | ✅ PASS | Owner de sources corregido a lfs:root |
| test02_toolchain.sh | ⚠️ FAIL esperado | Script hardcodeado para x86_64 |
| test03_temp_tools_presence.sh | ✅ PASS | Todas las herramientas presentes |
| test04_linkage_inspection.sh | ✅ PASS | Binarios correctamente linkedos |
| test05_ch7_precheck.sh | ✅ PASS | Directorios dev/proc/sys/run creados |

---

## Archivos y Directorios Clave

```bash
$LFS/tools/bin/aarch64-lfs-linux-gnu-gcc   # Cross-compiler (Cap. 5)
$LFS/tools/bin/aarch64-lfs-linux-gnu-ld    # Cross-linker (Cap. 5)
$LFS/usr/bin/gcc                            # GCC Pass 2 (Cap. 6)
$LFS/usr/bin/cc                             # Symlink a gcc
$LFS/usr/bin/ld                             # Linker (Cap. 6)
$LFS/usr/bin/bash                           # Bash (Cap. 6)
$LFS/usr/sbin/chroot                        # Para entrar al chroot (Cap. 7)
$LFS/usr/lib/libc.so.6                      # Glibc
$LFS/usr/lib/ld-linux-aarch64.so.1         # Dynamic linker
$LFS/usr/lib64/libstdc++.so.6              # Libstdc++ (aarch64)
$LFS/sources/                               # 98 tarballs y parches
$LFS/dev/                                   # Creado para Cap. 7
$LFS/proc/                                  # Creado para Cap. 7
$LFS/sys/                                   # Creado para Cap. 7
$LFS/run/                                   # Creado para Cap. 7
```

---

## Para Iniciar la Fase 2 (Cap. 7–11)

### Paso 1 — Conectarse
```bash
ssh jjaquez@192.168.64.4
```

### Paso 2 — Verificar montaje
```bash
echo $LFS && mount | grep lfs
```

### Paso 3 — El Cap. 7 empieza como root (jjaquez con sudo)
El Capítulo 7 requiere montar sistemas de archivos virtuales y entrar al entorno chroot. Se hace como `root`, NO como usuario `lfs`.

```bash
# Montar sistemas de archivos virtuales (Cap. 7.3)
sudo mount -v --bind /dev $LFS/dev
sudo mount -vt devpts devpts -o gid=5,mode=0620 $LFS/dev/pts
sudo mount -vt proc proc $LFS/proc
sudo mount -vt sysfs sysfs $LFS/sys
sudo mount -vt tmpfs tmpfs $LFS/run

# Entrar al chroot (Cap. 7.4)
sudo chroot "$LFS" /usr/bin/env -i \
    HOME=/root \
    TERM="$TERM" \
    PS1='(lfs chroot) \u:\w\$ ' \
    PATH=/usr/bin:/usr/sbin \
    MAKEFLAGS="-j4" \
    /bin/bash --login
```

---

## Notas Importantes para Claude Code

1. **Arquitectura es aarch64** — todos los binarios son ARM64, no x86_64
2. **No tocar `$LFS/tools/`** — es el cross-toolchain del Cap. 5, ya construido
3. **Cap. 7-11 se trabaja dentro del chroot** — los comandos corren dentro del entorno LFS
4. **`MAKEFLAGS=-j4`** — siempre usar para paralelizar la compilación
5. **El libro muestra outputs x86_64** — los outputs aarch64 son diferentes pero correctos
6. **Dentro del chroot no existe `sudo`** — se trabaja directamente como root
7. **`$LFS/sources/`** tiene todos los tarballs necesarios para Cap. 8
