Herramientas y Certificados por Nodo
Nodos Bootstrap
Herramientas:
OpenShift Installer: Utilizado para iniciar el proceso de instalación.
CRI-O: Container runtime para correr contenedores.
Kubelet: Agente que corre en cada nodo del clúster.
Ignition: Utilizado para inicializar y configurar el nodo.
Certificados:
Certificado individual para el kubelet.
Certificados compartidos para el servidor API y otros componentes comunes.
Nodos Master
Herramientas:
API Server: Provee las API para la administración del clúster.
Etcd: Almacén de datos distribuido para la configuración del clúster.
Controller Manager: Maneja los controladores que regulan el estado del clúster.
Scheduler: Asigna pods a los nodos basándose en restricciones y disponibilidad de recursos.
CRI-O: Container runtime.
Kubelet: Agente que corre en cada nodo del clúster.
Certificados:
Certificado individual para etcd.
Certificado individual para el kubelet.
Certificados compartidos para el servidor API y otros componentes comunes.
Nodos Worker
Herramientas:
CRI-O: Container runtime.
Kubelet: Agente que corre en cada nodo del clúster.
CNI Plugins: Manejan la red de contenedores.
Certificados:
Certificado individual para el kubelet.
Certificados compartidos para los componentes comunes que el nodo worker necesita acceder.
Certificados Detallados
Certificados Individuales:

Etcd: Cada nodo master que corre etcd debe tener su propio certificado y clave privada.
Ubicación: /etc/kubernetes/pki/etcd/
Kubelet: Cada nodo (bootstrap, master, worker) debe tener su propio certificado y clave privada para el kubelet.
Ubicación: /etc/kubernetes/pki/kubelet/
Certificados Compartidos:

API Server: El servidor API puede compartir el mismo certificado entre todos los nodos masters.
Ubicación: /etc/kubernetes/pki/apiserver/
CA Cert: La autoridad certificadora que firma todos los certificados del clúster.
Ubicación: /etc/kubernetes/pki/ca.crt
Service Account Key: Llave para firmar tokens de cuentas de servicio.
Ubicación: /etc/kubernetes/pki/sa.key
Referencias y Recomendaciones
OpenShift Documentation: Provee una guía completa sobre la instalación y configuración de OKD, incluyendo detalles sobre los certificados.
OpenShift Installation Guide
Kubernetes Documentation: Para detalles técnicos sobre cada componente y su configuración.
Kubernetes Components
Etcd Documentation: Para configuraciones específicas de etcd y cómo asegurar su comunicación.
Etcd Security Guide
Configurar correctamente los certificados y las herramientas en cada nodo es crucial para la seguridad y el funcionamiento adecuado de tu clúster OKD. Asegúrate de seguir las mejores prácticas y la documentación oficial para evitar problemas de configuración y seguridad.


Herramientas y Certificados por Nodo en OKD
Aquí se presenta una tabla ordenada en formato Markdown que detalla las herramientas y los certificados necesarios para cada tipo de nodo en un clúster OKD.

Nodos Bootstrap
Herramientas	Descripción
OpenShift Installer	Utilizado para iniciar el proceso de instalación.
CRI-O	Container runtime para correr contenedores.
Kubelet	Agente que corre en cada nodo del clúster.
Ignition	Utilizado para inicializar y configurar el nodo.
Certificados	Descripción
Certificado individual para kubelet	Certificado y clave privada para el kubelet del nodo bootstrap.
Certificados compartidos	Certificados del servidor API y otros componentes comunes (CA Cert, API Server, Service Account Key).
Nodos Master
Herramientas	Descripción
API Server	Provee las API para la administración del clúster.
Etcd	Almacén de datos distribuido para la configuración del clúster.
Controller Manager	Maneja los controladores que regulan el estado del clúster.
Scheduler	Asigna pods a los nodos basándose en restricciones y disponibilidad de recursos.
CRI-O	Container runtime.
Kubelet	Agente que corre en cada nodo del clúster.
Certificados	Descripción
Certificado individual para etcd	Certificado y clave privada para etcd en cada nodo master.
Certificado individual para kubelet	Certificado y clave privada para el kubelet del nodo master.
Certificados compartidos	Certificados del servidor API y otros componentes comunes (CA Cert, API Server, Service Account Key).
Nodos Worker
Herramientas	Descripción
CRI-O	Container runtime.
Kubelet	Agente que corre en cada nodo del clúster.
CNI Plugins	Manejan la red de contenedores.
Certificados	Descripción
Certificado individual para kubelet	Certificado y clave privada para el kubelet del nodo worker.
Certificados compartidos	Certificados de componentes comunes que el nodo worker necesita acceder (CA Cert).
Certificados Detallados
Certificados Individuales
Etcd: Cada nodo master que corre etcd debe tener su propio certificado y clave privada.

Ubicación: /etc/kubernetes/pki/etcd/
Kubelet: Cada nodo (bootstrap, master, worker) debe tener su propio certificado y clave privada para el kubelet.

Ubicación: /etc/kubernetes/pki/kubelet/
Certificados Compartidos
API Server: El servidor API puede compartir el mismo certificado entre todos los nodos masters.

Ubicación: /etc/kubernetes/pki/apiserver/
CA Cert: La autoridad certificadora que firma todos los certificados del clúster.

Ubicación: /etc/kubernetes/pki/ca.crt
Service Account Key: Llave para firmar tokens de cuentas de servicio.

Ubicación: /etc/kubernetes/pki/sa.key
Referencias y Recomendaciones
OpenShift Documentation: Provee una guía completa sobre la instalación y configuración de OKD, incluyendo detalles sobre los certificados.
Kubernetes Documentation: Para detalles técnicos sobre cada componente y su configuración.
Etcd Documentation: Para configuraciones específicas de etcd y cómo asegurar su comunicación.


Aquí se presenta una tabla ordenada en formato Markdown que detalla las herramientas y los certificados necesarios para cada tipo de nodo en un clúster OKD.

---

#### Nodos Bootstrap

| Herramientas                    | Descripción                                             |
|---------------------------------|---------------------------------------------------------|
| OpenShift Installer             | Utilizado para iniciar el proceso de instalación.       |
| CRI-O                           | Container runtime para correr contenedores.             |
| Kubelet                         | Agente que corre en cada nodo del clúster.              |
| Ignition                        | Utilizado para inicializar y configurar el nodo.        |

| Certificados                    | Descripción                                             |
|---------------------------------|---------------------------------------------------------|
| Certificado individual para kubelet | Certificado y clave privada para el kubelet del nodo bootstrap. |
| Certificados compartidos        | Certificados del servidor API y otros componentes comunes (CA Cert, API Server, Service Account Key). |

---

#### Nodos Master

| Herramientas                    | Descripción                                             |
|---------------------------------|---------------------------------------------------------|
| API Server                      | Provee las API para la administración del clúster.      |
| Etcd                            | Almacén de datos distribuido para la configuración del clúster. |
| Controller Manager              | Maneja los controladores que regulan el estado del clúster. |
| Scheduler                       | Asigna pods a los nodos basándose en restricciones y disponibilidad de recursos. |
| CRI-O                           | Container runtime.                                      |
| Kubelet                         | Agente que corre en cada nodo del clúster.              |

| Certificados                    | Descripción                                             |
|---------------------------------|---------------------------------------------------------|
| Certificado individual para etcd| Certificado y clave privada para etcd en cada nodo master. |
| Certificado individual para kubelet | Certificado y clave privada para el kubelet del nodo master. |
| Certificados compartidos        | Certificados del servidor API y otros componentes comunes (CA Cert, API Server, Service Account Key). |

---

#### Nodos Worker

| Herramientas                    | Descripción                                             |
|---------------------------------|---------------------------------------------------------|
| CRI-O                           | Container runtime.                                      |
| Kubelet                         | Agente que corre en cada nodo del clúster.              |
| CNI Plugins                     | Manejan la red de contenedores.                         |

| Certificados                    | Descripción                                             |
|---------------------------------|---------------------------------------------------------|
| Certificado individual para kubelet | Certificado y clave privada para el kubelet del nodo worker. |
| Certificados compartidos        | Certificados de componentes comunes que el nodo worker necesita acceder (CA Cert). |

---

### Certificados Detallados

#### Certificados Individuales

- **Etcd**: Cada nodo master que corre etcd debe tener su propio certificado y clave privada.
  - Ubicación: `/etc/kubernetes/pki/etcd/`

- **Kubelet**: Cada nodo (bootstrap, master, worker) debe tener su propio certificado y clave privada para el kubelet.
  - Ubicación: `/etc/kubernetes/pki/kubelet/`

#### Certificados Compartidos

- **API Server**: El servidor API puede compartir el mismo certificado entre todos los nodos masters.
  - Ubicación: `/etc/kubernetes/pki/apiserver/`

- **CA Cert**: La autoridad certificadora que firma todos los certificados del clúster.
  - Ubicación: `/etc/kubernetes/pki/ca.crt`

- **Service Account Key**: Llave para firmar tokens de cuentas de servicio.
  - Ubicación: `/etc/kubernetes/pki/sa.key`

### Referencias y Recomendaciones

- **[OpenShift Documentation](https://docs.openshift.com)**: Provee una guía completa sobre la instalación y configuración de OKD, incluyendo detalles sobre los certificados.
- **[Kubernetes Documentation](https://kubernetes.io/docs/home/)**: Para detalles técnicos sobre cada componente y su configuración.
- **[Etcd Documentation](https://etcd.io/docs/)**: Para configuraciones específicas de etcd y cómo asegurar su comunicación.

---


Configurar correctamente los certificados y las herramientas en cada nodo es crucial para la seguridad y el funcionamiento adecuado de tu clúster OKD. Asegúrate de seguir las mejores prácticas y la documentación oficial para evitar problemas de configuración y seguridad.

sudo systemctl daemon-reload
sudo systemctl restart crio
sudo systemctl status crio


sudo systemctl status crio kubelet kube-proxy etcd kube-controller-manager kube-apiserver kube-scheduler
sudo systemctl restart crio kubelet kube-proxy etcd kube-controller-manager kube-apiserver kube-scheduler




___

El problema que estás experimentando está relacionado con la validación del certificado del servidor etcd. Específicamente, el mensaje de error indica que el certificado del servidor etcd no contiene ninguna SAN (Subject Alternative Name) para la dirección IP 127.0.0.1.

Kubernetes y etcd utilizan certificados para autenticación mutua y encriptación de las comunicaciones. El kube-apiserver está intentando conectarse a etcd en 127.0.0.1:2379, pero el certificado presentado por etcd no tiene un SAN que coincida con la IP 127.0.0.1, lo que está causando el error de validación.

Solución
Para solucionar este problema, deberás regenerar el certificado etcd.crt y asegurarte de que incluya 127.0.0.1 en las SANs. Aquí están los pasos a seguir:

Crear un archivo de configuración de OpenSSL que especifique las SANs necesarias:

Crea un archivo de configuración, por ejemplo, etcd-openssl.cnf, con el siguiente contenido:

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
ST = State
L  = City
O  = Organization
OU = Organizational Unit
CN = etcd

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = etcd
IP.1 = 127.0.0.1
IP.2 = 10.17.4.21  # Reemplaza con la IP de tu servidor etcd si es diferente
Generar una nueva clave y certificado usando OpenSSL:

Utiliza el archivo de configuración para generar una nueva clave privada y un certificado:

bash
Copiar código
sudo openssl req -x509 -newkey rsa:2048 -keyout etcd.key -out etcd.crt -days 365 -nodes -config etcd-openssl.cnf
Esto generará un nuevo par de claves (etcd.key y etcd.crt) con el SAN apropiado.

Reemplaza los archivos de certificados en /etc/kubernetes/pki/etcd/:

bash
Copiar código
sudo mv etcd.crt /etc/kubernetes/pki/etcd/etcd.crt
sudo mv etcd.key /etc/kubernetes/pki/etcd/etcd.key
sudo chown etcd:etcd /etc/kubernetes/pki/etcd/etcd.key
sudo chmod 600 /etc/kubernetes/pki/etcd/etcd.key
Reinicia los servicios de etcd y kube-apiserver:

bash
Copiar código
sudo systemctl daemon-reload
sudo systemctl restart etcd
sudo systemctl restart kube-apiserver
Verifica el estado de los servicios:

bash
Copiar código
sudo systemctl status etcd
sudo systemctl status kube-apiserver