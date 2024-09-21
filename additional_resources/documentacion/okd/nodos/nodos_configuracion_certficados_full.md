1. Certificado de la Autoridad Certificadora (CA)
Archivos Generados:
Clave privada de la CA: /etc/kubernetes/pki/ca.key
Certificado de la CA: /etc/kubernetes/pki/ca.crt
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para la CA
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/ca.key -pkeyopt rsa_keygen_bits:2048

# Crear el certificado de la CA
sudo openssl req -x509 -new -key /etc/kubernetes/pki/ca.key -subj "/CN=Kubernetes-CA" -days 3650 -out /etc/kubernetes/pki/ca.crt
2. Certificado para el Admin (kubernetes-admin)
Archivos Generados:
Clave privada del Admin: /etc/kubernetes/pki/admin.key
CSR del Admin: /etc/kubernetes/pki/admin.csr
Certificado del Admin: /etc/kubernetes/pki/admin.crt
Archivo de configuración OpenSSL: /etc/kubernetes/pki/admin-openssl.cnf
Archivo de Configuración (/etc/kubernetes/pki/admin-openssl.cnf):
ini
Copiar código
[ req ]
req_extensions = v3_req
distinguished_name = req_distinguished_name
prompt = no

[ req_distinguished_name ]
CN = kubernetes-admin
O = system:masters

[ v3_req ]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = clientAuth
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para el Admin
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/admin.key -pkeyopt rsa_keygen_bits:2048

# Crear el CSR para el Admin
sudo openssl req -new -key /etc/kubernetes/pki/admin.key -out /etc/kubernetes/pki/admin.csr -config /etc/kubernetes/pki/admin-openssl.cnf

# Firmar el CSR con la CA para obtener el certificado Admin
sudo openssl x509 -req -in /etc/kubernetes/pki/admin.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/admin.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/admin-openssl.cnf
3. Certificado para el API Server
Archivos Generados:
Clave privada del API Server: /etc/kubernetes/pki/apiserver.key
CSR del API Server: /etc/kubernetes/pki/apiserver.csr
Certificado del API Server: /etc/kubernetes/pki/apiserver.crt
Archivo de configuración OpenSSL: /etc/kubernetes/pki/
Archivo de Configuración (/etc/kubernetes/pki/):
ini
Copiar código
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
req_extensions     = v3_req

[ dn ]
CN = kube-apiserver

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = kube-apiserver
DNS.2 = kube-apiserver.kube-system
DNS.3 = master1
DNS.4 = master1.cefaslocalserver.com
IP.1 = 127.0.0.1
IP.2 = <MASTER1_IP_ADDRESS>
IP.3 = <CLUSTER_IP_ADDRESS>
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para el API Server
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/apiserver.key -pkeyopt rsa_keygen_bits:2048

# Crear el CSR para el API Server
sudo openssl req -new -key /etc/kubernetes/pki/apiserver.key -out /etc/kubernetes/pki/apiserver.csr -config /etc/kubernetes/pki/

# Firmar el CSR con la CA para obtener el certificado del API Server
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/
4. Certificado para el Kubelet
Archivos Generados:
Clave privada del Kubelet: /etc/kubernetes/pki/kubelet.key
CSR del Kubelet: /etc/kubernetes/pki/kubelet.csr
Certificado del Kubelet: /etc/kubernetes/pki/kubelet.crt
Archivo de configuración OpenSSL: /tmp/kubelet-openssl.cnf (o puedes guardarlo en /etc/kubernetes/pki/kubelet-openssl.cnf)
Archivo de Configuración (/tmp/kubelet-openssl.cnf):
ini
Copiar código
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
req_extensions     = req_ext
distinguished_name = dn

[ dn ]
C  = US
ST = CA
L  = San Francisco
O  = Kubernetes
OU = System
CN = system:node:master1.cefaslocalserver.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = master1.cefaslocalserver.com
IP.1 = 10.17.4.21
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para el Kubelet
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/kubelet.key -pkeyopt rsa_keygen_bits:2048

# Crear el CSR para el Kubelet
sudo openssl req -new -key /etc/kubernetes/pki/kubelet.key -out /etc/kubernetes/pki/kubelet.csr -config /tmp/kubelet-openssl.cnf

# Firmar el CSR con la CA para obtener el certificado del Kubelet
sudo openssl x509 -req -in /etc/kubernetes/pki/kubelet.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kubelet.crt -days 365 -extensions req_ext -extfile /tmp/kubelet-openssl.cnf
5. Certificado para Kube-scheduler
Archivos Generados:
Clave privada del Kube-scheduler: /etc/kubernetes/pki/kube-scheduler.key
CSR del Kube-scheduler: /etc/kubernetes/pki/kube-scheduler.csr
Certificado del Kube-scheduler: /etc/kubernetes/pki/kube-scheduler.crt
Archivo de configuración OpenSSL: /etc/kubernetes/pki/kube-scheduler-openssl.cnf
Archivo de Configuración (/etc/kubernetes/pki/kube-scheduler-openssl.cnf):
ini
Copiar código
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
req_extensions     = v3_req

[ dn ]
CN = system:kube-scheduler

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para Kube-scheduler
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/kube-scheduler.key -pkeyopt rsa_keygen_bits:2048

# Crear el CSR para Kube-scheduler
sudo openssl req -new -key /etc/kubernetes/pki/kube-scheduler.key -out /etc/kubernetes/pki/kube-scheduler.csr -config /etc/kubernetes/pki/kube-scheduler-openssl.cnf

# Firmar el CSR con la CA para obtener el certificado del Kube-scheduler
sudo openssl x509 -req -in /etc/kubernetes/pki/kube-scheduler.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kube-scheduler.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/kube-scheduler-openssl.cnf
6. Certificado para Kube-proxy
Archivos Generados:
Clave privada del Kube-proxy: /etc/kubernetes/pki/kube-proxy.key
CSR del Kube-proxy: /etc/kubernetes/pki/kube-proxy.csr
Certificado del Kube-proxy: /etc/kubernetes/pki/kube-proxy.crt
Archivo de configuración OpenSSL: /etc/kubernetes/pki/kube-proxy-openssl.cnf
Archivo de Configuración (/etc/kubernetes/pki/kube-proxy-openssl.cnf):
ini
Copiar código
[ req ]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
req_extensions     = v3_req

[ dn ]
CN = system:kube-proxy

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para Kube-proxy
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/kube-proxy.key -pkeyopt rsa_keygen_bits:2048

# Crear el CSR para Kube-proxy
sudo openssl req -new -key /etc/kubernetes/pki/kube-proxy.key -out /etc/kubernetes/pki/kube-proxy.csr -config /etc/kubernetes/pki/kube-proxy-openssl.cnf

# Firmar el CSR con la CA para obtener el certificado del Kube-proxy
sudo openssl x509 -req -in /etc/kubernetes/pki/kube-proxy.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kube-proxy.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/kube-proxy-openssl.cnf
7. Certificado para el Servidor Etcd
Archivos Generados:
Clave privada de Etcd: /etc/kubernetes/pki/etcd/etcd.key
CSR de Etcd: /etc/kubernetes/pki/etcd/etcd.csr
Certificado de Etcd: /etc/kubernetes/pki/etcd/etcd.crt
Archivo de configuración OpenSSL: /etc/kubernetes/pki/etcd/etcd-openssl.cnf
Archivo de Configuración (/etc/kubernetes/pki/etcd/etcd-openssl.cnf):
ini
Copiar código
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
prompt = no

[req_distinguished_name]
CN = etcd

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[alt_names]
IP.1 = 127.0.0.1
IP.2 = 10.17.4.21
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para Etcd
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/etcd/etcd.key -pkeyopt rsa_keygen_bits:2048

# Crear el CSR para Etcd
sudo openssl req -new -key /etc/kubernetes/pki/etcd/etcd.key -out /etc/kubernetes/pki/etcd/etcd.csr -config /etc/kubernetes/pki/etcd/etcd-openssl.cnf

# Firmar el CSR con la CA para obtener el certificado de Etcd
sudo openssl x509 -req -in /etc/kubernetes/pki/etcd/etcd.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/etcd/etcd.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/etcd/etcd-openssl.cnf
8. Certificado para el Cliente del API Server de Etcd
Archivos Generados:
Clave privada del Cliente API Server Etcd: /etc/kubernetes/pki/apiserver-etcd-client.key
CSR del Cliente API Server Etcd: /etc/kubernetes/pki/apiserver-etcd-client.csr
Certificado del Cliente API Server Etcd: /etc/kubernetes/pki/apiserver-etcd-client.crt
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para el cliente API Server Etcd
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/apiserver-etcd-client.key -pkeyopt rsa_keygen_bits:2048

# Crear el CSR para el cliente API Server Etcd
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-etcd-client.key -subj "/CN=apiserver-etcd-client" -out /etc/kubernetes/pki/apiserver-etcd-client.csr

# Firmar el CSR con la CA para obtener el certificado del cliente API Server Etcd
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-etcd-client.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-etcd-client.crt -days 365
9. Certificado para el Cliente Kubelet del API Server
Archivos Generados:
Clave privada del Cliente API Server Kubelet: /etc/kubernetes/pki/apiserver-kubelet-client.key
CSR del Cliente API Server Kubelet: /etc/kubernetes/pki/apiserver-kubelet-client.csr
Certificado del Cliente API Server Kubelet: /etc/kubernetes/pki/apiserver-kubelet-client.crt
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para el cliente API Server Kubelet
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/apiserver-kubelet-client.key -pkeyopt rsa_keygen_bits:2048

# Crear el CSR para el cliente API Server Kubelet
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-kubelet-client.key -subj "/CN=kube-apiserver-kubelet-client" -out /etc/kubernetes/pki/apiserver-kubelet-client.csr

# Firmar el CSR con la CA para obtener el certificado del cliente API Server Kubelet
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-kubelet-client.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-kubelet-client.crt -days 365
10. Certificado para la Clave de la Cuenta de Servicio (Service Account)
Archivos Generados:
Clave privada de la Service Account: /etc/kubernetes/pki/sa.key
Clave pública de la Service Account: /etc/kubernetes/pki/sa.pub
Comandos para Generar:
bash
Copiar código
# Crear la clave privada para la Service Account
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/sa.key -pkeyopt rsa_keygen_bits:2048

# Crear la clave pública para la Service Account
sudo openssl rsa -in /etc/kubernetes/pki/sa.key -pubout -out /etc/kubernetes/pki/sa.pub
Conclusión
Siguiendo estos pasos, generarás todos los certificados necesarios para un clúster de Kubernetes, organizados por servicio. Asegúrate de ajustar las configuraciones específicas, como las direcciones IP y los nombres DNS, para que coincidan con tu entorno. Cada certificado se almacena en la ruta correspondiente en /etc/kubernetes/pki/, y es crucial asegurarse de que los permisos de estos archivos sean configurados correctamente para mantener la seguridad del clúster.