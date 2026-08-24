# Algoritmos fundamentales con lectura biomédica

El capítulo anterior construyó una disciplina de representación. Una medición necesitó unidad; una ausencia necesitó estado; una tabla necesitó esquema; una entidad necesitó identidad; un análisis necesitó denominador; un artefacto necesitó trazabilidad. Ahora aparece una pregunta distinta:

```text
Una vez que los datos tienen una forma defendible, ¿qué procedimiento conviene usar para operar sobre ellos?
```

No basta con que el procedimiento produzca una respuesta. Dos algoritmos pueden devolver el mismo resultado y, sin embargo, diferir en tiempo, memoria, precondiciones, facilidad de verificación y consecuencias cuando los datos crecen. Buscar un evento entre diez registros permite casi cualquier estrategia. Buscarlo millones de veces en una base longitudinal obliga a pensar de otra manera.

Este capítulo estudia algoritmos fundamentales como modelos mentales para organizar trabajo computacional. La meta no es memorizar nombres ni reproducir implementaciones canónicas de forma mecánica. La meta es aprender a reconocer la estructura de un problema y elegir un procedimiento compatible con sus datos, su escala y su riesgo.

## Tesis del capítulo

<div class="definition-block">
<strong>Tesis de trabajo.</strong><br />
Elegir un algoritmo es declarar qué operación se necesita, qué precondiciones se aceptan, qué costo puede pagarse y qué errores deben permanecer visibles.
</div>

La palabra *fundamental* tiene aquí un sentido deliberado. Incluye algoritmos clásicos, pero no se limita a su historia. Búsqueda, conteo, ordenamiento, hashing, recursión, estrategias voraces, programación dinámica y grafos siguen siendo fundamentales porque reaparecen dentro de bases de datos, bioinformática, sistemas clínicos, procesamiento de señales y modelos contemporáneos.

## De la respuesta al costo

En los primeros capítulos preguntábamos si una función era correcta para una entrada. En este capítulo agregaremos preguntas nuevas:

- ¿cuántos elementos necesita inspeccionar?;
- ¿qué ocurre cuando el conjunto crece?;
- ¿necesita que los datos estén ordenados?;
- ¿consume memoria adicional?;
- ¿modifica la colección original?;
- ¿cómo se comporta ante duplicados, ausencias o empates?;
- ¿qué propiedad permite comprobar que la respuesta es correcta?;
- ¿qué costo operativo o biomédico tiene equivocarse?;

La eficiencia no reemplaza la validez. Un algoritmo rápido sobre datos mal representados procesa el error con mayor velocidad. Tampoco toda mejora de velocidad es relevante: reducir milisegundos en un lote pequeño puede no justificar una estructura más compleja. La escala y el uso deciden qué costo importa.

## Laboratorio acumulativo

El capítulo construirá un **registro biomédico sintético**. Su primer escenario será una cohorte oncológica educativa con eventos de tratamiento, laboratorio y seguimiento. No contiene datos reales, no representa una historia clínica y no implementa reglas asistenciales.

El mismo artefacto crecerá con cada sección:

1. localizar eventos mediante búsqueda lineal;
2. aprovechar orden para realizar búsqueda binaria;
3. contar frecuencias sin perder denominadores;
4. construir índices por identificador;
5. ordenar y priorizar con criterios explícitos;
6. medir costo temporal y espacial;
7. descomponer estructuras recursivas;
8. estudiar decisiones locales con estrategias voraces;
9. reutilizar subproblemas con programación dinámica;
10. representar relaciones mediante grafos;
11. comparar algoritmos según representación, escala y riesgo.

El escenario oncológico da continuidad al aprendizaje científico del autor, pero no encierra el capítulo en una sola enfermedad. Cuando un concepto se comprenda mejor con secuencias, redes biológicas, imágenes o señales, el ejemplo cambiará de dominio sin abandonar el mismo criterio algorítmico.

## Arquitectura del recorrido

| Sección | Pregunta central | Capacidad esperada |
|---|---|---|
| De los datos a los algoritmos | ¿Qué cambia cuando ya no basta representar? | Formular un problema computacional con contrato y costo. |
| Búsqueda lineal | ¿Qué significa inspeccionar uno por uno? | Buscar y hacer visible el trabajo realizado. |
| Búsqueda binaria | ¿Qué permite ganar el orden? | Usar una precondición sin ocultarla. |
| Conteo y frecuencias | ¿Cómo resumir sin perder población? | Construir acumuladores robustos. |
| Hashing e identificadores | ¿Cuándo conviene pagar por un índice? | Recuperar por clave y razonar sobre colisiones. |
| Ordenamiento y priorización | ¿Qué significa poner primero? | Separar orden técnico de prioridad de dominio. |
| Complejidad y costo | ¿Cómo crece el trabajo? | Comparar alternativas más allá del cronómetro. |
| Recursión | ¿Cuándo un problema contiene versiones de sí mismo? | Definir caso base y progreso verificable. |
| Estrategias voraces | ¿Cuándo sirve una buena decisión local? | Distinguir heurística, optimalidad y límite. |
| Programación dinámica | ¿Qué subproblemas se repiten? | Conservar resultados parciales con criterio. |
| Grafos introductorios | ¿Qué cambia cuando importan las relaciones? | Recorrer redes clínicas y biológicas. |
| Cierre integrador | ¿Qué algoritmo corresponde a este problema? | Justificar una elección por datos, costo y riesgo. |

## Regla de lectura

Cada sección seguirá una secuencia estable:

```text
problema -> representación -> algoritmo -> propiedad -> costo -> límite
```

El código seguirá siendo pequeño y ejecutable. Cuando Python ya ofrezca una implementación madura, primero estudiaremos la idea y luego usaremos la herramienta estándar. Implementar un algoritmo con fines pedagógicos no implica reemplazar bibliotecas probadas en software real.

## Límites

- Los datos del laboratorio son sintéticos.
- Las prioridades usadas en ejemplos no son escalas clínicas validadas.
- La complejidad computacional no mide por sí sola beneficio, seguridad ni utilidad clínica.
- Un algoritmo correcto necesita validación adicional antes de cualquier uso biomédico real.
- El capítulo enseña selección y razonamiento algorítmico; no construye un sistema asistencial.

## Bibliografía y fuentes

- Black, P. E. (ed.). (s. f.). *Dictionary of Algorithms and Data Structures*. National Institute of Standards and Technology. <https://www.nist.gov/dads/>.
- Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2022). *Introduction to Algorithms* (4th ed.). MIT Press.
- Knuth, D. E. (1997-1998). *The Art of Computer Programming* (2nd ed., vols. 1-3). Addison-Wesley.
- Python Software Foundation. (s. f.). *The Python Standard Library*. <https://docs.python.org/3/library/>.
