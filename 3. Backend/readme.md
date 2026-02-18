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
Instalemos las dependencias necesarioas
```
pip install fastapi uvicorn
```

Luego vamos a crear un primer template para analizar 
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


# POST que recibe usuario y password
@app.post("/login")
def login(username: str = Form(...), password: str = Form(...)):
    print("Usuario:", username)
    print("Password:", password)
    return {"mensaje": "Datos recibidos en consola"}
```
