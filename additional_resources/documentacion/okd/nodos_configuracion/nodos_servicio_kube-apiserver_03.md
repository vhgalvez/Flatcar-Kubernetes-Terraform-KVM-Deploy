# Configuración del servicio kube-apiserver

-----------------------------------------------

Este archivo define el servicio kube-apiserver y cómo se comunica con etcd usando TLS.

Archivo: `cat /etc/systemd/system/kube-apiserver.service`

```bash
sudo vim /etc/systemd/system/kube-apiserver.service
```

```bash
sudo tee /etc/systemd/system/kube-apiserver.service <<EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://kubernetes.io/docs/concepts/overview/components/
After=network.target

[Service]
ExecStart=/opt/bin/kube-apiserver \
  --advertise-address=10.17.4.23 \
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
EOF
```

## 5. Certificados para kube-apiserver


**Generar nueva clave privada:**

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/apiserver.key -pkeyopt rsa_keygen_bits:2048
```

**Crear la solicitud de firma de certificado (CSR):**

```bash
sudo openssl req -new -key /etc/kubernetes/pki/apiserver.key -out /etc/kubernetes/pki/apiserver.csr -subj "/CN=kube-apiserver"
```

**Crear archivo de configuración de OpenSSL:**
    
```bash
sudo vim /etc/kubernetes/pki/v3_req.cnf
```

**Contenido del archivo de configuración:**


```bash
sudo tee /etc/kubernetes/pki/v3_req.cnf <<EOF
[ v3_req ]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.17.4.23
IP.2 = 10.96.0.1
EOF
```

**Firmar el CSR para generar el certificado:**

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/ca.key -pkeyopt rsa_keygen_bits:2048
```

```bash
sudo openssl req -x509 -new -key /etc/kubernetes/pki/ca.key -subj "/CN=Kubernetes-CA" -days 3650 -out /etc/kubernetes/pki/ca.crt
```

```bash
sudo chown etcd:etcd /etc/kubernetes/pki/ca.crt
sudo chmod 644 /etc/kubernetes/pki/ca.crt
sudo chown etcd:etcd /etc/kubernetes/pki/ca.key
sudo chmod 600 /etc/kubernetes/pki/ca.key
```

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/v3_req.cnf
```

## 6. Certificado de cliente para etcd


**Generar nueva clave privada para apiserver-etcd-client:**

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/apiserver-etcd-client.key -pkeyopt rsa_keygen_bits:2048
```

**Crear la CSR para el cliente apiserver-etcd-client:**

```bash
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-etcd-client.key -subj "/CN=apiserver-etcd-client" -out /etc/kubernetes/pki/apiserver-etcd-client.csr
```

**Firmar el CSR con la CA de etcd:**

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-etcd-client.csr -CA /etc/kubernetes/pki/etcd/ca.crt -CAkey /etc/kubernetes/pki/etcd/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-etcd-client.crt -days 365
```



**Verificar los permisos:**

```bash
sudo chmod 600 /etc/kubernetes/pki/apiserver-etcd-client.key
sudo chmod 644 /etc/kubernetes/pki/apiserver-etcd-client.crt
```


## 7. Certificados para kubelet

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/sa.key -pkeyopt rsa_keygen_bits:2048

sudo openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
```

```bash
sudo chown root:root /etc/kubernetes/pki/sa.key /etc/kubernetes/pki/sa.pub
sudo chmod 600 /etc/kubernetes/pki/sa.key
sudo chmod 644 /etc/kubernetes/pki/sa.pub
```


## 8. Reiniciar servicios

**Recarga y reinicia ambos servicios:**

```bash
sudo systemctl daemon-reload
sudo systemctl restart etcd
sudo systemctl restart kube-apiserver
```








**Verifica el estado de los servicios:**

```bash
sudo systemctl status kube-apiserver
```


## 9. Verificar los logs

Verifica que no haya errores de certificados en los logs:

```bash
sudo journalctl -u etcd -f
sudo journalctl -u kube-apiserver -f
```
 _____________________


## 10  Generar la clave privada para el cliente kubelet:

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/apiserver-kubelet-client.key -pkeyopt rsa_keygen_bits:2048
```

Crear la solicitud de firma de certificado (CSR):

```bash
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-kubelet-client.key -out /etc/kubernetes/pki/apiserver-kubelet-client.csr -subj="/CN=apiserver-kubelet-client"
```

Firmar el certificado utilizando la CA (Certificate Authority) del cluster:

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-kubelet-client.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-kubelet-client.crt -days 365
```

Verificar y ajustar los permisos:

```bash
sudo chmod 600 /etc/kubernetes/pki/apiserver-kubelet-client.key
sudo chmod 644 /etc/kubernetes/pki/apiserver-kubelet-client.crt
```

Reiniciar los servicios:

```bash 
sudo systemctl daemon-reload
sudo systemctl restart kube-apiserver
```