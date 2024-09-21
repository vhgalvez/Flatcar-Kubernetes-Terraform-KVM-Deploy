Genera la clave privada para el cliente kubelet:

```bash
sudo openssl genrsa -out /etc/kubernetes/pki/apiserver-kubelet-client.key 2048
```

Crea la CSR (Certificate Signing Request):

```bash
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-kubelet-client.key -out /etc/kubernetes/pki/apiserver-kubelet-client.csr -subj="/CN=apiserver-kubelet-client"
```

Firma el certificado utilizando la CA (Certificate Authority):

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-kubelet-client.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-kubelet-client.crt -days 365
```

Después de generar estos archivos, asegúrate de ajustar los permisos correctamente:

```bash
sudo chmod 600 /etc/kubernetes/pki/apiserver-kubelet-client.key
sudo chmod 644 /etc/kubernetes/pki/apiserver-kubelet-client.crt
```