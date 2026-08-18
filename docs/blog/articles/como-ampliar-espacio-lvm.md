---
title: "Cómo ampliar el espacio de disco en LVM sin despeinarte"
description: "Amplía el espacio de disco en LVM en caliente, sin reiniciar ni desmontar nada: guía paso a paso con pvcreate, lvextend, resize2fs y xfs_growfs."
date: 2026-08-18
authors: [Visin]
categories:
  - Administración de sistemas
  - Linux
tags:
  - LVM
  - Linux
  - disco
  - almacenamiento
  - ext4
  - XFS
  - administración de sistemas
  - particiones
draft: false
---

# Cómo ampliar el espacio de disco en LVM sin despeinarte

Llega el día. El día en que `df -h` te mira con un 100% de uso, el monitor del servidor está en rojo y el móvil no para de sonar con alertas. Respira. Si montaste LVM en su día, esto tiene solución en caliente: sin reiniciar, sin desmontar nada y, con un poco de suerte, sin que nadie se entere de que el servidor ha estado a punto de quedarse sin sitio.

<!-- more -->

## ¿Qué es esto de LVM y por qué te va a salvar el pellejo?

LVM (Logical Volume Manager) es un intermediario entre tus discos duros y tus sistemas de archivos. En lugar de formatear directamente la partición y vivir con ese tamaño para siempre, LVM mete una capa de abstracción con tres conceptos que te tienes que aprender de memoria (son pocos, no te asustes):

- **Physical Volume (PV)**: el disco duro o partición que le "donas" a LVM. Es la materia prima.
- **Volume Group (VG)**: el saco donde va toda esa materia prima. Imagina un almacén donde echas discos y de donde luego sacas espacio.
- **Logical Volume (LV)**: los "discos virtuales" que creas dentro del VG. Aquí es donde montas tu filesystem (ext4, XFS, lo que sea).

La idea es parecida a la de un trastero compartido: en lugar de tener cuatro cajas cada una con su tamaño fijo, tienes un almacén común y vas sacando espacio según lo necesitas.

Y aquí está la joya de la corona: **puedes ampliar un LV en caliente**, sin desmontar el filesystem y sin reiniciar el equipo. El usuario ni se entera. Eso sí, el usuario notará que el disco está más grande, que tampoco es mala noticia.

## Primero, mira qué tienes: `pvs`, `vgs`, `lvs` y `lsblk`

Antes de tocar nada, hay que saber dónde estamos parados. Cuatro comandos y ya tienes el mapa completo:

```bash
lsblk
```

`lsblk` te da la vista general: todos los discos, sus particiones y en qué punto de montaje están. Es el "¿qué hay por aquí?" de toda la vida.

```bash
sudo pvs
```

`pvs` te lista los **Physical Volumes**: qué discos o particiones le has dado a LVM y qué tamaño tienen. Se ven bien, con su nombre, su VG y su tamaño.

```bash
sudo vgs
```

`vgs` te muestra los **Volume Groups**: cuánto espacio tiene tu saco, cuánto está en uso y, muy importante, cuánto queda **libre**. Ese número es el que te va a dar de comer más adelante.

```bash
sudo lvs
```

`lvs` te lista los **Logical Volumes**: los discos virtuales, su tamaño y a qué VG pertenecen.

Tómate un minuto para mirar la salida. Anota el nombre de tu VG y de tu LV. Los vas a necesitar para todo lo que viene.

## Escenario A: le añades un disco nuevo al servidor

Imagina que el servidor tiene un hueco libre en el chasis, has conectado un disco nuevo (o el proveedor te ha asignado uno extra en la nube) y quieres usarlo para dar aire a un LV que está al borde del colapso.

### 1. Prepara el disco con `fdisk` (o con `parted`, si eres de los modernos)

Con `fdisk`:

```bash
sudo fdisk /dev/sdX
```

Dentro de la utilidad interactiva:

- `n` → nueva partición (acepta los valores por defecto si quieres usar todo el disco).
- `t` → cambia el tipo de partición.
- El tipo a elegir depende de tu tabla de particiones: si tu tabla es MBR, escribe `8e`; si es GPT, elige **"Linux LVM"** en la lista de tipos de partición. En cualquier caso, es la marca que identifica la partición como LVM para el resto del sistema.
- `w` → escribe los cambios y sal.

Si prefieres `parted`, la cosa es más directa:

```bash
sudo parted /dev/sdX mklabel gpt
sudo parted /dev/sdX mkpart primary 1MiB 100%
sudo parted /dev/sdX set 1 lvm on
```

Como ves: tabla de particiones GPT, una partición que ocupa todo el disco y la marca `lvm` activada. Tres líneas y listo.

### 2. Dónalo a LVM: `pvcreate`

Ahora convertimos esa partición en un Physical Volume:

```bash
sudo pvcreate /dev/sdX1
```

Este comando escribe la cabecera de LVM en la partición. A partir de este momento, esa partición es de LVM. Para siempre. Bueno, para lo que quede de vida útil del disco.

### 3. Échalo al saco: `vgextend` (o `vgcreate` si el saco no existe)

Si tu VG ya existe (lo normal), lo ampliamos:

```bash
sudo vgextend <vg> /dev/sdX1
```

Si por lo que sea estás empezando de cero, creas el VG directamente:

```bash
sudo vgcreate <vg> /dev/sdX1
```

A partir de aquí, ese espacio ya forma parte de tu grupo de volúmenes. Puedes comprobarlo con `vgs` y verás que la cifra de espacio libre ha subido.

### 4. El momento mágico: `lvextend` (y el dichoso signo `+`)

Aquí es donde se produce la magia. Extiendes el LV. Tienes dos opciones:

**Darle un tamaño concreto:**

```bash
sudo lvextend -L +10G /dev/<vg>/<lv>
```

**Darle todo el espacio libre del VG:**

```bash
sudo lvextend -l +100%FREE /dev/<vg>/<lv>
```

Y ahora, ojo, que esto es de los errores que más veces he visto en producción: **el signo `+` es obligatorio**.

`lvextend -l 100%FREE` (sin el `+`) no le dice "añade todo el espacio libre". Le dice "deja el LV exactamente con ese tamaño". Como el LV ya es más grande que eso, el comando se niega en redondo y te escupe algo así:

```
New size given not larger than existing size
```

Traducción: "¿pero tú me estás vacilando?" La diferencia entre `+10G` (añade 10 gigas) y `10G` (pon el LV a 10 gigas) es la diferencia entre un día tranquilo y una tarde de soporte. Que no te pase.

### 5. Dile al filesystem que ahora tiene más sitio

Ojo, que aquí no termina la cosa. El LV es más grande, pero el filesystem que hay dentro **sigue teniendo el tamaño antiguo**. Hay que avisarle. Y aquí la cosa se divide según tu filesystem:

**Si usas ext4** (el clásico de toda la vida):

```bash
sudo resize2fs /dev/<vg>/<lv>
```

La maravilla es que funciona **con el filesystem montado**. Sí, has leído bien. Sin desmontar nada, en caliente.

**Si usas XFS**:

```bash
sudo xfs_growfs /<mountpoint>
```

Fíjate en la diferencia: con XFS le pasas el **punto de montaje**, no el dispositivo. Y XFS sí exige que el filesystem esté **montado**. Si lo lanzas a lo loco, te responde con un rotundo:

```
is not a mounted XFS filesystem
```

Que es la forma que tiene XFS de decirte "¿de qué vas, colega?"

### 6. Verificación final: que todo cuadre

Tres comprobaciones rápidas y te vas contento a casa:

```bash
df -h
df -hT /<mountpoint>
```

`df -h` te muestra el tamaño nuevo del filesystem. `df -hT` además te dice el tipo de filesystem, por si acaso.

Y para confirmar que LVM está contento:

```bash
sudo lvs
sudo vgs
sudo pvs
```

Si el LV, el VG y el filesystem coinciden en tamaño, misión cumplida.

## Escenario B: amplías el disco de la VM y que Linux se entere

Este es el caso típico de la nube: vas al panel de tu proveedor, le dices a la máquina virtual "a partir de ahora tienes 100 G en vez de 50 G" y... el sistema operativo se queda tan pancho, sin enterarse. Porque el disco físico ha crecido, pero nadie le ha dicho al kernel que vuelva a mirar. Vamos paso a paso.

### 1. El rescan: despierta al kernel

El primer paso es hacer que el kernel detecte el nuevo tamaño del disco virtual:

```bash
sudo sh -c 'echo 1 > /sys/class/scsi_device/<id>/device/rescan'
# alternativas:
sudo sh -c 'echo 1 > /sys/class/scsi_disk/<id>/device/rescan'
sudo sh -c 'echo 1 > /sys/class/block/sda/device/rescan'
sudo sh -c 'echo "- - -" > /sys/class/scsi_host/host0/scan'
```

El `<id>` es la ruta del dispositivo SCSI, que puedes sacar de `lsblk` o `lsscsi`. Hay varias alternativas por si una no te funciona. Eso sí, fíjate en que todos van con `sudo`: escriben directamente en `/sys`, que no es territorio de simples mortales (por eso van con `sh -c`, para que el redireccionamiento también se ejecute como root).

Las tres primeras hacen lo mismo: le dicen al kernel que vuelva a mirar un dispositivo concreto del bus SCSI, porque ha cambiado algo. La última, la de `host0/scan`, es la escoba gorda: escanea **todo** el bus. Normalmente con una de ellas basta, pero si estás en una VM de esas mañosas, prueba varias hasta que `lsblk` te muestre el tamaño nuevo.

### 2. Agranda la partición: `partprobe` y `growpart`

Ahora el disco entero es más grande, pero la partición sigue con el tamaño antiguo. Primero avisamos al kernel de que la tabla de particiones ha cambiado:

```bash
sudo partprobe /dev/sdX
```

Y luego agrandamos la partición para que ocupe todo el disco. Si el PV está en una partición (lo más típico), usa `growpart`:

```bash
sudo growpart /dev/sda 1
```

Es decir: "crece la partición 1 del disco `/dev/sda`". Si `growpart` no lo tienes instalado o no te funciona, con `parted` también se puede:

```bash
sudo parted /dev/sda resizepart 1 100%
```

"Redimensiona la partición 1 hasta el 100% del disco".

### 3. Actualiza el PV: `pvresize`

El disco y la partición ya son grandes, pero el Physical Volume sigue pensando que mide lo de antes. Se lo dices con:

```bash
sudo pvresize /dev/sdX
```

Y si tu PV está en una partición concreta:

```bash
sudo pvresize /dev/sdX1
```

`pvresize` detecta el tamaño real de la partición y actualiza el PV. Compruébalo con `pvs` y verás que el tamaño ha subido. Por lo tanto, el espacio libre del VG también ha subido.

### 4. Extiende el LV (otra vez el `+`, no lo olvides)

Ahora sí, la jugada de siempre:

```bash
sudo lvextend -l +100%FREE /dev/<vg>/<lv>
```

Todo el espacio libre del VG, directo al LV. Y sí, con el `+`, que ya sabemos lo que pasa sin él.

### 5. Y el filesystem, para rematar

El mismo baile de antes:

```bash
sudo resize2fs /dev/<vg>/<lv>   # si es ext4
```

o, si eres de XFS:

```bash
sudo xfs_growfs /<mountpoint>   # con el filesystem montado, ¿eh?
```

### 6. Verificación

```bash
df -h
df -hT /<mountpoint>
sudo pvs && sudo vgs && sudo lvs
```

Todo en su sitio, todo en orden, y el panel de la nube y la realidad por fin de acuerdo.

## Consejos y avisos (léelos antes de que sea tarde)

Vale, ya tienes el mecanismo aprendido. Ahora los avisos importantes, esos que aprendes a base de sustos:

**XFS solo se agranda, nunca se reduce.** Como dice la documentación oficial de Red Hat, "no es posible actualmente reducir el tamaño de los filesystems XFS". Y no, no es un "todavía". Es que la arquitectura de XFS no lo permite. Si necesitas reducir un XFS, la única vía es respaldar, recrear el filesystem con el tamaño que quieras y restaurar. Doloroso, pero es lo que hay.

**ext4 sí se puede reducir, pero con cuidado y en orden.** Primero hay que desmontar el filesystem. Luego se reduce primero el filesystem y después el LV. El orden es sagrado:

```bash
sudo umount /dev/<vg>/<lv>
sudo resize2fs /dev/<vg>/<lv> SIZE
sudo lvreduce -L SIZE /dev/<vg>/<lv>
```

Si lo haces al revés (primero el LV y luego el filesystem), te cargas el filesystem. Sin aviso previo. No es una amenaza, es una certeza.

**Para crecer, el orden es: PV → LV → filesystem.** Primero que el espacio esté en el saco (PV), luego sácalo al LV y por último avisa al filesystem. Si te saltas un paso, verás cómo las cosas no cuadran.

**El atajo: `lvextend -r`.** Si no te apetece lanzar `resize2fs` o `xfs_growfs` a mano, LVM puede hacerlo por ti con la opción `--resizefs`:

```bash
sudo lvextend -l+100%FREE -r vg01/lvol01
```

La `-r` se encarga de redimensionar también el filesystem de forma automática (vía `fsadm`), detectando el tipo y haciendo lo correcto. Un solo comando y a otra cosa.

**Respaldar antes de tocar nada.** Suena a sermón, pero es que Red Hat lo dice con todas las letras: "este es un primer paso no negociable en un entorno de producción". Y tienen toda la razón. Las ampliaciones suelen ir bien, pero cuando van mal, van muy mal. Un respaldo te convierte una noche de pánico en un susto de cinco minutos.

**Si recreas una partición con `fdisk`, usa el mismo cilindro de inicio.** Suena a cosa de abuelos, pero es la advertencia literal del man de `resize2fs`: si borras y recreas una partición y le das un punto de inicio distinto, puedes perder todo el filesystem. La tabla de particiones no perdona. Apunta el valor de inicio antes de borrar nada.

**`xfs_growfs` no funciona con el filesystem desmontado.** Ya lo hemos visto: "is not a mounted XFS filesystem". XFS quiere estar montado para crecer. No es capricho, es cómo funciona.

**Y el último consejo, el bueno:** mantén siempre un poco de espacio libre en el VG. Da igual que sea un 5% o unos cuantos gigas, pero que quede margen. Porque cuando el disco se llena a las tres de la mañana, tener espacio libre en el saco te permite ampliar el LV en dos comandos en lugar de andar metiendo discos nuevos a deshoras. Es tu colchón de emergencias.

## Conclusión

LVM es de esas herramientas que al principio dan un poco de pereza aprender, pero que cuando la necesitas, la necesitas de verdad. La secuencia completa es casi un mantra: disco nuevo → `pvcreate` → `vgextend` → `lvextend` (con su `+`) → `resize2fs` o `xfs_growfs`. Un puñado de comandos bien entendidos y puedes ampliar el espacio en caliente, sin reiniciar y sin desmontar nada.

Y recuerda: el `+` de `lvextend`, el orden PV → LV → filesystem, y el respaldo antes de tocar nada. Con eso, el día que el disco se llene, lo amplías en cinco minutos y todos contentos. Hasta el siguiente disco lleno, claro.
