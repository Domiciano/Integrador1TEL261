<p><img src="https://logos-download.com/wp-content/uploads/2016/09/React_logo_wordmark-3000x1007.png" width="310"></p>

---

React es una librería de JavaScript desarrollada por Meta para construir interfaces de usuario de manera declarativa y eficiente, especialmente en aplicaciones web de una sola página (SPA). 

Permite crear componentes reutilizables que gestionan su propio estado, facilitando la construcción de interfaces dinámicas y reactivas. 

React utiliza un *Virtual DOM* para minimizar las manipulaciones reales del DOM, lo que mejora el rendimiento. Además, su enfoque basado en componentes y su integración con herramientas modernas lo han convertido en una de las tecnologías más populares para el desarrollo frontend.

Cree aplicaciones de react por medio de este comando

```sh
npm create vite@latest misuperapp -- --template react
```

<br><br>

# <a href="https://mui.com/material-ui/all-components/">Material UI</a>

```sh
npm install @mui/material @emotion/react @emotion/styled
```

```sh
npm install @mui/icons-material
```



# Diccionario de eventos de React

Aquí los eventos más comunes que puedes usar en componentes React, con ejemplos simples.

---

## Eventos de Mouse

### `onClick`
Se dispara cuando se hace clic en un elemento.
```jsx
<button onClick={() => alert('¡Hiciste clic!')}>Click aquí</button>
```

### `onDoubleClick`
Se ejecuta cuando se hace doble clic sobre un elemento.
```jsx
<div onDoubleClick={() => alert('Doble clic')}>Haz doble clic</div>
```

### `onMouseEnter`
Se activa cuando el cursor entra al área del elemento.
```jsx
<div onMouseEnter={() => console.log('Entraste al área')}>Pasa el mouse</div>
```

### `onMouseLeave`
Se activa cuando el cursor sale del área del elemento.
```jsx
<div onMouseLeave={() => console.log('Saliste del área')}>Pasa el mouse</div>
```

### `onMouseMove`
Detecta el movimiento del cursor sobre un elemento.
```jsx
<div onMouseMove={(e) => console.log(`X: ${e.clientX}, Y: ${e.clientY}`)}>Mueve el mouse</div>
```

### `onMouseDown`
Se activa al presionar el botón del mouse.
```jsx
<div onMouseDown={() => console.log('Mouse presionado')}>Presiona mouse</div>
```

### `onMouseUp`
Se ejecuta al soltar el botón del mouse.
```jsx
<div onMouseUp={() => console.log('Mouse soltado')}>Suelta el mouse</div>
```

### `onContextMenu`
Se ejecuta al hacer clic derecho.
```jsx
<div onContextMenu={(e) => { e.preventDefault(); alert('Menú contextual bloqueado'); }}>
  Clic derecho aquí
</div>
```

---

## Eventos de Teclado

### `onKeyDown`
Detecta cuando una tecla es presionada.
```jsx
<input onKeyDown={(e) => console.log(`Tecla abajo: ${e.key}`)} />
```

### `onKeyUp`
Se activa al soltar una tecla.
```jsx
<input onKeyUp={(e) => console.log(`Tecla arriba: ${e.key}`)} />
```

---

## Eventos de Formulario

### `onChange`
Se dispara cuando cambia el valor de un input o select.
```jsx
<input onChange={(e) => console.log(e.target.value)} />
```

### `onInput`
Similar a `onChange`, pero se dispara en cada entrada (más inmediato).
```jsx
<input onInput={(e) => console.log(e.target.value)} />
```

### `onSubmit`
Se activa al enviar un formulario.
```jsx
<form onSubmit={(e) => { e.preventDefault(); alert('Formulario enviado'); }}>
  <button type="submit">Enviar</button>
</form>
```

### `onFocus`
Se activa cuando un elemento recibe el foco.
```jsx
<input onFocus={() => console.log('Enfocado')} />
```

### `onBlur`
Se dispara cuando un elemento pierde el foco.
```jsx
<input onBlur={() => console.log('Desenfocado')} />
```

### `onReset`
Se ejecuta al reiniciar un formulario.
```jsx
<form onReset={() => console.log('Formulario reiniciado')}>
  <button type="reset">Reset</button>
</form>
```

### `onSelect`
Se activa cuando el usuario selecciona texto.
```jsx
<input onSelect={(e) => console.log('Texto seleccionado')} />
```

### `onInvalid`
Se activa cuando un input inválido intenta enviarse.
```jsx
<form onSubmit={(e) => e.preventDefault()}>
  <input required onInvalid={() => alert('Campo requerido')} />
  <button type="submit">Enviar</button>
</form>
```

---

## Eventos de Scroll

### `onScroll`
Se dispara cuando el usuario hace scroll.
```jsx
<div onScroll={() => console.log('Scroll detectado')} style={{ height: 100, overflow: 'auto' }}>
  <div style={{ height: 300 }}>Contenido largo</div>
</div>
```

---

## Eventos Misceláneos

### `onError`
Se ejecuta cuando ocurre un error de carga, por ejemplo en imágenes.
```jsx
<img src="imagen-inexistente.jpg" onError={() => alert('Error al cargar imagen')} />
```
