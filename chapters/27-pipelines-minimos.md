# Pipelines mínimos

La sección anterior cerró una idea importante: un análisis reproducible no es solo un número correcto. Es una ejecución reconstruible: datos, parámetros, reglas, consulta, denominador y resultado.

Pero un análisis rara vez aparece solo. Antes de calcular hay que cargar datos, revisar forma, normalizar valores, separar registros válidos, conservar rechazos y producir una salida que otro componente pueda leer.

Cuando esas etapas se encadenan aparece un pipeline.

La palabra puede sonar a infraestructura grande: servicios distribuidos, colas, orquestadores, contenedores, monitoreo y despliegue continuo. Todo eso existe, pero no es el punto de entrada. Antes de pensar en herramientas, conviene entender la idea mínima: un pipeline es una secuencia explícita de transformaciones donde cada etapa recibe algo, produce algo y deja evidencia suficiente para revisar el flujo.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un pipeline mínimo es una secuencia ordenada de etapas que transforma datos de entrada en artefactos de salida, conservando contratos, conteos, errores y resultados intermedios suficientes para reconstruir qué ocurrió en cada paso.
</div>

Esta definición evita dos errores comunes.

Primero, un pipeline no es cualquier script largo. Si no declara etapas, entradas, salidas y fallos, solo es una acumulación de instrucciones.

Segundo, un pipeline no necesita empezar con una herramienta especializada. Puede comenzar como funciones pequeñas conectadas por estructuras explícitas.

## Versión ingenua: hacer todo en una sola celda

Supongamos que recibimos un CSV de laboratorio y queremos calcular creatinina promedio en urgencias.

```python
import csv
import io

texto_csv = """paciente_id,servicio,valor,unidad,estado
P001,urgencias,1.2,mg/dL,medido
P002,urgencias,1.5,mg/dL,medido
P003,urgencias,,mg/dL,pendiente
"""

filas = list(csv.DictReader(io.StringIO(texto_csv)))
valores = []

for fila in filas:
    if fila["servicio"] == "urgencias" and fila["estado"] == "medido":
        valores.append(float(fila["valor"]))

print(round(sum(valores) / len(valores), 2))
```

Salida esperada:

```text
1.35
```

El resultado sale. Pero el flujo no declara dónde terminó la carga, dónde empezó la normalización, cuántas filas se rechazaron, qué reglas se aplicaron ni qué artefactos produjo cada paso.

Para un ejercicio corto puede bastar. Para un proceso biomédico que debe revisarse, no.

## Crítica técnica: cada etapa debe tener frontera

Un pipeline mínimo necesita al menos cinco fronteras:

**Carga.** Convertir una fuente externa en una estructura de trabajo.

**Normalización.** Preparar valores crudos para comparación sin borrar su origen conceptual.

**Validación.** Separar registros aceptables de registros rechazados con razones.

**Análisis.** Calcular resultados sobre datos ya interpretados.

**Reporte.** Producir una salida que conserve resultado, denominador, conteos y advertencias.

Si esas etapas se mezclan, el resultado puede ser útil hoy y opaco mañana.

## Declarar el flujo como dato

Antes de escribir funciones, podemos declarar el flujo esperado.

```python
PIPELINE_LABORATORIO = [
    {"etapa": "carga", "entrada": "csv_laboratorio", "salida": "filas_crudas"},
    {"etapa": "normalizacion", "entrada": "filas_crudas", "salida": "filas_normalizadas"},
    {"etapa": "validacion", "entrada": "filas_normalizadas", "salida": "filas_validas_y_rechazadas"},
    {"etapa": "analisis", "entrada": "filas_validas", "salida": "resultado_derivado"},
    {"etapa": "reporte", "entrada": "resultado_derivado", "salida": "reporte_final"},
]

print([paso["etapa"] for paso in PIPELINE_LABORATORIO])
```

Salida esperada:

```text
['carga', 'normalizacion', 'validacion', 'analisis', 'reporte']
```

La declaración no ejecuta el pipeline. Sirve como mapa. En una versión más avanzada, ese mapa puede convertirse en documentación, validación o registro automático.

## Cargar datos crudos

La carga no debe interpretar demasiado. Su tarea principal es convertir la fuente externa en filas legibles.

```python
import csv
import io


def cargar_csv(texto):
    return list(csv.DictReader(io.StringIO(texto)))


texto_csv = """paciente_id,servicio,valor,unidad,estado
P001,urgencias,1.2,mg/dL,medido
P002,urgencias,1.5,mg/dL,medido
"""

filas = cargar_csv(texto_csv)

print(filas[0])
print(len(filas))
```

Salida esperada:

```text
{'paciente_id': 'P001', 'servicio': 'urgencias', 'valor': '1.2', 'unidad': 'mg/dL', 'estado': 'medido'}
2
```

En este punto `valor` sigue siendo texto. Eso no es un error. Es una señal de que la carga todavía no debe fingir que interpretó el dato.

## Normalizar sin decidir de más

La normalización prepara valores para validación. No debería convertir una fila inválida en válida por fuerza.

```python
def normalizar_fila(fila):
    normalizada = dict(fila)
    normalizada["servicio"] = normalizada["servicio"].strip().lower()
    normalizada["unidad"] = normalizada["unidad"].strip()
    normalizada["estado"] = normalizada["estado"].strip().lower()

    valor_crudo = normalizada["valor"].strip()
    if valor_crudo:
        normalizada["valor"] = float(valor_crudo.replace(",", "."))
    else:
        normalizada["valor"] = None

    return normalizada


fila = {"paciente_id": "P001", "servicio": " Urgencias ", "valor": "1,2", "unidad": "mg/dL", "estado": " Medido "}

print(normalizar_fila(fila))
```

Salida esperada:

```text
{'paciente_id': 'P001', 'servicio': 'urgencias', 'valor': 1.2, 'unidad': 'mg/dL', 'estado': 'medido'}
```

La normalización corrige espacios, mayúsculas y formato decimal. No decide si el paciente existe, si la unidad es aceptable o si la medición debe entrar al análisis.

## Validar y separar rechazos

La validación produce dos conjuntos: filas válidas y filas rechazadas con razones.

```python
ESTADOS_PERMITIDOS = {"medido", "pendiente", "no_medido"}
UNIDADES_PERMITIDAS = {"mg/dL"}


def validar_fila(fila):
    errores = []

    for campo in ["paciente_id", "servicio", "unidad", "estado"]:
        if not fila.get(campo):
            errores.append({"campo": campo, "razon": "campo_obligatorio_vacio"})

    if fila.get("estado") not in ESTADOS_PERMITIDOS:
        errores.append({"campo": "estado", "razon": "estado_no_permitido"})

    if fila.get("unidad") not in UNIDADES_PERMITIDAS:
        errores.append({"campo": "unidad", "razon": "unidad_no_soportada"})

    if fila.get("estado") == "medido" and fila.get("valor") is None:
        errores.append({"campo": "valor", "razon": "valor_requerido_para_estado_medido"})

    return {"valida": not errores, "fila": fila, "errores": errores}


evaluacion = validar_fila(
    {"paciente_id": "P005", "servicio": "urgencias", "valor": 100.0, "unidad": "umol/L", "estado": "medido"}
)

print(evaluacion)
```

Salida esperada:

```text
{'valida': False, 'fila': {'paciente_id': 'P005', 'servicio': 'urgencias', 'valor': 100.0, 'unidad': 'umol/L', 'estado': 'medido'}, 'errores': [{'campo': 'unidad', 'razon': 'unidad_no_soportada'}]}
```

La fila no desaparece. Queda rechazada con razón. Esa diferencia es central para trazabilidad.

## Miniatura completa

Ahora juntamos las etapas en un pipeline mínimo. Sigue siendo pequeño, pero ya conserva artefactos y conteos.

```python
import csv
import io

ESTADOS_PERMITIDOS = {"medido", "pendiente", "no_medido"}
UNIDADES_PERMITIDAS = {"mg/dL"}


def cargar_csv(texto):
    return list(csv.DictReader(io.StringIO(texto)))


def normalizar_fila(fila):
    normalizada = dict(fila)
    normalizada["servicio"] = normalizada["servicio"].strip().lower()
    normalizada["unidad"] = normalizada["unidad"].strip()
    normalizada["estado"] = normalizada["estado"].strip().lower()

    valor_crudo = normalizada["valor"].strip()
    normalizada["valor"] = float(valor_crudo.replace(",", ".")) if valor_crudo else None
    return normalizada


def validar_fila(fila):
    errores = []
    for campo in ["paciente_id", "servicio", "unidad", "estado"]:
        if not fila.get(campo):
            errores.append({"campo": campo, "razon": "campo_obligatorio_vacio"})

    if fila.get("estado") not in ESTADOS_PERMITIDOS:
        errores.append({"campo": "estado", "razon": "estado_no_permitido"})
    if fila.get("unidad") not in UNIDADES_PERMITIDAS:
        errores.append({"campo": "unidad", "razon": "unidad_no_soportada"})
    if fila.get("estado") == "medido" and fila.get("valor") is None:
        errores.append({"campo": "valor", "razon": "valor_requerido_para_estado_medido"})

    return {"valida": not errores, "fila": fila, "errores": errores}


def separar_evaluaciones(evaluaciones):
    validas = [evaluacion["fila"] for evaluacion in evaluaciones if evaluacion["valida"]]
    rechazadas = [evaluacion for evaluacion in evaluaciones if not evaluacion["valida"]]
    return validas, rechazadas


def analizar_creatinina(validas, servicio):
    valores = [
        fila["valor"]
        for fila in validas
        if fila["servicio"] == servicio and fila["estado"] == "medido" and fila["unidad"] == "mg/dL"
    ]
    denominador = len(valores)
    promedio = round(sum(valores) / denominador, 2) if denominador else None
    return {
        "servicio": servicio,
        "promedio_creatinina": promedio,
        "unidad": "mg/dL",
        "denominador_medidos": denominador,
        "filas_validas": len(validas),
    }


def construir_reporte(resultado, rechazadas):
    return {
        "estado": "completo",
        "resultado": resultado,
        "filas_rechazadas": len(rechazadas),
    }


def ejecutar_pipeline(texto_csv, parametros):
    crudas = cargar_csv(texto_csv)
    normalizadas = [normalizar_fila(fila) for fila in crudas]
    evaluaciones = [validar_fila(fila) for fila in normalizadas]
    validas, rechazadas = separar_evaluaciones(evaluaciones)
    resultado = analizar_creatinina(validas, parametros["servicio"])
    reporte = construir_reporte(resultado, rechazadas)

    return {
        "artefactos": {
            "filas_crudas": crudas,
            "filas_normalizadas": normalizadas,
            "filas_validas": validas,
            "filas_rechazadas": rechazadas,
            "resultado_derivado": resultado,
            "reporte_final": reporte,
        },
        "conteos": {
            "crudas": len(crudas),
            "validas": len(validas),
            "rechazadas": len(rechazadas),
        },
    }


texto_csv = """paciente_id,servicio,valor,unidad,estado
P001,urgencias,1.2,mg/dL,medido
P002,urgencias,1.5,mg/dL,medido
P003,urgencias,,mg/dL,pendiente
P004,consulta,0.9,mg/dL,medido
P005,urgencias,100,umol/L,medido
"""

ejecucion = ejecutar_pipeline(texto_csv, {"servicio": "urgencias"})

print(ejecucion["conteos"])
print(ejecucion["artefactos"]["reporte_final"])
```

Salida esperada:

```text
{'crudas': 5, 'validas': 4, 'rechazadas': 1}
{'estado': 'completo', 'resultado': {'servicio': 'urgencias', 'promedio_creatinina': 1.35, 'unidad': 'mg/dL', 'denominador_medidos': 2, 'filas_validas': 4}, 'filas_rechazadas': 1}
```

El pipeline no oculta el rechazo por unidad. Tampoco incluye la fila pendiente en el denominador de mediciones. La salida final conserva resultado y conteo de rechazo.

## Artefactos intermedios

Una ventaja del pipeline es que permite inspeccionar dónde ocurrió un problema.

```python
print(sorted(ejecucion["artefactos"]))
print(ejecucion["artefactos"]["filas_rechazadas"][0]["errores"])
```

Salida esperada:

```text
['filas_crudas', 'filas_normalizadas', 'filas_rechazadas', 'filas_validas', 'reporte_final', 'resultado_derivado']
[{'campo': 'unidad', 'razon': 'unidad_no_soportada'}]
```

No siempre hay que guardar todos los artefactos para siempre. Pero durante aprendizaje, auditoría o depuración, conservarlos ayuda a entender el flujo.

## CODE CLEAN: etapas pequeñas, nombres del dominio

La versión frágil usa una función total que hace todo.

```python
def procesar(texto):
    # Carga, limpia, valida, calcula e imprime en una sola pieza.
    pass
```

La versión más limpia usa nombres que expresan etapas del dominio.

```python
def bosquejar_pipeline_limpio(texto_csv, parametros):
    crudas = cargar_csv(texto_csv)
    normalizadas = normalizar_lote(crudas)
    validas, rechazadas = validar_lote(normalizadas)
    resultado = analizar_creatinina(validas, parametros["servicio"])
    return construir_reporte(resultado, rechazadas)
```

La segunda versión no es mejor por tener más líneas. Es mejor porque permite probar y reemplazar cada etapa sin reescribir el flujo entero.

## Prueba mínima

Una prueba mínima del pipeline debe verificar resultado, denominador y rechazo.

```python
ejecucion = ejecutar_pipeline(texto_csv, {"servicio": "urgencias"})
reporte = ejecucion["artefactos"]["reporte_final"]

assert ejecucion["conteos"]["crudas"] == 5
assert ejecucion["conteos"]["validas"] == 4
assert ejecucion["conteos"]["rechazadas"] == 1
assert reporte["resultado"]["promedio_creatinina"] == 1.35
assert reporte["resultado"]["denominador_medidos"] == 2
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Esta prueba no garantiza que el pipeline sea clínicamente válido. Garantiza algo más modesto y necesario: que el flujo conserva las propiedades mínimas que dice conservar.

## Errores frecuentes

**Llamar pipeline a un script monolítico.** Si no hay etapas distinguibles, la palabra pipeline solo maquilla un bloque difícil de auditar.

**Borrar rechazos.** Una fila inválida debe quedar fuera del cálculo, pero no fuera de la explicación.

**Normalizar como si fuera validar.** Pasar texto a minúsculas no prueba que el dato tenga sentido biomédico.

**Analizar antes de validar.** Un promedio calculado sobre datos no interpretados puede ser numéricamente correcto y conceptualmente inválido.

**No guardar conteos por etapa.** Sin conteos, no se sabe si el pipeline perdió filas de forma silenciosa.

**Confundir pipeline con herramienta.** La arquitectura conceptual existe antes de Airflow, Prefect, Make, cron o cualquier orquestador.

## Argumentos críticos

### Desacuerdo 1: función simple contra pipeline

Pregunta: ¿cuándo una función simple debe convertirse en pipeline?

Cuando el proceso tiene etapas con fallos distintos, artefactos intermedios relevantes o consecuencias que exigen auditoría. Si cargar, validar, analizar y reportar pueden fallar por razones diferentes, conviene separarlos.

Consenso operativo: usar pipeline cuando la trazabilidad entre etapas sea parte del valor del resultado.

### Desacuerdo 2: guardar todo contra guardar lo mínimo

Pregunta: ¿deben conservarse todos los artefactos intermedios?

No siempre. En producción pueden existir límites de privacidad, almacenamiento y seguridad. Pero el diseño debe saber qué artefactos existen y decidir cuáles se guardan, cuáles se resumen y cuáles se descartan.

Consenso operativo: durante diseño y auditoría, conservar más; en operación real, conservar lo necesario con criterios explícitos.

### Desacuerdo 3: pipeline local contra sistema real

Pregunta: ¿por qué no usar desde ya un orquestador profesional?

Porque un orquestador ejecuta etapas; no decide qué significa una fila válida, un rechazo, un denominador o una salida responsable. Aprender la estructura mínima evita depender de una herramienta para pensar.

Consenso operativo: primero entender el flujo; después elegir infraestructura proporcional.

## Puente hacia sistemas reales

Los pipelines mínimos preparan el terreno para procesos biomédicos más exigentes: carga programada de archivos, validación de lotes, análisis reproducibles, reportes automáticos, tableros, modelos predictivos y vigilancia de calidad de datos.

En bioinformática, un pipeline puede encadenar limpieza de secuencias, alineamiento, llamado de variantes y anotación. En datos clínicos, puede encadenar extracción, normalización, validación, análisis y reporte. En modelos predictivos, puede encadenar entrenamiento, evaluación, calibración y monitoreo.

La forma técnica cambia. La pregunta permanece: ¿qué entra, qué sale, qué se rechaza, qué se calcula y cómo se reconstruye?

## Preguntas de comprensión profunda

1. ¿Qué diferencia hay entre un script largo y un pipeline mínimo?
2. ¿Por qué la carga no debería interpretar todos los datos desde el inicio?
3. ¿Qué riesgo aparece si se normaliza sin validar?
4. ¿Por qué una fila rechazada debe conservar razón?
5. ¿Qué artefactos intermedios conviene inspeccionar durante depuración?
6. ¿Por qué el denominador sigue siendo importante dentro de un pipeline?
7. ¿Cuándo tendría sentido guardar menos artefactos intermedios?
8. ¿Qué parte del ejemplo cambiaría al pasar de CSV local a base de datos?

## Vacíos de comprensión que debes vigilar

1. Pensar que un pipeline es una herramienta específica. La herramienta puede ayudar, pero el pipeline es primero una arquitectura de etapas.
2. Creer que separar funciones es siempre mejor. La separación vale cuando expresa responsabilidades reales.
3. Olvidar que los rechazos también son salida del sistema. Sin ellos, el reporte final pierde contexto.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** dibuja un pipeline de cinco etapas para un archivo CSV biomédico: carga, normalización, validación, análisis y reporte.
2. **Segunda hora:** implementa cada etapa como función pequeña y conserva conteos.
3. **Tercera hora:** introduce una fila inválida y verifica que el pipeline la rechace sin ocultarla.

## Bibliografía y fuentes

- Python Software Foundation. [csv: CSV File Reading and Writing](https://docs.python.org/3/library/csv.html).
- Python Software Foundation. [io: Core tools for working with streams](https://docs.python.org/3/library/io.html).
- Kleppmann, M. (2017). *Designing Data-Intensive Applications*. O'Reilly Media.
- Sandve, G. K., Nekrutenko, A., Taylor, J., & Hovig, E. (2013). Ten simple rules for reproducible computational research. *PLoS Computational Biology*, 9(10), e1003285.

## Siguiente paso

Un pipeline mínimo encadena etapas y conserva artefactos. La siguiente sección puede avanzar hacia validación por lotes y reportes de calidad: cómo resumir errores, advertencias y métricas de completitud cuando llegan muchas filas a la vez.
