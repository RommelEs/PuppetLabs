# Instalación de Puppet desde Ubuntu 22.04
# Actualizamos todo. 
sudo apt update
sudo apt upgrade

# Root ubuntu: 
sudo -i

# Instalación de herramienta básica 
sudo apt install -y wget

# Configuración de hostname del servidor maestro
sudo hostnamectl set-hostname puppetmaster.local
echo "127.0.0.1 $(hostname)" | sudo tee -a /etc/hosts” 

# Configuración de hostname del servidor agente
sudo hostnamectl set-hostname puppetagent.local
echo "127.0.0.1 $(hostname)" | sudo tee -a /etc/hosts” 

# Usa wget para descargar el archivo desde el repositorio de Puppet: 
# Puppet Open Source 
wget 'https://apt.puppetlabs.com/puppet-release-jammy.deb'

# Ejecuta el instalador: 
# Puppet Open Source: 
dpkg –i puppet-release-jammy.deb

# Actualizamos para obtener el servidor maestro o el servidor agente.
apt update

# Instalación del servidor maestro:
apt install puppetserver

# Instalación del servidor agente:
apt install puppet-agent

# Configuración del servidor maestro:
# Usa la opción puppet config para establecer los valores para el dns_alt_names de la configuración: 
/opt/puppetlabs/bin/puppet config set dns_alt_names 'puppet,puppetmaster.local' -section main 
 
# Si inspecciona el archivo de configuración, verá que se ha añadido la configuración: 
cat /etc/puppetlabs/puppet/puppet.conf

# [main]  
# dns_alt_names = puppet,puppet.example.com

# Actualizamos el Puppet master /etc/hosts para resolver las direcciones IP de sus nodos gestionados. Por ejemplo, su /etc/hosts El archivo podría parecerse a lo siguiente: 
Sudo nano /etc/hosts

# 127.0.0.1   localhost 
# 192.168.1.42   puppetmaster.local puppet 
# 192.168.1.44   puppetagent.local agent 
# The following lines are desirable for IPv6 capable hosts 
# ::1     localhost ip6-localhost ip6-loopback 
# ff02::1 ip6-allnodes 
# ff02::2 ip6-allrouters

# Modificar puppet principal: 
Sudo nano /etc/default/pupetserver

# JAVA_ARGS="-Xms2g -Xmx2g Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"

# Inicia el servicio: 
sudo systemctl start puppetserver 
sudo systemctl enable puppetserver 

# Verifica que Puppet Server esté funcionando: 
sudo systemctl status puppetserver

# Por defecto, el Puppet master escucha las conexiones de los clientes en el puerto 8140. Si el puppetserver Si el servicio no se inicia, compruebe que el puerto no esté ya en uso: 
netstat -anpl | grep 8140 

# O habilitamos el puerto que utiliza puppet en nuestras máquinas. 
# Habilitar UFW: 
sudo ufw enable 

#Permitir el puerto 8140 para Puppet: 
sudo ufw allow 8140/tcp 

# Verificar las reglas de UFW: 
sudo ufw statu

# Configuración del servidor agente:
# Modifique los archivos hosts de sus nodos administrados para resolver la IP del Puppet master. Para ello, añada una línea como: 
sudo /etc/hosts 
# 192.168.1.42    puppetmaster.local puppet 

# En cada nodo gestionado, utilice la opción puppet config para establecer el valor de su server en el FQDN del maestro: 
/opt/puppetlabs/bin/puppet config set server 'puppetmaster.local' --section main

# Si inspecciona el archivo de configuración de los nodos, verá que se ha añadido la configuración: 
cat /etc/puppetlabs/puppet/puppet.conf 
# [main] 
# server = puppetmaster.com

# Usa la opción puppet resource para iniciar y activar el servicio de agente Puppet: 
/opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true

# En los sistemas systemd, el comando anterior es equivalente a usar estos dos systemctl comandos: 
systemctl start puppet 
systemctl enable puppet

# Verifica que Puppet Server esté funcionando: 
sudo systemctl status puppet

# Firmamos el certificado del cliente: 
# Antes de que sus nodos administrados puedan recibir configuraciones del maestro, primero deben ser autenticados: 
# En sus agentes Puppet, genere un certificado para que el Puppet master lo firme: 
/opt/puppetlabs/bin/puppet agent -t

# Este comando emitirá un error, indicando que no se ha encontrado ningún certificado. Este error se debe a que el certificado generado debe ser aprobado por el Puppet master. 
# Cuando Puppet se configura por primera vez, necesita generar un certificado para el servidor (Puppet Master). Si ese archivo no está presente, puedes intentar regenerar los certificados. 
# Ejecuta el siguiente comando para configurar la autoridad de certificación (CA) y generar los certificados del Puppet Master: 
sudo /opt/puppetlabs/bin/puppetserver ca setup 
  
# Ingrese a su Puppet master y haga una lista de los certificados que necesitan ser aprobados:
/opt/puppetlabs/bin/puppetserver ca list 
 
# Debería mostrar una lista con los nombres de host de los nodos del agente. 
# Aprobar los certificados: 
/opt/puppetlabs/bin/puppetserver ca sign - -certname  puppetagent.local 
 
# Vuelve a los nodos del agente Puppet y ejecuta el agente Puppet de nuevo: 
/opt/puppetlabs/bin/puppet agent –t

###############################################################################################################################
# En el caso de que no funciona la creación de certificación: 
# Asegúrate de que el nombre de host de tu nodo coincide con el certname especificado en su archivo de configuración: 
sudo nano /etc/puppetlabs/puppet/puppet.conf 
# El certname debe coincidir con el nombre que usaste al generar el certificado. 
# certname = puppetagent.local 
 
# Si realizas cambios, elimina los certificados locales del nodo para regenerarlos: 
sudo rm -rf /etc/puppetlabs/puppet/ssl/*

# Reinicia el agente Puppet para que solicite un nuevo certificado: 
sudo /opt/puppetlabs/bin/puppet agent –test 
################################################################################################################################
# En el caso del estado de puppet sale: /lib/systemd/system/puppetserver.service:45: standard 
# output type syslog is obsolete 
# Editar el archivo de unidad del servicio puppetserver.service: 
  
# El archivo de configuración del servicio de Puppet Server (puppetserver.service) está ubicado en “/lib/systemd/system/puppetserver.service”. Necesitarás editar este archivo para cambiar el tipo de salida. 
sudo nano  /lib/systemd/system/puppetserver.service 
 
# Busca la línea que contiene StandardOutput=syslog y lo cambiamos a StandardOutput=journal 
# Esto hace que los logs del servicio de Puppet Server se registren en el journal de systemd en lugar de syslog. 
# Recargar el daemon de systemd y reiniciar el servicio: 
  
# Una vez que hayas modificado el archivo de unidad, es necesario recargar los archivos de configuración de systemd y reiniciar el servicio de Puppet Server para aplicar los cambios: 
sudo systemctl daemon-reload 
sudo systemctl restart puppetserver
################################################################################################################################
