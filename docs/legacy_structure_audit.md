# Auditoría de estructura heredada

## Propósito

Este documento registra solo la estructura temática heredada del repositorio antes de la reconstrucción editorial. No reutiliza ni valida el contenido antiguo como texto final. La nueva edición del libro se construye con inmersión científica y crítica de fuentes, bibliografía rastreable y una arquitectura editorial nueva.

## Estructura heredada detectada

### 00-Presaberes

- Introducción a pruebas en Python.
- Formateo de cadenas con f-strings.
- Clases y programación orientada a objetos.

### 01-Analisis_de_Complejidad_Big_O

- Notación Big O, Omega y Theta.

### 02-Estructuras_de_Datos_Lineales

- Arrays, listas y tuplas.
- Listas enlazadas.
- Código demostrativo y pruebas asociadas.

## Valor estructural

La estructura antigua aporta una pista útil sobre temas que el libro debe cubrir, pero no define el orden final ni la profundidad final. Debe tratarse como mapa de posibles estaciones, no como índice vigente.

## Temas que conviene vincular en la nueva arquitectura

1. **Pruebas y verificación.** Deben vincularse con seguridad del cálculo, reproducibilidad, software médico responsable y auditoría de decisiones.
2. **F-strings y salida de información.** Deben aparecer como recurso técnico puntual cuando el libro trabaje reportes, trazabilidad y comunicación de resultados.
3. **Programación orientada a objetos.** Debe reservarse para modelos, entidades clínicas, estructuras de datos y sistemas más complejos; no corresponde al arranque conceptual.
4. **Complejidad Big O/Omega/Theta.** Debe convertirse en un capítulo fuerte posterior, conectado con costo computacional, triage, búsqueda, genomas, sistemas hospitalarios y escalabilidad.
5. **Arrays, listas y tuplas.** Deben integrarse en la unidad de estructuras de datos para ciencias de la vida, con ejemplos de signos vitales, tablas, series, matrices e imagen biomédica.
6. **Listas enlazadas.** Deben tratarse como estructura histórica y conceptual para entender punteros, memoria, colas, pilas y flujos; no como prioridad inicial.

## Temas que no deben vincularse como capítulos independientes

- F-strings.
- Presaberes aislados sin problema biomédico.
- Clases y POO mientras no exista una necesidad estructural clara.

## Decisión operativa aceptada

No restaurar carpetas antiguas en la rama activa.

Usar la estructura heredada para enriquecer el índice maestro y decidir destinos temáticos, pero redactar cada sección desde cero con el estándar actual del proyecto.

## Vinculación acordada por destino

| Tema heredado | Destino editorial | Forma de integración |
| --- | --- | --- |
| Presaberes de Python | Front Matter: `Presaberes mínimos` | Guía breve de orientación, no capítulo numerado. |
| Pruebas | Capítulo futuro sobre verificación, seguridad del cálculo y software médico responsable | Redacción nueva con ejemplos biomédicos y pruebas ejecutables. |
| F-strings | Secciones donde aparezcan reportes, trazabilidad y salida de resultados | Recurso técnico puntual, no tema independiente. |
| Programación orientada a objetos | Capítulo futuro sobre modelos, entidades, reglas y sistemas | Solo cuando exista una necesidad estructural clara. |
| Big O, Omega y Theta | Unidad de algoritmos clásicos y costo clínico-operativo | Capítulo fuerte con inmersión científica y crítica de fuentes. |
| Arrays, listas y tuplas | Unidad de estructuras de datos para ciencias de la vida | Secciones sobre representación, signos vitales, tablas, series y matrices. |
| Listas enlazadas | Unidad de estructuras lineales, memoria y flujos | Tema conceptual posterior; no prioridad inicial. |

## Decisión sobre presaberes

Los presaberes no deben convertirse en el capítulo 1. Hacerlo movería el centro del libro hacia un curso introductorio de Python y debilitaría la entrada intelectual de la obra.

La solución adoptada es crear una página de Front Matter llamada **Presaberes mínimos**. Esa página ayuda al lector primerizo a no perderse, pero mantiene el primer capítulo real como **El lenguaje de las decisiones**.

## Punto actual del libro

La reconstrucción está trabajando el primer capítulo real: **El lenguaje de las decisiones**.

Sus secciones actuales son:

1. Qué es un algoritmo.
2. Pensar en pasos.
3. Variables, datos y decisiones.

Esta lista no está cerrada. El capítulo debe crecer si la arquitectura intelectual lo exige. Candidatos razonables:

- estados y transiciones;
- condiciones y umbrales;
- excepciones y datos faltantes;
- trazabilidad de decisiones;
- del razonamiento clínico al flujo computable.
