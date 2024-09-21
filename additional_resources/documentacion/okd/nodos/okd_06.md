Paso 1: Conectar y Preparar el Nodo Bootstrap
Accede al nodo bootstrap1:

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.14 -p 22
Instalar paquetes necesarios:

bash
Copiar código
sudo dnf install -y wget tar
Descargar el Cliente de OpenShift (oc):

bash
Copiar código
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
Extraer y mover el Cliente:

bash
Copiar código
tar -xzvf openshift-client-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
sudo mv oc /usr/local/bin/
sudo chmod +x /usr/local/bin/oc
Verificar la Instalación:

bash
Copiar código
oc version
Paso 2: Descargar e Instalar el Instalador de OKD
Descargar openshift-install:

bash
Copiar código
wget https://github.com/okd-project/okd/releases/download/4.12.0-0.okd-2023-03-18-084815/openshift-install-linux-4.12.0-0.okd-2023-03-18-084815.tar.gz
Extraer y mover el Instalador:

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
Paso 4: Instalar y Configurar el Servidor HTTP en Nodo Bootstrap
Instalar Apache HTTP Server (httpd):

bash
Copiar código
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
Configurar el Servidor HTTP:

bash
Copiar código
sudo mkdir -p /var/www/html/okd
sudo chown -R core:core /var/www/html/okd
sudo chmod -R 755 /var/www/html/okd
Reiniciar el Servicio httpd:

bash
Copiar código
sudo systemctl restart httpd
Paso 5: Descargar Imágenes de Bootstrap y Cluster
Descargar la imagen de Bootstrap:

bash
Copiar código
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.12/latest/rhcos-4.12.0-x86_64-live-initramfs.x86_64.img -O /var/www/html/okd/bootstrap-image.img
Descargar la imagen del Cluster:

bash
Copiar código
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/dependencies/rhcos/4.12/latest/rhcos-4.12.0-x86_64-live.x86_64.iso -O /var/www/html/okd/cluster-image.iso
Verificar la accesibilidad de las imágenes:

bash
Copiar código
curl http://10.17.3.14/okd/bootstrap-image.img
curl http://10.17.3.14/okd/cluster-image.iso
Paso 6: Configurar install-config.yaml
Crear install-config.yaml en el nodo bootstrap:

bash
Copiar código
sudo mkdir okd-install
nano /home/core/okd-install/install-config.yaml
Agregar el siguiente contenido:

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
  machineNetwork:
  - cidr: 10.17.4.0/24
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
Paso 9: Copiar la Clave SSH al Nodo Bootstrap
Copiar la clave SSH:

bash
Copiar código
sudo scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.14:/home/core/.ssh/id_rsa_key_cluster_openshift
Acceder al Nodo Bootstrap:

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.14
Una vez dentro del nodo bootstrap, ejecutar los siguientes comandos:

bash
Copiar código
mv /home/core/.ssh/id_rsa_key_cluster_openshift /home/core/.ssh/id_rsa
chmod 600 /home/core/.ssh/id_rsa