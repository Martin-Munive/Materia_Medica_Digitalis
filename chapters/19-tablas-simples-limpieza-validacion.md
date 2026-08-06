# Tablas simples, limpieza y validación

Una lista de registros ya puede parecer una tabla, pero una tabla responsable exige más que filas acumuladas. En medicina y ciencias de la vida, una tabla puede representar pacientes, muestras, eventos, mediciones, variantes, procedimientos, fármacos, sedes, seguimientos o resultados. Cada fila parece pequeña. El problema aparece cuando muchas filas arrastran columnas mal nombradas, unidades mezcladas, valores faltantes, duplicados y tipos inconsistentes.

La sección anterior mostró cómo un registro necesita campos nombrados. Esta sección pregunta qué ocurre cuando muchos registros se convierten en una estructura tabular. El objetivo no es aprender todavía `pandas` en profundidad. El objetivo es entender qué promesas debe cumplir una tabla antes de calcular sobre ella.

## Origen técnico: una tabla es filas, columnas y contrato

Una tabla organiza observaciones en filas y variables en columnas. En una hoja de cálculo se ve como una cuadrícula. En Python puede empezar como una lista de diccionarios.

```python
tabla = [
    {"paciente_id": "P001", "prueba": "creatinina", "valor": "1.8", "unidad": "mg/dL"},
    {"paciente_id": "P002", "prueba": "creatinina", "valor": "0.9", "unidad": "mg/dL"},
]

print(tabla[0]["paciente_id"])
print(tabla[1]["valor"])
```

Salida esperada:

```text
P001
0.9
```

Aunque `valor` parece numérico, llega como texto. Eso es común al leer CSV, formularios, hojas de cálculo o exportaciones. El programa no debe asumir que una columna que se llama `valor` ya tiene el tipo correcto.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una tabla biomédica simple es una colección de filas con columnas nombradas y contrato explícito. Limpiar una tabla no es embellecerla: es transformar valores crudos en valores interpretables, separar registros válidos de registros rechazados y conservar razones de exclusión antes de calcular.
</div>

Esta definición separa cinco piezas.

**Fila.** Una observación: un paciente, una muestra, una medición, un evento.

**Columna.** Una variable o campo compartido por las filas: `paciente_id`, `prueba`, `valor`, `unidad`.

**Esquema.** Conjunto mínimo de columnas, tipos esperados, unidades permitidas y reglas de validez.

**Limpieza.** Transformación documentada de valores crudos: espacios, mayúsculas, separadores, números como texto, símbolos de ausencia.

**Validación.** Decisión explícita sobre si una fila cumple el contrato necesario para una operación.

Una tabla puede estar limpia para una pregunta y no para otra. Por eso el contrato depende del uso posterior.

## Versión ingenua: sumar una columna sin limpiar

Supongamos que importamos creatininas desde un archivo. Los valores vienen como texto porque muchas fuentes tabulares leen todo inicialmente como cadena.

```python
filas = [
    {"paciente_id": "P001", "valor": "1.8"},
    {"paciente_id": "P002", "valor": "0.9"},
    {"paciente_id": "P003", "valor": ""},
]

total = 0
for fila in filas:
    total = total + fila["valor"]

print(total)
```

Salida esperada:

```text
TypeError
```

El error técnico evita un cálculo inválido, pero llega tarde. El programa intentó sumar texto. Una reacción rápida podría ser convertir todo con `float()`.

```python
filas = [
    {"paciente_id": "P001", "valor": "1.8"},
    {"paciente_id": "P002", "valor": "0.9"},
    {"paciente_id": "P003", "valor": ""},
]

valores = [float(fila["valor"]) for fila in filas]
print(sum(valores) / len(valores))
```

Salida esperada:

```text
ValueError
```

Ahora el problema es la cadena vacía. Python detecta que `""` no puede convertirse en número. Pero una fuente real puede traer `"NA"`, `"no aplica"`, `"pendiente"`, `"1,8"`, `" 1.8 "`, `"mg/dL 1.8"` o un número imposible. La limpieza no puede reducirse a `float()`.

## Crítica técnica: qué está mal

Primero, la tabla no declara columnas requeridas. Si falta `unidad`, el cálculo puede seguir sin saber si los valores son comparables.

Segundo, los tipos llegan crudos. Una celda textual puede representar número, ausencia, comentario, error de captura o separador decimal local.

Tercero, no hay unidad. Promediar creatinina sin unidad puede mezclar `mg/dL` y `umol/L`.

Cuarto, no se separan filas válidas y rechazadas. Si una fila falla, el programa se detiene o la elimina sin conservar razón.

Quinto, no hay conteo de limpieza. No sabemos cuántas filas fueron aceptadas, cuántas faltaron, cuántas tenían unidad incompatible ni cuántas eran imposibles.

Sexto, no hay regla de duplicados. Dos filas del mismo paciente pueden ser mediciones distintas, repeticiones válidas o duplicados accidentales.

## Versión mejorada: limpiar celda, validar fila, resumir tabla

Construyamos una miniatura pedagógica. No es una regla clínica ni un sistema de laboratorio. Es una forma de aprender cómo una tabla entra a un cálculo sin mentir.

```python
from enum import Enum


class EstadoFila(Enum):
    VALIDA = "valida"
    FALTAN_COLUMNAS = "faltan_columnas"
    VALOR_AUSENTE = "valor_ausente"
    VALOR_NO_NUMERICO = "valor_no_numerico"
    UNIDAD_INVALIDA = "unidad_invalida"
    VALOR_INVALIDO = "valor_invalido"
    PRUEBA_NO_SOPORTADA = "prueba_no_soportada"


COLUMNAS_REQUERIDAS = {"paciente_id", "prueba", "valor", "unidad"}


ESQUEMA_CREATININA = {
    "prueba": "creatinina",
    "unidad": "mg/dL",
    "minimo": 0,
    "maximo": 25,
}


MARCADORES_AUSENCIA = {"", "NA", "N/A", "SIN DATO", "PENDIENTE"}


def limpiar_texto(valor):
    """Normaliza texto crudo para comparación básica."""
    if valor is None:
        return ""

    return str(valor).strip().upper()


def convertir_numero(valor_crudo):
    """Convierte una celda cruda a número o devuelve una razón de falla."""
    texto = limpiar_texto(valor_crudo)

    if texto in MARCADORES_AUSENCIA:
        return {
            "estado": EstadoFila.VALOR_AUSENTE,
            "valor": None,
            "razon": "marcador_de_ausencia",
        }

    texto = texto.replace(",", ".")

    try:
        numero = float(texto)
    except ValueError:
        return {
            "estado": EstadoFila.VALOR_NO_NUMERICO,
            "valor": None,
            "razon": "conversion_float_fallida",
        }

    return {
        "estado": EstadoFila.VALIDA,
        "valor": numero,
        "razon": "numero_convertido",
    }


def validar_fila_creatinina(fila):
    """Valida una fila tabular de creatinina antes de usarla en cálculos."""
    faltantes = COLUMNAS_REQUERIDAS - set(fila)
    if faltantes:
        return {
            "estado": EstadoFila.FALTAN_COLUMNAS,
            "fila": dict(fila),
            "razon": f"faltan_columnas:{sorted(faltantes)}",
        }

    if fila["prueba"] != ESQUEMA_CREATININA["prueba"]:
        return {
            "estado": EstadoFila.PRUEBA_NO_SOPORTADA,
            "fila": dict(fila),
            "razon": "prueba_no_corresponde_a_este_esquema",
        }

    if fila["unidad"] != ESQUEMA_CREATININA["unidad"]:
        return {
            "estado": EstadoFila.UNIDAD_INVALIDA,
            "fila": dict(fila),
            "razon": "unidad_no_comparable",
        }

    conversion = convertir_numero(fila["valor"])
    if conversion["estado"] != EstadoFila.VALIDA:
        return {
            "estado": conversion["estado"],
            "fila": dict(fila),
            "razon": conversion["razon"],
        }

    valor = conversion["valor"]
    if valor < ESQUEMA_CREATININA["minimo"] or valor > ESQUEMA_CREATININA["maximo"]:
        return {
            "estado": EstadoFila.VALOR_INVALIDO,
            "fila": dict(fila),
            "razon": "valor_fuera_de_rango_pedagogico",
        }

    fila_limpia = {
        "paciente_id": fila["paciente_id"],
        "prueba": fila["prueba"],
        "valor": valor,
        "unidad": fila["unidad"],
        "estado": EstadoFila.VALIDA,
        "razon": "fila_aceptada",
    }

    return {
        "estado": EstadoFila.VALIDA,
        "fila": fila_limpia,
        "razon": "cumple_esquema_minimo",
    }
```

El validador trabaja por capas. Primero revisa columnas. Luego prueba y unidad. Después convierte número. Finalmente aplica un rango pedagógico.

```python
filas_crudas = [
    {"paciente_id": "P001", "prueba": "creatinina", "valor": "1.8", "unidad": "mg/dL"},
    {"paciente_id": "P002", "prueba": "creatinina", "valor": "0,9", "unidad": "mg/dL"},
    {"paciente_id": "P003", "prueba": "creatinina", "valor": "", "unidad": "mg/dL"},
    {"paciente_id": "P004", "prueba": "creatinina", "valor": "160", "unidad": "umol/L"},
]

resultados = [validar_fila_creatinina(fila) for fila in filas_crudas]

print(resultados[0]["estado"].value)
print(resultados[1]["fila"]["valor"])
print(resultados[2]["estado"].value)
print(resultados[3]["estado"].value)
```

Salida esperada:

```text
valida
0.9
valor_ausente
unidad_invalida
```

El valor `"0,9"` se normalizó a `0.9`. La cadena vacía no se convirtió en cero. La unidad incompatible no entró al cálculo.

## Resumir una tabla sin borrar los rechazos

Una tabla limpia no debería devolver solo filas válidas. También debe devolver trazabilidad de lo excluido.

```python
def resumir_tabla_creatinina(filas):
    """Separa filas válidas y rechazadas, calcula promedio y conserva conteos."""
    resultados = [validar_fila_creatinina(fila) for fila in filas]
    validas = [
        resultado["fila"]
        for resultado in resultados
        if resultado["estado"] == EstadoFila.VALIDA
    ]
    rechazadas = [
        resultado
        for resultado in resultados
        if resultado["estado"] != EstadoFila.VALIDA
    ]

    conteo_por_estado = {}
    for resultado in resultados:
        estado = resultado["estado"].value
        conteo_por_estado[estado] = conteo_por_estado.get(estado, 0) + 1

    if not validas:
        return {
            "estado": "no_calculable",
            "promedio": None,
            "unidad": ESQUEMA_CREATININA["unidad"],
            "validas": validas,
            "rechazadas": rechazadas,
            "conteo_por_estado": conteo_por_estado,
            "razon": "sin_filas_validas",
        }

    promedio = sum(fila["valor"] for fila in validas) / len(validas)

    return {
        "estado": "calculable",
        "promedio": promedio,
        "unidad": ESQUEMA_CREATININA["unidad"],
        "validas": validas,
        "rechazadas": rechazadas,
        "conteo_por_estado": conteo_por_estado,
        "razon": "promedio_sobre_filas_validas",
    }


resumen = resumir_tabla_creatinina(filas_crudas)
print(resumen["estado"])
print(round(resumen["promedio"], 2), resumen["unidad"])
print(resumen["conteo_por_estado"])
```

Salida esperada:

```text
calculable
1.35 mg/dL
{'valida': 2, 'valor_ausente': 1, 'unidad_invalida': 1}
```

El promedio ya no oculta la limpieza. La tabla informa cuántas filas entraron y cuántas quedaron fuera.

## Leer CSV no equivale a entender una tabla

El formato CSV es ubicuo porque muchas hojas de cálculo, bases de datos y sistemas pueden exportarlo. Pero CSV no tiene un estándar único de uso cotidiano: delimitadores, comillas, saltos de línea y convenciones cambian entre fuentes. Además, al leer CSV con el módulo estándar de Python, las filas llegan como texto salvo conversiones explícitas.

```python
import csv
from io import StringIO


contenido = """paciente_id,prueba,valor,unidad
P001,creatinina,1.8,mg/dL
P002,creatinina,"0,9",mg/dL
"""

archivo = StringIO(contenido)
lector = csv.DictReader(archivo)
filas = list(lector)

print(filas[0]["valor"])
print(type(filas[0]["valor"]))
```

Salida esperada:

```text
1.8
<class 'str'>
```

`DictReader` ayuda porque convierte cada fila en diccionario con claves tomadas del encabezado. Pero no valida tipos, unidades ni significado. La lectura es adquisición. La interpretación empieza después.

## Anatomía del contrato tabular

El contrato mínimo de una tabla declara columnas requeridas.

```python
COLUMNAS_REQUERIDAS = {"paciente_id", "prueba", "valor", "unidad"}
```

También declara unidades.

```python
"unidad": "mg/dL"
```

Declara conversión.

```python
texto = texto.replace(",", ".")
numero = float(texto)
```

Declara límites.

```python
if valor < 0 or valor > 25:
    ...
```

Y declara salida.

```python
"conteo_por_estado": conteo_por_estado
```

Una tabla sin contrato puede verse ordenada. Una tabla con contrato puede defender por qué cada fila entró o no entró al cálculo.

## Pruebas mínimas

```python
# Propiedad 1: un número con coma decimal se convierte de forma controlada.
conversion = convertir_numero("0,9")
assert conversion["estado"] == EstadoFila.VALIDA
assert conversion["valor"] == 0.9

# Propiedad 2: una cadena vacía no se convierte en cero.
ausente = convertir_numero("")
assert ausente["estado"] == EstadoFila.VALOR_AUSENTE
assert ausente["valor"] is None

# Propiedad 3: una unidad incompatible no entra como fila válida.
unidad_errada = validar_fila_creatinina({
    "paciente_id": "P004",
    "prueba": "creatinina",
    "valor": "160",
    "unidad": "umol/L",
})
assert unidad_errada["estado"] == EstadoFila.UNIDAD_INVALIDA

# Propiedad 4: el promedio usa solo filas válidas y conserva rechazos.
resumen = resumir_tabla_creatinina(filas_crudas)
assert round(resumen["promedio"], 2) == 1.35
assert len(resumen["validas"]) == 2
assert len(resumen["rechazadas"]) == 2

# Propiedad 5: sin filas válidas no se inventa promedio.
sin_validas = resumir_tabla_creatinina([
    {"paciente_id": "P003", "prueba": "creatinina", "valor": "", "unidad": "mg/dL"},
])
assert sin_validas["estado"] == "no_calculable"
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas no buscan cubrir todo. Buscan proteger fronteras estructurales: conversión, ausencia, unidad, promedio y ausencia total de filas válidas.

## CODE CLEAN: separar importación, limpieza, validación y cálculo

Una versión frágil suele mezclar todo.

```python
promedio = sum(float(fila["valor"]) for fila in filas) / len(filas)
```

La línea es compacta, pero oculta preguntas críticas:

- ¿los valores están en la misma unidad?
- ¿hay ausentes?
- ¿hay texto no numérico?
- ¿el denominador son todas las filas o solo las válidas?
- ¿qué filas fueron excluidas?

Una versión más limpia separa responsabilidades.

```python
filas = leer_csv()
resultados = [validar_fila_creatinina(fila) for fila in filas]
resumen = resumir_tabla_creatinina(filas)
```

En un programa real, `leer_csv`, `validar_fila_creatinina` y `resumir_tabla_creatinina` deberían ser piezas separadas. Esto permite probar la limpieza sin leer archivos, probar la validación sin calcular y probar el cálculo con filas ya validadas.

Código limpio no significa escribir muchas funciones por costumbre. Significa que cada función responda una pregunta verificable.

## Límites y errores frecuentes

1. **Confundir tabla visible con tabla válida.** Que una cuadrícula se vea ordenada no significa que sus columnas cumplan contrato.
2. **Confiar en nombres de columnas sin validar contenido.** Una columna llamada `valor` puede contener texto, ausencias o unidades pegadas.
3. **Promediar antes de declarar unidad.** El promedio puede mezclar magnitudes incompatibles.
4. **Convertir ausencias en cero.** `""`, `"NA"` o `"pendiente"` no deben transformarse en fisiología.
5. **Eliminar filas sin conservar razón.** La limpieza silenciosa impide auditar el resultado.
6. **Usar el total de filas como denominador automático.** A veces el denominador correcto son filas válidas, documentadas o elegibles.
7. **Ignorar duplicados.** Dos filas pueden ser mediciones repetidas o duplicados accidentales; no se decide sin regla.
8. **Mezclar importación con cálculo.** Hace difícil probar y depurar.
9. **Asumir que CSV conserva tipos.** Lo habitual es leer texto y convertir después.
10. **No registrar versión de esquema.** Si la regla cambia, las salidas antiguas pierden trazabilidad.

## Argumentos críticos

### Desacuerdo 1: limpiar rápido contra validar formalmente

Pregunta: ¿por qué no limpiar rápido y seguir?

Porque una limpieza rápida puede resolver el bloqueo técnico y crear un problema interpretativo. Convertir todo a número, eliminar ausentes y calcular puede producir una salida elegante sin informar qué se perdió.

Consenso operativo: limpiar lo necesario para interpretar, validar contra un contrato y conservar rechazos.

### Desacuerdo 2: fila inválida contra dato corregible

Pregunta: si la unidad es `umol/L`, ¿la fila es inválida o debe convertirse?

Depende del contrato. En una miniatura pedagógica conservadora la rechazamos para no mezclar unidades. En un sistema real podría existir una conversión validada, trazable y probada. Lo incorrecto es convertir sin declararlo.

Consenso operativo: rechazar si no hay regla explícita de conversión; convertir solo con unidad origen, unidad destino, fórmula, versión y prueba.

### Desacuerdo 3: estándar mínimo contra flexibilidad exploratoria

Pregunta: ¿un esquema rígido no bloquea exploración?

La exploración necesita flexibilidad. El cálculo responsable necesita contrato. No son etapas enemigas. Primero se explora para entender la fuente; luego se fija el contrato de la operación.

Consenso operativo: distinguir exploración, limpieza, validación y análisis.

### Desacuerdo 4: listas de diccionarios contra DataFrame

Pregunta: ¿por qué no pasar directamente a `pandas`?

`pandas` será necesario para tablas reales, análisis, filtrado y limpieza más potente. Pero antes de usar una herramienta poderosa conviene entender el contrato que debe cumplir la tabla. Un `DataFrame` no vuelve correcto un dato mal definido.

Consenso operativo: aprender primero el contrato tabular; después usar `pandas` como herramienta, no como sustituto del razonamiento.

## Puente hacia la frontera

Las tablas son el punto de entrada a gran parte de la medicina computacional. Historias clínicas electrónicas, cohortes, registros de laboratorio, ensayos clínicos, datos ómicos preprocesados, fenotipos computacionales y matrices de seguimiento suelen pasar por alguna forma tabular antes de convertirse en modelos, señales, grafos o secuencias.

Más adelante, el libro podrá escalar hacia `pandas`, esquemas formales, validación de columnas, auditoría de calidad, duplicados, joins, claves primarias, bases SQL, cohortes longitudinales, datasets FAIR y pipelines reproducibles.

El principio mínimo seguirá igual: una tabla no está lista porque abrió. Está lista cuando sus columnas, tipos, unidades, ausencias, denominadores y rechazos tienen contrato.

## Evaluar si entendiste

1. ¿Qué diferencia hay entre una lista de registros y una tabla validada?
2. ¿Por qué leer CSV no equivale a interpretar una tabla?
3. ¿Qué debe contener un esquema mínimo para una tabla biomédica?
4. ¿Por qué una cadena vacía no debe convertirse en cero?
5. ¿Cuándo una unidad incompatible debería rechazarse?
6. ¿Qué significa conservar filas rechazadas?
7. ¿Por qué el denominador de una tabla no siempre es el total de filas?
8. ¿Qué prueba escribirías para evitar que `"0,9"` falle por separador decimal?
9. ¿Por qué `pandas` no reemplaza la validación de dominio?
10. ¿Qué información debe acompañar un promedio calculado desde una tabla con rechazos?

## Vacíos de comprensión que debes vigilar

1. Pensar que limpiar datos es hacer que el código no falle. La meta es preservar significado.
2. Creer que una tabla tabularmente completa está biomédicamente completa. Puede tener todas las celdas llenas y aun así mezclar unidades.
3. Usar herramientas potentes antes de declarar el contrato mínimo de la operación.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** crea una tabla pequeña con cinco filas y errores deliberados: valor ausente, unidad incompatible, coma decimal, columna faltante y texto no numérico.
2. **Segunda hora:** escribe funciones separadas para limpiar texto, convertir número y validar fila.
3. **Tercera hora:** resume la tabla con promedio, filas válidas, filas rechazadas y conteo por estado.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *csv: CSV File Reading and Writing*. Python 3 documentation. <https://docs.python.org/3/library/csv.html>
- pandas development team. (2026). *Intro to data structures*. pandas documentation. <https://pandas.pydata.org/docs/user_guide/dsintro.html>
- pandas development team. (2026). *Working with missing data*. pandas documentation. <https://pandas.pydata.org/docs/user_guide/missing_data.html>
- Wickham, H., & Grolemund, G. (2017). *R for Data Science*. O'Reilly Media.
- Wilkinson, M. D., et al. (2016). The FAIR Guiding Principles for scientific data management and stewardship. *Scientific Data, 3*, 160018. <https://doi.org/10.1038/sdata.2016.18>

## Siguiente paso

Las tablas simples obligan a declarar columnas, tipos, unidades y rechazos. La siguiente sección seguirá con esquemas mínimos y validación formal: cómo pasar de validadores hechos a mano a contratos más explícitos, reutilizables y preparados para crecer hacia bases de datos, APIs y análisis reproducibles.
