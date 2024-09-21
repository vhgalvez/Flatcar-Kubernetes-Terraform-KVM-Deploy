
# Creación y Configuración del Servicio kube-proxy con Certificados

Este tutorial te guiará a través de los pasos para generar certificados para kube-proxy, configurar su servicio con systemd, y verificar que se esté ejecutando correctamente en un clúster de Kubernetes.

## 1. Generar Certificados para kube-proxy

El kube-proxy requiere certificados para autenticar la comunicación con el servidor API de Kubernetes.

### 1.1 Generar una clave privada

Ejecuta el siguiente comando para generar una nueva clave privada para kube-proxy:

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/kube-proxy.key -pkeyopt rsa_keygen_bits:2048
```

### 1.2 Crear una Solicitud de Firma de Certificado (CSR)

Crea una CSR para el kube-proxy utilizando la clave privada generada anteriormente:

```bash
sudo openssl req -new -key /etc/kubernetes/pki/kube-proxy.key -subj "/CN=system:kube-proxy" -out /etc/kubernetes/pki/kube-proxy.csr
```

### 1.3 Firmar la CSR con la CA de Kubernetes

Usa la CA de Kubernetes para firmar la CSR y generar el certificado para kube-proxy:

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/kube-proxy.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kube-proxy.crt -days 365
```

### 1.4 Ajustar los permisos de los certificados

Es importante establecer los permisos correctos para garantizar que los archivos sean accesibles solo para los usuarios necesarios:

```bash
sudo chmod 600 /etc/kubernetes/pki/kube-proxy.key
sudo chmod 644 /etc/kubernetes/pki/kube-proxy.crt
```

## 2. Crear el Archivo de Configuración de kube-proxy

### 2.1 Crear el archivo de configuración kube-proxy-config.yaml

Crea el archivo de configuración para kube-proxy:

```bash
sudo vim /etc/kubernetes/kube-proxy-config.yaml
```

Añade el siguiente contenido:

```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
clientConnection:
  kubeconfig: "/etc/kubernetes/kube-proxy.kubeconfig"
mode: "iptables"
clusterCIDR: "10.244.0.0/16"
```

### 2.2 Crear el archivo kube-proxy.kubeconfig

Crea el archivo kubeconfig para kube-proxy:

```bash
sudo vim /etc/kubernetes/kube-proxy.kubeconfig
```

Añade el siguiente contenido:

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://10.17.4.23:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: system:kube-proxy
  name: default
current-context: default
users:
- name: system:kube-proxy
  user:
    client-certificate: /etc/kubernetes/pki/kube-proxy.crt
    client-key: /etc/kubernetes/pki/kube-proxy.key
```

## 3. Crear el Servicio kube-proxy

### 3.1 Crear el archivo del servicio en systemd

Crea el archivo de servicio para kube-proxy:

```bash
sudo vim /etc/systemd/system/kube-proxy.service
```

Agrega el siguiente contenido:

```bash
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://kubernetes.io/docs/
After=network.target

[Service]
ExecStart=/opt/bin/kube-proxy --config=/etc/kubernetes/kube-proxy-config.yaml
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

### 3.2 

## 4. Iniciar y Verificar el Servicio

## 4.1 Recargar systemd

Recarga systemd para aplicar los cambios:

```bash
sudo systemctl daemon-reload
```

### 4.2 Iniciar el servicio kube-proxy

Inicia y habilita el servicio kube-proxy para que se ejecute en cada inicio:

```bash
sudo systemctl start kube-proxy
sudo systemctl enable kube-proxy
```

### 4.3 Verificar el estado del servicio

Verifica que el servicio esté corriendo correctamente:

```bash
sudo systemctl status kube-proxy
```

## 5. Verificar los Logs y la Conectividad

### 5.1 Verificar los logs de kube-proxy

Monitorea los logs del servicio para identificar posibles errores o advertencias:

```bash
sudo journalctl -u kube-proxy -f
```


### 5.2 Verificar la conectividad con el servidor API de Kubernetes

Comprueba que kube-proxy esté conectado correctamente al servidor API de Kubernetes:

```bash
oc get nodes --kubeconfig /etc/kubernetes/admin.conf
```

## 6. Crear un ClusterRoleBinding para kube-proxy

Para asegurarse de que kube-proxy tiene los permisos adecuados para interactuar con el API de Kubernetes, debes crear un ClusterRoleBinding.

```bash
sudo oc --kubeconfig=/etc/kubernetes/admin.conf create clusterrolebinding kubelet-bootstrap --clusterrole=system:node --user=kubelet
```

# Conclusión

Este tutorial te permite configurar el servicio `kube-proxy` con los certificados necesarios para que se ejecute correctamente en tu clúster de Kubernetes. Siguiendo los pasos descritos, asegurarás que el servicio `kube-proxy` esté correctamente configurado y autenticado.




```bash
sudo vim /etc/systemd/system/kube-proxy.service
```

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

