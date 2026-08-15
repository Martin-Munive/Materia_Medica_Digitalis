# Restricciones, índices y consultas reproducibles

La sección anterior separó entidades, relaciones y claves. Esa separación permite decir que una medición pertenece a un paciente, que un fármaco puede aparecer en muchas exposiciones y que una fila no debería existir si apunta a una identidad inexistente.

Pero el diseño relacional mínimo todavía necesita tres piezas para volverse operable:

**Restricciones.** Reglas que la base de datos debe hacer cumplir.

**Índices.** Estructuras que permiten buscar sin recorrer todo cada vez.

**Consultas reproducibles.** Preguntas escritas de forma explícita, revisable y repetible.

Sin restricciones, la base guarda errores con disciplina. Sin índices, algunas preguntas se vuelven lentas cuando el volumen crece. Sin consultas reproducibles, cada análisis puede reescribirse con pequeñas diferencias que cambian el resultado.

Esta sección no convierte SQLite en un sistema clínico completo. Enseña una idea más modesta y más poderosa: una base de datos no solo guarda datos; también conserva parte del contrato operacional del sistema.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una restricción es una regla declarada en la base para impedir estados inválidos. Un índice es una estructura auxiliar que acelera búsquedas y uniones sobre columnas relevantes. Una consulta reproducible es una pregunta escrita de forma explícita, parametrizada y versionable, de modo que pueda ejecutarse otra vez sobre los mismos datos y producir el mismo criterio de selección, cálculo o resumen.
</div>

La idea central es separar tres responsabilidades.

**Validez estructural.** La base rechaza lo que viola reglas mínimas.

**Recuperación eficiente.** La base encuentra filas por claves, fechas, estados o columnas consultadas con frecuencia.

**Análisis repetible.** La pregunta no queda escondida en una exploración manual.

## Versión ingenua: guardar primero y preguntar después

Supongamos una tabla de mediciones. Si no declaramos restricciones, podemos guardar estados o valores que contradicen el contrato.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT,
            paciente_id TEXT,
            estado TEXT,
            creatinina_mg_dl REAL
        )
        """
    )

    conexion.executemany(
        """
        INSERT INTO mediciones
        (medicion_id, paciente_id, estado, creatinina_mg_dl)
        VALUES (?, ?, ?, ?)
        """,
        [
            ("M001", "P001", "medido", 1.1),
            ("M002", "P001", "pendientee", 999.0),
            ("M003", "P002", "medido", -5.0),
        ],
    )

    filas = conexion.execute(
        "SELECT medicion_id, estado, creatinina_mg_dl FROM mediciones"
    ).fetchall()

print(filas)
```

Salida esperada:

```text
[('M001', 'medido', 1.1), ('M002', 'pendientee', 999.0), ('M003', 'medido', -5.0)]
```

SQLite hizo lo que le pedimos: guardó las filas. El problema es que le pedimos muy poco. `pendientee` entró como estado y `-5.0` entró como creatinina.

Una base que acepta todo no es flexible. Es muda.

## Crítica técnica: validar solo en Python no basta

La validación en Python sigue siendo necesaria. Permite limpiar archivos, normalizar unidades, separar aceptados y rechazos, producir razones y conservar versión de regla.

Pero si la base es el lugar persistente, también debe defender algunas reglas mínimas.

Hay tres razones.

Primero, no todos los datos entrarán siempre por la misma función. Una importación futura, un script de mantenimiento o una carga manual podrían saltarse el validador original.

Segundo, las restricciones documentan el contrato cerca del dato persistido.

Tercero, los errores estructurales deben fallar temprano. Es mejor que una inserción falle a que un análisis falle semanas después.

La regla práctica es clara: Python valida con riqueza semántica; la base protege invariantes mínimas.

## Restricciones mínimas

SQLite permite declarar restricciones como `PRIMARY KEY`, `NOT NULL`, `UNIQUE`, `CHECK` y `FOREIGN KEY`.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute("PRAGMA foreign_keys = ON")
    conexion.execute(
        """
        CREATE TABLE pacientes (
            paciente_id TEXT PRIMARY KEY
        )
        """
    )
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            estado TEXT NOT NULL CHECK (estado IN ('medido', 'no_medido', 'pendiente')),
            creatinina_mg_dl REAL,
            regla_version TEXT NOT NULL,
            CHECK (creatinina_mg_dl IS NULL OR creatinina_mg_dl >= 0),
            FOREIGN KEY (paciente_id) REFERENCES pacientes (paciente_id)
        )
        """
    )

    conexion.execute("INSERT INTO pacientes (paciente_id) VALUES (?)", ("P001",))
    conexion.execute(
        """
        INSERT INTO mediciones
        (medicion_id, paciente_id, estado, creatinina_mg_dl, regla_version)
        VALUES (?, ?, ?, ?, ?)
        """,
        ("M001", "P001", "medido", 1.1, "creatinina.v1"),
    )

    try:
        conexion.execute(
            """
            INSERT INTO mediciones
            (medicion_id, paciente_id, estado, creatinina_mg_dl, regla_version)
            VALUES (?, ?, ?, ?, ?)
            """,
            ("M002", "P001", "pendientee", 999.0, "creatinina.v1"),
        )
    except sqlite3.IntegrityError as error:
        print(type(error).__name__)
```

Salida esperada:

```text
IntegrityError
```

La base rechazó el estado mal escrito. No entiende medicina, pero sí puede defender una lista cerrada de estados.

## `CHECK` no reemplaza el dominio

Una restricción `CHECK` puede impedir valores negativos, estados fuera de lista o combinaciones imposibles. Pero no debe usarse como si fuera juicio clínico completo.

Por ejemplo, `creatinina_mg_dl >= 0` impide un valor técnicamente imposible. No decide si `2.4` es esperable para una persona concreta, si requiere atención, si cambia con edad, sexo, masa muscular o contexto clínico.

El diseño responsable separa niveles:

- imposibilidad técnica: puede vivir como restricción;
- regla pedagógica de ejemplo: puede vivir en validador y documentación;
- interpretación clínica: requiere contexto, evidencia y validación externa.

## Índices: buscar sin recorrer todo

Un índice es una estructura que ayuda a localizar filas por una o varias columnas. Conceptualmente se parece al índice de un libro: no cambia el contenido, pero cambia el costo de encontrarlo.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            fecha TEXT NOT NULL,
            creatinina_mg_dl REAL NOT NULL
        )
        """
    )
    conexion.execute(
        "CREATE INDEX idx_mediciones_paciente_fecha ON mediciones (paciente_id, fecha)"
    )

    indices = conexion.execute(
        "PRAGMA index_list('mediciones')"
    ).fetchall()

print(indices)
```

Salida esperada:

```text
[(0, 'idx_mediciones_paciente_fecha', 0, 'c', 0), (1, 'sqlite_autoindex_mediciones_1', 1, 'pk', 0)]
```

El índice `idx_mediciones_paciente_fecha` fue creado para consultas frecuentes por paciente y fecha. El índice automático existe porque `medicion_id` es clave primaria.

Un índice no mejora todas las consultas. También ocupa espacio y agrega costo al insertar o actualizar. Por eso se diseña a partir de preguntas reales, no por reflejo.

## Consulta reproducible

Una consulta reproducible no es una pregunta improvisada en una consola. Es una pregunta escrita con nombres claros, parámetros y criterio explícito.

Supongamos que queremos recuperar mediciones de un paciente en un rango de fechas.

```python
import sqlite3

CONSULTA_MEDICIONES_PACIENTE = """
SELECT medicion_id, paciente_id, fecha, creatinina_mg_dl
FROM mediciones
WHERE paciente_id = ?
  AND fecha >= ?
  AND fecha <= ?
ORDER BY fecha
"""

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            fecha TEXT NOT NULL,
            creatinina_mg_dl REAL NOT NULL
        )
        """
    )
    conexion.executemany(
        """
        INSERT INTO mediciones
        (medicion_id, paciente_id, fecha, creatinina_mg_dl)
        VALUES (?, ?, ?, ?)
        """,
        [
            ("M001", "P001", "2026-01-10", 1.1),
            ("M002", "P001", "2026-01-20", 1.3),
            ("M003", "P002", "2026-01-15", 2.4),
        ],
    )

    filas = conexion.execute(
        CONSULTA_MEDICIONES_PACIENTE,
        ("P001", "2026-01-01", "2026-01-31"),
    ).fetchall()

print(filas)
```

Salida esperada:

```text
[('M001', 'P001', '2026-01-10', 1.1), ('M002', 'P001', '2026-01-20', 1.3)]
```

La consulta tiene tres virtudes: no concatena valores dentro del SQL, declara el rango temporal y ordena la salida.

## CODE CLEAN: consulta nombrada antes que SQL disperso

La versión frágil reparte SQL por el programa.

```python
filas = conexion.execute(f"SELECT * FROM mediciones WHERE paciente_id = '{paciente_id}'").fetchall()
```

Esa línea mezcla consulta, formato de salida y valor externo. Además, construir SQL con cadenas es una mala práctica cuando hay entradas variables.

La versión más limpia nombra la pregunta y parametriza los valores.

```python
CONSULTA_MEDICIONES_PACIENTE = """
SELECT medicion_id, fecha, creatinina_mg_dl
FROM mediciones
WHERE paciente_id = ?
ORDER BY fecha
"""

filas = conexion.execute(CONSULTA_MEDICIONES_PACIENTE, (paciente_id,)).fetchall()
```

El beneficio no es solo seguridad. También es legibilidad: una consulta nombrada puede revisarse, probarse, versionarse y reutilizarse.

## Vista reproducible

Cuando una consulta se usa muchas veces, puede convertirse en vista. Una vista no duplica los datos; guarda una forma de preguntar.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT PRIMARY KEY,
            servicio TEXT NOT NULL,
            estado TEXT NOT NULL,
            creatinina_mg_dl REAL
        )
        """
    )
    conexion.executemany(
        """
        INSERT INTO mediciones
        (medicion_id, servicio, estado, creatinina_mg_dl)
        VALUES (?, ?, ?, ?)
        """,
        [
            ("M001", "urgencias", "medido", 1.1),
            ("M002", "urgencias", "pendiente", None),
            ("M003", "consulta", "medido", 2.4),
        ],
    )
    conexion.execute(
        """
        CREATE VIEW mediciones_calculables AS
        SELECT medicion_id, servicio, creatinina_mg_dl
        FROM mediciones
        WHERE estado = 'medido'
          AND creatinina_mg_dl IS NOT NULL
        """
    )

    resumen = conexion.execute(
        """
        SELECT servicio, COUNT(*) AS n
        FROM mediciones_calculables
        GROUP BY servicio
        ORDER BY servicio
        """
    ).fetchall()

print(resumen)
```

Salida esperada:

```text
[('consulta', 1), ('urgencias', 1)]
```

La vista hace explícito qué significa `calculable` para esta miniatura. No reemplaza el validador, pero evita reescribir el filtro en cada análisis.

## Pruebas mínimas

Podemos probar restricciones y consultas como propiedades del sistema.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute("PRAGMA foreign_keys = ON")
    conexion.execute("CREATE TABLE pacientes (paciente_id TEXT PRIMARY KEY)")
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            estado TEXT NOT NULL CHECK (estado IN ('medido', 'pendiente')),
            valor REAL CHECK (valor IS NULL OR valor >= 0),
            FOREIGN KEY (paciente_id) REFERENCES pacientes (paciente_id)
        )
        """
    )
    conexion.execute("INSERT INTO pacientes (paciente_id) VALUES (?)", ("P001",))
    conexion.execute(
        "INSERT INTO mediciones (medicion_id, paciente_id, estado, valor) VALUES (?, ?, ?, ?)",
        ("M001", "P001", "medido", 1.1),
    )

    # Propiedad 1: un estado fuera del vocabulario se rechaza.
    try:
        conexion.execute(
            "INSERT INTO mediciones (medicion_id, paciente_id, estado, valor) VALUES (?, ?, ?, ?)",
            ("M002", "P001", "medidoo", 1.2),
        )
        estado_rechazado = False
    except sqlite3.IntegrityError:
        estado_rechazado = True

    # Propiedad 2: la consulta parametrizada recupera solo el paciente pedido.
    filas = conexion.execute(
        "SELECT medicion_id FROM mediciones WHERE paciente_id = ?",
        ("P001",),
    ).fetchall()

assert estado_rechazado is True
assert filas == [("M001",)]
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas no demuestran que la base sea clínicamente suficiente. Demuestran que dos invariantes mínimas se sostienen: vocabulario controlado y consulta por identidad.

## Errores frecuentes

**Creer que una restricción reemplaza el validador.** La restricción protege la base; el validador explica y clasifica los errores antes de persistir.

**Crear índices para todo.** Cada índice tiene costo de mantenimiento. Debe responder a consultas frecuentes o críticas.

**Usar `SELECT *` como salida estable.** Si cambian las columnas, cambia la forma de la salida. En consultas reproducibles conviene nombrar columnas.

**Construir SQL con cadenas.** Los valores variables deben ir como parámetros.

**Guardar una consulta solo en la memoria del analista.** Si la pregunta importa, debe quedar escrita.

**Confundir vista con tabla derivada.** Una vista guarda la consulta; una tabla derivada guarda resultados. Cada una tiene riesgos y usos distintos.

## Argumentos críticos

### Desacuerdo 1: restricciones en Python contra restricciones en SQL

Pregunta: ¿por qué duplicar reglas?

No todas las reglas deben duplicarse. Pero las invariantes estructurales mínimas sí merecen defensa en la base: identidad, obligatoriedad, vocabularios cerrados, no negatividad técnica, relaciones existentes.

Consenso operativo: validar rico en Python; blindar mínimo en SQL.

### Desacuerdo 2: índices tempranos contra diseño prematuro

Pregunta: ¿cuándo crear índices?

No conviene indexar por reflejo. Pero si el diseño ya sabe que consultará por `paciente_id`, fecha, estado o clave foránea, un índice puede ser parte razonable del modelo.

Consenso operativo: crear índices cuando exista una consulta frecuente, una unión central o un volumen que justifique el costo.

### Desacuerdo 3: consulta ad hoc contra consulta versionada

Pregunta: ¿no es más rápido escribir la consulta cada vez?

Para explorar, sí. Para reportar, auditar o comparar resultados en el tiempo, no. Una consulta que define cohorte, denominador o regla de inclusión debe poder reconstruirse.

Consenso operativo: explorar libremente; convertir en consulta nombrada lo que sostenga una conclusión.

## Puente hacia APIs y análisis reproducibles

Las restricciones, índices y consultas preparan la transición hacia sistemas más grandes.

Una API necesita validar entradas antes de escribir y consultar salidas con criterios estables. Un análisis reproducible necesita saber qué consulta generó la cohorte. Un sistema de seguimiento longitudinal necesita índices por paciente y tiempo. Un pipeline biomédico necesita separar datos crudos, datos aceptados, rechazos, reglas y vistas analíticas.

Más adelante, cuando aparezcan APIs, modelos y sistemas de decisión, esta sección seguirá funcionando como una regla base: no basta con que el dato exista; debe existir bajo contrato, debe poder encontrarse y debe poder preguntarse de nuevo.

## Preguntas de comprensión profunda

1. ¿Qué diferencia hay entre validar en Python y declarar una restricción en SQL?
2. ¿Qué tipo de regla sí pondrías como `CHECK` y cuál dejarías fuera de la base?
3. ¿Por qué `SELECT *` puede ser peligroso en una consulta reproducible?
4. ¿Qué consulta justificaría un índice por `paciente_id` y `fecha`?
5. ¿Qué costo tiene crear demasiados índices?
6. ¿Por qué una vista puede ayudar a conservar el significado de `calculable`?
7. ¿Qué diferencia hay entre una consulta exploratoria y una consulta que define una cohorte?
8. ¿Qué propiedad mínima probarías después de crear una restricción nueva?

## Vacíos de comprensión que debes vigilar

1. Pensar que la base de datos solo obedece al programa. Una base bien diseñada también defiende reglas.
2. Creer que rendimiento significa agregar índices sin criterio. El índice debe nacer de una pregunta.
3. Tratar una consulta como texto desechable. Si la consulta define el resultado, forma parte del método.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma una tabla de mediciones y escribe tres restricciones mínimas: identidad, estado permitido y valor técnicamente posible.
2. **Segunda hora:** identifica dos consultas frecuentes y decide si justifican índice.
3. **Tercera hora:** escribe una consulta parametrizada y una prueba con `assert` que verifique su denominador.

## Bibliografía y fuentes

- Python Software Foundation. [sqlite3: DB-API 2.0 interface for SQLite databases](https://docs.python.org/3/library/sqlite3.html).
- SQLite. [CREATE TABLE](https://www.sqlite.org/lang_createtable.html).
- SQLite. [Indexes](https://www.sqlite.org/lang_createindex.html).
- SQLite. [Query Planning](https://www.sqlite.org/queryplanner.html).
- SQLite. [CREATE VIEW](https://www.sqlite.org/lang_createview.html).
- Silberschatz, A., Korth, H. F., & Sudarshan, S. (2020). *Database System Concepts* (7th ed.). McGraw Hill.

## Siguiente paso

Las restricciones, índices y consultas reproducibles cierran la entrada mínima a bases relacionales. La siguiente transición puede llevar estos contratos hacia APIs: cómo exponer operaciones de lectura y escritura sin perder validación, identidad, trazabilidad ni responsabilidad sobre el dato.
