# Año: 2026
## Mes: Abril
### Día 22

- Migración de fileserver unidad b (CT LXC id = 160): bajó la ocupación del volumen `data` del 38% al 33% (ver con input: `lvs`)

### Día 24
Verificación de integridad de protocolo websocket para vaultwarden:

**input:**
```
docker exec -it vaultwarden netstat -tulpn | grep 3012
```

**output:** \
No salió nada

al no salir nada con la ejecución rápida de un contenedor hemos de entender que el contenedor no levanta internamente el protocolo aún y cuando en `compose.yml` se declare la habilitación de websocket (WEBSOCKET_ENABLED: "true")

### Día 28

- Se requiere gestionar el protocolo websocket para vaultwarden (según lo diagnosticado el 24 de abril), esta situación es la causa presunta de los problemas de sincronización que han afectado la experiencia de los usuarios (agentes de DSI). Se hizo backup de todo el directorio donde está alojado tanto la base de datos como los archivos de configuración sin realizar modificaciones que siendo necesarias requieren una suspensión temporal del servicio lo cual podría chocar con la operación.

- informe de backup:
    - ruta del volumen y configuración de vaultwarden: /opt/vaultwarden (en contenedor LXC con id 158).
    - verificación de montaje del contenedor: \
    **input:**
    pct mount 158 \
    **output:**
        mounted CT 158 in '/var/lib/lxc/158/rootfs'

    - Creación del directorio en el host de proxmox y copia 
        ```
        mkdir -p /root/backups_allitech/vaultwarden_2026-04-28
        ```
        ```
        cp -av /var/lib/lxc/158/rootfs/opt/vaultwarden/. /root/backups_allitech/vaultwarden_2026-04-28/
        ```
    - creación de tar.gz (para preservar integridad de Linux a Windows)
        ```
        sudo tar -cvzf /root/backups_allitech/vaultwarden_2026-04-28.tar.gz -C /root/backups_allitech vaultwarden_2026-04-28
        ```
    - Generación de hash para verificación de integridad (Linux [bash]) \
    **input:**
        ```
        sha256sum /root/backups_allitech/vaultwarden_2026-04-28.tar.gz
        ```
        **output:** \
        958f82d3acadc121a8deada99571c79feeac0d096fab8c6b76057ffd1cccf345  /root/backups_allitech/vaultwarden_2026-04-28.tar.gz
    - Generación de hash para verificación de integridad(Windows [pwsh]) \
    **input:**
        ```
        Get-FileHash Get-FileHash D:\Documentos\backups\vaultwarden_2026-04-28.tar.gz -Algorithm SHA256
        ```
        **output:** \
        SHA256          958F82D3ACADC121A8DEADA99571C79FEEAC0D096FAB8C6B76057FFD1CCCF345       D:\Documentos\backups\vaultwarden_2026-04-28.tar.gz
    
        ambos hash deben ser idénticos
    - Verificación de identidad [pwsh] \
    **input:**
        ```
        "958f82d3acadc121a8deada99571c79feeac0d096fab8c6b76057ffd1cccf345".ToUpper() -eq "958F82D3ACADC121A8DEADA99571C79FEEAC0D096FAB8C6B76057FFD1CCCF345"
        ```
         **output:** True
    
        al haber obtenido `True` se demuestra que el backup alojado en proxmox es idéntico a su copia en máquina de trabajo windows 


- el plan (experimental): 
    - eliminar archivo de configuración .json que es prioridad sobre `compose.yml` para que la prioridad sea este último que si declara la habilitación del protocolo de websocket.

    - bajar servicio `docker compose down`

    - levantar servicio `docker up -d`

de lunes a viernes el nivel de usuarios es crítico y acorde a la jornada del personal de sistemas hay poco espacio de maniobra para poder ejecutar el plan sin comprometer la operación, por lo que se requiere definir un espacio de tiempo adecuado.
