# Exportación, auditoría y artefactos compartibles

La sección anterior convirtió un lote de registros en un reporte de calidad: filas recibidas, aceptadas, rechazadas, advertencias, razones y completitud. Ese reporte permite decidir si el lote puede avanzar.

Pero una decisión que solo vive en la memoria de Python desaparece cuando termina el proceso. Si otra persona necesita revisar el resultado, si un segundo programa debe consumirlo o si el equipo debe reconstruir lo ocurrido una semana después, hacen falta artefactos persistentes.

Exportar no consiste únicamente en llamar a `to_csv()` o escribir un diccionario en un archivo. En un flujo biomédico, una salida compartible debe conservar estructura, versión, procedencia, conteos y una forma de detectar cambios posteriores.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un artefacto compartible es una salida persistente con formato, propósito y contrato explícitos. Una exportación auditable añade metadatos suficientes para identificar qué se produjo, con qué versión, a partir de qué lote y si los archivos conservan el contenido original.
</div>

La palabra *compartible* describe una propiedad técnica: otra herramienta o persona puede abrir el artefacto. No significa que el archivo sea público, anónimo, autorizado ni seguro para enviarlo fuera de su entorno.

## Versión ingenua: guardar lo que salió

Una exportación frágil puede reducirse a convertir un objeto en texto.

```python
reporte = {
    "estado_lote": "requiere_correccion",
    "filas_validas": 2,
    "filas_rechazadas": 2,
}

with open("resultado.txt", "w", encoding="utf-8") as archivo:
    archivo.write(str(reporte))

print("resultado.txt")
```

Salida esperada:

```text
resultado.txt
```

El archivo existe, pero su contrato es débil:

- usa la representación interna de un diccionario de Python;
- no declara versión del esquema;
- no identifica el lote ni el procedimiento;
- no conserva la lista de filas aceptadas o rechazadas;
- no permite detectar si alguien modificó el contenido;
- no diferencia un resultado completo de una escritura interrumpida.

Guardar no es todavía exportar de forma defendible.

## Crítica técnica: una salida necesita contrato

Antes de elegir formato conviene responder cinco preguntas.

**Qué contiene.** Una tabla de filas aceptadas no tiene el mismo contrato que un reporte agregado.

**Quién la consumirá.** Una persona puede preferir CSV; un programa puede necesitar JSON; una aplicación local puede requerir SQLite.

**Qué versión representa.** Si las columnas o reglas cambian, el consumidor debe poder reconocerlo.

**De dónde proviene.** El identificador del lote, la versión del pipeline y los conteos mínimos conectan el archivo con su ejecución.

**Cómo se verifica.** Una huella de contenido permite detectar cambios accidentales o deliberados después de la exportación.

El formato es una decisión de interoperabilidad, no una decoración del nombre del archivo.

## Elegir el formato según la estructura

Tres formatos cubren buena parte de los casos iniciales.

| Formato | Adecuado para | Límite principal |
|---|---|---|
| CSV | filas y columnas simples | no expresa bien jerarquías, tipos complejos ni metadatos |
| JSON | reportes, manifiestos y estructuras anidadas | requiere un contrato para tipos, claves y valores especiales |
| SQLite | varias tablas relacionadas y consultas locales | exige esquema, migraciones, integridad y procedimiento de respaldo |

Un mismo paquete puede usar más de un formato: CSV para observaciones, JSON para el reporte y un manifiesto JSON para describir el conjunto.

## JSON estable para reportes

El módulo `json` permite serializar estructuras básicas. Para favorecer comparación y lectura usamos claves ordenadas, indentación, UTF-8 y rechazo de valores no finitos.

```python
import json
from pathlib import Path


def escribir_json(ruta, contenido):
    texto = json.dumps(
        contenido,
        ensure_ascii=False,
        indent=2,
        sort_keys=True,
        allow_nan=False,
    )
    Path(ruta).write_text(texto + "\n", encoding="utf-8")


reporte = {
    "schema_version": "reporte-calidad-v1",
    "estado_lote": "requiere_correccion",
    "total_filas": 4,
    "filas_validas": 2,
    "filas_rechazadas": 2,
}

escribir_json("reporte_calidad.json", reporte)
print(json.loads(Path("reporte_calidad.json").read_text(encoding="utf-8"))["schema_version"])
```

Salida esperada:

```text
reporte-calidad-v1
```

`allow_nan=False` evita producir valores como `NaN` o `Infinity`, que no pertenecen al estándar JSON estricto. Si el dominio necesita representar un valor no calculable, debe hacerlo mediante un estado explícito, no mediante un número especial silencioso.

## CSV con columnas declaradas

En CSV, el orden y el nombre de las columnas forman parte del contrato.

```python
import csv
from pathlib import Path


def escribir_csv(ruta, filas, columnas):
    with Path(ruta).open("w", encoding="utf-8", newline="") as archivo:
        escritor = csv.DictWriter(archivo, fieldnames=columnas, extrasaction="raise")
        escritor.writeheader()
        escritor.writerows(filas)


filas_aceptadas = [
    {"paciente_id": "P001", "valor": 1.2, "unidad": "mg/dL", "estado": "medido"},
    {"paciente_id": "P004", "valor": 16.2, "unidad": "mg/dL", "estado": "medido"},
]

columnas = ["paciente_id", "valor", "unidad", "estado"]
escribir_csv("filas_aceptadas.csv", filas_aceptadas, columnas)

print(Path("filas_aceptadas.csv").read_text(encoding="utf-8").splitlines()[0])
```

Salida esperada:

```text
paciente_id,valor,unidad,estado
```

`extrasaction="raise"` evita ignorar silenciosamente un campo inesperado. No valida por sí solo el significado de cada valor, pero obliga a que la exportación respete las columnas declaradas.

## Huellas de contenido

Una función hash resume una secuencia de bytes. Si cambia un byte del archivo, la huella cambia con probabilidad extremadamente alta.

```python
import hashlib
from pathlib import Path


def sha256_archivo(ruta):
    digestor = hashlib.sha256()
    with Path(ruta).open("rb") as archivo:
        for bloque in iter(lambda: archivo.read(65536), b""):
            digestor.update(bloque)
    return digestor.hexdigest()


huella = sha256_archivo("filas_aceptadas.csv")
print(len(huella))
```

Salida esperada:

```text
64
```

La huella no demuestra que los datos sean correctos, verdaderos o autorizados. Solo permite verificar integridad de contenido respecto a una referencia previa.

## El manifiesto de exportación

Un manifiesto describe el paquete. Puede registrar:

- identificador del lote;
- versión del esquema de exportación;
- versión del pipeline;
- instante de creación;
- conteos principales;
- nombre, tamaño y huella de cada artefacto;
- política de sensibilidad o alcance de uso.

```python
def describir_artefacto(ruta):
    ruta = Path(ruta)
    return {
        "nombre": ruta.name,
        "bytes": ruta.stat().st_size,
        "sha256": sha256_archivo(ruta),
    }


artefacto = describir_artefacto("filas_aceptadas.csv")
print(artefacto["nombre"], artefacto["bytes"] > 0, len(artefacto["sha256"]))
```

Salida esperada:

```text
filas_aceptadas.csv True 64
```

El manifiesto funciona como índice verificable del paquete. No debe contener secretos innecesarios ni identificadores clínicos si su finalidad no los requiere.

## Construir un paquete compartible

La miniatura siguiente exporta filas aceptadas, rechazos y reporte de calidad. Después construye un manifiesto con las huellas de esos tres artefactos.

```python
import csv
import hashlib
import json
from pathlib import Path


def escribir_json(ruta, contenido):
    Path(ruta).write_text(
        json.dumps(
            contenido,
            ensure_ascii=False,
            indent=2,
            sort_keys=True,
            allow_nan=False,
        )
        + "\n",
        encoding="utf-8",
    )


def escribir_csv(ruta, filas, columnas):
    with Path(ruta).open("w", encoding="utf-8", newline="") as archivo:
        escritor = csv.DictWriter(archivo, fieldnames=columnas, extrasaction="raise")
        escritor.writeheader()
        escritor.writerows(filas)


def sha256_archivo(ruta):
    digestor = hashlib.sha256()
    with Path(ruta).open("rb") as archivo:
        for bloque in iter(lambda: archivo.read(65536), b""):
            digestor.update(bloque)
    return digestor.hexdigest()


def describir_artefacto(ruta):
    ruta = Path(ruta)
    return {
        "nombre": ruta.name,
        "bytes": ruta.stat().st_size,
        "sha256": sha256_archivo(ruta),
    }


def exportar_lote(
    directorio,
    filas_aceptadas,
    filas_rechazadas,
    reporte,
    metadatos,
):
    directorio = Path(directorio)
    directorio.mkdir(parents=True, exist_ok=True)

    ruta_aceptadas = directorio / "filas_aceptadas.csv"
    ruta_rechazadas = directorio / "filas_rechazadas.csv"
    ruta_reporte = directorio / "reporte_calidad.json"

    escribir_csv(
        ruta_aceptadas,
        filas_aceptadas,
        ["paciente_id", "valor", "unidad", "estado"],
    )
    escribir_csv(
        ruta_rechazadas,
        filas_rechazadas,
        ["posicion", "paciente_id", "razon"],
    )
    escribir_json(ruta_reporte, reporte)

    artefactos = [
        describir_artefacto(ruta)
        for ruta in [ruta_aceptadas, ruta_rechazadas, ruta_reporte]
    ]

    manifiesto = {
        "schema_version": "exportacion-lote-v1",
        "lote_id": metadatos["lote_id"],
        "pipeline_version": metadatos["pipeline_version"],
        "creado_en": metadatos["creado_en"],
        "alcance": metadatos["alcance"],
        "conteos": {
            "aceptadas": len(filas_aceptadas),
            "rechazadas": len(filas_rechazadas),
        },
        "artefactos": artefactos,
    }
    escribir_json(directorio / "manifiesto.json", manifiesto)
    return manifiesto


filas_aceptadas = [
    {"paciente_id": "P001", "valor": 1.2, "unidad": "mg/dL", "estado": "medido"},
    {"paciente_id": "P004", "valor": 16.2, "unidad": "mg/dL", "estado": "medido"},
]

filas_rechazadas = [
    {"posicion": 2, "paciente_id": "P002", "razon": "valor_ausente"},
    {"posicion": 3, "paciente_id": "P003", "razon": "unidad_no_soportada"},
]

reporte = {
    "schema_version": "reporte-calidad-v1",
    "estado_lote": "requiere_correccion",
    "total_filas": 4,
    "filas_validas": 2,
    "filas_rechazadas": 2,
}

metadatos = {
    "lote_id": "LOTE-2026-001",
    "pipeline_version": "pipeline-minimo-v1",
    "creado_en": "2026-08-17T00:00:00Z",
    "alcance": "ejemplo_pedagogico_sin_datos_reales",
}

manifiesto = exportar_lote(
    "salida_lote",
    filas_aceptadas,
    filas_rechazadas,
    reporte,
    metadatos,
)

print([item["nombre"] for item in manifiesto["artefactos"]])
print([len(item["sha256"]) for item in manifiesto["artefactos"]])
```

Salida esperada:

```text
['filas_aceptadas.csv', 'filas_rechazadas.csv', 'reporte_calidad.json']
[64, 64, 64]
```

El manifiesto no incluye su propia huella. Para registrar el paquete completo, un sistema exterior puede calcular la huella de `manifiesto.json`, firmarla o almacenarla en una bitácora separada.

## Auditar el paquete

Auditar significa comparar los artefactos actuales con lo que declara el manifiesto.

```python
def auditar_exportacion(directorio):
    directorio = Path(directorio)
    manifiesto = json.loads(
        (directorio / "manifiesto.json").read_text(encoding="utf-8")
    )
    hallazgos = []

    for artefacto in manifiesto["artefactos"]:
        ruta = directorio / artefacto["nombre"]
        if not ruta.exists():
            hallazgos.append(
                {"archivo": artefacto["nombre"], "razon": "archivo_ausente"}
            )
            continue

        if ruta.stat().st_size != artefacto["bytes"]:
            hallazgos.append(
                {"archivo": artefacto["nombre"], "razon": "tamano_modificado"}
            )

        if sha256_archivo(ruta) != artefacto["sha256"]:
            hallazgos.append(
                {"archivo": artefacto["nombre"], "razon": "huella_no_coincide"}
            )

    return {
        "valida": not hallazgos,
        "artefactos_esperados": len(manifiesto["artefactos"]),
        "hallazgos": hallazgos,
    }


auditoria = auditar_exportacion("salida_lote")
print(auditoria["valida"])
print(auditoria["artefactos_esperados"])
```

Salida esperada:

```text
True
3
```

La auditoría verifica presencia, tamaño y contenido. No verifica que el lote original haya sido correcto ni que la política de exportación haya sido adecuada.

## Detectar una modificación

Si el archivo cambia después de producir el manifiesto, la auditoría debe detectarlo.

```python
ruta = Path("salida_lote/filas_aceptadas.csv")
contenido_original = ruta.read_text(encoding="utf-8")

ruta.write_text(contenido_original + "P999,9.9,mg/dL,medido\n", encoding="utf-8")
auditoria_modificada = auditar_exportacion("salida_lote")

print(auditoria_modificada["valida"])
print(sorted({hallazgo["razon"] for hallazgo in auditoria_modificada["hallazgos"]}))

# Restauramos el ejemplo para no dejar el paquete alterado.
ruta.write_text(contenido_original, encoding="utf-8")
```

Salida esperada:

```text
False
['huella_no_coincide', 'tamano_modificado']
```

La huella convierte una alteración silenciosa en un hallazgo explícito.

## SQLite como artefacto compartible

SQLite puede ser útil cuando el paquete necesita varias tablas, claves, restricciones y consultas. Sin embargo, copiar un archivo de base de datos mientras otra conexión escribe puede producir una copia incoherente.

La biblioteca estándar de Python ofrece `Connection.backup()` para crear una copia consistente desde una conexión SQLite.

```python
import sqlite3


def respaldar_sqlite(origen, destino):
    with sqlite3.connect(origen) as conexion_origen:
        with sqlite3.connect(destino) as conexion_destino:
            conexion_origen.backup(conexion_destino)
```

Salida esperada: no imprime nada. Crea el respaldo cuando la base de origen existe y puede abrirse.

Una base compartible todavía necesita versión de esquema, reglas de migración, control de acceso y auditoría. El respaldo consistente resuelve la copia técnica; no resuelve el gobierno del dato.

## Privacidad: compartible no significa publicable

Antes de mover un artefacto fuera de su entorno deben revisarse, como mínimo:

- finalidad y destinatario;
- presencia de identificadores directos o indirectos;
- necesidad real de cada columna;
- reglas institucionales, éticas y legales;
- periodo de conservación;
- canal de transferencia;
- permisos de lectura y modificación;
- posibilidad de reconstruir identidad mediante combinación de variables.

Cambiar nombres por códigos no garantiza anonimización. Una fecha exacta, un evento raro, una localización y una combinación clínica pueden volver identificable un registro.

En este libro, los ejemplos usan datos inventados. En sistemas reales, la política de datos manda sobre la comodidad técnica de exportar.

## CODE CLEAN: separar cálculo, escritura y auditoría

Una función que calcula, escribe, comprime, publica y notifica en el mismo bloque es difícil de probar y peligrosa de reutilizar.

Versión frágil:

```python
def procesar_y_guardar(filas):
    reporte = {"total": len(filas)}
    open("reporte.txt", "w", encoding="utf-8").write(str(reporte))
    print("guardado")
```

Versión mejorada:

```python
def construir_resumen(filas):
    return {"total": len(filas)}


def guardar_resumen(ruta, resumen):
    escribir_json(ruta, resumen)


resumen = construir_resumen([{"id": 1}, {"id": 2}])
guardar_resumen("resumen.json", resumen)
print(resumen)
```

Salida esperada:

```text
{'total': 2}
```

La función pura construye el contenido. La función con efecto escribe. La auditoría verifica después. Cada responsabilidad puede probarse por separado.

## Prueba mínima

La prueba del paquete debe verificar tanto el contenido como la integridad.

```python
manifiesto = json.loads(
    Path("salida_lote/manifiesto.json").read_text(encoding="utf-8")
)
auditoria = auditar_exportacion("salida_lote")

assert manifiesto["schema_version"] == "exportacion-lote-v1"
assert manifiesto["conteos"] == {"aceptadas": 2, "rechazadas": 2}
assert len(manifiesto["artefactos"]) == 3
assert all(len(item["sha256"]) == 64 for item in manifiesto["artefactos"])
assert auditoria["valida"] is True
assert auditoria["hallazgos"] == []
```

Salida esperada: no imprime nada si el contrato se cumple.

Estas pruebas no certifican privacidad ni validez clínica. Certifican el contrato técnico mínimo del ejemplo.

## Errores frecuentes

**Usar la extensión como contrato.** Que un archivo termine en `.json` no garantiza claves, tipos ni versión.

**Exportar campos sin orden ni propósito.** En CSV, el encabezado también es una interfaz.

**Confiar en la fecha del archivo como única trazabilidad.** Puede cambiar al copiar, restaurar o sincronizar.

**Calcular huellas antes de terminar la escritura.** El manifiesto debe describir artefactos ya cerrados.

**Publicar el manifiesto junto con datos sensibles.** Los metadatos también pueden revelar información.

**Confundir integridad con validez.** Una huella correcta puede corresponder a un archivo perfectamente conservado y conceptualmente equivocado.

**Sobrescribir exportaciones previas sin versión.** Se pierde la posibilidad de comparar ejecuciones.

**Copiar una base SQLite activa como si fuera un archivo cualquiera.** Es preferible usar un respaldo consistente.

## Argumentos críticos

### Desacuerdo 1: un solo archivo contra un paquete

Pregunta: ¿no sería más simple poner todo en un JSON?

Puede serlo para flujos pequeños. Sin embargo, las tablas grandes se inspeccionan mejor como CSV o base de datos, mientras los reportes y metadatos encajan mejor en JSON.

Consenso operativo: usar el menor número de artefactos que preserve contratos claros, no el menor número de archivos a cualquier costo.

### Desacuerdo 2: huella contra firma

Pregunta: ¿SHA-256 demuestra quién produjo el archivo?

No. Una huella permite detectar diferencias respecto a un valor conocido. No prueba identidad del autor ni autorización. Para eso hacen falta mecanismos adicionales, como firmas digitales, controles de acceso o registros externos confiables.

Consenso operativo: huella para integridad; identidad y autorización requieren controles separados.

### Desacuerdo 3: artefacto legible contra artefacto seguro

Pregunta: ¿si un CSV se abre fácilmente, puede compartirse?

No necesariamente. La facilidad de apertura aumenta interoperabilidad, pero también puede ampliar exposición. La seguridad depende de contenido, permisos, finalidad y canal.

Consenso operativo: primero gobernar el dato; después elegir el formato de intercambio.

## Puente hacia sistemas reales

En sistemas mayores, el mismo patrón crece hacia catálogos de datos, almacenamiento de objetos, versionado de datasets, firmas, procedencia formal, registros inmutables y controles de acceso. El paquete pedagógico de esta sección conserva la idea esencial:

```text
contenido + contrato + procedencia + integridad + política de uso
```

Sin ese conjunto, una exportación puede ser fácil de abrir y difícil de defender.

## Preguntas de comprensión profunda

1. ¿Qué diferencia hay entre guardar un objeto y exportar un artefacto con contrato?
2. ¿Por qué CSV y JSON resuelven problemas estructurales distintos?
3. ¿Qué información mínima debe conservar un manifiesto?
4. ¿Qué detecta una huella SHA-256 y qué no puede demostrar?
5. ¿Por qué un artefacto técnicamente compartible puede seguir siendo privado?
6. ¿Qué ventaja ofrece separar construcción, escritura y auditoría?
7. ¿Por qué una copia de SQLite requiere más cuidado que un CSV cerrado?
8. ¿Cómo ayuda el manifiesto a reconstruir una ejecución?

## Vacíos de comprensión que debes vigilar

1. Pensar que serializar equivale a documentar. El formato no reemplaza el contrato.
2. Pensar que una huella valida el contenido clínico. Solo verifica integridad respecto a una referencia.
3. Pensar que quitar nombres vuelve público un conjunto de datos. La reidentificación puede usar combinaciones indirectas.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** exporta una tabla simple a CSV con columnas declaradas y un reporte a JSON con versión.
2. **Segunda hora:** calcula huellas SHA-256 y construye un manifiesto con conteos y metadatos.
3. **Tercera hora:** altera una copia del archivo, ejecuta la auditoría y explica qué detectó y qué sigue sin demostrar.

## Bibliografía y fuentes

- Python Software Foundation. (s. f.). *csv — CSV File Reading and Writing*. <https://docs.python.org/3/library/csv.html>.
- Python Software Foundation. (s. f.). *hashlib — Secure hashes and message digests*. <https://docs.python.org/3/library/hashlib.html>.
- Python Software Foundation. (s. f.). *json — JSON encoder and decoder*. <https://docs.python.org/3/library/json.html>.
- Python Software Foundation. (s. f.). *sqlite3 — DB-API 2.0 interface for SQLite databases*. <https://docs.python.org/3/library/sqlite3.html#sqlite3.Connection.backup>.
- World Wide Web Consortium. (2013). *PROV-Overview: An Overview of the PROV Family of Documents*. <https://www.w3.org/TR/prov-overview/>.

## Siguiente paso

El Capítulo II ya recorrió desde tipos y estados hasta tablas, esquemas, persistencia, análisis, pipelines, reportes de calidad y artefactos auditables. La siguiente sección cerrará el capítulo integrando esas piezas en una capacidad única: transformar un dato biomédico crudo en un flujo verificable, con contratos y límites explícitos, antes de entrar a algoritmos clásicos con lectura biomédica.
