# APIs mínimas y contratos de entrada/salida

La sección anterior cerró una entrada mínima a bases relacionales: restricciones para impedir estados inválidos, índices para recuperar datos sin recorrer todo y consultas reproducibles para que una pregunta pueda ejecutarse otra vez con el mismo criterio.

El siguiente paso no es publicar una aplicación web completa. Es más básico: aprender qué ocurre cuando una operación deja de ser una función local o una consulta interna y se convierte en una puerta de entrada para otro programa.

Ahí aparece una API.

Una API puede sonar a infraestructura remota, servidor, autenticación, documentación interactiva o arquitectura grande. Pero la idea central es más simple: una API define cómo otro componente puede pedir una operación y qué forma tendrá la respuesta.

En datos biomédicos, esa frontera importa. Si una operación permite registrar una medición, consultar resultados o recuperar rechazos, debe conservar lo que ya hemos construido: tipos, validadores, esquemas, claves, restricciones, consultas y trazabilidad.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una API mínima es un contrato explícito entre un consumidor y un sistema: qué operación está disponible, qué entrada acepta, qué validación aplica, qué salida devuelve y qué errores puede reportar. En datos biomédicos, una API responsable no expone datos crudos sin contrato; traduce solicitudes externas en operaciones validadas, trazables y limitadas por el dominio.
</div>

Esta definición evita dos confusiones.

Primero, una API no es solo una URL. También puede ser una función pública de un módulo, un comando, un endpoint HTTP o un método documentado. Lo importante es el contrato.

Segundo, una API no vuelve correcto el dato. Si expone una operación mal diseñada, solo la hace más fácil de usar mal.

## Versión ingenua: función que recibe cualquier cosa

Supongamos una operación para registrar una medición de creatinina.

```python
mediciones = []


def registrar_medicion(datos):
    mediciones.append(datos)
    return {"ok": True}


respuesta = registrar_medicion(
    {
        "paciente_id": "P001",
        "creatinina": "pendiente",
        "unidad": "mg/dL",
    }
)

print(respuesta)
print(mediciones)
```

Salida esperada:

```text
{'ok': True}
[{'paciente_id': 'P001', 'creatinina': 'pendiente', 'unidad': 'mg/dL'}]
```

La operación respondió `ok`. Pero no validó campo, tipo, estado, unidad ni trazabilidad. El problema no es que sea una función local. El problema es que ya actúa como puerta de entrada y no tiene contrato suficiente.

Si mañana esa misma función queda detrás de un endpoint HTTP, el error conceptual no desaparece. Solo queda publicado.

## Crítica técnica: una API amplifica el diseño previo

Una API es una frontera. Todo lo que entra por ella puede afectar el sistema persistente. Por eso debe hacer explícitas cuatro capas.

**Entrada.** Qué campos acepta y en qué forma.

**Validación.** Qué reglas se aplican antes de escribir, consultar o calcular.

**Operación.** Qué acción real ocurre: insertar, buscar, actualizar, rechazar, resumir.

**Salida.** Qué devuelve el sistema: datos, estado, razón, identificador, error o resumen.

Una API que solo devuelve `ok` empobrece el sistema. No permite saber qué regla se aplicó, qué entidad fue creada, qué error ocurrió o si la entrada quedó rechazada.

## Contrato de entrada

Un contrato de entrada puede representarse con un diccionario simple antes de usar herramientas más formales.

```python
CONTRATO_CREAR_MEDICION = {
    "operacion": "crear_medicion_creatinina",
    "version": "1.0.0",
    "campos_requeridos": ["paciente_id", "valor", "unidad", "estado"],
    "unidades_permitidas": ["mg/dL", "umol/L"],
    "estados_permitidos": ["medido", "no_medido", "pendiente"],
}

print(CONTRATO_CREAR_MEDICION["operacion"])
print(CONTRATO_CREAR_MEDICION["version"])
```

Salida esperada:

```text
crear_medicion_creatinina
1.0.0
```

El contrato no ejecuta nada. Declara qué espera la frontera.

## Validar antes de operar

Ahora escribimos una validación mínima de solicitud. No reemplaza todos los validadores previos del libro; solo muestra el patrón de API.

```python
CONTRATO_CREAR_MEDICION = {
    "version": "1.0.0",
    "campos_requeridos": ["paciente_id", "valor", "unidad", "estado"],
    "unidades_permitidas": ["mg/dL", "umol/L"],
    "estados_permitidos": ["medido", "no_medido", "pendiente"],
}


def validar_solicitud_medicion(solicitud, contrato):
    errores = []

    for campo in contrato["campos_requeridos"]:
        if campo not in solicitud:
            errores.append({"campo": campo, "razon": "campo_requerido_ausente"})

    if errores:
        return {"valida": False, "errores": errores, "contrato_version": contrato["version"]}

    if solicitud["unidad"] not in contrato["unidades_permitidas"]:
        errores.append({"campo": "unidad", "razon": "unidad_no_soportada"})

    if solicitud["estado"] not in contrato["estados_permitidos"]:
        errores.append({"campo": "estado", "razon": "estado_no_permitido"})

    if solicitud["estado"] == "medido":
        try:
            valor = float(str(solicitud["valor"]).replace(",", "."))
        except ValueError:
            errores.append({"campo": "valor", "razon": "valor_no_numerico"})
            valor = None
    else:
        valor = None

    return {
        "valida": not errores,
        "valor_normalizado": valor,
        "errores": errores,
        "contrato_version": contrato["version"],
    }


resultado = validar_solicitud_medicion(
    {"paciente_id": "P001", "valor": "1.2", "unidad": "mg/dL", "estado": "medido"},
    CONTRATO_CREAR_MEDICION,
)

print(resultado)
```

Salida esperada:

```text
{'valida': True, 'valor_normalizado': 1.2, 'errores': [], 'contrato_version': '1.0.0'}
```

La validación produce una salida estructurada. No imprime un mensaje suelto ni devuelve solo `True`.

## Operación mínima sobre SQLite

Después de validar, la API puede llamar una operación interna. Esa operación no debe recibir datos crudos, sino valores ya interpretados.

```python
import sqlite3


def crear_base_minima():
    conexion = sqlite3.connect(":memory:")
    conexion.execute("PRAGMA foreign_keys = ON")
    conexion.execute("CREATE TABLE pacientes (paciente_id TEXT PRIMARY KEY)")
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            valor REAL,
            unidad TEXT NOT NULL,
            estado TEXT NOT NULL CHECK (estado IN ('medido', 'no_medido', 'pendiente')),
            contrato_version TEXT NOT NULL,
            FOREIGN KEY (paciente_id) REFERENCES pacientes (paciente_id)
        )
        """
    )
    return conexion


def insertar_medicion(conexion, medicion):
    conexion.execute(
        """
        INSERT INTO mediciones
        (medicion_id, paciente_id, valor, unidad, estado, contrato_version)
        VALUES (?, ?, ?, ?, ?, ?)
        """,
        (
            medicion["medicion_id"],
            medicion["paciente_id"],
            medicion["valor"],
            medicion["unidad"],
            medicion["estado"],
            medicion["contrato_version"],
        ),
    )


with crear_base_minima() as conexion:
    conexion.execute("INSERT INTO pacientes (paciente_id) VALUES (?)", ("P001",))
    insertar_medicion(
        conexion,
        {
            "medicion_id": "M001",
            "paciente_id": "P001",
            "valor": 1.2,
            "unidad": "mg/dL",
            "estado": "medido",
            "contrato_version": "1.0.0",
        },
    )
    total = conexion.execute("SELECT COUNT(*) FROM mediciones").fetchone()[0]

print(total)
```

Salida esperada:

```text
1
```

La función `insertar_medicion` es deliberadamente delgada. No decide qué es válido. Inserta una estructura ya validada y deja que la base defienda restricciones mínimas.

## Respuesta de API

Una API mínima debe devolver algo más útil que `ok`.

```python
def respuesta_exito(medicion_id, contrato_version):
    return {
        "estado": "aceptada",
        "medicion_id": medicion_id,
        "contrato_version": contrato_version,
    }


def respuesta_error(errores, contrato_version):
    return {
        "estado": "rechazada",
        "errores": errores,
        "contrato_version": contrato_version,
    }


print(respuesta_exito("M001", "1.0.0"))
print(respuesta_error([{"campo": "unidad", "razon": "unidad_no_soportada"}], "1.0.0"))
```

Salida esperada:

```text
{'estado': 'aceptada', 'medicion_id': 'M001', 'contrato_version': '1.0.0'}
{'estado': 'rechazada', 'errores': [{'campo': 'unidad', 'razon': 'unidad_no_soportada'}], 'contrato_version': '1.0.0'}
```

La salida conserva estado, identificador o errores y versión de contrato. Esa información permite depurar, auditar y explicar.

## CODE CLEAN: frontera delgada, dominio explícito

La versión frágil hace todo en una sola función.

```python
def endpoint_crear_medicion(datos):
    guardar(datos)
    return {"ok": True}
```

La versión más limpia separa frontera, validación y operación.

```python
def endpoint_crear_medicion(solicitud, conexion):
    validacion = validar_solicitud_medicion(solicitud, CONTRATO_CREAR_MEDICION)
    if not validacion["valida"]:
        return respuesta_error(validacion["errores"], validacion["contrato_version"])

    medicion = construir_medicion_validada(solicitud, validacion)
    insertar_medicion(conexion, medicion)
    return respuesta_exito(medicion["medicion_id"], validacion["contrato_version"])
```

El endpoint no debería contener todo el conocimiento del dominio. Debe coordinar piezas que ya tienen responsabilidades claras.

## Miniatura completa sin servidor

Podemos simular una API sin levantar HTTP. Eso permite concentrarse en contrato y flujo.

```python
import sqlite3

CONTRATO_CREAR_MEDICION = {
    "version": "1.0.0",
    "campos_requeridos": ["medicion_id", "paciente_id", "valor", "unidad", "estado"],
    "unidades_permitidas": ["mg/dL", "umol/L"],
    "estados_permitidos": ["medido", "no_medido", "pendiente"],
}


def validar_solicitud_medicion(solicitud, contrato):
    errores = []
    for campo in contrato["campos_requeridos"]:
        if campo not in solicitud:
            errores.append({"campo": campo, "razon": "campo_requerido_ausente"})

    if errores:
        return {"valida": False, "errores": errores, "valor_normalizado": None}

    if solicitud["unidad"] not in contrato["unidades_permitidas"]:
        errores.append({"campo": "unidad", "razon": "unidad_no_soportada"})
    if solicitud["estado"] not in contrato["estados_permitidos"]:
        errores.append({"campo": "estado", "razon": "estado_no_permitido"})

    valor = None
    if solicitud["estado"] == "medido":
        try:
            valor = float(str(solicitud["valor"]).replace(",", "."))
        except ValueError:
            errores.append({"campo": "valor", "razon": "valor_no_numerico"})

    return {
        "valida": not errores,
        "errores": errores,
        "valor_normalizado": valor,
    }


def crear_base_minima():
    conexion = sqlite3.connect(":memory:")
    conexion.execute("PRAGMA foreign_keys = ON")
    conexion.execute("CREATE TABLE pacientes (paciente_id TEXT PRIMARY KEY)")
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            valor REAL,
            unidad TEXT NOT NULL,
            estado TEXT NOT NULL CHECK (estado IN ('medido', 'no_medido', 'pendiente')),
            contrato_version TEXT NOT NULL,
            FOREIGN KEY (paciente_id) REFERENCES pacientes (paciente_id)
        )
        """
    )
    return conexion


def endpoint_crear_medicion(solicitud, conexion):
    validacion = validar_solicitud_medicion(solicitud, CONTRATO_CREAR_MEDICION)
    version = CONTRATO_CREAR_MEDICION["version"]

    if not validacion["valida"]:
        return {"estado": "rechazada", "errores": validacion["errores"], "contrato_version": version}

    try:
        conexion.execute(
            """
            INSERT INTO mediciones
            (medicion_id, paciente_id, valor, unidad, estado, contrato_version)
            VALUES (?, ?, ?, ?, ?, ?)
            """,
            (
                solicitud["medicion_id"],
                solicitud["paciente_id"],
                validacion["valor_normalizado"],
                solicitud["unidad"],
                solicitud["estado"],
                version,
            ),
        )
    except sqlite3.IntegrityError as error:
        return {
            "estado": "rechazada",
            "errores": [{"campo": "base", "razon": type(error).__name__}],
            "contrato_version": version,
        }

    return {
        "estado": "aceptada",
        "medicion_id": solicitud["medicion_id"],
        "contrato_version": version,
    }


with crear_base_minima() as conexion:
    conexion.execute("INSERT INTO pacientes (paciente_id) VALUES (?)", ("P001",))

    aceptada = endpoint_crear_medicion(
        {
            "medicion_id": "M001",
            "paciente_id": "P001",
            "valor": "1.2",
            "unidad": "mg/dL",
            "estado": "medido",
        },
        conexion,
    )
    rechazada = endpoint_crear_medicion(
        {
            "medicion_id": "M002",
            "paciente_id": "P999",
            "valor": "1.4",
            "unidad": "mg/dL",
            "estado": "medido",
        },
        conexion,
    )

print(aceptada)
print(rechazada)
```

Salida esperada:

```text
{'estado': 'aceptada', 'medicion_id': 'M001', 'contrato_version': '1.0.0'}
{'estado': 'rechazada', 'errores': [{'campo': 'base', 'razon': 'IntegrityError'}], 'contrato_version': '1.0.0'}
```

La segunda solicitud tenía forma correcta, pero apuntaba a un paciente inexistente. La validación de API pasó; la restricción relacional falló. Eso muestra por qué las capas se complementan.

## Pruebas mínimas

Una API mínima se puede probar sin servidor.

```python
with crear_base_minima() as conexion:
    conexion.execute("INSERT INTO pacientes (paciente_id) VALUES (?)", ("P001",))

    respuesta = endpoint_crear_medicion(
        {
            "medicion_id": "M001",
            "paciente_id": "P001",
            "valor": "1.2",
            "unidad": "mg/dL",
            "estado": "medido",
        },
        conexion,
    )
    assert respuesta["estado"] == "aceptada"
    assert respuesta["medicion_id"] == "M001"

    respuesta = endpoint_crear_medicion(
        {
            "medicion_id": "M002",
            "paciente_id": "P001",
            "valor": "1.2",
            "unidad": "mmol/L",
            "estado": "medido",
        },
        conexion,
    )
    assert respuesta["estado"] == "rechazada"
    assert respuesta["errores"][0]["razon"] == "unidad_no_soportada"
```

Salida esperada: no imprime nada si las propiedades se cumplen.

La prueba no valida una API clínica real. Verifica dos propiedades mínimas: una entrada válida se acepta y una unidad fuera del contrato se rechaza con razón explícita.

## Errores frecuentes

**Confundir endpoint con contrato.** La ruta o el nombre de función no bastan; hay que declarar entrada, salida y errores.

**Aceptar diccionarios crudos.** Una solicitud externa debe validarse antes de tocar la base.

**Devolver solo `ok`.** Una respuesta responsable debe conservar estado, identificador, errores y versión de contrato.

**Ocultar errores relacionales.** Si la base rechaza una fila por clave foránea, la API debe traducirlo a una respuesta comprensible.

**Mezclar servidor, dominio y base.** El endpoint debe coordinar; el dominio valida; la base persiste y defiende restricciones.

**Publicar operaciones sin límites.** Una API real necesita autenticación, permisos, auditoría, límites de tasa y privacidad. Esta miniatura no cubre esas capas.

## Argumentos críticos

### Desacuerdo 1: función local contra API

Pregunta: ¿cuándo una función se vuelve API?

Cuando otro componente empieza a depender de su forma de entrada y salida. Puede ser otro módulo, un script, una interfaz, un notebook o un cliente HTTP. La frontera aparece antes que el servidor.

Consenso operativo: tratar como API toda función pública cuyo contrato no pueda cambiarse sin afectar consumidores.

### Desacuerdo 2: validación en API contra validación en base

Pregunta: ¿dónde debe vivir la validación?

La API debe validar forma, contrato y errores comunicables. La base debe proteger invariantes mínimas. El dominio debe definir significado. Ninguna capa reemplaza por completo a las otras.

Consenso operativo: frontera clara, dominio explícito y persistencia defensiva.

### Desacuerdo 3: miniatura sin servidor contra realismo

Pregunta: ¿por qué no usar FastAPI, Flask o Django desde ya?

Porque el objetivo aquí es entender el contrato antes de agregar infraestructura. Un framework puede ser excelente, pero si el contrato conceptual está mal, solo lo ejecutará con más comodidad.

Consenso operativo: aprender el flujo sin servidor; luego mapearlo a HTTP cuando el diseño esté claro.

## Puente hacia sistemas reales

Una API real agregará capas que esta sección no cubre: autenticación, autorización, privacidad, logs, auditoría, paginación, límites de carga, versionamiento de endpoints, documentación formal y manejo de concurrencia.

Pero esas capas se apoyan en lo aprendido aquí. Si una solicitud no tiene contrato, si una respuesta no declara estado, si una operación no conserva identidad o si un error no deja razón, la infraestructura no corrige el problema de fondo.

Más adelante, esta línea permitirá entrar a sistemas biomédicos completos: formularios, servicios de consulta, pipelines reproducibles, dashboards, motores de reglas y soporte de decisión. La API será una puerta; el contrato seguirá siendo la base.

## Preguntas de comprensión profunda

1. ¿Por qué una API no es solo una URL?
2. ¿Qué información mínima debe declarar un contrato de entrada?
3. ¿Por qué una API que devuelve solo `ok` es pobre para auditoría?
4. ¿Qué diferencia hay entre validar forma de solicitud y validar relación en la base?
5. ¿Por qué conviene probar una API mínima sin levantar servidor?
6. ¿Qué error del ejemplo solo pudo detectar la base de datos?
7. ¿Qué cambiaría al convertir esta miniatura en un endpoint HTTP real?
8. ¿Qué riesgos aparecen cuando una API acepta datos biomédicos sin autenticación ni auditoría?

## Vacíos de comprensión que debes vigilar

1. Pensar que API significa necesariamente internet. Una API es una frontera contractual entre componentes.
2. Creer que un framework resuelve el diseño del dominio. El framework transporta solicitudes; no decide qué significa el dato.
3. Olvidar que los errores también son parte del contrato. Una API responsable especifica cómo rechaza.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** escribe el contrato de entrada y salida para crear una medición, sin código.
2. **Segunda hora:** implementa una función que valide la solicitud y devuelva errores estructurados.
3. **Tercera hora:** conecta esa función con SQLite y prueba una aceptación, un rechazo por unidad y un rechazo por clave foránea.

## Bibliografía y fuentes

- Fielding, R. T. (2000). *Architectural styles and the design of network-based software architectures* [Doctoral dissertation, University of California, Irvine].
- Python Software Foundation. [sqlite3: DB-API 2.0 interface for SQLite databases](https://docs.python.org/3/library/sqlite3.html).
- Richardson, L., Amundsen, M., & Ruby, S. (2013). *RESTful Web APIs*. O'Reilly Media.
- OpenAPI Initiative. [OpenAPI Specification](https://spec.openapis.org/oas/latest.html).

## Siguiente paso

Una API mínima convierte contratos internos en operaciones disponibles para otros componentes. La siguiente sección puede avanzar hacia análisis reproducibles: cómo registrar datos, consultas, parámetros y resultados para que un cálculo pueda reconstruirse sin depender de memoria manual.
