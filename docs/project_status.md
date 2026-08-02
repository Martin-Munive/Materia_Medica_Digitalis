# Estado del proyecto

## Propósito

Este documento resume la definición actual de `Materia Médica Digitalis`, el avance real, los pendientes y el porcentaje estimado de progreso para retomar el trabajo sin depender del contexto conversacional.

## Definición del proyecto

`Materia Médica Digitalis: Algoritmos en Python para Medicina y Ciencias de la Vida` es un libro web técnico-científico construido con Jupyter Book y publicado mediante GitHub Pages.

El proyecto no es un curso rápido de sintaxis ni una colección de apuntes. Es una arquitectura de aprendizaje para formar pensamiento algorítmico aplicado a medicina, ciencias de la vida, datos biomédicos, estructuras de información, algoritmos clásicos, bioinformática, genética computacional, neurología computacional y sistemas médico-computacionales responsables.

## Propósito editorial

El libro busca que el lector aprenda a:

- traducir fenómenos biomédicos a datos, reglas, estados, condiciones y algoritmos;
- escribir Python como instrumento de pensamiento, no como sintaxis aislada;
- reconocer límites, excepciones, datos faltantes, sesgos y trazabilidad;
- conectar algoritmos clásicos con problemas biomédicos reales;
- avanzar desde fundamentos hacia frontera científica sin salto brusco;
- entender que automatizar una decisión en medicina exige responsabilidad técnica, clínica y epistemológica.

El proyecto también cumple una función formativa interna: el libro se construye para que el autor aprenda primero, y para que ese aprendizaje pueda compartirse después con otros lectores. Por eso la creación del libro debe enseñar durante el proceso, no solo producir páginas publicables.

## Estado actual verificado

### Infraestructura

- Jupyter Book clásico con `jupyter-book<2`.
- Sphinx Book Theme.
- GitHub Pages mediante `.github/workflows/deploy-book.yml`.
- `_config.yml` y `_toc.yml` activos.
- Build local verificado con `.\venv\Scripts\jupyter-book.exe build .`.

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

- Creación de la sección `Texto libre, códigos y vocabularios controlados`. Eje: diferencia entre `str`, texto libre, texto normalizado, categoría controlada y código; uso pedagógico de `Enum`; patrón mínimo de normalización, mapeo, rechazo explícito y conservación de razón. Ejemplos de código verificados antes del commit.
- Actualización de `_toc.yml` y del glosario con `Código`, `Normalización`, `Texto libre` y `Vocabulario controlado`.

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

1. Continuar con la siguiente sección de la Unidad II: booleanos, estados e incertidumbre.
2. Mantener el patrón `tipo + validador` y la línea transversal `CODE CLEAN`.
3. Revisar visualmente la nueva sección HTML si se va a publicar en GitHub Pages en este ciclo.

Siguientes secciones candidatas:

- Texto libre, códigos y vocabularios controlados.
- Booleanos, estados e incertidumbre.
- Fechas, tiempos, intervalos y granularidad clínica.
- Ausencia, centinelas y datos faltantes.

## Riesgos activos

- La portada actual es sobria pero demasiado simple para enganchar.
- El libro puede parecer menos ambicioso de lo que realmente es si la primera pantalla no comunica medicina, algoritmos, Python y ciencias de la vida.
- La primera unidad ya tiene buena profundidad; el siguiente riesgo es no convertir esa profundidad en práctica ejecutable suficiente.
- Los artefactos generados y carpetas heredadas deben mantenerse fuera del contenido publicable.

## Punto de reanudación

Retomar por:

1. `chapters/14-texto-libre-codigos-vocabularios.md`.
2. Continuar con la sección posterior: booleanos, estados e incertidumbre.
3. Mantener la línea `CODE CLEAN`: versión frágil, crítica, versión mejorada, salida esperada y prueba mínima.
4. Build local con `.\venv\Scripts\jupyter-book.exe build .`.
