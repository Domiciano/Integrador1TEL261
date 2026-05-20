# Taller: Spec Driven Development para Prototipado Rapido

## Contexto

En este taller vas a construir una aplicacion web de visualizacion de datos de sensores usando un flujo de trabajo basado en especificaciones (SDD). La IA es una herramienta de construccion, no de pensamiento. Tu defines que se construye y como; la IA ejecuta dentro de los limites que tus specs establezcan.

---

## El problema de negocio

Una empresa necesita una aplicacion donde usuarios registrados puedan:

1. Ingresar con sus credenciales.
2. Ver la lista de sensores disponibles.
3. Seleccionar un sensor y visualizar graficamente los datos que ha recolectado.

Los datos ya existen y son accesibles mediante los siguientes endpoints publicos:

**Lista de sensores**
```
GET https://facelogprueba.firebaseio.com/api/sensors.json
```

**Lecturas de un sensor especifico**
```
GET https://facelogprueba.firebaseio.com/api/data/sensors/<SERIAL>/readings/readings.json

Ejemplo:
GET https://facelogprueba.firebaseio.com/api/data/sensors/IOJ145/readings/readings.json
```

Explora los endpoints antes de escribir cualquier spec. Necesitas entender la forma de los datos para escribir criterios de aceptacion validos.

---

## Stack y code style obligatorio

El proyecto usa React. Toda la logica de estado y efectos secundarios se maneja exclusivamente con `useState` y `useEffect`. No se permite el uso de librerias adicionales de estado o data fetching.

### Capa de datos: servicios con axios

Las llamadas HTTP viven en archivos `.js` separados dentro de `src/services/`. Cada archivo exporta funciones nombradas. El componente nunca llama a axios directamente.

```javascript
// src/services/sensorService.js
import axios from "axios";

const BASE_URL = "https://facelogprueba.firebaseio.com/api";

export async function getSensors() {
    let response = await axios.get(`${BASE_URL}/sensors.json`);
    return response.data;
}

export async function getSensorReadings(serial) {
    let response = await axios.get(
        `${BASE_URL}/data/sensors/${serial}/readings/readings.json`
    );
    return response.data;
}
```

### Componentes: tres tipos de funciones internas

Dentro de un componente existen exactamente tres tipos de funciones, con roles distintos y no intercambiables:

**1. Effect handlers**: se declaran dentro del `useEffect` donde se usan. Manejan la logica que ocurre al montar el componente o cuando cambia una dependencia. No se llaman desde el JSX.

**2. Event handlers**: se declaran dentro del cuerpo del componente, fuera de cualquier efecto. Se conectan a eventos del JSX (onClick, onChange, onSubmit, etc.).

**3. Funciones de servicio**: viven en `src/services/` y son importadas. Nunca se definen dentro del componente.

```javascript
import { useState, useEffect } from "react";
import { getSensors, getSensorReadings } from "../services/sensorService";

export default function SensorList() {

    const [sensors, setSensors] = useState([]);
    const [selected, setSelected] = useState(null);
    const [readings, setReadings] = useState(null);

    // Effect handler: declarado adentro del efecto, se ejecuta al montar
    useEffect(() => {
        async function fetchSensors() {
            let data = await getSensors();
            setSensors(data);
        }
        fetchSensors();
    }, []);

    
    // Event handler: declarado en el cuerpo del componente, conectado al JSX
    function handleSelectSensor(serial) {
        setSelected(serial);
    }

    return (
        <>
            {sensors.map(sensor => (
                <button key={sensor.serial} onClick={() => handleSelectSensor(sensor.serial)}>
                    {sensor.name}
                </button>
            ))}
            {readings && <pre>{JSON.stringify(readings, null, 2)}</pre>}
        </>
    );
}
```

### Reglas que el agente debe respetar

- `axios` solo se importa en archivos de `src/services/`. Nunca en un componente.
- Los effect handlers son siempre `async` y se declaran dentro del `useEffect`, no fuera.
- Los event handlers se nombran con el prefijo `handle` seguido de la accion en PascalCase: `handleSelectSensor`, `handleSubmit`, `handleDelete`.
- Un componente no llama a otro servicio que no sea el que le corresponde por responsabilidad.
- No se usan `.then()` ni `.catch()`. Toda la logica asincrona usa `async/await`.

Este estilo debe estar documentado en un archivo `CODE_STYLE.md` en la raiz del proyecto, y debe ser referenciado desde tu `GEMINI.md` o `CLAUDE.md` para que el agente lo respete en todo momento.

---

## Herramienta: spec-writer

Tienes disponible el siguiente skill para tu agente de IA (Gemini CLI u otro). Instalalo en tu proyecto antes de comenzar.

```
---
name: spec-writer
description: >
  Turns vague feature requests into structured specs, technical plans, and
  ordered task breakdowns ready for any coding agent. Use this skill when the
  user provides a feature description, a ticket, a PRD fragment, or any rough
  idea and asks to "write a spec", "plan this feature", "break this into
  tasks", or similar. Trigger keywords: spec, plan, tasks, feature, PRD,
  breakdown, acceptance criteria.
---

# spec-writer

You are an expert in Spec Driven Development (SDD). When this skill is active,
your job is to turn a vague feature description into three structured artifacts
— a Spec, a Plan, and a Task breakdown — in a single response.

## How to respond

Generate all three sections immediately. Do NOT ask clarifying questions first.
Instead, mark every implicit decision you make with [ASSUMPTION: ...] inline,
then collect all assumptions into a prioritized list at the end.

---

## Output format

### 1. Spec (functional, technology-agnostic)

- Purpose: One sentence describing what the feature does and why.
- Users: Who interacts with this feature and in what context.
- Requirements: Numbered list of functional requirements.
- Edge cases: What can go wrong, boundary conditions, unauthorized access.
- Acceptance criteria: Written in Given/When/Then format. Each criterion
  must be binary — pass or fail.

### 2. Plan (technical and concrete)

- Architecture: Where this fits in the existing system.
- Data model: New or modified entities, fields, relationships.
- API contracts: Endpoints, methods, request/response shapes, status codes.
- Testing strategy: Unit, integration, and e2e coverage expectations.
- Security constraints: Auth, authorization, input validation.
- Dependencies: External services, libraries, or internal modules required.

### 3. Tasks (ordered, self-contained)

Each task must:
- Be completable in a single agent session.
- Have its own acceptance criteria (binary, testable).
- List any tasks it depends on.
- Never say "implement the feature" — be specific.

Format:

Task N: [Title]
Depends on: Task X (or "none")
What to build: [Specific, concrete description]
Acceptance criteria:
- [Binary criterion]
- [Binary criterion]

---

## Assumptions summary (end of every response)

## Assumptions to review

1. [Decision made] — Impact: HIGH | MEDIUM | LOW
   Correct this if: [when the assumption is wrong]

---

## Quality rules

- The Spec MUST NOT contain implementation details.
- Every assumption is visible.
- Every task is independently verifiable.
- Acceptance criteria are binary.
```

---

## Pantallas de la aplicacion

Antes de escribir specs o codigo, debes disenar las pantallas en Figma (usando Figma Make u otro modo). Las pantallas minimas son:

- Pantalla de login
- Pantalla de lista de sensores
- Pantalla de detalle de sensor con grafica de lecturas

El diseno es un insumo para tus specs. No es un entregable independiente.

---

## Flujo de trabajo del taller

El flujo es secuencial. No avanzas al siguiente paso sin completar el anterior.

**Paso 1: Explorar**
Consulta los dos endpoints. Entiende la estructura de la respuesta. Documenta en un archivo `API_NOTES.md` los campos disponibles y el tipo de datos.

**Paso 2: Disenar**
Crea las pantallas en Figma con base en el problema de negocio y los datos disponibles.

**Paso 3: Especificar**
Usa el skill `spec-writer` para generar una spec por cada pantalla o flujo. Tu escribes el prompt de entrada; el agente genera el artefacto. Revisa cada spec y valida que los criterios de aceptacion sean binarios y probables.

**Paso 4: Configurar el agente**
Crea tu `GEMINI.md` o `CLAUDE.md` en la raiz del proyecto. Debe referenciar `CODE_STYLE.md` y establecer que el agente no puede salirse del patron de componentes definido.

**Paso 5: Construir**
Entrega las specs al agente tarea por tarea, en orden de dependencia. No entregues todas las tareas de golpe. Valida cada tarea contra sus criterios de aceptacion antes de continuar.

**Paso 6: Revisar**
Al final, verifica que cada criterio de aceptacion de todas las specs se cumpla en la aplicacion corriendo.

