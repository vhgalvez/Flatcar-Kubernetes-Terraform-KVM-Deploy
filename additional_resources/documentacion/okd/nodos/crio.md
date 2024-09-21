## Instalación de CRI-O

1. **Descargar el binario de CRI-O en el directorio `/tmp`:**

    ```bash
    sudo wget -O /tmp/crio.tar.gz https://storage.googleapis.com/cri-o/artifacts/cri-o.amd64.v1.30.3.tar.gz
    ```

2. **Crear un directorio temporal para extraer los archivos:**

    ```bash
    sudo mkdir -p /tmp/crio
    ```

3. **Extraer el archivo descargado en el directorio temporal:**

    ```bash
    sudo tar -xzf /tmp/crio.tar.gz -C /tmp/crio
    ```

4. **Crear el directorio `/opt/bin/crio` si no existe:**

    ```bash
    sudo mkdir -p /opt/bin/crio
    ```

5. **Mover los binarios a `/opt/bin/crio/` donde deberían tener permisos de escritura:**

    ```bash
    sudo mv /tmp/crio/cri-o/bin/* /opt/bin/crio/
    ```

6. **Verificar la instalación de CRI-O:**

    ```bash
    /opt/bin/crio/crio --version
    ```

Estos pasos asegurarán que el binario de CRI-O se descargue, extraiga y coloque en el directorio adecuado para su ejecución.
