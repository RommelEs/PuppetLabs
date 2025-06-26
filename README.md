# Puppet + Apache en Ubuntu 22.04

Este repositorio contiene una guía paso a paso para instalar **Puppet** (Master y Agent) en Ubuntu 22.04 y un **manifiesto Puppet** para instalar y configurar el servidor web **Apache** en un nodo gestionado.

---

## 📁 Estructura del repositorio

```
.
├── Instalacion.md                   # Guía detallada para instalar Puppet
├── manifiesto_servidor_apache.md   # Explicación del manifiesto para Apache
├── apache.pp                        # Manifiesto Puppet para Apache
└── README.md                        # Este archivo
```

---

## 🚀 Objetivo

Automatizar la instalación y configuración de un servidor Apache en un sistema gestionado mediante Puppet, partiendo desde la instalación completa del entorno Puppet Master-Agent.

---

## 🛠️ Requisitos

- Ubuntu 22.04 (en Puppet Master y Agent)
- Conexión entre Master y Agente
- Acceso a terminal con privilegios de superusuario

---

## 📘 Documentación

Lee la guía completa en [`Instalacion.md`](Instalacion.md), donde encontrarás:

- Instalación del Puppet Master y Agent
- Configuración de certificados
- Solución de errores comunes
- Configuración de puertos y servicios

---

## 📦 Manifiesto Apache

Lee la explicación del manifiesto en [`manifiesto_servidor_apache.md`](manifiesto_servidor_apache.md).

El archivo `apache.pp` contiene un manifiesto Puppet que:

- Instala Apache (`apache2`)
- Asegura que el servicio esté en ejecución y habilitado
- Crea una página web HTML básica

---

## ▶️ Aplicación rápida

```bash
sudo /opt/puppetlabs/bin/puppet apply apache.pp
```
