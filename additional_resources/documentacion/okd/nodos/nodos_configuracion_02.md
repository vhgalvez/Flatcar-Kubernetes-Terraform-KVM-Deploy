# Clasificación y Distribución de Certificados en un Clúster Kubernetes

## 1. Certificados que Deben Estar en Todos los Nodos (Masters, Workers, Bootstrap)

Estos certificados son esenciales para la autenticidad y comunicación segura entre los componentes del clúster en todos los nodos.

- **CA Certificate (ca.crt)**
  - **Ubicación:** `/etc/kubernetes/pki/ca.crt`
  - **Propósito:** Es la autoridad de certificación para todo el clúster, utilizado por todos los nodos para verificar otros certificados.

- **Kubelet Certificate (kubelet.crt)**
  - **Ubicación:** `/etc/kubernetes/pki/kubelet.crt`
  - **Propósito:** Autentica el kubelet en cada nodo con el API server, asegurando que el nodo es legítimo y está autorizado a unirse al clúster.

- **Kubelet Key (kubelet.key)**
  - **Ubicación:** `/etc/kubernetes/pki/kubelet.key`
  - **Propósito:** Es la clave privada correspondiente al certificado del kubelet, permitiendo que el nodo firme su identidad al comunicarse con el API server.

## 2. Certificados Específicos para los Nodos Master

Los nodos Master administran componentes críticos como el API server, el controlador, y etcd, y por lo tanto requieren certificados adicionales.

- **API Server Certificate (apiserver.crt)**
  - **Ubicación:** `/etc/kubernetes/pki/apiserver.crt`
  - **Propósito:** Autentica el API server en los nodos Master con los clientes (como kubectl), asegurando la comunicación segura.

- **API Server Key (apiserver.key)**
  - **Ubicación:** `/etc/kubernetes/pki/apiserver.key`
  - **Propósito:** Es la clave privada correspondiente al certificado del API server, utilizada para firmar y asegurar las comunicaciones.

- **Service Account Key Pair (sa.key, sa.pub)**
  - **Ubicación:** `/etc/kubernetes/pki/sa.key`, `/etc/kubernetes/pki/sa.pub`
  - **Propósito:** Utilizados por el API server y el controlador de cuentas de servicio para firmar y verificar los tokens de servicio dentro del clúster.

- **Etcd Certificates**

  - **Etcd Server Certificate (etcd.crt)**
    - **Ubicación:** `/etc/kubernetes/pki/etcd/etcd.crt`
    - **Propósito:** Asegura la comunicación dentro del clúster de etcd en los nodos Master.

  - **Etcd Server Key (etcd.key)**
    - **Ubicación:** `/etc/kubernetes/pki/etcd/etcd.key`
    - **Propósito:** Es la clave privada utilizada por el servidor etcd para firmar y asegurar las comunicaciones.

  - **Etcd CA Certificate (etcd/ca.crt)**
    - **Ubicación:** `/etc/kubernetes/pki/etcd/ca.crt`
    - **Propósito:** Autoridad de certificación para los certificados utilizados en el clúster de etcd.

- **API Server Etcd Client Certificates**

  - **API Server Etcd Client Certificate (apiserver-etcd-client.crt)**
    - **Ubicación:** `/etc/kubernetes/pki/apiserver-etcd-client.crt`
    - **Propósito:** Utilizado por el API server para autenticarse con el clúster de etcd.

  - **API Server Etcd Client Key (apiserver-etcd-client.key)**
    - **Ubicación:** `/etc/kubernetes/pki/apiserver-etcd-client.key`
    - **Propósito:** Clave privada utilizada por el API server para autenticarse con el clúster de etcd.

## 3. Certificados Específicos para el Nodo Bootstrap

El nodo Bootstrap es utilizado para iniciar el clúster y luego su rol puede ser reemplazado por los nodos Master. Inicialmente, requiere los mismos certificados que los Masters, pero una vez que el clúster está configurado, su rol puede cambiar.

- **Inicialmente:**
  - Los mismos certificados que los nodos Master:
    - API Server Certificate (apiserver.crt)
    - API Server Key (apiserver.key)
    - Etcd Certificates (etcd.crt, etcd.key, etcd/ca.crt)
    - API Server Etcd Client Certificates (apiserver-etcd-client.crt, apiserver-etcd-client.key)

- **Posteriormente:**
  - Puede ser eliminado o reintegrado con los certificados básicos si sigue en operación con un rol diferente:
    - CA Certificate (ca.crt)
    - Kubelet Certificate (kubelet.crt)
    - Kubelet Key (kubelet.key)

## Resumen de Certificados por Nodo

- **Todos los Nodos (Masters, Workers, Bootstrap):**
  - ca.crt
  - kubelet.crt, kubelet.key

- **Nodos Master (además de los anteriores):**
  - apiserver.crt, apiserver.key
  - sa.key, sa.pub
  - etcd.crt, etcd.key, etcd/ca.crt
  - apiserver-etcd-client.crt, apiserver-etcd-client.key

- **Nodo Bootstrap (al iniciar el clúster):**
  - Inicialmente: Los mismos certificados que los nodos Master.
  - Posteriormente: Puede ser eliminado o reintegrado con los certificados básicos (ca.crt, kubelet.crt, kubelet.key) si sigue en operación con un rol diferente.

Este esquema de clasificación y distribución asegura que cada nodo tenga los certificados necesarios para su función en el clúster Kubernetes.

## Estructura de Archivos de Configuración en un Clúster Kubernetes

1. /etc/kubernetes/kubelet-config.yaml
yaml
Copiar código
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  x509:
    clientCAFile: "/etc/kubernetes/pki/ca.crt"
authorization:
  mode: Webhook
serverTLSBootstrap: true
tlsCertFile: "/etc/kubernetes/pki/kubelet.crt"
tlsPrivateKeyFile: "/etc/kubernetes/pki/kubelet.key"
cgroupDriver: systemd
runtimeRequestTimeout: "15m"
containerRuntimeEndpoint: "unix:///var/run/crio/crio.sock"
2. /etc/kubernetes/manifests/kube-apiserver.yaml
yaml
Copiar código
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - name: kube-apiserver
    image: k8s.gcr.io/kube-apiserver:v1.20.0
    command:
    - kube-apiserver
    - --advertise-address=10.17.4.21
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://10.17.4.21:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    volumeMounts:
    - mountPath: /etc/kubernetes/pki
      name: pki
      readOnly: true
  volumes:
  - name: pki
    hostPath:
      path: /etc/kubernetes/pki
3. /etc/systemd/system/crio.service
ini
Copiar código
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
4. /etc/systemd/system/kubelet.service
ini
Copiar código
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/
Wants=crio.service
After=crio.service

[Service]
ExecStart=/opt/bin/kubelet --config=/etc/kubernetes/kubelet-config.yaml --kubeconfig=/etc/kubernetes/kubelet.conf
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target
5. /etc/systemd/system/etcd.service
ini
Copiar código
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
After=network.target

[Service]
User=etcd
Type=notify
Environment="ETCD_DATA_DIR=/var/lib/etcd"
Environment="ETCD_NAME=etcd0"
Environment="ETCD_INITIAL_ADVERTISE_PEER_URLS=https://10.17.4.21:2380"
Environment="ETCD_LISTEN_PEER_URLS=https://10.17.4.21:2380"
Environment="ETCD_LISTEN_CLIENT_URLS=https://10.17.4.21:2379,https://127.0.0.1:2379"
Environment="ETCD_ADVERTISE_CLIENT_URLS=https://10.17.4.21:2379"
Environment="ETCD_INITIAL_CLUSTER=etcd0=https://10.17.4.21:2380"
Environment="ETCD_INITIAL_CLUSTER_STATE=new"
Environment="ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster"
Environment="ETCD_CERT_FILE=/etc/kubernetes/pki/etcd/etcd.crt"
Environment="ETCD_KEY_FILE=/etc/kubernetes/pki/etcd/etcd.key"
Environment="ETCD_TRUSTED_CA_FILE=/etc/kubernetes/pki/etcd/ca.crt"
Environment="ETCD_CLIENT_CERT_AUTH=true"
Environment="ETCD_PEER_CERT_FILE=/etc/kubernetes/pki/etcd/etcd.crt"
Environment="ETCD_PEER_KEY_FILE=/etc/kubernetes/pki/etcd/etcd.key"
Environment="ETCD_PEER_TRUSTED_CA_FILE=/etc/kubernetes/pki/etcd/ca.crt"
Environment="ETCD_PEER_CLIENT_CERT_AUTH=true"
ExecStart=/opt/bin/etcd
Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
6. /etc/systemd/system/kube-apiserver.service
ini
Copiar código
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
7. /etc/systemd/system/kube-controller-manager.service
ini
Copiar código
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://kubernetes.io/docs/concepts/overview/components/
After=network.target

[Service]
ExecStart=/opt/bin/kube-controller-manager \
  --kubeconfig=/etc/kubernetes/controller-manager.conf \
  --bind-address=0.0.0.0 \
  --leader-elect=true \
  --use-service-account-credentials=true \
  --controllers=*,bootstrapsigner,tokencleaner \
  --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt \
  --cluster-signing-key-file=/etc/kubernetes/pki/ca.key \
  --root-ca-file=/etc/kubernetes/pki/ca.crt \
  --service-account-private-key-file=/etc/kubernetes/pki/sa.key \
  --cluster-name=kubernetes \
  --cluster-cidr=10.244.0.0/16 \
  --allocate-node-cidrs=true \
  --node-cidr-mask-size=24 \
  --service-cluster-ip-range=10.96.0.0/12 \
  --v=2
Restart=on-failure

[Install]
WantedBy=multi-user.target
8. /etc/systemd/system/kube-scheduler.service
ini
Copiar código
[Unit]
Description=Kubernetes Scheduler
Documentation=https://kubernetes.io/docs/concepts/overview/components/
After=network.target

[Service]
ExecStart=/opt/bin/kube-scheduler   --address=127.0.0.1   --kubeconfig=/etc/kubernetes/scheduler.conf   --leader-elect=true   --v=2
Restart=on-failure

[Install]
WantedBy=multi-user.target
9. /etc/systemd/system/set-hosts.service
ini
Copiar código
[Unit]
Description=Set /etc/hosts file
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'echo "127.0.0.1   localhost" > /etc/hosts; echo "::1         localhost" >> /etc/hosts; echo "10.17.4.21  master1.cefaslocalserver.com master1" >> /etc/hosts'
RemainAfterExit=true

[Install]
WantedBy=multi-user.target

Esta estructura te permitirá identificar rápidamente dónde se encuentra cada configuración y facilita la gestión de los archivos relacionados con tu clúster Kubernetes.





_

sudo systemctl restart etcd
sudo systemctl restart kube-apiserver
sudo systemctl restart kube-controller-manager
sudo systemctl restart kube-scheduler
sudo systemctl restart kubelet
sudo systemctl restart kube-proxy
sudo systemctl restart crio



sudo systemctl status etcd
sudo systemctl status kube-apiserver
sudo systemctl status kube-controller-manager
sudo systemctl status kube-scheduler
sudo systemctl status kubelet
sudo systemctl status kube-proxy
sudo systemctl status crio
