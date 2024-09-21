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
Conéctate a tu servidor FreeIPA y autentícate como administrador.

bash
Copiar código
sudo ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.11 -p 22
Autentícate como administrador en FreeIPA.

bash
Copiar código
kinit admin
Password for admin@CEFASLOCALSERVER.COM:
Configura las entradas DNS necesarias para tu clúster en FreeIPA.

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
4.1 Transferir los Archivos Ignition
Desde el servidor físico donde se generaron los archivos Ignition:

bash
Copiar código
sudo scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift /home/victory/terraform-openshift-kvm-deployment_linux_Flatcar/nat_network_03/ignition-configs/*.ign core@10.17.3.14:/home/core/
Paso 5: Crear el Archivo de Configuración de Instalación
5.1 Crear install-config.yaml
En el nodo bootstrap, crea el archivo install-config.yaml:

bash
Copiar código
nano /home/core/okd-install/install-config.yaml
Y agrega el siguiente contenido:

yaml
Copiar código
apiVersion: v1
baseDomain: cefaslocalserver.com
metadata:
  name: produccion
compute:
- name: worker
  replicas: 3
controlPlane:
  replicas: 3
platform:
  none: {}
networking:
  networkType: OpenShiftSDN
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  serviceNetwork:
  - 172.30.0.0/16
pullSecret: '{"auths": ...}'  # Reemplaza con tu pull secret
sshKey: 'ssh-rsa AAAA...'  # Reemplaza con tu clave SSH pública
Paso 6: Generar y Aplicar Manifiestos
6.1 Generar Manifiestos
En el nodo bootstrap, genera los manifiestos:

bash
Copiar código
openshift-install create manifests --dir=/home/core/okd-install
6.2 Generar Configuraciones Ignition
En el nodo bootstrap, genera las configuraciones Ignition:

bash
Copiar código
openshift-install create ignition-configs --dir=/home/core/okd-install
Paso 7: Crear el Clúster
7.1 Iniciar la Instalación
Inicia la instalación del clúster de OKD:

bash
Copiar código
openshift-install create cluster --dir=/home/core/okd-install --log-level=debug
Paso 8: Verificar la Instalación
8.1 Verificar Nodos
Verifica que todos los nodos del clúster estén funcionando correctamente:

bash
Copiar código
oc get nodes
8.2 Aprobar Solicitudes de Certificado (CSR)
Aprobar las solicitudes de certificados que sean necesarias para que los nodos se unan al clúster:

bash
Copiar código
oc get csr
oc certificate approve <CSR_NAME>
Paso 9: Completar la Instalación
9.1 Esperar la Finalización
Sigue los logs y espera a que la instalación se complete:

bash
Copiar código
openshift-install --dir=/home/core/okd-install wait-for install-complete --log-level=debug
Recursos Adicionales
Para más detalles y pasos avanzados, consulta los siguientes recursos:

Documentación de OKD
Guía de instalación de OKD
Releases de OKD en GitHub
Explicación de las Entradas DNS y el Rol del Balanceador de Carga (Traefik)
Arquitectura del Clúster OKD
IP Pública (192.168.0.21): La dirección IP accesible desde el exterior para conexiones HTTPS.
Load Balancer (Traefik, IP: 10.17.3.12): Distribuye el tráfico entrante a los nodos maestros.
Nodos Maestros: (Master 1, Master 2, Master 3): Gestionan el plano de control del clúster.
Nodo Bootstrap (IP: 10.17.3.14): Utilizado durante la configuración inicial del clúster.
Nodos Trabajadores: (Worker 1, Worker 2, Worker 3): Ejecutan las aplicaciones desplegadas en el clúster.
FreeIPA (IP: 10.17.3.11): Proporciona servicios de DNS y gestión de identidades.
Nodo Bastion (IP: 192.168.0.20): Punto de acceso seguro para la gestión del clúster.
Entradas DNS
Estas entradas DNS se configuran para facilitar el acceso a diferentes componentes del clúster.

bash
Copiar código
ipa dnsrecord-add cefaslocalserver.com api.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com api-int.produccion --a-rec=10.17.4.21
ipa dnsrecord-add cefaslocalserver.com master1.produccion --a-rec=10.17.4.21
Propósitos de las Entradas DNS
api.produccion (10.17.4.21):

Propósito: Facilita el acceso externo a la API del clúster OKD.
Uso: Utilizado por desarrolladores, herramientas CI/CD y otros usuarios externos para interactuar con el clúster.
api-int.produccion (10.17.4.21):

Propósito: Facilita el acceso interno a la API del clúster OKD.
Uso: Utilizado por componentes internos del clúster para comunicarse entre sí, optimizando la eficiencia y reduciendo la carga en el balanceador de carga.
master1.produccion (10.17.4.21):

Propósito: Permite el acceso directo al primer nodo maestro.
Uso: Utilizado para tareas de administración, mantenimiento y monitoreo específico del nodo maestro.
Función del Balanceador de Carga (Traefik)
El Balanceador de Carga (Traefik) con la IP 10.17.3.12 se utiliza para distribuir el tráfico entrante entre los nodos maestros. Aunque api.produccion y api-int.produccion apuntan a la misma IP (10.17.4.21), en la práctica, el balanceador de carga dirige este tráfico a los nodos maestros disponibles.

Diagramas de Flujo de Conexión
Conexiones Entrantes:

Traefik: Las conexiones HTTPS entrantes llegan a Traefik y se distribuyen entre los nodos maestros.
DNS: Las entradas DNS (api.produccion y api-int.produccion) apuntan a la IP manejada por Traefik, facilitando el balanceo de carga.
Comunicación Interna y Externa:

Interna: api-int.produccion asegura la comunicación entre componentes internos del clúster.
Externa: api.produccion gestiona el acceso externo al clúster.
Esta guía te ayudará a instalar un clúster OKD multinodo en KVM con los pasos necesarios y las configuraciones específicas para asegurar un despliegue exitoso.

Contacto
Para cualquier duda o problema, por favor abra un issue en el repositorio o contacte al mantenedor del proyecto.

Mantenedor del Proyecto: Victor Galvez



___

Para copiar los archivos .ign desde tu máquina local a la máquina bootstrap (bootstrap1) en tu clúster de OpenShift, sigue estos pasos. Aquí asumimos que ya tienes acceso a la máquina bootstrap y que los archivos .ign están en el directorio /home/victory/terraform-openshift-kvm-deployment_linux_Flatcar/nat_network_03/ignition-configs.

Pasos para copiar los archivos .ign a bootstrap1
Navega al directorio que contiene los archivos .ign:

sh
Copiar código
cd /home/victory/terraform-openshift-kvm-deployment_linux_Flatcar/nat_network_03/ignition-configs
Copia los archivos .ign usando scp:
Utiliza el siguiente comando para copiar todos los archivos .ign al servidor bootstrap1. Asegúrate de que el usuario core en el bootstrap tenga permisos suficientes para recibir estos archivos.

sh
Copiar código
scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift *.ign core@10.17.3.14:/home/core/
Este comando:

Usa scp para copiar los archivos.
La opción -i especifica la clave privada SSH a usar para la autenticación.
*.ign indica que se deben copiar todos los archivos con extensión .ign en el directorio actual.
core@10.17.3.14:/home/core/ es el destino en la máquina bootstrap.
Verifica los archivos en la máquina bootstrap:
Una vez que los archivos se han copiado, inicia sesión en la máquina bootstrap para verificar que los archivos .ign se han copiado correctamente.

sh
Copiar código
ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift core@10.17.3.14 -p 22
ls /home/core/*.ign
Pasos detallados
1. Navega al directorio de los archivos .ign
sh
Copiar código
cd /home/victory/terraform-openshift-kvm-deployment_linux_Flatcar/nat_network_03/ignition-configs
2. Copia los archivos con scp
sh
Copiar código
scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift *.ign core@10.17.3.14:/home/core/
3. Verifica los archivos en la máquina bootstrap
sh
Copiar código
ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift core@10.17.3.14 -p 22
ls /home/core/*.ign
Si todos los pasos anteriores se ejecutan correctamente, deberías tener tus archivos Ignition copiados en la máquina bootstrap (bootstrap1) listos para su uso en la configuración de tu clúster OKD.

Si tienes alguna otra duda o problema durante el proceso, no dudes en preguntar.


[victory@physical1 ignition-configs]$ scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift *.ign core@10.17.3.14:/home/core/
Warning: Identity file /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift not accessible: Permission denied.
The authenticity of host '10.17.3.14 (10.17.3.14)' can't be established.
ED25519 key fingerprint is SHA256:HBiQu585YDdke+tP884X+9u06dfTiggVwnoWmEPhj4k.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.17.3.14' (ED25519) to the list of known hosts.
core@10.17.3.14: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
Connection closed
[victory@physical1 ignition-configs]$

ChatGPT
El problema con los permisos de la clave de identidad SSH se puede resolver de la siguiente manera:

Verifica los permisos del archivo de clave privada SSH:
Asegúrate de que el archivo de clave privada SSH tiene los permisos correctos. Los permisos deben ser tales que solo el propietario del archivo tenga acceso de lectura y escritura.

sh
Copiar código
sudo chmod 600 /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift
Asegúrate de que estás utilizando el archivo de clave privada correcto:
Verifica que el archivo de clave privada que estás utilizando es el correcto y tiene los permisos adecuados.

Ejecuta el comando scp con permisos de superusuario:
Dado que el archivo de clave privada está en el directorio /root/.ssh/, necesitas permisos de superusuario para acceder a él.

sh
Copiar código
sudo scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift *.ign core@10.17.3.14:/home/core/
Asegúrate de que el usuario core en el servidor de destino tiene la clave pública correspondiente:
Verifica que la clave pública correspondiente a la clave privada está añadida al archivo ~/.ssh/authorized_keys del usuario core en el servidor de destino (10.17.3.14).

Pasos detallados:
1. Cambia los permisos del archivo de clave privada SSH
sh
Copiar código
sudo chmod 600 /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift
2. Copia los archivos con scp usando permisos de superusuario
sh
Copiar código
sudo scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift *.ign core@10.17.3.14:/home/core/
3. Verifica que la clave pública está en el archivo authorized_keys del usuario core
Asegúrate de que el contenido del archivo id_rsa_key_cluster_openshift.pub (la clave pública) esté en el archivo /home/core/.ssh/authorized_keys en el servidor de destino.

Posibles comandos adicionales para solucionar problemas:
Acceso a la máquina bootstrap y verificación de claves:

sh
Copiar código
ssh -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift core@10.17.3.14 -p 22
cat /home/core/.ssh/authorized_keys
Añadir la clave pública manualmente si es necesario:

sh
Copiar código
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDC9XqGWEd2de3Ud8TgvzFchK2/SYh+WHohA1KEuveXjCbse9aXKmNAZ369vaGFFGrxbSptMeEt41ytEFpU09gAXM6KSsQWGZxfkCJQSWIaIEAdft7QHnTpMeronSgYZIU+5P7/RJcVhHBXfjLHV6giHxFRJ9MF7n6sms38VsuF2s4smI03DWGWP6Ro7siXvd+LBu2gDqosQaZQiz5/FX5YWxvuhq0E/ACas/JE8fjIL9DQPcFrgQkNAv1kHpIWRqSLPwyTMMxGgFxGI8aCTH/Uaxbqa7Qm/aBfdG2lZBE1XU6HRjAToFmqsPJv4LkBxaC1Ag62QPXONNxAA97arICr vhgalvez@gmail.com" >> /home/core/.ssh/authorized_keys
Estos pasos deberían permitirte copiar los archivos .ign a la máquina bootstrap sin problemas. Si continúas teniendo problemas, asegúrate de que los permisos y configuraciones de las claves SSH están configurados correctamente en ambas máquinas (local y destino).


[victory@physical1 ignition-configs]$ sudo chmod 600 /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift
[victory@physical1 ignition-configs]$ sudo scp -i /root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift *.ign core@10.17.3.14:/home/core/
master1.ign                                          100% 3716   534.0KB/s   00:00
master2.ign                                          100% 3554   466.8KB/s   00:00
master3.ign                                          100% 3554     3.2MB/s   00:00
worker1.ign                                          100% 3554     3.3MB/s   00:00
worker2.ign                                          100% 3558     2.9MB/s   00:00
worker3.ign                                          100% 3558     3.5MB/s   00:00
documentoalo
ChatGPT
¡Excelente! Los archivos .ign se copiaron correctamente en el servidor bootstrap. Ahora, deberías estar listo para continuar con la instalación del clúster. Aquí tienes los pasos siguientes:

Verificar los archivos .ign en el servidor bootstrap:
Asegúrate de que los archivos se copiaron correctamente y están en el directorio correcto.

sh
Copiar código
ls -l /home/core/*.ign
Crear los nodos del clúster:
Utiliza el comando openshift-install para crear el clúster. Ya que tienes los archivos .ign en su lugar, este paso iniciará el proceso de configuración de los nodos maestros y trabajadores.

sh
Copiar código
openshift-install create cluster --dir=/home/core/okd-install --log-level=debug
Monitorear el proceso de instalación:
La instalación de OpenShift puede tardar un tiempo. Es importante monitorear los registros para asegurarse de que no haya errores críticos.

sh
Copiar código
tail -f /home/core/okd-install/.openshift_install.log
Si encuentras algún error durante el proceso de instalación, revisa los registros detenidamente y compártelos para poder ofrecer una solución más específica. Aquí tienes algunos comandos útiles para diagnosticar problemas durante la instalación:

Verificar el estado de los nodos:

sh
Copiar código
oc get nodes
Verificar los pods en el clúster:

sh
Copiar código
oc get pods --all-namespaces
Revisar los logs de un pod específico:

sh
Copiar código
oc logs <nombre-del-pod> -n <namespace>
Con estos pasos, deberías poder continuar con la configuración de tu clúster OpenShift. Si hay algún problema específico, por favor proporciona los detalles y los registros relevantes para poder ayudarte mejor.



sudo virsh destroy master1
sudo virsh undefine master1
sudo virsh destroy master2
sudo virsh undefine master2
sudo virsh destroy master3
sudo virsh undefine master3
sudo virsh destroy worker1
sudo virsh undefine worker1
sudo virsh destroy worker2
sudo virsh undefine worker2
sudo virsh destroy worker3
sudo virsh undefine worker3
__________


Pasos para Instalar CRI-O en Flatcar Container Linux
1. Descargar e Instalar binarios de CRI-O:
Primero, asegúrate de tener los binarios necesarios para CRI-O. Debido a que la descarga anterior de CRI-O no funcionó, intentaremos con un enlace diferente o mediante la construcción desde el código fuente si es necesario.

Usar CRI-O desde Source (código fuente)
1. Descargar el código fuente de CRI-O:
sh
Copiar código
curl -L -o crio-v1.21.0.tar.gz https://github.com/cri-o/cri-o/archive/refs/tags/v1.21.0.tar.gz
tar -xzf crio-v1.21.0.tar.gz -C /tmp
2. Construir y mover el binario de crio:
Flatcar no tiene herramientas de construcción por defecto, necesitarás instalarlas o usarlas en un contenedor.

Usar un contenedor para construir CRI-O:
Puedes usar un contenedor de golang para construir CRI-O:

sh
Copiar código
# Ejecuta el contenedor de golang
docker run --rm -it -v /tmp/cri-o-1.21.0:/usr/src/cri-o -w /usr/src/cri-o golang:1.16 bash

# Dentro del contenedor, ejecuta:
make clean
make binaries

# Sal del contenedor y luego copia los binarios a tu sistema Flatcar
exit
sudo cp /tmp/cri-o-1.21.0/bin/* /opt/bin/
sudo chmod +x /opt/bin/crio
Configuración de CRI-O:
Crear la configuración necesaria para CRI-O:
sh
Copiar código
sudo mkdir -p /etc/crio
cat <<EOF | sudo tee /etc/crio/crio.conf
[crio]
version = "1.21.0"
[crio.runtime]
conmon = "/usr/bin/conmon"
EOF
Crear el servicio systemd para CRI-O:
sh
Copiar código
sudo mkdir -p /etc/systemd/system/
cat <<EOF | sudo tee /etc/systemd/system/crio.service
[Unit]
Description=CRI-O daemon
Documentation=https://github.com/cri-o/cri-o
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/opt/bin/crio
Restart=always
RestartSec=10s
StartLimitInterval=0
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
EOF
Habilitar y arrancar el servicio de CRI-O:
sh
Copiar código
sudo systemctl daemon-reload
sudo systemctl enable crio
sudo systemctl start crio
Nota
Flatcar Container Linux es un sistema operativo especializado para ejecutar contenedores y puede no tener todas las herramientas disponibles en sistemas operativos más generales como CentOS o Ubuntu. En caso de cualquier problema con las dependencias, considera usar un sistema operativo que tenga mejor soporte para estas herramientas como Fedora CoreOS que también está diseñado para Kubernetes y OKD.