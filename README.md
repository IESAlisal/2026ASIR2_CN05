# 2026ASIR2_CN05

Repositorio con plantillas y scripts para desplegar una infraestructura de AWS y preparar un servidor WordPress.

## 💡 Prompt para IA con el que obtener la plantilla de cloudformation

```
Crea una plantilla de Plantilla cloud formation con 1 VPC, 1 subred pública /24, 1 subred privada /24, CIDR IPv4 10.1NN.0.0/16, CIDR IPv6 de AWS (AWS::EC2::VPCCidrBlock),  IGW, EIGW con sus tablas de enrutamiento también con rutas al internet gateway y egresss internet gateway.
La etiqueta Name debe empezar por el nombre de la pila + NN + el recurso 
Debe asignar por defecto IPv4 pública e IPv6 en la subred pública e IPv6 en la subred privada
```

## 📁 Archivos

### ☁️ `VPC_EC2.yaml`

Plantilla de AWS CloudFormation que crea:

- Una VPC con un bloque IPv4 configurable y un bloque IPv6 asignado por AWS .
- Una subred pública y otra privada, cada una con direccionamiento IPv4 e IPv6 (AWS::EC2::VPCCidrBlock).
- Un Internet Gateway (IGW) y un Egress-Only Internet Gateway (EIGW).
- Tablas de rutas: la subred pública tiene salida IPv4 e IPv6 mediante el IGW; la privada tiene salida IPv6 mediante el EIGW.
- Las asociaciones entre subredes y tablas de rutas.
- Una instancia Ubuntu 24.04 LTS x86_64 en la subred pública, con Apache, PHP y WordPress instalados mediante *user data* basado en el script de `RA03`.
- Un grupo de seguridad que permite HTTP IPv4 desde el rango configurado, sin abrir SSH. La administración se realiza mediante Systems Manager Session Manager.
- Un rol y un perfil IAM para Systems Manager y CloudWatch Agent, y un grupo de logs con 30 días de retención.

El parámetro `NN` es un identificador de dos dígitos que personaliza los bloques CIDR IPv4 y las etiquetas de los recursos. `InstanceType` permite elegir entre `t3.micro`, `t3.small` y `t3.medium`. `UbuntuAmi` selecciona el parámetro público de Systems Manager para la AMI oficial de Ubuntu 24.04 x86_64; si se cambia, debe apuntar a una AMI compatible con el script. `WebAccessCidr` controla el acceso HTTP IPv4 y por defecto permite cualquier origen; es recomendable restringirlo, especialmente durante la instalación. No se habilita acceso HTTP entrante por IPv6.

La plantilla devuelve los identificadores de la VPC, las subredes y la instancia, la URL HTTP inicial y el nombre del grupo de logs. El *user data* integrado usa la región de la pila y su propio grupo de logs; el script independiente conserva sus valores originales.

WordPress necesita una base de datos MySQL o MariaDB externa: la plantilla no crea RDS ni configura sus credenciales. Debe ser accesible desde la instancia, con sus reglas de red correspondientes. Tras el arranque, abre la URL indicada en `WordPressUrl` desde una dirección permitida y completa el asistente con los datos de la base de datos. Protege la instalación: HTTP no cifra credenciales ni tráfico; para producción configura HTTPS y restringe el acceso. La dirección IP pública puede cambiar si se detiene y arranca la instancia.

Ejemplo de despliegue (sustituye el rango por tu IP o red):

```bash
aws cloudformation deploy --template-file VPC_EC2.yaml --stack-name wordpress --capabilities CAPABILITY_IAM --parameter-overrides NN=01 WebAccessCidr=203.0.113.0/24
```

La creación de la pila no garantiza que el *user data* haya terminado correctamente. Consulta `/var/log/cloud-init-output.log` mediante Session Manager si falla la instalación; después comprueba los logs de syslog y Apache en el grupo de CloudWatch indicado en las salidas. La instancia, el volumen y los servicios utilizados pueden generar costes.

### 🖥️ `RA03/userdata_cloudwatch_systemmanager_Wordpress.sh`

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

### 🖥️ Comprobar la EC2 y el *user data*

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
