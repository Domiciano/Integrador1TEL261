
# React Router

React Router es una librería de JavaScript que permite a las aplicaciones hechas con React tener navegación entre páginas sin necesidad de recargar el navegador.

En otras palabras, nos ayuda a construir aplicaciones de una sola página (SPA) donde podemos tener múltiples "pantallas" (como Inicio, Acerca de, Contacto), cada una con su propia URL, pero sin salir de la misma aplicación.

Para instalarlo

```bash
npm install react-router-dom
```

# Páginas

Crea una carpeta llamada `pages` dentro de `src/` para agregar nuestras páginas o screens.

Las screen son componentes que representan toda la pantalla que ve el usuario. Regularmente son componentes que son compuestos por componentes más pequeños


```jsx
export default function Login() {
  return (
    <h1>Login</h1>
  );
}
```

```jsx
export default function About() {
  return (
    <h1>Acerca de mi super aplicación</h1>
  );
}
```

```jsx
export default function Home() {
  return (
    <h1>Home</h1>
  );
}
```

# Definición de rutas

Una vez tenemos algunos 

```jsx
import { BrowserRouter, Routes, Route, Link } from "react-router-dom";
import Home from "./pages/Home";
import About from "./pages/About";
import Login from "./pages/Login";

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Inicio</Link> | 
        <Link to="/about">Acerca de</Link> | 
        <Link to="/contact">Contacto</Link>
      </nav>

      <Routes>
        <Route path="/" element={<Login />} />
        <Route path="/about" element={<About />} />
        <Route path="/home" element={<Home />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```


¿Qué pasa si se intenta acceder a una ruta inexistente?

```jsx
<Route path="*" element={<NotFound />} />
```


# Navegación programática

Podemos usar `Link` si queremos que nuestro usuario pueda viajar a un click. Sin embargo también puede navegar programáticamente asi:

```jsx
import {useNavigate} from "react-router-dom";

export function MyComponent() {
  const navigate = useNavigate();
  ...
  navigate('/home')
  ...
}
```

# Extracción de datos de formularios

En este ejemplo sencillo podemos ver cómo se hace la extracción de valores

```jsx
import { useState } from "react";
import { Container, Box, TextField, Button } from "@mui/material";

function App() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Correo:", email);
    console.log("Contraseña:", password);
  };

  return (
    <Container maxWidth="sm" sx={{ mt: 4 }}>
      <Box
        component="form"
        sx={{ display: "flex", flexDirection: "column", gap: 2 }}
      >
        <TextField
          label="Correo electrónico"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          fullWidth
        />
        <TextField
          label="Contraseña"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          fullWidth
        />
        <Button onClick={handleSubmit} type="submit" variant="contained">
          Enviar
        </Button>
      </Box>
    </Container>
  );
}

export default App;
```


# `useEffect`

Es un hook de React que permite ejecutar efectos secundarios (side effects) en un componente funcional, como llamadas a APIs, suscripciones, o manipulación del DOM, después de que el componente se monta, actualiza o desmonta, dependiendo de las dependencias que se le pasen.

```jsx
useEffect(async ()=>{
      let alfa = await axios.get("http://localhost:8080/api/v1/courses", {
        headers: {
          Authorization: `Bearer ${token}`
        }
      });
      console.log(alfa);
},[]);
```
