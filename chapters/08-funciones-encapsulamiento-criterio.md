# Funciones como encapsulamiento de criterio

Una función parece una forma de evitar repetir código. Esa definición es cierta, pero queda corta. En un sistema serio, una función no solo ahorra escritura: encierra una operación con nombre, entradas definidas, salida verificable y límites explícitos.

En medicina y ciencias de la vida, esto importa porque muchas operaciones no son simples cálculos. Clasificar una presión, decidir si un dato está incompleto, normalizar una unidad o resumir una serie de mediciones son actos pequeños, pero pueden cambiar el significado de una salida. Si esos actos quedan dispersos en el código, el criterio se vuelve difícil de revisar.

Una función bien escrita convierte una intención en una unidad de trabajo que se puede leer, probar, discutir y corregir.

## Origen técnico: nombrar una operación

En Python, una función se define con `def`.

```python
def duplicar(valor):
    return valor * 2
```

La palabra `def` anuncia que vamos a definir una operación. El nombre `duplicar` expresa qué hace. El parámetro `valor` representa la entrada. La instrucción `return` devuelve el resultado.

Esta estructura mínima ya contiene una idea poderosa:

```text
entrada -> operación nombrada -> salida
```

En código pequeño, esta estructura parece innecesaria. En código clínico, científico o de análisis de datos, se vuelve una herramienta de gobierno. Permite preguntar:

1. ¿Qué entra?
2. ¿Qué regla se aplica?
3. ¿Qué sale?
4. ¿Qué casos quedan fuera?
5. ¿Cómo sabemos que funciona?

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una función es una unidad reutilizable de código que recibe entradas, aplica una operación y puede devolver una salida. En este libro, además, una función es una cápsula de criterio: nombra una regla, limita su alcance, permite probarla y hace visible dónde vive una decisión.
</div>

La definición tiene cinco piezas:

1. **Nombre:** comunica la intención de la operación.
2. **Parámetros:** declaran qué necesita la función para trabajar.
3. **Cuerpo:** contiene la lógica.
4. **Retorno:** entrega el resultado.
5. **Contrato:** define qué promete la función y qué no promete.

El contrato puede estar escrito en un docstring, en pruebas, en tipos, en documentación o en una combinación de todos ellos.

## Versión ingenua: repetir criterio en varios lugares

Supongamos que queremos clasificar presiones sistólicas como alertas pedagógicas cuando son menores de 90 mmHg. Una versión ingenua puede repetir la regla en varios fragmentos.

```python
presion_1 = 88
presion_2 = 96

if presion_1 is not None and presion_1 < 90:
    print("alerta")
else:
    print("sin alerta")

if presion_2 is not None and presion_2 < 90:
    print("alerta")
else:
    print("sin alerta")
```

Salida esperada:

```text
alerta
sin alerta
```

El código funciona, pero deja un problema de diseño: la regla vive duplicada. Si mañana el umbral cambia, si aparece una categoría para datos faltantes o si necesitamos guardar razones, hay que corregir varios lugares.

Duplicar una regla no solo duplica texto. Duplica riesgo.

## Crítica técnica: qué está mal

La versión anterior falla por cuatro razones.

Primero, el criterio no tiene nombre propio. El lector debe reconstruir mentalmente qué significa `presion_1 is not None and presion_1 < 90`.

Segundo, el comportamiento ante datos faltantes está mezclado con el comportamiento ante valores normales. La ausencia se convierte en una rama silenciosa de "sin alerta".

Tercero, la regla es difícil de probar de forma aislada. Para saber si funciona, hay que ejecutar todo el bloque donde aparece.

Cuarto, la regla no tiene contrato. No sabemos si clasifica presión sistólica, presión media, presión arterial en general o cualquier número.

CODE CLEAN no consiste en esconder líneas dentro de funciones. Consiste en poner el criterio en un lugar donde pueda inspeccionarse con precisión.

## Versión mejorada: una función con salida explícita

Podemos encapsular la regla en una función.

```python
def clasificar_presion_sistolica(presion_sistolica_mmhg):
    """Clasifica una presión sistólica para un ejemplo pedagógico."""
    if presion_sistolica_mmhg is None:
        return {
            "estado": "pendiente",
            "razon": "presion_no_registrada",
        }

    if presion_sistolica_mmhg < 90:
        return {
            "estado": "alerta",
            "razon": "hipotension_sistolica_pedagogica",
        }

    return {
        "estado": "sin_alerta",
        "razon": "umbral_no_alcanzado",
    }


print(clasificar_presion_sistolica(88))
print(clasificar_presion_sistolica(None))
print(clasificar_presion_sistolica(96))
```

Salida esperada:

```text
{'estado': 'alerta', 'razon': 'hipotension_sistolica_pedagogica'}
{'estado': 'pendiente', 'razon': 'presion_no_registrada'}
{'estado': 'sin_alerta', 'razon': 'umbral_no_alcanzado'}
```

Esta función no reemplaza criterio clínico. Su valor aquí es pedagógico: muestra cómo una regla puede tener nombre, límites y salida trazable.

## Anatomía de la función

Miremos la función por partes.

```python
def clasificar_presion_sistolica(presion_sistolica_mmhg):
```

El nombre dice qué se clasifica. El parámetro incluye unidad: `mmhg`. Esto reduce ambigüedad. No es lo mismo recibir milímetros de mercurio que kilopascales, ni presión sistólica que presión media.

```python
    """Clasifica una presión sistólica para un ejemplo pedagógico."""
```

El docstring no debe adornar. Debe orientar. Aquí declara que la función pertenece a un ejemplo pedagógico, no a un protocolo médico real.

```python
    if presion_sistolica_mmhg is None:
```

La primera rama separa ausencia de dato. Esto evita tratar un dato faltante como normalidad.

```python
    return {
        "estado": "alerta",
        "razon": "hipotension_sistolica_pedagogica",
    }
```

El retorno no entrega solo una etiqueta. Entrega una razón. Esa razón permite auditar por qué la función clasificó como clasificó.

## Parámetros: lo que la función necesita

Un parámetro es una entrada nombrada. La calidad de sus nombres afecta la calidad del razonamiento.

Comparar:

```python
def evaluar(x):
    return x < 90
```

con:

```python
def esta_bajo_umbral_sistolico(presion_sistolica_mmhg):
    return presion_sistolica_mmhg < 90
```

La segunda versión es más larga, pero reduce ambigüedad. El nombre del parámetro conserva dominio, magnitud y unidad.

Esto no significa que todos los nombres deban ser extensos. Significa que deben conservar la información necesaria para no confundir el criterio.

## Retorno: lo que la función promete entregar

Una función puede retornar un número, un texto, un booleano, una lista, un diccionario o un objeto más complejo.

El retorno debe responder una pregunta clara.

```python
def es_presion_sistolica_baja(presion_sistolica_mmhg):
    return presion_sistolica_mmhg < 90
```

Esta función responde:

```text
¿el valor está por debajo del umbral?
```

Pero no responde:

```text
¿qué pasó si el dato falta?
¿cuál fue la razón?
¿qué versión de regla se usó?
```

Por eso a veces un booleano basta y a veces empobrece la salida. La elección depende del riesgo, del contexto y de la necesidad de trazabilidad.

## Funciones dentro de bucles

El capítulo anterior mostró bucles que recorren datos. Una función permite separar la regla del recorrido.

```python
def clasificar_presion_sistolica(presion_sistolica_mmhg):
    """Devuelve una clasificación pedagógica y trazable."""
    if presion_sistolica_mmhg is None:
        return {"estado": "pendiente", "razon": "presion_no_registrada"}

    if presion_sistolica_mmhg < 90:
        return {"estado": "alerta", "razon": "hipotension_sistolica_pedagogica"}

    return {"estado": "sin_alerta", "razon": "umbral_no_alcanzado"}


presiones_sistolicas_mmhg = [118, 88, None, 84]
resultados = []

for indice, presion_sistolica_mmhg in enumerate(presiones_sistolicas_mmhg):
    clasificacion = clasificar_presion_sistolica(presion_sistolica_mmhg)
    resultados.append({
        "indice": indice,
        "valor": presion_sistolica_mmhg,
        **clasificacion,
    })

print(resultados)
```

Salida esperada:

```text
[
  {'indice': 0, 'valor': 118, 'estado': 'sin_alerta', 'razon': 'umbral_no_alcanzado'},
  {'indice': 1, 'valor': 88, 'estado': 'alerta', 'razon': 'hipotension_sistolica_pedagogica'},
  {'indice': 2, 'valor': None, 'estado': 'pendiente', 'razon': 'presion_no_registrada'},
  {'indice': 3, 'valor': 84, 'estado': 'alerta', 'razon': 'hipotension_sistolica_pedagogica'}
]
```

El bucle conserva el recorrido. La función conserva la regla. Esa separación permite revisar cada cosa por separado.

## Prueba mínima de una función

Una función bien delimitada puede probarse con `assert`.

```python
def clasificar_presion_sistolica(presion_sistolica_mmhg):
    """Clasifica una presión sistólica para un ejemplo pedagógico."""
    if presion_sistolica_mmhg is None:
        return {"estado": "pendiente", "razon": "presion_no_registrada"}

    if presion_sistolica_mmhg < 90:
        return {"estado": "alerta", "razon": "hipotension_sistolica_pedagogica"}

    return {"estado": "sin_alerta", "razon": "umbral_no_alcanzado"}


assert clasificar_presion_sistolica(None)["estado"] == "pendiente"
assert clasificar_presion_sistolica(88)["estado"] == "alerta"
assert clasificar_presion_sistolica(90)["estado"] == "sin_alerta"
```

Salida esperada: no imprime nada si las pruebas pasan.

Estas pruebas son pequeñas, pero cubren tres fronteras importantes:

1. dato faltante;
2. valor por debajo del umbral;
3. valor justo en el umbral.

Probar el borde `90` es importante porque obliga a decidir si el criterio es menor que 90 o menor o igual que 90.

## Comentarios, docstrings y nombres

Un comentario no debe repetir lo obvio.

Mal comentario:

```python
# Si la presión es menor de 90, retorna alerta.
if presion_sistolica_mmhg < 90:
    return {"estado": "alerta"}
```

El comentario no agrega información. El código ya lo decía.

Mejor comentario:

```python
# En este ejemplo, el umbral es pedagógico; no representa una guía clínica.
if presion_sistolica_mmhg < 90:
    return {"estado": "alerta"}
```

Este comentario sí agrega contexto. Advierte el límite de uso de la regla.

Regla práctica:

- usar nombres para explicar qué es cada cosa;
- usar docstrings para explicar qué promete la función;
- usar comentarios para explicar límites, decisiones no obvias o riesgos del dominio.

## Argumentos críticos

### Desacuerdo 1: función corta contra función explícita

Pregunta: ¿una función debe ser siempre corta?

No necesariamente. Una función debe tener una responsabilidad clara. A veces una función explícita, con ramas bien nombradas y salida trazable, es mejor que una función brevísima que obliga a adivinar.

Consenso operativo: preferir funciones pequeñas, pero no sacrificar significado por brevedad.

### Desacuerdo 2: retornar booleanos contra retornar razones

Pregunta: ¿por qué no devolver simplemente `True` o `False`?

Porque un booleano responde una pregunta estrecha. En tareas de bajo riesgo puede bastar. En tareas donde importa auditar, enseñar o depurar, conviene devolver también razón, estado o metadatos.

Consenso operativo: usar booleanos para predicados simples; usar estructuras trazables cuando la salida pueda guiar decisiones posteriores.

### Desacuerdo 3: muchas funciones contra flujo legible

Pregunta: ¿dividir demasiado puede empeorar el código?

Sí. Si cada línea se convierte en una función, el lector salta de nombre en nombre sin ver el proceso. La abstracción excesiva también puede ocultar.

Consenso operativo: crear una función cuando nombra un criterio real, reduce duplicación significativa o permite probar una unidad de comportamiento.

## Puente hacia la frontera

Las funciones son la base de diseños más avanzados.

Una biblioteca científica está hecha de funciones y objetos que encapsulan operaciones. Un pipeline de datos encadena funciones que limpian, transforman, validan y resumen. Un modelo predictivo depende de funciones de preprocesamiento, entrenamiento, evaluación y calibración.

En bioinformática, una función puede calcular una distancia entre secuencias. En epidemiología, puede transformar una tabla de casos. En medicina de precisión, puede aplicar una regla de inclusión para una cohorte. En sistemas clínicos, puede separar dato faltante, dato inválido y dato que activa alerta.

La pregunta madura no es:

```text
¿cómo evito repetir estas líneas?
```

La pregunta madura es:

```text
¿qué criterio merece tener nombre, contrato y prueba?
```

## Evaluar si entendiste

1. ¿Por qué una función no debe entenderse solo como una forma de ahorrar escritura?
2. ¿Qué diferencia hay entre parámetro y retorno?
3. ¿Por qué el nombre de una función puede cambiar la claridad del razonamiento?
4. ¿Qué riesgo aparece cuando una regla se repite en varios lugares?
5. ¿Cuándo basta retornar `True` o `False`?
6. ¿Cuándo conviene retornar una estructura con estado y razón?
7. ¿Por qué probar el borde de un umbral es más importante que probar solo casos obvios?
8. ¿Qué diferencia hay entre comentario útil y comentario redundante?
9. ¿Cómo se conectan funciones y bucles?
10. ¿Qué criterio del capítulo merecería convertirse en función en un sistema real?

## Vacíos de comprensión que debes vigilar

1. Creer que una función mejora el código solo por existir.
2. Escribir nombres genéricos como `procesar`, `validar` o `calcular` sin aclarar qué procesan, validan o calculan.
3. Usar comentarios para compensar nombres pobres.
4. Retornar salidas demasiado pobres para el riesgo del dominio.
5. Confundir una prueba mínima con validación clínica o científica completa.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma tres bloques repetidos de código y conviértelos en funciones con nombres específicos.
2. **Segunda hora:** agrega pruebas con `assert` para caso normal, dato faltante y caso de borde.
3. **Tercera hora:** revisa si cada función devuelve suficiente información para auditar su salida.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *Defining functions*. Python 3.14.4 documentation. https://docs.python.org/3/tutorial/controlflow.html#defining-functions
- Python Software Foundation. (2026). *More on defining functions*. Python 3.14.4 documentation. https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions
- Python Software Foundation. (2026). *PEP 257: Docstring conventions*. https://peps.python.org/pep-0257/
- Martin, R. C. (2008). *Clean code: A handbook of agile software craftsmanship*. Prentice Hall.

## Siguiente paso

El siguiente tema natural es separar funciones puras, funciones con efectos y funciones que coordinan procesos. Esa distinción ayuda a entender por qué algunas piezas son fáciles de probar y otras necesitan pruebas de integración.
