# Tipos de datos para problemas biomédicos

La primera unidad construyó una idea: programar no es escribir instrucciones sueltas, sino formalizar decisiones. Un algoritmo responsable necesita entradas, reglas, excepciones, funciones, coordinación y pruebas. Pero antes de decidir algo con un dato hay una pregunta más básica: ¿qué promete ese dato?

Un peso no es simplemente un número. Una fecha no es simplemente texto. Un resultado ausente no es cero. Una categoría diagnóstica no es una frase decorativa. Cada tipo de dato conserva unas operaciones posibles y prohíbe otras. Cuando esa promesa se rompe, el error parece pequeño: una comparación funciona, una suma ejecuta, una cadena se guarda. El daño aparece después, cuando el programa toma una decisión clínicamente absurda con una representación que nunca debió aceptar.

Esta unidad estudia los tipos de datos como decisiones de representación. Python ofrece números, textos, booleanos, fechas, listas, diccionarios y estructuras más complejas. El libro no los tratará como una lista de métodos para memorizar, sino como respuestas a preguntas biomédicas concretas:

- qué tipo conserva una medición sin perder su unidad;
- cómo distinguir ausencia de normalidad;
- cuándo una categoría debe ser texto y cuándo debe ser vocabulario cerrado;
- cómo representar tiempo clínico, intervalos y seguimiento;
- cuándo una lista alcanza y cuándo se necesita una tabla;
- cómo evitar que una estructura flexible oculte errores de dominio.

## La tesis de la unidad

<div class="definition-block">
<strong>Tesis de trabajo.</strong><br />
Un tipo de dato es una promesa operacional: define qué representa un valor, qué operaciones tienen sentido sobre él, qué errores debe rechazar y qué información debe conservar para que una decisión posterior sea trazable.
</div>

Esta definición desplaza el foco. La pregunta no será "¿qué tipo de Python uso?", sino:

```text
¿Qué representación protege mejor el significado biomédico del fenómeno?
```

El tipo equivocado no siempre falla rápido. A veces hace algo peor: permite que el programa siga. Un resultado de potasio guardado como texto se puede imprimir, concatenar, ordenar alfabéticamente y exportar. Todo eso puede funcionar mientras se vuelve imposible comparar con seguridad si `6.2` es mayor que `5.1`. Un dato faltante representado como `0` permite sumar y promediar, pero convierte ausencia en fisiología.

## De la sintaxis al contrato

Python permite descubrir el tipo de un valor con `type()`. Esa operación sirve, pero no basta.

```python
valor = 6.2
print(type(valor))
```

Salida esperada:

```text
<class 'float'>
```

Saber que `6.2` es un `float` informa cómo lo manipula Python. No informa qué mide, en qué unidad, de qué paciente proviene, si fue validado, si está dentro de rango o si es comparable con otro valor. En dominios biomédicos, el tipo técnico es el piso; el contrato de dominio es el edificio.

Por eso esta unidad irá de lo simple a lo responsable:

1. **Números, unidades y mediciones.** Un número sin unidad puede ser ejecutable y aun así ser ambiguo.
2. **Texto y vocabularios.** No todo texto libre debe circular como texto libre.
3. **Booleanos y estados.** Verdadero/falso rara vez alcanza para describir incertidumbre clínica.
4. **Fechas, tiempos e intervalos.** El tiempo clínico tiene granularidad, zona, duración y orden.
5. **Ausencia y datos faltantes.** `None`, centinelas y estructuras explícitas.
6. **Listas, diccionarios y registros.** Cómo agrupar observaciones sin perder significado.
7. **Tablas, esquemas y validación formal.** Cómo pasar de una forma visible a un contrato comprobable.
8. **Herramientas tabulares y persistencia.** Cómo trabajar, guardar y recuperar sin confundir capas.
9. **Entidades, relaciones, restricciones y consultas.** Cómo proteger identidad e invariantes.
10. **APIs y análisis reproducibles.** Cómo declarar fronteras, parámetros y denominadores.
11. **Pipelines y calidad por lotes.** Cómo conservar etapas, rechazos y estado operativo.
12. **Exportación y auditoría.** Cómo producir artefactos compartibles sin confundirlos con datos públicos.
13. **Flujo verificable.** Cómo integrar el recorrido completo y declarar sus límites.

## Mapa de decisión

| Pregunta biomédica | Representación inicial | Riesgo si se elige mal |
|---|---|---|
| ¿Cuánto mide algo? | número + unidad explícita | comparar valores incompatibles |
| ¿Qué categoría tiene? | vocabulario cerrado | aceptar sinónimos, errores o ambigüedad |
| ¿Está presente? | estado explícito | confundir ausencia con normalidad |
| ¿Cuándo ocurrió? | fecha/tiempo | ordenar mal eventos o mezclar granularidades |
| ¿Qué observaciones pertenecen juntas? | registro/diccionario | separar dato, contexto y razón |
| ¿Qué secuencia se recorre? | lista | perder índice, orden o repetición |
| ¿Qué regla valida el valor? | tipo + validador | aceptar valores imposibles con sintaxis correcta |

El mapa no es definitivo. Es una brújula para no empezar por la sintaxis. Un dato biomédico entra a un programa con historia: alguien lo midió, lo registró, lo transformó, lo omitió o lo codificó. El tipo debe conservar la parte de esa historia que será necesaria para decidir.

## Regla de lectura

En esta unidad los ejemplos seguirán siendo miniaturas pedagógicas. No son escalas clínicas validadas ni motores asistenciales. Su función es entrenar una sensibilidad técnica: aprender a ver cuándo un dato está mal representado incluso si Python lo acepta.

El objetivo no es usar tipos más complicados. Es usar el tipo mínimo que preserve la promesa del dato.

## Cierre de la unidad

El recorrido comienza con el caso más común —un número que necesita unidad y validación— y termina con una capacidad integrada: transformar un lote crudo en un flujo que conserva contratos, rechazos, denominadores, calidad y artefactos auditables.

Esa capacidad prepara el Capítulo III. Los algoritmos de búsqueda, ordenamiento, conteo, hashing y optimización solo pueden ser defendibles si operan sobre representaciones que no han perdido identidad, unidad, ausencia ni procedencia.

## Bibliografía y fuentes

- Beaulieu-Jones, B. K., et al. (2018). Examining the use of real-world evidence in the regulatory process. *Clinical Pharmacology & Therapeutics, 104*(5), 843-852. <https://doi.org/10.1002/cpt.1226>
- Hripcsak, G., & Albers, D. J. (2013). Next-generation phenotyping of electronic health records. *Journal of the American Medical Informatics Association, 20*(1), 117-121. <https://doi.org/10.1136/amiajnl-2012-001145>
- Martin Fowler. (2018). *Refactoring* (2nd ed.). Addison-Wesley.
- Wilkinson, M. D., et al. (2016). The FAIR Guiding Principles for scientific data management and stewardship. *Scientific Data, 3*, 160018. <https://doi.org/10.1038/sdata.2016.18>
