
# Guía para Configurar la Variable `KUBECONFIG` de Manera Persistente en Kubernetes

En esta guía, aprenderás a configurar la variable `KUBECONFIG` correctamente para que puedas acceder al clúster de Kubernetes sin tener que especificar manualmente el archivo de configuración cada vez que uses `oc` o `kubectl`.

## 1. Configurar la Variable `KUBECONFIG`

### 1.1. Editar el archivo `.bashrc` de Root

Para que el archivo `admin.conf` sea utilizado de forma predeterminada, edita el archivo `.bashrc` de root agregando la siguiente línea:

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Guarda y cierra el archivo. Ahora recarga la configuración:

```bash
source /root/.bashrc
```

## 2. Evitar el uso de sudo en source

No necesitas usar sudo para ejecutar source. El comando source carga el archivo en el contexto actual del shell, y no puede ejecutarse con sudo. Simplemente usa:

```bash
source ~/.bashrc
```

## 3. Verificar permisos y propiedades
Para asegurarte de que el archivo /etc/kubernetes/admin.conf es accesible, ajusta los permisos y la propiedad como se mostró antes:

```bash
sudo chmod 600 /etc/kubernetes/admin.conf
sudo chown root:root /etc/kubernetes/admin.conf
```

## 4. Verificar persistencia tras reiniciar
   
Si los cambios aún no persisten después de reiniciar, asegúrate de que el archivo .bashrc o .bash_profile esté configurado correctamente para ejecutarse en cada inicio de sesión.

### 4.1. Verificar .bash_profile

Si los cambios en .bashrc no tienen efecto, edita también el archivo .bash_profile para asegurar que se cargue correctamente en el inicio de sesión:

```bash
sudo vim /root/.bash_profile
```

Asegúrate de que el archivo incluya lo siguiente:

```bash
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

Esto asegura que el archivo .bashrc se cargue en cada inicio de sesión.

## 5. Prueba final

Después de hacer estos cambios, reinicia el sistema para asegurarte de que el archivo admin.conf se cargue correctamente y no tengas que especificarlo manualmente cada vez que ejecutes oc.

```bash
sudo reboot
```

Tras el reinicio, prueba:

```bash
oc get nodes
```

Esto debería funcionar sin tener que especificar manualmente `--kubeconfig`.