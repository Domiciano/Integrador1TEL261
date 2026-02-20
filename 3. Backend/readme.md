# Servidor web
Un servidor web es un programa que escucha solicitudes de los navegadores a través de la red (normalmente usando el protocolo HTTP sobre TCP) y responde enviando recursos como páginas HTML, imágenes o datos. En otras palabras, es el intermediario que recibe una petición como “quiero ver esta página” y devuelve el contenido necesario para que el navegador lo muestre al usuario.

Por ejemplo, si quiere habilitar un servidor web en la carpeta donde esta parado use
```bash
python -m http.server 8080
```
Puede también generar el servidor con parámetros crear un archivo `server.py` y ejecutarlo
```python
from http.server import HTTPServer, SimpleHTTPRequestHandler

HOST = "0.0.0.0"
PORT = 8000

server = HTTPServer((HOST, PORT), SimpleHTTPRequestHandler)

print(f"Servidor corriendo en http://localhost:{PORT}")
server.serve_forever()
```

En este caso, este servidor ofrecerá los archivos que estén contenidos en el folder donde está ejecutándose


# HTTP 

En el <a href="https://datatracker.ietf.org/doc/html/rfc1945">RCF 1945</a> está el esquema de request y response de un servidor HTTP. 

<img src="http://i2thub.icesi.edu.co/compu2/assets/image2-B7B2EaGa.png">

### Request
```
GET /index.html HTTP/1.0\r\n
Host: www.ejemplo.com\r\n
User-Agent: Mozilla/5.0\r\n
Accept: text/html\r\n
Connection: close\r\n
\r\n
Body del mensaje usualmente JSON
```

### Response 
```
HTTP/1.0 200 OK\r\n
Content-Type: text/html\r\n
Content-Length: 34\r\n
Connection: close\r\n
\r\n
<html><body>Hola Mundo</body></html>
```

# Servidor de Aplicaciones

Creemos un virtual enviroment de python. Un `venv` (virtual environment) en Python es un entorno virtual aislado que permite instalar y gestionar dependencias (librerías y versiones de paquetes) de manera independiente para cada proyecto. Esto evita conflictos entre proyectos que necesitan versiones distintas de las mismas librerías y mantiene el entorno global de Python limpio. Básicamente, es como crear una “mini instalación” de Python específica para un proyecto.
```
python -m venv venv
```
Eso crea la carpeta `venv`

Para activar el environment en Linux/Mac OS
```
source venv/bin/activate
```
En Windows CMD
```
venv\Scripts\activate
```
Y en windows (powershell)
```
venv\Scripts\Activate.ps1
```
Si da problemas de permisos, primero
```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
Para desactivar el `venv` use
```
deactivate
```
Instalemos las dependencias necesarioas
```
pip install fastapi uvicorn python-multipart
```

Luego vamos a crear `server.py`
```python
from fastapi import FastAPI, Form
from fastapi.responses import HTMLResponse, FileResponse
from datetime import datetime

app = FastAPI()

# GET que devuelve la hora del sistema
@app.get("/hora")
def obtener_hora():
    return {"hora": datetime.now().strftime("%H:%M:%S")}


# GET que suma dos números
@app.get("/sumar")
def sumar(a: int, b: int):
    return {"resultado": a + b}


# GET que devuelve un archivo HTML de la raíz
@app.get("/", response_class=HTMLResponse)
def mostrar_html():
    return FileResponse("index.html")

```

Para ejecutarlo
```
uvicorn main:app --reload
```

# Parámetros en una URL
Cuando un cliente hace una petición HTTP, puede enviar información adicional en la URL.
Existen dos formas comunes de hacerlo: Query Parameters, Path Parameters

## Query Parameters
Son parámetros que se envían después del signo ? en la URL y se separan con &.

Ejemplo de URL
```
http://localhost:8000/sumar?a=5&b=3
```
En FastAPI se definen como parámetros normales en la función:

```python
@app.get("/sumar")
def sumar(a: int, b: int):
    return {"resultado": a + b}
```
En este caso a y b son query parameters. FastAPI los convierte automáticamente al tipo indicado (int). Si falta uno, FastAPI genera un error automáticamente

# Path Parameters
Son parámetros que hacen parte de la ruta misma.

Ejemplo de URL
```
http://localhost:8000/sensor/10
```
En FastAPI se definen usando llaves `{}` en la ruta
```python
@app.get("/sensor/{id}")
def obtener_sensor(id: int):
    return {"sensor_id": id}
```
Aquí `{id}` es un path parameter. El valor se extrae directamente de la URL. También se convierte automáticamente al tipo indicado

# Responses HTML
Por defecto se responde un diccionario desde cualquier método de python, esto se transforma automáticament en JSON. Sin embargo habrán momentos que puede ser que requiera responder un HTML.

## On-the-fly
Puede responder una página usando
```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse
from datetime import datetime

app = FastAPI()

@app.get("/dinamico", response_class=HTMLResponse)
def respuesta_dinamica():
    html = f"""
    <html>
        <body>
            <h1>Hora actual</h1>
            <p>{datetime.now()}</p>
        </body>
    </html>
    """
    return html
```
Con esto, usted no está usando la memoria de almacenamiento, sólo la RAM

## Un recurso estático

```python
from fastapi.responses import FileResponse

@app.get("/", response_class=HTMLResponse)
def mostrar_html():
    return FileResponse("index.html")
```

## Una carpeta de recursos estática
Si quiere servir toda una carpeta puede hacer
```python
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")
```
Aquí deberá tener una carpeta de 'static' en la raíz del proyecto.

A partir de esto se puede dar rutas a las diferentes páginas usando
```python
from fastapi.responses import FileResponse
...
@app.get("/alias")
def raiz():
    return FileResponse("static/alfa.html")
```
En este caso, se sirve el archivo con el alias `alias` en la URL http://localhost:8000/alias

# Recopilar dependencias
Para guardar en un archivo la lista exacta de dependencias y sus versiones, de modo que puedan reinstalarse igual en otra máquina usando pip install -r requirements.txt.
```
pip freeze > requirements.txt
```
