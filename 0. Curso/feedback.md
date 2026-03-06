# Retroalimentación — Análisis de Requerimientos EARS (15%)

**TEL 261 — Proyecto Integrador I | Período 2026-1**

---

## EQUIPO 1 — AgriSense-Sevilla

**Estudiantes:** Carol Andrea Mosquera Mena, Angy María Hurtado Osorio, Adri Jhoanny Martínez Murillo, María Paz Henao Bedoya

**Nota:** 5.0 / 5.0

### Fortalezas generales

Su trabajo es ejemplar en todos los aspectos de la rúbrica. Contextualizaron el proyecto con precisión en la introducción y citaron la metodología EARS correctamente desde la fuente original (Mavin & Wilkinson, 2010). Sus 20 referencias son pertinentes, correctamente citadas en IEEE y directamente relacionadas con los requisitos técnicos del sistema. Delimitaron geográficamente la formulación del problema y la sustentaron con datos concretos. Su justificación presenta hipótesis verificables respaldadas por literatura científica.

Sus requisitos cubren los cinco patrones EARS con sintaxis precisa, criterios de verificación cuantitativos, justificación técnica individualizada por requisito y una matriz de trazabilidad que conecta cada requisito con el problema identificado en el PIT1 y los NFR correspondientes.

### Consejos para seguir mejorando

1. **En los requisitos STATE, asegúrense de que el estado precondición sea un estado del sistema discreto y registrado.** En REQ-STATE-04 y REQ-STATE-05, "CUALQUIER ETAPA ACTIVA" es válido como precondición, pero en versiones futuras del documento conviene nombrar los estados posibles explícitamente (p. ej., listarlos) para facilitar la implementación del motor de alertas sin ambigüedad.

2. **Los criterios de verificación son su punto más diferenciador — manténganlos.** En proyectos reales de ingeniería de requisitos, pasar de "el sistema generará una alerta" a "inyectar lectura = 26 °C durante 48 h y verificar alerta en ≤ 5 min" transforma un requisito deseable en uno verificable. Consideren en la siguiente fase añadir un caso de prueba de integración formal por cada requisito COM.

3. **El requisito REQ-OPT-03 clasifica el edge computing como opcional**, pero el alcance en la sección 2 lo describe como componente base de la gateway. Verifiquen la consistencia entre el alcance y la clasificación OPT de ese requisito — si el edge computing es base, debería ser UBI o STATE, no OPT.

4. **Revisen la numeración de los IDs en los requisitos COM.** El documento referencia "REQ-OPT-07" en la matriz de trazabilidad, pero en la sección 5.4 solo existen hasta REQ-OPT-05. Verifiquen que todas las referencias cruzadas entre secciones sean consistentes.

5. **Consideren añadir una sección de requisitos de no funcionamiento (Unwanted behavior / NOT-EARS)** para escenarios que el sistema explícitamente NO debe hacer (p. ej., el sistema NO DEBERÁ transmitir datos de sensores sin cifrado AES-128). Este es el sexto patrón EARS frecuentemente omitido.

---

## EQUIPO 2 — Sistema de Control de Acceso a Salas

**Estudiantes:** Jhony Bolaños García, José David Libreros Álvarez, Santiago Mosquera Sánchez, Luis Fernando Soto Bedoya

**Nota:** 3.0 / 5.0

> **Nota de evaluación:** Su entrega consistió únicamente en el documento de requisitos EARS. El contexto (introducción, problema, justificación, referencias) se evaluó a partir del PIT1 que entregaron previamente, el cual describe con claridad el sistema de reservas de la Universidad ICESI, el cuello de botella operativo del área de Multimedios y presenta un análisis causa-raíz estructurado con análisis PESTLE. La calificación de contexto refleja esa entrega previa.

### Análisis por criterio de la rúbrica de requisitos

**Uso del patrón EARS — Aceptable:** Los requisitos utilizan las palabras clave de EARS (*Cuando*, *Mientras*, *Si*), lo que indica familiaridad con la sintaxis. Sin embargo, ningún requisito está clasificado explícitamente por tipo de patrón (UBI, STATE, EVT, OPT, COM) ni organizado en secciones separadas por categoría. Los patrones OPT y COM están completamente ausentes, y no existe ningún requisito UBI (ubicuo sin condición previa). Para alcanzar el nivel Bueno, cada requisito debe llevar su etiqueta de patrón y el documento debe cubrir los cinco tipos.

**Claridad — Bueno:** La mayoría de los requisitos son comprensibles y concretos. RF-02 especifica que la clave no puede reutilizarse en otro intervalo, RF-08 lista los campos del log de auditoría, y RF-11 define el umbral de bloqueo con precisión (5 intentos, 10 minutos). La redacción es directa y sin ambigüedades graves. Se pierde un punto de claridad en RF-10, que describe una decisión de diseño técnico ("el backend se comunicará con las cerraduras a través de la red") en lugar de especificar el comportamiento observable del sistema.

**Completitud — Aceptable:** El conjunto de requisitos cubre los flujos principales (reserva, generación de clave, acceso, cancelación, auditoría, bloqueo por intentos fallidos) pero presenta vacíos importantes: no hay requisitos de autenticación o gestión de sesión del usuario, no hay requisitos para administración del sistema (crear/editar/eliminar salas), no existe ningún requisito opcional (OPT) ni complejo (COM), y ningún requisito incluye criterio de verificación. Sin criterio de verificación, ningún requisito es formalmente comprobable.

**Uso de condiciones (WHEN, IF, WHILE, WHERE) — Aceptable:** Las condiciones WHEN (*Cuando*), IF (*Si*) y WHILE (*Mientras*) están presentes y aplicadas correctamente en la mayoría de los casos. La condición WHERE (*DONDE*) está completamente ausente, lo que impide modelar componentes opcionales como el módulo de cerraduras electrónicas (que podría no estar disponible en todos los espacios). Algunos requisitos IF/WHILE podrían precisarse más: RF-05 usa *"dentro del tiempo adecuado"* como condición, que es vaga; debería decir *"dentro de la franja horaria asignada a esa reserva"*.

**Comportamiento del sistema (THEN / SHALL) — Aceptable:** El comportamiento del sistema está descrito en la mayoría de los requisitos, pero la convención SHALL no se aplica. Los verbos usados son "debe", "deberá" (minúscula) y en algunos casos futuro simple ("validará", "se comunicará"), lo que introduce ambigüedad sobre si el comportamiento es obligatorio o esperado. En EARS, todo comportamiento obligatorio del sistema se expresa con **DEBERÁ** (mayúsculas) para diferenciarlo de comportamientos recomendados o de diseño. RF-10 en particular suena a restricción de arquitectura, no a requisito de comportamiento observable.

**Consistencia de formato — Bueno:** El formato de tabla (ID, Tipo, Requerimiento) es uniforme en todos los requisitos funcionales y no funcionales. Los encabezados son coherentes y la separación entre funcionales y no funcionales es clara. La inconsistencia principal es el uso mezclado de tiempo verbal (condicional "debe", futuro "validará", subjuntivo "se comunicará") dentro de requisitos del mismo documento. Estandarizar a DEBERÁ en todos los casos resolvería esta inconsistencia con poco esfuerzo.

### Aspectos adicionales a mejorar

1. **Clasifiquen y etiqueten cada requisito según el patrón EARS explícito.** Organícenlos en secciones separadas con su etiqueta y estructura canónica:
   - **UBI:** `El sistema DEBERÁ [acción].`
   - **STATE:** `MIENTRAS [estado], CUANDO [condición], EL sistema DEBERÁ [acción].`
   - **EVT:** `CUANDO [evento], EL sistema DEBERÁ [acción].`
   - **OPT:** `DONDE [componente opcional presente], EL sistema DEBERÁ [acción].`
   - **COM:** Combinación de las cláusulas anteriores.

2. **Incluyan requisitos ubicuos (UBI).** El documento no tiene ninguno. Por ejemplo:
   - *"El sistema DEBERÁ registrar un log de auditoría de toda operación que modifique el estado de una reserva."*
   - *"El sistema DEBERÁ garantizar que todas las comunicaciones entre cliente y servidor usen TLS 1.3."*

3. **Incluyan al menos un requisito complejo (COM)** que integre las condiciones dispersas entre RF-04, RF-05 y RF-06. Por ejemplo:
   *"DONDE el sistema incluya cerraduras electrónicas, MIENTRAS una reserva esté activa y dentro de su franja horaria, CUANDO un usuario ingrese el código temporal en la cerradura, SI el código es válido para esa sala y ese horario, ENTONCES EL sistema DEBERÁ conceder el acceso y registrar el evento con marca de tiempo."*

4. **Agreguen criterios de verificación a cada requisito.** Ejemplo para RF-02:
   *"Criterio de verificación: Generar 100 reservas y verificar que cada clave es única; intentar usar la misma clave en dos franjas horarias distintas y verificar que el sistema la rechaza en ambos casos."*

5. **Revisen la categorización de algunos RNF.** RNF-02 (*"El sistema cifrará los códigos..."*) describe una acción del sistema — es un requisito funcional de seguridad, no un atributo de calidad. Los NFR deben definir atributos medibles. Reformúlenlo: *"El sistema DEBERÁ almacenar todos los códigos de acceso cifrados con bcrypt (factor de costo ≥ 12), sin almacenamiento en texto plano."* RNF-08 duplica parcialmente RNF-02 — consolídenlos.

6. **RNF-03 (Ley 1581) es un requisito de conformidad regulatoria, no un NFR de calidad.** Inclúyanlo como UBI: *"El sistema DEBERÁ cumplir con la Ley 1581 de 2012 sobre protección de datos personales, garantizando consentimiento informado para el tratamiento de datos de usuarios."*

7. **Los requisitos no trazan al contexto institucional específico del PIT1.** Su PIT1 describe el sistema de "planetas" de ICESI, el cuello de botella de Multimedios y situaciones vividas como estudiantes. Los requisitos entregados son completamente genéricos y podrían aplicar a cualquier sistema de reservas del mundo. Anclen cada requisito al contexto real — mencionen ICESI, los roles específicos (estudiante, docente, personal de Multimedios) y los escenarios exactos del problema. Ejemplo:
   - **Genérico (actual):** *"Cuando un usuario autenticado seleccione una sala disponible..."*
   - **Contextualizado (mejor):** *"CUANDO un usuario institucional de la Universidad ICESI (estudiante, docente o personal) seleccione un espacio académico disponible en el sistema de reservas..."*

---

## EQUIPO 3 — CatFeeder IoT (Comederos Inteligentes para Gatos Callejeros)

**Estudiantes:** Laura Sofía Armero Ordóñez, Ommhes Samuel León Díaz, José David Rodríguez Pinilla, Nicol Valeria Suárez Serrato

**Nota:** 4.0 / 5.0

### Fortalezas generales

Su trabajo tiene una introducción muy bien fundamentada con referentes ecológicos internacionales y estudios locales colombianos. Su formulación del problema es profunda e incluye análisis causal, árbol de problemas y delimitación geográfica precisa. Su justificación de viabilidad es sólida y cita antecedentes concretos del contexto colombiano (Fundación Paraíso de la Mascota, Fundación Alma Perruna, modelo HelloStreetCat). Sus referencias están en formato IEEE y son pertinentes. Su idea del proyecto es innovadora y bien argumentada.

### Aspectos a mejorar en los requisitos EARS

1. **Los requisitos STATE están incompletos — les falta el disparador CUANDO.** El patrón STATE en EARS tiene dos partes obligatorias: el estado precondición (*MIENTRAS*) y un evento o condición disparador (*CUANDO*). Sus STATE actuales solo tienen *MIENTRAS*, lo que los convierte técnicamente en requisitos ubicuos con condición, no en STATE:

   - **Actual (incorrecto):** *"Mientras el comedero inteligente esté encendido y en funcionamiento, el sistema deberá monitorear constantemente el estado del dispositivo."*
   - **Correcto:** *"MIENTRAS el comedero esté encendido y en funcionamiento, CUANDO un componente (sensor, dispensador o cámara) deje de responder, EL sistema DEBERÁ registrar el fallo y notificar al administrador."*

   Reescriban todos sus STATE añadiendo un disparador CUANDO que defina exactamente qué activa el comportamiento dentro de ese estado.

2. **Los requisitos COM no siguen el patrón completo de EARS.** Un COM combina DONDE + MIENTRAS + CUANDO/SI + ENTONCES. Sus COM actuales son EVT con una condición adicional, pero sin las cláusulas de componente (DONDE) ni el ENTONCES explícito:

   - **Actual (incompleto):** *"Cuando un usuario realice una donación y el comedero tenga alimento disponible, el sistema deberá activar el dispensador..."*
   - **Correcto:** *"DONDE el sistema incluya el módulo de donaciones digitales, MIENTRAS el comedero tenga alimento disponible, CUANDO un usuario realice una donación confirmada en la plataforma, ENTONCES EL sistema DEBERÁ activar el dispensador, liberar la porción correspondiente de alimento y registrar el evento en el historial."*

3. **Los requisitos OPT usan "En caso de que" — el patrón EARS usa DONDE.** La plantilla correcta para requisitos opcionales es: *"DONDE [componente opcional está presente], EL sistema DEBERÁ [acción]."* Cambien "En caso de que el sistema incluya..." por "DONDE el sistema incluya...", nombrando el componente opcional de forma específica:

   - **Actual:** *"En caso de que el sistema incluya herramientas de análisis de datos, deberá generar reportes..."*
   - **Correcto:** *"DONDE el sistema incluya el módulo de análisis de datos de visitas, EL sistema DEBERÁ generar reportes de frecuencia de visitas por comedero y franjas horarias de mayor actividad."*

4. **Algunos requisitos no funcionales son demasiado vagos — incluyan métricas verificables.**
   - NFR-01: *"mediante mecanismos de seguridad adecuados"* no es verificable. Especifiquen el protocolo: *"...usando TLS 1.3 para comunicaciones externas y AES-256 para datos en reposo."*
   - NFR-03: *"conexión estable a internet"* no define qué es estable. Especifiquen: *"...con una latencia máxima de 500 ms para sincronización de datos entre el comedero y la plataforma, y reconexión automática en menos de 30 segundos tras una interrupción."*
   - NFR-06: *"facilitar las tareas de mantenimiento"* es subjetivo. Cuantifíquenlo: *"El sistema DEBERÁ permitir actualización de firmware remota (OTA) en menos de 10 minutos sin interrumpir el monitoreo activo."*

5. **Ningún requisito incluye criterio de verificación.** Esto es fundamental para hacer el documento formalmente verificable. Para cada requisito añadan cómo se comprueba que el sistema lo cumple. Ejemplo para EVT-01:
   *"Criterio de verificación: Realizar 20 donaciones de prueba en la plataforma y verificar que el dispensador se activa en menos de 5 segundos en cada caso; registrar los 20 eventos en el historial."*

6. **EVT-02 (detección de movimiento) carece de especificidad técnica.** *"Deberá registrar imágenes o video del evento"* — ¿cuántos frames? ¿Durante cuántos segundos? ¿Con qué resolución mínima? Especifiquen los parámetros técnicos mínimos para que el requisito sea implementable y verificable.

7. **UBI-01 mezcla el propósito de negocio con el comportamiento del sistema.** *"Estas donaciones se utilizarán para activar el comedero y contribuir al mantenimiento..."* — la segunda parte describe el propósito, no el comportamiento técnico del sistema. Sepárenlos: el propósito va en la justificación; el requisito debe describir solo lo que el sistema hace. Reformulen: *"El sistema DEBERÁ permitir a los usuarios registrar donaciones en la plataforma digital mediante los métodos de pago habilitados (Nequi, Daviplata, PSE), generando un recibo de confirmación inmediato."*

8. **Verifiquen que los actores del sistema estén claramente identificados en cada requisito.** Algunos requisitos no especifican qué componente del sistema ejecuta la acción. En EVT-02: *"Cuando el sistema detecte movimiento..."* — ¿lo detecta el sensor PIR del comedero, la cámara, o ambos? Ser específico sobre el actor y el sensor reduce ambigüedad para el equipo de implementación.

---

## Nota sobre el documento de requisitos del Equipo 3

**José David Rodríguez Pinilla**, **Nicol Valeria Suárez Serrato** y **Ommhes Samuel León Díaz** aparecen correctamente como integrantes del equipo en el documento de contexto PIT1 ("The Walking Cats"). Sin embargo, el documento de requisitos entregados (V 1.3) solo muestra el nombre de Laura Sofía Armero Ordóñez en la portada. Para la siguiente entrega, asegúrense de que **todos los integrantes del equipo aparezcan explícitamente en la portada del documento de requisitos**. Adicionalmente, verifiquen que la entrega individual de cada integrante esté registrada correctamente en la plataforma.

---

*Profesor: Domiciano Rincón | TEL 261 — Proyecto Integrador I | Marzo 2026*
