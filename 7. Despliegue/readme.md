# Guia de Despliegue en AWS EC2 con Docker Compose

**Nivel:** Principiante  
**Tiempo estimado:** 60 - 90 minutos  
**Objetivo:** Desplegar una aplicacion con tres contenedores (NGINX, FastAPI y PostgreSQL) en una instancia EC2 de Amazon Linux usando Docker Compose.

---

## Arquitectura de la aplicacion

La aplicacion que vamos a desplegar tiene tres componentes que corren como contenedores independientes:

- **NGINX**: Servidor web que sirve la pagina estatica y actua como proxy reverso.
- **FastAPI (Python)**: Backend de la aplicacion con la API REST.
- **PostgreSQL**: Base de datos relacional.

El archivo `docker-compose.yml` ya viene en el repositorio y define como se conectan estos tres servicios entre si.

---

## Requisitos previos

Antes de comenzar necesitas tener lo siguiente:

- Una cuenta activa en AWS.
- Acceso a la consola de AWS (AWS Management Console).
- Un par de claves SSH creado en la region donde vas a trabajar (o lo crearemos en el paso 1).
- El repositorio de la aplicacion disponible en GitHub o GitLab con el `docker-compose.yml` incluido.

---

## Paso 1: Crear la instancia EC2

### 1.1 Acceder al servicio EC2

1. Inicia sesion en [https://console.aws.amazon.com](https://console.aws.amazon.com).
2. En el buscador de servicios escribe **EC2** y seleccionalo.
3. En el panel izquierdo haz clic en **Instances** y luego en el boton **Launch instances**.

### 1.2 Configurar la instancia

Completa los siguientes campos:

**Name and tags**
- Dale un nombre descriptivo a tu instancia, por ejemplo: `mi-app-docker`.

**Application and OS Images (AMI)**
- Selecciona **Amazon Linux 2023 AMI** (o Amazon Linux 2, ambas son validas).
- Asegurate de que diga `Free tier eligible` si estas usando la capa gratuita.

**Instance type**
- Selecciona `t2.micro` o `t3.micro` (elegibles para la capa gratuita).

**Key pair (login)**
- Si ya tienes un par de claves, seleccionalo del desplegable.
- Si no tienes uno, haz clic en **Create new key pair**:
  - Nombre: `mi-clave-ec2`
  - Tipo: `RSA`
  - Formato: `.pem` (para Linux/Mac) o `.ppk` (para PuTTY en Windows)
  - Haz clic en **Create key pair** y guarda el archivo en un lugar seguro.

**Network settings**
- Deja la VPC y la subred por defecto.
- En **Firewall (security groups)** selecciona **Create security group**.
- Asegurate de que la regla **SSH (puerto 22)** este habilitada con source `My IP` (recomendado por seguridad).

**Configure storage**
- Deja el disco por defecto (8 GB es suficiente para esta practica).

### 1.3 Lanzar la instancia

- Revisa el resumen en el panel derecho.
- Haz clic en **Launch instance**.
- Espera entre 1 y 2 minutos hasta que el estado sea `Running` y la verificacion de estado muestre `2/2 checks passed`.

---

## Paso 2: Conectarse a la instancia via SSH

### En Linux o macOS

Abre una terminal y ejecuta:

```bash
chmod 400 /ruta/a/tu/mi-clave-ec2.pem

ssh -i /ruta/a/tu/mi-clave-ec2.pem ec2-user@<IP_PUBLICA_DE_TU_INSTANCIA>
```

Reemplaza `<IP_PUBLICA_DE_TU_INSTANCIA>` con la IP publica que aparece en la consola de EC2 (columna **Public IPv4 address**).

### En Windows (con PowerShell)

```powershell
ssh -i C:\ruta\a\mi-clave-ec2.pem ec2-user@<IP_PUBLICA_DE_TU_INSTANCIA>
```

Si es la primera vez que te conectas, el sistema te pedira confirmar la autenticidad del host. Escribe `yes` y presiona Enter.

Cuando veas el prompt `[ec2-user@ip-xxx ~]$` significa que la conexion fue exitosa.

---

## Paso 3: Instalar Docker y Docker Compose

Una vez dentro de la instancia, ejecuta los siguientes comandos uno por uno.

### 3.1 Actualizar los paquetes del sistema

```bash
sudo dnf update -y
```

> En Amazon Linux 2 usa `sudo yum update -y` en lugar de `dnf`.

### 3.2 Instalar Docker

```bash
sudo dnf install -y docker
```

### 3.3 Iniciar el servicio de Docker y habilitarlo para que arranque automaticamente

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### 3.4 Agregar el usuario actual al grupo docker

Esto permite ejecutar comandos de Docker sin necesidad de usar `sudo` cada vez.

```bash
sudo usermod -aG docker ec2-user
```

Despues de ejecutar este comando, cierra la sesion SSH y vuelve a conectarte para que el cambio de grupo tenga efecto:

```bash
exit
```

Vuelve a conectarte con el mismo comando SSH del paso 2. Luego verifica que Docker funciona correctamente:

```bash
docker --version
docker run hello-world
```

Deberias ver un mensaje que dice `Hello from Docker!`.

### 3.5 Instalar Docker Compose

Docker Compose V2 viene como plugin integrado en instalaciones modernas de Docker. Verifica si ya esta disponible:

```bash
docker compose version
```

Si el comando no existe, instalalo manualmente:

```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```

Verifica la instalacion:

```bash
docker compose version
```

---

## Paso 4: Instalar Git y clonar el repositorio

### 4.1 Instalar Git

```bash
sudo dnf install -y git
```

Verifica:

```bash
git --version
```

### 4.2 Clonar el repositorio

Reemplaza la URL con la de tu repositorio:

```bash
git clone https://github.com/tu-usuario/tu-repositorio.git
```

Ingresa al directorio del proyecto:

```bash
cd tu-repositorio
```

### 4.3 Verificar la estructura del proyecto

Asegurate de que el archivo `docker-compose.yml` este presente:

```bash
ls -la
cat docker-compose.yml
```

Un ejemplo de como deberia verse el `docker-compose.yml` de esta aplicacion:

```yaml
version: "3.9"

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: usuario
      POSTGRES_PASSWORD: contrasena
      POSTGRES_DB: mi_base_datos
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  api:
    build: ./api
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://usuario:contrasena@db:5432/mi_base_datos
    depends_on:
      - db

  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./web:/usr/share/nginx/html
    depends_on:
      - api

volumes:
  postgres_data:
```

---

## Paso 5: Levantar la aplicacion con Docker Compose

### 5.1 Construir y levantar los contenedores

```bash
docker compose up --build -d
```

- `--build`: reconstruye las imagenes si hay cambios en el codigo.
- `-d`: corre los contenedores en segundo plano (modo detached).

Este proceso puede tardar varios minutos la primera vez porque Docker descarga las imagenes base y construye la imagen de la API.

### 5.2 Verificar que los contenedores esten corriendo

```bash
docker compose ps
```

Deberias ver los tres servicios (`db`, `api`, `web`) con estado `Up` o `running`.

### 5.3 Ver los logs de los contenedores

Si algo no funciona, revisa los logs:

```bash
# Logs de todos los servicios
docker compose logs

# Logs de un servicio especifico
docker compose logs api
docker compose logs db
docker compose logs web

# Seguir los logs en tiempo real
docker compose logs -f api
```

---

## Paso 6: Probar la aplicacion con CURL

Desde dentro de la misma instancia EC2 puedes hacer pruebas locales antes de abrir el trafico al exterior.

### 6.1 Probar NGINX (pagina web)

```bash
curl http://localhost:80
```

Deberias recibir el HTML de la pagina principal.

### 6.2 Probar la API de FastAPI

```bash
# Endpoint raiz
curl http://localhost:8000/

# Documentacion automatica de FastAPI (responde con HTML)
curl http://localhost:8000/docs

# Ejemplo de endpoint GET (ajusta segun tu API)
curl http://localhost:8000/items

# Ejemplo de endpoint POST con datos JSON
curl -X POST http://localhost:8000/items \
  -H "Content-Type: application/json" \
  -d '{"nombre": "producto1", "precio": 100}'
```

### 6.3 Verificar la conexion con PostgreSQL

```bash
docker compose exec db psql -U usuario -d mi_base_datos -c "\dt"
```

Este comando lista las tablas de la base de datos. Si no hay tablas aun, el comando devuelve `Did not find any relations.`, lo cual es normal si la aplicacion aun no ha creado el esquema.

---

## Paso 7: Configurar las reglas de seguridad (Security Group)

Hasta este momento la aplicacion funciona internamente en la instancia, pero no es accesible desde internet. Para eso necesitas abrir los puertos en el Security Group de EC2.

### 7.1 Ir a la configuracion del Security Group

1. En la consola de EC2, selecciona tu instancia.
2. En la pestana **Security** (parte inferior de la pantalla), haz clic en el enlace del Security Group (algo como `sg-0abc123...`).
3. En la pantalla del Security Group, selecciona la pestana **Inbound rules**.
4. Haz clic en **Edit inbound rules**.

### 7.2 Agregar las reglas de entrada

Agrega las siguientes reglas haciendo clic en **Add rule** para cada una:

| Type | Protocol | Port range | Source | Descripcion |
|------|----------|------------|--------|-------------|
| HTTP | TCP | 80 | 0.0.0.0/0 | Trafico web (NGINX) |
| Custom TCP | TCP | 8000 | 0.0.0.0/0 | API FastAPI |
| Custom TCP | TCP | 5432 | 0.0.0.0/0 | PostgreSQL (solo si se necesita acceso externo) |

> **Nota de seguridad:** Abrir el puerto 5432 de PostgreSQL al publico general (`0.0.0.0/0`) no es recomendable en entornos de produccion. Para esta practica lo habilitamos para que puedan hacer pruebas, pero en un entorno real debes restringir ese acceso unicamente a las IPs que lo necesiten.

### 7.3 Guardar las reglas

Haz clic en **Save rules**.

---

## Paso 8: Probar desde el navegador y desde tu maquina local

Una vez configuradas las reglas, obtén la IP publica de tu instancia desde la consola de EC2 (columna **Public IPv4 address**).

### Desde el navegador

Abre estas URLs reemplazando `<IP_PUBLICA>` con la IP real:

```
http://<IP_PUBLICA>/           -> Pagina web servida por NGINX
http://<IP_PUBLICA>:8000/      -> Raiz de la API FastAPI
http://<IP_PUBLICA>:8000/docs  -> Documentacion interactiva de FastAPI
```

### Desde tu maquina local con CURL

```bash
# Pagina web
curl http://<IP_PUBLICA>/

# API FastAPI
curl http://<IP_PUBLICA>:8000/

# Endpoint con datos
curl http://<IP_PUBLICA>:8000/items
```

---

## Paso 9: Comandos utiles para la administracion

Una vez que la aplicacion esta corriendo, estos comandos te seran de utilidad:

```bash
# Ver el estado de los contenedores
docker compose ps

# Detener los contenedores (sin eliminarlos)
docker compose stop

# Volver a iniciar los contenedores detenidos
docker compose start

# Detener y eliminar los contenedores (los datos en volumen se conservan)
docker compose down

# Detener, eliminar contenedores Y eliminar los volumenes (se pierden los datos de la BD)
docker compose down -v

# Reconstruir e iniciar despues de cambios en el codigo
docker compose up --build -d

# Entrar al shell de un contenedor especifico
docker compose exec api bash
docker compose exec db bash

# Ver el uso de recursos de los contenedores
docker stats
```

---

## Solucion de problemas comunes

**El comando `docker` dice "permission denied"**  
Significa que el usuario no esta en el grupo `docker`. Ejecuta `sudo usermod -aG docker ec2-user`, cierra la sesion y vuelve a conectarte.

**Un contenedor aparece como `Exit` en `docker compose ps`**  
Revisa sus logs con `docker compose logs <nombre_servicio>` para ver el error especifico.

**La pagina no carga desde el navegador pero si desde dentro de la instancia**  
Verifica que las reglas del Security Group esten bien configuradas con los puertos correctos y source `0.0.0.0/0`.

**Error de conexion a la base de datos desde la API**  
Revisa la variable `DATABASE_URL` en el `docker-compose.yml`. El hostname debe ser el nombre del servicio definido en el compose (por ejemplo `db`), no `localhost`.

**No hay espacio en disco**  
Ejecuta `docker system prune -a` para eliminar imagenes, contenedores y redes que no esten en uso. Verifica el espacio con `df -h`.

---

## Resumen del flujo completo

```
1. Crear instancia EC2 (Amazon Linux)
       |
2. Conectarse via SSH
       |
3. Instalar Docker y Docker Compose
       |
4. Instalar Git y clonar el repositorio
       |
5. docker compose up --build -d
       |
6. Probar con CURL (localhost)
       |
7. Configurar reglas del Security Group (puertos 80, 8000)
       |
8. Probar desde el navegador y CURL externo
```

---

*Guia elaborada para estudiantes que inician en despliegue de aplicaciones con contenedores en la nube.*
