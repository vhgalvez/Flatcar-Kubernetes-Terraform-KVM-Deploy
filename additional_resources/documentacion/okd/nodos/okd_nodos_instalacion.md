Para proceder con la instalación de OKD (OpenShift Kubernetes Distribution) en un entorno KVM con los nodos Bootstrap, Master y Worker, siguiendo las configuraciones y pasos detallados anteriormente, se puede seguir esta guía estructurada:

Guía de Instalación de OKD Multinodo en KVM
Requisitos Previos
Servidor con KVM y libvirt instalados.
Sistema Operativo: Rocky Linux 9.3 o similar.
Terraform: v0.13 o superior.
Acceso a un servidor DNS (FreeIPA) configurado.
Llave SSH configurada.
Paso 1: Preparar el Nodo Bootstrap
Conectar al Nodo Bootstrap

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.14 -p 22
Instalar Paquetes Necesarios

bash
Copiar código
sudo dnf install -y wget tar
Descargar e Instalar el Cliente de OpenShift (oc)

bash
Copiar código
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
tar -xzvf openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv oc /usr/local/bin/
sudo chmod +x /usr/local/bin/oc
Verificar la Instalación

bash
Copiar código
oc version
Paso 2: Descargar e Instalar el Instalador de OKD
Descargar el Instalador de OKD

bash
Copiar código
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
Extraer y Mover el Instalador

bash
Copiar código
tar -xzvf openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv openshift-install /usr/local/bin/
sudo chmod +x /usr/local/bin/openshift-install
Paso 3: Configurar DNS en FreeIPA
Conectar y Autenticar en el Servidor FreeIPA

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.11 -p 22
kinit admin
Añadir Entradas DNS

bash
Copiar código
ipa dnsrecord-add cefaslocalserver.com api.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com api-int.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master1.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master2.produccion --a-rec=10.17.4.22
ipa dnsrecord-add cefaslocalserver.com master3.produccion --a-rec=10.17.4.23
ipa dnsrecord-add cefaslocalserver.com worker1.produccion --a-rec=10.17.4.24
ipa dnsrecord-add cefaslocalserver.com worker2.produccion --a-rec=10.17.4.25
ipa dnsrecord-add cefaslocalserver.com worker3.produccion --a-rec=10.17.4.26
Paso 4: Transferir Archivos Ignition al Nodo Bootstrap
Transferir Archivos Ignition
bash
Copiar código
sudo scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift /home/victory/terraform-openshift-kvm-deployment_linux_Flatcar/nat_network_03/ignition-configs/*.ign core@10.17.3.14:/home/core/
Paso 5: Crear el Archivo de Configuración de Instalación
Crear y Editar install-config.yaml

bash
Copiar código
sudo mkdir okd-install
nano /home/core/okd-install/install-config.yaml
Contenido de install-config.yaml

yaml
Copiar código
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
Paso 6: Generar y Aplicar Manifiestos
Ajustar Permisos del Directorio de Instalación

bash
Copiar código
sudo chown -R core:core /home/core/okd-install
sudo chmod -R 755 /home/core/okd-install
Generar Manifiestos

bash
Copiar código
openshift-install create manifests --dir=/home/core/okd-install
Generar Configuraciones Ignition

bash
Copiar código
openshift-install create ignition-configs --dir=/home/core/okd-install
Paso 7: Crear el Clúster
Iniciar la Instalación
bash
Copiar código
openshift-install create cluster --dir=/home/core/okd-install --log-level=debug
Paso 8: Verificar la Instalación
Verificar Nodos

bash
Copiar código
oc get nodes
Aprobar Solicitudes de Certificado (CSR)

bash
Copiar código
oc get csr
oc certificate approve <CSR_NAME>
Paso 9: Completar la Instalación
Esperar la Finalización

bash
Copiar código
openshift-install --dir=/home/core/okd-install wait-for install-complete --log-level=debug
Instalar bind-utils

bash
Copiar código
sudo dnf install bind-utils -y
Recursos Adicionales
Documentación de OKD
Guía de instalación de OKD
Releases de OKD en GitHub
Arquitectura del Clúster OKD
IP Pública: 192.168.0.21
Load Balancer (Traefik, IP: 10.17.3.12): Distribuye el tráfico entrante a los nodos maestros.
Nodos Maestros: Master 1, Master 2, Master 3
Nodo Bootstrap (IP: 10.17.3.14): Utilizado durante la configuración inicial del clúster.
Nodos Trabajadores: Worker 1, Worker 2, Worker 3
FreeIPA (IP: 10.17.3.11): Proporciona servicios de DNS y gestión de identidades.
Nodo Bastion (IP: 192.168.0.20): Punto de acceso seguro para la gestión del clúster.
Entradas DNS
bash
Copiar código
ipa dnsrecord-add cefaslocalserver.com api.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com api-int.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master1.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master2.produccion --a-rec=10.17.4.22
ipa dnsrecord-add cefaslocalserver.com master3.produccion --a-rec=10.17.4.23
ipa dnsrecord-add cefaslocalserver.com worker1.produccion --a-rec=10.17.4.24
ipa dnsrecord-add cefaslocalserver.com worker2.produccion --a-rec=10.17.4.25
ipa dnsrecord-add cefaslocalserver.com worker3.produccion --a-rec=10.17.4.26
Función del Balanceador de Carga (Traefik)
El Balanceador de Carga (Traefik) con la IP 10.17.3.12 se utiliza para distribuir el tráfico entrante entre los nodos maestros. Aunque api.produccion y api-int.produccion apuntan a la misma IP (10.17.4.21), en la práctica, el balanceador de carga dirige este tráfico a los nodos maestros disponibles.

Flujo de Conexión
Conexiones Entrantes: Traefik distribuye las conexiones HTTPS entrantes a los nodos maestros.
Comunicación Interna y Externa: api-int.produccion asegura la comunicación entre componentes internos del clúster y api.produccion gestiona el acceso externo al clúster.
Hardware del Servidor
Modelo: ProLiant DL380 G7
CPU: Intel Xeon X5650 (24 cores) @ 2.666GHz
GPU: AMD ATI 01:03.0 ES1000
Memoria: 1093MiB / 35904MiB
Almacenamiento: Disco Duro Principal: 1.5TB, Disco Duro Secundario: 3.0TB
Configuración de Red
VPN con WireGuard
Firewall, DHCP en KVM, NAT y Bridge
Switch y Router
Máquinas Virtuales y Sistemas Operativos
Bastion Node: Rocky Linux Minimal
Bootstrap Node: Rocky Linux Minimal
Master Nodes: Flatcar Container Linux
Worker Nodes: Flatcar Container Linux
FreeIPA Node: Rocky Linux Minimal
Load Balancer Node: Rocky Linux Minimal
Especificaciones de Almacenamiento y Memoria
Configuración de Disco y Particiones:

/dev/sda: 3.27 TiB
/dev/sdb: 465.71 GiB
Particiones:

/dev/sda1: Sistema
/dev/sda2: 2 GB Linux Filesystem
/dev/sda3: ~2.89 TiB Linux Filesystem
Uso de Memoria:

Total Memory: 35GiB
Free Memory: 33GiB
Swap: 17GiB
Uso del Filesystem:

/dev/mapper/rl-root: 100G (7.5G usado)
/dev/sda2: 1014M (718M usado)
/dev/mapper/rl-home: 3.0T (25G usado)
Máquinas Virtuales y Roles
Bootstrap Node: 2 CPUs, 2048 MB, inicializa el clúster
Master Nodes: 3 x (2 CPUs, 4096 MB), gestionan el clúster
Worker Nodes: 3 x (2 CPUs, 3584 MB), ejecutan aplicaciones
Bastion Node: 2 CPUs, 2048 MB, seguridad y acceso
Load Balancer: 2 CPUs, 2048 MB, con Traefik
FreeIPA Node: 2 CPUs, 2048 MB, servidor de DNS y gestión de identidades
PostgreSQL Node: 2 CPUs, 2048 MB, gestión de bases de datos
Resumen de los Hostnames e IPs
10.17.3.11: freeipa1.cefaslocalserver.com
10.17.3.12: load_balancer1.cefaslocalserver.com
10.17.3.13: postgresql1.cefaslocalserver.com
10.17.3.14: bootstrap1.cefaslocalserver.com
10.17.4.21: master1.cefaslocalserver.com
10.17.4.22: master2.cefaslocalserver.com
10.17.4.23: master3.cefaslocalserver.com
10.17.4.24: worker1.cefaslocalserver.com
10.17.4.25: worker2.cefaslocalserver.com
10.17.4.26: worker3.cefaslocalserver.com