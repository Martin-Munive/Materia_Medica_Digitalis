# Estados, condiciones y umbrales

Una variable permite que una parte del mundo entre al programa. Pero una variable aislada todavía no decide nada. Para que un procedimiento actúe, necesita reconocer estados, evaluar condiciones y aplicar umbrales.

Esta sección introduce una idea central para todo el libro: un algoritmo no solo transforma datos; también gobierna cambios de estado. Un paciente puede pasar de `no_evaluado` a `evaluacion_incompleta`, de `evaluacion_incompleta` a `alto_riesgo`, de `alto_riesgo` a `requiere_intervencion`, o de `estable` a `reevaluar`. Cada transición exige una condición defendible.

En medicina y ciencias de la vida, esa condición rara vez es inocente. Puede depender de un número, una tendencia, una ausencia, una medición dudosa, una escala validada, un contexto clínico o una decisión institucional. Por eso los umbrales no deben tratarse como líneas mágicas. Son instrumentos: útiles, necesarios y peligrosos si se usan sin contexto.

## Origen histórico: decidir por cortes

La idea de separar estados mediante condiciones aparece en muchos territorios antes de la computación moderna. En lógica, una proposición puede ser verdadera o falsa. En matemáticas, una función puede definirse por partes: una regla se aplica bajo cierta condición y otra regla bajo otra. En medicina, clasificar por signos, síntomas o valores de laboratorio ha sido una forma histórica de ordenar incertidumbre.

La computación convirtió esa intuición en estructura ejecutable. Una instrucción condicional permite que una máquina tome caminos diferentes según el valor de una expresión. Pero el problema profundo no es escribir `if`. El problema profundo es decidir qué condición merece gobernar una transición.

En medicina clínica, el enfoque de umbrales recibió una formulación influyente en el trabajo de Pauker y Kassirer sobre toma de decisiones diagnósticas. La idea general es que una probabilidad estimada puede ubicarse por debajo de un umbral de prueba, entre un umbral de prueba y tratamiento, o por encima de un umbral terapéutico. Aunque este libro no convierte ese modelo en una receta, sí toma una lección importante: decidir no depende solo del dato, sino del costo de actuar, no actuar o buscar más información.

Los sistemas de triage enseñan otra versión de la misma idea. El objetivo no es describir toda la complejidad del paciente, sino separar estados operativos bajo presión de tiempo: emergencia, prioridad o no urgente. La condición que activa cada estado debe ser suficientemente clara para actuar rápido, pero suficientemente prudente para no ocultar excepciones.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un estado es una representación discreta de la situación actual de un sistema; una condición es una expresión que permite decidir si debe ocurrir una transición; un umbral es un valor o criterio que separa estados, acciones o interpretaciones.
</div>

La definición tiene tres piezas.

**Estado.** Describe dónde está el sistema en un momento dado: `no_evaluado`, `estable`, `alto_riesgo`, `dato_pendiente`, `error_de_medicion`, `seguimiento`, `emergencia`.

**Condición.** Evalúa si algo es verdadero o falso: `presion_sistolica < 90`, `saturacion_oxigeno is None`, `criterios_emergencia != []`, `tiempo_desde_ultima_medicion > 6`.

**Umbral.** Separa regiones: una presión sistólica menor de cierto valor, una saturación bajo cierto punto, una probabilidad por encima de la cual se trata, un tiempo máximo antes de reevaluar.

Estas piezas forman una transición:

```text
estado actual + condicion verdadera -> nuevo estado
```

En un programa, esto aparece como código. En un sistema médico, aparece como decisión operativa.

## Condiciones como arquitectura de decisión

Observa una condición sencilla:

```python
# Entrada: presión arterial sistólica medida.
presion_sistolica = 88

# Condición: el umbral separa dos estados operativos.
if presion_sistolica < 90:
    estado_hemodinamico = "hipotension"
else:
    estado_hemodinamico = "sin_hipotension_detectada"
```

Salida esperada: no imprime nada. `estado_hemodinamico` queda como `"hipotension"`.

El código es comprensible, pero su aparente claridad oculta varias decisiones.

Primero, eligió una variable. Segundo, asumió una unidad. Tercero, aplicó un umbral. Cuarto, redujo un fenómeno continuo a dos estados. Quinto, omitió contexto: edad, medición previa, técnica, tendencia, síntomas, tratamiento, comorbilidades y objetivo del procedimiento.

La condición funciona, pero no agota el problema. Esa es la madurez que buscamos: saber cuándo una condición es útil sin olvidar lo que deja fuera.

## Estados explícitos contra estados implícitos

Un error frecuente es dejar que los estados existan solo de forma implícita.

Versión pobre:

```python
# Entrada ausente: no hay saturación registrada.
saturacion_oxigeno = None

# Patrón pobre: mezcla ausencia de dato con ausencia de riesgo.
if saturacion_oxigeno and saturacion_oxigeno < 90:
    clasificacion = "alto_riesgo"
else:
    clasificacion = "sin_criterios"
```

Salida esperada: no imprime nada. `clasificacion` queda como `"sin_criterios"`, que es justamente el problema conceptual del ejemplo.

Este patrón es peligroso porque mezcla ausencia de dato con ausencia de riesgo. Si la saturación no está registrada, el sistema no sabe si el paciente está bien. Sabe otra cosa: que falta una medición relevante.

Versión más responsable:

```python
# Entrada ausente: no hay saturación registrada.
saturacion_oxigeno = None

# Patrón responsable: la ausencia tiene su propio estado.
if saturacion_oxigeno is None:
    clasificacion = "evaluacion_incompleta"
elif saturacion_oxigeno < 90:
    clasificacion = "alto_riesgo_por_hipoxemia"
else:
    clasificacion = "sin_hipoxemia_detectada"
```

Salida esperada: no imprime nada. `clasificacion` queda como `"evaluacion_incompleta"`.

La segunda versión no es más larga por capricho. Es más precisa porque separa tres estados:

- dato faltante;
- criterio de riesgo presente;
- criterio de riesgo no detectado.

En programación médica, esa distinción reduce errores silenciosos. En ciencia de datos, evita limpiar la incertidumbre hasta volverla invisible. En sistemas de soporte de decisión, permite auditar por qué el sistema no produjo una recomendación.

## Modelos mentales expertos

### 1. El estado como memoria del procedimiento

Un estado resume dónde está el sistema. No es solo una etiqueta; es memoria operativa. Le dice al procedimiento qué se sabe, qué falta, qué se decidió y qué puede ocurrir después.

Ejemplo: `evaluacion_incompleta` no es un fracaso del algoritmo. Es una salida válida cuando faltan datos necesarios. Un sistema que no puede decir "no sé todavía" empuja al usuario hacia falsas certezas.

### 2. La condición como frontera justificable

Una condición debe poder defenderse. No basta con que sea fácil de escribir. Debe responder por qué ese dato, ese operador y ese punto de corte gobiernan una transición.

Ejemplo: `temperatura_celsius >= 38.0` puede servir como miniatura pedagógica de fiebre, pero no debe confundirse con una evaluación clínica completa. Si el paciente es inmunosuprimido o si la medición es poco confiable, la condición necesita contexto adicional.

### 3. El umbral como simplificación útil, no como verdad natural

Muchos fenómenos biomédicos son continuos. El umbral los corta para decidir. Esa simplificación puede ser necesaria: los sistemas deben actuar. Pero el corte no convierte la realidad en dos mundos naturales.

Ejemplo: una saturación de 89% y una de 90% pueden caer en categorías distintas, aunque biológicamente estén muy cerca. El umbral ayuda a decidir, pero también debe invitar a revisar tendencia, contexto y error de medición.

### 4. La transición como compromiso entre riesgo y costo

Cambiar de estado tiene consecuencias. Clasificar a alguien como alto riesgo consume recursos, activa intervención, genera alarma y modifica prioridades. No clasificarlo puede retrasar una acción necesaria.

Ejemplo: en triage, un umbral sensible puede detectar más pacientes graves, pero también aumentar carga operativa. Un umbral muy específico puede reducir falsas alarmas, pero dejar pasar casos peligrosos.

### 5. La excepción como estado de primera clase

El pensamiento débil trata las excepciones como molestias. El pensamiento experto las modela. Dato faltante, medición inválida, paciente fuera de población, regla no aplicable, señal contaminada o resultado contradictorio deben tener representación explícita.

Ejemplo: `fuera_de_poblacion_del_modelo` puede ser una salida más segura que forzar una predicción en un paciente para quien el algoritmo no fue diseñado.

## Dónde se solapan

Los estados necesitan condiciones para cambiar. Las condiciones suelen usar umbrales. Los umbrales producen transiciones. Las transiciones deben registrar razones. Las razones permiten auditoría. La auditoría revela excepciones.

El meta-modelo es este:

> Un algoritmo responsable no solo calcula salidas; gobierna transiciones entre estados bajo condiciones justificables.

## Dónde crean tensión

Los estados discretos hacen posible decidir, pero pueden empobrecer fenómenos continuos. Las condiciones explícitas facilitan auditoría, pero pueden dejar fuera juicio experto. Los umbrales permiten actuar, pero pueden volverse rígidos. Las excepciones aumentan seguridad, pero también complejidad.

La solución no es abandonar estados, condiciones y umbrales. La solución es diseñarlos con humildad técnica: declarar qué hacen, qué no hacen, qué datos requieren y cuándo deben ceder ante revisión humana.

## Ejemplo: un pequeño sistema de estados

Podemos extender el ejemplo de signos vitales hacia una máquina de estados mínima.

```python
# Entradas: signos vitales disponibles y ausentes.
presion_sistolica = 88
saturacion_oxigeno = None
frecuencia_respiratoria = 32

# Razones explican riesgo; pendientes explican incompletitud.
razones = []
pendientes = []

if presion_sistolica is None:
    pendientes.append("presion_sistolica")
elif presion_sistolica < 90:
    razones.append("hipotension")

if saturacion_oxigeno is None:
    pendientes.append("saturacion_oxigeno")
elif saturacion_oxigeno < 90:
    razones.append("hipoxemia")

if frecuencia_respiratoria is None:
    pendientes.append("frecuencia_respiratoria")
elif frecuencia_respiratoria > 30:
    razones.append("taquipnea_severa")

# Prioridad de salida: riesgo presente antes que incompletitud.
if razones:
    estado = "alto_riesgo"
elif pendientes:
    estado = "evaluacion_incompleta"
else:
    estado = "sin_criterios_de_alto_riesgo"

print("estado:", estado)
print("razones:", razones)
print("pendientes:", pendientes)
```

Salida esperada:

```text
estado: alto_riesgo
razones: ['hipotension', 'taquipnea_severa']
pendientes: ['saturacion_oxigeno']
```

Este código no es una escala clínica. Es una miniatura conceptual. Su valor está en la estructura:

- separa razones de datos pendientes;
- evita tratar ausencia como normalidad;
- conserva trazabilidad;
- produce un estado operativo;
- permite ampliar la regla sin destruirla.

La idea será reutilizada muchas veces. En búsqueda, un algoritmo puede estar en estado `encontrado` o `no_encontrado`. En programación dinámica, una celda puede estar `calculada` o `pendiente`. En bioinformática, una posición puede estar alineada, penalizada por brecha o marcada como variante. En sistemas clínicos, un paciente puede estar pendiente de evaluación, en seguimiento, priorizado o fuera de criterio.

## Argumentos críticos

### Desacuerdo 1: umbral fijo contra juicio contextual

Pregunta: ¿debe una regla usar un punto de corte fijo o adaptarse al contexto?

El argumento por el umbral fijo es fuerte: simplifica, permite reproducibilidad y facilita auditoría. El argumento contextual también es fuerte: edad, comorbilidad, método de medición, trayectoria temporal y objetivo clínico pueden cambiar el significado del mismo número.

Importancia práctica: si todo se adapta, el sistema puede perder consistencia. Si nada se adapta, puede volverse ciego ante casos reales.

Consenso operativo: usar umbrales explícitos cuando sean necesarios, pero registrar contexto, población, límites y condiciones de no aplicabilidad.

### Desacuerdo 2: más estados contra menos estados

Pregunta: ¿conviene modelar muchos estados para ser preciso o pocos estados para ser usable?

Pocos estados reducen carga cognitiva y facilitan implementación. Muchos estados preservan matices importantes, especialmente cuando hay datos faltantes, excepciones o distintos niveles de riesgo.

Importancia práctica: un sistema con demasiados estados puede ser difícil de enseñar y validar. Uno con muy pocos puede confundir situaciones distintas.

Consenso operativo: empezar con los estados que cambian una acción real. Agregar estados nuevos cuando reduzcan error, mejoren auditoría o protejan contra una decisión equivocada.

### Desacuerdo 3: automatizar la transición contra pedir revisión humana

Pregunta: ¿cuándo debe el algoritmo cambiar de estado por sí mismo y cuándo debe pedir confirmación?

La automatización es útil cuando la condición es clara, repetible y de bajo riesgo relativo. La revisión humana es necesaria cuando el dato es dudoso, el paciente está fuera de población, la consecuencia es grave o la regla no captura el contexto.

Importancia práctica: un sistema que automatiza demasiado puede ocultar responsabilidad. Un sistema que pide confirmación para todo puede volverse inútil.

Consenso operativo: automatizar transiciones simples y verificables; escalar transiciones inciertas, de alto impacto o fuera del contrato del algoritmo.

## Puente hacia la frontera

Estados, condiciones y umbrales no pertenecen solo a ejemplos introductorios.

En modelos de enfermedad, un paciente puede moverse entre estados de susceptibilidad, infección, recuperación o recaída. En señales fisiológicas, un algoritmo puede detectar estados de sueño, vigilia, crisis, artefacto o ritmo anormal. En genética computacional, una variante puede pasar por estados de detección, anotación, filtrado, clasificación e interpretación. En sistemas biomédicos computacionales, un caso puede estar dentro o fuera de distribución, con confianza suficiente o con necesidad de revisión.

La pregunta madura no será "¿qué if escribo?". Será:

> ¿Qué estados existen en este problema, qué condiciones autorizan pasar de uno a otro y qué consecuencias tiene equivocarse?

## Límites de esta miniatura

Los umbrales usados en ejemplos pedagógicos no deben leerse como recomendaciones clínicas. Un umbral real depende de población, contexto, método de medición, riesgo aceptable y validación. Aquí importan la forma de representar estados y transiciones, no la autoridad médica del número elegido.

## Evaluar si entendiste

Estas preguntas buscan criterio, no memoria. Están ordenadas de mayor a menor dificultad.

1. ¿Cómo diseñarías estados para distinguir entre paciente estable, paciente no evaluado y paciente con evaluación incompleta?
2. ¿Cuándo un umbral fijo es más seguro que una regla contextual, y cuándo ocurre lo contrario?
3. ¿Qué daño puede causar un sistema que no representa explícitamente el estado `fuera_de_poblacion_del_modelo`?
4. ¿Cómo decidirías si una excepción debe convertirse en estado del algoritmo o en alerta para revisión humana?
5. ¿Por qué una saturación de 89% y una de 90% pueden activar decisiones diferentes aunque estén biológicamente cerca?
6. ¿Qué información mínima debería registrarse cuando un paciente cambia a `alto_riesgo`?
7. ¿Cómo se conecta la idea de estado con una tabla clínica, una señal neurológica y una secuencia genética?
8. ¿Qué diferencia hay entre una condición técnicamente válida y una condición clínicamente defendible?
9. ¿Por qué el dato faltante debe producir una transición distinta a "sin criterios"?
10. ¿Cómo sabrías si estás agregando estados útiles o solo complejidad decorativa?

## Vacíos de comprensión que debes vigilar

1. Confundir umbral con verdad natural. Un punto de corte ayuda a decidir, pero no convierte un fenómeno continuo en una realidad binaria simple.
2. Confundir estado ausente con estado normal. Si el sistema no sabe algo, debe poder decirlo.
3. Confundir condición válida con condición suficiente. Que una expresión pueda evaluarse no significa que represente bien el problema.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** reescribe el ejemplo de signos vitales agregando estados `dato_invalido`, `evaluacion_incompleta`, `alto_riesgo` y `sin_criterios`.
2. **Segunda hora:** dibuja una tabla con tres columnas: estado actual, condición y estado siguiente. Úsala para describir un mini flujo de triage.
3. **Tercera hora:** identifica un umbral biomédico que uses con frecuencia y escribe qué conserva, qué simplifica, qué excepciones tiene y qué daño produciría aplicarlo fuera de contexto.

## Bibliografía y fuentes

- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>
- Pauker, S. G., & Kassirer, J. P. (1980). The threshold approach to clinical decision making. *New England Journal of Medicine, 302*(20), 1109-1117. <https://doi.org/10.1056/NEJM198005153022003>
- Sittig, D. F., & Singh, H. (2010). A new sociotechnical model for studying health information technology in complex adaptive healthcare systems. *Quality & Safety in Health Care, 19*(Suppl. 3), i68-i74. <https://doi.org/10.1136/qshc.2010.042085>
- Stevens, S. S. (1946). On the theory of scales of measurement. *Science, 103*(2684), 677-680. <https://doi.org/10.1126/science.103.2684.677>
- World Health Organization. (2005). *Emergency triage assessment and treatment (ETAT) course*. <https://www.who.int/publications/i/item/9241546875>

## Siguiente paso

Ahora podemos avanzar hacia excepciones, datos faltantes y trazabilidad. Si los estados y condiciones permiten decidir, las excepciones enseñan cuándo una decisión debe detenerse, degradarse o pedir revisión.
