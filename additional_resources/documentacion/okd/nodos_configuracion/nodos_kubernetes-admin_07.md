# Guía para la Generación de Certificados y Configuración de kubernetes-admin en Kubernetes

Esta guía cubre los pasos necesarios para generar un nuevo par de certificados para el usuario `kubernetes-admin`, configurar el acceso al clúster y verificar la conectividad del servicio `kube-proxy`.

## 1. Generar la Clave Privada para kubernetes-admin

El primer paso es generar una clave privada para el usuario `kubernetes-admin`. Esta clave será necesaria para la autenticación con el clúster de Kubernetes.

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/admin.key -pkeyopt rsa_keygen_bits:2048
```


## 2. Crear la Solicitud de Firma de Certificado (CSR)

Con la clave privada generada, es necesario crear una Solicitud de Firma de Certificado (CSR). El CSR será firmado por la autoridad certificadora (CA) del clúster para generar el certificado del usuario `kubernetes-admin`.


```bash
sudo openssl req -new -key /etc/kubernetes/pki/admin.key -out /etc/kubernetes/pki/admin.csr -subj "/CN=kubernetes-admin/O=system:masters"
```


## 3. Firmar el CSR con la CA de Kubernetes

A continuación, firma el CSR utilizando la CA del clúster de Kubernetes para obtener el certificado del usuario `kubernetes-admin`.

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/admin.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/admin.crt -days 365
```

## 4. Ajustar los Permisos de los Archivos Generados y la Propiedad del Archivo

Es importante ajustar los permisos de los archivos generados para asegurar que la clave privada esté protegida y el certificado sea accesible.


Ajustar los permisos de los archivos generados

```bash
sudo chmod 600 /etc/kubernetes/pki/admin.key
sudo chmod 644 /etc/kubernetes/pki/admin.crt
```

Ajustar la propiedad del archivo

```bash
sudo chmod 644 /etc/kubernetes/pki/admin.key
sudo chown core:core /etc/kubernetes/pki/admin.key
```

Ajustar la propiedad del directorio

```bash
sudo chmod 755 /etc/kubernetes
sudo chmod 755 /etc/kubernetes/pki
sudo chown root:root /etc/kubernetes
sudo chown root:root /etc/kubernetes/pki
```

## 5. Configurar el Archivo kubeconfig para kubernetes-admin

Una vez generados los certificados, es necesario configurar el archivo kubeconfig para que el usuario `kubernetes-admin` pueda conectarse al clúster de Kubernetes.


Edita el archivo `/etc/kubernetes/admin.conf` o crea uno nuevo si no existe:

```bash
sudo vim /etc/kubernetes/admin.conf
```

Agrega el siguiente contenido al archivo `admin.conf`:

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://10.17.4.23:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate: /etc/kubernetes/pki/admin.crt
    client-key: /etc/kubernetes/pki/admin.key
```

1. Probar la Conexión al Clúster
   
Una vez que los certificados estén configurados y el archivo kubeconfig esté en su lugar, puedes probar la conexión al clúster con el siguiente comando:

```bash
oc get nodes --kubeconfig /etc/kubernetes/admin.conf
```

Si todo está correctamente configurado, deberías ver los nodos del clúster listados.


## 7. Verificar la Conectividad con el Servidor API de Kubernetes

Después de haber configurado `kube-proxy` y generado los certificados, verifica que `kube-proxy` esté conectado correctamente al servidor API de Kubernetes:

## 8. Crear un ClusterRoleBinding para kube-proxy

Para asegurarte de que `kube-proxy` tenga los permisos necesarios para interactuar con el API de Kubernetes, debes crear un `ClusterRoleBinding`. Esto le otorgará los permisos necesarios para interactuar con el clúster.

```bash
sudo oc --kubeconfig=/etc/kubernetes/admin.conf create clusterrolebinding kubelet-bootstrap --clusterrole=system:node --user=kubelet
```

```bash
core@master3 ~ $ sudo oc --kubeconfig=/etc/kubernetes/admin.conf create clusterrolebinding kubelet-bootstrap --clusterrole=system:node --user=kubelet
clusterrolebinding.rbac.authorization.k8s.io/kubelet-bootstrap created
```

## Conclusión

Siguiendo esta guía, habrás generado y configurado correctamente los certificados necesarios para el usuario kubernetes-admin. Además, te habrás asegurado de que kube-proxy tenga los permisos adecuados para interactuar con el servidor API de Kubernetes. Con esto, deberías poder gestionar y operar el clúster sin inconvenientes.
