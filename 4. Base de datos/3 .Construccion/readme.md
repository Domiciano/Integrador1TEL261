# Relaciones entre tablas con SQLAlchemy ORM

## Contexto: el proyecto base

Partimos de una tabla `readings` que almacena lecturas de sensores:

```python
class ReadingTable(Base):
    __tablename__ = "readings"
    id        = Column(Integer, primary_key=True, index=True)
    value     = Column(Integer)
    timestamp = Column(Integer)
    deviceName = Column(String)
    units     = Column(String)
```

El problema: `deviceName` se repite en cada fila. Si un dispositivo cambia de nombre, habría que actualizar cientos de registros. La solución es separar los dispositivos en su propia tabla y relacionarlas.

---

## Relación 1 a muchos (One-to-Many)

> **Un dispositivo → muchas lecturas**

Un dispositivo puede tener muchas lecturas, pero cada lectura pertenece a un solo dispositivo. Este es el caso de uso más común.

### Cómo se define en SQLAlchemy

```python
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

class Device(Base):
    __tablename__ = "devices"

    id   = Column(Integer, primary_key=True, index=True)
    name = Column(String, unique=True, nullable=False)

    # Lado "uno": referencia a la lista de lecturas
    readings = relationship("ReadingTable", back_populates="device")


class ReadingTable(Base):
    __tablename__ = "readings"

    id        = Column(Integer, primary_key=True, index=True)
    value     = Column(Integer)
    timestamp = Column(Integer)
    units     = Column(String)

    # Lado "muchos": clave foránea que apunta al dispositivo
    device_id = Column(Integer, ForeignKey("devices.id"), nullable=False)
    device    = relationship("Device", back_populates="readings")
```

### Puntos clave

| Elemento | Dónde va | Para qué sirve |
|---|---|---|
| `ForeignKey("devices.id")` | Tabla hija (`readings`) | Crea el vínculo en la base de datos |
| `relationship(...)` | Ambas tablas | Permite navegar entre objetos en Python |
| `back_populates` | Ambos lados | Mantiene sincronizados los dos extremos de la relación |

---

## Insertar datos

### 1. Crear un dispositivo nuevo con sus lecturas

```python
db = SessionLocal()

# Crear el dispositivo
nuevo_device = Device(name="Temp01")

# Crear lecturas y asignarlas directamente al dispositivo
lectura1 = ReadingTable(value=514, timestamp=1700000000, units="celsius")
lectura2 = ReadingTable(value=520, timestamp=1700000060, units="celsius")

# Asignar a través del relationship (sin tocar device_id manualmente)
nuevo_device.readings = [lectura1, lectura2]

db.add(nuevo_device)   # SQLAlchemy agrega las lecturas en cascada
db.commit()
db.close()
```

### 2. Agregar una lectura a un dispositivo existente

```python
db = SessionLocal()

# Buscar el dispositivo por nombre
device = db.query(Device).filter(Device.name == "Temp01").first()

# Crear la nueva lectura con el device_id explícito
nueva_lectura = ReadingTable(
    value=530,
    timestamp=1700000120,
    units="celsius",
    device_id=device.id      # ← asignamos la FK directamente
)

db.add(nueva_lectura)
db.commit()
db.close()
```

---

## Leer datos

### Obtener todas las lecturas de un dispositivo

```python
db = SessionLocal()

device = db.query(Device).filter(Device.name == "Temp01").first()

# El relationship carga las lecturas automáticamente (lazy loading por defecto)
for lectura in device.readings:
    print(lectura.value, lectura.units)

db.close()
```

### Obtener la lectura con su dispositivo (JOIN implícito)

```python
db = SessionLocal()

# SQLAlchemy hace el JOIN por nosotros
lecturas = db.query(ReadingTable).join(Device).filter(Device.name == "Temp01").all()

for l in lecturas:
    print(f"Dispositivo: {l.device.name} | Valor: {l.value} {l.units}")

db.close()
```

### Obtener todos los dispositivos con su cantidad de lecturas

```python
from sqlalchemy import func

db = SessionLocal()

resultados = (
    db.query(Device.name, func.count(ReadingTable.id).label("total"))
    .join(ReadingTable)
    .group_by(Device.name)
    .all()
)

for nombre, total in resultados:
    print(f"{nombre}: {total} lecturas")

db.close()
```
