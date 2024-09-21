# Instalación del Servicio etcd

-----------------------------------------

## 1. Crear el servicio etcd

Primero, definimos el archivo de servicio para `etcd`:

Archivo: `cat /etc/systemd/system/etcd.service`

```bash
sudo vim /etc/systemd/system/etcd.service
```

```bash
sudo tee /etc/systemd/system/etcd.service <<EOF
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
After=network.target

[Service]
User=etcd
Type=notify
Environment="ETCD_DATA_DIR=/var/lib/etcd"
Environment="ETCD_NAME=etcd0"
Environment="ETCD_INITIAL_ADVERTISE_PEER_URLS=https://10.17.4.23:2380"
Environment="ETCD_LISTEN_PEER_URLS=https://10.17.4.23:2380"
Environment="ETCD_LISTEN_CLIENT_URLS=https://10.17.4.23:2379,https://127.0.0.1:2379"
Environment="ETCD_ADVERTISE_CLIENT_URLS=https://10.17.4.23:2379"
Environment="ETCD_INITIAL_CLUSTER=etcd0=https://10.17.4.23:2380"
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
EOF
```


## 2. Crear el servicio etcd

Sigue estos pasos para generar los certificados y claves necesarios para etcd:

**Crear el directorio de certificados:**

```bash
sudo mkdir -p /etc/kubernetes/pki/etcd
cd /etc/kubernetes/pki/etcd
```

**Generar la clave y certificado de la CA:**

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/etcd/ca.key -pkeyopt rsa_keygen_bits:2048

sudo openssl req -x509 -new -nodes -key /etc/kubernetes/pki/etcd/ca.key -subj "/CN=etcd-ca" -days 3650 -out /etc/kubernetes/pki/etcd/ca.crt
```

**Generar la clave privada y la solicitud de firma (CSR) para etcd:**

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/etcd/etcd.key -pkeyopt rsa_keygen_bits:2048

sudo openssl req -new -key /etc/kubernetes/pki/etcd/etcd.key -subj "/CN=etcd-server" -out /etc/kubernetes/pki/etcd/etcd.csr
```

**Crear archivo de configuración para OpenSSL:**

```bash
sudo tee /etc/kubernetes/pki/etcd/etcd-openssl.cnf <<EOF
[ v3_req ]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = etcd
DNS.2 = etcd.local
IP.1 = 127.0.0.1
IP.2 = 10.17.4.27
EOF
```

**Firmar el CSR con la CA:**

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/etcd/etcd.csr -CA /etc/kubernetes/pki/etcd/ca.crt -CAkey /etc/kubernetes/pki/etcd/ca.key -CAcreateserial -out /etc/kubernetes/pki/etcd/etcd.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/etcd/etcd-openssl.cnf
```

**Cambiar permisos:**

```bash
sudo chown etcd:etcd /etc/kubernetes/pki/etcd/*
sudo chmod 600 /etc/kubernetes/pki/etcd/*.key
sudo chmod 644 /etc/kubernetes/pki/etcd/*.crt
```


```bash
ls -ld /var/lib/etcd
sudo chown -R etcd:etcd /var/lib/etcd
```


## 3. Iniciar y verificar el servicio etcd

Recargar el daemon de systemd y reiniciar el servicio `etcd`:

```bash
sudo systemctl daemon-reload
sudo systemctl start etcd
sudo systemctl enable etcd
```

Verificar el estado del servicio `etcd`:

```bash
sudo systemctl status etcd
```


4. Verificar los logs de etcd
 
Para asegurar que no haya errores, revisa los logs del servicio:

```bash
sudo journalctl -u etcd -f
```



sudo chmod 600 /etc/kubernetes/pki/*.key /etc/kubernetes/pki/etcd/*.key
sudo chmod 644 /etc/kubernetes/pki/*.crt /etc/kubernetes/pki/etcd/*.crt
sudo chown etcd:etcd /etc/kubernetes/pki/etcd/*.key /etc/kubernetes/pki/etcd/*.crt
sudo chown etcd:etcd /etc/kubernetes/pki/apiserver-etcd-client.crt /etc/kubernetes/pki/apiserver-etcd-client.key
