Aquí tienes cómo hacerlo:

bash
Copiar código
# Generar CSR para etcd
openssl req -new -key /etc/kubernetes/pki/etcd/etcd.key -out /etc/kubernetes/pki/etcd/etcd.csr -subj "/CN=etcd" -config /etc/kubernetes/pki/etcd/etcd-openssl.cnf

# Firmar el CSR con la CA
openssl x509 -req -in /etc/kubernetes/pki/etcd/etcd.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/etcd/etcd.crt -days 365 -extensions v3_req -extfile /etc/kubernetes/pki/etcd/etcd-openssl.cnf

# Generar CSR para apiserver-etcd-client
openssl req -new -key /etc/kubernetes/pki/apiserver-etcd-client.key -out /etc/kubernetes/pki/apiserver-etcd-client.csr -subj "/CN=kube-apiserver"

# Firmar el CSR con la CA
openssl x509 -req -in /etc/kubernetes/pki/apiserver-etcd-client.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-etcd-client.crt -days 365
Reiniciar los servicios:

Después de haber regenerado y firmado correctamente los certificados, reinicia los servicios de etcd y kube-apiserver para aplicar los cambios:
bash
Copiar código
sudo systemctl daemon-reload
sudo systemctl restart etcd
sudo systemctl restart kube-apiserver
Verificar los certificados:

Una vez regenerados los certificados y reiniciados los servicios, verifica nuevamente los certificados con openssl verify para asegurarte de que están firmados correctamente por la CA.
bash
Copiar código
sudo openssl verify -CAfile /etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/etcd/etcd.crt
sudo openssl verify -CAfile /etc/kubernetes/pki/ca.crt /etc/kubernetes/pki/apiserver-e