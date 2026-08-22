# Cierre integrador: del dato biomédico al flujo verificable

El capítulo comenzó con una advertencia: un número no es todavía una medición biomédica. Necesita unidad, contexto, estado, procedencia y una regla que permita decidir si puede usarse.

Desde allí avanzamos por texto y códigos, estados e incertidumbre, tiempo, ausencia, colecciones, tablas, esquemas, `pandas`, persistencia, relaciones, consultas, APIs, análisis, pipelines, reportes de calidad y artefactos auditables.

Cada pieza resolvió un problema distinto. El cierre exige algo más difícil: conectarlas sin perder el significado que cada una protege.

Un flujo verificable no es una secuencia larga de funciones. Es una cadena de transformaciones en la que cada etapa declara qué recibe, qué produce, qué rechaza, qué conserva y qué no puede demostrar.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un flujo biomédico verificable es un proceso compuesto por etapas con contratos explícitos, artefactos identificables y controles que permiten reconstruir cómo un dato crudo llegó a convertirse en un resultado. Debe conservar tanto los valores aceptados como las razones por las que otros valores no pudieron avanzar.
</div>

La verificabilidad no garantiza verdad clínica. Permite examinar el trayecto técnico y semántico de una conclusión.

## La versión ingenua: llegar rápido a un promedio

Supongamos que recibimos valores de proteína C reactiva en dos unidades distintas. Una solución rápida podría convertir texto a números, eliminar lo incompleto y calcular una media.

```python
filas_crudas = [
    {"paciente_id": " P001 ", "valor": "1.2", "unidad": "mg/dL"},
    {"paciente_id": "P002", "valor": "", "unidad": "mg/L"},
    {"paciente_id": "P003", "valor": "5", "unidad": "desconocida"},
    {"paciente_id": "P004", "valor": "16.2", "unidad": "mg/L"},
]

valores = [
    float(fila["valor"])
    for fila in filas_crudas
    if fila["valor"]
]

print(sum(valores) / len(valores))
```

Salida esperada:

```text
7.466666666666666
```

El programa produce un número. Sin embargo, el resultado mezcla `mg/dL` con `mg/L`, ignora por qué faltó un valor, conserva una unidad desconocida y pierde el denominador original.

La ejecución terminó. El problema no quedó resuelto.

## Crítica técnica: el resultado ocultó el trayecto

La versión ingenua falla en varios niveles:

- **representación:** trata cadenas como si ya fueran mediciones;
- **normalización:** mezcla unidades sin conversión;
- **validación:** elimina datos sin registrar razones;
- **análisis:** usa como denominador solo lo que pudo convertir;
- **trazabilidad:** no conserva versión de regla ni identificador de lote;
- **auditoría:** no produce un artefacto que otra persona pueda revisar;
- **responsabilidad:** ofrece un valor numérico con apariencia de precisión mayor que la evidencia disponible.

El error central no está en la fórmula de la media. Está en permitir que el cálculo reciba datos que todavía no tienen un contrato defendible.

## El mapa completo

El patrón construido durante el capítulo puede resumirse así:

```text
dato crudo
  -> adquisición
  -> normalización
  -> validación
  -> aceptados + rechazos
  -> persistencia o tabla de trabajo
  -> análisis reproducible
  -> reporte de calidad
  -> exportación
  -> auditoría
```

Cada flecha representa una decisión. Si esas decisiones se ocultan dentro de una sola función, el pipeline puede ejecutarse, pero resulta difícil de explicar, probar y corregir.

## Contratos por etapa

| Etapa | Entrada | Salida | Pregunta que debe responder |
|---|---|---|---|
| Adquisición | archivo, API o registros crudos | copia de trabajo | ¿qué se recibió y desde dónde? |
| Normalización | valores crudos | forma comparable | ¿qué cambios de forma se aplicaron? |
| Validación | forma normalizada + esquema | aceptado o rechazo | ¿cumple el contrato mínimo? |
| Análisis | registros aceptados + parámetros | resultado derivado | ¿cuál fue el denominador y la regla? |
| Calidad | aceptados + rechazos | métricas y estado del lote | ¿el lote puede avanzar? |
| Exportación | artefactos cerrados | paquete + manifiesto | ¿qué se comparte y con qué versión? |
| Auditoría | paquete + referencia | hallazgos | ¿el contenido conserva su integridad? |

El contrato evita que una etapa compense silenciosamente los errores de otra.

## Una medición mínima y explícita

Usaremos una miniatura pedagógica de proteína C reactiva. No es una regla clínica ni establece intervalos de referencia. Solo normaliza unidades y protege requisitos técnicos.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class MedicionPCR:
    paciente_id: str
    valor_mg_l: float
    fecha: str
    estado: str
    fuente: str
```

Salida esperada: no imprime nada. Define una representación inmutable para las mediciones aceptadas.

El sufijo `valor_mg_l` hace visible la unidad canónica. El objeto no conserva solo el número: conserva identidad, tiempo, estado y fuente.

## Normalizar sin decidir todavía

Normalizar transforma forma; validar decide si el resultado cumple el contrato.

```python
UNIDADES_PCR = {
    "mg/l": ("mg/L", 1.0),
    "mg/dl": ("mg/L", 10.0),
}


def normalizar_texto(valor):
    if valor is None:
        return None
    texto = str(valor).strip()
    return texto or None


def normalizar_unidad_pcr(unidad_cruda):
    unidad = normalizar_texto(unidad_cruda)
    if unidad is None:
        return None
    return UNIDADES_PCR.get(unidad.lower())


print(normalizar_texto(" P001 "))
print(normalizar_unidad_pcr("mg/dL"))
```

Salida esperada:

```text
P001
('mg/L', 10.0)
```

La función reconoce una equivalencia declarada. No adivina abreviaturas desconocidas ni convierte cualquier texto que se parezca a una unidad.

## Validar una fila y conservar la razón

La función siguiente produce uno de dos resultados:

- una medición aceptada;
- un rechazo estructurado con posición, identidad disponible y razones.

```python
def convertir_numero(valor_crudo):
    texto = normalizar_texto(valor_crudo)
    if texto is None:
        return None
    try:
        return float(texto.replace(",", "."))
    except ValueError:
        return None


def validar_fila_pcr(fila, posicion):
    paciente_id = normalizar_texto(fila.get("paciente_id"))
    fecha = normalizar_texto(fila.get("fecha"))
    estado = normalizar_texto(fila.get("estado"))
    fuente = normalizar_texto(fila.get("fuente"))
    valor = convertir_numero(fila.get("valor"))
    unidad = normalizar_unidad_pcr(fila.get("unidad"))

    razones = []
    if paciente_id is None:
        razones.append("paciente_id_ausente")
    if fecha is None:
        razones.append("fecha_ausente")
    if estado != "medido":
        razones.append("estado_no_medido")
    if fuente is None:
        razones.append("fuente_ausente")
    if valor is None:
        razones.append("valor_no_numerico_o_ausente")
    elif valor < 0:
        razones.append("valor_negativo")
    if unidad is None:
        razones.append("unidad_no_soportada")

    if razones:
        return None, {
            "posicion": posicion,
            "paciente_id": paciente_id,
            "razones": razones,
        }

    _, factor = unidad
    medicion = MedicionPCR(
        paciente_id=paciente_id,
        valor_mg_l=round(valor * factor, 6),
        fecha=fecha,
        estado=estado,
        fuente=fuente,
    )
    return medicion, None
```

Salida esperada: no imprime nada. Define el contrato de aceptación y rechazo.

La validación no borra una fila defectuosa. La transforma en evidencia de calidad del lote.

## Procesar el lote completo

```python
def procesar_lote_pcr(filas):
    aceptadas = []
    rechazadas = []

    for posicion, fila in enumerate(filas, start=1):
        medicion, rechazo = validar_fila_pcr(fila, posicion)
        if medicion is not None:
            aceptadas.append(medicion)
        else:
            rechazadas.append(rechazo)

    return aceptadas, rechazadas


filas_crudas = [
    {
        "paciente_id": " P001 ",
        "valor": "1.2",
        "unidad": "mg/dL",
        "fecha": "2026-08-01",
        "estado": "medido",
        "fuente": "laboratorio_demo",
    },
    {
        "paciente_id": "P002",
        "valor": "",
        "unidad": "mg/L",
        "fecha": "2026-08-01",
        "estado": "medido",
        "fuente": "laboratorio_demo",
    },
    {
        "paciente_id": "P003",
        "valor": "5",
        "unidad": "desconocida",
        "fecha": "2026-08-02",
        "estado": "medido",
        "fuente": "laboratorio_demo",
    },
    {
        "paciente_id": "P004",
        "valor": "16,2",
        "unidad": "mg/L",
        "fecha": "2026-08-02",
        "estado": "medido",
        "fuente": "laboratorio_demo",
    },
]

aceptadas, rechazadas = procesar_lote_pcr(filas_crudas)

print([(item.paciente_id, item.valor_mg_l) for item in aceptadas])
print([(item["posicion"], item["razones"]) for item in rechazadas])
```

Salida esperada:

```text
[('P001', 12.0), ('P004', 16.2)]
[(2, ['valor_no_numerico_o_ausente']), (3, ['unidad_no_soportada'])]
```

La primera fila fue convertida de `mg/dL` a `mg/L`. La segunda y la tercera no desaparecieron: quedaron representadas como rechazos explicables.

## Analizar con especificación y denominador

Un resultado reproducible necesita declarar qué calcula y sobre qué población técnica.

```python
def resumir_mediciones_pcr(mediciones, lote_id, regla_version):
    valores = [medicion.valor_mg_l for medicion in mediciones]
    media = sum(valores) / len(valores) if valores else None

    return {
        "analysis_version": "resumen-pcr-v1",
        "lote_id": lote_id,
        "regla_version": regla_version,
        "analito": "proteina_c_reactiva",
        "unidad": "mg/L",
        "denominador": len(valores),
        "media": round(media, 3) if media is not None else None,
        "minimo": min(valores) if valores else None,
        "maximo": max(valores) if valores else None,
    }


resumen = resumir_mediciones_pcr(
    aceptadas,
    lote_id="LOTE-PCR-001",
    regla_version="validacion-pcr-v1",
)

print(resumen["denominador"], resumen["media"])
```

Salida esperada:

```text
2 14.1
```

La media ahora tiene unidad, denominador, lote y versión de regla. Continúa siendo un resumen pedagógico, no una interpretación clínica.

## Construir el reporte de calidad

```python
from collections import Counter


def construir_reporte_calidad(total_filas, aceptadas, rechazadas, lote_id):
    razones = Counter(
        razon
        for rechazo in rechazadas
        for razon in rechazo["razones"]
    )
    proporcion_valida = len(aceptadas) / total_filas if total_filas else 0.0

    if not rechazadas:
        estado_lote = "aceptado"
    elif aceptadas:
        estado_lote = "requiere_correccion"
    else:
        estado_lote = "rechazado"

    return {
        "schema_version": "reporte-calidad-v1",
        "lote_id": lote_id,
        "estado_lote": estado_lote,
        "total_filas": total_filas,
        "filas_aceptadas": len(aceptadas),
        "filas_rechazadas": len(rechazadas),
        "proporcion_valida": round(proporcion_valida, 3),
        "rechazos_por_razon": dict(sorted(razones.items())),
    }


reporte = construir_reporte_calidad(
    total_filas=len(filas_crudas),
    aceptadas=aceptadas,
    rechazadas=rechazadas,
    lote_id="LOTE-PCR-001",
)

print(reporte["estado_lote"])
print(reporte["rechazos_por_razon"])
```

Salida esperada:

```text
requiere_correccion
{'unidad_no_soportada': 1, 'valor_no_numerico_o_ausente': 1}
```

El estado no depende de una impresión subjetiva. Surge de una política simple y visible. En un sistema real, esa política tendría umbrales, responsables y versión propia.

## Persistir no significa autorizar

Los registros aceptados podrían guardarse en SQLite, los rechazos en una tabla separada y la ejecución en una bitácora. La persistencia permite consultar y comparar; no convierte los datos en publicables.

Una arquitectura mínima separaría:

```text
mediciones_aceptadas
rechazos_validacion
ejecuciones_analisis
reportes_calidad
```

La separación evita que una fila rechazada reaparezca como medición válida por compartir la misma tabla.

## Preparar artefactos sin acoplar el cálculo a la escritura

Antes de escribir archivos, convertimos los objetos aceptados en filas serializables.

```python
from dataclasses import asdict


def preparar_filas_aceptadas(mediciones):
    return [asdict(medicion) for medicion in mediciones]


filas_exportables = preparar_filas_aceptadas(aceptadas)
print(filas_exportables[0])
```

Salida esperada:

```text
{'paciente_id': 'P001', 'valor_mg_l': 12.0, 'fecha': '2026-08-01', 'estado': 'medido', 'fuente': 'laboratorio_demo'}
```

La función solo prepara contenido. Otra función podrá escribir CSV; una tercera calculará huellas; una cuarta auditará. Esa separación reduce efectos invisibles.

## El paquete mínimo de evidencia

Una ejecución defendible podría producir:

```text
LOTE-PCR-001/
  filas_aceptadas.csv
  filas_rechazadas.json
  resumen_analisis.json
  reporte_calidad.json
  manifiesto.json
```

Cada archivo responde una pregunta diferente:

- qué pudo avanzar;
- qué no pudo avanzar y por qué;
- qué se calculó;
- cuál fue la calidad del lote;
- qué artefactos forman el paquete y cuáles son sus huellas.

El manifiesto conecta esas salidas sin obligarlas a tener el mismo formato.

## Prueba integradora

El cierre del capítulo debe probar propiedades del flujo, no solo una cifra final.

```python
# La conversión declarada conserva equivalencia de unidades.
assert aceptadas[0].valor_mg_l == 12.0

# Una fila inválida no desaparece.
assert len(aceptadas) + len(rechazadas) == len(filas_crudas)

# El análisis usa únicamente mediciones aceptadas y declara denominador.
assert resumen["denominador"] == len(aceptadas) == 2
assert resumen["unidad"] == "mg/L"

# El reporte conserva el tamaño original y las razones de rechazo.
assert reporte["total_filas"] == 4
assert reporte["filas_rechazadas"] == 2
assert reporte["rechazos_por_razon"] == {
    "unidad_no_soportada": 1,
    "valor_no_numerico_o_ausente": 1,
}

# La misma entrada produce la misma clasificación.
aceptadas_2, rechazadas_2 = procesar_lote_pcr(filas_crudas)
assert aceptadas_2 == aceptadas
assert rechazadas_2 == rechazadas
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas no demuestran que la fuente sea verdadera, que el paciente esté correctamente identificado o que la proteína C reactiva deba guiar una decisión. Demuestran propiedades técnicas del flujo pedagógico.

## CODE CLEAN: el coordinador no debe saberlo todo

Una función monolítica suele crecer así:

```python
def procesar_todo(filas):
    # Normaliza, valida, calcula, escribe archivos, imprime y decide.
    valores = [float(fila["valor"]) for fila in filas if fila["valor"]]
    print(sum(valores) / len(valores))
```

Salida esperada: depende de las filas recibidas. La función mezcla responsabilidades y no conserva rechazos.

Una coordinación más limpia describe el orden sin absorber el detalle de cada etapa:

```python
def ejecutar_flujo_pcr(filas, lote_id, regla_version):
    aceptadas, rechazadas = procesar_lote_pcr(filas)
    resumen = resumir_mediciones_pcr(aceptadas, lote_id, regla_version)
    reporte = construir_reporte_calidad(
        total_filas=len(filas),
        aceptadas=aceptadas,
        rechazadas=rechazadas,
        lote_id=lote_id,
    )
    return {
        "aceptadas": aceptadas,
        "rechazadas": rechazadas,
        "resumen": resumen,
        "reporte": reporte,
    }


ejecucion = ejecutar_flujo_pcr(
    filas_crudas,
    lote_id="LOTE-PCR-001",
    regla_version="validacion-pcr-v1",
)

print(ejecucion["reporte"]["estado_lote"])
```

Salida esperada:

```text
requiere_correccion
```

El coordinador sigue siendo explícito, pero cada función conserva una responsabilidad comprobable.

## Qué significa cerrar el capítulo

Al inicio, el lector podía ver valores y escribir operaciones. Ahora debe poder preguntar:

- ¿qué promete este dato?
- ¿qué unidad y estado conserva?
- ¿qué significa que esté ausente?
- ¿qué esquema debe cumplir?
- ¿qué entidad representa y con qué clave se relaciona?
- ¿qué registros fueron rechazados?
- ¿qué denominador usó el análisis?
- ¿qué versión de regla produjo el resultado?
- ¿qué artefacto permite reconstruirlo?
- ¿qué control detectaría una modificación?
- ¿qué límite técnico, clínico o legal sigue abierto?

Esa capacidad es más importante que memorizar una lista de tipos de Python.

## Lo que este flujo todavía no resuelve

Un pipeline verificable puede ser insuficiente por razones que el código no corrige por sí solo:

- la fuente puede estar sesgada o equivocada;
- la medición puede provenir de un instrumento mal calibrado;
- el identificador puede referirse a la persona incorrecta;
- la unidad puede estar bien convertida y el momento clínico mal elegido;
- la cohorte puede no representar la población de interés;
- la regla puede ser técnicamente consistente y clínicamente inapropiada;
- el resultado puede estar prohibido para la finalidad prevista;
- un archivo íntegro puede contener datos que nunca debieron compartirse.

La trazabilidad permite localizar estas preguntas. No las responde automáticamente.

## Errores frecuentes al integrar

**Normalizar y validar como si fueran lo mismo.** Cambiar forma no demuestra validez.

**Conservar solo los aceptados.** El lote pierde denominador, calidad y posibilidad de corrección.

**Calcular antes de fijar la unidad canónica.** La fórmula puede ser correcta y el resultado absurdo.

**Usar el reporte como sustituto de los datos.** Un agregado no permite inspeccionar cada caso.

**Usar los datos como sustituto del reporte.** Una tabla no resume por sí sola calidad, rechazos ni completitud.

**Escribir archivos dentro de cada función de dominio.** Los efectos quedan dispersos y las pruebas se vuelven frágiles.

**Versionar solo el código.** También cambian esquemas, vocabularios, consultas, parámetros y políticas.

**Confundir reproducibilidad con corrección.** Un error puede repetirse de manera perfectamente reproducible.

## Argumentos críticos

### Desacuerdo 1: rechazar temprano o corregir automáticamente

Pregunta: ¿no sería mejor corregir toda unidad o texto probable para salvar más registros?

La corrección automática aumenta rendimiento cuando las equivalencias son seguras y documentadas. También puede fabricar certeza cuando el texto es ambiguo.

Consenso operativo: normalizar equivalencias explícitas; rechazar o escalar lo ambiguo; nunca ocultar la transformación aplicada.

### Desacuerdo 2: pipeline estricto o exploración flexible

Pregunta: ¿un flujo con tantos contratos no impide explorar datos nuevos?

La exploración necesita flexibilidad. Sin embargo, una conclusión que sale del entorno exploratorio debe declarar qué datos y reglas terminó usando.

Consenso operativo: explorar con libertad controlada; promover a flujo reproducible solo las transformaciones que sostienen resultados compartidos.

### Desacuerdo 3: un reporte humano o artefactos para máquinas

Pregunta: ¿basta con un documento legible para explicar lo ocurrido?

Un documento favorece comprensión. Los artefactos estructurados favorecen comprobación, reutilización y automatización.

Consenso operativo: producir una narrativa proporcional para personas y contratos estructurados para sistemas cuando el uso lo requiera.

## Puente hacia algoritmos fundamentales

Hasta aquí hemos preparado el material sobre el que operarán algoritmos más explícitos.

El próximo capítulo preguntará, entre otras cosas:

- cómo buscar una observación sin revisar todo innecesariamente;
- cómo ordenar prioridades sin confundir orden técnico con urgencia clínica;
- cómo contar y agrupar de forma eficiente;
- cómo usar índices y hashing para recuperar información;
- cómo comparar costo temporal, espacial y operativo;
- cómo reconocer cuándo un problema se beneficia de recursión, grafos, estrategias voraces o programación dinámica.

Los algoritmos clásicos no operan en el vacío. Operan sobre representaciones. Si la representación pierde unidad, identidad, ausencia o procedencia, un algoritmo más rápido solo procesa el error con mayor eficiencia.

## Preguntas de comprensión profunda

1. ¿Por qué una media correctamente calculada puede ser inválida antes de revisar su fórmula?
2. ¿Qué diferencia existe entre normalizar, validar y analizar?
3. ¿Por qué aceptados y rechazados forman juntos la evidencia de una ejecución?
4. ¿Qué propiedades debe probar una prueba integradora además del resultado final?
5. ¿Por qué el denominador pertenece al contrato del análisis?
6. ¿Qué información aporta un reporte de calidad que no aporta la tabla aceptada?
7. ¿Qué aporta un manifiesto que no aporta el reporte de calidad?
8. ¿Por qué un flujo reproducible puede seguir siendo clínicamente incorrecto?
9. ¿Qué responsabilidad debería tener el coordinador y cuáles debería delegar?
10. ¿Cómo prepara la representación de datos el estudio de algoritmos clásicos?

## Vacíos de comprensión que debes vigilar

1. Creer que la última función del pipeline es la más importante. El resultado depende de todos los contratos anteriores.
2. Creer que rechazar una fila equivale a perderla. Un rechazo bien estructurado conserva información operativa.
3. Creer que un artefacto auditable autoriza su uso. Integridad, privacidad, licencia y finalidad son controles distintos.
4. Creer que el capítulo trató principalmente de bibliotecas. Su tema real fue la representación verificable.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma un lote pequeño y dibuja sus etapas, entradas, salidas y razones de rechazo antes de programar.
2. **Segunda hora:** implementa normalización y validación como funciones separadas; comprueba conservación del total `aceptados + rechazados`.
3. **Tercera hora:** añade un análisis con denominador, un reporte de calidad y una prueba integradora que detecte una modificación intencional de la entrada.

## Bibliografía y fuentes

- Python Software Foundation. (s. f.). *dataclasses — Data Classes*. <https://docs.python.org/3/library/dataclasses.html>.
- Python Software Foundation. (s. f.). *collections — Container datatypes*. <https://docs.python.org/3/library/collections.html#collections.Counter>.
- Python Software Foundation. (s. f.). *csv — CSV File Reading and Writing*. <https://docs.python.org/3/library/csv.html>.
- Python Software Foundation. (s. f.). *json — JSON encoder and decoder*. <https://docs.python.org/3/library/json.html>.
- Wilkinson, M. D., Dumontier, M., Aalbersberg, I. J. J., et al. (2016). The FAIR Guiding Principles for scientific data management and stewardship. *Scientific Data, 3*, 160018. <https://doi.org/10.1038/sdata.2016.18>.
- World Wide Web Consortium. (2013). *PROV-Overview: An Overview of the PROV Family of Documents*. <https://www.w3.org/TR/prov-overview/>.

## Cierre del Capítulo II

Un dato biomédico útil no es el que simplemente cabe en una variable. Es el que conserva suficiente significado para atravesar transformaciones sin que el sistema fabrique una certeza nueva.

El Capítulo II termina con una capacidad: transformar un lote crudo en un flujo verificable que declara contratos, separa errores, conserva denominadores, produce artefactos y reconoce sus límites.

El siguiente capítulo abrirá una pregunta distinta: una vez que los datos tienen forma defendible, ¿qué algoritmos permiten buscarlos, ordenarlos, recorrerlos, relacionarlos y resolver problemas con un costo explícito?
