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

Axios es una librería de JavaScript para hacer peticiones HTTP desde el navegador o desde Node.js. Es más cómoda que el `fetch` nativo porque:

- Convierte la respuesta a JSON automáticamente
- Tiene mejor manejo de errores
- El código es más legible

### Instalar Axios

```bash
npm install axios
```

### GET — Obtener datos

Una petición **GET** se usa para **pedir datos** a un servidor sin modificar nada.

```jsx
import { useState, useEffect } from 'react'
import axios from 'axios'

function App() {
  const [usuarios, setUsuarios] = useState([])

  useEffect(() => {
    axios({
      method: 'GET',                                        // Método HTTP explícito
      url: 'https://jsonplaceholder.typicode.com/users',
    })
      .then((respuesta) => {
        setUsuarios(respuesta.data)
      })
      .catch((error) => {
        console.error('Error en GET:', error)
      })
  }, [])

  return (
    <div>
      <h2>Lista de usuarios</h2>
      {usuarios.map((u) => (
        <p key={u.id}>{u.name} — {u.email}</p>
      ))}
    </div>
  )
}

export default App
```

> **`jsonplaceholder.typicode.com`** es una API pública de prueba. No necesita cuenta ni token, devuelve datos falsos listos para practicar.

#### Qué devuelve Axios en un GET

```js
respuesta.data    // el cuerpo de la respuesta (ya convertido a objeto JS)
respuesta.status  // código HTTP, ej: 200
respuesta.headers // cabeceras de la respuesta
```

---

### POST — Enviar datos

Una petición **POST** se usa para **enviar datos nuevos** al servidor, por ejemplo para crear un registro.

```jsx
import { useState } from 'react'
import axios from 'axios'

function App() {
  const [nombre, setNombre] = useState('')
  const [respuesta, setRespuesta] = useState(null)

  const crearUsuario = () => {
    if (!nombre) return

    axios({
      method: 'POST',                                         // Método HTTP explícito
      url: 'https://jsonplaceholder.typicode.com/users',
      data: {                                                 // Cuerpo de la petición
        name: nombre,
        email: `${nombre.toLowerCase()}@test.com`,
      },
    })
      .then((respuesta) => {
        setRespuesta(respuesta.data)
        setNombre('')
      })
      .catch((error) => {
        console.error('Error en POST:', error)
      })
  }

  return (
    <div>
      <h2>Crear usuario</h2>
      <input
        value={nombre}
        onChange={(e) => setNombre(e.target.value)}
        placeholder="Nombre del usuario"
      />
      <button onClick={crearUsuario}>Crear</button>

      {respuesta && (
        <div>
          <h3>Respuesta del servidor:</h3>
          <p>ID asignado: {respuesta.id}</p>
          <p>Nombre: {respuesta.name}</p>
        </div>
      )}
    </div>
  )
}

export default App
```

> En `jsonplaceholder`, el POST no guarda realmente el dato, pero sí devuelve una respuesta simulada con un `id` asignado. Es suficiente para ver cómo funciona el flujo completo.

---

### GET vs POST — Diferencias clave

| Característica | GET | POST |
|---|---|---|
| Para qué sirve | Pedir / leer datos | Enviar / crear datos |
| Dónde van los datos | En la URL (query params) | En el cuerpo (`data`) |
| Se usa en Axios como | `method: 'GET'` | `method: 'POST'` |
| Respuesta típica | Lista o un objeto | El objeto recién creado |

---

### Código combinado: GET y POST en el mismo componente

```jsx
import { useState, useEffect } from 'react'
import axios from 'axios'

const URL_BASE = 'https://jsonplaceholder.typicode.com/posts'

function App() {
  const [posts, setPosts]   = useState([])
  const [titulo, setTitulo] = useState('')
  const [cuerpo, setCuerpo] = useState('')

  // GET al montar el componente
  useEffect(() => {
    axios({
      method: 'GET',
      url: URL_BASE,
    }).then((res) => {
      setPosts(res.data.slice(0, 5))  // solo los primeros 5
    })
  }, [])

  // POST al hacer clic en "Publicar"
  const publicar = () => {
    if (!titulo || !cuerpo) return

    axios({
      method: 'POST',
      url: URL_BASE,
      data: {
        title: titulo,
        body: cuerpo,
        userId: 1,
      },
    }).then((res) => {
      setPosts((prev) => [res.data, ...prev])  // agrega al inicio de la lista
      setTitulo('')
      setCuerpo('')
    })
  }

  return (
    <div style={{ maxWidth: '500px', margin: '40px auto', fontFamily: 'sans-serif' }}>
      <h1>Posts</h1>

      {/* Formulario POST */}
      <h2>Nuevo post</h2>
      <input
        value={titulo}
        onChange={(e) => setTitulo(e.target.value)}
        placeholder="Título"
        style={{ display: 'block', width: '100%', marginBottom: '8px', padding: '8px' }}
      />
      <textarea
        value={cuerpo}
        onChange={(e) => setCuerpo(e.target.value)}
        placeholder="Contenido"
        style={{ display: 'block', width: '100%', marginBottom: '8px', padding: '8px' }}
      />
      <button onClick={publicar}>Publicar (POST)</button>

      {/* Lista GET */}
      <h2>Posts existentes (GET)</h2>
      {posts.map((p) => (
        <div key={p.id} style={{ background: '#f0f0f0', padding: '8px', marginBottom: '8px', borderRadius: '4px' }}>
          <strong>{p.title}</strong>
          <p>{p.body}</p>
        </div>
      ))}
    </div>
  )
}

export default App
```

### Resumen del módulo

| Concepto | Qué hace |
|---|---|
| `npm install axios` | Instala la librería |
| `import axios from 'axios'` | La importa en el componente |
| `method: 'GET'` | Declara explícitamente el método HTTP |
| `method: 'POST'` | Declara explícitamente el método HTTP |
| `url` | Dirección del endpoint |
| `data` | Cuerpo del POST (lo que se envía) |
| `respuesta.data` | Lo que devuelve el servidor |
| `.catch(error)` | Maneja errores de red o del servidor |
