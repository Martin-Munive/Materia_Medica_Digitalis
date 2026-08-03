# Condicionales como arquitectura de decisión

Un condicional parece una estructura sencilla: si ocurre algo, hacer una cosa; si no ocurre, hacer otra. Pero en un libro sobre algoritmos para medicina y ciencias de la vida, un condicional no puede tratarse solo como sintaxis. Un `if` es una frontera de decisión.

Cada vez que escribimos un condicional estamos diciendo: bajo estas condiciones, este dato cambia de significado; esta persona entra en otro estado; esta muestra requiere otro flujo; este cálculo ya no debe continuar igual.

Por eso los condicionales son el primer lugar donde el código revela la calidad del pensamiento. Un condicional puede hacer visible una regla responsable o esconder una simplificación peligrosa.

## Origen técnico: bifurcar el flujo

La programación estructurada convirtió la bifurcación en una herramienta central: un programa no solo ejecuta instrucciones en secuencia; puede tomar caminos distintos según el estado de sus datos.

En Python, `if`, `elif` y `else` permiten expresar esa bifurcación de forma legible. La documentación oficial de Python presenta el `if` como una estructura para ejecutar bloques alternativos según condiciones. Esa explicación es correcta, pero para este libro es apenas el punto de partida.

La pregunta importante no es solo:

```text
¿cómo escribo un if?
```

La pregunta madura es:

```text
¿qué condición merece cambiar el camino del algoritmo?
```

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un condicional es una estructura de control que selecciona una rama de ejecución según una condición. En este libro, además, un condicional es una decisión explícita sobre el significado operativo de los datos.
</div>

La definición tiene dos capas.

**Capa técnica.** Python evalúa una expresión. Si es verdadera, ejecuta un bloque; si no, puede probar otra condición o caer en una rama alternativa.

**Capa de dominio.** La condición representa una hipótesis sobre el problema: riesgo, ausencia, error, prioridad, transición, no aplicabilidad o necesidad de revisión.

Un condicional limpio debe permitir leer ambas capas.

## Versión ingenua: el condicional que parece útil

Supongamos que queremos clasificar una medición de saturación de oxígeno.

```python
# Entrada: saturación de oxígeno registrada.
saturacion = None

# Versión frágil: mezcla dato ausente con ausencia de hipoxemia.
if saturacion and saturacion < 90:
    estado = "alto_riesgo"
else:
    estado = "sin_riesgo"

print(estado)
```

Salida esperada:

```text
sin_riesgo
```

El código corre. Ese es el problema: corre sin quejarse, pero representa mal el mundo.

La variable `saturacion` no contiene una saturación normal. Contiene `None`, es decir, ausencia de dato. Sin embargo, el condicional la empuja hacia la rama `else` y produce `sin_riesgo`.

Este es un error de diseño, no solo de sintaxis.

## Crítica técnica: qué está mal

La versión anterior falla por varias razones.

Primero, usa un nombre incompleto. `saturacion` no dice unidad, fuente ni contexto. En un ejemplo corto puede tolerarse, pero en un sistema biomédico real conviene un nombre más explícito.

Segundo, usa la verdad implícita de Python para decidir. `None` se evalúa como falso. Eso puede ser útil en algunos contextos, pero aquí oculta una diferencia crítica: no tener dato no equivale a no tener riesgo.

Tercero, la rama `else` mezcla demasiados casos. Allí caen saturaciones normales, datos ausentes y quizá valores inválidos. Cuando una rama significa muchas cosas, el lector ya no puede auditar la decisión.

Cuarto, la salida no conserva razones. El estado final aparece sin explicar si hubo hipoxemia, dato faltante o regla no aplicable.

Esta crítica es parte de `CODE CLEAN`: no basta con corregir el código; hay que aprender a ver por qué una solución aparentemente simple es pobre.

## Versión mejorada: separar ausencia, regla y razón

Una versión más responsable hace explícitas las ramas relevantes.

```python
# Entrada: saturación periférica de oxígeno en porcentaje.
saturacion_oxigeno_porcentaje = None

# Salidas explicativas.
razones = []
pendientes = []

if saturacion_oxigeno_porcentaje is None:
    estado = "evaluacion_incompleta"
    pendientes.append("saturacion_oxigeno_porcentaje")
elif saturacion_oxigeno_porcentaje < 90:
    estado = "alto_riesgo"
    razones.append("hipoxemia")
else:
    estado = "sin_hipoxemia_detectada"
    razones.append("saturacion_en_rango_no_critico")

print("estado:", estado)
print("razones:", razones)
print("pendientes:", pendientes)
```

Salida esperada:

```text
estado: evaluacion_incompleta
razones: []
pendientes: ['saturacion_oxigeno_porcentaje']
```

Esta versión no es mejor porque sea más larga. Es mejor porque separa significados.

El dato faltante tiene una rama propia. El riesgo tiene una razón propia. La salida conserva qué se decidió y qué falta. El nombre de la variable declara magnitud y unidad. El lector puede revisar el flujo sin adivinar.

## Lección transferible: un `else` no debe ser un basurero

Una regla práctica:

> Un `else` responsable debe significar una cosa clara.

Si una rama `else` recibe valores normales, datos faltantes, errores de medición, unidades equivocadas y casos fuera de población, el código no está simplificando: está escondiendo estados distintos detrás de una sola etiqueta.

En medicina y ciencias de la vida, esa mezcla puede producir daño intelectual y operativo. El lector aprende mal. El sistema registra mal. La auditoría queda pobre. El siguiente capítulo o algoritmo hereda una decisión ambigua.

## Cláusulas de guarda: salir antes de mezclar

A veces conviene revisar primero las condiciones que impiden continuar. Esta forma se conoce como cláusula de guarda: una condición temprana que protege al resto del procedimiento.

```python
# Entrada inválida: edad ausente.
edad_anios = None

if edad_anios is None:
    resultado = "no_calculable"
    razon = "edad_no_registrada"
elif edad_anios < 0:
    resultado = "dato_invalido"
    razon = "edad_negativa"
elif edad_anios < 18:
    resultado = "fuera_de_poblacion"
    razon = "regla_pedagogica_solo_adultos"
else:
    resultado = "calculable"
    razon = "edad_adulta_registrada"

print(resultado)
print(razon)
```

Salida esperada:

```text
no_calculable
edad_no_registrada
```

El punto no es memorizar una forma. El punto es aprender una disciplina: antes de aplicar una regla, revisar si el dato existe, si es válido y si el caso pertenece al contrato del algoritmo.

## Condicionales y código limpio

En esta sección aprendemos una pieza transversal de código limpio:

> La claridad de un condicional depende menos de su brevedad que de la precisión con la que separa casos.

Una condición limpia:

- tiene nombres que revelan magnitud, unidad o intención;
- evita depender de valores falsos implícitos cuando eso confunde el dominio;
- separa dato faltante, dato inválido, criterio presente y criterio ausente;
- conserva razones cuando la salida afecta interpretación;
- reduce ramas que significan demasiadas cosas;
- puede probarse con ejemplos mínimos.

Esto no contradice la búsqueda de simplicidad. La simplicidad real no consiste en escribir menos líneas. Consiste en que cada línea cargue menos ambigüedad.

## Prueba mínima de la regla

Un condicional que representa una decisión importante debe poder probarse con casos pequeños.

```python
def clasificar_saturacion(saturacion_oxigeno_porcentaje):
    """Clasifica una saturación pedagógica sin reemplazar validación clínica."""
    if saturacion_oxigeno_porcentaje is None:
        return "evaluacion_incompleta"
    if saturacion_oxigeno_porcentaje < 90:
        return "alto_riesgo"
    return "sin_hipoxemia_detectada"


assert clasificar_saturacion(None) == "evaluacion_incompleta"
assert clasificar_saturacion(88) == "alto_riesgo"
assert clasificar_saturacion(96) == "sin_hipoxemia_detectada"
```

Salida esperada: no imprime nada si las pruebas pasan. Si alguna regla cambia de forma inesperada, Python produciría un `AssertionError`.

La prueba no valida una escala clínica. Valida algo más pequeño y necesario: que la función conserva las tres ramas conceptuales que prometió representar.

## Argumentos críticos

### Desacuerdo 1: brevedad contra explicitud

Pregunta: ¿debe preferirse siempre el condicional más corto?

La brevedad facilita lectura cuando no elimina significado. Pero si la brevedad mezcla estados distintos, se vuelve peligrosa. En código biomédico pedagógico, la explicitud suele ser preferible cuando enseña una diferencia conceptual importante.

Consenso operativo: escribir el condicional más simple que conserve los casos que cambian la decisión.

### Desacuerdo 2: regla directa contra función nombrada

Pregunta: ¿conviene escribir la condición en línea o extraerla a una función?

Una condición corta puede leerse bien dentro del flujo. Una condición repetida, densa o dependiente del dominio merece nombre propio. El nombre permite discutir el criterio, probarlo y cambiarlo sin esconderlo.

Consenso operativo: extraer a función cuando el nombre aumente comprensión, prueba o reutilización. No extraer por ceremonia.

### Desacuerdo 3: salida simple contra salida trazable

Pregunta: ¿basta con devolver una etiqueta?

Para ejercicios básicos puede bastar. Para dominios con consecuencias, una etiqueta sin razones empobrece auditoría y aprendizaje. La salida trazable permite saber qué datos faltaron y qué criterio se activó.

Consenso operativo: cuanto mayor sea el impacto de la decisión, más importante se vuelve conservar razones, pendientes y versión de regla.

## Puente hacia la frontera

Los condicionales no desaparecen cuando llegan los algoritmos avanzados. Cambian de forma.

En búsqueda, un condicional decide si un elemento cumple el criterio. En ordenamiento, decide prioridad relativa. En bioinformática, decide si dos caracteres coinciden, si se abre una brecha o si se penaliza una alineación. En modelos de decisión clínica, decide si hay suficiente evidencia para actuar, pedir más datos o detener la recomendación.

La pregunta se vuelve más profunda:

> ¿Qué condiciones gobiernan el paso entre datos, evidencia, acción y responsabilidad?

## Límites de esta miniatura

Los condicionales permiten expresar rutas de decisión, pero no vuelven correcta una regla de dominio. Una arquitectura `if/elif/else` puede ser legible y aun así representar mal el fenómeno. La sección enseña estructura de decisión; la validez médica exige fuentes, contexto y pruebas posteriores.

## Evaluar si entendiste

1. ¿Por qué `if saturacion and saturacion < 90` puede producir una salida peligrosa cuando `saturacion` vale `None`?
2. ¿Cuándo un `else` es claro y cuándo se convierte en una rama que oculta demasiados casos?
3. ¿Qué diferencia hay entre dato faltante, dato inválido y criterio ausente?
4. ¿Qué gana el lector cuando una versión frágil se compara con una versión mejorada?
5. ¿Cuándo conviene extraer una condición a una función nombrada?
6. ¿Por qué una prueba con `assert` puede enseñar más que una explicación larga?
7. ¿Qué campos mínimos conservarías para hacer trazable una decisión condicional?
8. ¿Cómo distinguirías simplicidad real de simplificación peligrosa?
9. ¿Qué condicional de un ejemplo anterior del libro podrías mejorar y por qué?
10. ¿Cómo se conecta una condición simple con decisiones más avanzadas como triage, búsqueda o alineamiento de secuencias?

## Vacíos de comprensión que debes vigilar

1. Creer que un condicional corto siempre es más limpio. Puede ser más ambiguo.
2. Confundir `else` con "todo lo demás no importa". En dominios biomédicos, lo demás suele importar.
3. Creer que probar una función equivale a validar clínicamente una regla. Son niveles distintos de evidencia.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma tres condicionales del libro y escribe qué caso maneja cada rama.
2. **Segunda hora:** reescribe un condicional que mezcle dato faltante y ausencia de riesgo.
3. **Tercera hora:** convierte una condición repetida en una función pequeña con tres `assert`.

## Bibliografía y fuentes

- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>
- Python Software Foundation. (2026). *More control flow tools*. Python 3.14.4 documentation. <https://docs.python.org/3/tutorial/controlflow.html>
- Python Software Foundation. (2026). *PEP 8: Style guide for Python code*. <https://peps.python.org/pep-0008/>
- Python Software Foundation. (2026). *PEP 20: The Zen of Python*. <https://peps.python.org/pep-0020/>
- Sittig, D. F., & Singh, H. (2010). A new sociotechnical model for studying health information technology in complex adaptive healthcare systems. *Quality & Safety in Health Care, 19*(Suppl. 3), i68-i74. <https://doi.org/10.1136/qshc.2010.042085>

## Siguiente paso

El siguiente tema natural son los bucles. Si los condicionales deciden qué camino tomar, los bucles deciden cómo repetir una operación sin perder control sobre estado, salida, error y trazabilidad.
