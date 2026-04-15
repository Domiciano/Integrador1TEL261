# Guía básica PostgreSQL con Docker


## Índice

1. [Instalación de base de datos](#1-instalación-de-base-de-datos)
2. [Entrar al contenedor](#2-entrar-al-contenedor)
3. [Lista de tablas](#3-lista-de-tablas)
4. [Creación de tablas](#4-creación-de-tablas)
5. [Insert Into](#5-insert-into)
6. [Pedir datos](#6-pedir-datos)
7. [Salir](#7-salir)


# 1. Instalación de base de datos

## Instalación de Postgres con Docker

Guarde el siguiente archivo como `docker-compose.yml`

``` bash
docker run -d --name db --platform linux/x86_64 -e POSTGRES_DB=db -e POSTGRES_USER=user -e POSTGRES_PASSWORD=password -p 5432:5432 -v my-db:/var/lib/postgresql/data postgres:17
```

Con esto tendrá una base de datos local en `127.0.0.1:5432`.\
Su usuario es `user`, su password es `password` y la base de datos se
llamará `db`.

También puede usar `docker-compose.yml`:

``` yml
services:
  db:
    platform: linux/x86_64
    image: postgres:17
    restart: always
    environment:
      POSTGRES_DB: 'db'
      POSTGRES_USER: 'user'
      POSTGRES_PASSWORD: 'password'
    ports:
      - '5432:5432'
    expose:
      - '5432'
    volumes:
      - my-db:/var/lib/postgresql/data

volumes:
  my-db:
```

Luego, use el comando:

``` sh
docker-compose up -d
```

# 2. Entrar al contenedor

En esta sección vamos a ingresar al contenedor para poder interactuar
directamente con la base de datos desde la consola.

Primero liste los contenedores activos:

``` bash
docker ps
```

Ubique el nombre o ID del contenedor y ejecute:

``` bash
docker exec -it db bash
```

Ahora entre a la consola de PostgreSQL:

``` bash
psql -U user -d db
```

Si todo sale bien, verá algo como:

    db=#

# 3. Lista de tablas

En esta sección vamos a listar las tablas existentes en la base de datos
actual para verificar qué estructuras están creadas.

Comando:

``` sql
\dt
```

También puede usar:

``` sql
SELECT tablename FROM pg_tables WHERE schemaname = 'public';
```


# 4. Creación de tablas

Ahora vamos a crear una tabla simple para almacenar estudiantes. Este
comando define la estructura de la tabla: columnas, tipos de datos y
clave primaria.

Comando:

``` sql
CREATE TABLE student (
    id SERIAL PRIMARY KEY,
    code VARCHAR(20) NOT NULL,
    name VARCHAR(100) NOT NULL,
    program VARCHAR(100)
);
```

Verifique que fue creada:

``` sql
\dt
```



# 5. Insert Into

En esta sección insertamos datos dentro de la tabla creada. Cada INSERT
agrega una nueva fila.

Comando:

``` sql
INSERT INTO student (code, name, program)
VALUES ('A00123456', 'Juan Perez', 'Ingenieria de Sistemas');
```

Insertar múltiples registros:

``` sql
INSERT INTO student (code, name, program)
VALUES 
('A00123457', 'Maria Gomez', 'Ingenieria Industrial'),
('A00123458', 'Carlos Ruiz', 'Administracion');
```


# 6. Pedir datos

Finalmente, consultamos los registros almacenados para verificar que los
datos fueron guardados correctamente.

Comando:

``` sql
SELECT * FROM student;
```


# 7. Salir

Salir de PostgreSQL:

``` sql
\q
```

Salir del contenedor:

``` bash
exit
```
