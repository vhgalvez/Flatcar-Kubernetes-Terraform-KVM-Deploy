# Tutorial: Creación y Configuración del Servicio kube-scheduler en Kubernetes

Este tutorial te guiará a través de los pasos necesarios para crear y configurar el servicio kube-scheduler en un clúster de Kubernetes, generando los certificados requeridos y asegurando que el servicio esté correctamente configurado.

## 1. Crear el Archivo de Servicio para kube-scheduler

### 1.1 Crear el archivo de servicio en systemd

Primero, necesitamos crear el archivo de servicio en el directorio `/etc/systemd/system/` para que systemd pueda gestionar el servicio kube-scheduler.

```bash
sudo vim /etc/systemd/system/kube-scheduler.service
```


### 1.2 Añadir el contenido del servicio

Copia y pega el siguiente contenido en el archivo de servicio:

```bash
[Unit]
Description=Kubernetes Scheduler
Documentation=https://kubernetes.io/docs/concepts/overview/components/
After=network.target

[Service]
ExecStart=/opt/bin/kube-scheduler --address=127.0.0.1 --kubeconfig=/etc/kubernetes/scheduler.conf --leader-elect=true --v=2
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Este archivo define cómo debe ejecutarse el kube-scheduler, su configuración de alta disponibilidad (opción `--leader-elect=true`) y dónde está ubicado su archivo de configuración (`/etc/kubernetes/scheduler.conf`).



## 2. Crear el Archivo de Configuración para kube-scheduler

### 2.1 Crear el archivo de configuración del kube-scheduler

El siguiente paso es crear el archivo de configuración que permitirá al kube-scheduler conectarse al kube-apiserver. Crea el archivo en `/etc/kubernetes/scheduler.conf`:


```bash
sudo vim /etc/kubernetes/scheduler.conf
```

### 2.2 Añadir el contenido del archivo de configuración

Copia y pega el siguiente contenido en el archivo de configuración:

```bash
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://10.17.4.23:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: system:kube-scheduler
  name: system:kube-scheduler@kubernetes
current-context: system:kube-scheduler@kubernetes
users:
- name: system:kube-scheduler
  user:
    client-certificate: /etc/kubernetes/pki/kube-scheduler.crt
    client-key: /etc/kubernetes/pki/kube-scheduler.key
```

Este archivo configura el kube-scheduler para que utilice los certificados correctos y se conecte al servidor de API en la dirección `https://10.17.4.22:6443`.


## 3. Generar los Certificados para kube-scheduler

### 3.1 Generar una nueva clave privada

Crea una nueva clave privada para el kube-scheduler:

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/kube-scheduler.key -pkeyopt rsa_keygen_bits:2048
```

### 3.2 Crear una solicitud de firma de certificado (CSR)

Crea una solicitud de firma de certificado utilizando la clave privada generada:

```bash
sudo openssl req -new -key /etc/kubernetes/pki/kube-scheduler.key -subj "/CN=system:kube-scheduler" -out /etc/kubernetes/pki/kube-scheduler.csr
```

### 3.3 Firmar el CSR con la CA

Firma el CSR usando la autoridad certificadora (CA) de Kubernetes para obtener el certificado final:


```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/kube-scheduler.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kube-scheduler.crt -days 365
```

### 3.4 Ajustar los permisos

Asegúrate de que los permisos de los archivos generados sean correctos para que solo root pueda leer la clave privada y todos puedan leer el certificado público:

```bash
sudo chmod 600 /etc/kubernetes/pki/kube-scheduler.key
sudo chmod 644 /etc/kubernetes/pki/kube-scheduler.crt
```

## 4. Iniciar y Habilitar el Servicio

### 4.1 Recargar el demonio de systemd

Después de crear el archivo de servicio, recarga el demonio de systemd para que reconozca el nuevo servicio:

```bash
sudo systemctl daemon-reload
```

### 4.2 Iniciar el servicio kube-scheduler

Inicia el servicio kube-scheduler y habilítalo para que se inicie automáticamente al reiniciar el sistema:
  
```bash
sudo systemctl start kube-scheduler
sudo systemctl enable kube-scheduler
```

### 4.3 Verificar el estado del servicio

Verifica si el servicio `kube-scheduler` se ha iniciado correctamente:

```bash
sudo systemctl status kube-scheduler
```

## 5. Verificar la Configuración del kube-scheduler

### 5.1 Verificar la conectividad con kube-apiserver

Asegúrate de que el kube-scheduler esté correctamente conectado al kube-apiserver comprobando el estado de los nodos en el clúster:
  
```bash
oc get nodes
```
### 5.2 Verificar los logs del servicio

Monitorea los logs del servicio kube-scheduler para identificar cualquier posible error o advertencia:

```bash
sudo journalctl -u kube-scheduler -f
```
