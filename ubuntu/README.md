# ☁️ Taller Práctico: Despliegue de Nube Local con Nextcloud y ONLYOFFICE en Ubuntu / Debian

En esta práctica desplegaremos un servidor de almacenamiento en la nube privado y colaborativo dentro de la red local utilizando **Docker**, **Nextcloud**, **MariaDB** y **ONLYOFFICE Document Server** en Linux.

---

## 📋 Prerrequisitos e Instalación de Docker

Si aún no tienes Docker instalado en tu sistema Ubuntu/Debian, ejecuta en la terminal:

```bash
# 1. Actualizar repositorios e instalar Docker + Docker Compose Plugin
sudo apt update
sudo apt install -y docker.io docker-compose-v2

# 2. Agregar tu usuario al grupo docker (para no necesitar 'sudo' en cada comando)
sudo usermod -aG docker $USER

# 3. Aplicar el cambio de grupo a la sesión actual
newgrp docker
```

---

## 🚀 Paso 1: Preparar la carpeta del proyecto

1. Abre tu terminal.
2. Crea y entra a la carpeta del taller en tu Escritorio:
```bash
cd ~/Desktop
mkdir nube-local
cd nube-local
```

---

## 🔐 Paso 2: Crear el archivo de configuración `.env`

Crea el archivo `.env` (puedes usar `nano .env`):

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

Crea el archivo `compose.yaml` (usando `nano compose.yaml`):

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

En la terminal, dentro de la carpeta `nube-local`, ejecuta:

```bash
docker compose up -d
```

> ⏳ *La primera vez puede tardar unos 2 a 3 minutos mientras descarga las imágenes e inicializa la base de datos.*

Verifica que los tres contenedores estén en estado `running`:
```bash
docker compose ps
```

---

## 🛡️ Paso 5: Configurar el Firewall en Ubuntu (UFW)

Si tienes el firewall de Ubuntu (`ufw`) activo, abre los puertos `8080` y `8081`:

```bash
sudo ufw allow 8080/tcp
sudo ufw allow 8081/tcp
sudo ufw reload
```

---

## 🌐 Paso 6: Obtener tu IP local y autorizarla en Nextcloud

### 1. Conocer tu IP local en Linux:
Ejecuta:
```bash
hostname -I
```
*(O también `ip -br a`). Toma la primera dirección IP que empiece con `192.168.x.x` o `10.x.x.x` (ejemplo: `192.168.1.50`).*

### 2. Autorizar tu IP y resolver la comunicación interna en Nextcloud:
Ejecuta los siguientes comandos (reemplazando `TU_IP_AQUI` por tu dirección IP real, por ejemplo `192.168.1.50`):

```bash
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

1. Pídele a tu compañero de laboratorio que abra su navegador.
2. Que ingrese a tu servidor: `http://TU_IP_AQUI:8080`
3. En tu Nextcloud, entra a **Usuarios** y créale una cuenta a tu compañero.
4. Tu compañero inicia sesión, crea un archivo `.docx` y compártelo contigo.
5. ¡Ambos editen el documento en tiempo real desde sus respectivas computadoras!

---

## 🛑 Comandos útiles para pausar y reanudar el servidor

* **Pausar el servidor al terminar el día:**
  ```bash
  docker compose stop
  ```
* **Volver a iniciar el servidor:**
  ```bash
  docker compose up -d
  ```
* **Ver los registros/logs en vivo ante cualquier duda:**
  ```bash
  docker compose logs -f
  ```
