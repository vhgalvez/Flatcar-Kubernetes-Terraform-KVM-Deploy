# Tutorial para configurar kubelet y generar certificados en Kubernetes con OpenShift SDN

## Paso 1: Revisión y ajuste de la configuración de kubelet

### 1.1 Verificar el estado del servicio kubelet

Revisa si el servicio kubelet está en ejecución. Si falla, continúa con los siguientes pasos.

```bash
sudo systemctl status kubelet
```

### 1.2 Verificar el archivo de configuración del kubelet

Revisa la configuración de kubelet para asegurarte de que sea correcta.

```bash
cat /etc/kubernetes/kubelet-config.yaml
```

El contenido debería ser similar a:

```yaml
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
featureGates:
  RotateKubeletServerCertificate: true
evictionHard:
  memory.available: "200Mi"
  nodefs.available: "10%"
  nodefs.inodesFree: "5%"
maxPods: 110
failSwapOn: false
cniConfDir: /etc/cni/net.d
cniBinDir: /opt/cni/bin
logging:
  format: json
```


### 1.3 Verificar la configuración de kubelet.conf

Este archivo debe incluir la IP del nodo maestro.

```bash
cat /etc/kubernetes/kubelet.conf
```

El archivo debe tener un contenido similar a este:

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://10.17.4.23:6443
  name: local
contexts:
- context:
    cluster: local
    user: kubelet
  name: local
current-context: local
users:
- name: kubelet
  user:
    client-certificate: /etc/kubernetes/pki/kubelet.crt
    client-key: /etc/kubernetes/pki/kubelet.key
```

## Paso 2: Crear el archivo kubeadm-flags.env

Si el archivo `/var/lib/kubelet/kubeadm-flags.env` no existe, créalo con el siguiente contenido:

```bash
sudo tee /var/lib/kubelet/kubeadm-flags.env <<EOF
KUBELET_KUBEADM_ARGS="--container-runtime=remote --container-runtime-endpoint=unix:///var/run/crio/crio.sock --fail-swap-on=false --cgroup-driver=systemd --max-pods=110 --eviction-hard=memory.available<200Mi,nodefs.available<10%,nodefs.inodesFree<5% --node-ip=10.17.4.23 --config=/etc/kubernetes/kubelet-config.yaml"
EOF
```

## Paso 3: Verificar la configuración de red de OpenShift SDN

Verifica que el archivo de configuración de red esté correctamente configurado.

Si estás utilizando Flannel:

```bash
cat /etc/cni/net.d/10-flannel.conf
Debe tener este contenido:

json
Copiar código
{
  "cniVersion": "0.3.1",
  "name": "cbr0",
  "type": "flannel",
  "delegate": {
    "isDefaultGateway": true
  }
}
```

Si estás utilizando OpenShift SDN, configura la red correspondiente. Para OpenShift SDN, verifica que el plugin SDN esté bien configurado.

## Paso 4: Generar los certificados necesarios para kubelet

### 4.1 Crear el archivo de configuración de la CSR para kubelet

```bash
sudo tee /etc/kubernetes/pki/kubelet-csr.conf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
prompt = no

[req_distinguished_name]
CN = system:node:master3.cefaslocalserver.com
O = system:nodes

[v3_req]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth
EOF
```

## 4.2 Generar la clave privada de kubelet

```bash
openssl genrsa -out /etc/kubernetes/pki/kubelet.key 2048
```

## 4.3 Generar la CSR (Solicitud de Firma de Certificado)

```bash
openssl req -new -key /etc/kubernetes/pki/kubelet.key -out /etc/kubernetes/pki/kubelet.csr -config /etc/kubernetes/pki/kubelet-csr.conf
```

### 4.4 Firmar el certificado con la CA del clúster

```bash
openssl x509 -req -in /etc/kubernetes/pki/kubelet.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kubelet.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/kubelet-csr.conf
```

## Paso 5: Reiniciar los servicios

Reinicia el servicio kubelet y asegúrate de que se recarguen los archivos de configuración.

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Revisa el estado después del reinicio:

```bash
sudo systemctl status kubelet
```

## Paso 6: Verificar los nodos en el clúster

Después de haber completado los pasos anteriores, revisa si el nodo aparece en el clúster:

```bash

sudo oc get nodes --kubeconfig=/etc/kubernetes/admin.conf
```

## Paso 7: Solución de problemas adicionales

Si el servicio kubelet sigue fallando, revisa los logs para obtener más detalles sobre el error:

```bash
sudo journalctl -xeu kubelet
```


cat /etc/kubernetes/kubelet-config.yaml
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





cat /etc/systemd/system/kubelet.service

[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=https://kubernetes.io/docs/
Wants=crio.service
After=crio.service

[Service]
ExecStart=/opt/bin/kubelet --config=/etc/kubernetes/kubelet-config.yaml --kubeconfig=/etc/kubernetes/kubelet.conf --hostname-override=master1.cefaslocalserver.com
Restart=always
StartLimitInterval=0
RestartSec=10

[Install]
WantedBy=multi-user.target



## Conclusión

Este tutorial cubre todos los pasos necesarios para configurar y solucionar problemas de kubelet en un nodo de Kubernetes con OpenShift SDN. Incluye la creación de los certificados necesarios, la configuración de los archivos de kubelet y el ajuste de la red con OpenShift SDN.




```bash
sudo tee /var/lib/kubelet/kubeadm-flags.env <<EOF
KUBELET_KUBEADM_ARGS="--container-runtime=remote --container-runtime-endpoint=unix:///var/run/crio/crio.sock --fail-swap-on=false --cgroup-driver=systemd --max-pods=110 --eviction-hard=memory.available<200Mi,nodefs.available<10%,nodefs.inodesFree<5% --node-ip=10.17.4.23 --config=/etc/kubernetes/kubelet-config.yaml"
EOF
```


hay usar OpenShift SDN, para OKD. OpenShift SDN es una red definida por software (SDN) que utiliza Open vSwitch (OVS) como su componente de red subyacente. OpenShift SDN crea una red de contenedores que permite la comunicación entre los pods en diferentes nodos del clúster. OpenShift SDN también proporciona una red de servicios que permite la comunicación entre los pods y los servicios en el clúster.


Para configurar OpenShift SDN, sigue estos pasos:

# Download and install CNI plugins
curl -L -o /tmp/cni-plugins.tgz https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz
sudo mkdir -p /opt/cni/bin
sudo tar -xzf /tmp/cni-plugins.tgz -C /opt/cni/bin
sudo rm -rf /tmp/cni-plugins.tgz





sudo chown root:root /etc/kubernetes/pki/*.crt /etc/kubernetes/pki/*.key
sudo chown etcd:etcd /etc/kubernetes/pki/etcd/*.crt /etc/kubernetes/pki/etcd/*.key



_____________________________________



```bash
oc get nodes --kubeconfig /etc/kubernetes/admin.conf
```

core@bootstrap ~ $ sudo etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.pem \
    --cert=/etc/kubernetes/pki/etcd/etcd-10.17.4.27.pem \
    --key=/etc/kubernetes/pki/etcd/etcd-10.17.4.27-key.pem \
    --endpoints=https://10.17.4.27:2379 \
    endpoint health
https://10.17.4.27:2379 is healthy: successfully committed proposal: took = 94.170665ms


core@bootstrap ~ $ https://10.17.4.27:2379 is healthy: successfully committed proposal: took = 94.170665ms
-bash: https://10.17.4.27:2379: No such file or directory
core@bootstrap ~ $


sudo chmod 644 /etc/kubernetes/pki/*.pem
sudo chmod 600 /etc/kubernetes/pki/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/etcd/*.pem
sudo chmod 600 /etc/kubernetes/pki/etcd/*-key.pem


sudo journalctl -u kube-apiserver -f

core@bootstrap ~ $ sudo etcdctl --endpoints=https://10.17.4.27:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.pem \
  --cert=/etc/kubernetes/pki/etcd/etcd-10.17.4.27.pem \
  --key=/etc/kubernetes/pki/etcd/etcd-10.17.4.27-key.pem \
  endpoint health
https://10.17.4.27:2379 is healthy: successfully committed proposal: took = 125.476333ms
core@bootstrap ~ $



core@bootstrap ~ $ sudo oc create clusterrolebinding kubelet-bootstrap \
    --clusterrole=system:node-bootstrapper \
    --user=system:bootstrap \
    --kubeconfig=/etc/kubernetes/admin.conf
clusterrolebinding.rbac.authorization.k8s.io/kubelet-bootstrap created
core@bootstrap ~ $

core@bootstrap ~ $ sudo oc get clusterrolebinding kubelet-bootstrap --kubeconfig=/etc/kubernetes/admin.conf
NAME                ROLE                                   AGE
kubelet-bootstrap   ClusterRole/system:node-bootstrapper   3m15s
core@bootstrap ~ $


sudo chmod 644 /etc/kubernetes/pki/*.pem
sudo chmod 600 /etc/kubernetes/pki/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/etcd/*.pem
sudo chmod 600 /etc/kubernetes/pki/etcd/*-key.pem


/opt/bin/openshift-install version

openshift-install version



sudo chmod 600 /etc/kubernetes/pki/kubelet-bootstrap-key.pem
sudo chown root:root /etc/kubernetes/pki/kubelet-bootstrap-key.pem


sudo chmod 600 /etc/kubernetes/pki/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/*.pem

chmod 600 /etc/kubernetes/pki/*-key.pem


sudo chown -R etcd:etcd /var/lib/etcd
sudo chmod -R 700 /var/lib/etcd

sudo ls -l /var/lib/etcd
sudo chown -R etcd:etcd /var/lib/etcd
sudo chmod -R 700 /var/lib/etcd

sudo chmod 600 /etc/kubernetes/pki/etcd/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/etcd/*.pem
sudo chown etcd:etcd /etc/kubernetes/pki/etcd/*

sudo chmod 600 /etc/kubernetes/pki/etcd/*.pem
sudo chown -R etcd:etcd /etc/kubernetes/pki/etcd

sudo chmod 600 /etc/kubernetes/pki/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/*.pem


sudo chmod 600 /etc/kubernetes/pki/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/*.pem


sudo chmod 600 /etc/kubernetes/pki/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/*.pem


sudo chmod 600 /etc/kubernetes/pki/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/*.pem


sudo chmod 600 /etc/kubernetes/pki/*-key.pem
sudo chmod 644 /etc/kubernetes/pki/*.pem
