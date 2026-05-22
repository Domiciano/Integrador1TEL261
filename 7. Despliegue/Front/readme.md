# Despliegue de una aplicacion Vite con Docker y Nginx

Esta guia explica como tomar una aplicacion creada con Vite, construirla para produccion y servirla con Nginx dentro de un contenedor Docker. Al final aprenderemos a integrarlo en un archivo Docker Compose.

---

## Requisitos previos

Antes de empezar, asegurate de tener instalado:

- Node.js (version 18 o superior)
- Docker Desktop (o Docker Engine en Linux)
- Una aplicacion Vite ya creada

Si aun no tienes una aplicacion Vite, puedes crear una de prueba con:

```bash
npm create vite@latest mi-app -- --template react
cd mi-app
npm install
```

---

## Paso 1: Construir la aplicacion para produccion

Vite incluye un comando de build que genera archivos estaticos optimizados listos para ser publicados. Estos archivos se guardan en una carpeta llamada `dist`.

Desde la raiz del proyecto ejecuta:

```bash
npm run build
```

Cuando termine, deberia aparecer una carpeta `dist/` con el contenido listo para produccion. Puedes verificarlo listando su contenido:

```bash
ls dist/
```

Deberias ver archivos como `index.html` y una subcarpeta `assets/` con los archivos de JavaScript y CSS compilados.

---

## Paso 2: Configurar Nginx

Nginx va a ser el servidor web que sirva los archivos estaticos del build. Necesitamos decirle a Nginx como comportarse: desde que carpeta servir los archivos y como manejar las rutas de la aplicacion.

Crea un archivo llamado `nginx.conf` en la raiz del proyecto con el siguiente contenido:

```nginx
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # Redirige todas las rutas al index.html
    # Esto es necesario para que el enrutamiento de React (o Vue) funcione correctamente
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Opcional: cabeceras de cache para archivos de assets
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Mensajes de error
    error_page 404 /index.html;
}
```

**Por que la linea `try_files $uri $uri/ /index.html`?**

Las aplicaciones modernas de una sola pagina (SPA) manejan el enrutamiento desde el navegador. Si el usuario visita directamente `/perfil` o `/productos`, Nginx buscaria un archivo llamado `perfil` en el disco y no lo encontraria. Con esta linea le decimos: "si no encuentras el archivo, devuelve el `index.html` y deja que la aplicacion se encargue del enrutamiento".

---

## Paso 3: Crear el Dockerfile

El `Dockerfile` es el archivo que le dice a Docker como construir la imagen de tu aplicacion. Crea un archivo llamado `Dockerfile` en la raiz del proyecto:

```dockerfile
# Etapa 1: construccion de la aplicacion
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Etapa 2: imagen final con Nginx
FROM nginx:alpine

# Copiar el resultado del build al directorio que sirve Nginx
COPY --from=build /app/dist /usr/share/nginx/html

# Reemplazar la configuracion por defecto de Nginx con la nuestra
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Que hace este Dockerfile paso a paso:**

1. Usa una imagen de Node para instalar dependencias y ejecutar `npm run build`.
2. Copia solo los archivos generados en `dist/` a la imagen de Nginx.
3. Reemplaza la configuracion de Nginx con nuestro archivo `nginx.conf`.
4. Expone el puerto 80 para recibir trafico HTTP.
5. Arranca Nginx en primer plano (requisito para que Docker lo mantenga corriendo).

La tecnica de usar dos etapas (`FROM ... AS build` y luego otro `FROM`) se llama **multi-stage build**. Sirve para que la imagen final no incluya Node.js ni el codigo fuente, solo los archivos necesarios para servir la aplicacion. Esto reduce el tamano de la imagen considerablemente.

---

## Paso 4: Construir la imagen Docker

Con el `Dockerfile` listo, construimos la imagen. Desde la raiz del proyecto ejecuta:

```bash
docker build -t mi-app-nginx .
```

**Explicacion del comando:**

- `docker build`: le dice a Docker que construya una imagen.
- `-t mi-app-nginx`: le da un nombre (tag) a la imagen. Puedes usar cualquier nombre.
- `.`: le indica a Docker que el `Dockerfile` esta en la carpeta actual.

Para verificar que la imagen se creo correctamente:

```bash
docker images
```

Deberias ver `mi-app-nginx` en la lista.

---

## Paso 5: Ejecutar el contenedor con docker run

Una vez creada la imagen, la ejecutamos como un contenedor:

```bash
docker run -d -p 8080:80 --name mi-app mi-app-nginx
```

**Explicacion del comando:**

- `-d`: ejecuta el contenedor en segundo plano (detached mode).
- `-p 8080:80`: mapea el puerto 8080 de tu maquina al puerto 80 del contenedor. Puedes cambiar el `8080` por cualquier puerto disponible en tu computadora.
- `--name mi-app`: le da un nombre al contenedor para poder referenciarlo facilmente.
- `mi-app-nginx`: el nombre de la imagen que queremos ejecutar.

Abre el navegador y visita `http://localhost:8080`. Deberia aparecer tu aplicacion.

**Comandos utiles para gestionar el contenedor:**

```bash
# Ver los contenedores que estan corriendo
docker ps

# Ver los logs del contenedor (util si algo no funciona)
docker logs mi-app

# Detener el contenedor
docker stop mi-app

# Eliminar el contenedor
docker rm mi-app
```

---

## Paso 6: Agregar el servicio a Docker Compose

Docker Compose permite definir y ejecutar multiples contenedores de forma declarativa usando un archivo YAML. Es util cuando la aplicacion necesita varios servicios (por ejemplo, un frontend, un backend y una base de datos).

Crea un archivo llamado `docker-compose.yml` en la raiz del proyecto:

```yaml
services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: mi-app
    ports:
      - "8080:80"
    restart: unless-stopped
```

**Explicacion de cada campo:**

- `services`: agrupa todos los servicios (contenedores) que va a manejar Compose.
- `frontend`: es el nombre del servicio. Puede ser cualquier nombre descriptivo.
- `build.context`: le dice a Compose donde encontrar el codigo fuente (`.` significa la carpeta actual).
- `build.dockerfile`: indica cual archivo usar para construir la imagen.
- `container_name`: nombre que tendra el contenedor al ejecutarse.
- `ports`: mapeo de puertos, igual que en `docker run -p`.
- `restart: unless-stopped`: el contenedor se reiniciara automaticamente si falla, a menos que lo hayas detenido manualmente.

### Ejecutar con Docker Compose

Para construir la imagen y arrancar el servicio:

```bash
docker compose up --build -d
```

Para detener y eliminar los contenedores:

```bash
docker compose down
```

Para ver los logs:

```bash
docker compose logs frontend
```

---

## Estructura final del proyecto

Al terminar, la raiz de tu proyecto deberia verse asi:

```
mi-app/
├── dist/               <- generado por npm run build
├── node_modules/
├── public/
├── src/
├── Dockerfile
├── docker-compose.yml
├── nginx.conf
├── package.json
└── index.html
```

---

## Resumen del flujo completo

```
Codigo fuente
    |
    v
npm run build  -->  carpeta dist/
    |
    v
docker build   -->  imagen Docker con Nginx
    |
    v
docker run     -->  contenedor corriendo en localhost:8080
    |
    v
docker compose -->  misma aplicacion gestionada con Compose
```

---

## Errores comunes

**La aplicacion carga pero las rutas dan error 404**

Verifica que el archivo `nginx.conf` tenga la linea `try_files $uri $uri/ /index.html;` dentro del bloque `location /`.

**El puerto 8080 ya esta en uso**

Cambia el puerto en el comando `docker run` o en el `docker-compose.yml`. Por ejemplo, usa `9090:80` para exponer en el puerto 9090.

**La imagen no se actualiza despues de cambiar el codigo**

Cuando cambias el codigo fuente, debes reconstruir la imagen. Con Docker Compose usa `docker compose up --build`. Con docker run, primero elimina la imagen anterior con `docker rmi mi-app-nginx` y luego vuelve a ejecutar `docker build`.

**El contenedor se detiene inmediatamente**

Revisa los logs con `docker logs mi-app`. Lo mas probable es que Nginx encontro un error en la configuracion. Verifica la sintaxis del archivo `nginx.conf`.
