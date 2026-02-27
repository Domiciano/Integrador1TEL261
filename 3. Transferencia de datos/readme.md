# Transferencia de datos
Debemos aprender los mecanismos que nos permiten obtener datos desde los clientes, de modo que aplicaciones ejecutadas por ejemplo desde un ESP32 puedan llegar a una base de datos en la nube o en un servicio local.

## Modelo de datos

```python
from pydantic import BaseModel

class Reading(BaseModel):
    id: int
    deviceId: str
    value: int
    unit: str
```
Los tipos de datos básicos son `int`, `float`, `str`, `bool`, `bytes`.

De aquí sale la pregunta clave que es `¿Qué sensores vamos a usar?`

Ya con el tipo de datos definido se puede recibir a partir de un `POST` Request.

## HTTP POST
El servicio para recibir una sola reading se puede hacer con el siguiente bloque
```python
from fastapi import FastAPI

app = FastAPI()

@app.post("/readings")
def register_reading(reading: Reading):
    return {
        "message": "Medida recibida",
        "data": reading
    }
```
Aquí deberá enviar la reading en el `body` del mensaje http con verbo `POST`. Por ejemplo

```json
{
    "id": 2,
    "deviceId": "dev-01",
    "value": 26,
    "unit": "C"
}
```


Si quiere recibir una lista de readings, lo puede hacer así

```python
@app.post("/readings/batch")
def register_readings(readings: list[Reading]):
    return {
        "message": "Medidas recibidas",
        "total": len(readings),
        "data": readings
    }
```

El `json` en el `body` debería ser así

```json
[
  {
    "id": 1,
    "deviceId": "dev-01",
    "value": 25,
    "unit": "C"
  },
  {
    "id": 2,
    "deviceId": "dev-01",
    "value": 26,
    "unit": "C"
  }
]
```

## Persistir la información
De momento tenemos los datos siendo recibidos, pero debemos asegurar de que los podamos almacenar, para eso, inicialmente instalemos las piezas necesarias.

```bash
pip install fastapi uvicorn sqlalchemy psycopg2-binary
```
Vamos a conectarnos con la base de datos, para eso, creemos una base de datos en https://console.neon.tech/

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import declarative_base

DATABASE_URL = "postgresql://user:password@localhost/dbname"

engine = create_engine(DATABASE_URL)
Base = declarative_base()
```
Posteriormente, creemos el modelo de readings para que se construya una tabla de base de datos.

```python
from sqlalchemy import Column, Integer, String

class Student(Base):
    __tablename__ = "students"
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String)
    age = Column(Integer)
```
Puede usar otros tipos de datos como `Float`, `BigInteger`, `DateTime`

Finalmente pongamos un trigger, que cuando la app de fastapi arranque, entonces se generen las tablas

```python
@app.on_event("startup")
def on_startup():
    Base.metadata.create_all(bind=engine)
```

## Almacenar los datos

Para almacenar finalmente debemos tener una sesión con la base de datos

```python
from sqlalchemy.orm import sessionmaker
...
SessionLocal = sessionmaker(bind=engine)
```
Luego debemos hacer el almacenamiento por medio de la sesión en el `POST`

```python
@app.post("/readings")
def register_reading(reading: Reading):
    db = SessionLocal()

    # Hacemos el mapeo
    db_reading = ReadingDB(
        deviceId=reading.deviceId,
        value=reading.value,
        unit=reading.unit
    )

    # Agregamos a la base de datos
    db.add(db_reading)
    db.commit()
    db.refresh(db_reading)

    # Cerramos la transacción
    db.close()

    # Escribimos la respuesta
    return {
        "message": "Medida guardada",
        "data": db_reading.id
    }
```

## Almacenar un batch

```python
@app.post("/readings/batch")
def register_readings_batch(readings: list[Reading]):
    db = SessionLocal()

    # Mapear lista
    db_readings = [
        ReadingDB(
            deviceId=r.deviceId,
            value=r.value,
            unit=r.unit
        )
        for r in readings
    ]

    # Insertar en bloque
    db.add_all(db_readings)
    db.commit()

    # Obtener IDs generados
    for r in db_readings:
        db.refresh(r)

    db.close()

    return {
        "message": f"{len(db_readings)} medidas guardadas",
        "data": [r.id for r in db_readings]
    }
```
--
# Notas sobre listas en python
Las listas (list) son una de las estructuras de datos más importantes en Python. Permiten almacenar múltiples elementos en orden, y son especialmente útiles cuando se reciben múltiples mediciones (batch) desde dispositivos como un ESP32.

Crear una lista
```
numbers = [10, 20, 30, 40]
devices = ["dev-01", "dev-02", "dev-03"]
```
También puede crear una lista vacía:
```
readings = []
```
Acceder a elementos

Cada elemento tiene una posición (índice), empezando desde 0.

```
numbers = [10, 20, 30]

print(numbers[0])  # 10
print(numbers[1])  # 20
print(numbers[2])  # 30
print(numbers[-1])  # 40
```
Agregar elementos

Puede agregar elementos usando append():
```
readings = []

readings.append(25)
readings.append(26)
readings.append(27)

print(readings)
# [25, 26, 27]
```
Recorrer una lista

Esto es extremadamente importante cuando trabajamos con batch:
```
readings = [25, 26, 27]

for r in readings:
    print(r)
```
Salida
```
25
26
27
```
Recorrer lista de objetos

Esto es exactamente lo que hacemos en FastAPI con batch:
```
readings = [
    {"deviceId": "dev-01", "value": 25},
    {"deviceId": "dev-01", "value": 26}
]

for r in readings:
    print(r["deviceId"], r["value"])
Obtener el tamaño de una lista
readings = [25, 26, 27]

print(len(readings))  # 3
```
Esto es útil en FastAPI:
```
return {
    "total": len(readings)
}
```
List comprehension

Es una forma compacta de construir listas:
```
numbers = [1, 2, 3, 4]

squared = [n*n for n in numbers]

print(squared)
# [1, 4, 9, 16]
```
Este mismo patrón se usa en SQLAlchemy:
```
db_readings = [
    ReadingDB(
        deviceId=r.deviceId,
        value=r.value,
        unit=r.unit
    )
    for r in readings
]
```
