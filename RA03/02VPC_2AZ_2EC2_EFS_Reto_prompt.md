# Prompt para generar una plantilla de CloudFormation con los siguientes requisitos:

## Creación VPC, Subredes, IGW, EIGW, Tablas de enrutamiento y 2 instancias EC2 Ubuntu 26.04

La plantilla se debe llamar `02VPC_2AZ_2EC2_EFS.yaml` y debe cumplir con los siguientes requisitos:

Crea una plantilla de Plantilla cloud formation con un parámetro NN (un número de dos cifras)
- 1 VPC, 2 Zonas de disponibilidad, 2 subredes públicas /24, 2 subredes privadas /24. Debe asignar por defecto IPv4 pública e IPv6 en la subred pública e IPv6 en la subred privada
- Rango CIDR de la VPC IPv4 10.1NN.0.0/16, CIDR IPv6 de AWS  (AWS::EC2::VPCCidrBlock),  IGW, EIGW con sus tablas de enrutamiento también con rutas al internet gateway y egresss internet gateway.
- Las subredes deben ser IPv4 /24 e IPv6 /64
- Todos los recursos etiquetados con la etiqueta Name que debe empezar por el nombre de la pila + NN + el recurso 

- Añadir una instancia EC2 ubuntu 26.04 en cada subred publica con el mismo grupo de seguridad y el mismo userdata obtenido del fichero del repositorio userdata_cloudwatch_systemmanager_Wordpress.sh
- Las instancias deben utilizar el Rol de IAM "LabRole" ya existente
- Cada instancia debe tener la IPv4 terminada en 100 

## Creación de un sistema de archivos EFS

- Crear un sistema de archivos EFS (AWS::EFS::FileSystem) cifrado, etiquetado con la etiqueta Name
- Crear un target de montaje (AWS::EFS::MountTarget) en cada subred privada, para que el EFS sea accesible desde las dos zonas de disponibilidad
- Crear un grupo de seguridad propio para el EFS (etiquetado con Name) cuya regla de entrada permita el puerto TCP 2049 (NFS) unicamente desde el grupo de seguridad de las instancias EC2 (origen = grupo de seguridad, no un rango CIDR) y asignarlo a los targets de montaje
- Añadir a las salidas (Outputs) el ID del sistema de archivos EFS

## Montaje del EFS desde el userdata

- Modificar el userdata de las dos instancias (el mismo en ambas) para que, además de todo lo que hace el fichero userdata_cloudwatch_systemmanager_Wordpress.sh, monte el EFS creado en la plantilla
- Instalar el cliente NFS (paquete nfs-common) y montar el EFS por NFSv4.1 usando su nombre DNS (ID del EFS + .efs. + región + .amazonaws.com) obtenido de la propia plantilla con !Sub, sin escribir el ID ni la región a mano
- Montarlo en /var/www/html/wp-content/uploads (crear el directorio si no existe), despues de copiar WordPress y antes de ajustar los permisos con chown a www-data, para que ambas instancias compartan los ficheros subidos
- Hacer el montaje persistente añadiendo una linea a /etc/fstab (con la opción _netdev) y ejecutar mount -a
- Las instancias deben depender (DependsOn) de los targets de montaje del EFS para que el EFS este disponible cuando arranque el userdata
- Al usar !Sub en el userdata, escapar con ${!VAR} cualquier variable de bash que use llaves

## Verificación del montaje del EFS

Desde una sesión de Systems Manager en cada instancia, comprobar que el EFS está montado:

- `df -hT /var/www/html/wp-content/uploads` debe mostrar tipo nfs4 y como origen el nombre DNS del EFS
- `findmnt /var/www/html/wp-content/uploads` debe mostrar el punto de montaje, el origen y las opciones (no muestra nada si no está montado)
- `mount | grep efs` debe mostrar la línea del montaje NFS
- `sudo mount -a` no debe dar errores (comprueba la línea de /etc/fstab)
- Comprobar que se comparte entre instancias: crear un fichero con `echo prueba | sudo tee /var/www/html/wp-content/uploads/test.txt` en una instancia y comprobar con `ls /var/www/html/wp-content/uploads` que aparece en la otra

- No utilices tildes/acentos en la plantilla 