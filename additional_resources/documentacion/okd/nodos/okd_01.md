
### Guía de Instalación de OKD (OpenShift Kubernetes Distribution) Multinodo en KVM

#### Requisitos Previos

- **Servidor con KVM y libvirt instalados**
- **Sistema Operativo:** Rocky Linux 9.3 o similar
- **Terraform:** v0.13 o superior
- **Acceso a un servidor DNS (FreeIPA) configurado**
- **Llave SSH configurada**

#### Introducción

OKD (OpenShift Kubernetes Distribution) es una distribución de Kubernetes de código abierto que incluye herramientas y características adicionales para facilitar la administración y el despliegue de aplicaciones en contenedores. En esta guía, aprenderás a instalar un clúster OKD multinodo en KVM con los pasos necesarios y las configuraciones específicas para asegurar un despliegue exitoso.

---

#### Paso 1: Conectar y Preparar el Nodo Bootstrap

Accede al nodo bootstrap1:

```bash
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.14 -p 22
```

##### 1.1 Instalar paquetes necesarios

Instala `wget` y otros paquetes necesarios:

```bash
sudo dnf update -y && sudo dnf upgrade -y
sudo dnf install -y wget tar
```

##### 1.2 Descargar el Cliente de OpenShift (oc)

Descarga el cliente de OpenShift:

```bash
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
```

##### 1.3 Extraer y Mover el Cliente

```bash
tar -xzvf openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv openshift-install /opt/bin/
sudo chmod +x /opt/bin/openshift-install

```

##### 1.4 Verificar la Instalación

```bash
oc version
```

---

#### Paso 2: Descargar e Instalar el Instalador de OKD

##### 2.1 Descargar openshift-install

```bash
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
```

##### 2.2 Extraer y Mover el Instalador

```bash
tar -xzvf openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv openshift-install /usr/local/bin/
sudo chmod +x /usr/local/bin/openshift-install
```

---

#### Paso 3: Configurar el DNS en FreeIPA

##### 3.1 Añadir Entradas DNS

Conéctate a tu servidor FreeIPA y autentícate como administrador:

```bash
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.11 -p 22
```

Autentícate como administrador en FreeIPA:

```bash
kinit admin
Password for admin@CEFASLOCALSERVER.COM:
```

Configura las entradas DNS necesarias para tu clúster en FreeIPA:

```bash
ipa dnsrecord-add cefaslocalserver.com api.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com api-int.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master1.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master2.produccion --a-rec=10.17.4.22
ipa dnsrecord-add cefaslocalserver.com master3.produccion --a-rec=10.17.4.23
ipa dnsrecord-add cefaslocalserver.com worker1.produccion --a-rec=10.17.4.24
ipa dnsrecord-add cefaslocalserver.com worker2.produccion --a-rec=10.17.4.25
ipa dnsrecord-add cefaslocalserver.com worker3.produccion --a-rec=10.17.4.26
```

---

#### Paso 4: Transferir Archivos Ignition al Nodo Bootstrap

##### 4.1 Transferir los Archivos Ignition

Desde el servidor físico donde se generaron los archivos Ignition:

```bash
sudo scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift /home/victory/terraform-openshift-kvm-deployment_linux_Flatcar/nat_network_03/ignition-configs/*.ign core@10.17.3.14:/home/core/
```

---

#### Paso 5: Crear el Archivo de Configuración de Instalación

##### 5.1 Crear `install-config.yaml`


En el nodo bootstrap, crea el archivo `install-config.yaml`:

```bash
sudo mkdir okd-install
nano /home/core/okd-install/install-config.yaml
```

Y agrega el siguiente contenido:

```yaml
apiVersion: v1
baseDomain: cefaslocalserver.com
metadata:
  name: okd-cluster
compute:
- name: worker
  replicas: 3
controlPlane:
  name: master
  replicas: 3
networking:
  networkType: OpenShiftSDN
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: '{"auths": ...}'  # Reemplaza con tu pull secret
sshKey: | ssh-rsa AAAA... # Reemplaza con tu clave SSH pública 
```

Asegúrate de ajustar el bloque networking para definir la red de máquinas como 10.17.3.0/24, que incluye las IPs 10.17.3.10 y 10.17.3.11. De esta manera, las IPs apiVIP y ingressVIP estarán en el rango de la red de máquinas especificada y el instalador no generará un error.









---

#### Paso 6: Generar y Aplicar Manifiestos

##### 6.1 Generar Manifiestos

En el nodo bootstrap, genera los manifiestos:

permisos de escritura en el directorio de instalación:

```bash
sudo chown -R core:core /home/core/okd-install
sudo chmod -R 755 /home/core/okd-install
```

genera los manifiestos:

```bash
openshift-install create manifests --dir=/home/core/okd-install
```

##### 6.2 Generar Configuraciones Ignition

En el nodo bootstrap, genera las configuraciones Ignition:

```bash
openshift-install create ignition-configs --dir=/home/core/okd-install
```

---

#### Paso 7: Crear el Clúster

##### 7.1 Iniciar la Instalación

Inicia la instalación del clúster de OKD:

```bash
openshift-install create cluster --dir=/home/core/okd-install --log-level=debug
```

---

#### Paso 8: Verificar la Instalación

##### 8.1 Verificar Nodos

Verifica que todos los nodos del clúster estén funcionando correctamente:

```bash
oc get nodes
```

##### 8.2 Aprobar Solicitudes de Certificado (CSR)

Aprobar las solicitudes de certificados que sean necesarias para que los nodos se unan al clúster:

```bash
oc get csr
oc certificate approve <CSR_NAME>
```

---

#### Paso 9: Completar la Instalación

##### 9.1 Esperar la Finalización

Sigue los logs y espera a que la instalación se complete:

```bash
openshift-install --dir=/home/core/okd-install wait-for install-complete --log-level=debug
```

```bash
sudo dnf install bind-utils -y
```

```bash



---

#### Recursos Adicionales
Para más detalles y pasos avanzados, consulta los siguientes recursos:
- [Documentación de OKD](https://docs.okd.io/latest/welcome/index.html)
- [Guía de instalación de OKD](https://docs.okd.io/latest/installing/index.html)
- [Releases de OKD en GitHub](https://github.com/okd-project/okd/releases)

---

### Arquitectura del Clúster OKD

- **IP Pública:** 192.168.0.21
- **Load Balancer (Traefik, IP: 10.17.3.12):** Distribuye el tráfico entrante a los nodos maestros.
- **Nodos Maestros:** Master 1, Master 2, Master 3
- **Nodo Bootstrap (IP: 10.17.3.14):** Utilizado durante la configuración inicial del clúster.
- **Nodos Trabajadores:** Worker 1, Worker 2, Worker 3
- **FreeIPA (IP: 10.17.3.11):** Proporciona servicios de DNS y gestión de identidades.
- **Nodo Bastion (IP: 192.168.0.20):** Punto de acceso seguro para la gestión del clúster.

### Entradas DNS
Configura las siguientes entradas DNS en FreeIPA:

```bash
ipa dnsrecord-add cefaslocalserver.com api.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com api-int.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master1.produccion --a-rec=10.17.4.21
```

### Función del Balanceador de Carga (Traefik)
El Balanceador de Carga (Traefik) con la IP 10.17.3.12 se utiliza para distribuir el tráfico entrante entre los nodos maestros. Aunque `api.produccion` y `api-int.produccion` apuntan


### Guía de Instalación de OKD (OpenShift Kubernetes Distribution) Multinodo en KVM (Continuación)

---

### Entradas DNS (Continuación)

Configura las siguientes entradas DNS en FreeIPA:

```bash
ipa dnsrecord-add cefaslocalserver.com api.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com api-int.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master1.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master2.produccion --a-rec=10.17.4.22
ipa dnsrecord-add cefaslocalserver.com master3.produccion --a-rec=10.17.4.23
ipa dnsrecord-add cefaslocalserver.com worker1.produccion --a-rec=10.17.4.24
ipa dnsrecord-add cefaslocalserver.com worker2.produccion --a-rec=10.17.4.25
ipa dnsrecord-add cefaslocalserver.com worker3.produccion --a-rec=10.17.4.26
```

#### Propósitos de las Entradas DNS

- **api.produccion (10.17.4.21):**
  - **Propósito:** Facilita el acceso externo a la API del clúster OKD.
  - **Uso:** Utilizado por desarrolladores, herramientas CI/CD y otros usuarios externos para interactuar con el clúster.

- **api-int.produccion (10.17.4.21):**
  - **Propósito:** Facilita el acceso interno a la API del clúster OKD.
  - **Uso:** Utilizado por componentes internos del clúster para comunicarse entre sí, optimizando la eficiencia y reduciendo la carga en el balanceador de carga.

- **master1.produccion (10.17.4.21):**
  - **Propósito:** Permite el acceso directo al primer nodo maestro.
  - **Uso:** Utilizado para tareas de administración, mantenimiento y monitoreo específico del nodo maestro.

### Función del Balanceador de Carga (Traefik)

El Balanceador de Carga (Traefik) con la IP 10.17.3.12 se utiliza para distribuir el tráfico entrante entre los nodos maestros. Aunque `api.produccion` y `api-int.produccion` apuntan a la misma IP (10.17.4.21), en la práctica, el balanceador de carga dirige este tráfico a los nodos maestros disponibles.

---

### Diagramas de Flujo de Conexión

#### Conexiones Entrantes

- **Traefik:** Las conexiones HTTPS entrantes llegan a Traefik y se distribuyen entre los nodos maestros.
- **DNS:** Las entradas DNS (`api.produccion` y `api-int.produccion`) apuntan a la IP manejada por Traefik, facilitando el balanceo de carga.

#### Comunicación Interna y Externa

- **Interna:** `api-int.produccion` asegura la comunicación entre componentes internos del clúster.
- **Externa:** `api.produccion` gestiona el acceso externo al clúster.

---
### Plantilla Ignition Actualizada

Aquí está la plantilla Ignition actualizada que incluye la configuración para instalar `oc` y establecer `KUBECONFIG` automáticamente:

```yaml
variant: flatcar
version: 1.1.0

ignition:
  version: 3.4.0

passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ${ssh_keys}
    - name: root
      password_hash: $6$hNh1nwO5OWWct4aZ$OoeAkQ4gKNBnGYK0ECi8saBMbUNeQRMICcOPYEu1bFuj9Axt4Rh6EnGba07xtIsGNt2wP9SsPlz543gfJww11/

storage:
  files:
    - path: /etc/hostname
      filesystem: "root"
      mode: 0644
      contents:
        inline: ${host_name}
    - path: /home/core/install-oc.sh
      filesystem: "root"
      mode: 0755
      contents:
        inline: |
          #!/bin/bash
          set -euo pipefail
          exec > /home/core/install-oc.log 2>&1

          mkdir -p /opt/bin
          curl -L -o /tmp/oc.tar.gz https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz
          tar -xzf /tmp/oc.tar.gz -C /tmp
          sudo mv /tmp/oc /opt/bin/oc
          sudo chmod +x /opt/bin/oc
          sudo rm -rf /tmp/oc.tar.gz

          if [ -w /home/core/.bashrc ]; then
            echo 'export PATH=$PATH:/opt/bin' >> /home/core/.bashrc
            echo 'export KUBECONFIG=/home/core/okd-install/auth/kubeconfig' >> /home/core/.bashrc
          else
            echo "/home/core/.bashrc no es modificable"
          fi
    - path: /etc/systemd/network/10-eth0.network
      filesystem: "root"
      mode: 0644
      contents:
        inline: |
          [Match]
          Name=eth0

          [Network]
          Address=${ip}/24
          Gateway=${gateway}
          DNS=${dns1}
          DNS=${dns2}
    - path: /etc/tmpfiles.d/hosts.conf
      filesystem: "root"
      mode: 0644
      contents:
        inline: |
          f /etc/hosts 0644 - - - -
          127.0.0.1   localhost
          ::1         localhost
          ${ip}  ${host_name} ${name}
    - path: /run/systemd/resolve/resolv.conf
      filesystem: "root"
      mode: 0644
      contents:
        inline: |
          nameserver ${dns1}
          nameserver ${dns2}
    - path: /etc/tmpfiles.d/resolv.conf
      filesystem: "root"
      mode: 0644
      contents:
        inline: |
          L /etc/resolv.conf - - - - /run/systemd/resolve/resolv.conf

systemd:
  units:
    - name: apply-network-routes.service
      enabled: true
      contents: |
        [Unit]
        Description=Apply custom network routes
        After=network-online.target
        Wants=network-online.target

        [Service]
        Type=oneshot
        ExecStart=/usr/bin/systemctl restart systemd-networkd.service
        RemainAfterExit=true

        [Install]
        WantedBy=multi-user.target
    - name: set-hosts.service
      enabled: true
      contents: |
        [Unit]
        Description=Set /etc/hosts file
        After=network.target

        [Service]
        Type=oneshot
        ExecStart=/bin/bash -c 'echo "127.0.0.1   localhost" > /etc/hosts; echo "::1         localhost" >> /etc/hosts; echo "${ip}  ${host_name} ${name}" >> /etc/hosts'
        RemainAfterExit=true

        [Install]
        WantedBy=multi-user.target
    - name: install-oc.service
      enabled: true
      contents: |
        [Unit]
        Description=Install OpenShift CLI (oc)
        After=network-online.target
        Wants=network-online.target

        [Service]
        Type=oneshot
        ExecStart=/bin/bash /home/core/install-oc.sh
        RemainAfterExit=true

        [Install]
        WantedBy=multi-user.target
```

### Verificación

Después de aplicar la configuración, puedes verificar la configuración y la disponibilidad de los nodos:

1. **Accede a los nodos:**
   ```bash
   ssh -i /ruta/a/tu/clave/id_rsa core@10.17.4.21
   ```

2. **Verifica que `oc` esté disponible y configurado:**
   ```bash
   
   oc version

   source ~/.bashrc
   oc get nodes
   ```

3. **Verifica el estado de los servicios de systemd:**
 
```bash
sudo systemctl status install-oc.service
```


Siguiendo estos pasos y asegurando la correcta configuración de los nodos, deberías poder instalar y configurar OKD correctamente desde el nodo bootstrap.

---
Esta guía te ayudará a instalar un clúster OKD multinodo en KVM con los pasos necesarios y las configuraciones específicas para asegurar un despliegue exitoso.

### Contacto

Para cualquier duda o problema, por favor abre un issue en el repositorio o contacta al mantenedor del proyecto.



**Mantenedor del Proyecto:** Victor Galvez



# Transferir Archivos Ignition al Nodo Bootstrap




### nodo bootstrap1:


sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```




```bash
sudo scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift /home/victory/terraform-openshift-kvm-deployment_linux_Flatcar/nat_network_03/ignition-configs/*.ign core@10.17.4.27:~/okd-install/
```



