# Introducción a React con Vite

## Índice

- [El problema del DOM manual](#el-problema-del-dom-manual)
- [Qué es React](#qué-es-react)
- [Qué es Vite](#qué-es-vite)
- [Módulo 2 — Configuración con Vite](#módulo-2--configuración-con-vite)
  - [Crear el proyecto](#crear-el-proyecto)
  - [Estructura del proyecto](#estructura-del-proyecto)
  - [Primer vistazo a `main.jsx`](#primer-vistazo-a-mainjsx)
- [Módulo 3 — Componentes y JSX](#módulo-3--componentes-y-jsx)
  - [Qué es un componente](#qué-es-un-componente)
  - [JSX](#jsx)
  - [Usar un componente dentro de otro](#usar-un-componente-dentro-de-otro)
- [Módulo 4 — Props y listas](#módulo-4--props-y-listas)
  - [Qué son las props](#qué-son-las-props)
  - [Renderizar listas con `.map()`](#renderizar-listas-con-map)
- [Módulo 5 — Estado con `useState`](#módulo-5--estado-con-usestate)
  - [Por qué no funcionan las variables normales](#por-qué-no-funcionan-las-variables-normales)
  - [`useState`](#usestate)
  - [Manejar un input con estado](#manejar-un-input-con-estado)
- [Módulo 6 — Mini-proyecto: Tarjetas de dispositivos](#módulo-6--mini-proyecto-tarjetas-de-dispositivos)
  - [Funcionalidades](#funcionalidades)
  - [Código completo](#código-completo)
  - [Puntos clave del mini-proyecto](#puntos-clave-del-mini-proyecto)
- [Bonus — `useEffect`](#bonus--useeffect)
  - [Cuándo usar `useEffect`](#cuándo-usar-useeffect)
- [Resumen de conceptos](#resumen-de-conceptos)

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

## Módulo 6 — Mini-proyecto: Tarjetas de dispositivos

Vamos a construir una aplicación donde el usuario puede registrar dispositivos y verlos como tarjetas.

### Funcionalidades

- Ingresar el nombre y tipo de un dispositivo
- Agregar el dispositivo a la lista
- Eliminar un dispositivo de la lista

### Código completo

```jsx
import { useState } from 'react'

// Componente para mostrar un dispositivo
function Tarjeta({ nombre, tipo, onEliminar }) {
  return (
    <div style={{
      border: "1px solid #ccc",
      borderRadius: "8px",
      padding: "16px",
      marginBottom: "8px"
    }}>
      <h3>{nombre}</h3>
      <p>{tipo}</p>
      <button onClick={onEliminar}>Eliminar</button>
    </div>
  )
}

// Componente principal
function App() {
  const [dispositivos, setDispositivos] = useState([])
  const [nombre, setNombre] = useState("")
  const [tipo, setTipo] = useState("")

  function agregarDispositivo() {
    if (nombre === "" || tipo === "") return

    const nuevo = {
      id: Date.now(),  // ID único basado en la hora actual
      nombre: nombre,
      tipo: tipo,
    }

    setDispositivos([...dispositivos, nuevo])
    setNombre("")   // Limpiar el input
    setTipo("")     // Limpiar el input
  }

  function eliminarDispositivo(id) {
    const actualizados = dispositivos.filter((d) => d.id !== id)
    setDispositivos(actualizados)
  }

  return (
    <div style={{ maxWidth: "500px", margin: "40px auto", fontFamily: "sans-serif" }}>
      <h1>Registro de dispositivos</h1>

      {/* Formulario */}
      <div style={{ marginBottom: "24px" }}>
        <input
          value={nombre}
          onChange={(e) => setNombre(e.target.value)}
          placeholder="Nombre del dispositivo"
          style={{ display: "block", marginBottom: "8px", width: "100%", padding: "8px" }}
        />
        <input
          value={tipo}
          onChange={(e) => setTipo(e.target.value)}
          placeholder="Tipo (temperatura, humedad...)"
          style={{ display: "block", marginBottom: "8px", width: "100%", padding: "8px" }}
        />
        <button onClick={agregarDispositivo}>Agregar dispositivo</button>
      </div>

      {/* Lista de tarjetas */}
      {dispositivos.map((dispositivo) => (
        <Tarjeta
          key={dispositivo.id}
          nombre={dispositivo.nombre}
          tipo={dispositivo.tipo}
          onEliminar={() => eliminarDispositivo(dispositivo.id)}
        />
      ))}

      {dispositivos.length === 0 && (
        <p style={{ color: "#999" }}>No hay dispositivos registrados.</p>
      )}
    </div>
  )
}

export default App
```

### Puntos clave del mini-proyecto

| Concepto | Dónde aparece |
|---|---|
| `useState` con arreglo | `const [dispositivos, setDispositivos] = useState([])` |
| Spread operator para agregar | `setDispositivos([...dispositivos, nuevo])` |
| `.filter()` para eliminar | `dispositivos.filter((d) => d.id !== id)` |
| Props con funciones | `onEliminar` pasada como prop a `<Tarjeta>` |
| Renderizado condicional | `{dispositivos.length === 0 && <p>...</p>}` |

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
