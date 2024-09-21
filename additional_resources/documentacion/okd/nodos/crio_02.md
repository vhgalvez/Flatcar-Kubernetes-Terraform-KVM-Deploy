# Documentación de Instalación y Configuración de CRI-O

1. Instalación de CRI-O

## Paso 1: Descargar el Binario de CRI-O

```bash
sudo wget -O /tmp/crio.tar.gz https://storage.googleapis.com/cri-o/artifacts/cri-o.amd64.v1.30.3.tar.gz
```
## Paso 2: Crear un Directorio Temporal

```bash
sudo mkdir -p /tmp/crio
```
## Paso 3: Extraer el Archivo Descargado

```bash
¡sudo tar -xzf /tmp/crio.tar.gz -C /tmp/crio
```

## Paso 4: Crear el Directorio de Instalación

```bash
sudo mkdir -p /opt/bin/crio
```

## Paso 5: Mover los Binarios a /opt/bin/crio/

```bash
sudo mv /tmp/crio/cri-o/bin/* /opt/bin/crio/
```

## Paso 6: Verificar la Instalación

```bash
/opt/bin/crio/crio --version
```

1. Configuración de CRI-O
   
Archivo de Configuración (/etc/crio/crio.conf):

Primero, asegúrate de que el directorio /etc/crio/ exista:

```bash
sudo mkdir -p /etc/crio/
```

Luego, edita el archivo de configuración:

```bash
sudo vi /etc/crio/crio.conf
```

Y agrega el siguiente contenido:

```ini
[crio]
log_level = "debug"
root = "/var/lib/crio"
runroot = "/var/run/crio"
log_dir = "/var/log/crio/pods"
version_file = "/var/run/crio/version"
clean_shutdown_file = "/var/lib/crio/clean.shutdown"

[crio.api]
listen = "/var/run/crio/crio.sock"
stream_address = "127.0.0.1"
stream_port = "0"
grpc_max_send_msg_size = 83886080
grpc_max_recv_msg_size = 83886080

[crio.runtime]
default_runtime = "runc"
no_pivot = false
decryption_keys_path = "/etc/crio/keys/"
cgroup_manager = "systemd"
drop_infra_ctr = true
infra_ctr_cpuset = ""
shared_cpuset = ""
namespaces_dir = "/var/run"
enable_criu_support = true
pinns_path = "/opt/bin/crio/pinns"

[crio.runtime.runtimes.runc]
runtime_path = "/opt/bin/crio/crio-runc"
runtime_type = "oci"
runtime_root = "/run/runc"

[crio.image]
pause_image = "k8s.gcr.io/pause:3.2"
image_volumes = "mkdir"

[crio.network]
network_dir = "/etc/cni/net.d/"
plugin_dirs = ["/opt/cni/bin/"]
```

Archivo del Servicio (/etc/systemd/system/crio.service):

```ini
[Unit]
Description=CRI-O container runtime
After=network.target

[Service]
Type=notify
ExecStart=/opt/bin/crio/crio
Environment="PATH=/opt/bin/crio:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

1. Verificación del Servicio
   
Estado del Servicio:

```bash
sudo systemctl status crio
```

Verificación de Logs:

```bash
sudo journalctl -u crio -f
```

Reiniciar el Servicio (si se realizan cambios):

```bash
sudo systemctl restart crio
sudo systemctl status crio
```

1. Instalación y Configuración de conmon
   
Descargar conmon:

```bash
wget https://github.com/containers/conmon/releases/download/v2.1.12/conmon.amd64
```

Mover y Renombrar el Archivo Descargado:

```bash
sudo mkdir -p /opt/bin/crio
sudo mv conmon.amd64 /opt/bin/crio/conmon
```

Dar Permisos de Ejecución:

```bash
sudo chmod +x /opt/bin/crio/conmon
```

Actualizar crio.conf para Reflejar la Ruta Correcta de conmon

Asegúrate de que la siguiente línea esté correctamente establecida en /etc/crio/crio.conf bajo la sección [crio.runtime]:

```ini
conmon = "/opt/bin/crio/conmon"
```

Recargar y Reiniciar CRI-O:

```bash
sudo systemctl daemon-reload
sudo systemctl restart crio
```

Verificar el Estado de CRI-O:

```bash
sudo systemctl status crio
```

Confirmar los Binarios de CRI-O y Permisos:

```bash
ls -l /opt/bin/crio/
```

```bash
core@worker3 ~ $ ls -l /opt/bin/crio/
total 130896
-rwxr-xr-x. 1 root root    1969712 May 17 08:50 conmon
-rwxr-xr-x. 1 1001 docker 58376628 Apr 18 08:21 crictl
-rwxr-xr-x. 1 1001 docker 51066800 Jul  1 11:27 crio
-rwxr-xr-x. 1 1001 docker  1969712 Jul  1 11:28 crio-conmon
-rwxr-xr-x. 1 1001 docker  6197224 Jul  1 11:28 crio-conmonrs
-rwxr-xr-x. 1 1001 docker  2931384 Jul  1 11:28 crio-crun
-rwxr-xr-x. 1 1001 docker 10802720 Jul  1 11:28 crio-runc
-rwxr-xr-x. 1 1001 docker   708720 Jul  1 11:27 pinns
core@worker3 ~ $ ls -l /opt/bin/
total 235528
drwxr-xr-x. 2 root root      4096 Jul 25 14:34 crio
-rwxr-xr-x. 1 root root 118062928 Jul 25 14:22 kubelet
-rwxr-xr-x. 1 root root 123112560 Aug 31  2021 oc
core@worker3 ~ $
```



Con estos pasos, CRI-O y conmon deberían estar correctamente instalados y configurados en tu sistema. Si encuentras algún problema, revisa los logs y la configuración de tu entorno para resolver cualquier inconveniente.


https://github.com/cri-o/cri-o/releases
https://github.com/containers/conmon/releases
