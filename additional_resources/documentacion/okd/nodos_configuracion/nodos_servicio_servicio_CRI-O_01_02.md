
# Guía Mejorada para Instalar y Configurar CRI-O en Flatcar Container Linux

Esta guía está optimizada para la instalación de CRI-O en Flatcar Container Linux, un sistema con un diseño de archivos de solo lectura en sus directorios principales. Utilizaremos los directorios accesibles y extensiones del sistema para asegurar una instalación persistente y funcional.

## Paso 1: Descargar los Binarios de CRI-O

El primer paso es descargar los binarios de CRI-O desde una fuente oficial y extraerlos en un directorio modificable, como /opt/bin:

```bash
sudo wget -O /tmp/crio.tar.gz https://storage.googleapis.com/cri-o/artifacts/cri-o.amd64.v1.30.5.tar.gz
sudo mkdir -p /opt/bin/crio
sudo tar -xzf /tmp/crio.tar.gz -C /opt/bin/crio
```
## Paso 2: Configurar los Plugins CNI

Los plugins CNI (Container Network Interface) son necesarios para la red de contenedores. Moveremos los plugins a /opt/cni/bin, un directorio accesible en Flatcar:

```bash
sudo mkdir -p /opt/cni/bin
sudo cp -r /opt/bin/crio/cri-o/cni-plugins/* /opt/cni/bin/
```
## Paso 3: Crear el Servicio Systemd para CRI-O

Flatcar tiene un sistema de archivos de solo lectura en directorios como /usr/libexec. Para manejar CRI-O, debes crear un servicio systemd en /etc/systemd/system/. Esto permitirá que CRI-O se ejecute y sea gestionado como un servicio:

```bash
sudo tee /etc/systemd/system/crio.service > /dev/null <<EOF
[Unit]
Description=CRI-O Container Runtime
Documentation=https://github.com/cri-o/cri-o
After=network.target

[Service]
ExecStart=/opt/bin/crio/cri-o/bin/crio
ExecReload=/bin/kill -HUP \$MAINPID
KillMode=process
Restart=on-failure
RestartSec=5
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```
## Paso 4: Habilitar e Iniciar CRI-O como Servicio

Una vez que hayas creado el servicio, puedes habilitarlo e iniciarlo con los siguientes comandos:

```bash
sudo systemctl enable crio.service
sudo systemctl start crio.service
```
Paso 5: Verificar la Instalación de CRI-O
Verifica que el servicio CRI-O esté en funcionamiento y que no haya errores:

```bash
sudo systemctl status crio.service
```
Si el servicio ha iniciado correctamente, debería aparecer como "activo (running)".

## Paso 6: Utilizar Extensiones del Sistema en Flatcar

Flatcar Container Linux utiliza un sistema de extensiones que permite añadir funcionalidades adicionales de manera persistente sin modificar el sistema de archivos principal.

### 6.1: Habilitar la Extensión de CRI-O

Para habilitar la extensión de CRI-O, ejecuta los siguientes comandos:

```bash
sudo systemctl enable extensions-crio.slice
sudo systemctl start extensions-crio.slice
```

Esto asegura que CRI-O esté disponible a través de las extensiones del sistema.

### 6.2: Verificar Extensiones Activas

Puedes verificar que las extensiones estén activas usando:

```bash
sudo systemctl status extensions-crio.slice
```

**Solución de Problemas:** Sistema de Archivos de Solo Lectura

Si encuentras problemas con permisos o con el sistema de archivos de solo lectura, asegúrate de trabajar en directorios modificables como /opt o /etc. Evita modificar /usr/ o cualquier otro directorio de solo lectura.

Verificación Final de la Instalación
Después de ejecutar el script de instalación, verifica nuevamente el estado del servicio:

```bash
sudo systemctl status crio.service
```

Si el archivo de servicio está configurado correctamente y el sistema está habilitado, CRI-O debería estar funcionando sin problemas.

## Conclusión

Este proceso asegura que CRI-O se instala correctamente en Flatcar Container Linux, utilizando directorios modificables y gestionando la persistencia a través de extensiones del sistema. Esta metodología es ideal para trabajar en un entorno con restricciones de solo lectura, permitiendo actualizaciones y mantenimiento sin comprometer la integridad del sistema.

Si tienes más preguntas o necesitas más ajustes, no dudes en preguntar.