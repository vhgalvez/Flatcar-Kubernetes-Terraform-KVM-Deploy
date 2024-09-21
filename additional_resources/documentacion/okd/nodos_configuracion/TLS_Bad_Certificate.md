# Solución para "TLS: Bad Certificate"

## Paso 1: Asegurarse de que el Certificado del Cliente para etcd sea el Correcto

El problema está relacionado con el certificado cliente (`apiserver-etcd-client.crt`) que `kube-apiserver` está usando para conectarse a `etcd`. Debes asegurarte de que este certificado ha sido firmado por la misma CA que está siendo utilizada por `etcd`.

### Verificar el Certificado del Cliente para etcd:

Asegúrate de que el certificado `apiserver-etcd-client.crt` está correctamente firmado por la CA de `etcd`. Si no es así, genera un nuevo certificado.

### Regenerar el Certificado del Cliente `apiserver-etcd-client` Usando la CA de `etcd`:

```bash
# Generar clave privada para el cliente apiserver-etcd-client
sudo openssl genrsa -out /etc/kubernetes/pki/apiserver-etcd-client.key 2048

# Crear una solicitud de firma de certificado (CSR)
sudo openssl req -new -key /etc/kubernetes/pki/apiserver-etcd-client.key -subj "/CN=apiserver-etcd-client" -out /etc/kubernetes/pki/apiserver-etcd-client.csr

# Firmar el CSR con la CA de etcd
sudo openssl x509 -req -in /etc/kubernetes/pki/apiserver-etcd-client.csr -CA /etc/kubernetes/pki/etcd/ca.crt -CAkey /etc/kubernetes/pki/etcd/ca.key -CAcreateserial -out /etc/kubernetes/pki/apiserver-etcd-client.crt -days 365
```


## Paso 2: Verificar los Certificados y las Rutas en kube-apiserver

El kube-apiserver debe utilizar los certificados correctos para conectarse a etcd. Asegúrate de que las rutas de los certificados en el archivo de servicio de kube-apiserver son correctas.

```bash
sudo vim /etc/systemd/system/kube-apiserver.service
```

Asegúrate de que las rutas para los archivos etcd-cafile, etcd-certfile y etcd-keyfile son correctas:

```bash
--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt \
--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt \
--etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key \
```

### Paso 3: Verificar los Permisos de los Certificados

Asegúrate de que los permisos de los archivos de certificados y claves son correctos para que los servicios puedan acceder a ellos.

```bash
# Cambiar permisos de los certificados y claves en etcd
sudo chown etcd:etcd /etc/kubernetes/pki/etcd/*
sudo chmod 600 /etc/kubernetes/pki/etcd/etcd.key
sudo chmod 644 /etc/kubernetes/pki/etcd/etcd.crt


# Cambiar permisos de los certificados y claves en kube-apiserver
sudo chmod 600 /etc/kubernetes/pki/apiserver-etcd-client.key
sudo chmod 644 /etc/kubernetes/pki/apiserver-etcd-client.crt
```

### Paso 4: Reiniciar los Servicios

Después de hacer los ajustes, recarga y reinicia los servicios:

```bash
# Recargar systemd
sudo systemctl daemon-reload

# Reiniciar etcd
sudo systemctl restart etcd

# Reiniciar kube-apiserver
sudo systemctl restart kube-apiserver

# Verificar los logs de etcd
sudo journalctl -u etcd -f

# Verificar los logs de kube-apiserver

sudo journalctl -u kube-apiserver -f
```

### Paso 5: Verificar la Conexión

Después de reiniciar los servicios, asegúrate de que kube-apiserver pueda conectarse correctamente a etcd revisando los logs de ambos servicios.

Verificar si la Conexión es Exitosa
Revisa los logs de kube-apiserver y asegúrate de que no hay más errores relacionados con TLS o certificados. Si el problema persiste, los errores relacionados con los certificados estarán más detallados en los logs.