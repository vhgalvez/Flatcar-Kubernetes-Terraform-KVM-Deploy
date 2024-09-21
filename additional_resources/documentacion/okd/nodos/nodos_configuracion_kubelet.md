Aquí tienes los pasos actualizados y ordenados con las rutas correctas para generar y configurar los certificados necesarios para el Kubelet en tu servidor:

Paso 1: Crear el Archivo de Configuración kubelet-openssl.cnf
Navega a la ruta /tmp:

bash
Copiar código
cd /tmp
Crea el archivo kubelet-openssl.cnf:

bash
Copiar código
sudo vim kubelet-openssl.cnf
Pega el siguiente contenido en el archivo:

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
ST = CA
L  = San Francisco
O  = Kubernetes
OU = System
CN = system:node:master1.cefaslocalserver.com

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = master1.cefaslocalserver.com
IP.1 = 10.17.4.21
Guarda y cierra el archivo.

Paso 2: Crear la Clave Privada y el CSR
Genera una nueva clave privada para Kubelet:

bash
Copiar código
sudo openssl genpkey -algorithm RSA -out /etc/kubernetes/pki/kubelet.key -pkeyopt rsa_keygen_bits:2048
Genera una Solicitud de Firma de Certificado (CSR):

bash
Copiar código
sudo openssl req -new -key /etc/kubernetes/pki/kubelet.key -out /etc/kubernetes/pki/kubelet.csr -config /tmp/kubelet-openssl.cnf
Paso 3: Firmar el CSR usando la CA de Kubernetes
Firma el CSR para generar el certificado Kubelet:

bash
Copiar código
sudo openssl x509 -req -in /etc/kubernetes/pki/kubelet.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kubelet.crt -days 365 -extensions req_ext -extfile /tmp/kubelet-openssl.cnf
Paso 4: Configurar Permisos Correctos
Configura los permisos correctos para los archivos generados:

bash
Copiar código
sudo chown root:root /etc/kubernetes/pki/kubelet.key /etc/kubernetes/pki/kubelet.crt
sudo chmod 600 /etc/kubernetes/pki/kubelet.key
sudo chmod 644 /etc/kubernetes/pki/kubelet.crt
Paso 5: Reiniciar el Servicio Kubelet
Reinicia el servicio kubelet:

bash
Copiar código
sudo systemctl restart kubelet
Paso 6: Verificar el Estado del Servicio Kubelet
Verifica que el servicio kubelet esté activo y funcionando correctamente:

bash
Copiar código
sudo systemctl status kubelet
Paso 7: Verificar Registro del Nodo
Verifica que el nodo se haya registrado correctamente en el clúster:

bash
Copiar código
oc get nodes --kubeconfig=/etc/kubernetes/admin.conf




__

Guía para Descordonar un Nodo Usando oc
Dado que kubectl no está disponible y enfrentaste problemas al intentar descordonar un nodo usando kubectl, te proporciono una guía sobre cómo proceder utilizando oc, que ya tienes instalado.

1. Descordonar el Nodo
Como kubectl no está instalado, puedes usar oc para lograr el mismo resultado. El comando equivalente para descordonar el nodo con oc es:

bash
Copiar código
oc adm uncordon master1.cefaslocalserver.com --kubeconfig=/etc/kubernetes/admin.conf
2. Verificar el Estado del Nodo
Después de descordonar el nodo, verifica su estado utilizando el siguiente comando:

bash
Copiar código
oc get nodes --kubeconfig=/etc/kubernetes/admin.conf
Este proceso te permitirá asegurarte de que el nodo esté en el estado correcto y listo para recibir cargas de trabajo.



Para evitar tener que especificar el parámetro --kubeconfig=/etc/kubernetes/admin.conf cada vez que ejecutas un comando oc, puedes hacer que esta configuración sea persistente estableciendo la variable de entorno KUBECONFIG o copiando el archivo de configuración a la ubicación predeterminada.

Opción 1: Establecer la Variable de Entorno KUBECONFIG
Puedes agregar esta variable a tu sesión de terminal actual o hacerla permanente en tu perfil de shell.

Para la sesión actual:

bash
Copiar código
export KUBECONFIG=/etc/kubernetes/admin.conf
Para hacerla permanente:

Agrega la línea anterior al final del archivo de configuración de tu shell (~/.bashrc, ~/.bash_profile, o ~/.zshrc, dependiendo de cuál estés usando).

bash
Copiar código
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> ~/.bashrc
Luego, recarga la configuración de tu shell:

bash
Copiar código
source ~/.bashrc
Opción 2: Copiar el Archivo de Configuración a la Ubicación Predeterminada
Otra opción es copiar el archivo de configuración a la ubicación predeterminada de kubectl y oc, que es ~/.kube/config.

bash
Copiar código
mkdir -p ~/.kube
cp /etc/kubernetes/admin.conf ~/.kube/config
Verificación
Después de realizar cualquiera de estas configuraciones, deberías poder ejecutar oc get nodes sin necesidad de especificar --kubeconfig.

bash
Copiar código
oc get nodes
Esto mostrará los nodos sin necesidad de repetir la ruta al archivo de configuración