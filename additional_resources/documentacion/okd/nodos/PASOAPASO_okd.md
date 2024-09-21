# Paso a Paso del Proyecto

## 1. Preparación del Sistema

### 1.1 Instalación de Dependencias
- Instalación de QEMU, KVM y Libvirt y Terraform

### 1.2 Configuración del Pool de Almacenamiento
- Creación Manual del Pool default
- Verificación de la Creación del Pool

### 1.3 Descarga de Imágenes
- Descarga de la Imagen de Flatcar Linux y Verificación
- Creación de la Imagen QCOW2 para Flatcar Linux
- Descarga de la Imagen de Rocky Linux 9

## 2. Clonar el Repositorio de Terraform

### 2.1 Clonar el Repositorio
- Clonar el Repositorio de Terraform

## 3. Configuración de un Adaptador Puente (Bridge)

### 3.1 Configuración usando nmcli
- Crear el Puente br0
- Agregar una Interfaz Esclava al Puente
- Configurar el Puente para Obtener una Dirección IP Automáticamente
- Reiniciar NetworkManager
- Activar la Conexión del Puente y la Interfaz Esclava
- Verificar el Estado del Puente

### 3.2 Configuración editando Archivos de Configuración
- Generar UUIDs para las Conexiones
- Crear el Archivo de Configuración del Puente br0
- Crear el Archivo de Configuración para la Interfaz Esclava
- Reiniciar NetworkManager
- Activar las Conexiones
- Verificar el Estado del Puente

## 4. Configuración de NAT y Reenvío de Tráfico en Rocky Linux 9

### 4.1 Configuración de Reenvío de IP
- Habilitar Reenvío de IP

### 4.2 Configuración de iptables
- Crear el Archivo de Configuración de iptables
- Reiniciar el Servicio iptables

### 4.3 Configuración de Rutas IP
- Agregar Rutas IP en cada Máquina Virtual

### 4.4 Verificación de la Configuración de Red
- Verificar la Configuración de Red en las Máquinas Virtuales

### 4.5 Comprobación de Conectividad
- Probar Conectividad a Internet y entre Subredes

## 5. Despliegue de Máquinas Virtuales con Terraform

### 5.1 Inicializar y Aplicar Terraform para br0_network
- Inicializar Terraform y Actualizar Proveedores
- Aplicar la Configuración de Terraform

### 5.2 Inicializar y Aplicar Terraform para nat_network_02
- Inicializar Terraform y Actualizar Proveedores
- Aplicar la Configuración de Terraform

### 5.3 Inicializar y Aplicar Terraform para nat_network_03
- Inicializar Terraform y Actualizar Proveedores
- Aplicar la Configuración de Terraform

## 6. Configuración del Servidor DNS con FreeIPA

### 6.1 Instalación de FreeIPA y Servidor DNS
- Actualización del Sistema
- Verificación del Nombre de Host
- Instalación de FreeIPA

### 6.2 Configuración del DNS en FreeIPA
- Añadir Entradas DNS
- Configuración de Reenvío de DNS

### 6.3 Verificación de la Configuración del DNS
- Verificación de la Configuración del DNS
- Configuración del DNSSEC

## 7. Configuración del Balanceador de Carga (Traefik)

### 7.1 Instalación de Traefik
- Descarga e Instalación de Traefik

### 7.2 Configuración de Traefik para Balanceo de Carga
- Configuración de Traefik para Balancear el Tráfico

## 8. Instalación y Configuración de OKD

### 8.1 Descargar e Instalar el Instalador de OKD
- Descarga del Instalador de OKD
- Verificación y Extracción del Archivo

### 8.2 Crear el Archivo de Configuración de Instalación
- Creación del Archivo install-config.yaml

### 8.3 Generar y Aplicar Manifiestos
- Generar Manifiestos
- Generar Configuraciones Ignition

### 8.4 Iniciar la Instalación del Clúster
- Inicio de la Instalación del Clúster de OKD

### 8.5 Verificación de la Instalación
- Verificación de los Nodos
- Aprobación de Solicitudes de Certificado (CSR)
- Esperar la Compleción de la Instalación

Este esquema organiza las tareas en el orden necesario para la configuración y despliegue del entorno, asegurando que cada componente esté preparado antes de pasar al siguiente.
