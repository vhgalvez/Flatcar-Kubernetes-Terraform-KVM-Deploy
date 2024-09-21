# Instalación del Servicio etcd

## 1. Crear el servicio etcd

Primero, definimos el archivo de servicio para `etcd`:

```bash
sudo vim /etc/systemd/system/etcd.service
```

Contenido del archivo `etcd.service`:

```bash
[Unit]
Description=etcd
Documentation=https://github.com/coreos/etcd
After=network.target

[Service]
User=etcd
Type=notify
Environment="ETCD_DATA_DIR=/var/lib/etcd"
Environment="ETCD_NAME=etcd0"
Environment="ETCD_INITIAL_ADVERTISE_PEER_URLS=https://10.17.4.22:2380"
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
```

## 2. Generación de los certificados para etcd

Crea el directorio necesario para almacenar los certificados de etcd:

```bash
mkdir -p /etc/kubernetes/pki/etcd

cd /etc/kubernetes/pki/etcd
```

### 2.1. Generar la clave privada y el certificado de la Autoridad Certificadora (CA)

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/etcd/ca.key -pkeyopt rsa_keygen_bits:2048
sudo openssl req -x509 -new -nodes -key /etc/kubernetes/pki/etcd/ca.key -subj "/CN=etcd-ca" -days 3650 -out /etc/kubernetes/pki/etcd/ca.crt
```


### 2.2. Generar la clave privada del servidor etcd

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/etcd/etcd.key -pkeyopt rsa_keygen_bits:2048
```

### 2.3. Crear la solicitud de firma de certificado (CSR) para etcd

```bash
sudo openssl req -new -key /etc/kubernetes/pki/etcd/etcd.key -subj "/CN=etcd-server" -out /etc/kubernetes/pki/etcd/etcd.csr
```


### 2.4. Crear el archivo de configuración de OpenSSL para etcd

Genera el archivo de configuración etcd-openssl.cnf con las extensiones requeridas:

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
IP.2 = 10.17.4.22
EOF
```

### 2.5. Firmar el CSR con la CA para generar el certificado de etcd


```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/etcd/etcd.csr -CA /etc/kubernetes/pki/etcd/ca.crt -CAkey /etc/kubernetes/pki/etcd/ca.key -CAcreateserial -out /etc/kubernetes/pki/etcd/etcd.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/etcd/etcd-openssl.cnf
```

### 2.6. Cambiar los permisos de los certificados

```bash
sudo chown etcd:etcd /etc/kubernetes/pki/etcd/*.*
sudo chmod 600 /etc/kubernetes/pki/etcd/*.key
sudo chmod 644 /etc/kubernetes/pki/etcd/*.crt

sudo chown etcd:etcd *.*
sudo chmod 600 etcd.key
sudo chmod 644 etcd.crt ca.crt
```


## 3. Iniciar y Verificar el Servicio etcd

Una vez que los certificados han sido generados y el archivo de servicio está configurado, recarga systemd, inicia el servicio de etcd, y verifica su estado:

```bash
# Recargar systemd para reflejar los cambios
sudo systemctl daemon-reload

# Iniciar el servicio etcd
sudo systemctl start etcd
sudo systemctl restart etcd

# Habilitar el servicio para que se inicie en el arranque
sudo systemctl enable etcd

# Verificar el estado del servicio
sudo systemctl status etcd


# Ver los logs del servicio etcd
sudo journalctl -u etcd -f
```
