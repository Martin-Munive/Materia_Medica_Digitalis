# Estado del proyecto

## Propósito

Este documento resume la definición actual de `Materia Médica Digitalis`, el avance real, los pendientes y el porcentaje estimado de progreso para retomar el trabajo sin depender del contexto conversacional.

## Definición del proyecto

`Materia Médica Digitalis: Algoritmos en Python para Medicina y Ciencias de la Vida` es un libro web técnico-científico construido con Jupyter Book y publicado mediante GitHub Pages.

El proyecto no es un curso rápido de sintaxis ni una colección de apuntes. Es una arquitectura de aprendizaje para formar pensamiento algorítmico aplicado a medicina, ciencias de la vida, datos biomédicos, estructuras de información, algoritmos clásicos, bioinformática, genética computacional, neurología computacional y sistemas médico-computacionales responsables.

La obra busca construir una arquitectura intelectual entre medicina, ciencias de la vida y computación: convertir fenómenos biomédicos complejos en representaciones formales, procedimientos verificables, decisiones trazables y límites declarados.

## Propósito editorial

El libro busca que el lector aprenda a:

- traducir fenómenos biomédicos a datos, reglas, estados, condiciones y algoritmos;
- escribir Python como instrumento de pensamiento, no como sintaxis aislada;
- reconocer límites, excepciones, datos faltantes, sesgos y trazabilidad;
- conectar algoritmos clásicos con problemas biomédicos reales;
- avanzar desde fundamentos hacia frontera científica sin salto brusco;
- entender que automatizar una decisión en medicina exige responsabilidad técnica, clínica y epistemológica.

El proyecto también cumple una función formativa interna: el libro se construye para que el autor aprenda primero, y para que ese aprendizaje pueda compartirse después con otros lectores. Por eso la creación del libro debe enseñar durante el proceso, no solo producir páginas publicables.

La función formativa queda elevada a criterio editorial, pero no agota el espíritu público del libro. Una sección correcta pero aislada no basta. Cada entrega debe educar al autor, sostener la promesa pública del libro y avanzar la progresión hacia algoritmos, datos biomédicos, bioinformática, genética computacional, señales, neurología computacional, medicina de precisión y sistemas responsables.

Documento rector agregado:

- `docs/editorial_thesis_scope.md`: tesis editorial, alcance máximo, función formativa y valor reputacional esperado.
- `docs/global_integrity_audit.md`: auditoría GLOBAL inaugural del libro como obra completa en curso.
- `docs/github_actions_maintenance.md`: mantenimiento del workflow de GitHub Pages y regla frente a deprecaciones de runtime.
- `docs/book_architecture_draft.md`: borrador operativo v0.1 de arquitectura editorial, cierre propuesto del Capítulo II y proyección temporal de capítulos.

## Estado actual verificado

### Infraestructura

- Jupyter Book clásico con `jupyter-book<2`.
- Sphinx Book Theme.
- GitHub Pages mediante `.github/workflows/deploy-book.yml`.
- `_config.yml` y `_toc.yml` activos.
- Build local verificado con la CLI interna de Jupyter Book cuando el wrapper `jupyter-book.exe` no entrega diagnóstico.
- Workflow `deploy-book` actualizado para evitar advertencias de runtime Node.js deprecado en GitHub Actions.

### Convención de reporte editorial

El libro distingue entre capítulos editoriales y archivos/secciones numeradas.

- Para trabajo interno, rutas, commits, `_toc.yml` y publicación, se pueden usar los nombres de archivo numerados: `01-...`, `12-...`, `25-...`.
- Para reportar estado al usuario, no se debe llamar "capítulo 25" a un archivo numerado. Se debe informar como capítulo editorial y sección dentro de ese capítulo.
- Estado vigente: `01-el-lenguaje-de-las-decisiones` es el Capítulo I y contiene 11 secciones; `12-tipos-datos-problemas-biomedicos` es el Capítulo II y quedó cerrado con 18 secciones hasta `30-cierre-dato-biomedico-flujo-verificable.md`.
- Forma preferida de reporte: `Capítulo II, sección 13: APIs mínimas y contratos de entrada/salida`.
- La convención no exige renombrar archivos ni reestructurar producción; solo corrige la forma de comunicación y continuidad editorial.

### Modalidad autónoma opt-in

`MMD-AUTO` es una modalidad explícita de producción editorial autónoma. No está activa por defecto.

Activación:

- el usuario debe pedirla de forma clara, por ejemplo: `MMD-AUTO`, `continúa en modo autónomo`, `cierra el Capítulo II en modo autónomo` o equivalente.

Alcance ordinario permitido cuando está activa:

- redactar la siguiente sección prevista en la arquitectura editorial;
- actualizar `_toc.yml`, glosario, `README.md`, `project_status.md`, `roadmap.md` y documentación de continuidad;
- ejecutar bloques de código, `git diff --check` y build local;
- hacer commit local con mensaje coherente en español;
- si el usuario incluyó publicación en la instrucción de activación, sincronizar al repositorio público, hacer commit público, push y verificar deploy;
- abrir, actualizar y cerrar handoffs.

Pausa obligatoria antes de operar:

- cambiar título, subtítulo, tesis central, estructura mayor o número previsto de secciones de un capítulo;
- abrir un capítulo nuevo sin auditoría GLOBAL cuando la arquitectura la exige;
- incorporar dependencias, frameworks, herramientas externas o recursos visuales nuevos;
- publicar material que pueda exponer contenido interno, privado o no publicable;
- resolver una contradicción entre documentos rectores;
- modificar reglas de operación, publicación o seguridad.

Regla de comunicación:

- en `MMD-AUTO`, no pedir autorización para cada sección ordinaria si ya coincide con la arquitectura aprobada;
- sí informar avances, commits, pushes y deploys con evidencia;
- reportar siempre al usuario con la convención acordada: capítulo editorial + sección interna.

### Contenido publicable existente

- Portada.
- Prefacio.
- Cómo leer este libro.
- Presaberes mínimos.
- Capítulo contenedor: `El lenguaje de las decisiones`.
- Secciones del primer capítulo:
  - `Qué es un algoritmo`.
  - `Pensar en pasos`.
  - `Variables, datos y decisiones`.
  - `Estados, condiciones y umbrales`.
  - `Excepciones, datos faltantes y trazabilidad`.
  - `Condicionales como arquitectura de decisión`.
  - `Bucles como control de procesos`.
  - `Funciones como encapsulamiento de criterio`.
  - `Errores, excepciones y seguridad del cálculo`.
  - `Funciones puras, efectos y coordinación de procesos`.
  - `Pruebas y verificación mínima`.
- Capítulo contenedor de la Unidad II: `Tipos de datos para problemas biomédicos`.
- Primera sección de la Unidad II: `Números, unidades y mediciones`.
- Segunda sección de la Unidad II: `Texto libre, códigos y vocabularios controlados`.
- Tercera sección de la Unidad II: `Booleanos, estados e incertidumbre`.
- Cuarta sección de la Unidad II: `Fechas, tiempos, intervalos y granularidad clínica`.
- Quinta sección de la Unidad II: `Ausencia de datos, valores centinela y marcadores especiales`.
- Sexta sección de la Unidad II: `Listas, diccionarios y registros`.
- Séptima sección de la Unidad II: `Tablas simples, limpieza y validación`.
- Octava sección de la Unidad II: `Esquemas mínimos y validación formal`.
- Novena sección de la Unidad II: `pandas como herramienta tabular controlada`.
- Décima sección de la Unidad II: `Archivos, tablas de trabajo y almacenamiento persistente`.
- Undécima sección de la Unidad II: `Entidades, relaciones y claves`.
- Duodécima sección de la Unidad II: `Restricciones, índices y consultas reproducibles`.
- Decimotercera sección de la Unidad II: `APIs mínimas y contratos de entrada/salida`.
- Decimocuarta sección de la Unidad II: `Análisis reproducibles`.
- Decimoquinta sección de la Unidad II: `Pipelines mínimos`.
- Decimosexta sección de la Unidad II: `Validación por lotes y reportes de calidad`.
- Decimoséptima sección de la Unidad II: `Exportación, auditoría y artefactos compartibles`.
- Decimoctava sección de la Unidad II: `Cierre integrador: del dato biomédico al flujo verificable`.
- Apéndice A: entorno de trabajo.
- Glosario vivo.

### Documentación interna existente

- `docs/index.md`.
- `docs/editorial_audit.md`.
- `docs/legacy_structure_audit.md`.
- `docs/roadmap.md`.
- `docs/project_status.md`.
- `docs/cover_design_brief.md`.
- `docs/clean_code_reference_strategy.md`.
- `docs/github_actions_maintenance.md`.

## Avance porcentual

Estimación vigente:

- Infraestructura editorial: 90%.
- Identidad, tesis y dirección editorial: 85%.
- Primera unidad conceptual: 95%.
- Primer lanzamiento mínimo publicable: 45%.
- Libro completo según la ambición actual: 12-15%.

La cifra global sigue siendo baja porque la obra completa apunta más allá de una introducción a Python: incluye algoritmos clásicos, estructuras de datos, análisis cuantitativo, bioinformática, genética computacional, señales, neurología computacional y frontera médico-computacional.

## Avance reciente

Cambios incorporados en el ciclo actual:

- Auditoría GLOBAL inaugural de la obra: se corrigió el contenedor de Unidad I, se añadió tesis/alcance máximo del libro, se normalizaron límites explícitos en secciones 01-09, se añadió bibliografía al contenedor de Unidad II y salida esperada al apéndice de entorno. La matriz automática quedó sin señales mecánicas pendientes.
- Creación de la sección `Texto libre, códigos y vocabularios controlados`. Eje: diferencia entre `str`, texto libre, texto normalizado, categoría controlada y código; uso pedagógico de `Enum`; patrón mínimo de normalización, mapeo, rechazo explícito y conservación de razón. Ejemplos de código verificados antes del commit.
- Creación de la sección `Booleanos, estados e incertidumbre`. Eje: diferencia entre `bool`, valor de verdad técnico y estado biomédico controlado; conservación de `presente`, `ausente`, `desconocido`, `no_evaluado` y `no_aplica`; prevención de convertir `None` o texto en ausencia clínica. Ejemplos de código verificados antes del commit.

- Creación de la sección `Fechas, tiempos, intervalos y granularidad clínica`. Eje: diferencia entre cadena, fecha, instante, duración, intervalo y ventana; conservación de tipo de evento, precisión, estado y fuente antes de calcular; prevención de precisión falsa, intervalos cerrados artificialmente y duraciones sin unidad.
- Actualización de `_toc.yml` y del glosario con `Código`, `Normalización`, `Texto libre` y `Vocabulario controlado`.

- Creación de la sección `Ausencia de datos, valores centinela y marcadores especiales`. Eje: diferencia entre ausencia técnica, ausencia de dominio, valor centinela, marcador especial y estado explícito; prevención de promediar centinelas, convertir cero real en faltante o perder denominadores.
- Actualización de `_toc.yml` y del glosario con `Dato ausente`, `Marcador especial` y `Denominador`.

- Creación de la sección `Listas, diccionarios y registros`. Eje: diferencia entre colección ordenada, mapa clave-valor y observación con campos; prevención de usar posiciones como significado biomédico; validación de registros antes de agrupar; separación entre válidos y rechazados.
- Actualización de `_toc.yml` y del glosario con `Campo`, `Clave`, `Colección`, `Lista` y `Registro`.

- Creación de la sección `Tablas simples, limpieza y validación`. Eje: diferencia entre tabla visible y tabla validada; lectura CSV como adquisición, no interpretación; limpieza de valores crudos, validación de filas, separación de válidos/rechazados y conservación de conteos por estado.
- Actualización de `_toc.yml` y del glosario con `Columna`, `Esquema`, `Fila`, `Limpieza de datos` y `Tabla`.
- Creación de la sección `Esquemas mínimos y validación formal`. Eje: diferencia entre validación dispersa con condiciones sueltas y contrato explícito versionado; esquema como dato; validador genérico pequeño; uso pedagógico de `dataclass`; separación `esquema -> validación -> cálculo`.
- Actualización de `_toc.yml` y del glosario con `Esquema mínimo`, `Validación formal` y `Versión de esquema`.

- Creación de la sección `pandas como herramienta tabular controlada`. Eje: introducción de `Series`, `DataFrame`, selección con `.loc`, lectura CSV, máscaras booleanas, normalización mínima de unidades, conteo con denominadores explícitos y uso de `pandas` como mesa de trabajo, no como sustituto de validación o base de datos.
- Actualización de `requirements.txt` con `pandas>=3.0,<4`.
- Actualización de `_toc.yml` y del glosario con `DataFrame`, `Serie` y `Máscara booleana`.

- Creación de la sección `Archivos, tablas de trabajo y almacenamiento persistente`. Eje: separación entre archivo de entrada, `DataFrame` de trabajo y SQLite como persistencia mínima; separación aceptados/rechazados; inserciones parametrizadas; recuperación con `pandas.read_sql`; límites de `to_sql`.
- Actualización de `_toc.yml` y del glosario con `Almacenamiento persistente`, `SQLite` y `Tabla de trabajo`.
- Creación de la sección `Entidades, relaciones y claves`. Eje: transición desde tablas planas hacia diseño relacional mínimo; separación entre pacientes, mediciones y exposiciones; claves primarias, claves foráneas, relaciones uno a muchos y muchos a muchos; pruebas mínimas de identidad única y rechazo de mediciones huérfanas.
- Actualización de `_toc.yml` y del glosario con `Entidad`, `Relación`, `Clave primaria` y `Clave foránea`.
- Creación de la sección `Restricciones, índices y consultas reproducibles`. Eje: uso de `PRIMARY KEY`, `NOT NULL`, `CHECK`, `FOREIGN KEY`, índices por paciente/fecha, consultas parametrizadas y vistas como forma de conservar preguntas analíticas.
- Actualización de `_toc.yml` y del glosario con `Restricción`, `Índice` y `Consulta reproducible`.
- Creación de la sección `APIs mínimas y contratos de entrada/salida`. Eje: API como frontera contractual; validación de solicitudes antes de operar; separación entre endpoint, dominio y persistencia; respuestas estructuradas con estado, errores y versión de contrato.
- Actualización de `_toc.yml` y del glosario con `API`, `Contrato de entrada` y `Contrato de salida`.
- Creación de la sección `Análisis reproducibles`. Eje: diferencia entre resultado aislado y ejecución reconstruible; especificación de análisis, versión de datos, parámetros serializados, denominador, resultado derivado y comparación de ejecuciones.
- Actualización de `_toc.yml` y del glosario con `Análisis reproducible`, `Ejecución de análisis` y `Resultado derivado`.
- Creación de la sección `Pipelines mínimos`. Eje: encadenamiento explícito de carga, normalización, validación, análisis y reporte; conservación de artefactos intermedios, conteos, rechazos y denominadores.
- Actualización de `_toc.yml` y del glosario con `Artefacto`, `Etapa` y `Pipeline`.
- Creación de la sección `Validación por lotes y reportes de calidad`. Eje: evaluación agregada de registros biomédicos; separación de errores y advertencias; conteos por razón, completitud por campo, proporción válida y estado operativo del lote.
- Actualización de `_toc.yml` y del glosario con `Calidad de datos`, `Lote` y `Reporte de calidad`.
- Creación de la sección `Exportación, auditoría y artefactos compartibles`. Eje: separación entre serialización y contrato de exportación; CSV para tablas, JSON para reportes, SHA-256 para integridad, manifiesto verificable, respaldo consistente de SQLite y distinción entre compartible y publicable.
- Actualización de `_toc.yml` y del glosario con `Artefacto compartible`, `Auditoría de artefactos`, `Huella de contenido` y `Manifiesto de exportación`.
- Creación de la sección `Cierre integrador: del dato biomédico al flujo verificable`. Eje: coordinación explícita de representación, normalización, validación, trazabilidad, agregación, reporte de calidad y exportación sin ocultar rechazos ni denominadores.
- Cierre formal del Capítulo II con 18 secciones y ampliación del glosario con `Flujo verificable`.

- Apertura de la Unidad II con el capítulo contenedor `Tipos de datos para problemas biomédicos`. Eje: los tipos de datos como promesas operacionales y decisiones de representación, no como lista sintáctica de Python.
- Creación de la sección `Números, unidades y mediciones`. Eje: una medición biomédica como valor numérico acompañado por unidad, regla de validez y significado de dominio; separación entre valor imposible, rango de referencia, alerta y dato faltante; patrón mínimo `tipo + validador`; ejemplo pedagógico de dosis por peso. Ejemplos de código verificados antes del commit.
- Actualización de `_toc.yml` y del glosario con `Medición`, `Tipo de dato`, `Unidad`, `Validador` y `Valor centinela`.

- Creación de la sección `Pruebas y verificación mínima` (sec. 11 dentro del cap. 01), con la que se cierra la Unidad I. Eje: la prueba como declaración ejecutable de una propiedad; patrón de cuatro propiedades (caso documentado, dato faltante, frontera, reproducibilidad); propiedad de monotonicidad de colas sobre la regla de alerta de laboratorio; analogía explícita con la interpretación clínica de pruebas diagnósticas negativas. Ejemplos verificados por ejecución antes del cierre.
- Actualización de `_toc.yml` y del glosario con `Prueba (test)`, `Propiedad (de una prueba)` y `Reproducibilidad`.

- Creación de la sección `Funciones puras, efectos y coordinación de procesos` (sec. 10 dentro del cap. 01), con foco en tres roles de funciones, el patrón de extracción `pura → coordinación → ejecutor delgado`, revisión de mediciones de laboratorio como ejemplo biomédico y prueba mínima sin simular el mundo. Ejemplos de código verificados por ejecución antes del cierre.
- Actualización de `_toc.yml`, del glosario con `Función pura`, `Efecto secundario` y `Coordinación de procesos`, y del `Siguiente paso` de la sec. 09 para encadenar la secuencia 09 → 10 → 11.

- Corrección pedagógica de los ejemplos de código existentes:
  - comentarios internos;
  - salidas esperadas;
  - aclaración cuando un fragmento no imprime salida.
- Creación de la sección `Excepciones, datos faltantes y trazabilidad`.
- Cierre conceptual de la primera unidad.
- Actualización del glosario con:
  - `Estado`;
  - `Regla de decisión`;
  - `Transición`;
  - `Excepción`;
  - `Versión de regla`.
- Documentación de propuesta de rediseño profesional de portada.
- Implementación inicial del rediseño de portada con hero editorial, mapa conceptual, progresión y rutas de lectura.
- Inicio de la línea transversal `CODE CLEAN` en una sección nueva sobre condicionales: versión frágil, crítica técnica, versión mejorada, prueba mínima y lección transferible.
- Creación de la sección `Bucles como control de procesos`, con foco en recorrido, acumuladores, condición de parada, trazabilidad y diferencias operativas entre `for` y `while`.
- Actualización de `_toc.yml` y del glosario con `Acumulador`, `Bucle` e `Iteración`.
- Creación de la sección `Funciones como encapsulamiento de criterio`, con foco en contrato, parámetros, retorno, docstrings, comentarios útiles y pruebas mínimas con `assert`.
- Actualización de `_toc.yml` y del glosario con `Contrato`, `Docstring`, `Parámetro` y `Retorno`.
- Creación de la sección `Errores, excepciones y seguridad del cálculo` (cap. 09 dentro del cap. 01), con foco en excepciones técnicas y de dominio, captura específica, traducción de excepciones, prueba mínima con `assert` y trazabilidad del error.

## Estándar vigente para ejemplos de código

Todo ejemplo de código publicable debe cumplir:

1. Tener comentarios breves que expliquen propósito de entradas, reglas, estados, salidas o trazabilidad.
2. Mostrar salida esperada si imprime algo.
3. Declarar explícitamente cuando no produce salida visible.
4. No presentar ejemplos biomédicos pedagógicos como escalas clínicas validadas.
5. Separar dato, interpretación, decisión y razón cuando el ejemplo trate riesgo o clasificación.
6. Preferir nombres semánticos sobre abreviaturas si el ejemplo cruza dominios médicos o científicos.

## Línea transversal CODE CLEAN

`CODE CLEAN` queda definido como una línea pedagógica transversal, no como una unidad aislada ni como transferencia textual de `Clean Code`.

Función:
- aprender principios de código limpio mientras se construye el libro;
- enseñar al lector qué está mal, por qué está mal y cómo puede hacerse mejor;
- usar comparaciones entre versiones ingenuas y versiones responsables cuando aporten claridad;
- aplicar inmersión científica y crítica de fuentes en implementaciones relevantes para contrastar fuentes, alternativas y límites;
- registrar los aprendizajes técnicos reutilizables para futuros libros técnicos del autor.

Límite:
- la meta principal del libro no cambia. Sigue siendo `Algoritmos en Python para Medicina y Ciencias de la Vida`.
- código limpio es una línea de aprendizaje simultánea, no el tema central de la obra.

## Qué falta

Prioridad siguiente:

1. Ejecutar la auditoría GLOBAL de cierre del Capítulo II.
2. Verificar el build y la presentación visual de la sección integradora.
3. Resolver el nombre definitivo y la apertura editorial del Capítulo III.

Decisión estructural vigente:

- El Capítulo II quedó cerrado con 18 secciones.
- La auditoría GLOBAL es la compuerta obligatoria entre los capítulos II y III.
- El Capítulo III no se abrirá hasta resolver su nombre y comprobar la continuidad pedagógica.

No hay una nueva sección candidata hasta superar esa compuerta editorial.

## Riesgos activos

- La portada actual es sobria pero demasiado simple para enganchar.
- El libro puede parecer menos ambicioso de lo que realmente es si la primera pantalla no comunica medicina, algoritmos, Python y ciencias de la vida.
- La primera unidad ya tiene buena profundidad; el siguiente riesgo es no convertir esa profundidad en práctica ejecutable suficiente.
- Los artefactos generados y carpetas heredadas deben mantenerse fuera del contenido publicable.

## Punto de reanudación

Retomar por:

1. `chapters/30-cierre-dato-biomedico-flujo-verificable.md` está completo.
2. Ejecutar y registrar la auditoría GLOBAL del Capítulo II.
3. Revisar visualmente el HTML de la nueva sección.
4. Resolver el nombre definitivo del Capítulo III antes de escribir su primera sección.
5. Validar el build local invocando la CLI interna de Jupyter Book: `.\venv\Scripts\python.exe -c "from jupyter_book.cli.main import main; raise SystemExit(main(['build', '.']))"`.
