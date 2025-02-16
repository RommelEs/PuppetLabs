#INSTALACIÓN DE SERVIDOR APACHE: 
MANIFIESTO:
# Lo que vamos hacer es definir qué configuraciones deben aplicarse a los nodos.
cd /etc/puppetlabs/code/environments/production/manifests/
nano site.pp

# site.pp - Manifiesto principal
# Lo que estamos haciendo aqui es que el nodo puppetagent.local recibirá la configuración de Apache.
node 'puppetagent.local' {  
class {  'apache':  }  
} 

# Lo que estamos haciendo aqui es que todos los nodos por defecto también recibirán la configuración de Apache, a menos que se indique lo contrario.
node default { 
class  {  ‘apache’:  } 
} 

# Iremos donde están los directorios donde se almacenan los módulos de Puppet en el entorno de producción.
cd /etc/puppetlabs/code/environments/production/modules 

# Creamos dentro del direcotio de modulos archivos donde estará la configuración del servidor apache.
Mkdir apache 
mkdir {files,manifests} 
cd manifests 
nano init.pp 

 

class apache { 
  # Instalación del paquete apache2 
  package { 'apache2': 
    ensure => installed, 
  } 

  # Asegurar que el servicio apache2 esté corriendo y habilitado 
  service { 'apache2': 
    ensure  => running, 
    enable  => true, 
    require => Package['apache2'], 
  } 

  # Archivo de página web personalizado 
  file { '/var/www/html/index.html': 
    ensure  => file, 
    content => @("EOF"), 
      <html> 
      <head> 
        <title>Página de Rommel</title> 
      </head> 
      <body> 
        <h1>Servidor de Rommel Apache gestionado por Puppet!</h1> 
      </body> 
      </html> 
      | EOF 
    owner   => 'www-data', 
    group   => 'www-data', 
    mode    => '0644', 
    require => Package['apache2'], 
  } 
} 

Verificamos que no haya un error de sintaxis: 
puppet parser validate /etc/puppetlabs/code/environments/production/modules/apache/manifests/init.pp 
puppet parser validate /etc/puppetlabs/code/environments/production/manifests/site.pp

# Reiniciamos el servidor: 
sudo systemctl restart puppetserver 

# Nodo agente: 
# Instalamos apache en el nodo: 
puppet agent –test 

# Verificamos que Apache este instalado y que el servidor este corriendo: 
dpkg –l | grep apache2 
systemctl status apache2 

# Por defecto, Apache podría estar configurado para escuchar solo en localhost (127.0.0.1). Necesitas asegurarte de que está escuchando en todas las interfaces de red (0.0.0.0). 
/etc/apache2/ports.conf 

# En la linea de “Listen 80” cambiarlo de dos formas: 
# Listen 0.0.0.0:80 
# Listen 192.168.1.44:80 

# Buscamos en el navegador el siguiente enlace:
http://192.168.1.44:80

# MODULO: 
# Con la ayuda de Puppet forge: https://forge.puppet.com/modules/puppetlabs/apache/readme
puppet module install puppetlabs-apache --version 12.2.0 # Este módulo te permitirá gestionar la instalación y configuración de Apache en tus nodos.

# Verificar el modulo instalado 
Puppet module list 

# Nos tiene que salir 
/etc/puppetlabs/code/environments/production/modules 
 # ├── puppetlabs-apache (v12.2.0) 

# Aplicar el módulo al nodo 
cd /etc/puppetlabs/code/environments/production/manifests 
nano site.pp 

node 'puppetagent.local' { 
  include apache 
} 

# O utilizar una declaración más detallada: 

node 'puppetagent.local' { 
  class { 'apache': 
    default_vhost => true, 
    mpm_module    => 'prefork', 
  } 
} 

# Sincronizar el catalogo en el agente 
# En el nodo agente: 
sudo /opt/puppetlabs/bin/puppet agent -t
# Y veremos como se nos instalará el servidopr apache con la plantilla por defecto.
