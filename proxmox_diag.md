# Diagnóstico de servidor PROXMOX

El servidor cuenta con una unidad física particionada para LVM que sustenta todo el sistema.

## 1. Almacenamiento Físico (Physical Volume)
**Input:** \
`pvs`

**Output:**
```
  PV         VG  Fmt  Attr PSize   PFree
  /dev/sda3  pve lvm2 a--  237.47g 16.00g
```
Almacenamiento total = 237.47g

## 2. memoria RAM
**Input:**
`free -h --si`

**Output:**
```
               total        used        free      shared  buff/cache   available
Mem:             20G        6.7G         12G         52M        2.0G         14G
Swap:           8.3G        122M        8.1G
```


## 3. Distribución del almacenamiento (reservas y ocupación)
**input:** \
`lvs -a --units G`

**output**
```
  LV            VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data          pve twi-aotz-- 141.45g             81.25  3.75
  root          pve -wi-ao---- <69.45g
  swap          pve -wi-ao----  <7.69g
  vm-151-disk-0 pve Vwi-aotz--  32.00g data        41.89
  vm-153-disk-0 pve Vwi-aotz--  64.00g data        33.56
  vm-154-disk-0 pve Vwi-aotz--  64.00g data        41.82
  vm-155-disk-0 pve Vwi-aotz--  32.00g data        39.38
  vm-156-disk-0 pve Vwi-aotz--  32.00g data        48.24
  vm-157-disk-0 pve Vwi-a-tz--  32.00g data        24.04
  vm-158-disk-0 pve Vwi-aotz--   8.00g data        26.89
  vm-159-disk-0 pve Vwi-aotz--   8.00g data        38.22
  vm-160-disk-0 pve Vwi-aotz--  16.00g data        46.46
  vm-161-disk-0 pve Vwi-aotz--  16.00g data        30.69
```
La partición `data` es la reservada para los guests.

## 4. Estado de volúmenes LVM
**input:** \
`df -h`

**output:**
```
Filesystem            Size  Used Avail Use% Mounted on
udev                  9.7G     0  9.7G   0% /dev
tmpfs                 2.0G  1.2M  2.0G   1% /run
/dev/mapper/pve-root   68G   13G   52G  20% /
tmpfs                 9.8G   46M  9.7G   1% /dev/shm
tmpfs                 5.0M     0  5.0M   0% /run/lock
/dev/sda2            1022M   12M 1011M   2% /boot/efi
/dev/fuse             128M   20K  128M   1% /etc/pve
tmpfs                 2.0G     0  2.0G   0% /run/user/0
```

udev: dispositivos de hardware

## 5. Estado de dispositivos de bloque

```
lsblk
NAME                         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                            8:0    0 238.5G  0 disk
├─sda1                         8:1    0  1007K  0 part
├─sda2                         8:2    0     1G  0 part /boot/efi
└─sda3                         8:3    0 237.5G  0 part
  ├─pve-swap                 252:0    0   7.7G  0 lvm  [SWAP]
  ├─pve-root                 252:1    0  69.4G  0 lvm  /
  ├─pve-data_tmeta           252:2    0   1.4G  0 lvm
  │ └─pve-data-tpool         252:4    0 141.5G  0 lvm
  │   ├─pve-data             252:5    0 141.5G  1 lvm
  │   ├─pve-vm--151--disk--0 252:6    0    32G  0 lvm
  │   ├─pve-vm--155--disk--0 252:7    0    32G  0 lvm
  │   ├─pve-vm--153--disk--0 252:8    0    64G  0 lvm
  │   ├─pve-vm--154--disk--0 252:9    0    64G  0 lvm
  │   ├─pve-vm--156--disk--0 252:10   0    32G  0 lvm
  │   ├─pve-vm--157--disk--0 252:11   0    32G  0 lvm
  │   ├─pve-vm--158--disk--0 252:12   0     8G  0 lvm
  │   ├─pve-vm--159--disk--0 252:13   0     8G  0 lvm
  │   ├─pve-vm--160--disk--0 252:14   0    16G  0 lvm
  │   └─pve-vm--161--disk--0 252:15   0    16G  0 lvm
  └─pve-data_tdata           252:3    0 141.5G  0 lvm
    └─pve-data-tpool         252:4    0 141.5G  0 lvm
      ├─pve-data             252:5    0 141.5G  1 lvm
      ├─pve-vm--151--disk--0 252:6    0    32G  0 lvm
      ├─pve-vm--155--disk--0 252:7    0    32G  0 lvm
      ├─pve-vm--153--disk--0 252:8    0    64G  0 lvm
      ├─pve-vm--154--disk--0 252:9    0    64G  0 lvm
      ├─pve-vm--156--disk--0 252:10   0    32G  0 lvm
      ├─pve-vm--157--disk--0 252:11   0    32G  0 lvm
      ├─pve-vm--158--disk--0 252:12   0     8G  0 lvm
      ├─pve-vm--159--disk--0 252:13   0     8G  0 lvm
      ├─pve-vm--160--disk--0 252:14   0    16G  0 lvm
      └─pve-vm--161--disk--0 252:15   0    16G  0 lvm
sdb                            8:16   0 476.9G  0 disk
├─sdb1                         8:17   0   128M  0 part
├─sdb2                         8:18   0   100M  0 part
└─sdb3                         8:19   0 476.7G  0 part
```

# Proceso de preparación de unidad bloque b
se preparará unidad de almacenamiento alterna para movimiento de guests

## 1. eliminación de particiones previas
`wipefs -a /dev/sdb`

## 2. Creación de volumen físico
`pvcreate /dev/sdb`

## 3. Creación del Grupo de Volúmenes
`vgcreate pve2 /dev/sdb`

## 4. Creación del Thin Pool
`lvcreate -l 100%FREE --thinpool data2 pve2`

## 5. Prueba de movimiento
Se seleccionó la VM de glpi con id = 155 para pruebas debido a su escaso uso durante meses, mediante la intefaz web de proxmox se aplicó el siguiente proceso (Se hizo con la máquina apagada):

### 5.1 Selección de la máquina
Se seleccionó la 155
![Pantalla principal Proxmox](/images/main_proxmox.png)

### 5.2 Selección de la sección de Hardware
Ir a la sección de `Hardware` &#8594; `Disco Duro` &#8594; `Disk Action` &#8594; Move Storage
![Hardware](/images/hardware.png) 

### 5.3 selección de Unidad de Almacenamiento
La unidad de interés se nombró previamente como `ssd-512` (Marcar checbox `Eliminar origen`)
![Hardware](/images/st_select.png)

Saldrá una pantalla de avance (`Task viewer`) que se puede cerrar cuando se complete es proceso.

# Estado post pruebas
Se movió también la máquina de `Snipeit` (id = 153) y se eliminó la 154 de `Zentyal` (con debida autorización de `IT Manager`)

## 1. Distribución de ocupación y reservas

**Input:** \
`lvs`

**Output:**
```
  LV            VG   Attr       LSize   Pool  Origin Data%  Meta%  Move Log Cpy%Sync Convert
  data          pve  twi-aotz-- 141.45g              38.24  2.36
  root          pve  -wi-ao---- <69.45g
  swap          pve  -wi-ao----  <7.69g
  vm-151-disk-0 pve  Vwi-aotz--  32.00g data         41.89
  vm-156-disk-0 pve  Vwi-aotz--  32.00g data         48.24
  vm-157-disk-0 pve  Vwi-a-tz--  32.00g data         24.04
  vm-158-disk-0 pve  Vwi-aotz--   8.00g data         26.89
  vm-159-disk-0 pve  Vwi-aotz--   8.00g data         38.22
  vm-160-disk-0 pve  Vwi-aotz--  16.00g data         46.46
  vm-161-disk-0 pve  Vwi-aotz--  16.00g data         30.69
  data2         pve2 twi-aotz-- 476.70g              7.06   12.25
  vm-153-disk-0 pve2 Vwi-a-tz--  64.00g data2        33.54
  vm-155-disk-0 pve2 Vwi-a-tz--  32.00g data2        38.08
```

de un 81% de ocupación pasamos a un 38%

## 2. Estado de dispositivos de bloque

**Input:** \
`lsblk`

**Output:**
```
NAME                         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                            8:0    0 238.5G  0 disk
├─sda1                         8:1    0  1007K  0 part
├─sda2                         8:2    0     1G  0 part /boot/efi
└─sda3                         8:3    0 237.5G  0 part
  ├─pve-swap                 252:0    0   7.7G  0 lvm  [SWAP]
  ├─pve-root                 252:1    0  69.4G  0 lvm  /
  ├─pve-data_tmeta           252:2    0   1.4G  0 lvm
  │ └─pve-data-tpool         252:4    0 141.5G  0 lvm
  │   ├─pve-data             252:5    0 141.5G  1 lvm
  │   ├─pve-vm--151--disk--0 252:6    0    32G  0 lvm
  │   ├─pve-vm--156--disk--0 252:10   0    32G  0 lvm
  │   ├─pve-vm--157--disk--0 252:11   0    32G  0 lvm
  │   ├─pve-vm--158--disk--0 252:12   0     8G  0 lvm
  │   ├─pve-vm--159--disk--0 252:13   0     8G  0 lvm
  │   ├─pve-vm--160--disk--0 252:14   0    16G  0 lvm
  │   └─pve-vm--161--disk--0 252:15   0    16G  0 lvm
  └─pve-data_tdata           252:3    0 141.5G  0 lvm
    └─pve-data-tpool         252:4    0 141.5G  0 lvm
      ├─pve-data             252:5    0 141.5G  1 lvm
      ├─pve-vm--151--disk--0 252:6    0    32G  0 lvm
      ├─pve-vm--156--disk--0 252:10   0    32G  0 lvm
      ├─pve-vm--157--disk--0 252:11   0    32G  0 lvm
      ├─pve-vm--158--disk--0 252:12   0     8G  0 lvm
      ├─pve-vm--159--disk--0 252:13   0     8G  0 lvm
      ├─pve-vm--160--disk--0 252:14   0    16G  0 lvm
      └─pve-vm--161--disk--0 252:15   0    16G  0 lvm
sdb                            8:16   0 476.9G  0 disk
├─pve2-data2_tmeta           252:16   0   120M  0 lvm
│ └─pve2-data2-tpool         252:18   0 476.7G  0 lvm
│   ├─pve2-vm--153--disk--0  252:7    0    64G  0 lvm
│   ├─pve2-data2             252:19   0 476.7G  1 lvm
│   └─pve2-vm--155--disk--0  252:20   0    32G  0 lvm
└─pve2-data2_tdata           252:17   0 476.7G  0 lvm
  └─pve2-data2-tpool         252:18   0 476.7G  0 lvm
    ├─pve2-vm--153--disk--0  252:7    0    64G  0 lvm
    ├─pve2-data2             252:19   0 476.7G  1 lvm
    └─pve2-vm--155--disk--0  252:20   0    32G  0 lvm
```
