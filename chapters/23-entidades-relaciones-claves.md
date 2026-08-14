# Entidades, relaciones y claves

La sección anterior mostró cómo pasar de un archivo de entrada a una tabla de trabajo y luego a una base SQLite mínima. Ese flujo ya separa adquisición, validación y persistencia. Pero todavía conserva una limitación importante: guarda mediciones en una tabla relativamente plana.

Una tabla plana puede servir para un lote pequeño. Sin embargo, en cuanto el dato biomédico crece, aparecen objetos que no conviene repetir en cada fila: pacientes, muestras, eventos, servicios, mediciones, reglas, fuentes y rechazos. Si todo se guarda como texto duplicado, el sistema queda expuesto a errores silenciosos: el mismo paciente aparece con dos nombres, una muestra queda sin dueño, una medición no puede reconstruir su fuente, y una corrección obliga a modificar muchas filas.

Aquí aparece el diseño mínimo de entidades, relaciones y claves.

No se trata todavía de construir una base clínica profesional. Se trata de aprender una frontera conceptual: cuando un dato debe persistir y relacionarse, ya no basta con preguntar qué columnas tiene una tabla. Hay que preguntar qué cosas existen, cómo se identifican y qué vínculos pueden defenderse.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una entidad es una cosa del dominio que necesita identidad estable dentro del sistema. Una relación declara cómo dos entidades se conectan. Una clave es el valor usado para identificar una entidad o enlazarla con otra. En datos biomédicos, entidades, relaciones y claves permiten separar identidad, observación, evento, medición y trazabilidad antes de consultar o analizar.
</div>

Esta definición contiene tres decisiones.

**Entidad.** Algo que merece existir como unidad propia: paciente, muestra, medición, visita, fármaco, variante, documento, resultado, regla.

**Relación.** La conexión explícita entre dos entidades: una medición pertenece a un paciente; una muestra fue tomada en una visita; un rechazo proviene de una fila de entrada; una regla validó una medición.

**Clave.** El identificador que permite reconocer una entidad o construir una relación sin depender de texto repetido.

La clave no es un adorno técnico. Es una decisión de identidad.

## Versión ingenua: repetir todo en una sola tabla

Supongamos que recibimos mediciones de creatinina. Una primera solución consiste en guardar cada fila con todos los datos visibles.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE mediciones_planas (
            paciente_id TEXT,
            paciente_nombre TEXT,
            servicio TEXT,
            creatinina_mg_dl REAL,
            regla_version TEXT
        )
        """
    )

    conexion.executemany(
        """
        INSERT INTO mediciones_planas
        (paciente_id, paciente_nombre, servicio, creatinina_mg_dl, regla_version)
        VALUES (?, ?, ?, ?, ?)
        """,
        [
            ("P001", "Ana Ruiz", "urgencias", 1.1, "creatinina_tabular.v1"),
            ("P001", "Ana R.", "consulta", 1.3, "creatinina_tabular.v1"),
            ("P002", "Luis Mora", "urgencias", 2.4, "creatinina_tabular.v1"),
        ],
    )

    filas = conexion.execute(
        """
        SELECT paciente_id, paciente_nombre, servicio, creatinina_mg_dl
        FROM mediciones_planas
        ORDER BY paciente_id, servicio
        """
    ).fetchall()

print(filas)
```

Salida esperada:

```text
[('P001', 'Ana R.', 'consulta', 1.3), ('P001', 'Ana Ruiz', 'urgencias', 1.1), ('P002', 'Luis Mora', 'urgencias', 2.4)]
```

La tabla funciona. También revela un problema: `P001` aparece con dos nombres. Si el nombre se repite en cada medición, cada fila puede contar una versión distinta de la identidad.

La base guardó datos. No protegió identidad.

## Crítica técnica: una fila no siempre es una entidad

Una fila puede representar muchas cosas distintas. Puede ser un paciente, una muestra, una medición, una visita, una línea de archivo, una alerta o una fila de rechazo.

Si no decidimos qué representa cada fila, el diseño queda ambiguo.

En la tabla plana anterior, una fila parece representar una medición. Pero también contiene nombre del paciente, servicio y versión de regla. Esos elementos tienen ritmos distintos:

- el paciente puede existir aunque no tenga mediciones;
- el nombre del paciente puede corregirse sin cambiar la medición;
- el servicio pertenece al contexto del evento o la visita;
- la creatinina pertenece a la medición;
- la regla pertenece al proceso que aceptó o rechazó esa medición.

Cuando esos ritmos se mezclan, aparecen cuatro riesgos: duplicación, inconsistencia, pérdida de trazabilidad y dificultad para consultar.

## Entidad mínima

Una entidad mínima no necesita ser compleja. Solo debe tener identidad, campos propios y un propósito claro.

En este ejemplo separaremos dos entidades:

- `pacientes`;
- `mediciones_creatinina`.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute(
        """
        CREATE TABLE pacientes (
            paciente_id TEXT PRIMARY KEY,
            nombre TEXT NOT NULL
        )
        """
    )
    conexion.execute(
        """
        CREATE TABLE mediciones_creatinina (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            servicio TEXT NOT NULL,
            creatinina_mg_dl REAL NOT NULL,
            regla_version TEXT NOT NULL
        )
        """
    )

    conexion.executemany(
        "INSERT INTO pacientes (paciente_id, nombre) VALUES (?, ?)",
        [
            ("P001", "Ana Ruiz"),
            ("P002", "Luis Mora"),
        ],
    )
    conexion.executemany(
        """
        INSERT INTO mediciones_creatinina
        (medicion_id, paciente_id, servicio, creatinina_mg_dl, regla_version)
        VALUES (?, ?, ?, ?, ?)
        """,
        [
            ("M001", "P001", "urgencias", 1.1, "creatinina_tabular.v1"),
            ("M002", "P001", "consulta", 1.3, "creatinina_tabular.v1"),
            ("M003", "P002", "urgencias", 2.4, "creatinina_tabular.v1"),
        ],
    )

    total_pacientes = conexion.execute("SELECT COUNT(*) FROM pacientes").fetchone()[0]
    total_mediciones = conexion.execute("SELECT COUNT(*) FROM mediciones_creatinina").fetchone()[0]

print(f"pacientes={total_pacientes}; mediciones={total_mediciones}")
```

Salida esperada:

```text
pacientes=2; mediciones=3
```

La separación todavía es pequeña, pero ya cambió el diseño. El paciente no se repite como texto en cada medición. La medición conserva su propia identidad con `medicion_id`.

## Clave primaria y clave foránea

En bases relacionales, una clave primaria identifica una fila dentro de su tabla. Una clave foránea apunta a una fila de otra tabla.

En el ejemplo anterior:

- `pacientes.paciente_id` identifica al paciente;
- `mediciones_creatinina.medicion_id` identifica la medición;
- `mediciones_creatinina.paciente_id` enlaza la medición con el paciente.

SQLite permite declarar ese vínculo.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute("PRAGMA foreign_keys = ON")

    conexion.execute(
        """
        CREATE TABLE pacientes (
            paciente_id TEXT PRIMARY KEY,
            nombre TEXT NOT NULL
        )
        """
    )
    conexion.execute(
        """
        CREATE TABLE mediciones_creatinina (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            creatinina_mg_dl REAL NOT NULL,
            FOREIGN KEY (paciente_id) REFERENCES pacientes (paciente_id)
        )
        """
    )

    conexion.execute("INSERT INTO pacientes (paciente_id, nombre) VALUES (?, ?)", ("P001", "Ana Ruiz"))
    conexion.execute(
        """
        INSERT INTO mediciones_creatinina
        (medicion_id, paciente_id, creatinina_mg_dl)
        VALUES (?, ?, ?)
        """,
        ("M001", "P001", 1.1),
    )

    try:
        conexion.execute(
            """
            INSERT INTO mediciones_creatinina
            (medicion_id, paciente_id, creatinina_mg_dl)
            VALUES (?, ?, ?)
            """,
            ("M002", "P999", 2.4),
        )
    except sqlite3.IntegrityError as error:
        print(type(error).__name__)
```

Salida esperada:

```text
IntegrityError
```

La segunda medición fue rechazada porque intenta apuntar a un paciente inexistente. Ese rechazo no depende de recordar una regla en la cabeza. La base lo sabe porque el vínculo fue declarado.

## CODE CLEAN: identidad antes de cálculo

La versión frágil calcula sobre texto repetido.

```python
pacientes_ana = tabla[tabla["paciente_nombre"].str.contains("Ana")]
```

Esa línea puede parecer útil para explorar, pero no debe gobernar identidad. Un nombre cambia, se abrevia, se escribe mal o se repite entre personas distintas.

La versión más limpia identifica primero y calcula después.

```python
paciente_id = "P001"
mediciones = obtener_mediciones_de_paciente(conexion, paciente_id)
```

El punto no es que `paciente_id` sea mágicamente perfecto. El punto es que el sistema declara qué campo usará como identidad y concentra allí sus garantías.

Una clave estable reduce ambigüedad. También hace visible cuándo la identidad no está resuelta.

## Relación uno a muchos

La relación entre paciente y mediciones suele ser uno a muchos: un paciente puede tener varias mediciones, y cada medición aceptada pertenece a un paciente.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute("PRAGMA foreign_keys = ON")
    conexion.execute(
        """
        CREATE TABLE pacientes (
            paciente_id TEXT PRIMARY KEY,
            nombre TEXT NOT NULL
        )
        """
    )
    conexion.execute(
        """
        CREATE TABLE mediciones_creatinina (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            servicio TEXT NOT NULL,
            creatinina_mg_dl REAL NOT NULL,
            FOREIGN KEY (paciente_id) REFERENCES pacientes (paciente_id)
        )
        """
    )

    conexion.executemany(
        "INSERT INTO pacientes (paciente_id, nombre) VALUES (?, ?)",
        [("P001", "Ana Ruiz"), ("P002", "Luis Mora")],
    )
    conexion.executemany(
        """
        INSERT INTO mediciones_creatinina
        (medicion_id, paciente_id, servicio, creatinina_mg_dl)
        VALUES (?, ?, ?, ?)
        """,
        [
            ("M001", "P001", "urgencias", 1.1),
            ("M002", "P001", "consulta", 1.3),
            ("M003", "P002", "urgencias", 2.4),
        ],
    )

    resumen = conexion.execute(
        """
        SELECT p.paciente_id, p.nombre, COUNT(m.medicion_id) AS mediciones
        FROM pacientes AS p
        LEFT JOIN mediciones_creatinina AS m
            ON p.paciente_id = m.paciente_id
        GROUP BY p.paciente_id, p.nombre
        ORDER BY p.paciente_id
        """
    ).fetchall()

print(resumen)
```

Salida esperada:

```text
[('P001', 'Ana Ruiz', 2), ('P002', 'Luis Mora', 1)]
```

`JOIN` combina filas relacionadas. No inventa la relación; la usa. La relación ya estaba en la clave compartida.

## Relación muchos a muchos

No todas las relaciones biomédicas son uno a muchos. Un paciente puede recibir varios fármacos y un fármaco puede aparecer en muchos pacientes. Eso es muchos a muchos.

La forma mínima de representarlo es una tabla intermedia.

```python
import sqlite3

with sqlite3.connect(":memory:") as conexion:
    conexion.execute("PRAGMA foreign_keys = ON")
    conexion.execute("CREATE TABLE pacientes (paciente_id TEXT PRIMARY KEY)")
    conexion.execute("CREATE TABLE farmacos (farmaco_id TEXT PRIMARY KEY, nombre TEXT NOT NULL)")
    conexion.execute(
        """
        CREATE TABLE exposiciones_farmaco (
            paciente_id TEXT NOT NULL,
            farmaco_id TEXT NOT NULL,
            fecha_inicio TEXT NOT NULL,
            PRIMARY KEY (paciente_id, farmaco_id, fecha_inicio),
            FOREIGN KEY (paciente_id) REFERENCES pacientes (paciente_id),
            FOREIGN KEY (farmaco_id) REFERENCES farmacos (farmaco_id)
        )
        """
    )

    conexion.executemany("INSERT INTO pacientes (paciente_id) VALUES (?)", [("P001",), ("P002",)])
    conexion.executemany(
        "INSERT INTO farmacos (farmaco_id, nombre) VALUES (?, ?)",
        [("F001", "metformina"), ("F002", "losartan")],
    )
    conexion.executemany(
        """
        INSERT INTO exposiciones_farmaco
        (paciente_id, farmaco_id, fecha_inicio)
        VALUES (?, ?, ?)
        """,
        [
            ("P001", "F001", "2026-01-10"),
            ("P001", "F002", "2026-01-12"),
            ("P002", "F001", "2026-02-01"),
        ],
    )

    filas = conexion.execute(
        """
        SELECT e.paciente_id, f.nombre, e.fecha_inicio
        FROM exposiciones_farmaco AS e
        JOIN farmacos AS f ON e.farmaco_id = f.farmaco_id
        ORDER BY e.paciente_id, f.nombre
        """
    ).fetchall()

print(filas)
```

Salida esperada:

```text
[('P001', 'losartan', '2026-01-12'), ('P001', 'metformina', '2026-01-10'), ('P002', 'metformina', '2026-02-01')]
```

La tabla `exposiciones_farmaco` no es relleno. Es la entidad que representa el hecho de exposición: quién, qué fármaco y desde cuándo.

## Claves naturales y claves artificiales

Una clave natural proviene del dominio: número de documento, código de muestra, identificador institucional, código de fármaco.

Una clave artificial es creada por el sistema: `medicion_id`, `evento_id`, `registro_id`.

Ninguna es perfecta por definición.

Las claves naturales pueden cambiar, venir con errores, tener duplicados o depender de una institución. Las claves artificiales son estables dentro del sistema, pero no significan nada fuera de él si no se acompañan de trazabilidad.

Regla práctica para este libro:

- usar claves naturales solo cuando el dominio las garantice razonablemente;
- usar claves artificiales para entidades internas como mediciones, eventos, rechazos o cargas;
- conservar fuente y versión cuando la identidad provenga de archivos externos;
- no usar nombres, fechas o posiciones de fila como identidad principal si pueden repetirse o cambiar.

## Pruebas mínimas

Una parte del diseño relacional se puede verificar con propiedades simples.

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
            valor REAL NOT NULL,
            FOREIGN KEY (paciente_id) REFERENCES pacientes (paciente_id)
        )
        """
    )

    conexion.execute("INSERT INTO pacientes (paciente_id) VALUES (?)", ("P001",))
    conexion.execute(
        "INSERT INTO mediciones (medicion_id, paciente_id, valor) VALUES (?, ?, ?)",
        ("M001", "P001", 1.1),
    )

    # Propiedad 1: no se puede repetir la identidad de una medición.
    try:
        conexion.execute(
            "INSERT INTO mediciones (medicion_id, paciente_id, valor) VALUES (?, ?, ?)",
            ("M001", "P001", 1.2),
        )
        duplicado_rechazado = False
    except sqlite3.IntegrityError:
        duplicado_rechazado = True

    # Propiedad 2: no se puede crear una medición para un paciente inexistente.
    try:
        conexion.execute(
            "INSERT INTO mediciones (medicion_id, paciente_id, valor) VALUES (?, ?, ?)",
            ("M002", "P999", 2.0),
        )
        huerfana_rechazada = False
    except sqlite3.IntegrityError:
        huerfana_rechazada = True

assert duplicado_rechazado is True
assert huerfana_rechazada is True
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas no validan un sistema clínico. Verifican algo más pequeño y fundamental: la base respeta identidad única y no acepta mediciones huérfanas.

## Diseño mínimo para una medición biomédica

Antes de crear tablas, conviene escribir el diseño en lenguaje de dominio.

Para una medición de laboratorio, una versión mínima podría decir:

1. Existe una entidad `paciente`.
2. Existe una entidad `medicion`.
3. Cada `medicion` pertenece a exactamente un `paciente`.
4. Una `medicion` tiene valor, unidad normalizada, fecha o contexto, y versión de regla.
5. Un `paciente` puede tener cero, una o muchas mediciones.
6. Una fila rechazada no es una medición aceptada, pero conserva referencia a carga, paciente declarado y razón de rechazo.
7. Ninguna medición aceptada debe existir sin paciente asociado.
8. Ninguna medición aceptada debe perder la regla que la volvió interpretable.

Ese diseño puede parecer obvio. Es precisamente por eso que debe escribirse. Las obviedades no escritas son una fuente común de errores.

## Errores frecuentes

**Confundir nombre con identidad.** Un nombre ayuda a leer, pero no identifica de forma estable.

**Usar una tabla plana demasiado tiempo.** Las tablas planas sirven para explorar, no siempre para persistir sistemas con relaciones.

**Crear claves sin significado operativo.** Un identificador debe responder qué entidad identifica y quién garantiza su estabilidad.

**No activar claves foráneas en SQLite.** En SQLite, `PRAGMA foreign_keys = ON` es necesario para que la base haga cumplir esas restricciones durante la conexión.

**Convertir una relación muchos a muchos en columnas repetidas.** Campos como `farmaco_1`, `farmaco_2`, `farmaco_3` esconden una tabla intermedia.

**Borrar rechazos del modelo.** Los rechazos también tienen entidad operativa cuando explican calidad de datos, cobertura o sesgo.

**Sobrediseñar antes de entender el flujo.** No toda miniatura necesita veinte tablas. El diseño mínimo debe crecer desde preguntas reales.

## Argumentos críticos

### Desacuerdo 1: tabla plana contra modelo relacional

Pregunta: ¿por qué no dejar todo en una sola tabla si funciona?

Una tabla plana es legítima para exploración, reportes simples y lotes pequeños. Pero cuando hay identidad repetida, correcciones, muchas mediciones por paciente o vínculos con reglas y fuentes, la tabla plana empieza a ocultar relaciones.

Consenso operativo: usar tabla plana para entender la fuente; pasar a entidades y relaciones cuando el dato deba persistir, auditarse o crecer.

### Desacuerdo 2: claves naturales contra claves artificiales

Pregunta: ¿no basta con usar el identificador que ya viene en el archivo?

A veces sí. Pero un archivo puede traer identificadores locales, incompletos o cambiantes. La clave natural debe evaluarse como cualquier otro dato: fuente, formato, estabilidad y riesgo de duplicado.

Consenso operativo: no adoptar una clave natural sin revisar quién la asigna, dónde es única y qué pasa cuando cambia.

### Desacuerdo 3: normalización contra simplicidad pedagógica

Pregunta: ¿separar pacientes, mediciones y exposiciones no vuelve el ejemplo demasiado complejo?

Puede volverlo más largo. Pero también vuelve visible lo que una tabla plana oculta. La pedagogía no debe eliminar la estructura que hace confiable el sistema.

Consenso operativo: enseñar primero el caso plano, mostrar su falla y luego introducir solo las entidades necesarias para corregirla.

## Puente hacia bases de datos, APIs y análisis reproducibles

Las entidades y relaciones preparan el terreno para temas más exigentes.

En bases de datos, se traducen en tablas, claves primarias, claves foráneas, índices y restricciones. En APIs, aparecen como recursos: pacientes, mediciones, muestras, eventos, reglas. En análisis reproducibles, permiten reconstruir de dónde salió cada cohorte, qué mediciones entraron, qué filas se rechazaron y qué versión de regla gobernó la salida.

Más adelante, cuando el libro estudie estructuras de datos y algoritmos clásicos, esta misma idea regresará con otra forma: índices para buscar, grafos para representar redes, árboles para organizar decisiones, colas para priorizar eventos y tablas relacionales para consultar evidencia.

La relación no es solo un concepto de base de datos. Es una forma de decir que el dato biomédico rara vez vive aislado.

## Preguntas de comprensión profunda

1. ¿Qué diferencia hay entre una fila, una entidad y una medición?
2. ¿Por qué repetir el nombre del paciente en cada medición puede producir inconsistencias?
3. ¿Qué protege una clave primaria?
4. ¿Qué protege una clave foránea?
5. ¿Por qué SQLite necesita `PRAGMA foreign_keys = ON` en estos ejemplos?
6. ¿Cuándo una tabla plana es suficiente y cuándo empieza a ser peligrosa?
7. ¿Qué entidad intermedia usarías para representar muchos pacientes expuestos a muchos fármacos?
8. ¿Por qué un rechazo puede merecer persistencia propia?
9. ¿Qué riesgo aparece al usar fecha y nombre como identidad de una muestra?
10. ¿Cómo cambiaría el diseño si una medición pudiera tener varias correcciones posteriores?

## Vacíos de comprensión que debes vigilar

1. Pensar que una base de datos es solo un lugar donde guardar tablas. En realidad también expresa identidad, vínculos y restricciones.
2. Creer que una clave artificial resuelve todos los problemas de identidad. La clave identifica dentro del sistema, pero todavía necesita fuente y trazabilidad.
3. Confundir normalización de base de datos con burocracia. La separación mínima evita duplicación e inconsistencia cuando el dominio empieza a crecer.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma una tabla plana de laboratorio y subraya qué columnas pertenecen a paciente, medición, evento, regla o fuente.
2. **Segunda hora:** dibuja tres tablas mínimas y escribe sus claves primarias y foráneas sin código.
3. **Tercera hora:** implementa esas tablas en SQLite, activa claves foráneas y prueba que una medición huérfana sea rechazada.

## Bibliografía y fuentes

- Python Software Foundation. [sqlite3: DB-API 2.0 interface for SQLite databases](https://docs.python.org/3/library/sqlite3.html).
- SQLite. [Foreign Key Support](https://www.sqlite.org/foreignkeys.html).
- SQLite. [CREATE TABLE](https://www.sqlite.org/lang_createtable.html).
- Elmasri, R., & Navathe, S. B. (2016). *Fundamentals of Database Systems* (7th ed.). Pearson.
- Silberschatz, A., Korth, H. F., & Sudarshan, S. (2020). *Database System Concepts* (7th ed.). McGraw Hill.

## Siguiente paso

Las entidades, relaciones y claves convierten la persistencia mínima en un diseño consultable. La siguiente sección puede avanzar hacia restricciones, índices y consultas reproducibles: cómo pedirle preguntas a una base sin volver a limpiar el archivo original cada vez.
