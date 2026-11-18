# Prompt para generar una plantilla de CloudFormation con los siguientes requisitos:

## Creación VPC, Subredes, IGW, EIGW, Tablas de enrutamiento

Crea una plantilla de Plantilla cloud formation con un parámetro NN (un número de dos cifras)
- 1 VPC, 2 Zonas de disponibilidad, 2 subredes públicas /24, 2 subredes privadas /24. Debe asignar por defecto IPv4 pública e IPv6 en la subred pública e IPv6 en la subred privada
- Rango CIDR de la VPC IPv4 10.1NN.0.0/16, CIDR IPv6 de AWS  (AWS::EC2::VPCCidrBlock),  IGW, EIGW con sus tablas de enrutamiento también con rutas al internet gateway y egresss internet gateway.
- Las subredes deben ser IPv4 /24 e IPv6 /64
- Todos los recursos etiquetados con la etiqueta Name que debe empezar por el nombre de la pila + NN + el recurso 


- No utilices tildes/acentos en la plantilla 