1. Crear el archivo del servicio
Para empezar, crea un archivo de servicio en el sistema en la ruta /etc/systemd/system/download-certificates.service. Esto definirá las acciones que el sistema llevará a cabo cuando inicies el servicio.

bash
Copiar código
sudo nano /etc/systemd/system/download-certificates.service
2. Definir el contenido del servicio
Dentro del archivo creado, pega el siguiente contenido. Este configura el servicio para ejecutar un script que descarga los certificados necesarios.

ini
Copiar código
[Unit]
Description=Download Kubernetes Certificates
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/bin/bash /home/core/download-certificates.sh
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
ExecStart: Define el comando o script que se ejecutará al iniciarse el servicio.
RemainAfterExit=true: Indica que el servicio se considera activo incluso después de que el script finalice su ejecución.
3. Recargar systemd y habilitar el servicio
Una vez que hayas guardado el archivo del servicio, es necesario recargar systemd para que pueda reconocer el nuevo servicio que acabas de definir.

bash
Copiar código
sudo systemctl daemon-reload
Luego, habilita el servicio para que se ejecute automáticamente en cada inicio del sistema:

bash
Copiar código
sudo systemctl enable download-certificates.service
4. Iniciar el servicio manualmente
Para iniciar el servicio de manera manual (cuando lo necesites), utiliza el siguiente comando:

bash
Copiar código
sudo systemctl start download-certificates.service
5. Verificar el estado del servicio
Si deseas verificar que el servicio se ejecutó correctamente, usa el siguiente comando:

bash
Copiar código
sudo systemctl status download-certificates.service
Esto te mostrará el estado actual del servicio y cualquier mensaje de log generado durante su ejecución, lo que te permitirá ver si todo funcionó correctamente.

6. Solución de problemas
En caso de que el servicio no funcione como se espera, puedes revisar los logs detallados del servicio usando el comando:

bash
Copiar código
sudo journalctl -u download-certificates.service
Este comando te mostrará los errores o advertencias que hayan ocurrido durante la ejecución del servicio, lo que puede ser útil para solucionar problemas.