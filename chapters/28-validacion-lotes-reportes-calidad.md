# Validación por lotes y reportes de calidad

La sección anterior construyó un pipeline mínimo: cargar, normalizar, validar, analizar y reportar. Esa secuencia permite seguir el camino de una fila desde el archivo crudo hasta el resultado final.

Pero un pipeline que procesa una fila no enfrenta todavía el problema operativo real. En medicina y ciencias de la vida casi nunca llega una sola observación. Llegan lotes: archivos de laboratorio, tablas de pacientes, registros de seguimiento, exportaciones de historia clínica, formularios, cargas regulatorias o mediciones acumuladas durante semanas.

Un lote puede tener filas válidas, errores graves, advertencias leves, datos faltantes, unidades incompatibles, duplicados, valores fuera de rango y campos incompletos. Si el sistema solo dice "falló" o "pasó", empobrece la decisión. Lo que se necesita es un reporte de calidad.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
La validación por lotes es la evaluación sistemática de un conjunto de registros contra reglas explícitas, produciendo conteos, errores, advertencias, métricas de completitud y una decisión operativa sobre si el lote puede usarse, debe corregirse o debe rechazarse.
</div>

La idea central es sencilla: validar un lote no significa buscar una fila perfecta. Significa describir el estado del conjunto con suficiente detalle para decidir qué hacer después.

Un lote biomédico puede ser parcialmente útil. Algunas filas pueden entrar al análisis. Otras deben quedar rechazadas. Otras pueden entrar con advertencia. Lo importante es que esa separación no ocurra de forma silenciosa.

## Versión ingenua: detenerse en el primer error

Una forma frágil de validar un lote es cortar apenas aparece el primer problema.

```python
filas = [
    {"paciente_id": "P001", "valor": "1.2", "unidad": "mg/dL"},
    {"paciente_id": "P002", "valor": "", "unidad": "mg/dL"},
    {"paciente_id": "P003", "valor": "100", "unidad": "umol/L"},
]

try:
    for fila in filas:
        if not fila["valor"]:
            raise ValueError("valor faltante")
    print("lote valido")
except ValueError as error:
    print(f"{type(error).__name__}: {error}")
```

Salida esperada:

```text
ValueError: valor faltante
```

El error es real, pero la información es pobre. No sabemos cuántas filas estaban bien, cuántas tenían unidad incorrecta, si el problema fue frecuente ni si el lote podía corregirse parcialmente.

En un proceso biomédico, detenerse en el primer error puede ser útil para depurar código. Para evaluar calidad de datos, suele ser insuficiente.

## Crítica técnica: el lote necesita diagnóstico

Un buen reporte de calidad debe responder:

**Cuántas filas llegaron.** Sin denominador no hay proporción.

**Cuántas filas fueron aceptadas.** El lote puede ser usable parcialmente.

**Cuántas filas fueron rechazadas.** Los rechazos deben tener razón.

**Qué advertencias aparecieron.** No todo problema invalida la fila.

**Qué campos están incompletos.** La completitud permite comparar fuentes, periodos y equipos.

**Qué severidad domina.** Un lote con errores críticos se maneja distinto a uno con advertencias leves.

La validación por lotes convierte errores dispersos en una señal de calidad.

## Reglas con severidad

Primero definimos reglas mínimas. Algunas producen errores; otras producen advertencias.

```python
REGLAS_LOTE = {
    "campos_requeridos": ["paciente_id", "valor", "unidad", "estado"],
    "unidades_permitidas": {"mg/dL"},
    "estados_permitidos": {"medido", "pendiente", "no_medido"},
    "valor_maximo_alerta": 15.0,
}

print(REGLAS_LOTE["campos_requeridos"])
print(sorted(REGLAS_LOTE["unidades_permitidas"]))
```

Salida esperada:

```text
['paciente_id', 'valor', 'unidad', 'estado']
['mg/dL']
```

La severidad importa. Una unidad no soportada puede impedir el cálculo. Un valor alto puede ser una advertencia pedagógica en esta miniatura, no una decisión clínica.

## Validar una fila con errores y advertencias

La validación de fila debe devolver una estructura, no solo `True` o `False`.

```python
def validar_fila_lote(fila, reglas):
    errores = []
    advertencias = []

    for campo in reglas["campos_requeridos"]:
        if campo not in fila or fila[campo] in ("", None):
            errores.append({"campo": campo, "razon": "campo_requerido_ausente"})

    if errores:
        return {"valida": False, "errores": errores, "advertencias": advertencias}

    if fila["unidad"] not in reglas["unidades_permitidas"]:
        errores.append({"campo": "unidad", "razon": "unidad_no_soportada"})

    if fila["estado"] not in reglas["estados_permitidos"]:
        errores.append({"campo": "estado", "razon": "estado_no_permitido"})

    valor_normalizado = None
    if fila["estado"] == "medido":
        try:
            valor_normalizado = float(str(fila["valor"]).replace(",", "."))
        except ValueError:
            errores.append({"campo": "valor", "razon": "valor_no_numerico"})

    if not errores and valor_normalizado is not None and valor_normalizado > reglas["valor_maximo_alerta"]:
        advertencias.append({"campo": "valor", "razon": "valor_extremo_revisar"})

    return {
        "valida": not errores,
        "errores": errores,
        "advertencias": advertencias,
        "valor_normalizado": valor_normalizado,
    }


fila = {"paciente_id": "P001", "valor": "16.2", "unidad": "mg/dL", "estado": "medido"}

print(validar_fila_lote(fila, REGLAS_LOTE))
```

Salida esperada:

```text
{'valida': True, 'errores': [], 'advertencias': [{'campo': 'valor', 'razon': 'valor_extremo_revisar'}], 'valor_normalizado': 16.2}
```

La fila queda válida, pero marcada. Eso permite análisis posterior sin perder señal de revisión.

## Validar el lote completo

Ahora recorremos todas las filas y conservamos posición, fila original y evaluación.

```python
def validar_lote(filas, reglas):
    evaluaciones = []

    for posicion, fila in enumerate(filas, start=1):
        evaluacion = validar_fila_lote(fila, reglas)
        evaluaciones.append(
            {
                "posicion": posicion,
                "fila": fila,
                "valida": evaluacion["valida"],
                "errores": evaluacion["errores"],
                "advertencias": evaluacion["advertencias"],
                "valor_normalizado": evaluacion.get("valor_normalizado"),
            }
        )

    return evaluaciones


filas = [
    {"paciente_id": "P001", "valor": "1.2", "unidad": "mg/dL", "estado": "medido"},
    {"paciente_id": "P002", "valor": "", "unidad": "mg/dL", "estado": "medido"},
    {"paciente_id": "P003", "valor": "100", "unidad": "umol/L", "estado": "medido"},
    {"paciente_id": "P004", "valor": "16.2", "unidad": "mg/dL", "estado": "medido"},
]

evaluaciones = validar_lote(filas, REGLAS_LOTE)

print([evaluacion["valida"] for evaluacion in evaluaciones])
```

Salida esperada:

```text
[True, False, False, True]
```

El lote ya no se reduce a una excepción. Cada fila conserva evaluación propia.

## Contar razones

Un reporte útil agrupa razones. Para eso, `collections.Counter` evita escribir acumuladores manuales frágiles.

```python
from collections import Counter


def contar_razones(evaluaciones, tipo):
    razones = []
    for evaluacion in evaluaciones:
        for hallazgo in evaluacion[tipo]:
            razones.append(hallazgo["razon"])
    return dict(Counter(razones))


print(contar_razones(evaluaciones, "errores"))
print(contar_razones(evaluaciones, "advertencias"))
```

Salida esperada:

```text
{'campo_requerido_ausente': 1, 'unidad_no_soportada': 1}
{'valor_extremo_revisar': 1}
```

El conteo de razones permite saber si estamos ante un error aislado o un patrón del lote.

## Métricas de completitud

La completitud por campo ayuda a comparar calidad entre fuentes.

```python
def calcular_completitud(filas, campos):
    total = len(filas)
    completitud = {}

    for campo in campos:
        presentes = sum(1 for fila in filas if fila.get(campo) not in ("", None))
        completitud[campo] = round(presentes / total, 2) if total else None

    return completitud


print(calcular_completitud(filas, ["paciente_id", "valor", "unidad", "estado"]))
```

Salida esperada:

```text
{'paciente_id': 1.0, 'valor': 0.75, 'unidad': 1.0, 'estado': 1.0}
```

La completitud no dice si el dato es correcto. Dice si está presente. Esa distinción evita una confusión frecuente: una tabla completa puede estar mal codificada.

## Construir el reporte de calidad

El reporte debe condensar el lote sin borrar detalle.

```python
def construir_reporte_calidad(filas, evaluaciones, campos):
    total = len(evaluaciones)
    validas = sum(1 for evaluacion in evaluaciones if evaluacion["valida"])
    rechazadas = total - validas
    con_advertencias = sum(1 for evaluacion in evaluaciones if evaluacion["advertencias"])

    estado = "aceptable"
    if rechazadas:
        estado = "requiere_correccion"
    if validas == 0:
        estado = "rechazado"

    return {
        "estado_lote": estado,
        "total_filas": total,
        "filas_validas": validas,
        "filas_rechazadas": rechazadas,
        "filas_con_advertencias": con_advertencias,
        "proporcion_valida": round(validas / total, 2) if total else None,
        "errores_por_razon": contar_razones(evaluaciones, "errores"),
        "advertencias_por_razon": contar_razones(evaluaciones, "advertencias"),
        "completitud": calcular_completitud(filas, campos),
    }


reporte = construir_reporte_calidad(
    filas,
    evaluaciones,
    ["paciente_id", "valor", "unidad", "estado"],
)

print(reporte)
```

Salida esperada:

```text
{'estado_lote': 'requiere_correccion', 'total_filas': 4, 'filas_validas': 2, 'filas_rechazadas': 2, 'filas_con_advertencias': 1, 'proporcion_valida': 0.5, 'errores_por_razon': {'campo_requerido_ausente': 1, 'unidad_no_soportada': 1}, 'advertencias_por_razon': {'valor_extremo_revisar': 1}, 'completitud': {'paciente_id': 1.0, 'valor': 0.75, 'unidad': 1.0, 'estado': 1.0}}
```

Este reporte ya permite una decisión operativa: el lote no debe usarse sin corrección completa, pero contiene filas válidas que pueden inspeccionarse por separado.

## Miniatura completa

La versión completa junta validación, conteos y reporte.

```python
from collections import Counter

REGLAS_LOTE = {
    "campos_requeridos": ["paciente_id", "valor", "unidad", "estado"],
    "unidades_permitidas": {"mg/dL"},
    "estados_permitidos": {"medido", "pendiente", "no_medido"},
    "valor_maximo_alerta": 15.0,
}


def validar_fila_lote(fila, reglas):
    errores = []
    advertencias = []

    for campo in reglas["campos_requeridos"]:
        if campo not in fila or fila[campo] in ("", None):
            errores.append({"campo": campo, "razon": "campo_requerido_ausente"})

    if errores:
        return {"valida": False, "errores": errores, "advertencias": advertencias}

    if fila["unidad"] not in reglas["unidades_permitidas"]:
        errores.append({"campo": "unidad", "razon": "unidad_no_soportada"})
    if fila["estado"] not in reglas["estados_permitidos"]:
        errores.append({"campo": "estado", "razon": "estado_no_permitido"})

    valor_normalizado = None
    if fila["estado"] == "medido":
        try:
            valor_normalizado = float(str(fila["valor"]).replace(",", "."))
        except ValueError:
            errores.append({"campo": "valor", "razon": "valor_no_numerico"})

    if not errores and valor_normalizado is not None and valor_normalizado > reglas["valor_maximo_alerta"]:
        advertencias.append({"campo": "valor", "razon": "valor_extremo_revisar"})

    return {
        "valida": not errores,
        "errores": errores,
        "advertencias": advertencias,
        "valor_normalizado": valor_normalizado,
    }


def validar_lote(filas, reglas):
    evaluaciones = []
    for posicion, fila in enumerate(filas, start=1):
        evaluacion = validar_fila_lote(fila, reglas)
        evaluaciones.append({"posicion": posicion, "fila": fila, **evaluacion})
    return evaluaciones


def contar_razones(evaluaciones, tipo):
    return dict(
        Counter(
            hallazgo["razon"]
            for evaluacion in evaluaciones
            for hallazgo in evaluacion[tipo]
        )
    )


def calcular_completitud(filas, campos):
    total = len(filas)
    return {
        campo: round(
            sum(1 for fila in filas if fila.get(campo) not in ("", None)) / total,
            2,
        )
        if total
        else None
        for campo in campos
    }


def construir_reporte_calidad(filas, evaluaciones, campos):
    total = len(evaluaciones)
    validas = sum(1 for evaluacion in evaluaciones if evaluacion["valida"])
    rechazadas = total - validas
    con_advertencias = sum(1 for evaluacion in evaluaciones if evaluacion["advertencias"])

    estado = "aceptable"
    if rechazadas:
        estado = "requiere_correccion"
    if validas == 0:
        estado = "rechazado"

    return {
        "estado_lote": estado,
        "total_filas": total,
        "filas_validas": validas,
        "filas_rechazadas": rechazadas,
        "filas_con_advertencias": con_advertencias,
        "proporcion_valida": round(validas / total, 2) if total else None,
        "errores_por_razon": contar_razones(evaluaciones, "errores"),
        "advertencias_por_razon": contar_razones(evaluaciones, "advertencias"),
        "completitud": calcular_completitud(filas, campos),
    }


filas = [
    {"paciente_id": "P001", "valor": "1.2", "unidad": "mg/dL", "estado": "medido"},
    {"paciente_id": "P002", "valor": "", "unidad": "mg/dL", "estado": "medido"},
    {"paciente_id": "P003", "valor": "100", "unidad": "umol/L", "estado": "medido"},
    {"paciente_id": "P004", "valor": "16.2", "unidad": "mg/dL", "estado": "medido"},
]

evaluaciones = validar_lote(filas, REGLAS_LOTE)
reporte = construir_reporte_calidad(
    filas,
    evaluaciones,
    REGLAS_LOTE["campos_requeridos"],
)

print(reporte["estado_lote"])
print(reporte["filas_validas"], reporte["filas_rechazadas"])
print(reporte["errores_por_razon"])
```

Salida esperada:

```text
requiere_correccion
2 2
{'campo_requerido_ausente': 1, 'unidad_no_soportada': 1}
```

La salida no intenta reemplazar el juicio humano. Hace algo más básico: impide que el lote entre al análisis como si todos sus registros tuvieran la misma calidad.

## CODE CLEAN: de banderas sueltas a reporte estructurado

La versión frágil acumula banderas sin estructura.

```python
errores = 0
for fila in filas:
    if fila["valor"] == "":
        errores += 1

print(errores)
```

Salida esperada:

```text
1
```

La versión más limpia separa evaluación y reporte.

```python
evaluaciones = validar_lote(filas, REGLAS_LOTE)
reporte = construir_reporte_calidad(
    filas,
    evaluaciones,
    REGLAS_LOTE["campos_requeridos"],
)

print(reporte["estado_lote"])
```

Salida esperada:

```text
requiere_correccion
```

El cambio no es cosmético. El reporte conserva severidad, razón, proporción y completitud. Eso permite auditar el lote sin volver a leer manualmente cada fila.

## Prueba mínima

La prueba debe cubrir conteos, estado del lote y una razón específica.

```python
evaluaciones = validar_lote(filas, REGLAS_LOTE)
reporte = construir_reporte_calidad(
    filas,
    evaluaciones,
    REGLAS_LOTE["campos_requeridos"],
)

assert reporte["estado_lote"] == "requiere_correccion"
assert reporte["total_filas"] == 4
assert reporte["filas_validas"] == 2
assert reporte["filas_rechazadas"] == 2
assert reporte["errores_por_razon"]["unidad_no_soportada"] == 1
assert reporte["completitud"]["valor"] == 0.75
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Una prueba así no demuestra que las reglas sean suficientes para uso clínico. Demuestra que el reporte cumple el contrato mínimo que declara.

## Errores frecuentes

**Validar solo una fila ejemplar.** Un lote puede fallar por patrones que no aparecen en el caso feliz.

**Tratar toda anomalía como error crítico.** Una advertencia puede requerir revisión sin invalidar la fila.

**No conservar la posición de la fila.** Sin posición, corregir el archivo original se vuelve más difícil.

**No contar razones.** Saber que hay diez errores no basta; importa saber si son diez causas distintas o una causa repetida.

**Confundir completitud con calidad.** Un campo presente puede estar mal codificado, fuera de unidad o fuera de contexto.

**Aceptar parcialmente sin declarar criterio.** Si algunas filas entran y otras no, el reporte debe decirlo con denominadores.

## Argumentos críticos

### Desacuerdo 1: rechazar todo el lote contra aceptar parcialmente

Pregunta: ¿si hay errores, debe rechazarse todo el lote?

Depende del uso. Para una carga regulatoria estricta puede rechazarse el lote completo. Para exploración o depuración puede aceptarse una parte y devolver errores. Lo inaceptable es mezclar ambas políticas sin declararlo.

Consenso operativo: la política de aceptación debe estar escrita antes de mirar el resultado.

### Desacuerdo 2: advertencias contra errores

Pregunta: ¿por qué no convertir toda advertencia en error?

Porque algunas señales exigen revisión, no exclusión automática. En datos biomédicos, un valor extremo puede ser error de captura, evento clínico real o unidad equivocada. La severidad debe permitir distinguir esas posibilidades.

Consenso operativo: error impide uso automático; advertencia conserva la fila, pero exige atención.

### Desacuerdo 3: reporte compacto contra detalle completo

Pregunta: ¿es suficiente un resumen?

No siempre. El resumen permite decidir rápido; el detalle permite corregir. Un buen sistema conserva ambos niveles: reporte agregado y lista trazable de hallazgos por fila.

Consenso operativo: reporte para decisión, detalle para corrección.

## Puente hacia sistemas reales

Los reportes de calidad son una pieza central antes de automatizar análisis más serios. Permiten comparar fuentes, detectar degradación de datos, negociar correcciones con equipos humanos y decidir si una carga puede avanzar al siguiente paso del pipeline.

Más adelante, la misma idea crecerá hacia tableros de calidad, pruebas de datos, contratos entre sistemas, validación continua, monitoreo de deriva y auditoría de modelos. La escala cambia, pero la pregunta sigue siendo la misma:

```text
Qué tan confiable es este lote para la operación que quiero ejecutar?
```

## Preguntas de comprensión profunda

1. ¿Por qué detenerse en el primer error puede ser insuficiente para calidad de datos?
2. ¿Qué diferencia hay entre error y advertencia?
3. ¿Por qué la completitud no equivale a calidad?
4. ¿Qué información debe conservarse para corregir una fila rechazada?
5. ¿Cuándo tendría sentido rechazar todo un lote?
6. ¿Cuándo tendría sentido aceptar parcialmente un lote?
7. ¿Por qué los conteos por razón son más útiles que un conteo total de errores?
8. ¿Cómo se conecta un reporte de calidad con un pipeline reproducible?

## Vacíos de comprensión que debes vigilar

1. Pensar que la validación por lotes es solo ejecutar la validación de fila muchas veces. El valor adicional está en el resumen agregado y la decisión operativa.
2. Confundir presencia con validez. Un campo lleno puede ser incorrecto.
3. Omitir severidad. No todo hallazgo debe tener la misma consecuencia.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** define reglas de error y advertencia para un lote de mediciones.
2. **Segunda hora:** valida diez filas inventadas y agrupa razones con `Counter`.
3. **Tercera hora:** diseña un reporte que permita decidir aceptar, corregir o rechazar el lote.

## Bibliografía y fuentes

- Great Expectations. (s. f.). *Data quality concepts*. <https://docs.greatexpectations.io/>
- Olson, J. E. (2003). *Data quality: The accuracy dimension*. Morgan Kaufmann.
- Python Software Foundation. (s. f.). *collections: Container datatypes*. <https://docs.python.org/3/library/collections.html>.
- Redman, T. C. (1996). *Data quality for the information age*. Artech House.
- Wang, R. Y., & Strong, D. M. (1996). Beyond accuracy: What data quality means to data consumers. *Journal of Management Information Systems, 12*(4), 5-33.

## Siguiente paso

Un reporte de calidad convierte un lote en evidencia operativa: cuántas filas llegaron, cuáles se aceptaron, cuáles se rechazaron y por qué. La siguiente sección puede avanzar hacia exportación, auditoría y artefactos compartibles: cómo producir salidas estables que otros procesos o personas puedan revisar sin depender de la sesión actual.
