# Estrategia de aprendizaje: código limpio

## Propósito

Definir cómo el proyecto aprenderá, practicará y enseñará principios de código limpio durante la creación de `Materia Médica Digitalis`, sin plagiar fuentes privadas ni reducir el proceso a una lista superficial de estilo.

La intención no es usar `Clean Code` como texto para transferir al libro. La intención es que, mientras se escribe el libro, el autor aprenda a diseñar mejores ejemplos, mejores explicaciones y mejores decisiones de programación, y que ese aprendizaje se convierta en enseñanza pública con voz propia.

## Fuente privada disponible

El autor conserva una copia privada de `Clean Code` como material de consulta.

Estado:
- consulta selectiva pendiente: se usará por temas puntuales, no como lectura completa;
- no se ha extraído ni se extraerá contenido textual para el libro.

## Regla de uso

`Clean Code` puede usarse como una referencia privada entre otras para orientar aprendizaje técnico, pero no como molde textual ni como autoridad única del libro.

No se debe:
- copiar párrafos;
- parafrasear de forma cercana;
- reproducir secciones;
- transferir ejemplos distintivos al libro;
- tratar el libro como corpus para reescritura;
- reducir el aprendizaje de código limpio a comentarios, nombres o salidas esperadas.

Sí se puede:
- consultar temas puntuales;
- extraer principios generales con redacción propia;
- contrastar decisiones de diseño de ejemplos;
- mejorar nombres, funciones, comentarios, pruebas y manejo de errores;
- convertir la escritura del libro en un proceso de aprendizaje técnico explícito;
- enseñar al lector por qué una versión de código es más clara, verificable, mantenible o segura que otra;
- mostrar evolución progresiva desde una solución ingenua hacia una solución mejor razonada;
- convertir aprendizajes generales en reglas propias del proyecto.

## Visión operativa

`CODE CLEAN` dentro de este proyecto debe entenderse como una práctica editorial y técnica de doble vía:

1. **Aprender mientras se construye.** Cada ejemplo de código debe servir para mejorar el criterio del autor sobre representación, nombres, funciones, pruebas, errores, límites, legibilidad y responsabilidad.
2. **Enseñar mientras se escribe.** El libro debe mostrar al lector cómo pensar esa mejora, no solo presentar una versión final pulida.
3. **Evitar plagio y dependencia textual.** Las fuentes privadas pueden orientar estudio, pero el contenido público debe ser síntesis propia, con ejemplos propios y progresión propia.
4. **Conectar código limpio con biomedicina.** La limpieza del código no es estética aislada: reduce ambigüedad, errores silenciosos, mala representación de datos, pérdida de trazabilidad y falsa certeza.
5. **Construir criterio.** El objetivo no es que el lector memorice reglas de estilo, sino que entienda por qué una decisión de diseño hace que un algoritmo sea más auditable, modificable y responsable.

## Línea transversal del libro

`CODE CLEAN` no será una unidad aislada ni cambiará la meta principal del libro. La meta sigue siendo la definida por la documentación del proyecto: enseñar algoritmos en Python para medicina y ciencias de la vida.

La diferencia es que, mientras el libro enseña algoritmos, también debe enseñar a escribir código con mayor criterio. Esa enseñanza aparece de forma transversal:

- cuando se elige un nombre de variable;
- cuando se separa dato, interpretación, decisión y razón;
- cuando se transforma una regla ingenua en una regla verificable;
- cuando se decide si una función tiene una responsabilidad clara;
- cuando un ejemplo maneja datos faltantes, errores o excepciones;
- cuando una salida conserva trazabilidad;
- cuando una prueba mínima muestra que el procedimiento hace lo prometido.

El libro existe primero como proceso de aprendizaje del autor. El producto público debe compartir ese aprendizaje con otros lectores sin ocultar la progresión intelectual: no solo mostrar la respuesta final, sino enseñar cómo se reconoce una mala solución y cómo se mejora.

## Patrón didáctico preferido

Cuando el tema lo permita, usar el patrón:

1. **Versión ingenua o frágil.** Mostrar qué parece funcionar, qué oculta y qué riesgo crea.
2. **Crítica técnica.** Explicar por qué el diseño es insuficiente: nombre ambiguo, mezcla de responsabilidades, ausencia tratada como normalidad, salida sin razones, condición no defendible, falta de prueba o trazabilidad.
3. **Versión mejorada.** Reescribir con nombres, estructura, límites, salida y verificación más claros.
4. **Lección transferible.** Nombrar el principio aprendido con redacción propia y conectarlo con el problema biomédico.

Este patrón no debe volverse mecánico. Puede integrarse de forma natural cuando una comparación explícita rompa el ritmo editorial.

## Inmersión científica y crítica de fuentes

Cada implementación importante debe pasar por una inmersión científica y crítica de fuentes.

Esto significa que `Clean Code` no se tratará como autoridad incuestionable. Puede orientar, pero debe contrastarse con otras fuentes, prácticas y criterios:

- diseño de software;
- refactorización;
- pruebas;
- documentación técnica;
- guías del lenguaje Python;
- experiencia de sistemas biomédicos, datos clínicos y seguridad del paciente.

Regla:
- si una recomendación de código limpio mejora claridad pero reduce fidelidad biomédica, verificabilidad o seguridad, debe cuestionarse;
- si existe una alternativa mejor para el objetivo del libro, se adopta la alternativa y se explica por qué;
- la crítica debe entrenar al autor y al lector, no solo decorar la sección con referencias.

## Estrategia de consulta selectiva

No leer el PDF completo.

Flujo recomendado:

1. Obtener o construir un índice temático mínimo.
2. Mapear tema -> aplicación en `Materia Médica Digitalis`.
3. Consultar solo la sección necesaria cuando el libro esté trabajando ese tipo de problema.
4. Registrar el aprendizaje aplicable con redacción propia.
5. Registrar el aprendizaje aplicable con redacción propia para futuros libros técnicos.

## Mapa inicial de aplicación

Hasta tener índice extraído, usar este mapa conceptual aproximado:

| Necesidad del libro | Tema a consultar en Clean Code | Aplicación propia |
|---|---|---|
| Nombres de variables biomédicas | Nombres significativos | Variables con unidad, contexto y propósito explícito. |
| Funciones pedagógicas | Funciones | Ejemplos pequeños, verificables y con una responsabilidad clara. |
| Comentarios en código | Comentarios | Comentarios que expliquen intención, no traducción literal del código. |
| Manejo de errores | Errores/excepciones | Diferenciar error técnico, dato faltante y excepción del dominio. |
| Pruebas | Unit tests / testing | Mostrar salidas esperadas y pruebas mínimas cuando el ejemplo lo amerite. |
| Refactorización | Smells/refactoring | Mostrar evolución de una versión pobre a una versión responsable. |

## Integración con el libro

Cuando se escriba una sección con código:

1. identificar el objetivo pedagógico;
2. revisar si hay principio de programación limpia aplicable;
3. mejorar el ejemplo sin sacrificar claridad biomédica;
4. mostrar salida esperada;
5. explicar por qué la versión mejora la representación, legibilidad o seguridad;
6. no citar ni copiar `Clean Code` salvo que se use como referencia bibliográfica puntual y no textual.

## Estado de implementación 2026-05-15

Primera pasada parcial aplicada sobre los ejemplos Python publicables de la unidad inicial:

- `chapters/00-presaberes-minimos.md`;
- `chapters/01-que-es-un-algoritmo.md`;
- `chapters/02-pensar-en-pasos.md`;
- `chapters/03-variables-datos-decisiones.md`;
- `chapters/04-estados-condiciones-umbrales.md`;
- `chapters/05-excepciones-datos-faltantes-trazabilidad.md`.

Resultado verificado:

- todos los bloques Python actuales tienen una sección de `Salida esperada`;
- los ejemplos distinguen entradas, reglas, estados, razones, pendientes o trazabilidad mediante comentarios breves;
- los ejemplos biomédicos se presentan como miniaturas pedagógicas, no como escalas clínicas validadas;
- el build local de Jupyter Book compila correctamente con `.\venv\Scripts\jupyter-book.exe build .`.

Límite de esta pasada:

- esto todavía no agota la visión `CODE CLEAN`;
- falta convertir el aprendizaje de código limpio en una progresión explícita dentro del libro;
- falta decidir cómo aparecerán las comparaciones entre código ingenuo, código legible, código verificable y código responsable;
- falta conversar y precisar la visión completa del libro antes de seguir expandiendo esta línea.

Pendiente:

- revisar visualmente la portada rediseñada antes de consolidar commit;
- aplicar el mismo estándar a las próximas secciones sobre condicionales, bucles, funciones, errores y verificación;
- convertir cada nuevo ejemplo de código en una pieza ejecutable, verificable y explicada, no en un bloque decorativo.

## Reutilización del aprendizaje

Si durante el aprendizaje de código limpio aparece una regla reusable:

- documentarla primero en este proyecto si afecta el libro;
- registrarla como criterio editorial si afecta escritura de futuros libros técnicos;
- convertirla en práctica de programación general si aplica fuera del libro.

## Pendiente

- Extraer o transcribir manualmente el índice del PDF para convertir este documento en un mapa de consulta más preciso.
- Evaluar si conviene instalar una herramienta local de extracción de PDF o si el usuario prefiere aportar el índice como texto.
