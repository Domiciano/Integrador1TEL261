# 1. Instalación de base de datos

## Instalación de Postgres con Docker

Guarde el siguiente archivo como docker-compose.yml

```
docker run -d \
  --name db \
  --platform linux/x86_64 \
  -e POSTGRES_DB=db \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 \
  -v my-db:/var/lib/postgresql/data \
  postgres:17
```
Con esto tendrá una base de datos local en 127.0.0.1:5432. Su usuario es user, su password es password y la base de datos se llamará db

También puede usar
```yml
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

Luego, use el comando
```sh
docker-compose up -d
```
