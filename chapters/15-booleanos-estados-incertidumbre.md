# Booleanos, estados e incertidumbre

Un booleano parece el tipo de dato más claro del mundo: `True` o `False`. Verdadero o falso. Sí o no. Presente o ausente. En programación, esa claridad es útil. En medicina y ciencias de la vida, puede ser peligrosa si se usa sin contrato.

Un paciente puede tener un síntoma presente, ausente, no evaluado, desconocido, dudoso, no aplicable o pendiente de confirmación. Un resultado puede ser positivo, negativo, indeterminado, inválido o no realizado. Una contraindicación puede estar presente, ausente o no documentada. Reducir todos esos estados a `True` y `False` puede producir una salida limpia y una decisión pobre.

La sección anterior mostró que el texto libre necesita vocabulario controlado. Esta sección agrega una tercera regla para la unidad: no todo dato que parece binario debe representarse como booleano.

## Origen técnico: verdad lógica no es verdad clínica

Python tiene un tipo booleano: `bool`. Sus dos valores son `True` y `False`. También permite evaluar muchos objetos por "valor de verdad" dentro de condiciones: `0`, `None`, listas vacías y cadenas vacías se comportan como falso; números distintos de cero, listas no vacías y cadenas no vacías se comportan como verdadero.

Esa capacidad es cómoda para programar. Pero no debe confundirse con significado biomédico. Que `None` se evalúe como falso en una condición no significa que "dato ausente" sea igual a "hallazgo ausente". Que `"no"` sea una cadena no vacía y por tanto verdadera no significa que exprese presencia de un hallazgo.

La lógica del lenguaje sirve para controlar flujo. La lógica del dominio debe decidir qué significa cada estado.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un booleano representa una distinción estrictamente binaria. Un estado biomédico representa una situación interpretada dentro de un contrato de dominio. Cuando existen ausencia de dato, incertidumbre, no aplicabilidad o revisión pendiente, el estado debe ser explícito y no esconderse dentro de `True` o `False`.
</div>

Esta definición tiene cuatro consecuencias.

**Falso no significa ausente.** Puede significar ausente, no registrado, no evaluado, inválido, no aplicable o simplemente una conversión técnica.

**Verdadero no significa confirmado.** Puede significar que una cadena no está vacía, que una lista tiene elementos o que una regla parcial se activó.

**La incertidumbre debe tener nombre.** Si un dato puede estar pendiente, indeterminado o desconocido, ese estado debe aparecer en la estructura.

**Los booleanos siguen siendo útiles.** Hay preguntas realmente binarias: un archivo existe o no existe, una prueba pasó o falló, una clave está presente o ausente en un diccionario. El problema no es `bool`; el problema es usarlo donde el dominio necesita más estados.

## Versión ingenua: convertir incertidumbre en falso

Supongamos una regla pedagógica de alerta respiratoria que pregunta por fiebre y disnea.

```python
def evaluar_alerta_respiratoria(tiene_fiebre, tiene_disnea):
    # Versión frágil: trata cualquier valor no verdadero como ausencia del hallazgo.
    if tiene_fiebre and tiene_disnea:
        return "alerta"
    return "sin_alerta"


print(evaluar_alerta_respiratoria(True, True))
print(evaluar_alerta_respiratoria(None, True))
print(evaluar_alerta_respiratoria(False, True))
```

Salida esperada:

```text
alerta
sin_alerta
sin_alerta
```

La función ejecuta, pero mezcla dos situaciones distintas. `False` puede representar fiebre ausente. `None` representa dato no disponible. Ambas terminan como `sin_alerta`. El programa no falló; el modelo de datos falló.

Esta es una de las formas más discretas de error: la ausencia de información se convierte en información negativa.

## Crítica técnica: qué está mal

Primero, la función recibe valores sin contrato. El nombre `tiene_fiebre` sugiere un booleano, pero el ejemplo demuestra que puede llegar `None`. Si el sistema admite `None`, debe decidir qué significa.

Segundo, el `if` usa valor de verdad de Python como si fuera interpretación clínica. `None` es falso para el flujo del programa; no es "fiebre ausente" para el dominio.

Tercero, la salida no conserva razón. `sin_alerta` puede significar que ambos hallazgos están ausentes, que uno falta, que uno no aplica o que el dato llegó mal.

Cuarto, no hay espacio para incertidumbre. La regla solo puede decidir o negar. En biomedicina, una salida responsable a veces debe decir: evaluación incompleta.

## Versión mejorada: estados controlados

Podemos separar el estado del hallazgo de la decisión posterior. Para eso usaremos `Enum`, igual que en la sección anterior, pero ahora para estados que suelen confundirse con booleanos.

```python
from enum import Enum


class EstadoDato(Enum):
    PRESENTE = "presente"
    AUSENTE = "ausente"
    DESCONOCIDO = "desconocido"
    NO_EVALUADO = "no_evaluado"
    NO_APLICA = "no_aplica"


def validar_estado(valor):
    """Acepta solo estados controlados antes de una decisión biomédica."""
    if isinstance(valor, EstadoDato):
        return {"estado": valor, "razon": "estado_controlado"}

    if valor is True:
        return {"estado": EstadoDato.PRESENTE, "razon": "booleano_convertido"}

    if valor is False:
        return {"estado": EstadoDato.AUSENTE, "razon": "booleano_convertido"}

    if valor is None:
        return {"estado": EstadoDato.NO_EVALUADO, "razon": "dato_no_evaluado"}

    return {"estado": EstadoDato.DESCONOCIDO, "razon": "valor_fuera_de_contrato"}


def evaluar_alerta_respiratoria(fiebre, disnea):
    """Evalúa una miniatura pedagógica sin convertir incertidumbre en ausencia."""
    fiebre_validada = validar_estado(fiebre)
    disnea_validada = validar_estado(disnea)

    estados = [fiebre_validada["estado"], disnea_validada["estado"]]
    razones = [fiebre_validada["razon"], disnea_validada["razon"]]

    if EstadoDato.DESCONOCIDO in estados or EstadoDato.NO_EVALUADO in estados:
        return {
            "decision": "evaluacion_incompleta",
            "accion": "revisar_datos",
            "razon": razones,
        }

    if estados == [EstadoDato.PRESENTE, EstadoDato.PRESENTE]:
        return {
            "decision": "alerta",
            "accion": "interpretar_con_contexto",
            "razon": "fiebre_y_disnea_presentes",
        }

    return {
        "decision": "sin_alerta_por_regla",
        "accion": "continuar_observacion",
        "razon": "criterio_pedagogico_no_cumplido",
    }


print(evaluar_alerta_respiratoria(None, True)["decision"])
print(evaluar_alerta_respiratoria(False, True)["decision"])
```

Salida esperada:

```text
evaluacion_incompleta
sin_alerta_por_regla
```

La diferencia es pequeña en código y grande en significado. `None` ya no cae silenciosamente en el mismo saco que `False`. El programa conserva que una evaluación está incompleta.

## Anatomía del contrato

La primera pieza es el vocabulario de estados.

```python
class EstadoDato(Enum):
    PRESENTE = "presente"
    AUSENTE = "ausente"
    DESCONOCIDO = "desconocido"
    NO_EVALUADO = "no_evaluado"
    NO_APLICA = "no_aplica"
```

Cada estado responde una pregunta distinta.

- `PRESENTE`: el hallazgo o condición fue identificado.
- `AUSENTE`: fue evaluado y no se encontró.
- `DESCONOCIDO`: el valor recibido no permite interpretación confiable.
- `NO_EVALUADO`: todavía no hay dato.
- `NO_APLICA`: la pregunta no corresponde a ese caso.

La segunda pieza es la validación antes de decidir.

```python
if valor is None:
    return {"estado": EstadoDato.NO_EVALUADO, "razon": "dato_no_evaluado"}
```

Aquí `None` no se convierte en `False`. Se convierte en un estado semántico. Esa traducción es parte del contrato del dominio.

La tercera pieza es la decisión.

```python
if EstadoDato.DESCONOCIDO in estados or EstadoDato.NO_EVALUADO in estados:
    return {
        "decision": "evaluacion_incompleta",
        "accion": "revisar_datos",
        "razon": razones,
    }
```

La regla reconoce que hay situaciones donde no debe clasificar. Esta salida puede parecer menos útil que una etiqueta rápida, pero protege contra falsa certeza.

## Pruebas mínimas

```python
# Propiedad 1: dos hallazgos presentes activan la alerta pedagógica.
assert evaluar_alerta_respiratoria(True, True)["decision"] == "alerta"

# Propiedad 2: un dato no evaluado no se trata como ausencia.
assert evaluar_alerta_respiratoria(None, True)["decision"] == "evaluacion_incompleta"

# Propiedad 3: ausencia evaluada conserva una salida distinta a dato faltante.
assert evaluar_alerta_respiratoria(False, True)["decision"] == "sin_alerta_por_regla"

# Propiedad 4: un valor fuera de contrato degrada la decisión.
assert evaluar_alerta_respiratoria("no", True)["decision"] == "evaluacion_incompleta"

# Propiedad 5: estados controlados pueden entrar sin conversión adicional.
assert evaluar_alerta_respiratoria(
    EstadoDato.PRESENTE,
    EstadoDato.AUSENTE,
)["decision"] == "sin_alerta_por_regla"
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Las pruebas no validan una regla clínica. Validan una propiedad de representación: desconocido, no evaluado y ausente no son lo mismo.

## Trampa clásica: cadenas que parecen booleanos

Una fuente frecuente de errores aparece cuando un formulario, archivo CSV o API envía texto.

```python
print(bool("False"))
print(bool("no"))
print(bool(""))
```

Salida esperada:

```text
True
True
False
```

Python no está interpretando español ni inglés. Solo evalúa si la cadena está vacía. `"False"` y `"no"` son cadenas con contenido, por eso se comportan como verdaderas en una condición.

Por eso un sistema que recibe texto debe parsear de forma explícita.

```python
def interpretar_respuesta_binaria(texto):
    """Interpreta respuestas textuales simples sin depender de bool(texto)."""
    if texto is None:
        return EstadoDato.NO_EVALUADO

    texto_normalizado = texto.strip().lower()
    respuestas_positivas = {"si", "sí", "true", "presente"}
    respuestas_negativas = {"no", "false", "ausente"}

    if texto_normalizado in respuestas_positivas:
        return EstadoDato.PRESENTE

    if texto_normalizado in respuestas_negativas:
        return EstadoDato.AUSENTE

    return EstadoDato.DESCONOCIDO


print(interpretar_respuesta_binaria("no").value)
print(interpretar_respuesta_binaria("False").value)
print(interpretar_respuesta_binaria("").value)
```

Salida esperada:

```text
ausente
ausente
desconocido
```

El punto no es construir un parser universal. El punto es no delegar significado biomédico a la conversión automática de Python.

## Ejemplo biomédico progresivo: elegibilidad pedagógica

La elegibilidad para una intervención, protocolo o ruta de atención suele parecer binaria: elegible o no elegible. En realidad, muchas veces hay una tercera salida: no se puede decidir todavía.

```python
def evaluar_elegibilidad_pedagogica(consentimiento, criterio_inclusion, contraindicacion):
    """Miniatura pedagógica: separa elegible, no elegible y evaluación incompleta."""
    datos = {
        "consentimiento": validar_estado(consentimiento)["estado"],
        "criterio_inclusion": validar_estado(criterio_inclusion)["estado"],
        "contraindicacion": validar_estado(contraindicacion)["estado"],
    }

    if EstadoDato.NO_EVALUADO in datos.values() or EstadoDato.DESCONOCIDO in datos.values():
        return {
            "estado": "evaluacion_incompleta",
            "razon": "faltan_datos_para_decidir",
            "datos": datos,
        }

    if datos["contraindicacion"] == EstadoDato.PRESENTE:
        return {
            "estado": "no_elegible",
            "razon": "contraindicacion_presente",
            "datos": datos,
        }

    if datos["consentimiento"] == EstadoDato.PRESENTE and datos["criterio_inclusion"] == EstadoDato.PRESENTE:
        return {
            "estado": "elegible_por_regla_pedagogica",
            "razon": "criterios_minimos_cumplidos",
            "datos": datos,
        }

    return {
        "estado": "no_elegible",
        "razon": "criterios_minimos_no_cumplidos",
        "datos": datos,
    }


resultado = evaluar_elegibilidad_pedagogica(
    consentimiento=True,
    criterio_inclusion=True,
    contraindicacion=None,
)

print(resultado["estado"])
print(resultado["razon"])
```

Salida esperada:

```text
evaluacion_incompleta
faltan_datos_para_decidir
```

Este ejemplo no define elegibilidad clínica real. No revisa indicación, riesgo, beneficio, contexto, edad, comorbilidades ni preferencias. Solo enseña una estructura: antes de decidir, separar lo afirmado, lo negado y lo no conocido.

## CODE CLEAN: no esconder significado en `if dato`

Comparemos dos estilos.

```python
if consentimiento:
    estado = "continua"
```

El código es breve, pero oculta una pregunta: ¿qué significa `consentimiento`? ¿Es `True`, `"si"`, un formulario firmado, un campo no vacío, una fecha, un documento, una observación pendiente?

Una versión más honesta separa validación y decisión.

```python
estado_consentimiento = validar_estado(consentimiento)["estado"]

if estado_consentimiento == EstadoDato.PRESENTE:
    estado = "continua"
elif estado_consentimiento == EstadoDato.NO_EVALUADO:
    estado = "requiere_documentar_consentimiento"
else:
    estado = "no_continua"
```

Hay más líneas, pero también más verdad operacional. Código limpio no es escribir menos; es hacer visible el supuesto que puede cambiar la decisión.

## Límites y errores frecuentes

1. **Usar `if dato` con datos biomédicos crudos.** Puede confundir ausencia, vacío, cero, texto y falso.
2. **Convertir `None` en `False`.** Borra la diferencia entre no evaluado y ausente.
3. **Aceptar cadenas como booleanos.** `"False"` y `"no"` son verdaderas si solo se aplica `bool(texto)`.
4. **Crear demasiados estados sin acción.** Un estado solo vale si cambia validación, decisión, trazabilidad o revisión.
5. **Usar `NO_APLICA` como basurero.** No aplicable significa que la pregunta no corresponde, no que faltó diligenciarla.
6. **Confundir indeterminado con desconocido.** Un resultado indeterminado puede venir de una prueba realizada; desconocido puede significar que no hay información interpretable.

## Argumentos críticos

### Desacuerdo 1: booleano contra enumeración

Pregunta: ¿cuándo basta con `True` y `False`?

El booleano basta cuando la pregunta es estrictamente binaria y el dato llega validado: un test de software pasó o falló, una clave existe en un diccionario, una opción fue activada o desactivada. La enumeración es mejor cuando la pregunta puede tener incertidumbre, no aplicabilidad, dato pendiente o resultado indeterminado.

Consenso operativo: usar `bool` para control de flujo técnico; usar estados controlados cuando la salida represente significado biomédico.

### Desacuerdo 2: detener contra degradar

Pregunta: si falta un estado, ¿el algoritmo debe detenerse?

Detener protege contra falsa certeza. Degradar permite conservar una salida útil: `evaluacion_incompleta`, `requiere_revision`, `no_calculable`. El riesgo depende de la decisión posterior.

Consenso operativo: si el estado faltante puede cambiar una acción importante, no producir una clasificación final. Devolver una salida degradada con razón.

### Desacuerdo 3: más estados contra simplicidad

Pregunta: ¿cuántos estados son suficientes?

Pocos estados simplifican el código, pero pueden borrar diferencias relevantes. Demasiados estados complican pruebas, interfaz y mantenimiento.

Consenso operativo: agregar estados solo cuando cambien una decisión, una prueba, una acción, una auditoría o una explicación.

## Puente hacia la frontera

Los booleanos son el primer contacto con un problema mayor: la incertidumbre. Más adelante, el libro tendrá que representar probabilidades, intervalos de confianza, sensibilidad, especificidad, valores predictivos, calibración, clases indeterminadas y decisiones bajo riesgo.

La frontera no elimina los estados discretos. Los combina con estimaciones, probabilidades y umbrales. Un sistema avanzado puede decir que una variante genética es patogénica, probablemente patogénica, de significado incierto, probablemente benigna o benigna. Un clasificador puede producir probabilidad, confianza, advertencia de fuera de distribución y recomendación de revisión.

La lección de esta sección seguirá vigente: antes de calcular, decidir qué estados existen y qué significa cada uno.

## Evaluar si entendiste

1. ¿Por qué `None` no debe tratarse como `False` en un dato biomédico?
2. ¿Cuándo un booleano sí es una buena representación?
3. ¿Qué diferencia hay entre `AUSENTE`, `NO_EVALUADO` y `NO_APLICA`?
4. ¿Por qué `bool("False")` devuelve `True`?
5. ¿Qué daño puede producir una salida `sin_alerta` cuando faltaba un dato importante?
6. ¿Por qué un estado debe cambiar alguna acción o explicación?
7. ¿Cuándo conviene devolver `evaluacion_incompleta`?
8. ¿Qué diferencia hay entre controlar flujo técnico y representar significado biomédico?
9. ¿Qué prueba escribirías para asegurar que dato faltante no se convierte en ausencia?
10. ¿Cómo crecería esta sección cuando entren probabilidades y calibración?

## Vacíos de comprensión que debes vigilar

1. Creer que `bool` es simple y por tanto siempre seguro. Su simplicidad técnica no garantiza suficiencia de dominio.
2. Pensar que más estados siempre significan mejor modelo. Los estados deben justificar acciones o trazabilidad.
3. Confundir incertidumbre con error. A veces la incertidumbre es el estado correcto del conocimiento, no una falla del sistema.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma cinco preguntas de un formulario clínico o científico y decide cuáles son realmente binarias y cuáles necesitan más estados.
2. **Segunda hora:** implementa un `Enum` para una de esas preguntas y escribe una función que rechace valores fuera de contrato.
3. **Tercera hora:** escribe pruebas para `True`, `False`, `None`, texto desconocido y estado `NO_APLICA`.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *Built-in Types: Truth Value Testing and Boolean Type*. Python 3.14.6 documentation. <https://docs.python.org/3/library/stdtypes.html#truth-value-testing>
- Python Software Foundation. (2026). *enum — Support for enumerations*. Python 3.12 documentation. <https://docs.python.org/3.12/library/enum.html>
- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>
- Possolo, A., Gelabert, M. V., Hibbert, D., Stohner, J., Bodnar, O., & Meija, J. (2025). *Guía breve sobre la incertidumbre de medición*. NIST Technical Note 2330. <https://doi.org/10.6028/NIST.TN.2330>
- Taylor, B. N., & Kuyatt, C. E. (1994). *Guidelines for Evaluating and Expressing the Uncertainty of NIST Measurement Results*. NIST Technical Note 1297. <https://www.nist.gov/pml/nist-technical-note-1297>

## Siguiente paso

Los booleanos muestran que una decisión necesita estados explícitos antes de parecer simple. El próximo problema será el tiempo: fechas, horas, intervalos y granularidad clínica. En biomedicina, una fecha no solo dice cuándo ocurrió algo; también puede decir con qué precisión se sabe, en qué orden ocurrió, cuánto tiempo pasó y qué tan comparable es con otro evento.
