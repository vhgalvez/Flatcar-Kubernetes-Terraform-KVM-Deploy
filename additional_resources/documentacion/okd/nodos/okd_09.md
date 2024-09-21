Guía de Instalación de OKD (OpenShift Kubernetes Distribution) Multinodo en KVM
Requisitos Previos
Servidor con KVM y libvirt instalados
Sistema Operativo: Rocky Linux 9.3 o similar
Terraform: v0.13 o superior
Acceso a un servidor DNS (FreeIPA) configurado
Llave SSH configurada
Introducción
OKD (OpenShift Kubernetes Distribution) es una distribución de Kubernetes de código abierto que incluye herramientas y características adicionales para facilitar la administración y el despliegue de aplicaciones en contenedores. En esta guía, aprenderás a instalar un clúster OKD multinodo en KVM con los pasos necesarios y las configuraciones específicas para asegurar un despliegue exitoso.

Paso 1: Conectar y Preparar el Nodo Bootstrap
Accede al nodo bootstrap1:

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.14 -p 22
1.1 Instalar paquetes necesarios
Instala wget y otros paquetes necesarios:

bash
Copiar código
sudo dnf install -y wget tar
1.2 Descargar el Cliente de OpenShift (oc)
Descarga el cliente de OpenShift:

bash
Copiar código
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
1.3 Extraer y Mover el Cliente
bash
Copiar código
tar -xzvf openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv oc /usr/local/bin/
sudo chmod +x /usr/local/bin/oc
1.4 Verificar la Instalación
bash
Copiar código
oc version
Paso 2: Descargar e Instalar el Instalador de OKD
2.1 Descargar openshift-install
bash
Copiar código
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
2.2 Extraer y Mover el Instalador
bash
Copiar código
tar -xzvf openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv openshift-install /usr/local/bin/
sudo chmod +x /usr/local/bin/openshift-install
Paso 3: Configurar el DNS en FreeIPA
3.1 Añadir Entradas DNS
Conéctate a tu servidor FreeIPA y autentícate como administrador:

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.11 -p 22
Autentícate como administrador en FreeIPA:

bash
Copiar código
kinit admin
Password for admin@CEFASLOCALSERVER.COM:
Configura las entradas DNS necesarias para tu clúster en FreeIPA:

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
Paso 4: Configurar el archivo install-config.yaml
Crea y edita el archivo install-config.yaml:

bash
Copiar código
nano install-config.yaml
Agrega la siguiente configuración:

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
platform:
  none: {}
pullSecret: '{"auths":{"cloud.openshift.com":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfZTk5M2RiN2MwMDNkNDFlNDhmZGU4ZGE0OWIxZmZkYTI6WkhMMk5HUVFTS01aS1JJSTBPME5aUzFaSVgwQU9GTTNDQjdQUkxMMjc5UzRZN1BKSTBURVdSQUhLSFFZVVJOUw==","email":"vhgalvez@gmail.com"}}}'
sshKey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDC9XqGWEd2de3Ud8TgvzFchK2/SYh+WHohA1KEuveXjCbse9aXKmNAZ369vaGFFGrxbSptMeEt41ytEFpU09gAXM6KSsQWGZxfkCJQSWIaIEAdft7QHnTpMeronSgYZIU+5P7/RJcVhHBXfjLHV6giHxFRJ9MF7n6sms38VsuF2s4smI03DWGWP6Ro7siXvd+LBu2gDqosQaZQiz5/FX5YWxvuhq0E/ACas/JE8fjIL9DQPcFrgQkNAv1kHpIWRqSLPwyTMMxGgFxGI8aCTH/Uaxbqa7Qm/aBfdG2lZBE1XU6HRjAToFmqsPJv4LkBxaC1Ag62QPXONNxAA97arICr vhgalvez@gmail.com'
Paso 5: Crear las Configuraciones de Manifiestos e Ignición
5.1 Crear Manifiestos
bash
Copiar código
openshift-install create manifests --dir=/home/core/okd-install
5.2 Crear Configuraciones de Ignición
bash
Copiar código
openshift-install create ignition-configs --dir=/home/core/okd-install
Paso 6: Desplegar el Clúster OKD
6.1 Iniciar la Instalación del Clúster
bash
Copiar código
openshift-install wait-for install-complete --dir=/home/core/okd-install --log-level=info
Paso 7: Verificación y Solución de Problemas
Si encuentras problemas de conectividad, verifica los estados de los nodos y asegúrate de que todos los servicios necesarios estén funcionando correctamente. Puedes usar herramientas como ping, ssh, y revisar los logs en /var/log para diagnosticar cualquier problema.

Para comprobar la conectividad de la red, puedes usar el script check_connectivity.sh desde cualquier ubicación utilizando el comando completo:

bash
Copiar código
sudo /home/core/check_connectivity_network/check_connectivity.sh
Con esta guía, deberías tener un clúster OKD funcionando en un entorno KVM con múltiples nodos. Asegúrate de seguir cada paso cuidadosamente y verificar que todas las configuraciones sean correctas.



The issue seems to be that the DNS zone for okd-cluster.cefaslocalserver.com has not been created in FreeIPA, hence the errors.

Follow these steps to resolve the issue:

Create the DNS Zone in FreeIPA:

bash
Copiar código
ipa dnszone-add okd-cluster.cefaslocalserver.com
Add the A Records:

bash
Copiar código
ipa dnsrecord-add okd-cluster.cefaslocalserver.com api --a-rec 10.17.4.21
ipa dnsrecord-add okd-cluster.cefaslocalserver.com api-int --a-rec 10.17.4.21
Verify the DNS Records:

bash
Copiar código
dig api.okd-cluster.cefaslocalserver.com
dig api-int.okd-cluster.cefaslocalserver.com
If systemd-networkd and systemd-resolved services are not available, ensure network services are properly configured and DNS settings are correctly pointing to your FreeIPA server. You can check and restart the relevant network services specific to Rocky Linux if necessary.



____

Reprovisionar los nodos
Aplica los archivos corregidos en tu configuración de Terraform e Ignition.
Destruye y vuelve a crear los nodos maestros y trabajadores con las nuevas configuraciones.
Validar la instalación
Verifica el estado de los servicios:


sudo systemctl status kubelet
sudo systemctl status crio
sudo systemctl status etcd
sudo systemctl status kube-apiserver
sudo systemctl status kube-controller-manager
sudo systemctl status kube-scheduler


/opt/bin/kubelet --version
/opt/bin/crio --version
/opt/bin/oc version
/opt/bin/etcd --version
/opt/bin/kube-apiserver --version
/opt/bin/kube-controller-manager --version
/opt/bin/kube-scheduler --version


cat /var/log/install-master-components.log
cat /var/log/install-worker-components.log

sudo journalctl -u install-master-components.service

sudo journalctl -u install-worker-components.service

_______________





# paquetes necesarios

# Create directory for binaries

sudo mkdir -p /opt/bin


# Install kubelet
curl -L -o /opt/bin/kubelet https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubelet
sudo chmod +x /opt/bin/kubelet

# Install oc
curl -L -o /tmp/oc.tar.gz https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz
tar -xzf /tmp/oc.tar.gz -C /tmp
sudo mv /tmp/oc /opt/bin/oc
sudo chmod +x /opt/bin/oc
sudo rm -rf /tmp/oc.tar.gz

# Install etcd
curl -L -o /tmp/etcd.tar.gz https://github.com/etcd-io/etcd/releases/download/v3.4.13/etcd-v3.4.13-linux-amd64.tar.gz
tar -xzf /tmp/etcd.tar.gz -C /tmp
sudo mv /tmp/etcd-v3.4.13-linux-amd64/etcd /opt/bin/etcd
sudo chmod +x /opt/bin/etcd
sudo rm -rf /tmp/etcd.tar.gz /tmp/etcd-v3.4.13-linux-amd64

# Install kube-apiserver
curl -L -o /tmp/kube-apiserver https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kube-apiserver
sudo mv /tmp/kube-apiserver /opt/bin/kube-apiserver
sudo chmod +x /opt/bin/kube-apiserver

# Install kube-controller-manager
curl -L -o /tmp/kube-controller-manager https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kube-controller-manager
sudo mv /tmp/kube-controller-manager /opt/bin/kube-controller-manager
sudo chmod +x /opt/bin/kube-controller-manager

# Install kube-scheduler
curl -L -o /tmp/kube-scheduler https://dl.k8s.io/release/v1.21.0/bin/linux/amd64/kube-scheduler
sudo mv /tmp/kube-scheduler /opt/bin/kube-scheduler
sudo chmod +x /opt/bin/kube-scheduler

__


# Install cri-o
curl -L -o /tmp/crio.tar.gz https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/1.21:/1.21.0/x86_64/cri-o-1.21.0.x86_64.rpm
sudo rpm2cpio /tmp/crio.tar.gz | sudo cpio -idmv
sudo mv ./usr/bin/crio /opt/bin/
sudo chmod +x /opt/bin/crio
rm -rf /tmp/crio.tar.gz ./usr


# Enable and start services
sudo systemctl daemon-reload
sudo systemctl enable kubelet
sudo systemctl start kubelet
sudo systemctl enable crio
sudo systemctl start crio


# Nodos Maestros (Flatcar Container Linux)

- kubelet
- cri-o
- oc (OpenShift Client)
- etcd
- kube-apiserver
- kube-controller-manager
- kube-scheduler

# Nodos Trabajadores (Flatcar Container Linux)

- kubelet
- cri-o
- oc (OpenShift Client)

# Nodo de Arranque (Bootstrap) (Rocky Linux)

- kubelet
- cri-o
- oc (OpenShift Client)
- openshift-install