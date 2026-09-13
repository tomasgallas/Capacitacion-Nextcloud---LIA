# ☁️ Taller Práctico: Despliegue de Nube Privada y Colaborativa (Nextcloud + ONLYOFFICE)

¡Bienvenido al repositorio de la capacitación en Nextcloud!

Este proyecto tiene como objetivo enseñar cómo desplegar una nube privada de almacenamiento y suite ofimática colaborativa en tiempo real dentro de una red de área local (LAN), utilizando contenedores **Docker**, **Nextcloud**, **MariaDB** y **ONLYOFFICE Document Server**.

---

## 🧭 Selecciona tu Sistema Operativo

Elige la guía según el sistema operativo de tu computadora para comenzar la práctica paso a paso:

| Sistema Operativo | Guía de Instalación y Configuración | Archivos |
| :--- | :--- | :--- |
| 🪟 **Windows** | [📖 Ver Guía para Windows](windows/README.md) | [`windows/`](windows/) |
| 🐧 **Ubuntu / Debian** | [📖 Ver Guía para Linux](ubuntu/README.md) | [`ubuntu/`](ubuntu/) |

---

## 🏗️ Arquitectura del Sistema

El taller orquesta **3 contenedores interconectados** sobre una red interna tipo *bridge*:

```text
+-------------------------------------------------------------+
|                        Red Local (LAN)                      |
|                                                             |
|   [ Navegador de Compañeros]-------> http://<TU_IP>:8080    |
|   [ Navegador Host ]      --------> http://localhost:8080   |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                      DOCKER ENGINE                          |
|                                                             |
|   +-------------------+          +----------------------+   |
|   |   nextcloud-app   | <======> |  nextcloud-mariadb   |   |
|   |    (Nextcloud)    | (MySQL)  |      (Base Datos)    |   |
|   +-------------------+          +----------------------+   |
|             ^                                               |
|             | (HTTP Interno / JWT)                          |
|             v                                               |
|   +-----------------------+                                 |
|   |  nextcloud-onlyoffice |                                 |
|   |   (Document Server)   |                                 |
|   +-----------------------+                                 |
|                                                             |
|   Red Docker Interna: nextcloud-net                         |
+-------------------------------------------------------------+
```

---

## 🎯 Objetivos de Aprendizaje

1. **Virtualización Ligera y Orquestación:** Uso de `Docker Compose` para declarar infraestructura como código (IaC).
2. **Persistencia de Datos:** Manejo de volúmenes para base de datos y archivos de usuario.
3. **Networking y Seguridad:** 
   - Comprensión de direccionamiento IP local (DHCP vs IP estática).
   - Configuración de reglas de firewall (ICMP / puertos TCP).
   - Mitigación de ataques de *Host Header Injection* mediante `trusted_domains`.
4. **Colaboración en Tiempo Real:** Integración de editores de documentos mediante comunicación inter-contenedor y tokens seguros JWT.

---

## 👥 Autores y Capacitación
- **Laboratorio de Informatica Aplicada.**
-   Tomas Gallastegui
-   Juan José Caputo
-   Martin Flores