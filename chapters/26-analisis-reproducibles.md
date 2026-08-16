# Análisis reproducibles

La sección anterior mostró una API mínima como frontera contractual: una operación disponible para otros componentes, con entrada, validación, salida y errores explícitos.

Esa frontera permite pedir una operación. Pero todavía falta una pregunta más exigente: si esa operación produce un cálculo, un resumen o una tabla derivada, ¿podemos reconstruir exactamente cómo se obtuvo?

En medicina y ciencias de la vida, un resultado aislado rara vez basta. Un promedio, una proporción, una alerta agregada o una lista de casos elegibles depende de datos, filtros, parámetros, consultas, versiones de reglas y denominadores. Si esos elementos no quedan registrados, el resultado puede verse preciso y aun así ser imposible de auditar.

Ahí aparece el análisis reproducible.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un análisis reproducible es un cálculo cuyo resultado puede reconstruirse a partir de entradas, parámetros, consultas, reglas, versiones y salidas registradas. No exige que el mundo no cambie; exige que el procedimiento aplicado a una versión definida de los datos pueda repetirse y explicar el mismo resultado.
</div>

Esta definición separa tres ideas.

Primero, reproducible no significa verdadero. Un análisis puede estar perfectamente registrado y aun así usar una población equivocada.

Segundo, reproducible no significa automático. Un script ejecutable ayuda, pero si no conserva parámetros, versión de datos y denominador, sigue siendo débil.

Tercero, reproducible no significa eterno. Un análisis puede depender de una versión concreta de datos, código y reglas. Si alguno cambia, debe quedar visible.

## Versión ingenua: calcular y copiar el resultado

Supongamos que queremos calcular la creatinina promedio de pacientes de urgencias.

```python
mediciones = [
    {"paciente_id": "P001", "servicio": "urgencias", "valor": 1.2, "estado": "medido"},
    {"paciente_id": "P002", "servicio": "urgencias", "valor": 1.5, "estado": "medido"},
    {"paciente_id": "P003", "servicio": "consulta", "valor": 0.9, "estado": "medido"},
]

valores = [fila["valor"] for fila in mediciones if fila["servicio"] == "urgencias"]
promedio = sum(valores) / len(valores)

print(round(promedio, 2))
```

Salida esperada:

```text
1.35
```

El cálculo es correcto para esa lista. Pero el resultado `1.35` no conserva qué población se filtró, cuántas filas entraron, qué estados fueron excluidos, qué versión de datos se usó ni qué regla definía "urgencias".

Si alguien encuentra ese número en un informe, no puede reconstruirlo sin depender de memoria manual.

## Crítica técnica: el resultado no es el análisis

Un resultado derivado debería responder al menos estas preguntas:

**Qué se calculó.** Nombre del análisis, objetivo y versión de la especificación.

**Sobre qué datos.** Fuente, versión, fecha de corte o identificador de carga.

**Con qué criterio.** Consulta, filtros, estados incluidos, unidades aceptadas y regla de exclusión.

**Con qué parámetros.** Servicio, ventana temporal, población, umbrales o grupo analítico.

**Qué salió.** Resultado, denominador, conteos auxiliares, errores o advertencias.

Una tabla final sin esos metadatos se parece a una respuesta, pero no a una evidencia reconstruible.

## Especificación mínima de análisis

Antes de ejecutar, conviene representar el análisis como dato.

```python
ESPECIFICACION_ANALISIS = {
    "analisis_id": "creatinina_promedio_por_servicio",
    "version": "1.0.0",
    "datos_version": "carga_laboratorio_2026_08_15",
    "consulta": "creatinina_medida_por_servicio",
    "parametros_requeridos": ["servicio"],
    "unidad_esperada": "mg/dL",
}

print(ESPECIFICACION_ANALISIS["analisis_id"])
print(ESPECIFICACION_ANALISIS["datos_version"])
```

Salida esperada:

```text
creatinina_promedio_por_servicio
carga_laboratorio_2026_08_15
```

La especificación no reemplaza el código. Declara qué operación analítica estamos intentando ejecutar y bajo qué versión.

## Huella estable de parámetros

Para reconstruir un análisis, los parámetros deben guardarse en una forma ordenada. En Python, `json.dumps(..., sort_keys=True)` permite serializar un diccionario con orden estable; `hashlib` permite crear una huella corta para identificar la combinación.

```python
import hashlib
import json


def huella_estable(objeto):
    texto = json.dumps(objeto, sort_keys=True, ensure_ascii=False)
    return hashlib.sha256(texto.encode("utf-8")).hexdigest()[:12]


parametros = {"servicio": "urgencias", "estado": "medido"}

print(json.dumps(parametros, sort_keys=True, ensure_ascii=False))
print(huella_estable(parametros))
```

Salida esperada:

```text
{"estado": "medido", "servicio": "urgencias"}
40aff1e90ed6
```

La huella no es una prueba de verdad. Es un identificador técnico para saber si dos ejecuciones usaron la misma configuración serializada.

## Registrar ejecución, resultado y denominador

Ahora conectamos la idea con SQLite. La base no solo guarda mediciones; también guarda la ejecución del análisis.

```python
import hashlib
import json
import sqlite3

ESPECIFICACION_ANALISIS = {
    "analisis_id": "creatinina_promedio_por_servicio",
    "version": "1.0.0",
    "datos_version": "carga_laboratorio_2026_08_15",
    "consulta": "creatinina_medida_por_servicio",
    "unidad_esperada": "mg/dL",
}


def huella_estable(objeto):
    texto = json.dumps(objeto, sort_keys=True, ensure_ascii=False)
    return hashlib.sha256(texto.encode("utf-8")).hexdigest()[:12]


def crear_base():
    conexion = sqlite3.connect(":memory:")
    conexion.execute(
        """
        CREATE TABLE mediciones (
            medicion_id TEXT PRIMARY KEY,
            paciente_id TEXT NOT NULL,
            servicio TEXT NOT NULL,
            valor REAL,
            unidad TEXT NOT NULL,
            estado TEXT NOT NULL
        )
        """
    )
    conexion.execute(
        """
        CREATE TABLE ejecuciones_analisis (
            ejecucion_id TEXT PRIMARY KEY,
            analisis_id TEXT NOT NULL,
            analisis_version TEXT NOT NULL,
            datos_version TEXT NOT NULL,
            parametros_json TEXT NOT NULL,
            resultado_json TEXT NOT NULL
        )
        """
    )
    return conexion


def calcular_creatinina_promedio(conexion, parametros):
    filas = conexion.execute(
        """
        SELECT valor
        FROM mediciones
        WHERE servicio = ?
          AND estado = 'medido'
          AND unidad = ?
        """,
        (parametros["servicio"], ESPECIFICACION_ANALISIS["unidad_esperada"]),
    ).fetchall()

    valores = [fila[0] for fila in filas]
    denominador = len(valores)
    promedio = round(sum(valores) / denominador, 2) if denominador else None

    return {
        "servicio": parametros["servicio"],
        "promedio_creatinina": promedio,
        "unidad": ESPECIFICACION_ANALISIS["unidad_esperada"],
        "denominador_medidos": denominador,
    }


with crear_base() as conexion:
    conexion.executemany(
        """
        INSERT INTO mediciones
        (medicion_id, paciente_id, servicio, valor, unidad, estado)
        VALUES (?, ?, ?, ?, ?, ?)
        """,
        [
            ("M001", "P001", "urgencias", 1.2, "mg/dL", "medido"),
            ("M002", "P002", "urgencias", 1.5, "mg/dL", "medido"),
            ("M003", "P003", "urgencias", None, "mg/dL", "pendiente"),
            ("M004", "P004", "consulta", 0.9, "mg/dL", "medido"),
        ],
    )

    parametros = {"servicio": "urgencias"}
    resultado = calcular_creatinina_promedio(conexion, parametros)
    ejecucion_id = huella_estable(
        {
            "analisis": ESPECIFICACION_ANALISIS,
            "parametros": parametros,
            "resultado": resultado,
        }
    )

    conexion.execute(
        """
        INSERT INTO ejecuciones_analisis
        (ejecucion_id, analisis_id, analisis_version, datos_version, parametros_json, resultado_json)
        VALUES (?, ?, ?, ?, ?, ?)
        """,
        (
            ejecucion_id,
            ESPECIFICACION_ANALISIS["analisis_id"],
            ESPECIFICACION_ANALISIS["version"],
            ESPECIFICACION_ANALISIS["datos_version"],
            json.dumps(parametros, sort_keys=True, ensure_ascii=False),
            json.dumps(resultado, sort_keys=True, ensure_ascii=False),
        ),
    )

    guardado = conexion.execute(
        "SELECT ejecucion_id, resultado_json FROM ejecuciones_analisis"
    ).fetchone()

print(resultado)
print(guardado)
```

Salida esperada:

```text
{'servicio': 'urgencias', 'promedio_creatinina': 1.35, 'unidad': 'mg/dL', 'denominador_medidos': 2}
('cef522204e86', '{"denominador_medidos": 2, "promedio_creatinina": 1.35, "servicio": "urgencias", "unidad": "mg/dL"}')
```

El promedio ya no viaja solo. Viaja con el servicio filtrado, la unidad, el denominador y una fila de ejecución que conserva especificación, versión de datos, parámetros y resultado.

## Reproducir no es recalcular a ciegas

La reproducción exige comparar la ejecución nueva con la ejecución registrada.

```python
def comparar_resultados(resultado_previo, resultado_nuevo):
    if resultado_previo == resultado_nuevo:
        return {"estado": "reproducido", "diferencias": []}

    diferencias = []
    for campo in sorted(set(resultado_previo) | set(resultado_nuevo)):
        if resultado_previo.get(campo) != resultado_nuevo.get(campo):
            diferencias.append(
                {
                    "campo": campo,
                    "previo": resultado_previo.get(campo),
                    "nuevo": resultado_nuevo.get(campo),
                }
            )
    return {"estado": "divergente", "diferencias": diferencias}


previo = {"promedio_creatinina": 1.35, "denominador_medidos": 2}
nuevo = {"promedio_creatinina": 1.4, "denominador_medidos": 3}

print(comparar_resultados(previo, nuevo))
```

Salida esperada:

```text
{'estado': 'divergente', 'diferencias': [{'campo': 'denominador_medidos', 'previo': 2, 'nuevo': 3}, {'campo': 'promedio_creatinina', 'previo': 1.35, 'nuevo': 1.4}]}
```

Una divergencia no siempre es un error. Puede indicar que los datos cambiaron, que la regla cambió o que el cálculo anterior estaba incompleto. Lo importante es que la diferencia quede nombrada.

## CODE CLEAN: cálculo delgado, registro explícito

La versión frágil mezcla selección, cálculo, impresión y memoria externa.

```python
def reporte_creatinina(mediciones):
    valores = [m["valor"] for m in mediciones if m["servicio"] == "urgencias"]
    print(sum(valores) / len(valores))
```

La versión más limpia separa responsabilidades.

```python
def seleccionar_mediciones(conexion, parametros):
    return conexion.execute(CONSULTA_CREATININA, parametros).fetchall()


def resumir_mediciones(filas):
    valores = [fila["valor"] for fila in filas]
    return construir_resultado(valores)


def registrar_ejecucion(conexion, especificacion, parametros, resultado):
    ejecucion = construir_registro_ejecucion(especificacion, parametros, resultado)
    guardar_ejecucion(conexion, ejecucion)
    return ejecucion
```

La limpieza no consiste en escribir más funciones por costumbre. Consiste en que cada parte pueda verificarse: selección, resumen y registro.

## Prueba mínima

Un análisis reproducible debe probar al menos el cálculo y el denominador.

```python
with crear_base() as conexion:
    conexion.executemany(
        """
        INSERT INTO mediciones
        (medicion_id, paciente_id, servicio, valor, unidad, estado)
        VALUES (?, ?, ?, ?, ?, ?)
        """,
        [
            ("M001", "P001", "urgencias", 1.2, "mg/dL", "medido"),
            ("M002", "P002", "urgencias", 1.5, "mg/dL", "medido"),
            ("M003", "P003", "urgencias", None, "mg/dL", "pendiente"),
        ],
    )

    resultado = calcular_creatinina_promedio(conexion, {"servicio": "urgencias"})

assert resultado["promedio_creatinina"] == 1.35
assert resultado["denominador_medidos"] == 2
assert resultado["unidad"] == "mg/dL"
```

Salida esperada: no imprime nada si las propiedades se cumplen.

La prueba impide un error común: incluir mediciones pendientes en el denominador o mezclar unidades compatibles solo en apariencia.

## Errores frecuentes

**Guardar solo el resultado final.** Un número sin parámetros ni denominador es débil como evidencia.

**No versionar los datos.** Si la fuente cambia, dos ejecuciones pueden producir resultados distintos sin que el código haya cambiado.

**No guardar la consulta.** Un análisis descrito verbalmente puede variar entre autores, equipos o fechas.

**Confundir reproducibilidad con validez.** Reproducir un cálculo no prueba que la población, el modelo o la interpretación sean adecuados.

**No registrar exclusiones.** En datos biomédicos, lo que queda fuera del análisis puede importar tanto como lo que entra.

**Usar la fecha del día como única trazabilidad.** La hora de ejecución ayuda, pero no reemplaza versión de datos, parámetros ni regla.

## Argumentos críticos

### Desacuerdo 1: script contra especificación

Pregunta: ¿basta con guardar el script?

No siempre. El script puede recibir parámetros externos, leer una tabla que cambió o depender de reglas implícitas. Guardar el script ayuda, pero el análisis necesita también datos, parámetros, versiones y resultado.

Consenso operativo: tratar el script como una parte del análisis, no como el análisis completo.

### Desacuerdo 2: reproducibilidad contra exploración

Pregunta: ¿todo análisis exploratorio debe registrarse con este nivel?

No. Durante la exploración puede haber cálculos rápidos y descartables. Pero cuando un resultado entra a un informe, una decisión, una comparación o una publicación, debe poder reconstruirse.

Consenso operativo: cuanto mayor sea la consecuencia del resultado, mayor debe ser la trazabilidad.

### Desacuerdo 3: metadatos contra fricción

Pregunta: ¿registrar tanto no vuelve lento el trabajo?

Sí, puede añadir fricción. Pero la alternativa es producir resultados que nadie puede explicar después. La solución no es eliminar metadatos, sino automatizar su captura mínima.

Consenso operativo: registrar lo indispensable en estructuras pequeñas y repetibles.

## Puente hacia sistemas reales

Los análisis reproducibles son el puente natural entre tablas, bases de datos, APIs y sistemas científicos más grandes. Más adelante, la misma idea aparecerá en notebooks, pipelines, modelos predictivos, bioinformática, análisis de señales y evaluación de sistemas de soporte de decisión.

La escala cambiará. Los elementos mínimos seguirán siendo reconocibles: datos versionados, parámetros explícitos, reglas identificables, resultados con denominador, errores visibles y una forma de reconstruir el camino.

## Preguntas de comprensión profunda

1. ¿Por qué un resultado correcto puede no ser reproducible?
2. ¿Qué diferencia hay entre guardar un número y guardar una ejecución de análisis?
3. ¿Por qué el denominador es parte del resultado y no solo un detalle secundario?
4. ¿Qué elementos mínimos debe conservar una especificación de análisis?
5. ¿Por qué una huella estable no prueba que el análisis sea válido?
6. ¿Qué puede significar una divergencia entre dos ejecuciones?
7. ¿Cuándo un análisis exploratorio debería convertirse en análisis registrado?
8. ¿Qué riesgos aparecen si una API entrega resultados derivados sin registrar parámetros?

## Vacíos de comprensión que debes vigilar

1. Pensar que reproducible significa clínicamente correcto. La reproducibilidad permite reconstruir; la validez exige evaluar población, método e interpretación.
2. Creer que un script basta por sí solo. Sin versión de datos y parámetros, el script puede producir otra respuesta.
3. Olvidar los denominadores. En biomedicina, el conteo de casos incluidos y excluidos condiciona la interpretación.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma un cálculo simple y escribe su especificación antes de programarlo.
2. **Segunda hora:** ejecuta el cálculo guardando parámetros, resultado y denominador.
3. **Tercera hora:** cambia un dato o parámetro y registra qué diferencia aparece entre ejecuciones.

## Bibliografía y fuentes

- Peng, R. D. (2011). Reproducible research in computational science. *Science*, 334(6060), 1226-1227.
- Sandve, G. K., Nekrutenko, A., Taylor, J., & Hovig, E. (2013). Ten simple rules for reproducible computational research. *PLoS Computational Biology*, 9(10), e1003285.
- Python Software Foundation. [json: JSON encoder and decoder](https://docs.python.org/3/library/json.html).
- Python Software Foundation. [hashlib: secure hashes and message digests](https://docs.python.org/3/library/hashlib.html).
- Python Software Foundation. [sqlite3: DB-API 2.0 interface for SQLite databases](https://docs.python.org/3/library/sqlite3.html).

## Siguiente paso

Un análisis reproducible convierte consultas, parámetros y resultados en una ejecución reconstruible. La siguiente sección puede avanzar hacia pipelines mínimos: cómo encadenar pasos de carga, validación, análisis y reporte sin perder trazabilidad entre etapas.
