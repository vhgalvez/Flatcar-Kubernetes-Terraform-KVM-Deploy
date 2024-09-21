Generar o descargar los certificados faltantes: Si tienes una CA configurada, puedes generar estos certificados. Aquí tienes un ejemplo de cómo podrías generarlos:

bash
Copiar código
openssl genrsa -out /etc/kubernetes/pki/kube-controller-manager.key 2048
openssl req -new -key /etc/kubernetes/pki/kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out /etc/kubernetes/pki/kube-controller-manager.csr
openssl x509 -req -in /etc/kubernetes/pki/kube-controller-manager.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out /etc/kubernetes/pki/kube-controller-manager.crt -days 365
Esto generará los certificados requeridos (kube-controller-manager.crt y kube-controller-manager.key) y los colocará en el directorio correcto.

Asegurar los permisos correctos para los archivos: Una vez que tengas los certificados, asegúrate de que los permisos sean correctos:

bash
Copiar código
sudo chown root:root /etc/kubernetes/pki/kube-controller-manager.*
sudo chmod 600 /etc/kubernetes/pki/kube-controller-manager.key
Reiniciar el servicio: Una vez que los certificados estén en su lugar, reinicia el servicio del kube-controller-manager:

bash
Copiar código
sudo systemctl restart kube-controller-manager
Verificar el estado del servicio: Finalmente, verifica el estado del servicio para confirmar que se está ejecutando correctamente:

bash
Copiar código
sudo systemctl status kube-controller-manager