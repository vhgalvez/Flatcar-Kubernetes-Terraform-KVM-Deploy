El error que estás enfrentando ocurre porque el certificado de etcd no contiene una extensión SAN (Subject Alternative Name) que incluya la IP 127.0.0.1. Cuando se utiliza una IP para conectarse a etcd, el cliente (en este caso, kube-apiserver) verifica si la IP está incluida en los SANs del certificado. Al no encontrarla, la verificación falla.

Solución: Regenerar el Certificado de etcd con IP SAN
Para solucionar este problema, necesitas regenerar el certificado de etcd y asegurarte de incluir 127.0.0.1 como SAN en el certificado. Aquí te indico los pasos:

Crear un archivo de configuración de OpenSSL para generar el nuevo certificado con SAN:

Crea un archivo de configuración (por ejemplo, etcd-openssl.cnf):

bash
Copiar código
cat <<EOF | sudo tee /etc/kubernetes/pki/etcd-openssl.cnf
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
CN = etcd

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[alt_names]
IP.1 = 127.0.0.1
IP.2 = 10.17.4.21  # Reemplaza esto con la IP de tu nodo

EOF
Generar la clave privada:

bash
Copiar código
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/etcd/etcd.key -pkeyopt rsa_keygen_bits:2048
Generar el CSR (Certificate Signing Request):

bash
Copiar código
sudo openssl req -new -key /etc/kubernetes/pki/etcd/etcd.key -out /etc/kubernetes/pki/etcd/etcd.csr -config /etc/kubernetes/pki/etcd-openssl.cnf
Firmar el CSR con el CA de etcd para generar el nuevo certificado:

bash
Copiar código
sudo openssl x509 -req -in /etc/kubernetes/pki/etcd/etcd.csr -CA /etc/kubernetes/pki/etcd/ca.crt -CAkey /etc/kubernetes/pki/etcd/ca.key -CAcreateserial -out /etc/kubernetes/pki/etcd/etcd.crt -days 365 -sha256 -extensions v3_req -extfile /etc/kubernetes/pki/etcd-openssl.cnf
Reiniciar los servicios:

Después de regenerar el certificado, reinicia los servicios etcd, kube-apiserver, y kubelet:

bash
Copiar código
sudo systemctl restart etcd
sudo systemctl restart kube-apiserver
sudo systemctl restart kubelet
Verificar la conectividad:

Vuelve a probar la conexión para asegurarte de que todo esté funcionando correctamente:

bash
Copiar código
sudo curl --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/apiserver-etcd-client.crt --key /etc/kubernetes/pki/apiserver-etcd-client.key https://127.0.0.1:2379/health
Estos pasos deberían resolver el problema de TLS entre kube-apiserver y etcd, asegurando que la verificación del certificado se realice correctamente.