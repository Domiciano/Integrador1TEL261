# Introducción a React con Vite

## Índice

- [El problema del DOM manual](#el-problema-del-dom-manual)
- [Qué es React](#qué-es-react)
- [Qué es Vite](#qué-es-vite)
- [Módulo 2 — Configuración con Vite](#módulo-2--configuración-con-vite)
- [Módulo 3 — Componentes y JSX](#módulo-3--componentes-y-jsx)
- [Módulo 4 — Props y listas](#módulo-4--props-y-listas)
- [Módulo 5 — Estado con `useState`](#módulo-5--estado-con-usestate)
- [Módulo 6 — Mini-proyecto: Chat](#módulo-6--mini-proyecto-chat)
- [Módulo 7 — MQTT](#módulo-7--mqtt)

---

### El problema del DOM manual

Con HTML y JavaScript puro, cada vez que el usuario interactúa con la página hay que actualizar el DOM manualmente:

```javascript
// JavaScript puro
const lista = document.getElementById("lista");
const nuevoItem = document.createElement("li");
nuevoItem.textContent = "Sensor A";
lista.appendChild(nuevoItem);
```

Esto funciona para páginas simples, pero cuando la interfaz crece —múltiples componentes, datos que cambian, eventos encadenados— el código se vuelve difícil de mantener.

### Qué es React

React es una librería de JavaScript creada por Meta que resuelve este problema. En lugar de manipular el DOM directamente, describes cómo debería verse la interfaz en función de los datos, y React se encarga de actualizar lo que sea necesario.

> En React no dices "agrega este elemento". Dices "cuando los datos sean así, muestra esto".

### Qué es Vite

Vite es la herramienta que usamos para crear y ejecutar proyectos de React. Es más rápida y simple que las alternativas anteriores. Se encarga de:

- Crear la estructura inicial del proyecto
- Ejecutar un servidor local de desarrollo
- Compilar el proyecto para producción

---

## Módulo 2 — Configuración con Vite

### Crear el proyecto

```bash
npm create vite@latest mi-proyecto -- --template react
cd mi-proyecto
npm install
npm run dev
```

### Estructura del proyecto

```
mi-proyecto/
├── public/          # Archivos estáticos
├── src/
│   ├── App.jsx      # Componente raíz
│   ├── main.jsx     # Punto de entrada
│   └── index.css    # Estilos globales
├── index.html
└── package.json
```

Los archivos que más vamos a editar son los que están dentro de `src/`. En particular, `App.jsx` es donde empieza todo.

### Primer vistazo a `main.jsx`

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

Este archivo conecta React con el HTML. Le dice a React que tome el elemento con `id="root"` del `index.html` y lo controle desde ahí.

---

## Módulo 3 — Componentes y JSX

### Qué es un componente

Un componente es una función de JavaScript que retorna lo que se va a mostrar en pantalla. Es la unidad básica de React: todo en una aplicación React es un componente.

```jsx
function Saludo() {
  return <h1>Hola desde React</h1>
}
```

### JSX

JSX es la sintaxis que parece HTML pero está dentro de JavaScript. React la convierte al código que el navegador entiende.

```jsx
// Esto es JSX
function Tarjeta() {
  return (
    <div>
      <h2>Sensor A</h2>
      <p>Temperatura</p>
    </div>
  )
}
```

#### Reglas importantes de JSX

| Regla | Ejemplo correcto |
|---|---|
| Solo un elemento raíz | `<div>...</div>` o `<>...</>` |
| Las clases se escriben como `className` | `<div className="tarjeta">` |
| Las etiquetas siempre se cierran | `<input />` |
| Las expresiones van entre llaves | `<p>{nombre}</p>` |

### Usar un componente dentro de otro

```jsx
function App() {
  return (
    <div>
      <Tarjeta />
      <Tarjeta />
    </div>
  )
}
```

Cada vez que escribes `<Tarjeta />`, React ejecuta esa función y muestra su resultado.

---

## Módulo 4 — Props y listas

### Qué son las props

Las props son los datos que un componente padre le pasa a un componente hijo. Funcionan como los atributos en HTML, pero pueden ser cualquier valor de JavaScript.

```jsx
// Componente hijo: recibe props
function Tarjeta(props) {
  return (
    <div>
      <h2>{props.nombre}</h2>
      <p>{props.tipo}</p>
    </div>
  )
}

// Componente padre: pasa props
function App() {
  return (
    <div>
      <Tarjeta nombre="Sensor A" tipo="Temperatura" />
      <Tarjeta nombre="Sensor B" tipo="Humedad" />
    </div>
  )
}
```

También se pueden desestructurar directamente:

```jsx
function Tarjeta({ nombre, tipo }) {
  return (
    <div>
      <h2>{nombre}</h2>
      <p>{tipo}</p>
    </div>
  )
}
```

### Renderizar listas con `.map()`

Cuando tenemos un arreglo de datos, usamos `.map()` para convertir cada elemento en un componente. React requiere que cada elemento tenga un `key` único.

```jsx
const dispositivos = [
  { id: 1, nombre: "Sensor A", tipo: "Temperatura" },
  { id: 2, nombre: "Sensor B", tipo: "Humedad" },
  { id: 3, nombre: "Sensor C", tipo: "Presión" },
]

function App() {
  return (
    <div>
      {dispositivos.map((dispositivo) => (
        <Tarjeta
          key={dispositivo.id}
          nombre={dispositivo.nombre}
          tipo={dispositivo.tipo}
        />
      ))}
    </div>
  )
}
```

> La prop `key` no se muestra en pantalla. React la usa internamente para saber qué elementos actualizar cuando los datos cambian.

---

## Módulo 5 — Estado con `useState`

### Por qué no funcionan las variables normales

```jsx
// Esto NO funciona
function Contador() {
  let contador = 0

  function incrementar() {
    contador = contador + 1
    // React no sabe que el valor cambió, no actualiza la pantalla
  }

  return <button onClick={incrementar}>{contador}</button>
}
```

El problema es que React solo actualiza la pantalla cuando detecta un cambio en el **estado**. Una variable normal no dispara esa detección.

### `useState`

`useState` es la herramienta de React para manejar valores que cambian en el tiempo.

```jsx
import { useState } from 'react'

function Contador() {
  const [contador, setContador] = useState(0)

  function incrementar() {
    setContador(contador + 1)  // React detecta el cambio y re-renderiza
  }

  return <button onClick={incrementar}>{contador}</button>
}
```

#### Anatomía de `useState`

```jsx
const [valor, setValor] = useState(valorInicial)
//     |       |                   |
//     |       |                   Valor con el que empieza
//     |       Función para cambiar el valor
//     El valor actual
```

### Manejar un input con estado

```jsx
function Formulario() {
  const [nombre, setNombre] = useState("")

  return (
    <input
      value={nombre}
      onChange={(e) => setNombre(e.target.value)}
      placeholder="Nombre del dispositivo"
    />
  )
}
```

El estado `nombre` siempre refleja lo que el usuario escribe, y el input siempre muestra el estado actual. Este patrón se llama **componente controlado**.

---

## Módulo 6 — Mini-proyecto: Chat

Un input y un botón. Cada mensaje enviado se acumula en pantalla como una ventana de chat.

```jsx
import { useState } from 'react'

function App() {
  const [mensajes, setMensajes] = useState([])
  const [texto, setTexto] = useState("")

  function enviar() {
    if (texto === "") return
    setMensajes([...mensajes, texto])
    setTexto("")
  }

  return (
    <div>
      {mensajes.map((msg, i) => (
        <p key={i}>{msg}</p>
      ))}
      <input
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
        placeholder="Escribe un mensaje"
      />
      <button onClick={enviar}>Enviar</button>
    </div>
  )
}

export default App
```

### Puntos clave del mini-proyecto

| Concepto | Dónde aparece |
|---|---|
| `useState` con arreglo | `const [mensajes, setMensajes] = useState([])` |
| Spread operator para agregar | `setMensajes([...mensajes, texto])` |
| Limpiar el input tras enviar | `setTexto("")` |
| Renderizar la lista | `.map((msg, i) => <p key={i}>{msg}</p>)` |

---

## Bonus — `useEffect`

`useEffect` permite ejecutar código cuando algo cambia, o cuando el componente aparece por primera vez en pantalla. Es útil para cargar datos desde una API, entre otras cosas.

```jsx
import { useState, useEffect } from 'react'

function App() {
  const [dispositivos, setDispositivos] = useState([])

  // Se ejecuta una vez cuando el componente se monta
  useEffect(() => {
    fetch("https://mi-api.com/dispositivos")
      .then((res) => res.json())
      .then((data) => setDispositivos(data))
  }, [])  // El arreglo vacío significa "solo al montar"

  return (
    <div>
      {dispositivos.map((d) => (
        <Tarjeta key={d.id} nombre={d.nombre} tipo={d.tipo} />
      ))}
    </div>
  )
}
```

### Cuándo usar `useEffect`

| Segundo argumento | Cuándo se ejecuta |
|---|---|
| `[]` | Solo al montar el componente |
| `[valor]` | Al montar y cada vez que `valor` cambia |
| Sin segundo argumento | En cada re-render (raramente útil) |

---

## Resumen de conceptos

| Concepto | Para qué sirve |
|---|---|
| Componente | Función que retorna JSX; unidad básica de React |
| JSX | Sintaxis parecida a HTML que se escribe dentro de JavaScript |
| Props | Datos que el padre le pasa al hijo |
| `useState` | Maneja valores que cambian y dispara re-renders |
| `useEffect` | Ejecuta código en respuesta a montaje o cambios |
| `.map()` | Convierte un arreglo de datos en arreglo de componentes |
| `.filter()` | Crea un nuevo arreglo sin el elemento eliminado |

---

## Módulo 7 — MQTT

### Qué es MQTT

MQTT es un protocolo de mensajería ligero pensado para dispositivos IoT. Funciona con un modelo **publicar / suscribir**:

- Los dispositivos **publican** mensajes en un *topic* (ej: `sensores/temperatura`)
- Otros dispositivos o aplicaciones **se suscriben** a ese topic y reciben los mensajes
- Un servidor central llamado **broker** se encarga de distribuirlos

```
[Sensor] --publica--> [Broker] --entrega--> [App React]
                                --entrega--> [Otro cliente]
```

En esta sección usaremos el broker público `broker.hivemq.com`, que no requiere cuenta ni configuración.

---

### Prueba rápida con Node.js

Antes de tocar React, vamos a verificar que todo funciona desde la terminal con un script simple.

**Instalar la librería:**

```bash
npm install mqtt
```

**Crear el archivo `mqtt.js`:**

> Asegúrate de tener `"type": "module"` en tu `package.json`, o guarda el archivo como `mqtt.mjs`.

```js
import mqtt from 'mqtt'

const BROKER = 'mqtt://broker.hivemq.com'
const TOPIC  = 'test/101/beta'

const client = mqtt.connect(BROKER)

client.on('connect', () => {
  console.log('✅ Conectado al broker')

  // Suscribirse para recibir mensajes
  client.subscribe(TOPIC, () => {
    console.log(`📡 Suscrito a: ${TOPIC}`)
  })

  // Publicar un mensaje de prueba después de conectarse
  client.publish(TOPIC, 'Hola desde Node.js')
  console.log('📤 Mensaje enviado')
})

client.on('message', (topic, message) => {
  console.log(`📨 Mensaje recibido en [${topic}]: ${message.toString()}`)
})
```

**Ejecutar:**

```bash
node mqtt.js
```

Deberías ver en consola algo como:

```
✅ Conectado al broker
📡 Suscrito a: test/101/beta
📤 Mensaje enviado
📨 Mensaje recibido en [test/101/beta]: Hola desde Node.js
```

> El cliente recibe su propio mensaje porque también está suscrito al mismo topic. Esto es normal en MQTT.

---

### Integración con React — Recibir mensajes

Para usar MQTT dentro de un proyecto React con Vite, se usa la misma librería `mqtt`:

```bash
npm install mqtt
```

El patrón es siempre el mismo: conectarse al broker dentro de un `useEffect`, y guardar los mensajes que llegan en un estado con `useState`.

```jsx
import { useState, useEffect } from 'react'
import mqtt from 'mqtt'

const BROKER = 'wss://broker.hivemq.com:8884/mqtt'
const TOPIC  = 'test/101/beta'

function App() {
  const [mensajes, setMensajes] = useState([])

  useEffect(() => {
    const client = mqtt.connect(BROKER)

    client.on('connect', () => {
      client.subscribe(TOPIC)
    })

    client.on('message', (topic, payload) => {
      const texto = payload.toString()
      setMensajes((prev) => [...prev, texto])  // agrega al historial
    })

    // Al desmontar el componente, cerrar la conexión
    return () => client.end()
  }, [])

  return (
    <div>
      <h2>Mensajes recibidos</h2>
      {mensajes.map((msg, i) => (
        <p key={i}>{msg}</p>
      ))}
    </div>
  )
}

export default App
```

> **¿Por qué `wss://` y no `mqtt://`?**
> Los navegadores no pueden abrir conexiones TCP directas. Usan WebSockets (`wss://`) en su lugar. La librería `mqtt` lo maneja automáticamente si la URL empieza con `wss://`.

---

### Integración con React — Enviar mensajes

Para enviar, se necesita acceder al cliente MQTT fuera del `useEffect`. La forma más simple es guardar el cliente en un estado o en una referencia con `useRef`.

```jsx
import { useState, useEffect, useRef } from 'react'
import mqtt from 'mqtt'

const BROKER = 'wss://broker.hivemq.com:8884/mqtt'
const TOPIC  = 'test/101/beta'

function App() {
  const [texto, setTexto] = useState('')
  const clientRef = useRef(null)  // guarda el cliente sin provocar re-renders

  useEffect(() => {
    clientRef.current = mqtt.connect(BROKER)
    return () => clientRef.current.end()
  }, [])

  const enviar = () => {
    if (!texto) return
    clientRef.current.publish(TOPIC, texto)
    setTexto('')
  }

  return (
    <div>
      <input
        value={texto}
        onChange={(e) => setTexto(e.target.value)}
        placeholder="Escribe un mensaje"
      />
      <button onClick={enviar}>Enviar</button>
    </div>
  )
}

export default App
```

> **`useRef`** permite guardar un valor que persiste entre renders sin que cambiarlo dispare un re-render. Es ideal para guardar referencias a conexiones, timers, o cualquier cosa "externa" a React.

---

### Código completo: envío y recepción en React

Este componente combina todo lo anterior: se conecta, recibe mensajes, y permite enviar desde un input.

```jsx
import { useState, useEffect, useRef } from 'react'
import mqtt from 'mqtt'

const BROKER = 'wss://broker.hivemq.com:8884/mqtt'
const TOPIC  = 'test/101/beta'

function App() {
  const [mensajes, setMensajes] = useState([])
  const [texto, setTexto]       = useState('')
  const [conectado, setConectado] = useState(false)
  const clientRef = useRef(null)

  useEffect(() => {
    const client = mqtt.connect(BROKER)
    clientRef.current = client

    client.on('connect', () => {
      setConectado(true)
      client.subscribe(TOPIC)
    })

    client.on('message', (topic, payload) => {
      const texto = payload.toString()
      setMensajes((prev) => [...prev, texto])
    })

    return () => client.end()
  }, [])

  const enviar = () => {
    if (!texto || !conectado) return
    clientRef.current.publish(TOPIC, texto)
    setTexto('')
  }

  return (
    <div style={{ maxWidth: '500px', margin: '40px auto', fontFamily: 'sans-serif' }}>
      <h1>Cliente MQTT</h1>
      <p>{conectado ? '🟢 Conectado' : '🔴 Conectando...'}</p>

      {/* Envío */}
      <div style={{ marginBottom: '24px' }}>
        <input
          value={texto}
          onChange={(e) => setTexto(e.target.value)}
          placeholder="Escribe un mensaje"
          style={{ width: '100%', padding: '8px', marginBottom: '8px' }}
        />
        <button onClick={enviar} disabled={!conectado}>Enviar</button>
      </div>

      {/* Historial */}
      <h2>Mensajes recibidos</h2>
      {mensajes.length === 0 && <p style={{ color: '#999' }}>Sin mensajes aún.</p>}
      {mensajes.map((msg, i) => (
        <p key={i} style={{ background: '#f0f0f0', padding: '8px', borderRadius: '4px' }}>
          {msg}
        </p>
      ))}
    </div>
  )
}

export default App
```

### Resumen del módulo

| Concepto | Qué hace |
|---|---|
| `mqtt.connect(url)` | Crea la conexión al broker |
| `client.subscribe(topic)` | Se suscribe para recibir mensajes de ese topic |
| `client.publish(topic, msg)` | Envía un mensaje a ese topic |
| `client.on('message', fn)` | Listener que se ejecuta al llegar un mensaje |
| `useRef` | Guarda el cliente MQTT sin provocar re-renders |
| `wss://` | Protocolo necesario para MQTT desde el navegador |


---

## Módulo 8 — Peticiones HTTP con Axios

### Qué es Axios

Axios es una librería de JavaScript para hacer peticiones HTTP desde el navegador o Node.js. Es más cómoda que `fetch` nativo porque maneja automáticamente la conversión de JSON y el manejo de errores.

**Instalar:**

```bash
npm install axios
```

---

### GET — Obtener datos

El caso más común: pedir datos a una API y mostrarlos en pantalla. Se combina con `useEffect` para hacer la petición al montar el componente.

```jsx
import { useState, useEffect } from 'react'
import axios from 'axios'

function App() {
  const [dispositivos, setDispositivos] = useState([])
  const [cargando, setCargando] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    axios.get('https://mi-api.com/dispositivos')
      .then((respuesta) => {
        setDispositivos(respuesta.data)
        setCargando(false)
      })
      .catch((err) => {
        setError('No se pudieron cargar los datos.')
        setCargando(false)
      })
  }, [])

  if (cargando) return <p>Cargando...</p>
  if (error)    return <p>{error}</p>

  return (
    <ul>
      {dispositivos.map((d) => (
        <li key={d.id}>{d.nombre} — {d.tipo}</li>
      ))}
    </ul>
  )
}

export default App
```

> A diferencia de `fetch`, Axios lanza automáticamente un error cuando el servidor responde con un código 4xx o 5xx. No es necesario verificar `respuesta.ok` manualmente.

Con `async/await` el mismo código se puede escribir así:

```jsx
useEffect(() => {
  const cargar = async () => {
    try {
      const respuesta = await axios.get('https://mi-api.com/dispositivos')
      setDispositivos(respuesta.data)
    } catch (err) {
      setError('No se pudieron cargar los datos.')
    } finally {
      setCargando(false)
    }
  }

  cargar()
}, [])
```

---

### POST — Enviar datos

Se usa para crear un nuevo recurso en el servidor. El segundo argumento de `axios.post` es el cuerpo de la petición; Axios lo convierte a JSON automáticamente.

```jsx
import { useState } from 'react'
import axios from 'axios'

function NuevoDispositivo() {
  const [nombre, setNombre] = useState('')
  const [tipo, setTipo]     = useState('')
  const [resultado, setResultado] = useState(null)

  const enviar = async () => {
    if (!nombre || !tipo) return

    try {
      const respuesta = await axios.post('https://mi-api.com/dispositivos', {
        nombre,
        tipo,
      })
      setResultado(`Dispositivo creado con id: ${respuesta.data.id}`)
      setNombre('')
      setTipo('')
    } catch (err) {
      setResultado('Error al crear el dispositivo.')
    }
  }

  return (
    <div>
      <input
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
        placeholder="Nombre"
      />
      <input
        value={tipo}
        onChange={(e) => setTipo(e.target.value)}
        placeholder="Tipo"
      />
      <button onClick={enviar}>Crear</button>
      {resultado && <p>{resultado}</p>}
    </div>
  )
}

export default NuevoDispositivo
```

---

### Comparativa rápida

| | `fetch` nativo | `axios` |
|---|---|---|
| Conversión de JSON | Manual (`.json()`) | Automática (`respuesta.data`) |
| Errores HTTP (4xx/5xx) | No los lanza como error | Los lanza automáticamente |
| Cancelación de peticiones | Requiere `AbortController` | Integrado con `CancelToken` |
| Instalación | No requiere nada | `npm install axios` |

---

## Módulo 9 — CORS en FastAPI

### Qué es CORS

Cuando una aplicación React corre en `http://localhost:5173` e intenta consumir una API en `http://localhost:8000`, el navegador bloquea la petición por política de seguridad. Este mecanismo se llama **CORS** (Cross-Origin Resource Sharing).

El servidor es quien debe indicarle al navegador qué orígenes tienen permiso de hacer peticiones.

---

### Habilitar CORS en FastAPI

FastAPI incluye un middleware oficial para esto. Solo hay que agregarlo al crear la aplicación.

**Instalación** (si no tienes FastAPI aún):

```bash
pip install fastapi uvicorn
```

**Configuración con CORS abierto (desarrollo):**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],       # Permite cualquier origen
    allow_methods=["*"],       # Permite GET, POST, PUT, DELETE, etc.
    allow_headers=["*"],       # Permite cualquier cabecera
)

@app.get("/dispositivos")
def obtener_dispositivos():
    return [
        {"id": 1, "nombre": "Sensor A", "tipo": "Temperatura"},
        {"id": 2, "nombre": "Sensor B", "tipo": "Humedad"},
    ]

@app.post("/dispositivos")
def crear_dispositivo(dispositivo: dict):
    return {"id": 3, **dispositivo}
```

**Ejecutar el servidor:**

```bash
uvicorn main:app --reload
```

> `allow_origins=["*"]` acepta peticiones de cualquier dominio. Es suficiente para desarrollo local, pero en producción conviene reemplazar `"*"` por los dominios específicos que deben tener acceso.

---

### CORS restrictivo para producción

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://mi-app.com",
        "https://www.mi-app.com",
    ],
    allow_methods=["GET", "POST"],
    allow_headers=["Content-Type", "Authorization"],
)
```
