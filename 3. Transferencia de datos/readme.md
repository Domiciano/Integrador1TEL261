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



