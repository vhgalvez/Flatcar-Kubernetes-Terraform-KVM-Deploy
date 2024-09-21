# Clasificación y Distribución de Certificados en un Clúster Kubernetes

## 1. Certificados que Deben Estar en Todos los Nodos (Masters, Workers, Bootstrap)

Estos certificados son esenciales para la autenticidad y comunicación segura entre los componentes del clúster en todos los nodos.

- **CA Certificate (ca.crt)**
  - **Ubicación:** `/etc/kubernetes/pki/ca.crt`
  - **Propósito:** Es la autoridad de certificación para todo el clúster, utilizado por todos los nodos para verificar otros certificados.

- **Kubelet Certificate (kubelet.crt)**
  - **Ubicación:** `/etc/kubernetes/pki/kubelet.crt`
  - **Propósito:** Autentica el kubelet en cada nodo con el API server, asegurando que el nodo es legítimo y está autorizado a unirse al clúster.

- **Kubelet Key (kubelet.key)**
  - **Ubicación:** `/etc/kubernetes/pki/kubelet.key`
  - **Propósito:** Es la clave privada correspondiente al certificado del kubelet, permitiendo que el nodo firme su identidad al comunicarse con el API server.

## 2. Certificados Específicos para los Nodos Master

Los nodos Master administran componentes críticos como el API server, el controlador, y etcd, y por lo tanto requieren certificados adicionales.

- **API Server Certificate (apiserver.crt)**
  - **Ubicación:** `/etc/kubernetes/pki/apiserver.crt`
  - **Propósito:** Autentica el API server en los nodos Master con los clientes (como kubectl), asegurando la comunicación segura.

- **API Server Key (apiserver.key)**
  - **Ubicación:** `/etc/kubernetes/pki/apiserver.key`
  - **Propósito:** Es la clave privada correspondiente al certificado del API server, utilizada para firmar y asegurar las comunicaciones.

- **Service Account Key Pair (sa.key, sa.pub)**
  - **Ubicación:** `/etc/kubernetes/pki/sa.key`, `/etc/kubernetes/pki/sa.pub`
  - **Propósito:** Utilizados por el API server y el controlador de cuentas de servicio para firmar y verificar los tokens de servicio dentro del clúster.

- **Etcd Certificates**

  - **Etcd Server Certificate (etcd.crt)**
    - **Ubicación:** `/etc/kubernetes/pki/etcd/etcd.crt`
    - **Propósito:** Asegura la comunicación dentro del clúster de etcd en los nodos Master.

  - **Etcd Server Key (etcd.key)**
    - **Ubicación:** `/etc/kubernetes/pki/etcd/etcd.key`
    - **Propósito:** Es la clave privada utilizada por el servidor etcd para firmar y asegurar las comunicaciones.

  - **Etcd CA Certificate (etcd/ca.crt)**
    - **Ubicación:** `/etc/kubernetes/pki/etcd/ca.crt`
    - **Propósito:** Autoridad de certificación para los certificados utilizados en el clúster de etcd.

- **API Server Etcd Client Certificates**

  - **API Server Etcd Client Certificate (apiserver-etcd-client.crt)**
    - **Ubicación:** `/etc/kubernetes/pki/apiserver-etcd-client.crt`
    - **Propósito:** Utilizado por el API server para autenticarse con el clúster de etcd.

  - **API Server Etcd Client Key (apiserver-etcd-client.key)**
    - **Ubicación:** `/etc/kubernetes/pki/apiserver-etcd-client.key`
    - **Propósito:** Clave privada utilizada por el API server para autenticarse con el clúster de etcd.

## 3. Certificados Específicos para el Nodo Bootstrap

El nodo Bootstrap es utilizado para iniciar el clúster y luego su rol puede ser reemplazado por los nodos Master. Inicialmente, requiere los mismos certificados que los Masters, pero una vez que el clúster está configurado, su rol puede cambiar.

- **Inicialmente:**
  - Los mismos certificados que los nodos Master:
    - API Server Certificate (apiserver.crt)
    - API Server Key (apiserver.key)
    - Etcd Certificates (etcd.crt, etcd.key, etcd/ca.crt)
    - API Server Etcd Client Certificates (apiserver-etcd-client.crt, apiserver-etcd-client.key)

- **Posteriormente:**
  - Puede ser eliminado o reintegrado con los certificados básicos si sigue en operación con un rol diferente:
    - CA Certificate (ca.crt)
    - Kubelet Certificate (kubelet.crt)
    - Kubelet Key (kubelet.key)

## Resumen de Certificados por Nodo

- **Todos los Nodos (Masters, Workers, Bootstrap):**
  - ca.crt
  - kubelet.crt, kubelet.key

- **Nodos Master (además de los anteriores):**
  - apiserver.crt, apiserver.key
  - sa.key, sa.pub
  - etcd.crt, etcd.key, etcd/ca.crt
  - apiserver-etcd-client.crt, apiserver-etcd-client.key

- **Nodo Bootstrap (al iniciar el clúster):**
  - Inicialmente: Los mismos certificados que los nodos Master.
  - Posteriormente: Puede ser eliminado o reintegrado con los certificados básicos (ca.crt, kubelet.crt, kubelet.key) si sigue en operación con un rol diferente.

Este esquema de clasificación y distribución asegura que cada nodo tenga los certificados necesarios para su función en el clúster Kubernetes.