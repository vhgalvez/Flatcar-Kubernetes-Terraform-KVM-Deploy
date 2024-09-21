# Configuración de Nodos Master y Worker

----------------------------------------
Para configurar los nodos master y worker y preparar el entorno para la instalación de OKD (OpenShift Kubernetes Distribution), se deben seguir una serie de pasos detallados, incluyendo la generación de certificados y la configuración de los servicios necesarios. A continuación, se describen los pasos y configuraciones necesarias para cada tipo de nodo.

Generación de Claves y Certificados

Primero, necesitamos generar varios certificados y claves que serán utilizados por los diferentes componentes de Kubernetes.

# Crear directorios necesarios


```bash
sudo mkdir -p /etc/kubernetes/pki/etcd
```

# Generar clave y certificado de la CA


```bash
sudo openssl genrsa -out /etc/kubernetes/pki/ca.key 2048
sudo openssl req -x509 -new -nodes -key /etc/kubernetes/pki/ca.key -subj "/CN=kube-ca" -days 10000 -out /etc/kubernetes/pki/ca.crt
```

# Generar clave y certificado para el API Server


# Generar clave y certificado para etcd

```bash
sudo openssl genrsa -out /etc/kubernetes/pki/etcd/ca.key 2048
sudo openssl req -new -key /etc/kubernetes/pki/etcd/ca.key -subj "/CN=etcd-ca" -out /etc/kubernetes/pki/etcd/ca.csr
sudo openssl x509 -req -in /etc/kubernetes/pki/etcd/ca.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/etcd/ca.crt -days 10000
```

# Generar clave y certificado para el cliente de etcd

```bash
sudo openssl genrsa -out /etc/kubernetes/pki/apiserver-etcd-client.key 2048
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-etcd-client.key -subj "/CN=etcd-client" -out /etc/kubernetes/pki/apiserver-etcd-client.csr
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-etcd-client.csr -CA /etc/kubernetes/pki/etcd/ca.crt -CAkey /etc/kubernetes/pki/etcd/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-etcd-client.crt -days 10000
```

# Generar clave y certificado para el cliente kubelet

```bash
sudo openssl genrsa -out /etc/kubernetes/pki/apiserver-kubelet-client.key 2048
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-kubelet-client.key -subj "/CN=kubelet-client" -out /etc/kubernetes/pki/apiserver-kubelet-client.csr
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-kubelet-client.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-kubelet-client.crt -days 10000
```

# Generar clave y certificado de servicio de cuenta

```bash
sudo openssl genrsa -out /etc/kubernetes/pki/sa.key 2048
sudo openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
```

Configuración de Servicios

```bash
sudo vim /etc/systemd/system/kube-apiserver.service
```

# Configuración de Servicios

```bash
[Unit]
Description=Kubernetes API Server
Documentation=https://kubernetes.io/docs/concepts/overview/components/
After=network.target

[Service]
ExecStart=/opt/bin/kube-apiserver \
  --advertise-address=10.17.4.21 \
  --allow-privileged=true \
  --authorization-mode=Node,RBAC \
  --client-ca-file=/etc/kubernetes/pki/ca.crt \
  --enable-admission-plugins=NodeRestriction \
  --etcd-servers=https://127.0.0.1:2379 \
  --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt \
  --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt \
  --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key \
  --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt \
  --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key \
  --runtime-config=api/all=true \
  --service-account-key-file=/etc/kubernetes/pki/sa.pub \
  --service-account-signing-key-file=/etc/kubernetes/pki/sa.key \
  --service-account-issuer=https://kubernetes.default.svc.cluster.local \
  --service-cluster-ip-range=10.96.0.0/12 \
  --tls-cert-file=/etc/kubernetes/pki/apiserver.crt \
  --tls-private-key-file=/etc/kubernetes/pki/apiserver.key \
  --v=2
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
export ETCD_NAME="etcd0"
export ETCD_DATA_DIR="/var/lib/etcd"
export ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.17.4.21:2380"
export ETCD_LISTEN_PEER_URLS="http://10.17.4.21:2380"
export ETCD_LISTEN_CLIENT_URLS="http://10.17.4.21:2379,http://127.0.0.1:2379"
export ETCD_ADVERTISE_CLIENT_URLS="http://10.17.4.21:2379"
export ETCD_INITIAL_CLUSTER="etcd0=http://10.17.4.21:2380"
export ETCD_INITIAL_CLUSTER_STATE="new"
export ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
```

```bash
sudo chown -R etcd:etcd /var/lib/etcd
sudo chmod -R 700 /var/lib/etcd
```

Para asegurar que etcd se ejecute correctamente, necesitas asegurarte de que la configuración es correcta y que etcd esté efectivamente escuchando en las direcciones y puertos adecuados. También es importante verificar los archivos de logs para entender qué puede estar fallando.

# Verificar el estado de etcd

Primero, verifica el estado del servicio etcd:

```bash
sudo systemctl status etcd
```

Si el servicio etcd no está activo, revisa los logs para identificar el problema:

```bash
sudo journalctl -u etcd -f
```

# Archivos de configuración de etcd

El archivo /etc/systemd/system/etcd.service que proporcionaste parece estar bien, pero asegúrate de que las direcciones IP y puertos están configurados correctamente para tu entorno.

Configuración del Servicio etcd

Verifica y ajusta la configuración del servicio etcd si es necesario. Aquí está un ejemplo de una configuración típica para un clúster etcd:

```ini
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
After=network.target

[Service]
User=etcd
Type=notify
Environment="ETCD_DATA_DIR=/var/lib/etcd"
Environment="ETCD_NAME=etcd0"
Environment="ETCD_INITIAL_ADVERTISE_PEER_URLS=http://10.17.4.21:2380"
Environment="ETCD_LISTEN_PEER_URLS=http://10.17.4.21:2380"
Environment="ETCD_LISTEN_CLIENT_URLS=http://10.17.4.21:2379,http://127.0.0.1:2379"
Environment="ETCD_ADVERTISE_CLIENT_URLS=http://10.17.4.21:2379"
Environment="ETCD_INITIAL_CLUSTER=etcd0=http://10.17.4.21:2380"
Environment="ETCD_INITIAL_CLUSTER_STATE=new"
Environment="ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster"
ExecStart=/opt/bin/etcd
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

Asegúrate de que las variables de entorno están definidas correctamente. Puedes definir estas variables en el archivo de servicio o exportarlas antes de iniciar el servicio:

```sh
export ETCD_NAME="etcd0"
export ETCD_DATA_DIR="/var/lib/etcd"
export ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.17.4.21:2380"
export ETCD_LISTEN_PEER_URLS="http://10.17.4.21:2380"
export ETCD_LISTEN_CLIENT_URLS="http://10.17.4.21:2379,http://127.0.0.1:2379"
export ETCD_ADVERTISE_CLIENT_URLS="http://10.17.4.21:2379"
export ETCD_INITIAL_CLUSTER="etcd0=http://10.17.4.21:2380"
export ETCD_INITIAL_CLUSTER_STATE="new"
export ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
```

Permisos y Propietario

Asegúrate de que los permisos y el propietario del directorio de datos de etcd están configurados correctamente:

```bash
sudo chown -R etcd:etcd /var/lib/etcd
sudo chmod -R 700 /var/lib/etcd
```

Reiniciar el Servicio etcd

Después de verificar la configuración y permisos, reinicia el servicio etcd:

```bash
sudo systemctl daemon-reload
sudo systemctl restart etcd
```

Verificar el Estado de etcd

Finalmente, verifica el estado de etcd nuevamente para asegurarte de que está activo y funcionando correctamente:

```bash
sudo systemctl status etcd
```

Si el servicio sigue sin funcionar, consulta los logs para identificar problemas específicos:

```bash
sudo journalctl -u etcd -f
```

Configuración para el Entorno de Producción

Si estás configurando etcd para un entorno de producción con múltiples nodos, necesitarás ajustar las direcciones IP y los puertos en consecuencia y asegurarte de que todos los nodos en el clúster de etcd pueden comunicarse entre sí.

Con estas configuraciones y pasos, deberías poder asegurarte de que etcd está configurado correctamente y funcionando como se espera en tu clúster de Kubernetes.

----------------------------------------


# Configuración de Nodos Master y Worker


Para configurar los nodos master y worker y preparar el entorno para la instalación de OKD (OpenShift Kubernetes Distribution), se deben seguir una serie de pasos detallados, incluyendo la generación de certificados y la configuración de los servicios necesarios. A continuación, se describen los pasos y configuraciones necesarias para cada tipo de nodo.

Generación de Claves y Certificados


```bash
sudo vim /etc/systemd/system/etcd.service
```


```ini
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
After=network.target

[Service]
User=etcd
Type=notify
Environment="ETCD_DATA_DIR=/var/lib/etcd"
Environment="ETCD_NAME=etcd0"
Environment="ETCD_INITIAL_ADVERTISE_PEER_URLS=http://10.17.4.21:2380"
Environment="ETCD_LISTEN_PEER_URLS=http://10.17.4.21:2380"
Environment="ETCD_LISTEN_CLIENT_URLS=http://10.17.4.21:2379,http://127.0.0.1:2379"
Environment="ETCD_ADVERTISE_CLIENT_URLS=http://10.17.4.21:2379"
Environment="ETCD_INITIAL_CLUSTER=etcd0=http://10.17.4.21:2380"
Environment="ETCD_INITIAL_CLUSTER_STATE=new"
Environment="ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster"
ExecStart=/opt/bin/etcd \
  --name ${ETCD_NAME} \
  --data-dir ${ETCD_DATA_DIR} \
  --listen-peer-urls ${ETCD_LISTEN_PEER_URLS} \
  --listen-client-urls ${ETCD_LISTEN_CLIENT_URLS} \
  --advertise-client-urls ${ETCD_ADVERTISE_CLIENT_URLS} \
  --initial-advertise-peer-urls ${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
  --initial-cluster ${ETCD_INITIAL_CLUSTER} \
  --initial-cluster-state ${ETCD_INITIAL_CLUSTER_STATE} \
  --initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN}
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```


Paso 1: Crear el archivo controller-manager.conf con oc
Utiliza sudo para ejecutar los comandos oc y crear el archivo controller-manager.conf:

bash
Copiar código
sudo /opt/bin/oc config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --embed-certs=true \
  --server=https://10.17.4.21:6443 \
  --kubeconfig=/etc/kubernetes/controller-manager.conf

sudo /opt/bin/oc config set-credentials system:kube-controller-manager \
  --client-certificate=/etc/kubernetes/pki/controller-manager.crt \
  --client-key=/etc/kubernetes/pki/controller-manager.key \
  --embed-certs=true \
  --kubeconfig=/etc/kubernetes/controller-manager.conf

sudo /opt/bin/oc config set-context system:kube-controller-manager@kubernetes \
  --cluster=kubernetes \
  --user=system:kube-controller-manager \
  --kubeconfig=/etc/kubernetes/controller-manager.conf

sudo /opt/bin/oc config use-context system:kube-controller-manager@kubernetes --kubeconfig=/etc/kubernetes/controller-manager.conf
Paso 2: Ajustar permisos del archivo de configuración
Asegúrate de que los permisos y propietarios del archivo sean correctos:

bash
Copiar código
sudo chown root:root /etc/kubernetes/controller-manager.conf
sudo chmod 644 /etc/kubernetes/controller-manager.conf
Paso 3: Recargar y reiniciar el servicio
Recarga los archivos de configuración del sistema y reinicia el servicio kube-controller-manager:

bash
Copiar código
sudo systemctl daemon-reload
sudo systemctl restart kube-controller-manager
Paso 4: Verificar el estado del servicio
Verifica el estado del servicio para asegurarte de que está funcionando correctamente:

bash
Copiar código
sudo systemctl status kube-controller-manager
Paso 5: Verificar los logs si hay problemas
Si el servicio sigue fallando, verifica los logs para identificar el problema:

bash
Copiar código
sudo journalctl -xeu kube-controller-manager

----------------------------------------

# Configuración de Nodos Master y Worker

Paso 1: Generar las claves y certificados necesarios para el kube-scheduler
bash
Copiar código
sudo openssl genrsa -out /etc/kubernetes/pki/scheduler.key 2048
sudo openssl req -new -key /etc/kubernetes/pki/scheduler.key -subj "/CN=system:kube-scheduler" -out /etc/kubernetes/pki/scheduler.csr
sudo openssl x509 -req -in /etc/kubernetes/pki/scheduler.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/scheduler.crt -days 10000
Paso 2: Crear el archivo de configuración del kube-scheduler
bash
Copiar código
sudo /opt/bin/oc config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --embed-certs=true \
  --server=https://10.17.4.21:6443 \
  --kubeconfig=/etc/kubernetes/scheduler.conf

sudo /opt/bin/oc config set-credentials system:kube-scheduler \
  --client-certificate=/etc/kubernetes/pki/scheduler.crt \
  --client-key=/etc/kubernetes/pki/scheduler.key \
  --embed-certs=true \
  --kubeconfig=/etc/kubernetes/scheduler.conf

sudo /opt/bin/oc config set-context system:kube-scheduler@kubernetes \
  --cluster=kubernetes \
  --user=system:kube-scheduler \
  --kubeconfig=/etc/kubernetes/scheduler.conf

sudo /opt/bin/oc config use-context system:kube-scheduler@kubernetes --kubeconfig=/etc/kubernetes/scheduler.conf
Paso 3: Asegurar los permisos correctos para el archivo de configuración
bash
Copiar código
sudo chown root:root /etc/kubernetes/scheduler.conf
sudo chmod 644 /etc/kubernetes/scheduler.conf
Paso 4: Reiniciar el servicio kube-scheduler
Después de realizar las verificaciones y ajustes necesarios, recarga los demonios de systemd y reinicia el servicio kube-scheduler:

bash
Copiar código
sudo systemctl daemon-reload
sudo systemctl restart kube-scheduler
sudo systemctl status kube-scheduler
Verificación final
Verifica nuevamente el estado del servicio kube-scheduler y revisa los registros para asegurarte de que se haya iniciado correctamente:

bash
Copiar código
sudo systemctl status kube-scheduler
sudo journalctl -u kube-scheduler -xe
