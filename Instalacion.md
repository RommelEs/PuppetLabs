# 🐾 Guía de Instalación de Puppet en Ubuntu 22.04

Esta guía cubre la instalación y configuración básica de Puppet Master y Puppet Agent en Ubuntu 22.04.

---

## 📋 Índice

1. [Requisitos previos](#requisitos-previos)  
2. [🏷️ Configuración de nombres de host](#configuración-de-nombres-de-host)  
3. [Instalación del repositorio de Puppet](#instalación-del-repositorio-de-puppet)  
4. [Instalación de Puppet](#instalación-de-puppet)  
5. [Configuración del Puppet Master](#configuración-del-puppet-master)  
6. [Configuración del Puppet Agent](#configuración-del-puppet-agent)  
7. [Firma de certificados](#firma-de-certificados)  
8. [Solución de errores comunes](#solución-de-errores-comunes)  
9. [Validación final](#validación-final)

---

## ✅ Requisitos previos

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y wget
sudo -i
```

## 🏷️ Configuración de nombres de host

### En el Puppet Master:

```bash
sudo hostnamectl set-hostname puppetmaster.local
echo "127.0.0.1 puppetmaster.local puppet" | sudo tee -a /etc/hosts
```

### En el Puppet Agent:

```bash
sudo hostnamectl set-hostname puppetagent.local
echo "127.0.0.1 puppetagent.local agent" | sudo tee -a /etc/hosts
```

## 📦 Instalación del repositorio de Puppet

```bash
wget https://apt.puppetlabs.com/puppet-release-jammy.deb
sudo dpkg -i puppet-release-jammy.deb
sudo apt update
```

## 🛠️ Instalación de Puppet

### En el Puppet Master:

```bash
sudo apt install -y puppetserver
```

### En el Puppet Agent:

```bash
sudo apt install -y puppet-agent
```

## ⚙️ Configuración del Puppet Master

```bash
/opt/puppetlabs/bin/puppet config set dns_alt_names 'puppet,puppetmaster.local' --section main
cat /etc/puppetlabs/puppet/puppet.conf
```

Edita el archivo `/etc/hosts` y añade:

```text
192.168.1.42   puppetmaster.local puppet
192.168.1.44   puppetagent.local agent
```

Edita la memoria para Java en `/etc/default/puppetserver`:

```bash
JAVA_ARGS="-Xms2g -Xmx2g -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"
```

Inicia y habilita el servicio:

```bash
sudo systemctl start puppetserver
sudo systemctl enable puppetserver
sudo systemctl status puppetserver
```

Habilita el puerto 8140:

```bash
sudo ufw allow 8140/tcp
sudo ufw enable
sudo ufw status
```

## 🤖 Configuración del Puppet Agent

Edita `/etc/hosts`:

```text
192.168.1.42    puppetmaster.local puppet
```

Configura el FQDN del master:

```bash
/opt/puppetlabs/bin/puppet config set server 'puppetmaster.local' --section main
cat /etc/puppetlabs/puppet/puppet.conf
```

Inicia y habilita el servicio:

```bash
/opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true
# o alternativamente:
sudo systemctl start puppet
sudo systemctl enable puppet
```

## 🔐 Firma de certificados

En el agente:

```bash
/opt/puppetlabs/bin/puppet agent -t
```

En el master:

```bash
sudo /opt/puppetlabs/bin/puppetserver ca setup
/opt/puppetlabs/bin/puppetserver ca list
/opt/puppetlabs/bin/puppetserver ca sign --certname puppetagent.local
```

Vuelve al agente y ejecuta:

```bash
/opt/puppetlabs/bin/puppet agent -t
```

## 🧩 Solución de errores comunes

### Certificado no coincide

Verifica `certname`:

```bash
sudo nano /etc/puppetlabs/puppet/puppet.conf
# certname = puppetagent.local
```

Si haces cambios:

```bash
sudo rm -rf /etc/puppetlabs/puppet/ssl/*
sudo /opt/puppetlabs/bin/puppet agent --test
```

### Error de `syslog` obsoleto:

Edita `/lib/systemd/system/puppetserver.service` y reemplaza:

```
StandardOutput=syslog
```

por:

```
StandardOutput=journal
```

Luego recarga daemon:

```bash
sudo systemctl daemon-reload
sudo systemctl restart puppetserver
```

## ✅ Validación final

Verifica que el servicio funcione en ambos nodos:

```bash
sudo systemctl status puppetserver
sudo systemctl status puppet
```
