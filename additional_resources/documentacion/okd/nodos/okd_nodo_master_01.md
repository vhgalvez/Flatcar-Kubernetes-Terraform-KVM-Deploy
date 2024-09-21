Parece que el servicio kube-apiserver ahora está activo y funcionando correctamente. A continuación, te proporcionaré los pasos detallados y la configuración necesaria para los nodos master, worker y bootstrap para una instalación completa de OKD.

Nodos Master

Software Requerido

etcd: Base de datos distribuida para almacenar datos de configuración.
kube-apiserver: Proporciona la API de Kubernetes.
kube-controller-manager: Ejecuta los controladores de Kubernetes.
kube-scheduler: Programa los pods en los nodos.
kubelet: Agente que se comunica con el kube-apiserver.
Certificados y Claves
Asegúrate de que los siguientes archivos estén presentes en /etc/kubernetes/pki/:

ca.crt y ca.key
apiserver.crt y apiserver.key
etcd/ca.crt, etcd/server.crt, etcd/server.key
sa.pub y sa.key
Configuración del Servicio kube-apiserver
Guarda la siguiente configuración en /etc/systemd/system/kube-apiserver.service:

```ini
[Unit]
Description=Kubernetes API Server
Documentation=https://kubernetes.io/docs/concepts/overview/components/
After=network.target

[Service]
ExecStart=/opt/bin/kube-apiserver \
  --advertise-address=<MASTER_IP> \
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

Reemplaza <MASTER_IP> con la dirección IP del nodo 

master correspondiente.

Nodos Worker

Software Requerido
kubelet: Agente que se comunica con el kube-apiserver.
crio o containerd: Runtime de contenedores.
kube-proxy: Gestiona las reglas de red en cada nodo.
Configuración del Servicio kubelet
Guarda la siguiente configuración en /etc/systemd/system/kubelet.service:

```ini
[Unit]
Description=Kubernetes Kubelet
Documentation=https://kubernetes.io/docs/home/
After=network.target

[Service]
ExecStart=/opt/bin/kubelet --config=/etc/kubernetes/kubelet-config.yaml --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --container-runtime=remote --container-runtime-endpoint=unix:///var/run/crio/crio.sock --allow-privileged=true --network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Nodo Bootstrap

El nodo bootstrap se utiliza para instalar y configurar el clúster OKD. Asegúrate de tener openshift-install en el nodo bootstrap y sigue estos pasos para iniciar el despliegue:

```sh
openshift-install create cluster --dir=<directory_with_install_config> --log-level=info
```

Sigue las instrucciones del instalador para completar la configuración y despliegue del clúster OKD.

Generación de Certificados

Si no tienes los certificados necesarios, puedes generarlos con los siguientes comandos:

Generación de Claves y Certificados

# CA

sudo openssl genrsa -out /etc/kubernetes/pki/ca.key 2048

sudo openssl req -x509 -new -nodes -key /etc/kubernetes/pki/ca.key -subj "/CN=kube-ca" -days 10000 -out /etc/kubernetes/pki/ca.crt

# API Server
sudo openssl genrsa -out /etc/kubernetes/pki/apiserver.key 2048
sudo openssl req -new -key /etc/kubernetes/pki/apiserver.key -subj "/CN=kube-apiserver" -out /etc/kubernetes/pki/apiserver.csr
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver.crt -days 10000

# etcd
sudo openssl genrsa -out /etc/kubernetes/pki/etcd/ca.key 2048
sudo openssl req -new -key /etc/kubernetes/pki/etcd/ca.key -subj "/CN=etcd-ca" -out /etc/kubernetes/pki/etcd/ca.csr
sudo openssl x509 -req -in /etc/kubernetes/pki/etcd/ca.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/etcd/ca.crt -days 10000

# apiserver-etcd-client

sudo openssl genrsa -out /etc/kubernetes/pki/apiserver-etcd-client.key 2048
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-etcd-client.key -subj "/CN=etcd-client" -out /etc/kubernetes/pki/apiserver-etcd-client.csr
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-etcd-client.csr -CA /etc/kubernetes/pki/etcd/ca.crt -CAkey /etc/kubernetes/pki/etcd/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-etcd-client.crt -days 10000

# apiserver-kubelet-client

sudo openssl genrsa -out /etc/kubernetes/pki/apiserver-kubelet-client.key 2048
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-kubelet-client.key -subj "/CN=kubelet-client" -out /etc/kubernetes/pki/apiserver-kubelet-client.csr
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-kubelet-client.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-kubelet-client.crt -days 10000

Despliegue de OKD

Inicia el despliegue desde el nodo bootstrap:

```sh
openshift-install create cluster --dir=<directory_with_install_config> --log-level=info
```

Sigue las instrucciones del instalador para completar la configuración y despliegue del clúster OKD.

Asegúrate de que cada paso se realice con éxito y que todos los nodos puedan comunicarse entre sí. Una vez completado, tu clúster OKD debería estar en funcionamiento.
