# Borrador de arquitectura editorial

Estado: borrador operativo v0.2
Fecha: 2026-08-24
Alcance: `Materia Médica Digitalis` como obra completa, con cierre propuesto del Capítulo II y proyección temporal de capítulos.

## Propósito del documento

Este documento fija una arquitectura provisional para evitar que la escritura avance como una sucesión indefinida de fragmentos.

No reemplaza la tesis editorial ni el roadmap. Los concreta.

Su función es responder tres preguntas:

1. Qué está intentando decir el libro como obra completa.
2. Dónde termina el Capítulo II.
3. Cuál es el borrador temporal de capítulos posteriores.

## Tesis sintética de la obra

`Materia Médica Digitalis` no es un curso de Python con ejemplos médicos.

Es una obra médico-computacional sobre cómo convertir fenómenos biomédicos en representaciones formales, procedimientos verificables, decisiones trazables y sistemas responsables.

La progresión debe conservar una línea:

```text
decisión -> dato -> estructura -> algoritmo -> análisis -> sistema -> frontera
```

Cada capítulo debe responder una pregunta doble:

```text
Qué concepto computacional se aprende y qué problema biomédico permite pensar mejor?
```

Si una sección solo enseña sintaxis, falla. Si solo comenta medicina sin volverla estructura ejecutable, falla. Si produce código que funciona pero no enseña representación, trazabilidad, límites o criterio, queda corta.

## Principios de arquitectura

### 1. Claridad sin infantilización

El lector puede ser principiante en programación, pero no debe ser tratado como principiante intelectual. Las miniaturas pedagógicas deben abrir la puerta a problemas reales, no reemplazarlos.

### 2. Python como instrumento, no como destino

Python es el lenguaje de ejecución. La obra no debe organizarse alrededor de "aprender listas", "aprender condicionales" o "aprender pandas" como objetivos finales, sino alrededor de problemas de representación, decisión, verificación y sistema.

### 3. Biomédico significa trazable

En medicina y ciencias de la vida, un dato siempre trae método, contexto, incertidumbre, unidad, estado, población, sesgo o consecuencia. El libro debe entrenar esa sensibilidad desde el primer número hasta los modelos de frontera.

### 4. La progresión debe subir de escala

Cada unidad debe aumentar la escala:

- de una regla a un flujo;
- de un dato a una tabla;
- de una tabla a una estructura;
- de una estructura a un algoritmo;
- de un algoritmo a un sistema;
- de un sistema a responsabilidad, incertidumbre y frontera.

### 5. Cada cierre de capítulo debe dejar una capacidad

Un capítulo no termina cuando se agotan temas relacionados. Termina cuando el lector adquirió una capacidad nueva y puede usarla para entrar al capítulo siguiente.

## Estado actual

Capítulos editoriales vigentes:

1. **Capítulo I: El lenguaje de las decisiones.** Cerrado con 11 secciones.
2. **Capítulo II: Tipos de datos para problemas biomédicos.** Cerrado con 18 secciones.

La numeración técnica de archivos puede seguir siendo interna (`01-...`, `12-...`, `27-...`). Al usuario se le debe reportar por capítulo editorial y sección.

## Cierre propuesto del Capítulo II

El Capítulo II ya hizo el recorrido:

```text
tipos básicos -> ausencia/incertidumbre -> colecciones -> tablas -> esquemas -> pandas -> persistencia -> relaciones -> consultas -> APIs -> análisis -> pipelines
```

La unidad debe cerrar cuando el lector pueda tomar un lote biomédico simple, validarlo, analizarlo y producir un reporte mínimo sin perder trazabilidad.

Por eso el cierre propuesto es:

| Sección | Estado | Función |
|---|---:|---|
| 1. Números, unidades y mediciones | Hecha | Mostrar que un número necesita contrato biomédico. |
| 2. Texto libre, códigos y vocabularios controlados | Hecha | Separar texto narrativo, normalización, categoría y código. |
| 3. Booleanos, estados e incertidumbre | Hecha | Evitar reducir incertidumbre clínica a verdadero/falso. |
| 4. Fechas, tiempos, intervalos y granularidad clínica | Hecha | Representar tiempo sin fabricar precisión falsa. |
| 5. Ausencia de datos, centinelas y marcadores | Hecha | Diferenciar ausencia, cero, pendiente, no aplica e inválido. |
| 6. Listas, diccionarios y registros | Hecha | Agrupar observaciones sin ocultar campos semánticos. |
| 7. Tablas simples, limpieza y validación | Hecha | Pasar de forma tabular a tabla defendible. |
| 8. Esquemas mínimos y validación formal | Hecha | Convertir reglas dispersas en contrato versionado. |
| 9. pandas como herramienta tabular controlada | Hecha | Usar DataFrames como mesa de trabajo, no como garantía de validez. |
| 10. Archivos, tablas de trabajo y almacenamiento persistente | Hecha | Separar entrada, trabajo y persistencia. |
| 11. Entidades, relaciones y claves | Hecha | Pasar de tabla plana a identidad relacional. |
| 12. Restricciones, índices y consultas reproducibles | Hecha | Proteger invariantes y consultas parametrizadas. |
| 13. APIs mínimas y contratos de entrada/salida | Hecha | Convertir operaciones internas en fronteras contractuales. |
| 14. Análisis reproducibles | Hecha | Registrar datos, parámetros, consulta, denominador y resultado. |
| 15. Pipelines mínimos | Hecha | Encadenar carga, normalización, validación, análisis y reporte. |
| 16. Validación por lotes y reportes de calidad | Hecha | Resumir errores, advertencias, completitud y métricas por lote. |
| 17. Exportación, auditoría y artefactos compartibles | Hecha | Producir salidas estables: CSV/JSON/SQLite, reporte y bitácora mínima. |
| 18. Cierre integrador: del dato biomédico al flujo verificable | Hecha | Integrar el capítulo y preparar el salto hacia algoritmos clásicos. |

Decisión operativa v0.1:

```text
El Capítulo II debe cerrar en 18 secciones, salvo hallazgo fuerte de auditoría.
```

Por tanto, desde el estado actual faltan:

```text
0 secciones del Capítulo II.
```

## Capacidad final esperada del Capítulo II

Al cerrar el Capítulo II, el lector debe poder:

- representar mediciones, estados, fechas, ausencia y registros con contratos mínimos;
- limpiar y validar una tabla biomédica pequeña sin borrar errores;
- usar pandas de forma controlada;
- persistir datos en SQLite con claves, restricciones y consultas;
- exponer operaciones mediante contratos mínimos de API;
- ejecutar análisis reproducibles;
- construir un pipeline mínimo con artefactos y reporte de calidad.

Esta capacidad prepara el paso natural al Capítulo III: algoritmos fundamentales.

## Borrador temporal de capítulos

La obra completa se propone como 10 capítulos editoriales mayores.

La extensión por capítulo es orientativa. Se revisará al cierre de cada capítulo y en auditorías GLOBAL.

### Capítulo I. El lenguaje de las decisiones

Estado: cerrado.
Secciones: 11.

Función: enseñar que un algoritmo es una forma de formalizar decisiones bajo entradas, reglas, estados, excepciones, funciones y pruebas.

Capacidad final: leer y escribir procedimientos pequeños como decisiones explícitas, verificables y limitadas.

### Capítulo II. Tipos de datos para problemas biomédicos

Estado: cerrado.
Secciones previstas: 18.
Hechas: 18.
Faltan: 0.

Función: mostrar que los datos biomédicos no son valores neutros, sino contratos de representación con unidad, estado, ausencia, fuente, validación y trazabilidad.

Capacidad final: construir un flujo mínimo desde datos crudos hasta reporte verificable.

### Capítulo III. Algoritmos fundamentales con lectura biomédica

Estado: iniciado.
Secciones previstas: 12.
Hechas: 1.
Faltan: 11.

Función: reintroducir algoritmos clásicos como modelos mentales para búsqueda, ordenamiento, conteo, costo, repetición estructurada y decisión.

Borrador de secciones:

1. De los datos a los algoritmos. **Hecha.**
2. Búsqueda lineal: encontrar pacientes, eventos y errores.
3. Búsqueda binaria y datos ordenados.
4. Conteo, frecuencias y acumuladores robustos.
5. Hashing e identificadores.
6. Ordenamiento, priorización y triage.
7. Complejidad temporal y costo operativo.
8. Recursión como descomposición de problemas.
9. Algoritmos voraces y decisiones locales.
10. Programación dinámica: subproblemas y memoria.
11. Grafos introductorios: relaciones clínicas y biológicas.
12. Cierre: elegir algoritmo según representación, costo y riesgo.

### Capítulo IV. Datos médicos y razonamiento cuantitativo

Estado: no iniciado.
Secciones previstas: 10-12.

Función: conectar cálculo, incertidumbre, probabilidad y evaluación clínica.

Borrador de secciones:

1. Del dato validado al indicador.
2. Proporciones, tasas y denominadores.
3. Sensibilidad, especificidad y matrices de confusión.
4. Valores predictivos y prevalencia.
5. Riesgo absoluto, relativo y diferencia de riesgos.
6. Scores clínicos como algoritmos.
7. Calibración y discriminación.
8. Sesgo de selección y medición.
9. Series temporales clínicas introductorias.
10. Incertidumbre, intervalos y comunicación de resultados.
11. Cierre: calcular no equivale a decidir.

### Capítulo V. Estructuras de datos para ciencias de la vida

Estado: no iniciado.
Secciones previstas: 10-12.

Función: estudiar estructuras más allá de tablas simples: arrays, matrices, pilas, colas, árboles y grafos como formas de organizar fenómenos biomédicos.

Borrador de secciones:

1. Estructura de datos como decisión de acceso.
2. Arrays y vectores de mediciones.
3. Matrices: pacientes por variables, imágenes y señales.
4. Pilas y colas: procesos, turnos y eventos.
5. Conjuntos y pertenencia.
6. Mapas e índices.
7. Árboles de decisión y jerarquías.
8. Grafos de relaciones.
9. Representación de conocimiento médico.
10. Cierre: estructura, operación y costo.

### Capítulo VI. Algoritmos para datos clínicos

Estado: no iniciado.
Secciones previstas: 10-12.

Función: construir algoritmos aplicados a problemas clínicos sin presentarlos como herramientas asistenciales validadas.

Borrador de secciones:

1. Algoritmos clínicos como miniaturas responsables.
2. Laboratorio longitudinal.
3. Riesgo cardiovascular pedagógico.
4. Priorización en urgencias.
5. Contraindicaciones y reglas de exclusión.
6. Conciliación de medicamentos.
7. Seguimiento de cohortes.
8. Alertas y fatiga de alertas.
9. Texto médico estructurable.
10. Imagen y señal como datos computables.
11. Cierre: utilidad, daño potencial y validación.

### Capítulo VII. Bioinformática y genética computacional

Estado: no iniciado.
Secciones previstas: 10-12.

Función: llevar los algoritmos hacia secuencias, genomas, variantes y datos ómicos.

Borrador de secciones:

1. Secuencias como datos.
2. Distancia y similitud.
3. Alineamiento local y global.
4. Programación dinámica en alineamiento.
5. Búsqueda en secuencias grandes.
6. Ensamblaje como problema algorítmico.
7. Variantes y anotación.
8. Expresión génica como matriz.
9. Redes génicas.
10. Datos ómicos y dimensionalidad.
11. Cierre: de la secuencia al significado biológico.

### Capítulo VIII. Señales, imagen y neurología computacional

Estado: no iniciado.
Secciones previstas: 9-11.

Función: introducir señales fisiológicas, matrices de imagen, grafos cerebrales y modelos dinámicos como continuidad natural de estructuras y algoritmos.

Borrador de secciones:

1. Señales fisiológicas como series temporales.
2. Muestreo, ruido y filtrado.
3. Ventanas, eventos y características.
4. Imagen biomédica como matriz.
5. Segmentación conceptual.
6. Grafos cerebrales.
7. Modelos dinámicos introductorios.
8. Redes neuronales: analogía, potencia y límites.
9. Cierre: cerebro, señal y representación.

### Capítulo IX. Del algoritmo al sistema

Estado: no iniciado.
Secciones previstas: 9-11.

Función: pasar de fragmentos ejecutables a sistemas médico-computacionales mantenibles.

Borrador de secciones:

1. Proyecto, módulo y frontera.
2. Pruebas unitarias, integración y regresión.
3. Documentación técnica y clínica.
4. Interfaces y experiencia de uso.
5. APIs reales y contratos versionados.
6. Persistencia, migraciones y auditoría.
7. Seguridad, privacidad y mínimos operativos.
8. Observabilidad y monitoreo.
9. Despliegue responsable.
10. Cierre: un sistema también decide.

### Capítulo X. Frontera, modelos y responsabilidad

Estado: no iniciado.
Secciones previstas: 8-10.

Función: cerrar la obra conectando medicina de precisión, modelos predictivos, modelos fundacionales, agentes, incertidumbre, explicabilidad y responsabilidad.

Borrador de secciones:

1. Medicina de precisión como problema computacional.
2. Modelos predictivos y validación externa.
3. Explicabilidad e interpretabilidad.
4. Incertidumbre y abstención.
5. Modelos fundacionales en biomedicina.
6. Agentes y herramientas clínicas.
7. Sesgo, responsabilidad y gobernanza.
8. Límites de automatización.
9. Cierre general: pensar computacionalmente sin renunciar al juicio biomédico.

## Proyección cuantitativa provisional

| Capítulo | Estado | Secciones previstas | Hechas | Pendientes |
|---|---:|---:|---:|---:|
| I. El lenguaje de las decisiones | Cerrado | 11 | 11 | 0 |
| II. Tipos de datos para problemas biomédicos | Cerrado | 18 | 18 | 0 |
| III. Algoritmos fundamentales | Iniciado | 12 | 1 | 11 |
| IV. Datos médicos y razonamiento cuantitativo | No iniciado | 10-12 | 0 | 10-12 |
| V. Estructuras de datos para ciencias de la vida | No iniciado | 10-12 | 0 | 10-12 |
| VI. Algoritmos para datos clínicos | No iniciado | 10-12 | 0 | 10-12 |
| VII. Bioinformática y genética computacional | No iniciado | 10-12 | 0 | 10-12 |
| VIII. Señales, imagen y neurología computacional | No iniciado | 9-11 | 0 | 9-11 |
| IX. Del algoritmo al sistema | No iniciado | 9-11 | 0 | 9-11 |
| X. Frontera, modelos y responsabilidad | No iniciado | 8-10 | 0 | 8-10 |

Estimación total de secciones del libro:

```text
107-121 secciones editoriales, incluyendo las 30 ya hechas.
```

Estimación pendiente:

```text
77-91 secciones editoriales, según profundidad final de capítulos IV-X.
```

Esta cifra no debe interpretarse como obligación mecánica. Es una escala de obra. El libro puede cerrarse antes si alcanza su promesa intelectual sin rellenar temas.

## Hitos editoriales

### Hito 1: Cierre del Capítulo II

Objetivo: completar secciones 16-18.

Resultado esperado:

- capacidad mínima de flujo biomédico verificable;
- auditoría GLOBAL de cierre de capítulo;
- actualización del glosario;
- decisión final sobre el índice del Capítulo III. **Cumplida el 2026-08-24.**

### Hito 2: Inicio fuerte del Capítulo III

Objetivo: demostrar que el libro ya no está en "tipos de Python", sino en algoritmos.

Riesgo principal:

- que el Capítulo III repita ejemplos genéricos de algoritmos sin lectura biomédica.

Control:

- cada algoritmo debe tener problema biomédico, costo, límite y prueba.

### Hito 3: Primer bloque reputacional completo

Objetivo: tener Capítulos I-III cerrados y publicables como evidencia seria.

Lectura externa esperada:

- el autor domina fundamentos, representación de datos y algoritmos iniciales con criterio biomédico.

### Hito 4: Cruce hacia frontera

Objetivo: cerrar el paso desde clínica y datos hacia bioinformática, señales, modelos y sistemas responsables.

Control:

- no saltar a IA o modelos fundacionales antes de tener base de datos, algoritmos, validación y sesgo.

## Reglas para no improvisar

1. No crear una nueva sección sin ubicarla en el capítulo editorial y en el arco de la obra.
2. No extender el Capítulo II más allá de la sección 18 salvo razón documentada.
3. No iniciar Capítulo III sin una auditoría GLOBAL de cierre del Capítulo II.
4. No llamar "capítulo" a los números técnicos de archivo en reportes al usuario.
5. Antes de cada nueva sección, declarar:
   - capítulo editorial;
   - número de sección dentro del capítulo;
   - función de esa sección en la progresión;
   - siguiente sección prevista.
6. Cada capítulo debe cerrar con:
   - resumen de capacidad adquirida;
   - glosario revisado;
   - build verificado;
   - auditoría GLOBAL proporcional;
   - punto de entrada del siguiente capítulo.

## Modo autónomo opt-in

La producción editorial puede operar en modalidad autónoma solo si el usuario la activa explícitamente.

Nombre operativo:

```text
MMD-AUTO
```

Función:

- reducir autorizaciones repetitivas en ciclos ordinarios;
- permitir avanzar por secciones cuando la arquitectura ya define el siguiente paso;
- mantener verificación, commits, publicación y handoffs sin convertir cada fragmento en una negociación nueva.

No es modo por defecto.

En `MMD-AUTO`, el agente puede ejecutar el ciclo completo de una sección ordinaria:

```text
retomar estado -> redactar sección prevista -> actualizar navegación y docs -> verificar código -> build -> commit local -> publicar si fue autorizado -> verificar deploy -> documentar handoff -> pasar al siguiente fragmento
```

El modo debe detenerse y pedir autorización si la acción cambia arquitectura, alcance, tesis, herramientas, política de publicación, datos sensibles o cualquier regla de seguridad.

## Decisiones abiertas

Estas decisiones no bloquean la progresión ordinaria del Capítulo III:

1. Si los capítulos V y VI deben mantenerse separados o fusionarse parcialmente.
2. Si bioinformática y genética computacional merecen uno o dos capítulos según profundidad.
3. Si neurología computacional debe estar junto a señales e imagen o como capítulo independiente.
4. Qué recursos visuales interactivos deben incorporarse cuando una sección concreta los justifique.

Decisión cerrada:

- El nombre definitivo es `Algoritmos fundamentales con lectura biomédica`.
- La promesa pedagógica sigue el patrón `problema -> representación -> algoritmo -> propiedad -> costo -> límite`.
- El artefacto acumulativo es un registro biomédico sintético con primer escenario oncológico educativo.

## Respuesta operativa a la pregunta actual

Con este borrador, el estado operativo es:

```text
El Capítulo II está cerrado con 18 secciones.
El Capítulo III está iniciado con 1 de 12 secciones.
```

Siguiente operación:

1. Publicar y verificar `De los datos a los algoritmos`.
2. Redactar `Búsqueda lineal` como sección 2 del Capítulo III.
3. Conservar el laboratorio sintético, sus propiedades y sus límites.
