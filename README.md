# 2026ASIR2_CN05

Repositorio con plantillas y scripts para desplegar una infraestructura de AWS y preparar un servidor WordPress.

## 💡 Prompt para IA con el que obtener la plantilla de cloudformation

```
Crea una plantilla de Plantilla cloud formation con 1 VPC, 1 subred pública /24, 1 subred privada /24, CIDR IPv4 10.1NN.0.0/16, CIDR IPv6 de AWS (AWS::EC2::VPCCidrBlock),  IGW, EIGW con sus tablas de enrutamiento también con rutas al internet gateway y egresss internet gateway.
La etiqueta Name debe empezar por el nombre de la pila + NN + el recurso 
Debe asignar por defecto IPv4 pública e IPv6 en la subred pública e IPv6 en la subred privada
```

## 📁 Archivos

### ☁️ `VPC_2Subnet_IGW_EIGW.yaml`

Plantilla de AWS CloudFormation que crea:

- Una VPC con un bloque IPv4 configurable y un bloque IPv6 asignado por AWS .
- Una subred pública y otra privada, cada una con direccionamiento IPv4 e IPv6 (AWS::EC2::VPCCidrBlock).
- Un Internet Gateway (IGW) y un Egress-Only Internet Gateway (EIGW).
- Tablas de rutas: la subred pública tiene salida IPv4 e IPv6 mediante el IGW; la privada tiene salida IPv6 mediante el EIGW.
- Las asociaciones entre subredes y tablas de rutas.

El parámetro `NN` es un identificador de dos dígitos que personaliza los bloques CIDR IPv4 y las etiquetas de los recursos. La plantilla devuelve los identificadores de la VPC y de ambas subredes.

### 🖥️ `userdata_cloudwatch_systemmanager_Wordpress.sh`

Script de *user data* para una instancia Ubuntu. Instala y configura:

- SSM Agent para administrar la instancia con AWS Systems Manager.
- Apache y los módulos/extensiones PHP requeridos por WordPress.
- La última versión de WordPress, que copia en `/var/www/html`.
- CloudWatch Agent para enviar a CloudWatch Logs los registros de syslog y los registros de acceso y errores de Apache. Usa el grupo `/ec2/ubuntu24` y la región `us-east-1`.

El script prepara los archivos de WordPress, pero no crea ni configura la base de datos: se necesita una base de datos externa, por ejemplo Amazon RDS, y completar la configuración inicial de WordPress. La instancia también debe tener permisos IAM para Systems Manager y CloudWatch, además de conectividad a los servicios de AWS. En una subred privada sin salida directa a Internet, hay que configurar la conectividad necesaria, por ejemplo mediante NAT o endpoints de VPC.


## ✅ Comprobación de la plantilla y del script

### ☁️ Revisar la plantilla de CloudFormation

Ejecutar la plantilla
Comprobar que se crean los recursos y que las subredes tienen direccionamiento IPv4 e IPv6.

### 🖥️ Creación de EC2 con el script de *user data* y *LabRole*

* Comprobar Apache con Wordpress
* Comprobar acceso por System Manager (Sin puerto 22)
* Comprobar envío de logs a CloudWatch Logs

## 🐙 GitHub

### ➕ Crear repo en IESAlisal

Crear el repositorio en Github y subir los archivos. Por ejemplo, para un repositorio llamado `2027ASIR2_CNNN`:

https://github.com/IESAlisal/2027ASIR2_CNNN

### 📥 Clonar el repo en local con VSCode

```bash
git clone https://github.com/IESAlisal/2027ASIR2_CNNN.git
```

### ⚙️ Inicializar el repo local con VSCode

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu_correo@ejemplo.com"
```
### 📄 Poner los archivos en el repo local con VSCode

Archivo .yaml con la plantilla de CloudFormation y el script .sh de *user data*.


### 🚀 Subir cambios al repo remoto con VSCode

```bash 
git add .
git commit -m "comentario"          
git push origin main
```

## 📌 Notas

https://github.com/IESAlisal/2026ASIR2_CN05
