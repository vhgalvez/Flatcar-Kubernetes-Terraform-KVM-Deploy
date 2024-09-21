# Tutorial: Creación y Regeneración de Certificados para el kube-controller-manager

En este tutorial, vamos a crear y configurar el servicio de `kube-controller-manager`, generar los certificados necesarios y asegurarnos de que el servicio esté corriendo correctamente en tu clúster de Kubernetes.

## 1. Creación del Servicio kube-controller-manager

Primero, definiremos el archivo del servicio systemd para el `kube-controller-manager` para que Kubernetes pueda iniciar y administrar el controlador.

### 1.1 Crear el archivo del servicio

Crea un archivo de servicio para el `kube-controller-manager` en `/etc/systemd/system/kube-controller-manager.service`:

```bash
sudo vim /etc/systemd/system/kube-controller-manager.service
```

### 1.2 Contenido del archivo de servicio

Agrega lo siguiente al archivo de servicio:

```bash
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://kubernetes.io/docs/concepts/overview/components/
After=network.target

[Service]
ExecStart=/opt/bin/kube-controller-manager \
  --kubeconfig=/etc/kubernetes/controller-manager.conf \
  --bind-address=0.0.0.0 \
  --leader-elect=true \
  --use-service-account-credentials=true \
  --controllers=*,bootstrapsigner,tokencleaner \
  --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt \
  --cluster-signing-key-file=/etc/kubernetes/pki/ca.key \
  --root-ca-file=/etc/kubernetes/pki/ca.crt \
  --service-account-private-key-file=/etc/kubernetes/pki/sa.key \
  --cluster-name=kubernetes \
  --cluster-cidr=10.244.0.0/16 \
  --allocate-node-cidrs=true \
  --node-cidr-mask-size=24 \
  --service-cluster-ip-range=10.96.0.0/12 \
  --v=2
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Este archivo configura el `kube-controller-manager` para que use el archivo de configuración `controller-manager.conf` y los certificados necesarios para funcionar correctamente en el clúster.

## 2. Crear el Archivo de Configuración de kube-controller-manager

El archivo de configuración de `kube-controller-manager` le indica cómo conectarse a la API de Kubernetes y qué certificados usar para la autenticación.

## 2.1 Crear el archivo de configuración

Crea el archivo `/etc/kubernetes/controller-manager.conf` con el siguiente contenido:

```bash
sudo vim /etc/kubernetes/controller-manager.conf
```

### 2.2 Contenido del archivo de configuración

Agrega el siguiente contenido al archivo:

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
      user: system:kube-controller-manager
    name: system:kube-controller-manager@kubernetes
current-context: system:kube-controller-manager@kubernetes
users:
  - name: system:kube-controller-manager
    user:
      client-certificate: /etc/kubernetes/pki/kube-controller-manager.crt
      client-key: /etc/kubernetes/pki/kube-controller-manager.key
```

Este archivo configura al `kube-controller-manager` para conectarse al `kube-apiserver` en la dirección `10.17.4.22:6443`, usando los certificados adecuados para la autenticación.

## 3. Generar los Certificados del kube-controller-manager

En este paso, regeneraremos los certificados necesarios para el `kube-controller-manager`.

### 3.1 Eliminar los certificados antiguos

Si ya tienes certificados antiguos, elimínalos antes de continuar:


```bash
sudo rm /etc/kubernetes/pki/kube-controller-manager.crt /etc/kubernetes/pki/kube-controller-manager.key /etc/kubernetes/pki/kube-controller-manager.csr
````

### 3.2 Generar una nueva clave privada

Genera una nueva clave privada para el `kube-controller-manager`:

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/kube-controller-manager.key -pkeyopt rsa_keygen_bits:2048
```

### 3.3 Crear una nueva CSR (Solicitud de Firma de Certificado)

Crea una nueva CSR para el `kube-controller-manager`:

```bash
sudo openssl req -new -key /etc/kubernetes/pki/kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out /etc/kubernetes/pki/kube-controller-manager.csr
```

### 3.4 Firmar el CSR con la CA

Firma la solicitud CSR con la Autoridad Certificadora (CA) de Kubernetes para generar un nuevo certificado:

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/kube-controller-manager.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kube-controller-manager.crt -days 365
```

### 3.5 Verificar los permisos de los certificados

Asegúrate de que los permisos de los archivos de certificados sean correctos:

```bash
sudo chmod 600 /etc/kubernetes/pki/kube-controller-manager.key
sudo chmod 644 /etc/kubernetes/pki/kube-controller-manager.crt
```

## 4. Reiniciar y Verificar el Servicio

4.1 Recargar el demonio de systemd

Recarga el daemon de systemd para asegurarte de que el nuevo servicio esté registrado:

```bash
sudo systemctl daemon-reload
```

### 4.2 Iniciar el servicio kube-controller-manager

Inicia el servicio y habilítalo para que se ejecute automáticamente al inicio:

```bash
sudo systemctl start kube-controller-manager
sudo systemctl enable kube-controller-manager
sudo systemctl status kube-controller-manager

```

### 4.3 Verificar el estado del servicio

Verifica que el servicio `kube-controller-manager` esté funcionando correctamente:

```bash
sudo systemctl status kube-controller-manager
```

## 5. Verificar los Logs y la Conectividad

### 5.1 Verificar los logs del servicio

Para monitorear los logs en tiempo real y asegurarte de que no haya errores, ejecuta:

```bash
sudo journalctl -u kube-controller-manager -f
```


Generar cerficados para el kube-controller-manager

```bash
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/kube-controller-manager.key -pkeyopt rsa_keygen_bits:2048
sudo openssl req -new -key /etc/kubernetes/pki/kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out /etc/kubernetes/pki/kube-controller-manager.csr
```

```bash
sudo openssl x509 -req -in /etc/kubernetes/pki/kube-controller-manager.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kube-controller-manager.crt -days 365
```