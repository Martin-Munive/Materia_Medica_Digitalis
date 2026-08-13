# Archivos, tablas de trabajo y almacenamiento persistente

La sección anterior introdujo `pandas` como herramienta tabular controlada. Esa precisión importa porque una tabla de trabajo no es una base de datos, y un archivo no es una tabla de trabajo. Confundir esas capas produce sistemas frágiles: se recalcula desde archivos sueltos, se pierden reglas de limpieza, se mezclan datos crudos con datos validados y no queda claro qué resultado fue usado para tomar una decisión.

Este capítulo separa tres responsabilidades:

**Archivo de entrada.** Material recibido desde una fuente: CSV, Excel, descarga, exportación de laboratorio, formulario o API. Puede estar incompleto, mal tipado o contener convenciones locales.

**Tabla de trabajo.** Estructura temporal para inspeccionar, limpiar, validar y transformar datos. En este libro, muchas veces será un `DataFrame`.

**Almacenamiento persistente.** Lugar donde se conserva una versión organizada y recuperable del dato o del resultado: por ejemplo, una base SQLite.

La separación parece administrativa. En realidad es conceptual: cada capa responde una pregunta distinta.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
El almacenamiento persistente es la conservación estructurada de datos más allá de una ejecución del programa. En un flujo biomédico mínimo, los archivos se tratan como entrada, las tablas de trabajo como espacio de validación y transformación, y la base de datos como registro consultable de datos aceptados, rechazados o derivados bajo reglas explícitas.
</div>

No todo dato debe ir a una base de datos. Pero cuando un dato va a consultarse, auditarse, acumularse o relacionarse con otros datos, guardarlo como archivo suelto deja de ser suficiente.

## Un flujo mínimo

El flujo que usaremos será:

1. leer un CSV;
2. convertir tipos técnicos;
3. declarar filas calculables;
4. separar aceptados y rechazados;
5. guardar el resultado en SQLite;
6. recuperar una vista tabular para análisis.

El objetivo no es construir todavía una arquitectura clínica. Es aprender la frontera entre adquisición, transformación y persistencia.

## Archivo de entrada: texto con forma tabular

Un CSV es texto. Puede tener encabezados y separadores, pero no trae por sí mismo el contrato completo del dato.

```python
from io import StringIO

contenido = StringIO(
    """paciente_id,servicio,creatinina,unidad,estado
P001,urgencias,1.1,mg/dL,medido
P002,urgencias,,mg/dL,no_medido
P003,consulta,212,umol/L,medido
P004,urgencias,999,mg/dL,pendiente
"""
)

print(contenido.getvalue().splitlines()[0])
```

Salida esperada:

```text
paciente_id,servicio,creatinina,unidad,estado
```

El encabezado ayuda, pero no explica si `999` es real, centinela, error o pendiente. Esa semántica debe incorporarse después.

## Tabla de trabajo: lectura y normalización

Ahora leemos el archivo en una tabla de trabajo. La tabla no es todavía la fuente de verdad; es el lugar donde hacemos explícitas las reglas.

```python
from io import StringIO

import pandas as pd

contenido = StringIO(
    """paciente_id,servicio,creatinina,unidad,estado
P001,urgencias,1.1,mg/dL,medido
P002,urgencias,,mg/dL,no_medido
P003,consulta,212,umol/L,medido
P004,urgencias,999,mg/dL,pendiente
"""
)

tabla = pd.read_csv(
    contenido,
    dtype={
        "paciente_id": "string",
        "servicio": "string",
        "unidad": "string",
        "estado": "string",
    },
)
tabla["creatinina"] = pd.to_numeric(tabla["creatinina"], errors="coerce")

print(tabla.dtypes[["paciente_id", "servicio", "creatinina", "unidad", "estado"]].to_string())
```

Salida esperada:

```text
paciente_id     string
servicio        string
creatinina     float64
unidad          string
estado          string
```

La conversión técnica ya ocurrió. Falta la validación de dominio.

## Separar aceptados y rechazados

La base de datos no debe recibir una mezcla silenciosa de filas interpretables y filas problemáticas. Conviene producir dos salidas: una tabla limpia para cálculo y una tabla de rechazo con razón.

```python
from io import StringIO

import pandas as pd

contenido = StringIO(
    """paciente_id,servicio,creatinina,unidad,estado
P001,urgencias,1.1,mg/dL,medido
P002,urgencias,,mg/dL,no_medido
P003,consulta,212,umol/L,medido
P004,urgencias,999,mg/dL,pendiente
"""
)

tabla = pd.read_csv(
    contenido,
    dtype={
        "paciente_id": "string",
        "servicio": "string",
        "unidad": "string",
        "estado": "string",
    },
)
tabla["creatinina"] = pd.to_numeric(tabla["creatinina"], errors="coerce")

medido = tabla["estado"].eq("medido")
unidad_valida = tabla["unidad"].isin(["mg/dL", "umol/L"])
valor_presente = tabla["creatinina"].notna()

tabla["razon_rechazo"] = ""
tabla.loc[~medido, "razon_rechazo"] = "estado_no_calculable"
tabla.loc[medido & ~valor_presente, "razon_rechazo"] = "valor_ausente"
tabla.loc[medido & valor_presente & ~unidad_valida, "razon_rechazo"] = "unidad_no_soportada"

calculable = medido & unidad_valida & valor_presente

aceptados = tabla.loc[calculable].copy()
rechazados = tabla.loc[~calculable].copy()

print(f"aceptados={len(aceptados)}")
print(f"rechazados={len(rechazados)}")
print(rechazados[["paciente_id", "estado", "razon_rechazo"]].to_string(index=False))
```

Salida esperada:

```text
aceptados=2
rechazados=2
paciente_id    estado        razon_rechazo
       P002 no_medido estado_no_calculable
       P004 pendiente estado_no_calculable
```

La fila `P004` no se pierde. Queda rechazada para este cálculo, con una razón explícita. En otro flujo podría reintentarse cuando el resultado deje de estar pendiente.

## Normalizar antes de persistir

El almacenamiento persistente debe recibir datos en una forma consistente. En este ejemplo, guardaremos creatinina en `mg/dL`, conservando además la unidad original.

```python
from io import StringIO

import pandas as pd

contenido = StringIO(
    """paciente_id,servicio,creatinina,unidad,estado
P001,urgencias,1.1,mg/dL,medido
P002,urgencias,,mg/dL,no_medido
P003,consulta,212,umol/L,medido
P004,urgencias,999,mg/dL,pendiente
"""
)

tabla = pd.read_csv(
    contenido,
    dtype={
        "paciente_id": "string",
        "servicio": "string",
        "unidad": "string",
        "estado": "string",
    },
)
tabla["creatinina"] = pd.to_numeric(tabla["creatinina"], errors="coerce")

calculable = tabla["estado"].eq("medido") & tabla["unidad"].isin(["mg/dL", "umol/L"]) & tabla["creatinina"].notna()
aceptados = tabla.loc[calculable].copy()

aceptados["creatinina_mg_dl"] = aceptados["creatinina"]
en_umol_l = aceptados["unidad"].eq("umol/L")
aceptados.loc[en_umol_l, "creatinina_mg_dl"] = aceptados.loc[en_umol_l, "creatinina"] / 88.4
aceptados["regla_version"] = "creatinina_tabular.v1"

print(
    aceptados[
        ["paciente_id", "servicio", "creatinina", "unidad", "creatinina_mg_dl", "regla_version"]
    ].to_string(index=False)
)
```

Salida esperada:

```text
paciente_id  servicio  creatinina unidad  creatinina_mg_dl         regla_version
       P001 urgencias         1.1  mg/dL           1.10000 creatinina_tabular.v1
       P003  consulta       212.0 umol/L           2.39819 creatinina_tabular.v1
```

La columna normalizada no borra el dato original. Lo acompaña.

## SQLite como persistencia mínima

SQLite es una base de datos relacional embebida. En Python se puede usar con el módulo estándar `sqlite3`, sin instalar un servidor. Eso la vuelve útil para aprendizaje, prototipos, lotes locales, pruebas y herramientas pequeñas.

Pero SQLite no convierte automáticamente un flujo en confiable. La confiabilidad viene de las reglas, las transacciones, los identificadores, las restricciones y la trazabilidad.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE mediciones_creatinina (
            paciente_id TEXT NOT NULL,
            servicio TEXT NOT NULL,
            creatinina_mg_dl REAL NOT NULL,
            regla_version TEXT NOT NULL
        )
        """
    )

    conexion.execute(
        """
        INSERT INTO mediciones_creatinina
        (paciente_id, servicio, creatinina_mg_dl, regla_version)
        VALUES (?, ?, ?, ?)
        """,
        ("P001", "urgencias", 1.1, "creatinina_tabular.v1"),
    )

    filas = conexion.execute(
        "SELECT paciente_id, servicio, creatinina_mg_dl FROM mediciones_creatinina"
    ).fetchall()

print(filas)
```

Salida esperada:

```text
[('P001', 'urgencias', 1.1)]
```

El signo `?` en la inserción es un marcador de posición. No se debe construir SQL pegando valores dentro de cadenas. Los parámetros protegen contra errores de formato y reducen riesgos de inyección cuando hay entradas externas.

## Persistir aceptados y rechazados

Un patrón más útil consiste en guardar datos aceptados y rechazos por separado. Así el sistema no pierde evidencia de lo que no entró al cálculo.

```python
from io import StringIO
import sqlite3

import pandas as pd

contenido = StringIO(
    """paciente_id,servicio,creatinina,unidad,estado
P001,urgencias,1.1,mg/dL,medido
P002,urgencias,,mg/dL,no_medido
P003,consulta,212,umol/L,medido
P004,urgencias,999,mg/dL,pendiente
"""
)

tabla = pd.read_csv(
    contenido,
    dtype={
        "paciente_id": "string",
        "servicio": "string",
        "unidad": "string",
        "estado": "string",
    },
)
tabla["creatinina"] = pd.to_numeric(tabla["creatinina"], errors="coerce")

medido = tabla["estado"].eq("medido")
unidad_valida = tabla["unidad"].isin(["mg/dL", "umol/L"])
valor_presente = tabla["creatinina"].notna()
calculable = medido & unidad_valida & valor_presente

aceptados = tabla.loc[calculable].copy()
aceptados["creatinina_mg_dl"] = aceptados["creatinina"]
en_umol_l = aceptados["unidad"].eq("umol/L")
aceptados.loc[en_umol_l, "creatinina_mg_dl"] = aceptados.loc[en_umol_l, "creatinina"] / 88.4
aceptados["regla_version"] = "creatinina_tabular.v1"

rechazados = tabla.loc[~calculable].copy()
rechazados["razon_rechazo"] = "estado_no_calculable"

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE mediciones_creatinina (
            paciente_id TEXT NOT NULL,
            servicio TEXT NOT NULL,
            creatinina_mg_dl REAL NOT NULL,
            regla_version TEXT NOT NULL
        )
        """
    )
    conexion.execute(
        """
        CREATE TABLE rechazos_creatinina (
            paciente_id TEXT NOT NULL,
            estado TEXT NOT NULL,
            razon_rechazo TEXT NOT NULL,
            regla_version TEXT NOT NULL
        )
        """
    )

    conexion.executemany(
        """
        INSERT INTO mediciones_creatinina
        (paciente_id, servicio, creatinina_mg_dl, regla_version)
        VALUES (?, ?, ?, ?)
        """,
        aceptados[["paciente_id", "servicio", "creatinina_mg_dl", "regla_version"]].itertuples(index=False, name=None),
    )
    conexion.executemany(
        """
        INSERT INTO rechazos_creatinina
        (paciente_id, estado, razon_rechazo, regla_version)
        VALUES (?, ?, ?, ?)
        """,
        rechazados.assign(regla_version="creatinina_tabular.v1")[
            ["paciente_id", "estado", "razon_rechazo", "regla_version"]
        ].itertuples(index=False, name=None),
    )

    total_aceptados = conexion.execute("SELECT COUNT(*) FROM mediciones_creatinina").fetchone()[0]
    total_rechazados = conexion.execute("SELECT COUNT(*) FROM rechazos_creatinina").fetchone()[0]

print(f"aceptados={total_aceptados}; rechazados={total_rechazados}")
```

Salida esperada:

```text
aceptados=2; rechazados=2
```

El resultado ya no depende de conservar la variable `tabla` en memoria. La información quedó persistida dentro de la conexión SQLite.

## Recuperar una vista con `pandas`

Después de persistir, `pandas` vuelve a ser útil como herramienta de lectura y análisis. La consulta debe ser explícita.

```python
import sqlite3

import pandas as pd

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE mediciones_creatinina (
            paciente_id TEXT,
            servicio TEXT,
            creatinina_mg_dl REAL,
            regla_version TEXT
        )
        """
    )
    conexion.executemany(
        """
        INSERT INTO mediciones_creatinina
        (paciente_id, servicio, creatinina_mg_dl, regla_version)
        VALUES (?, ?, ?, ?)
        """,
        [
            ("P001", "urgencias", 1.1, "creatinina_tabular.v1"),
            ("P003", "consulta", 2.39819, "creatinina_tabular.v1"),
        ],
    )

    resumen = pd.read_sql(
        """
        SELECT servicio, COUNT(*) AS mediciones, AVG(creatinina_mg_dl) AS promedio
        FROM mediciones_creatinina
        GROUP BY servicio
        ORDER BY servicio
        """,
        conexion,
    )

print(resumen.to_string(index=False))
```

Salida esperada:

```text
 servicio  mediciones  promedio
 consulta           1   2.39819
urgencias           1   1.10000
```

La tabla final es una vista analítica, no la fuente cruda. Esa diferencia debe mantenerse visible.

## `to_sql`: útil, pero no suficiente

`pandas` también puede escribir un `DataFrame` directamente a SQL con `to_sql`. Es útil para cargas simples, prototipos y exportaciones controladas. Pero no debe confundirse con diseño de base de datos.

```python
import sqlite3

import pandas as pd

aceptados = pd.DataFrame(
    [
        {"paciente_id": "P001", "servicio": "urgencias", "creatinina_mg_dl": 1.1},
        {"paciente_id": "P003", "servicio": "consulta", "creatinina_mg_dl": 2.39819},
    ]
)

with sqlite3.connect(":memory:") as conexion:
    aceptados.to_sql("mediciones_creatinina", conexion, index=False, if_exists="replace")
    conteo = pd.read_sql("SELECT COUNT(*) AS n FROM mediciones_creatinina", conexion)

print(conteo.to_string(index=False))
```

Salida esperada:

```text
 n
 2
```

El método creó una tabla y cargó filas. Aun así, una aplicación seria debe decidir nombres, restricciones, claves, índices, migraciones, relaciones y permisos. `to_sql` no reemplaza esas decisiones.

## Errores frecuentes

**Tratar el CSV como base de datos.** Un archivo puede ser una entrada reproducible, pero no gobierna relaciones, restricciones ni consultas concurrentes.

**Guardar datos antes de validar.** Persistir basura solo hace que la basura sea más duradera.

**Borrar rechazos.** Las filas rechazadas explican cobertura, sesgo, calidad de fuente y límites del análisis.

**Usar SQLite como excusa para no diseñar.** Una base local también necesita contrato: tablas, campos, tipos, claves y versiones.

**Confundir tabla de trabajo con fuente de verdad.** Un `DataFrame` puede reconstruirse; la base persistente debe decir qué versión de datos aceptados quedó disponible.

## Puente hacia diseño de bases de datos

El siguiente nivel no consiste en guardar una tabla más grande. Consiste en modelar entidades y relaciones.

En datos biomédicos, pronto aparecen preguntas como:

- qué identifica a un paciente, una muestra, un evento o una medición;
- cómo se relaciona una medición con su unidad, método, fecha y fuente;
- dónde se guarda una versión de regla;
- cómo se registran rechazos, correcciones y auditoría;
- qué consultas debe soportar el sistema sin recalcular todo desde archivos.

Ahí comienza el diseño de base de datos. El paso desde archivo a `DataFrame` y desde `DataFrame` a SQLite prepara esa conversación.

## Preguntas de comprensión profunda

1. ¿Qué información pertenece al archivo de entrada y qué información debe agregarse durante la validación?
2. ¿Por qué conviene persistir rechazos además de datos aceptados?
3. ¿Qué riesgo aparece si una tabla SQLite se crea automáticamente desde un `DataFrame` sin revisar tipos y restricciones?
4. ¿Cuándo usarías `executemany` con parámetros en lugar de construir sentencias SQL con cadenas?
5. ¿Por qué una consulta SQL recuperada con `pandas` debe interpretarse como vista analítica y no como dato crudo?
6. ¿Qué cambiaría si este flujo dejara de ser local y tuviera varios usuarios escribiendo al mismo tiempo?

## Bibliografía y fuentes

- Python Software Foundation. [sqlite3: DB-API 2.0 interface for SQLite databases](https://docs.python.org/3/library/sqlite3.html).
- The pandas development team. [IO tools: SQL queries](https://pandas.pydata.org/docs/user_guide/io.html#sql-queries).
- The pandas development team. [pandas.read_sql](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_sql.html).
- The pandas development team. [pandas.DataFrame.to_sql](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.to_sql.html).
