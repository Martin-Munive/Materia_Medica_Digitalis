# Bucles como control de procesos

Un bucle parece una instrucción para repetir. Esa definición es correcta, pero incompleta. En medicina y ciencias de la vida, repetir no es una acción neutra: significa seguir observando, seguir buscando, seguir acumulando evidencia o seguir vigilando hasta que algo cambie.

Un bucle puede recorrer pacientes, signos vitales, genes, resultados de laboratorio, intentos de aprendizaje, registros de una base de datos o pasos de un protocolo. Por eso un bucle no debe entenderse solo como sintaxis. Es una forma de controlar un proceso en el tiempo.

Si los condicionales deciden qué camino tomar, los bucles deciden cómo sostener una operación sin perder control sobre estado, terminación, error y trazabilidad.

## Origen técnico: repetir con control

En programación, un bucle permite ejecutar un bloque de código varias veces. Python ofrece dos formas centrales:

- `for`, cuando queremos recorrer una colección o una secuencia conocida;
- `while`, cuando queremos repetir mientras una condición se mantenga verdadera.

La diferencia no es solo gramatical. También es conceptual.

Un `for` suele decir:

```text
aplica esta operación a cada elemento de una colección
```

Un `while` suele decir:

```text
continúa mientras el estado del sistema lo permita o lo exija
```

En un contexto biomédico, esa diferencia importa. No es lo mismo recorrer una lista de pacientes ya cargados que vigilar un proceso hasta que aparezca un criterio de parada.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un bucle es una estructura de control que repite una operación bajo una regla de recorrido o permanencia. En este libro, además, un bucle es una forma de gobernar procesos: qué se repite, sobre qué datos, con qué estado acumulado, cuándo se detiene y qué evidencia deja.
</div>

La definición tiene cinco piezas:

1. **Operación repetida:** qué acción se ejecuta.
2. **Colección o condición:** sobre qué elementos o bajo qué regla se repite.
3. **Estado interno:** qué cambia durante la repetición.
4. **Condición de parada:** cuándo deja de repetirse.
5. **Salida trazable:** qué queda al final y por qué.

Un bucle limpio permite identificar esas cinco piezas sin adivinar.

## Versión ingenua: contar alertas sin explicar nada

Supongamos que tenemos mediciones de presión arterial sistólica durante una guardia y queremos contar cuántas lecturas están por debajo de 90 mmHg.

```python
# Entrada: presiones sistólicas registradas durante una observación.
presiones = [118, 102, 88, None, 84, 96]

# Versión frágil: ignora datos faltantes y solo acumula un número.
alertas = 0

for presion in presiones:
    if presion and presion < 90:
        alertas = alertas + 1

print(alertas)
```

Salida esperada:

```text
2
```

El resultado parece razonable: hay dos valores menores de 90. Pero el bucle escondió una parte importante del proceso. Había un dato faltante. El código no lo registró. Tampoco conservó en qué posiciones ocurrieron las alertas ni qué mediciones quedaron pendientes.

El problema no es que el bucle sea corto. El problema es que la repetición produce una salida pobre para auditar.

## Crítica técnica: qué está mal

La versión anterior falla por cuatro razones.

Primero, usa `if presion and presion < 90`. Esa condición trata `None` como falso, pero también trataría `0` como falso. Aunque una presión sistólica de 0 no sea una medición fisiológica válida en este ejemplo, el patrón mental es peligroso: mezcla ausencia, invalidez y criterio clínico.

Segundo, solo acumula un conteo. Un número puede ser útil, pero no explica qué pasó durante el recorrido.

Tercero, no separa datos faltantes. En un sistema biomédico, un dato ausente puede ser tan importante como un dato anormal, porque cambia la confianza de la interpretación.

Cuarto, no deja trazabilidad. Si el resultado final es `2`, no sabemos cuáles lecturas activaron la alerta ni cuántas quedaron sin evaluar.

CODE CLEAN no consiste en escribir bucles elegantes. Consiste en escribir bucles que no destruyan significado mientras repiten.

## Versión mejorada: recorrer, clasificar y conservar razones

Una versión más responsable separa las salidas.

```python
# Entrada: presiones sistólicas en mmHg.
presiones_sistolicas_mmhg = [118, 102, 88, None, 84, 96]

# Salidas trazables.
alertas = []
pendientes = []

for indice, presion_sistolica_mmhg in enumerate(presiones_sistolicas_mmhg):
    if presion_sistolica_mmhg is None:
        pendientes.append({
            "indice": indice,
            "razon": "presion_no_registrada",
        })
    elif presion_sistolica_mmhg < 90:
        alertas.append({
            "indice": indice,
            "valor": presion_sistolica_mmhg,
            "razon": "hipotension_sistolica_pedagogica",
        })

print("alertas:", alertas)
print("pendientes:", pendientes)
```

Salida esperada:

```text
alertas: [{'indice': 2, 'valor': 88, 'razon': 'hipotension_sistolica_pedagogica'}, {'indice': 4, 'valor': 84, 'razon': 'hipotension_sistolica_pedagogica'}]
pendientes: [{'indice': 3, 'razon': 'presion_no_registrada'}]
```

Esta versión no pretende ser una escala clínica. Es un ejemplo pedagógico de diseño. El bucle recorre datos, identifica casos relevantes y conserva evidencia mínima para revisar el resultado.

La mejora central es conceptual: el bucle ya no solo cuenta; produce una pequeña historia verificable del recorrido.

## El acumulador: memoria del recorrido

Muchos bucles tienen una variable que conserva lo que se ha encontrado hasta el momento. Esa variable se llama acumulador.

En el ejemplo anterior, `alertas` y `pendientes` son acumuladores. Empiezan vacíos y se llenan durante el recorrido.

Un acumulador puede guardar:

- conteos;
- sumas;
- máximos o mínimos;
- elementos filtrados;
- razones;
- errores;
- estados intermedios;
- decisiones tomadas.

El acumulador es una forma de memoria. Por eso debe diseñarse con cuidado. Si solo acumula un número, el resultado puede ser compacto pero poco explicativo. Si acumula demasiada información sin estructura, puede volverse ilegible. La pregunta de diseño es:

```text
¿qué necesito recordar para que la salida sea útil y verificable?
```

## `for`: recorrer colecciones conocidas

El bucle `for` es apropiado cuando ya tenemos una colección definida.

```python
# Entrada: nombres de medicamentos que deben normalizarse.
medicamentos = [" Metformina ", "ENALAPRIL", "  atorvastatina"]

normalizados = []

for medicamento in medicamentos:
    nombre_limpio = medicamento.strip().lower()
    normalizados.append(nombre_limpio)

print(normalizados)
```

Salida esperada:

```text
['metformina', 'enalapril', 'atorvastatina']
```

El `for` dice: toma cada elemento de esta colección y aplica una transformación. Es una herramienta natural para limpiar datos, recorrer registros, revisar filas, contar eventos o preparar entradas para otro algoritmo.

## `while`: repetir mientras el estado lo exige

El bucle `while` es diferente. No recorre necesariamente una colección. Repite mientras una condición se mantenga verdadera.

```python
# Simulación pedagógica: intentos de lectura hasta obtener un dato válido.
lecturas = [None, None, 37.8]
indice = 0
temperatura_celsius = None

while temperatura_celsius is None and indice < len(lecturas):
    temperatura_celsius = lecturas[indice]
    indice = indice + 1

print("temperatura:", temperatura_celsius)
print("intentos:", indice)
```

Salida esperada:

```text
temperatura: 37.8
intentos: 3
```

Aquí la lógica no es "para cada elemento, haz algo" sino "continúa hasta encontrar un dato útil o hasta agotar los intentos".

Esa diferencia se parece más a muchos procesos reales: repetir medición, reintentar consulta, buscar una coincidencia, esperar una condición, revisar un lote hasta hallar un error.

## El riesgo del bucle infinito

Un `while` mal diseñado puede no terminar.

```python
# Ejemplo incorrecto: no ejecutar como patrón real.
conteo = 0

while conteo < 3:
    print("sigue")
```

Este bucle nunca cambia `conteo`. La condición `conteo < 3` siempre seguirá siendo verdadera.

Un bucle responsable debe tener una condición de parada clara. En un `while`, esa condición suele depender de que algo cambie dentro del bucle:

```python
conteo = 0

while conteo < 3:
    print("sigue")
    conteo = conteo + 1
```

Salida esperada:

```text
sigue
sigue
sigue
```

La lección no es temer a `while`. La lección es exigirle terminación explícita.

## Filtrar no es lo mismo que decidir

Un uso común de bucles es filtrar elementos.

```python
edades = [12, 45, 71, 33, 16]
adultos = []

for edad in edades:
    if edad >= 18:
        adultos.append(edad)

print(adultos)
```

Salida esperada:

```text
[45, 71, 33]
```

Pero filtrar no equivale automáticamente a decidir. Si el filtro excluye datos, debemos saber por qué. En medicina, excluir por edad, sexo, embarazo, función renal, dato faltante o regla no aplicable puede tener consecuencias.

Una versión más trazable podría separar incluidos y excluidos:

```python
edades = [12, 45, 71, 33, 16]
incluidos = []
excluidos = []

for indice, edad in enumerate(edades):
    if edad >= 18:
        incluidos.append({"indice": indice, "edad": edad})
    else:
        excluidos.append({
            "indice": indice,
            "edad": edad,
            "razon": "fuera_de_poblacion_adulta",
        })

print("incluidos:", incluidos)
print("excluidos:", excluidos)
```

Esta forma es más larga, pero enseña una idea importante: el bucle no solo selecciona; también puede justificar.

## Prueba mínima de un bucle

Un bucle que clasifica datos debe probarse con casos pequeños.

```python
def resumir_presiones(presiones_sistolicas_mmhg):
    """Resume alertas pedagogicas de presion sin reemplazar criterio clinico."""
    alertas = []
    pendientes = []

    for indice, presion_sistolica_mmhg in enumerate(presiones_sistolicas_mmhg):
        if presion_sistolica_mmhg is None:
            pendientes.append(indice)
        elif presion_sistolica_mmhg < 90:
            alertas.append(indice)

    return {
        "alertas": alertas,
        "pendientes": pendientes,
    }


resultado = resumir_presiones([118, 88, None, 84])

assert resultado["alertas"] == [1, 3]
assert resultado["pendientes"] == [2]
```

Salida esperada: no imprime nada si las pruebas pasan.

Esta prueba no valida un protocolo médico. Valida que el bucle conserve dos distinciones conceptuales: valores que activan alerta y valores que no pueden evaluarse.

## Argumentos críticos

### Desacuerdo 1: conteo simple contra trazabilidad

Pregunta: ¿siempre debemos guardar razones, índices y detalles?

No. A veces un conteo basta. Pero cuanto más importante sea la salida, más necesario es saber de dónde vino. Un conteo puede servir para una vista rápida; una lista trazable sirve para auditoría, depuración y aprendizaje.

Consenso operativo: usar la salida más simple que conserve la evidencia necesaria para revisar la decisión.

### Desacuerdo 2: `for` contra `while`

Pregunta: ¿cuál es mejor?

Ninguno es superior en abstracto. `for` es natural cuando hay una colección definida. `while` es natural cuando la repetición depende de un estado que cambia.

Consenso operativo: si sabes qué vas a recorrer, usa `for`; si necesitas repetir hasta una condición de parada, usa `while` con límite explícito.

### Desacuerdo 3: bucle explícito contra comprensión de listas

Python permite escribir filtros compactos con comprensiones de listas.

```python
adultos = [edad for edad in edades if edad >= 18]
```

Esto es legible cuando la regla es simple. Pero si necesitas conservar razones, manejar datos faltantes, registrar errores o enseñar el proceso, el bucle explícito suele ser mejor.

Consenso operativo: compactar cuando no se pierda significado; expandir cuando el proceso necesita explicación.

## Puente hacia la frontera

Los bucles son la puerta hacia algoritmos más avanzados.

Una búsqueda lineal es un bucle que recorre elementos hasta encontrar una coincidencia. Un algoritmo de ordenamiento contiene bucles que comparan y reorganizan. Un alineamiento de secuencias en bioinformática llena matrices mediante recorridos repetidos. Un sistema de vigilancia clínica procesa flujos de datos con bucles que actualizan estado.

La pregunta madura no es:

```text
¿cómo repito instrucciones?
```

La pregunta madura es:

```text
¿qué estado cambia en cada iteración y qué evidencia necesito conservar?
```

## Evaluar si entendiste

1. ¿Por qué un bucle no debe entenderse solo como repetición?
2. ¿Qué diferencia conceptual hay entre `for` y `while`?
3. ¿Qué es un acumulador y por qué puede mejorar o empobrecer la trazabilidad?
4. ¿Por qué `if presion and presion < 90` es una condición frágil?
5. ¿Qué información se pierde cuando un bucle solo devuelve un conteo?
6. ¿Qué condición debe tener todo `while` responsable?
7. ¿Cuándo conviene guardar índices o razones durante un recorrido?
8. ¿Por qué filtrar datos no equivale automáticamente a tomar una decisión responsable?
9. ¿Cómo probarías que un bucle conserva datos faltantes y alertas por separado?
10. ¿Cómo se conecta un bucle simple con búsqueda, ordenamiento o bioinformática?

## Vacíos de comprensión que debes vigilar

1. Creer que repetir muchas veces es automáticamente automatizar bien.
2. Confundir una salida compacta con una salida verificable.
3. Usar `while` sin una estrategia clara de terminación.
4. Tratar datos faltantes como si fueran datos normales que no activan nada.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** escribe tres bucles `for` que recorran listas pequeñas y expliquen qué acumulan.
2. **Segunda hora:** convierte uno de esos bucles en una función con `assert`.
3. **Tercera hora:** escribe un `while` con una condición de parada explícita y explica qué variable cambia para que termine.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *More control flow tools*. Python 3.14.4 documentation. <https://docs.python.org/3/tutorial/controlflow.html>
- Python Software Foundation. (2026). *PEP 8: Style guide for Python code*. <https://peps.python.org/pep-0008/>
- Miller, G. A. (1956). The magical number seven, plus or minus two. *Psychological Review, 63*(2), 81-97. <https://doi.org/10.1037/h0043158>
- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>

## Siguiente paso

El siguiente tema natural son las funciones. Si los bucles controlan procesos repetidos, las funciones permiten encapsular criterios, nombrar operaciones, probarlas y reutilizarlas sin duplicar lógica.

