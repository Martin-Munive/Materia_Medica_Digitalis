# Listas, diccionarios y registros

Hasta ahora hemos tratado valores individuales: números con unidad, texto controlado, booleanos insuficientes, fechas, intervalos y ausencias. Pero los datos biomédicos casi nunca aparecen solos. Un paciente tiene múltiples mediciones. Una muestra tiene identificador, tipo, fecha, técnica y resultado. Un evento clínico tiene persona, momento, estado, fuente y razón. Un gen tiene nombre, coordenadas, transcritos, variantes y anotaciones.

Esta sección da el siguiente paso: pasar de valores aislados a colecciones. En Python, las primeras colecciones que conviene dominar son `list` y `dict`. Una lista conserva una secuencia de elementos. Un diccionario conserva asociaciones entre claves y valores. Un registro es una unidad de observación con campos definidos.

La diferencia no es solo sintáctica. Elegir entre lista, diccionario y registro decide qué puede buscarse, qué puede repetirse, qué queda nombrado, qué depende del orden y qué se pierde si cambia una posición.

## Origen técnico: secuencias y mapas

Python trata las listas como secuencias mutables: colecciones ordenadas que se pueden recorrer, modificar y extender. Los diccionarios son mapas: estructuras que asocian claves con valores. Una clave permite acceder a un valor por nombre, no por posición.

```python
valores = [120, 128, 132]
registro = {
    "paciente_id": "P001",
    "prueba": "presion_sistolica",
    "valor": 120,
    "unidad": "mmHg",
}

print(valores[0])
print(registro["valor"])
```

Salida esperada:

```text
120
120
```

Ambas estructuras guardan datos, pero prometen cosas distintas. En la lista, `valores[0]` significa "primer elemento". En el diccionario, `registro["valor"]` significa "campo valor". El segundo es más explícito cuando la posición no tiene significado biomédico suficiente.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una lista es una colección ordenada de elementos. Un diccionario es una colección de asociaciones clave-valor. Un registro biomédico es una observación representada por campos nombrados, con contrato de completitud, tipo, unidad, estado y trazabilidad proporcional al uso posterior.
</div>

Esta definición separa tres niveles:

**Lista.** Útil para guardar varios elementos del mismo tipo o de una misma función: mediciones, eventos, errores, identificadores, resultados validados.

**Diccionario.** Útil para nombrar partes de un objeto: `valor`, `unidad`, `estado`, `razon`, `fuente`.

**Registro.** Útil para representar una observación completa: una presión, una dosis, una variante, una muestra, un resultado de laboratorio o un evento clínico.

En una miniatura biomédica responsable, una lista de diccionarios suele representar una tabla pequeña: cada diccionario es una fila; cada clave es un campo.

## Versión ingenua: usar posiciones como si fueran significado

Supongamos que un sistema recibe mediciones de laboratorio como listas. Cada posición tiene un significado implícito.

```python
medicion = ["P001", "creatinina", 1.8, "mg/dL"]

print(medicion[0])
print(medicion[2])
```

Salida esperada:

```text
P001
1.8
```

El código funciona, pero el significado vive fuera de la estructura. Hay que recordar que:

- posición 0 significa paciente;
- posición 1 significa prueba;
- posición 2 significa valor;
- posición 3 significa unidad.

Si alguien cambia el orden, agrega un campo o elimina una posición, el programa puede seguir ejecutándose y producir una interpretación falsa.

```python
medicion = ["P001", 1.8, "creatinina", "mg/dL"]

prueba = medicion[1]
valor = medicion[2]

print(prueba)
print(valor)
```

Salida esperada:

```text
1.8
creatinina
```

Python no sabe que eso es absurdo. La lista solo promete orden, no significado clínico.

## Crítica técnica: qué está mal

Primero, la estructura depende de memoria humana. El significado de cada posición no está escrito en el dato.

Segundo, no hay contrato de campos. Una medición incompleta puede tener tres elementos y aun así pasar hasta una etapa posterior.

Tercero, no hay validación de unidad. El valor `1.8` puede ser creatinina en `mg/dL`, creatinina en otra unidad, glucosa, lactato o una puntuación.

Cuarto, no hay estado. No sabemos si el resultado es válido, pendiente, corregido, preliminar, censurado o inválido.

Quinto, no hay trazabilidad. No aparece fuente, fecha, versión de regla ni razón de aceptación o rechazo.

La lista no es el problema. El problema es usarla para representar un registro cuando el registro necesita campos nombrados.

## Versión mejorada: lista de registros validados

Construyamos una representación mínima para mediciones pedagógicas de laboratorio. No es un sistema clínico real ni una escala validada. Es una miniatura para aprender estructura.

```python
from enum import Enum


class EstadoRegistro(Enum):
    VALIDO = "valido"
    INCOMPLETO = "incompleto"
    UNIDAD_INVALIDA = "unidad_invalida"
    TIPO_INVALIDO = "tipo_invalido"
    VALOR_INVALIDO = "valor_invalido"
    PRUEBA_NO_SOPORTADA = "prueba_no_soportada"


CONTRATO_LABORATORIO = {
    "creatinina": {
        "unidad": "mg/dL",
        "minimo": 0,
        "maximo": 25,
    },
    "hemoglobina": {
        "unidad": "g/dL",
        "minimo": 0,
        "maximo": 25,
    },
}


CAMPOS_REQUERIDOS = {"paciente_id", "prueba", "valor", "unidad"}


def validar_registro_laboratorio(registro):
    """Valida una medición pedagógica antes de agregarla a una colección."""
    if not isinstance(registro, dict):
        return {
            "estado": EstadoRegistro.TIPO_INVALIDO,
            "registro": None,
            "razon": "registro_no_es_diccionario",
        }

    faltantes = CAMPOS_REQUERIDOS - set(registro)
    if faltantes:
        return {
            "estado": EstadoRegistro.INCOMPLETO,
            "registro": dict(registro),
            "razon": f"faltan_campos:{sorted(faltantes)}",
        }

    prueba = registro["prueba"]
    if prueba not in CONTRATO_LABORATORIO:
        return {
            "estado": EstadoRegistro.PRUEBA_NO_SOPORTADA,
            "registro": dict(registro),
            "razon": "prueba_fuera_del_contrato_pedagogico",
        }

    contrato = CONTRATO_LABORATORIO[prueba]
    if registro["unidad"] != contrato["unidad"]:
        return {
            "estado": EstadoRegistro.UNIDAD_INVALIDA,
            "registro": dict(registro),
            "razon": "unidad_no_esperada_para_prueba",
        }

    valor = registro["valor"]
    if not isinstance(valor, (int, float)):
        return {
            "estado": EstadoRegistro.TIPO_INVALIDO,
            "registro": dict(registro),
            "razon": "valor_no_numerico",
        }

    if valor < contrato["minimo"] or valor > contrato["maximo"]:
        return {
            "estado": EstadoRegistro.VALOR_INVALIDO,
            "registro": dict(registro),
            "razon": "valor_fuera_de_rango_pedagogico",
        }

    registro_validado = {
        "paciente_id": registro["paciente_id"],
        "prueba": prueba,
        "valor": valor,
        "unidad": registro["unidad"],
        "estado": EstadoRegistro.VALIDO,
        "razon": "registro_aceptado",
    }

    return {
        "estado": EstadoRegistro.VALIDO,
        "registro": registro_validado,
        "razon": "cumple_contrato_minimo",
    }
```

El validador no devuelve solo `True` o `False`. Devuelve estado, registro y razón. Esto permite que una lista de resultados conserve tanto los registros aceptados como los rechazados.

```python
registros_crudos = [
    {"paciente_id": "P001", "prueba": "creatinina", "valor": 1.8, "unidad": "mg/dL"},
    {"paciente_id": "P001", "prueba": "hemoglobina", "valor": 13.2, "unidad": "g/dL"},
    {"paciente_id": "P002", "prueba": "creatinina", "valor": 160, "unidad": "umol/L"},
]

validados = [validar_registro_laboratorio(registro) for registro in registros_crudos]

print(validados[0]["estado"].value)
print(validados[2]["estado"].value)
print(validados[2]["razon"])
```

Salida esperada:

```text
valido
unidad_invalida
unidad_no_esperada_para_prueba
```

La lista conserva el conjunto. El diccionario conserva el significado de cada campo. El registro validado conserva contrato, estado y razón.

## Agrupar sin perder estructura

Una vez los registros están validados, podemos agruparlos. Agrupar no es solo contar. Es decidir qué clave organiza la colección.

```python
def agrupar_validos_por_paciente(resultados):
    """Agrupa registros válidos por paciente y conserva rechazos separados."""
    por_paciente = {}
    rechazados = []

    for resultado in resultados:
        if resultado["estado"] != EstadoRegistro.VALIDO:
            rechazados.append(resultado)
            continue

        registro = resultado["registro"]
        paciente_id = registro["paciente_id"]

        if paciente_id not in por_paciente:
            por_paciente[paciente_id] = []

        por_paciente[paciente_id].append(registro)

    return {
        "por_paciente": por_paciente,
        "rechazados": rechazados,
    }


resumen = agrupar_validos_por_paciente(validados)
print(len(resumen["por_paciente"]["P001"]))
print(len(resumen["rechazados"]))
```

Salida esperada:

```text
2
1
```

Aquí aparece una estructura frecuente:

- una lista de registros crudos;
- una lista de resultados validados;
- un diccionario de agrupación por paciente;
- una lista de rechazos.

Cada colección tiene una función distinta. No todo debe vivir en una sola lista ni en un solo diccionario.

## Anatomía del contrato

La lista responde a una pregunta: ¿qué elementos componen la colección?

```python
registros_crudos = [registro_1, registro_2, registro_3]
```

El orden puede importar si representa secuencia temporal, prioridad o llegada. Si no importa, no debe inventarse significado clínico a partir de la posición.

El diccionario responde a otra pregunta: ¿qué valor corresponde a esta clave?

```python
registro["unidad"]
```

La clave debe ser estable, clara y semántica. `registro["u"]` ahorra caracteres y pierde claridad. `registro["unidad"]` permite leer el dato.

El registro responde a una tercera pregunta: ¿qué campos mínimos necesita esta observación para ser interpretable?

```python
CAMPOS_REQUERIDOS = {"paciente_id", "prueba", "valor", "unidad"}
```

El contrato no exige que todos los sistemas tengan esos mismos campos. Exige que el sistema declare cuáles necesita.

La agrupación responde a una cuarta pregunta: ¿por qué criterio se reorganiza la colección?

```python
por_paciente[paciente_id].append(registro)
```

Agrupar por paciente, por prueba, por fecha, por sede o por estado produce estructuras distintas y preguntas distintas.

## Pruebas mínimas

```python
# Propiedad 1: un registro válido conserva estado y unidad.
creatinina = validar_registro_laboratorio({
    "paciente_id": "P001",
    "prueba": "creatinina",
    "valor": 1.8,
    "unidad": "mg/dL",
})
assert creatinina["estado"] == EstadoRegistro.VALIDO
assert creatinina["registro"]["unidad"] == "mg/dL"

# Propiedad 2: un campo faltante no debe producir registro válido.
incompleto = validar_registro_laboratorio({
    "paciente_id": "P001",
    "prueba": "creatinina",
    "valor": 1.8,
})
assert incompleto["estado"] == EstadoRegistro.INCOMPLETO

# Propiedad 3: una unidad equivocada se rechaza antes de agrupar.
unidad_errada = validar_registro_laboratorio({
    "paciente_id": "P002",
    "prueba": "creatinina",
    "valor": 160,
    "unidad": "umol/L",
})
assert unidad_errada["estado"] == EstadoRegistro.UNIDAD_INVALIDA

# Propiedad 4: la agrupación conserva válidos y rechazos separados.
resumen = agrupar_validos_por_paciente([creatinina, incompleto, unidad_errada])
assert len(resumen["por_paciente"]["P001"]) == 1
assert len(resumen["rechazados"]) == 2
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas verifican una propiedad estructural: un registro no entra a una colección operativa solo porque tiene forma de diccionario. Debe cumplir contrato.

## `list`, `dict` y registros no son tablas todavía

Una lista de diccionarios puede parecer una tabla.

```python
tabla_pequena = [
    {"paciente_id": "P001", "prueba": "creatinina", "valor": 1.8},
    {"paciente_id": "P002", "prueba": "creatinina", "valor": 0.9},
]
```

Pero todavía no es una tabla robusta. No hay esquema formal, tipos obligatorios, índices, manejo de valores faltantes, control de duplicados, auditoría de cambios ni reglas de transformación masiva.

Esta forma es suficiente para aprender. También es suficiente para escribir validadores pequeños. Pero no debe confundirse con un sistema de datos clínicos.

El paso siguiente será natural: si una lista de registros crece, necesitaremos pensar en tablas, columnas, limpieza, validación, importación y exportación.

## CODE CLEAN: nombrar estructura antes de recorrerla

Comparemos dos estilos.

```python
for x in datos:
    if x[2] > 1.5:
        print(x[0])
```

El código es breve y frágil. No sabemos qué es `x`, qué significa `x[2]`, qué unidad tiene el valor ni por qué `1.5` importa.

Una versión más limpia no necesariamente es más larga por capricho. Es más explícita porque el dominio lo exige.

```python
for registro in registros_validados:
    if registro["prueba"] == "creatinina" and registro["valor"] > 1.5:
        print(registro["paciente_id"])
```

Esta versión sigue siendo pedagógica y limitada, pero el lector puede reconstruir la intención.

Mejor todavía: separar selección, razón y salida.

```python
def seleccionar_creatininas_altas(registros):
    """Selecciona registros pedagógicos de creatinina por encima de un umbral declarado."""
    seleccionados = []

    for registro in registros:
        if registro["prueba"] != "creatinina":
            continue

        if registro["valor"] > 1.5:
            seleccionados.append({
                "paciente_id": registro["paciente_id"],
                "valor": registro["valor"],
                "unidad": registro["unidad"],
                "razon": "creatinina_mayor_a_umbral_pedagogico",
            })

    return seleccionados
```

El nombre de la función declara el propósito. El umbral queda dentro de una razón explícita. La salida no es solo una lista de identificadores; conserva valor, unidad y motivo.

## Límites y errores frecuentes

1. **Usar listas posicionales para registros complejos.** Cambiar el orden puede cambiar el significado sin romper el programa.
2. **Creer que todo diccionario es un registro válido.** Un diccionario puede tener claves incorrectas, faltantes o valores fuera de contrato.
3. **Agrupar antes de validar.** La colección queda ordenada, pero puede estar llena de datos incompatibles.
4. **Usar claves crípticas.** `p`, `v`, `u` ahorran espacio y aumentan ambigüedad.
5. **Sobrescribir claves sin darse cuenta.** En un diccionario, una clave duplicada no conserva dos valores; el último reemplaza al anterior.
6. **Confundir índice con identidad.** La posición `0` no es un identificador estable de paciente, muestra o evento.
7. **Mezclar registros válidos y rechazados sin estado.** Después nadie sabe qué puede calcularse.
8. **No declarar si el orden importa.** Una lista puede representar llegada, prioridad, tiempo o simple colección; debe quedar claro.
9. **Convertir una tabla grande en listas de diccionarios sin esquema.** Sirve para aprender, no para gobernar datos complejos.
10. **No probar casos estructurales.** También deben probarse faltantes, claves erradas, unidades incorrectas y tipos inválidos.

## Argumentos críticos

### Desacuerdo 1: lista simple contra registro explícito

Pregunta: ¿por qué no guardar todo en listas si son más rápidas de escribir?

Porque escribir rápido no equivale a representar bien. Una lista simple puede ser adecuada cuando todos los elementos tienen el mismo significado: presiones válidas, edades, identificadores, errores. Pero si cada posición representa un campo distinto, el dato necesita nombres.

Consenso operativo: usar listas para colecciones; usar diccionarios o estructuras equivalentes para registros con campos.

### Desacuerdo 2: diccionario flexible contra esquema rígido

Pregunta: ¿no es mejor permitir cualquier campo para no limitar el sistema?

La flexibilidad ayuda durante exploración, importación o prototipos. Pero una decisión biomédica necesita saber qué campos son obligatorios. Sin contrato, la libertad se convierte en ambigüedad.

Consenso operativo: permitir campos adicionales si se desea, pero validar los campos mínimos que sostienen la operación.

### Desacuerdo 3: rechazar datos contra conservarlos

Pregunta: si un registro tiene unidad inválida, ¿debe eliminarse?

No necesariamente. Debe excluirse del cálculo que exige esa unidad, pero conviene conservarlo como rechazo trazable. Puede revelar una conversión pendiente, un problema de importación o un sistema de origen diferente.

Consenso operativo: separar registros operativos de registros rechazados, sin borrar la evidencia.

### Desacuerdo 4: diccionarios contra clases

Pregunta: ¿por qué no usar clases desde el principio?

Las clases y `dataclass` serán útiles más adelante. En este punto del libro, los diccionarios permiten aprender la estructura sin introducir demasiada sintaxis nueva. La lección importante no es la herramienta exacta; es el contrato del registro.

Consenso operativo: empezar con diccionarios validados; migrar a clases, esquemas o modelos cuando el dominio lo justifique.

## Puente hacia la frontera

Las listas, diccionarios y registros son la antesala de casi todo lo que vendrá. Una tabla clínica es una colección de registros con columnas. Un conjunto de variantes genéticas es una colección de registros con coordenadas, alelos y anotaciones. Un grafo biológico puede representarse inicialmente con diccionarios de listas. Una serie temporal es una lista ordenada de mediciones con tiempo, unidad y estado.

Más adelante, estas ideas crecerán hacia `pandas`, validación de esquemas, índices, búsqueda, hashing, grafos, matrices, datos longitudinales, secuencias biológicas y modelos de representación de conocimiento.

El principio mínimo seguirá igual: antes de recorrer una colección, hay que saber qué contiene; antes de agruparla, hay que validar sus registros; antes de calcular, hay que declarar qué significan sus campos.

## Evaluar si entendiste

1. ¿Cuándo una lista es suficiente para representar datos biomédicos?
2. ¿Por qué una lista posicional puede ser peligrosa para una medición de laboratorio?
3. ¿Qué gana un diccionario al usar claves como `valor`, `unidad` y `estado`?
4. ¿Qué diferencia hay entre diccionario y registro?
5. ¿Por qué un registro debe validarse antes de entrar a una colección operativa?
6. ¿Qué debe ocurrir con un registro rechazado?
7. ¿Por qué agrupar por paciente no responde la misma pregunta que agrupar por prueba?
8. ¿Qué significa que una clave sea estable?
9. ¿Por qué una lista de diccionarios todavía no es una tabla robusta?
10. ¿Qué prueba escribirías para detectar un campo faltante?

## Vacíos de comprensión que debes vigilar

1. Creer que una colección ordenada siempre tiene significado temporal. El orden debe declararse.
2. Pensar que un diccionario valida por sí mismo. Solo nombra campos; no garantiza contrato.
3. Usar registros flexibles sin distinguir exploración, importación y cálculo confiable.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma cinco mediciones biomédicas pequeñas y represéntalas primero como listas posicionales; identifica qué significado queda oculto.
2. **Segunda hora:** conviértelas en diccionarios con campos `paciente_id`, `prueba`, `valor`, `unidad`, `estado` y `razon`.
3. **Tercera hora:** escribe una función que valide campos requeridos, separe registros válidos y rechazados, y agrupe los válidos por paciente.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *Built-in Types: Sequence Types: list, tuple, range*. Python 3 documentation. <https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range>
- Python Software Foundation. (2026). *Built-in Types: Mapping Types: dict*. Python 3 documentation. <https://docs.python.org/3/library/stdtypes.html#mapping-types-dict>
- Python Software Foundation. (2026). *Data Structures*. Python 3 tutorial. <https://docs.python.org/3/tutorial/datastructures.html>
- McKinney, W. (2022). *Python for Data Analysis* (3rd ed.). O'Reilly Media.
- Van Rossum, G., Warsaw, B., & Coghlan, N. (2001). *PEP 8 — Style Guide for Python Code*. Python Enhancement Proposals. <https://peps.python.org/pep-0008/>

## Siguiente paso

Las listas, diccionarios y registros permiten organizar observaciones pequeñas. La siguiente sección llevará esta estructura hacia tablas simples: filas, columnas, limpieza, validación y preparación de datos antes de analizarlos con herramientas más potentes.
