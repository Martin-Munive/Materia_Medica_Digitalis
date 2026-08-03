# El lenguaje de las decisiones

Un libro sobre algoritmos para medicina y ciencias de la vida no puede comenzar con una lista de instrucciones ni con una definición escolar. Debe comenzar con una idea más exigente: antes de escribir código, el profesional aprende a convertir fenómenos biológicos, clínicos y científicos en decisiones observables, discutibles y ejecutables.

Este primer capítulo no está dedicado a "los algoritmos" como tema aislado. Está dedicado a la formación de una manera de pensar. La pregunta central no es solo qué es un algoritmo, sino qué significa representar una situación real con datos, reglas, estados, excepciones, incertidumbre y consecuencias.

## Mapa del capítulo

1. **Qué es un algoritmo.** La definición común es insuficiente. Un algoritmo será tratado como una estructura de decisión bajo restricciones, no como una receta inocente.
2. **Pensar en pasos.** La secuencia de acciones es solo la superficie; debajo están los estados, las prioridades, las excepciones y la trazabilidad.
3. **Variables, datos y decisiones.** Nombrar una variable es decidir qué parte del mundo merece existir dentro del programa.
4. **Estados, condiciones y umbrales.** Una decisión computable necesita saber en qué estado se encuentra, qué condiciones activan transiciones y qué umbrales justifican una acción.
5. **Excepciones, datos faltantes y trazabilidad.** Un algoritmo serio no solo decide; también reconoce incertidumbre, datos incompletos, límites y razones auditables.
6. **Condicionales como arquitectura de decisión.** Las ramas de un programa se leen como rutas explícitas de interpretación y acción.
7. **Bucles como control de procesos.** Repetir no es hacer lo mismo sin pensar; es recorrer observaciones con condición de parada, acumulación y trazabilidad.
8. **Funciones como encapsulamiento de criterio.** Una función bien escrita guarda una regla de dominio con entradas, contrato, retorno y prueba mínima.
9. **Errores, excepciones y seguridad del cálculo.** La seguridad técnica exige capturar fallas específicas y traducirlas en estados comprensibles.
10. **Funciones puras, efectos y coordinación de procesos.** Separar cálculo, coordinación y efectos vuelve más verificable un procedimiento biomédico.
11. **Pruebas y verificación mínima.** Una prueba ejecutable declara una propiedad que el algoritmo debe conservar.

El objetivo de este capítulo es construir el lenguaje mínimo para todo lo que vendrá después: búsqueda, complejidad, estructuras de datos, grafos, programación dinámica, bioinformática, genética computacional, neurología de sistemas y modelos de soporte de decisión.

Las once secciones forman la primera unidad conceptual completa. A partir de aquí, el libro puede avanzar hacia tipos de datos, estructuras de representación y validadores: el puente entre una decisión pensada y una decisión ejecutable.

## Criterio de lectura

Cada sección debe leerse como una pieza de entrenamiento intelectual. No basta con recordar definiciones. El lector debe poder explicar qué problema resuelve cada concepto, qué simplifica, qué oculta, qué riesgos introduce y cómo podría escalar hacia escenarios biomédicos reales.

## Siguiente paso

La siguiente unidad pregunta qué ocurre antes de aplicar reglas: qué promete cada dato que entra al algoritmo. Números, texto, booleanos, fechas, listas y registros no son solo tipos de Python; son decisiones de representación biomédica.

## Bibliografía y fuentes

- Abelson, H., Sussman, G. J., & Sussman, J. (1996). *Structure and interpretation of computer programs* (2nd ed.). MIT Press.
- Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2022). *Introduction to algorithms* (4th ed.). MIT Press.
- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>
- Wing, J. M. (2006). Computational thinking. *Communications of the ACM, 49*(3), 33-35. <https://doi.org/10.1145/1118178.1118215>
