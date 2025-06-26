# 📦 Instalación y Configuración de Servidor Apache con Puppet

Este documento describe cómo automatizar la instalación del servidor Apache en un nodo usando Puppet.

---

## 📁 Paso 1: Crear el manifiesto principal

```bash
cd /etc/puppetlabs/code/environments/production/manifests/
nano site.pp
```

### Contenido de `site.pp`
```puppet
# Nodo específico
node 'puppetagent.local' {
  class { 'apache': }
}

# Nodo por defecto
node default {
  class { 'apache': }
}
```

---

## 📁 Paso 2: Crear el módulo Apache

```bash
cd /etc/puppetlabs/code/environments/production/modules
mkdir apache
cd apache
mkdir files manifests
cd manifests
nano init.pp
```

### Contenido de `init.pp`
```puppet
class apache {
  # Instalación del paquete
  package { 'apache2':
    ensure => installed,
  }

  # Habilitar y asegurar que esté corriendo
  service { 'apache2':
    ensure  => running,
    enable  => true,
    require => Package['apache2'],
  }

  # Archivo web personalizado
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
```

---

## ✅ Paso 3: Validar la sintaxis

```bash
puppet parser validate /etc/puppetlabs/code/environments/production/modules/apache/manifests/init.pp
puppet parser validate /etc/puppetlabs/code/environments/production/manifests/site.pp
```

---

## 🔄 Paso 4: Reiniciar el servicio Puppet

```bash
sudo systemctl restart puppetserver
```

---

## 🧪 Paso 5: Ejecutar en el nodo agente

```bash
sudo /opt/puppetlabs/bin/puppet agent -t
```

Verificar que Apache esté instalado:
```bash
dpkg -l | grep apache2
systemctl status apache2
```

Editar archivo:
```bash
sudo nano /etc/apache2/ports.conf
```
Modificar:
```text
Listen 0.0.0.0:80
# o
Listen 192.168.1.44:80
```

Probar en navegador:  
[http://192.168.1.44:80](http://192.168.1.44:80)

---

## 📦 Paso 6: Usar módulo de Puppet Forge

Instalar módulo:
```bash
puppet module install puppetlabs-apache --version 12.2.0
```

Verificar:
```bash
puppet module list
```
Debe aparecer:
```
/etc/puppetlabs/code/environments/production/modules
├─ puppetlabs-apache (v12.2.0)
```

---

## 🧩 Paso 7: Aplicar el módulo desde site.pp

```bash
cd /etc/puppetlabs/code/environments/production/manifests
nano site.pp
```

### Opcion 1: Simple
```puppet
node 'puppetagent.local' {
  include apache
}
```

### Opcion 2: Detallada
```puppet
node 'puppetagent.local' {
  class { 'apache':
    default_vhost => true,
    mpm_module    => 'prefork',
  }
}
```

---

## 🔁 Paso 8: Sincronizar en el agente

```bash
sudo /opt/puppetlabs/bin/puppet agent -t
```

Esto instalará Apache con la configuración gestionada por Puppet. ✨

