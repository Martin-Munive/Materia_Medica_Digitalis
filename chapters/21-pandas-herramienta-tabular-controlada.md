# `pandas` como herramienta tabular controlada

Hasta ahora el libro ha construido tablas con listas, diccionarios, registros, validadores y esquemas mínimos. Ese recorrido no fue accidental. Antes de usar una herramienta potente, el lector necesitaba entender qué problema resuelve y qué problemas puede ocultar.

`pandas` aparece en este punto porque permite trabajar con datos tabulares de forma expresiva: columnas nombradas, filas indexadas, selección por etiquetas, operaciones por columnas, lectura de archivos y resúmenes. Pero una tabla en `pandas` no deja de ser una tabla biomédica. Sigue necesitando contrato, unidades, estado, ausencias, trazabilidad y denominadores.

La meta de esta sección no es enseñar todo `pandas`. Es introducirlo con disciplina: usarlo como una mesa de trabajo para datos biomédicos, no como una excusa para olvidar lo aprendido sobre representación.

## De registros a `DataFrame`

En `pandas`, una `Series` representa una columna o arreglo unidimensional con etiquetas. Un `DataFrame` representa una estructura bidimensional con filas y columnas. En términos biomédicos, un `DataFrame` puede parecer una hoja de laboratorio, una cohorte, una tabla de eventos, un inventario de muestras o un corte de datos clínicos.

```python
import pandas as pd

registros = [
    {"paciente_id": "P001", "creatinina": 1.1, "unidad": "mg/dL", "estado": "medido"},
    {"paciente_id": "P002", "creatinina": None, "unidad": "mg/dL", "estado": "no_medido"},
    {"paciente_id": "P003", "creatinina": 2.4, "unidad": "mg/dL", "estado": "medido"},
]

tabla = pd.DataFrame(registros)

print(tabla[["paciente_id", "creatinina", "unidad", "estado"]].to_string(index=False))
```

Salida esperada:

```text
paciente_id  creatinina unidad    estado
       P001         1.1  mg/dL    medido
       P002         NaN  mg/dL no_medido
       P003         2.4  mg/dL    medido
```

El cambio técnico es evidente: ya no recorremos manualmente una lista de diccionarios para imprimir o filtrar. El cambio conceptual es más delicado: la tabla no se volvió correcta por estar en `pandas`. Solo se volvió más fácil de manipular.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
En este libro, `pandas` es una herramienta tabular para representar, seleccionar, limpiar, transformar y resumir datos organizados por filas y columnas. Su uso biomédico correcto exige conservar el contrato de cada columna: significado, tipo, unidad, estado, ausencia, fuente y regla de cálculo.
</div>

La biblioteca aporta una estructura potente. El dominio aporta el significado. Si alguno de los dos falta, el resultado puede ser rápido y equivocado.

## Versión ingenua: calcular sobre una columna sin contrato

Supongamos que queremos contar pacientes con creatinina elevada. Una primera tentación es operar directamente sobre la columna.

```python
import pandas as pd

tabla = pd.DataFrame(
    [
        {"paciente_id": "P001", "creatinina": 1.1, "unidad": "mg/dL", "estado": "medido"},
        {"paciente_id": "P002", "creatinina": None, "unidad": "mg/dL", "estado": "no_medido"},
        {"paciente_id": "P003", "creatinina": 2.4, "unidad": "mg/dL", "estado": "medido"},
        {"paciente_id": "P004", "creatinina": 999.0, "unidad": "mg/dL", "estado": "pendiente"},
    ]
)

elevadas = tabla["creatinina"] > 1.2

print(elevadas.to_string(index=False))
print(int(elevadas.sum()))
```

Salida esperada:

```text
False
False
 True
 True
2
```

El cálculo terminó. También cometió un error de dominio: contó `999.0` como creatinina elevada, aunque en esta tabla ese valor acompaña un estado `pendiente`. La operación fue sintácticamente correcta, pero no respetó el contrato biomédico del dato.

Este es uno de los riesgos centrales de `pandas`: al facilitar operaciones masivas, también facilita errores masivos.

## Crítica técnica: `DataFrame` no equivale a dato limpio

Primero, cada columna tiene un tipo técnico, pero ese tipo no expresa todo el significado. Una columna numérica puede contener mediciones reales, valores centinela, conversiones pendientes o errores de captura.

Segundo, las ausencias tienen semántica. Un `NaN` puede significar no medido, no aplicable, pendiente, ilegible, censurado o perdido. Si esas razones no viven en otra columna, el cálculo no puede distinguirlas.

Tercero, el denominador no se deduce solo. Contar sobre todos los pacientes, sobre pacientes medidos, sobre pacientes elegibles o sobre filas válidas produce resultados diferentes.

Cuarto, la selección de filas debe ser explícita. `pandas` permite máscaras booleanas, pero la máscara debe expresar una regla de dominio, no solo una comparación numérica.

## CODE CLEAN: máscara de validez antes del cálculo

La versión mejorada separa cuatro pasos:

1. construir una tabla;
2. declarar qué filas son interpretables para el cálculo;
3. calcular solo sobre esas filas;
4. reportar numerador y denominador.

```python
import pandas as pd

tabla = pd.DataFrame(
    [
        {"paciente_id": "P001", "creatinina": 1.1, "unidad": "mg/dL", "estado": "medido"},
        {"paciente_id": "P002", "creatinina": None, "unidad": "mg/dL", "estado": "no_medido"},
        {"paciente_id": "P003", "creatinina": 2.4, "unidad": "mg/dL", "estado": "medido"},
        {"paciente_id": "P004", "creatinina": 999.0, "unidad": "mg/dL", "estado": "pendiente"},
    ]
)

fila_medida = tabla["estado"].eq("medido")
unidad_esperada = tabla["unidad"].eq("mg/dL")
valor_presente = tabla["creatinina"].notna()

fila_valida = fila_medida & unidad_esperada & valor_presente
filas_calculables = tabla.loc[fila_valida].copy()

filas_calculables["creatinina_elevada"] = filas_calculables["creatinina"] > 1.2

numerador = int(filas_calculables["creatinina_elevada"].sum())
denominador = int(fila_valida.sum())

print(f"filas calculables: {denominador}")
print(f"creatinina elevada: {numerador}/{denominador}")
print(
    filas_calculables[
        ["paciente_id", "creatinina", "unidad", "creatinina_elevada"]
    ].to_string(index=False)
)
```

Salida esperada:

```text
filas calculables: 2
creatinina elevada: 1/2
paciente_id  creatinina unidad  creatinina_elevada
       P001         1.1  mg/dL               False
       P003         2.4  mg/dL                True
```

La diferencia no es estética. La segunda versión permite defender qué se contó, qué se excluyó y por qué.

## Selección por columnas y filas

`pandas` ofrece varias formas de seleccionar datos. Para código que debe leerse con claridad, conviene distinguir dos accesos:

**Por etiqueta con `.loc`.** Selecciona filas y columnas por nombre o máscara.

```python
import pandas as pd

tabla = pd.DataFrame(
    [
        {"paciente_id": "P001", "estado": "medido", "creatinina": 1.1},
        {"paciente_id": "P002", "estado": "no_medido", "creatinina": None},
        {"paciente_id": "P003", "estado": "medido", "creatinina": 2.4},
    ]
)

medidas = tabla["estado"].eq("medido")
vista = tabla.loc[medidas, ["paciente_id", "creatinina"]]

print(vista.to_string(index=False))
```

Salida esperada:

```text
paciente_id  creatinina
       P001         1.1
       P003         2.4
```

**Por posición con `.iloc`.** Selecciona por ubicación numérica. Puede ser útil para inspección, pero en datos biomédicos suele ser menos expresivo que seleccionar por nombres estables.

```python
import pandas as pd

tabla = pd.DataFrame(
    [
        {"paciente_id": "P001", "estado": "medido", "creatinina": 1.1},
        {"paciente_id": "P002", "estado": "no_medido", "creatinina": None},
        {"paciente_id": "P003", "estado": "medido", "creatinina": 2.4},
    ]
)

print(tabla.iloc[0, 0])
```

Salida esperada:

```text
P001
```

La regla práctica de este libro será simple: para lógica de dominio, preferir nombres de columnas y máscaras explícitas. La posición puede cambiar cuando llega un archivo nuevo; el significado no debería depender de que una columna siga siendo la tercera.

## Lectura de CSV como adquisición, no como interpretación

Leer un archivo no equivale a validarlo. `read_csv` convierte texto en tabla, pero no puede saber por sí mismo si una columna representa una unidad compatible, un estado permitido o un centinela.

```python
from io import StringIO

import pandas as pd

contenido = StringIO(
    """paciente_id,creatinina,unidad,estado
P001,1.1,mg/dL,medido
P002,,mg/dL,no_medido
P003,212,umol/L,medido
P004,999,mg/dL,pendiente
"""
)

tabla = pd.read_csv(
    contenido,
    dtype={"paciente_id": "string", "unidad": "string", "estado": "string"},
)

tabla["creatinina"] = pd.to_numeric(tabla["creatinina"], errors="coerce")

print(tabla.dtypes[["paciente_id", "creatinina", "unidad", "estado"]].to_string())
```

Salida esperada:

```text
paciente_id     string
creatinina     float64
unidad          string
estado          string
```

El archivo ya está en memoria como tabla. Todavía no está listo para una conclusión biomédica. El siguiente paso debe ser validar y normalizar.

## Normalización mínima de unidades

Una operación frecuente en datos biomédicos es convertir unidades para llevar mediciones a una escala común. La conversión debe aplicarse solo a filas cuyo estado y unidad permitan interpretarla.

```python
from io import StringIO

import pandas as pd

contenido = StringIO(
    """paciente_id,creatinina,unidad,estado
P001,1.1,mg/dL,medido
P002,,mg/dL,no_medido
P003,212,umol/L,medido
P004,999,mg/dL,pendiente
"""
)

tabla = pd.read_csv(
    contenido,
    dtype={"paciente_id": "string", "unidad": "string", "estado": "string"},
)
tabla["creatinina"] = pd.to_numeric(tabla["creatinina"], errors="coerce")

medida = tabla["estado"].eq("medido")
en_mg_dl = tabla["unidad"].eq("mg/dL")
en_umol_l = tabla["unidad"].eq("umol/L")
valor_presente = tabla["creatinina"].notna()

tabla["creatinina_mg_dl"] = pd.NA
tabla.loc[medida & en_mg_dl & valor_presente, "creatinina_mg_dl"] = tabla.loc[
    medida & en_mg_dl & valor_presente, "creatinina"
]
tabla.loc[medida & en_umol_l & valor_presente, "creatinina_mg_dl"] = (
    tabla.loc[medida & en_umol_l & valor_presente, "creatinina"] / 88.4
)

tabla["calculable"] = tabla["creatinina_mg_dl"].notna()

print(
    tabla[
        ["paciente_id", "creatinina", "unidad", "estado", "creatinina_mg_dl", "calculable"]
    ].to_string(index=False)
)
```

Salida esperada:

```text
paciente_id  creatinina unidad    estado creatinina_mg_dl  calculable
       P001         1.1  mg/dL    medido              1.1        True
       P002         NaN  mg/dL no_medido             <NA>       False
       P003       212.0 umol/L    medido         2.39819        True
       P004       999.0  mg/dL pendiente             <NA>       False
```

El valor `999` ya no contamina el cálculo. No fue eliminado; quedó visible como fila no calculable por estado `pendiente`.

## Agrupar no es concluir

`pandas` facilita agrupar por categorías y calcular resúmenes. Pero cada resumen necesita saber qué filas entraron.

```python
import pandas as pd

tabla = pd.DataFrame(
    [
        {"servicio": "urgencias", "estado": "medido", "creatinina_mg_dl": 1.0},
        {"servicio": "urgencias", "estado": "medido", "creatinina_mg_dl": 2.1},
        {"servicio": "urgencias", "estado": "pendiente", "creatinina_mg_dl": None},
        {"servicio": "consulta", "estado": "medido", "creatinina_mg_dl": 0.9},
    ]
)

calculable = tabla["estado"].eq("medido") & tabla["creatinina_mg_dl"].notna()

resumen = (
    tabla.loc[calculable]
    .groupby("servicio", as_index=False)
    .agg(
        pacientes_medidos=("creatinina_mg_dl", "size"),
        creatinina_promedio=("creatinina_mg_dl", "mean"),
    )
)

print(resumen.to_string(index=False))
```

Salida esperada:

```text
 servicio  pacientes_medidos  creatinina_promedio
 consulta                  1                 0.90
urgencias                  2                 1.55
```

El resumen no dice “promedio de todos los pacientes”. Dice promedio de pacientes medidos, porque ese fue el denominador declarado.

## Errores frecuentes

**Convertir todo a número demasiado pronto.** Si se fuerza una columna a número sin conservar el texto original, pueden desaparecer razones de error, unidades, símbolos o estados.

**Confundir `NaN` con una explicación clínica.** `NaN` indica ausencia técnica de valor, no su razón.

**Usar `dropna` sin justificar.** Eliminar filas faltantes puede ser razonable para una operación, pero debe dejar claro qué denominador queda.

**Usar posiciones como contrato.** `tabla.iloc[:, 3]` puede funcionar hoy y fallar mañana si el archivo cambia de orden.

**Calcular antes de validar.** El cálculo debe recibir filas interpretables, no archivos recién leídos.

## Puente hacia bases de datos, APIs y reproducibilidad

`pandas` no reemplaza una base de datos. Es una herramienta excelente para explorar, limpiar, transformar y resumir lotes tabulares, pero no gobierna por sí sola persistencia, concurrencia, relaciones, permisos, migraciones ni auditoría histórica.

En una arquitectura biomédica seria, `pandas` puede ocupar un lugar claro:

- adquirir un lote desde CSV, Excel, SQL o una API;
- inspeccionar columnas, tipos, ausencias y valores inesperados;
- limpiar y normalizar datos bajo reglas explícitas;
- producir tablas derivadas verificables;
- exportar resultados hacia una base de datos, reporte o análisis reproducible.

La siguiente transición natural del libro será separar tres responsabilidades: archivo de entrada, tabla de trabajo y almacenamiento persistente.

## Preguntas de comprensión profunda

1. ¿Por qué una operación vectorizada sobre una columna puede ser técnicamente correcta y biomédicamente incorrecta?
2. ¿Qué diferencia hay entre `NaN`, `no_medido`, `pendiente` y `no_aplica`?
3. ¿Por qué `.loc` suele ser más expresivo que `.iloc` para reglas de dominio?
4. ¿Qué denominador usarías para reportar proporción de creatininas elevadas si el 30% de las filas no tiene medición?
5. ¿Qué información perderías si eliminas filas faltantes antes de revisar su estado?
6. ¿Cuándo conviene usar `pandas` y cuándo conviene pasar a una base de datos?

## Bibliografía y fuentes

- The pandas development team. [pandas User Guide](https://pandas.pydata.org/docs/user_guide/).
- The pandas development team. [10 minutes to pandas](https://pandas.pydata.org/docs/user_guide/10min.html).
- The pandas development team. [Intro to data structures](https://pandas.pydata.org/docs/user_guide/dsintro.html).
- The pandas development team. [Indexing and selecting data](https://pandas.pydata.org/docs/user_guide/indexing.html).
- The pandas development team. [IO tools](https://pandas.pydata.org/docs/user_guide/io.html).
