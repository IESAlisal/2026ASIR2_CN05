# 2026ASIR2_CN05

Repositorio con plantillas y scripts para desplegar una infraestructura de AWS y preparar un servidor WordPress.

## Archivos

### `RA03/lunes.yaml`

Plantilla de AWS CloudFormation que crea:

- Una VPC con un bloque IPv4 configurable y un bloque IPv6 asignado por AWS.
- Una subred pública y otra privada, cada una con direccionamiento IPv4 e IPv6.
- Un Internet Gateway (IGW) y un Egress-Only Internet Gateway (EIGW).
- Tablas de rutas: la subred pública tiene salida IPv4 e IPv6 mediante el IGW; la privada tiene salida IPv6 mediante el EIGW.
- Las asociaciones entre subredes y tablas de rutas.

El parámetro `NN` es un identificador de dos dígitos que personaliza los bloques CIDR IPv4 y las etiquetas de los recursos. La plantilla devuelve los identificadores de la VPC y de ambas subredes.

### `RA03/userdata_cloudwatch_systemmanager_Wordpress.sh`

Script de *user data* para una instancia Ubuntu. Instala y configura:

- SSM Agent para administrar la instancia con AWS Systems Manager.
- Apache y los módulos/extensiones PHP requeridos por WordPress.
- La última versión de WordPress, que copia en `/var/www/html`.
- CloudWatch Agent para enviar a CloudWatch Logs los registros de syslog y los registros de acceso y errores de Apache. Usa el grupo `/ec2/ubuntu24` y la región `us-east-1`.

El script prepara los archivos de WordPress, pero no crea ni configura la base de datos: se necesita una base de datos externa, por ejemplo Amazon RDS, y completar la configuración inicial de WordPress. La instancia también debe tener permisos IAM para Systems Manager y CloudWatch, además de conectividad a los servicios de AWS. En una subred privada sin salida directa a Internet, hay que configurar la conectividad necesaria, por ejemplo mediante NAT o endpoints de VPC.
