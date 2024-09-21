Para asegurar que todo el proceso de instalación de OKD 4.12 en un entorno KVM con red NAT se realice correctamente, aquí tienes la guía completa con todos los pasos detallados:

Paso 1: Conectar y Preparar el Nodo Bootstrap
Accede al nodo bootstrap1:

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.14 -p 22
Instala los paquetes necesarios:

bash
Copiar código
sudo dnf install -y wget tar
Descarga el Cliente de OpenShift (oc):

bash
Copiar código
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
Extrae y mueve el Cliente:

bash
Copiar código
tar -xzvf openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv oc /usr/local/bin/
sudo chmod +x /usr/local/bin/oc
Verifica la Instalación:

bash
Copiar código
oc version
Paso 2: Descargar e Instalar el Instalador de OKD
Descarga openshift-install:

bash
Copiar código
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
Extrae y mueve el Instalador:

bash
Copiar código
tar -xzvf openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv openshift-install /usr/local/bin/
sudo chmod +x /usr/local/bin/openshift-install
Paso 3: Configurar el DNS en FreeIPA
Conéctate a tu servidor FreeIPA y autentícate como administrador:

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.11 -p 22
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
Paso 4: Descargar las Imágenes Necesarias
Para descargar las imágenes necesarias para la instalación de OKD 4.12, puedes utilizar las siguientes URLs correctas:

Imagen de Bootstrap (bootstrap-image.img):

bash
Copiar código
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.12/latest/rhcos-live.x86_64.iso -O /var/www/html/okd/bootstrap-image.img
Imagen del Cluster (cluster-image.iso):

bash
Copiar código
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.12/latest/rhcos-live.x86_64.iso -O /var/www/html/okd/cluster-image.iso
Paso 5: Instalar y Configurar el Servidor HTTP en el Nodo Bootstrap
Configurar el servidor HTTP para servir las imágenes:

bash
Copiar código
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo mkdir -p /var/www/html/okd
sudo chown -R core:core /var/www/html/okd
sudo chmod -R 755 /var/www/html/okd
Verificar la accesibilidad de las imágenes:

bash
Copiar código
curl http://10.17.3.14/okd/bootstrap-image.img
curl http://10.17.3.14/okd/cluster-image.iso
Paso 6: Configurar install-config.yaml
Crea install-config.yaml en el nodo bootstrap:

bash
Copiar código
sudo mkdir -p /home/core/okd-install
nano /home/core/okd-install/install-config.yaml
Agrega el siguiente contenido:

yaml
Copiar código
apiVersion: v1
baseDomain: cefaslocalserver.com
metadata:
  name: cluster-cefas
compute:
- name: worker
  replicas: 3
controlPlane:
  name: master
  replicas: 3
networking:
  networkType: OpenShiftSDN
  machineNetwork:
  - cidr: 10.17.3.0/24
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
bootstrapOSImage: "http://10.17.3.14/okd/bootstrap-image.img"
clusterOSImage: "http://10.17.3.14/okd/cluster-image.iso"
pullSecret: 'YOUR_PULL_SECRET'
sshKey: |
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDC9XqGWEd2de3Ud8TgvzFchK2/SYh+WHohA1KEuveXjCbse9aXKmNAZ369vaGFFGrxbSptMeEt41ytEFpU09gAXM6KSsQWGZxfkCJQSWIaIEAdft7QHnTpMeronSgYZIU+5P7/RJcVhHBXfjLHV6giHxFRJ9MF7n6sms38VsuF2s4smI03DWGWP6Ro7siXvd+LBu2gDqosQaZQiz5/FX5YWxvuhq0E/ACas/JE8fjIL9DQPcFrgQkNAv1kHpIWRqSLPwyTMMxGgFxGI8aCTH/Uaxbqa7Qm/aBfdG2lZBE1XU6HRjAToFmqsPJv4LkBxaC1Ag62QPXONNxAA97arICr vhgalvez@gmail.com
Paso 7: Crear Manifiestos e Ignition Configs
Generar Manifiestos:

bash
Copiar código
openshift-install create manifests --dir=/home/core/okd-install
Generar Configuraciones Ignition:

bash
Copiar código
openshift-install create ignition-configs --dir=/home/core/okd-install
Paso 8: Iniciar la Instalación del Clúster
Iniciar la Instalación:

bash
Copiar código
openshift-install create cluster --dir=/home/core/okd-install --log-level=debug
Paso 9: Verificar el Estado de los Nodos
En el servidor físico, puedes verificar el estado de los nodos ejecutando:

bash
Copiar código
sudo virsh list
Detalles Adicionales
Asegúrate de que los archivos de imágenes se hayan descargado correctamente y sean accesibles desde el servidor HTTP:

bash
Copiar código
ls -l /var/www/html/okd/
Verifica la conectividad de red y asegúrate de que todas las direcciones IP y nombres de host sean correctos en tu configuración.

Con estos pasos, deberías poder instalar un clúster OKD multinodo en KVM. Asegúrate de seguir cada paso cuidadosamente y verificar las configuraciones para asegurar un despliegue exitoso.

___


arece que hay un problema con la resolución de nombres DNS, específicamente con api.cluster-cefas.cefaslocalserver.com. Según el error de openshift-install, no puede resolver la dirección api.cluster-cefas.cefaslocalserver.com.

Paso para Solucionar el Problema
Verificar el registro DNS:

El registro DNS api.cluster-cefas.cefaslocalserver.com no está listado en la salida de ipa dnsrecord-find. Debes agregarlo.
Agregar el Registro DNS Faltante:

Agrega el registro DNS api.cluster-cefas para el dominio cefaslocalserver.com.
bash
Copiar código
ipa dnsrecord-add cefaslocalserver.com api.cluster-cefas --a-rec 10.17.4.21
Verificar el Registro DNS:
Confirma que el registro DNS se ha agregado correctamente.
bash
Copiar código
ipa dnsrecord-show cefaslocalserver.com api.cluster-cefas
Reiniciar el Proceso de Instalación:
Una vez que el registro DNS esté correctamente configurado, vuelve a ejecutar el comando para esperar a que la instalación se complete.