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
- aplicar `INMERSION CIENTIFICA` en implementaciones relevantes para contrastar fuentes, alternativas y límites;
- promover aprendizajes técnicos reutilizables hacia BRAIN mediante `KINTARO` cuando mejoren `S-ESCRIBA`, `S-MITNICK` u otra skill.

Límite:
- la meta principal del libro no cambia. Sigue siendo `Algoritmos en Python para Medicina y Ciencias de la Vida`.
- código limpio es una línea de aprendizaje simultánea, no el tema central de la obra.

## Qué falta

Prioridad siguiente:

1. Revisar visualmente la portada rediseñada en navegador.
2. Ajustar detalles visuales si aparecen problemas de proporción o legibilidad.
3. Continuar la siguiente unidad: Python como arquitectura de decisión, aplicando la línea transversal `CODE CLEAN`.

Siguientes secciones candidatas:

- Funciones puras, efectos y coordinación de procesos.
- Errores, excepciones y seguridad del cálculo.
- Pruebas y verificación mínima.

## Riesgos activos

- La portada actual es sobria pero demasiado simple para enganchar.
- El libro puede parecer menos ambicioso de lo que realmente es si la primera pantalla no comunica medicina, algoritmos, Python y ciencias de la vida.
- La primera unidad ya tiene buena profundidad; el siguiente riesgo es no convertir esa profundidad en práctica ejecutable suficiente.
- Los artefactos generados y carpetas heredadas deben mantenerse fuera del contenido publicable.

## Punto de reanudación

Retomar por:

1. `docs/cover_design_brief.md`.
2. Continuar con la sección posterior a `chapters/08-funciones-encapsulamiento-criterio.md`.
3. Mantener la línea `CODE CLEAN`: versión frágil, crítica, versión mejorada, salida esperada y prueba mínima.
4. Build local con `.\venv\Scripts\jupyter-book.exe build .`.
