# Excepciones, datos faltantes y trazabilidad

Un algoritmo serio no solo debe decidir cuando todo está limpio. Debe saber qué hacer cuando el mundo llega incompleto, contradictorio, fuera de rango o fuera del contrato del procedimiento.

En los ejemplos anteriores apareció una idea recurrente: un dato ausente no es un dato normal. Ahora esa idea se vuelve estructura. Las excepciones, los datos faltantes y la trazabilidad no son detalles finales; son parte de la arquitectura de una decisión responsable.

En medicina y ciencias de la vida, muchos errores no nacen de una fórmula compleja, sino de una simplificación silenciosa: asumir que lo no registrado es normal, que una medición dudosa es confiable, que un paciente fuera de población puede procesarse igual, que una salida sin razones es suficiente o que el código correcto equivale a decisión correcta.

Esta sección cierra la primera unidad conceptual del libro: si un algoritmo formaliza decisiones, entonces debe formalizar también sus límites.

## Origen histórico: del error como accidente al error como objeto de diseño

Durante mucho tiempo, los errores en cálculo, medición o registro pudieron verse como fallas externas al procedimiento. La computación obligó a pensarlos de otra manera. Un programa debe responder ante entradas inesperadas, archivos inexistentes, divisiones por cero, tipos incompatibles, valores fuera de rango y estados no previstos.

En software, una excepción es una interrupción controlada del flujo normal. No siempre significa desastre; a veces significa que el programa encontró una condición que exige tratamiento especial. Esa idea es profundamente útil para medicina: no todo caso debe forzarse dentro de la vía estándar.

La seguridad del paciente también transformó la manera de entender el error. Informes sobre diagnóstico y sistemas de salud han insistido en que muchos fallos emergen de procesos, comunicación, organización, tecnología, sesgos cognitivos y datos incompletos. La lección para este libro es clara: un algoritmo biomédico debe diseñarse dentro de un sistema sociotécnico, no como una fórmula aislada.

En ciencia de datos, el dato faltante también dejó de ser una molestia menor. Puede reflejar azar, sesgo de medición, desigualdad de acceso, flujo clínico, gravedad, pérdida de seguimiento o decisiones humanas previas. Eliminarlo sin preguntar por qué puede volver más limpio el archivo y más pobre el conocimiento.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una excepción es una condición que interrumpe o modifica el flujo esperado de un procedimiento; un dato faltante es una ausencia informativa que debe representarse de forma explícita; la trazabilidad es la capacidad de reconstruir cómo se produjo una salida, con qué datos, reglas, versiones, límites y razones.
</div>

La definición tiene tres consecuencias.

**No toda anomalía es un error fatal.** Algunas anomalías deben detener el procedimiento. Otras deben degradar la salida, pedir revisión, marcar incertidumbre o solicitar más datos.

**No todo dato faltante significa lo mismo.** Puede ser no medido, no registrado, no aplicable, perdido, inválido, censurado o desconocido.

**No toda salida es aceptable si no puede explicarse.** Una clasificación sin razón puede acelerar decisiones, pero también puede impedir aprendizaje, auditoría y corrección.

## El dato faltante como estado explícito

Comparemos dos diseños.

Versión ingenua:

```python
# Entrada incompleta: la saturación no fue registrada.
saturacion_oxigeno = None

# Error conceptual: reemplaza ausencia por normalidad.
if saturacion_oxigeno is None:
    saturacion_oxigeno = 98

if saturacion_oxigeno < 90:
    estado = "alto_riesgo"
else:
    estado = "sin_hipoxemia"

print(estado)
```

Salida esperada:

```text
sin_hipoxemia
```

El código ejecuta, pero la decisión es pobre. No había evidencia de saturación normal; el programa la inventó para poder continuar.

Versión responsable:

```python
# Entrada incompleta: la saturación no fue registrada.
saturacion_oxigeno = None

# El dato faltante produce su propio estado.
if saturacion_oxigeno is None:
    estado = "evaluacion_incompleta"
    razon = "saturacion_oxigeno_no_registrada"
elif saturacion_oxigeno < 90:
    estado = "alto_riesgo"
    razon = "hipoxemia"
else:
    estado = "sin_hipoxemia_detectada"
    razon = "saturacion_en_rango_no_critico"

print(estado)
print(razon)
```

Salida esperada:

```text
evaluacion_incompleta
saturacion_oxigeno_no_registrada
```

La segunda versión no sabe más medicina que la primera. Sabe algo más importante para un sistema responsable: sabe reconocer su propia incompletitud.

## Excepciones técnicas y excepciones del dominio

Python puede detectar algunas excepciones técnicas:

```python
# Entrada incompatible con una comparación numérica.
saturacion_oxigeno = None

try:
    hipoxemia = saturacion_oxigeno < 90
except TypeError:
    hipoxemia = None
    estado = "dato_invalido_o_ausente"

print(hipoxemia)
print(estado)
```

Salida esperada:

```text
None
dato_invalido_o_ausente
```

Este ejemplo muestra una excepción técnica: Python no puede comparar `None` con un número.

Pero un algoritmo biomédico necesita reconocer excepciones que el lenguaje no puede detectar por sí solo:

- valor fisiológicamente imposible;
- unidad equivocada;
- población fuera del alcance del modelo;
- dato antiguo;
- medición tomada con técnica dudosa;
- resultado contradictorio con otra variable;
- salida de alto impacto sin evidencia suficiente.

El lenguaje puede proteger contra ciertos errores de ejecución. El diseñador debe proteger contra errores de representación y decisión.

## Trazabilidad: dejar razones

Una salida útil debe poder explicar su origen.

```python
# Entradas con un dato faltante y dos criterios presentes.
presion_sistolica = 88
saturacion_oxigeno = None
frecuencia_respiratoria = 32

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

if razones:
    estado = "alto_riesgo"
elif pendientes:
    estado = "evaluacion_incompleta"
else:
    estado = "sin_criterios_de_alto_riesgo"

registro_decision = {
    "estado": estado,
    "razones": razones,
    "datos_pendientes": pendientes,
    "version_regla": "triage_pedagogico_v0",
}

print(registro_decision)
```

Salida esperada:

```text
{'estado': 'alto_riesgo', 'razones': ['hipotension', 'taquipnea_severa'], 'datos_pendientes': ['saturacion_oxigeno'], 'version_regla': 'triage_pedagogico_v0'}
```

Este registro no convierte el ejemplo en una escala validada. Lo convierte en algo más revisable. Permite responder preguntas básicas:

- qué estado se produjo;
- qué razones lo activaron;
- qué datos faltaron;
- qué versión de la regla se usó.

En sistemas reales, la trazabilidad puede incluir fecha, hora, fuente del dato, usuario, versión del modelo, umbrales, excepciones, advertencias y enlace a documentación.

## Modelos mentales expertos

### 1. La excepción como señal de frontera

Una excepción muestra dónde termina el camino normal. No siempre indica que el sistema esté mal; puede indicar que el caso exige otra ruta.

Ejemplo: un paciente fuera de la población de validación de un score no debe entrar silenciosamente al mismo flujo. El sistema debe marcar límite de aplicabilidad.

### 2. El dato faltante como información estructural

La ausencia puede decir algo sobre el proceso. Un laboratorio faltante puede reflejar que no se solicitó, que no llegó, que el paciente no accedió al servicio o que el sistema no lo integró.

Ejemplo: en investigación clínica, eliminar todos los registros incompletos puede sesgar el análisis si la falta de datos no ocurre al azar.

### 3. La trazabilidad como memoria de responsabilidad

La trazabilidad no es burocracia. Es memoria técnica y ética. Permite revisar decisiones, detectar errores, comparar versiones y aprender.

Ejemplo: si un sistema clasificó alto riesgo, no basta con guardar la etiqueta. Debe guardar las razones.

### 4. La degradación controlada como alternativa a la falsa certeza

Cuando faltan datos, el algoritmo no siempre debe fallar por completo. Puede producir una salida degradada: `evaluacion_incompleta`, `requiere_revision`, `no_calculable`, `fuera_de_contrato`.

Ejemplo: si falta creatinina para ajustar dosis renal, la salida responsable puede ser "no recomendar dosis automática" y pedir medición.

### 5. El error como oportunidad de rediseño

Un error repetido no debe tratarse como accidente aislado. Debe preguntar si falta una validación, un estado, una excepción, una prueba o una explicación.

Ejemplo: si muchos usuarios ingresan saturación como `0.89` en vez de `89`, el problema no es solo del usuario. El sistema necesita validar unidad o aclarar formato.

## Dónde se solapan

Las excepciones revelan límites. Los datos faltantes obligan a modelar estados. La trazabilidad conserva razones. Las razones permiten auditoría. La auditoría muestra si las excepciones fueron suficientes.

El meta-modelo es este:

> Un algoritmo responsable debe poder decidir, detenerse, degradarse y explicar por qué hizo cada una de esas cosas.

## Dónde crean tensión

Hay tensiones reales.

Manejar demasiadas excepciones puede volver el sistema difícil de leer. Manejar pocas puede hacerlo peligroso. Registrar trazabilidad detallada mejora auditoría, pero puede aumentar complejidad, costo y riesgos de privacidad. Degradar una salida protege contra falsas certezas, pero puede frustrar al usuario si ocurre con demasiada frecuencia.

La solución es proporcionalidad: registrar lo que cambia decisiones, seguridad, reproducibilidad o aprendizaje. No todo detalle merece persistir, pero todo elemento que justifica una salida importante debe quedar disponible.

## Argumentos críticos

### Desacuerdo 1: detener el algoritmo contra continuar con advertencia

Pregunta: ¿qué debe hacer un algoritmo cuando falta un dato importante?

Detenerse protege contra decisiones falsas. Continuar con advertencia puede ser útil cuando hay información suficiente para una salida parcial. El conflicto depende del riesgo.

Consenso operativo: si el dato faltante puede cambiar una decisión de alto impacto, el sistema debe detener, degradar o pedir revisión. Si solo limita precisión, puede continuar con advertencia explícita.

### Desacuerdo 2: imputar datos contra conservar ausencia

Pregunta: ¿conviene reemplazar datos faltantes por estimaciones?

La imputación puede ser útil en análisis estadístico cuando se justifica y se evalúa. Pero en decisiones individuales, imputar sin declarar puede inventar certeza.

Consenso operativo: nunca convertir ausencia en normalidad por comodidad. Si se imputa, debe declararse el método, propósito, límites y efecto sobre la decisión.

### Desacuerdo 3: trazabilidad mínima contra trazabilidad extensa

Pregunta: ¿cuánto debe registrar un sistema?

Poca trazabilidad hace difícil auditar. Demasiada puede saturar, exponer datos sensibles o volver inmanejable el sistema.

Consenso operativo: registrar como mínimo entradas relevantes, razones de salida, datos faltantes, versión de regla/modelo y excepciones que modificaron el flujo.

## Puente hacia la frontera

Las excepciones, los datos faltantes y la trazabilidad reaparecen en todo el libro.

En análisis de cohortes, la ausencia de seguimiento puede sesgar conclusiones. En señales fisiológicas, los artefactos deben separarse de eventos reales. En bioinformática, lecturas de baja calidad pueden alterar variantes. En modelos fundacionales biomédicos, una respuesta debe distinguir evidencia, incertidumbre, fuente y límite de aplicabilidad.

Cuando el libro avance hacia sistemas complejos, la pregunta será:

> ¿Qué hace el algoritmo cuando no puede confiar plenamente en sus entradas, su modelo o su contexto?

## Límites de esta miniatura

La trazabilidad presentada aquí es mínima. Un sistema real requiere gobierno de privacidad, seguridad, auditoría institucional, control de versiones, gestión de acceso y validación del flujo clínico. La sección entrena el principio: una salida importante debe conservar razones y límites.

## Evaluar si entendiste

Estas preguntas buscan criterio, no memoria. Están ordenadas de mayor a menor dificultad.

1. ¿Cuándo un algoritmo debe detenerse ante un dato faltante y cuándo puede continuar con advertencia?
2. ¿Qué diferencia hay entre excepción técnica y excepción del dominio?
3. ¿Qué daño puede producir reemplazar un dato faltante por un valor normal?
4. ¿Qué campos mínimos incluirías en un registro de trazabilidad para una decisión de riesgo?
5. ¿Por qué la trazabilidad es una forma de seguridad y no solo documentación?
6. ¿Cómo distinguirías un dato no medido, no aplicable, inválido y perdido?
7. ¿Qué debería hacer un algoritmo si detecta que un paciente está fuera de la población para la cual fue diseñado?
8. ¿Cuándo una salida `evaluacion_incompleta` es mejor que una clasificación aparentemente útil?
9. ¿Qué relación hay entre datos faltantes y sesgo?
10. ¿Cómo sabrías si estás registrando trazabilidad suficiente o solo acumulando ruido?

## Vacíos de comprensión que debes vigilar

1. Creer que ejecutar sin error equivale a decidir bien. El código puede correr y aun así representar mal el problema.
2. Creer que el dato faltante es un obstáculo técnico menor. Puede ser una señal de proceso, sesgo o riesgo.
3. Creer que explicar una salida es opcional. En dominios de consecuencias, una decisión sin razones es una deuda de seguridad.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma un ejemplo anterior y agrega estados `dato_pendiente`, `dato_invalido`, `fuera_de_contrato` y `requiere_revision`.
2. **Segunda hora:** escribe un `registro_decision` con entradas, razones, pendientes, versión de regla y advertencias.
3. **Tercera hora:** identifica tres situaciones donde continuar sería peligroso y tres donde continuar con advertencia sería razonable.

## Bibliografía y fuentes

- Little, R. J. A., & Rubin, D. B. (2019). *Statistical analysis with missing data* (3rd ed.). Wiley. <https://doi.org/10.1002/9781119482260>
- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>
- Obermeyer, Z., Powers, B., Vogeli, C., & Mullainathan, S. (2019). Dissecting racial bias in an algorithm used to manage the health of populations. *Science, 366*(6464), 447-453. <https://doi.org/10.1126/science.aax2342>
- Sittig, D. F., & Singh, H. (2010). A new sociotechnical model for studying health information technology in complex adaptive healthcare systems. *Quality & Safety in Health Care, 19*(Suppl. 3), i68-i74. <https://doi.org/10.1136/qshc.2010.042085>
- Wilkinson, M. D., et al. (2016). The FAIR Guiding Principles for scientific data management and stewardship. *Scientific Data, 3*, 160018. <https://doi.org/10.1038/sdata.2016.18>

## Siguiente paso

Con esta sección queda completo el lenguaje inicial de las decisiones: algoritmo, pasos, variables, estados, condiciones, umbrales, excepciones, datos faltantes y trazabilidad.

El siguiente movimiento natural es entrar en Python como arquitectura de decisión: condicionales, bucles, funciones y verificación. Allí el libro dejará de presentar solo conceptos y empezará a construir procedimientos cada vez más ejecutables.
