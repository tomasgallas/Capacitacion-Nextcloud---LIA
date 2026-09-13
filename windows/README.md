# ☁️ Taller Práctico: Despliegue de Nube Local con Nextcloud y ONLYOFFICE en Windows

En esta práctica desplegaremos un servidor de almacenamiento en la nube privado y colaborativo dentro de la red local utilizando **Docker**, **Nextcloud**, **MariaDB** y **ONLYOFFICE Document Server**.

---

## 📋 Prerrequisitos
- Tener instalado **Docker Desktop** con motor **WSL2** activo y corriendo.
- Tener permisos de Administrador en la máquina para configurar las reglas de red en Windows.

---

## 🚀 Paso 1: Preparar la carpeta del proyecto

1. Abre una terminal de **PowerShell**.
2. Navega a tu Escritorio y crea la carpeta del taller:
```powershell
cd ~/Desktop
mkdir nube-local
cd nube-local
```

---

## 🔐 Paso 2: Crear el archivo de configuración `.env`

Crea un archivo llamado `.env` dentro de la carpeta `nube-local` con el siguiente contenido:

```env
# Contraseñas de la Base de Datos
MYSQL_ROOT_PASSWORD=RootPasswordSegura123
MYSQL_PASSWORD=NextcloudDbPass123
MYSQL_DATABASE=nextcloud
MYSQL_USER=nextcloud

# Usuario Administrador inicial de Nextcloud
NEXTCLOUD_ADMIN_USER=admin
NEXTCLOUD_ADMIN_PASSWORD=admin123
```

---

## 🐳 Paso 3: Crear el archivo `compose.yaml`

Crea un archivo llamado `compose.yaml` en la misma carpeta:

```yaml
services:
  db:
    image: mariadb:10.11
    container_name: nextcloud-mariadb
    restart: always
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
    networks:
      - nextcloud-net

  app:
    image: nextcloud:apache
    container_name: nextcloud-app
    restart: always
    ports:
      - "8080:80"
    volumes:
      - nextcloud_data:/var/www/html
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - NEXTCLOUD_ADMIN_USER=${NEXTCLOUD_ADMIN_USER}
      - NEXTCLOUD_ADMIN_PASSWORD=${NEXTCLOUD_ADMIN_PASSWORD}
    depends_on:
      - db
    networks:
      - nextcloud-net

  onlyoffice:
    image: onlyoffice/documentserver:latest
    container_name: nextcloud-onlyoffice
    restart: always
    ports:
      - "8081:80"
    environment:
      - JWT_SECRET=super_secreto_123
    extra_hosts:
      - "host.docker.internal:host-gateway"
    networks:
      - nextcloud-net

volumes:
  db_data:
  nextcloud_data:

networks:
  nextcloud-net:
    driver: bridge
```

---

## ⚡ Paso 4: Levantar los contenedores

En tu terminal de PowerShell, dentro de la carpeta `nube-local`, ejecuta:

```powershell
docker compose up -d
```

> ⏳ *La primera vez puede tardar unos 2 a 3 minutos mientras descarga las imágenes e inicializa la base de datos.*

Verifica que los tres contenedores estén en estado `running`:
```powershell
docker compose ps
```

---

## 🛡️ Paso 5: Configurar el Firewall de Windows para la Red Local

Para que las computadoras de tus compañeros puedan conectarse a tu servidor, necesitamos permitir la respuesta a `ping` y abrir los puertos `8080` (Nextcloud) y `8081` (ONLYOFFICE).

1. Abre **PowerShell como Administrador**.
2. Copia y ejecuta los siguientes 3 comandos:

```powershell
# Permitir responder a Ping (ICMPv4)
netsh advfirewall firewall add rule name="Permitir Ping IPv4" protocol=icmpv4:8,any dir=in action=allow

# Abrir el puerto 8080 para Nextcloud
New-NetFirewallRule -DisplayName "Nextcloud Server 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow

# Abrir el puerto 8081 para ONLYOFFICE
New-NetFirewallRule -DisplayName "OnlyOffice Server 8081" -Direction Inbound -LocalPort 8081 -Protocol TCP -Action Allow
```

---

## 🌐 Paso 6: Obtener tu IP local y autorizarla en Nextcloud

### 1. Conocer tu IP local:
En la terminal, ejecuta:
```powershell
ipconfig
```
Busca tu adaptador activo (Wi-Fi o Ethernet) y anota la **Dirección IPv4** (ejemplo: `192.168.1.50`).

### 2. Autorizar tu IP y resolver la comunicación interna en Nextcloud:
Ejecuta los siguientes comandos (reemplazando `TU_IP_AQUI` por tu dirección IP real, por ejemplo `192.168.1.50`):

```powershell
# Dominios y nombres de contenedor de confianza
docker compose exec --user www-data app php occ config:system:set trusted_domains 1 --value="nextcloud-app"
docker compose exec --user www-data app php occ config:system:set trusted_domains 2 --value="app"
docker compose exec --user www-data app php occ config:system:set trusted_domains 3 --value="localhost:8080"
docker compose exec --user www-data app php occ config:system:set trusted_domains 4 --value="TU_IP_AQUI:8080"
docker compose exec --user www-data app php occ config:system:set trusted_domains 5 --value="TU_IP_AQUI"

# Permitir comunicación entre contenedores en red local
docker compose exec --user www-data app php occ config:system:set allow_local_remote_servers --value=true --type=boolean
```

---

## 📝 Paso 7: Configurar la App de ONLYOFFICE en Nextcloud

1. Abre tu navegador y entra a: `http://localhost:8080`
2. Inicia sesión con el usuario `admin` y la contraseña `admin123`.
3. Ve al ícono de usuario arriba a la derecha > **Aplicaciones**.
4. En el buscador escribe **ONLYOFFICE** y haz clic en **Descargar y activar**.
5. Ve a **Ajustes de administración** (menú superior derecho) > sección **ONLYOFFICE** (panel izquierdo).
6. Configura los siguientes campos:
   * **Dirección de ONLYOFFICE Docs:** `http://TU_IP_AQUI:8081/` *(Reemplaza por tu IP real)*
   * **Clave secreta:** `super_secreto_123`
   * Despliega **Ajustes de servidor avanzados**:
     * **Dirección de ONLYOFFICE Docs para solicitudes internas:** `http://nextcloud-onlyoffice/`
     * **Dirección de servidor para solicitudes internas de ONLYOFFICE:** `http://nextcloud-app/`
7. Haz clic en **Guardar**. Debería confirmarse con éxito y mostrar los formatos compatibles.

---

## 👥 Paso 8: ¡Prueba colaborativa con tu compañero!

1. Pídele al compañero que está al lado tuyo que abra el navegador en su PC.
2. Pídele que ingrese a tu servidor: `http://TU_IP_AQUI:8080`
3. En tu Nextcloud, entra a **Usuarios** y créale una cuenta a tu compañero.
4. Tu compañero inicia sesión, crea un archivo de texto `.docx` y compártelo contigo.
5. ¡Ambos abran el documento al mismo tiempo y verán la **edición colaborativa en tiempo real**!

---

## 🛑 Comandos útiles para pausar y reanudar el servidor

* **Pausar el servidor al terminar el día:**
  ```powershell
  docker compose stop
  ```
* **Volver a iniciar el servidor:**
  ```powershell
  docker compose up -d
  ```
* **Ver los registros/logs en vivo ante cualquier duda:**
  ```powershell
  docker compose logs -f
  ```
