# Puppet + Apache en Ubuntu 22.04

Este repositorio contiene una guía paso a paso para instalar **Puppet** (Master y Agent) en Ubuntu 22.04 y un **manifiesto Puppet** para instalar y configurar el servidor web **Apache** en un nodo gestionado.

---

## 📁 Estructura del repositorio

```
.
├── docs/
│   └── instalacion_puppet.md   # Guía detallada para instalar Puppet
├── manifests/
│   └── apache.pp               # Manifiesto para instalar y configurar Apache
└── README.md                   # Este archivo
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

Lee la guía completa en [`docs/instalacion_puppet.md`](docs/instalacion_puppet.md), donde encontrarás:

- Instalación del Puppet Master y Agent
- Configuración de certificados
- Solución de errores comunes
- Configuración de puertos y servicios

---

## 📦 Manifiesto Apache

El archivo [`manifests/apache.pp`](manifests/apache.pp) contiene un manifiesto Puppet que:

- Instala Apache (`apache2`)
- Asegura que el servicio esté en ejecución y habilitado
- Crea una página web HTML básica
