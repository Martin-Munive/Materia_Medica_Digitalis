# Texto libre, códigos y vocabularios controlados

Un texto biomédico parece menos peligroso que un número. Si un programa recibe `disnea`, `dificultad respiratoria`, `ahogo`, `falta de aire` o `dyspnea`, un lector humano puede reconocer una familia de significados. Python no. Para Python, cada una de esas formas es una cadena distinta. Puede compararlas, imprimirlas, cortarlas, unirlas y buscarlas. No puede decidir por sí solo si hablan del mismo fenómeno.

La sección anterior mostró que un número sin unidad es ambiguo. Esta sección agrega una segunda regla: un texto sin contrato semántico es flexible, pero no necesariamente interpretable.

El texto libre es indispensable para narrar. Una historia clínica, una impresión diagnóstica, una nota operatoria o una observación cualitativa necesitan lenguaje. Pero cuando el sistema debe contar, filtrar, agrupar, comparar o activar una regla, el texto libre deja de ser suficiente. En ese punto se necesita una frontera: qué queda como narración, qué se normaliza y qué se codifica.

## Origen técnico: escribir no es codificar

Python representa texto con el tipo `str`. Ese tipo conserva secuencias Unicode y permite operaciones potentes: buscar subcadenas, convertir a minúsculas, reemplazar caracteres, dividir frases, eliminar espacios y formatear salidas. Es una herramienta técnica para manipular texto, no un vocabulario clínico.

Un vocabulario controlado cumple otra función. Define un conjunto limitado de valores aceptables y, cuando el sistema madura, puede conectar esos valores con códigos estables. El código no reemplaza al texto visible; lo ancla. Permite que `disnea`, `dificultad respiratoria` y una etiqueta local no se pierdan como variantes imposibles de agregar.

En sistemas biomédicos reales aparecen estándares como LOINC para observaciones, mediciones y documentos, y SNOMED CT para conceptos clínicos con significado formal. En este libro no implementaremos esos estándares todavía. Los usaremos como horizonte: muestran que el problema del texto no es cosmético, sino semántico e interoperable.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un vocabulario controlado es un conjunto explícito de valores permitidos, cada uno con significado definido, nombre legible y, cuando corresponde, código estable. Su función es reducir ambigüedad sin borrar la posibilidad de conservar texto narrativo.
</div>

Esta definición separa cuatro objetos que a menudo se mezclan.

**Texto libre.** Lenguaje escrito por una persona o importado desde una fuente externa. Conserva matices, pero no promete uniformidad.

**Texto normalizado.** Texto transformado para comparación técnica: minúsculas, espacios controlados, acentos manejados, variantes reducidas. Ayuda, pero no resuelve el significado.

**Categoría controlada.** Valor permitido dentro de una lista cerrada. Por ejemplo: `presente`, `ausente`, `desconocido`, `no_evaluado`.

**Código.** Identificador estable que apunta a un concepto, observación, procedimiento o documento dentro de un sistema definido.

La regla práctica es esta: si un texto solo será leído, puede permanecer libre. Si un texto será usado para decidir, agrupar o medir, necesita al menos normalización y vocabulario controlado. Si además debe cruzar sistemas, necesita código.

## Versión ingenua: buscar palabras dentro de frases

Supongamos que queremos detectar si una nota menciona disnea.

```python
def detectar_disnea(nota):
    # Versión frágil: busca una palabra literal dentro del texto.
    if "disnea" in nota.lower():
        return "presente"
    return "ausente"


print(detectar_disnea("Paciente con disnea de esfuerzo."))
print(detectar_disnea("Niega disnea."))
print(detectar_disnea("Refiere falta de aire al caminar."))
```

Salida esperada:

```text
presente
presente
ausente
```

La función ejecuta, pero interpreta mal. Marca `Niega disnea` como presencia, porque encontró la palabra. No reconoce `falta de aire`, porque no aparece la palabra exacta. Tampoco distingue texto no evaluado, negación, sinónimo, idioma, abreviatura o error ortográfico.

El problema no es que `str` sea débil. El problema es pedirle a una cadena que haga trabajo semántico sin contrato.

## Crítica técnica: qué está mal

Primero, la búsqueda literal confunde mención con afirmación. Un texto puede decir que un síntoma está presente, negado, descartado, pendiente, referido por un familiar o usado en una explicación educativa. La cadena contiene palabras; el registro clínico contiene estados.

Segundo, la función no declara vocabulario. Devuelve `presente` o `ausente`, pero esos valores no están gobernados por una lista cerrada. Otro programador podría devolver `si`, `sí`, `positivo`, `no`, `negado` o `normal`, y el sistema empezaría a acumular variantes.

Tercero, no conserva la razón. Una decisión textual responsable debe poder decir si clasificó por coincidencia exacta, por sinónimo aceptado, por código recibido o por rechazo del dato.

Cuarto, no separa narración de estructura. El texto original puede ser valioso y debe conservarse; pero la decisión necesita una representación estable.

## Versión mejorada: vocabulario controlado mínimo

Python ofrece `Enum` para representar conjuntos cerrados de nombres simbólicos. No convierte automáticamente un texto clínico en concepto, pero ayuda a evitar que los estados se dispersen.

```python
from enum import Enum
import unicodedata


class EstadoHallazgo(Enum):
    PRESENTE = "presente"
    AUSENTE = "ausente"
    DESCONOCIDO = "desconocido"
    NO_EVALUADO = "no_evaluado"


VOCABULARIO_DISNEA = {
    "disnea": EstadoHallazgo.PRESENTE,
    "dificultad respiratoria": EstadoHallazgo.PRESENTE,
    "falta de aire": EstadoHallazgo.PRESENTE,
    "ahogo": EstadoHallazgo.PRESENTE,
    "sin disnea": EstadoHallazgo.AUSENTE,
    "niega disnea": EstadoHallazgo.AUSENTE,
    "no refiere disnea": EstadoHallazgo.AUSENTE,
}


def normalizar_texto(texto):
    """Prepara texto para comparación técnica sin destruir el original."""
    texto = texto.strip().lower()
    texto = " ".join(texto.split())
    texto = unicodedata.normalize("NFD", texto)
    texto = "".join(caracter for caracter in texto if unicodedata.category(caracter) != "Mn")
    return texto


def validar_hallazgo_textual(texto, vocabulario):
    """Convierte una expresión textual en un estado controlado si existe equivalencia."""
    if texto is None:
        return {
            "estado": EstadoHallazgo.NO_EVALUADO,
            "accion": "conservar_ausencia",
            "razon": "texto_ausente",
        }

    texto_normalizado = normalizar_texto(texto)
    if texto_normalizado not in vocabulario:
        return {
            "estado": EstadoHallazgo.DESCONOCIDO,
            "accion": "revision_humana",
            "razon": "expresion_fuera_de_vocabulario",
        }

    return {
        "estado": vocabulario[texto_normalizado],
        "accion": "aceptar_estado_controlado",
        "razon": "expresion_mapeada",
    }


resultado = validar_hallazgo_textual("Niega disnea", VOCABULARIO_DISNEA)
print(resultado["estado"].value)
print(resultado["razon"])
```

Salida esperada:

```text
ausente
expresion_mapeada
```

Esta versión tampoco interpreta lenguaje clínico general. Hace algo más modesto y más seguro: acepta solo expresiones conocidas, conserva estados controlados y manda a revisión lo que no puede mapear.

## Anatomía del contrato

La primera pieza es el conjunto cerrado de estados.

```python
class EstadoHallazgo(Enum):
    PRESENTE = "presente"
    AUSENTE = "ausente"
    DESCONOCIDO = "desconocido"
    NO_EVALUADO = "no_evaluado"
```

El valor ya no es una cadena arbitraria. Es un miembro de un vocabulario interno. Eso permite comparar por identidad, listar opciones válidas y evitar variantes accidentales.

La segunda pieza es la normalización.

```python
texto = texto.strip().lower()
texto = " ".join(texto.split())
```

Esta normalización elimina diferencias técnicas simples: mayúsculas, espacios sobrantes y formas equivalentes para comparación. No pretende entender negación, temporalidad ni contexto. Solo prepara el texto para una tabla explícita.

La tercera pieza es el mapeo.

```python
if texto_normalizado not in vocabulario:
    return {
        "estado": EstadoHallazgo.DESCONOCIDO,
        "accion": "revision_humana",
        "razon": "expresion_fuera_de_vocabulario",
    }
```

Cuando una expresión queda por fuera del vocabulario, el programa no inventa. Devuelve un estado controlado y una acción: revisión humana. Este comportamiento es menos espectacular que "entender" lenguaje natural, pero más responsable para un sistema inicial.

## Pruebas mínimas

```python
# Propiedad 1: expresión afirmativa conocida.
assert validar_hallazgo_textual("Falta de aire", VOCABULARIO_DISNEA)["estado"] == EstadoHallazgo.PRESENTE

# Propiedad 2: expresión negativa conocida.
assert validar_hallazgo_textual("Niega disnea", VOCABULARIO_DISNEA)["estado"] == EstadoHallazgo.AUSENTE

# Propiedad 3: ausencia de texto no se transforma en ausencia clínica.
assert validar_hallazgo_textual(None, VOCABULARIO_DISNEA)["estado"] == EstadoHallazgo.NO_EVALUADO

# Propiedad 4: expresión desconocida no se clasifica por intuición.
assert validar_hallazgo_textual("respira raro", VOCABULARIO_DISNEA)["estado"] == EstadoHallazgo.DESCONOCIDO

# Propiedad 5: la normalización controla mayúsculas, acentos y espacios.
assert validar_hallazgo_textual("  DÍSNEA  ", VOCABULARIO_DISNEA)["estado"] == EstadoHallazgo.PRESENTE
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas no validan una ontología clínica. Validan que el sistema no miente sobre lo que sabe. Esa es la propiedad mínima de un vocabulario controlado: aceptar lo conocido, rechazar lo no gobernado y conservar la diferencia entre ausencia de texto y ausencia del hallazgo.

## Ejemplo biomédico progresivo: observación con código local

Un sistema pequeño puede empezar con códigos locales, siempre que declare que son locales. El código local no reemplaza estándares; permite ordenar el aprendizaje y luego mapear con más cuidado.

```python
CATALOGO_OBSERVACIONES = {
    "MMD-DEMO-001": {
        "nombre": "disnea",
        "sistema": "MMD-DEMO",
        "version": "2026-08",
        "vocabulario": VOCABULARIO_DISNEA,
    }
}


def evaluar_observacion_textual(codigo_observacion, texto):
    """Evalúa texto solo si el código de observación existe en el catálogo local."""
    if codigo_observacion not in CATALOGO_OBSERVACIONES:
        return {
            "estado": EstadoHallazgo.DESCONOCIDO,
            "accion": "revisar_codigo",
            "razon": "codigo_observacion_no_registrado",
        }

    observacion = CATALOGO_OBSERVACIONES[codigo_observacion]
    resultado = validar_hallazgo_textual(texto, observacion["vocabulario"])

    return {
        "codigo": codigo_observacion,
        "sistema": observacion["sistema"],
        "version": observacion["version"],
        "estado": resultado["estado"],
        "accion": resultado["accion"],
        "razon": resultado["razon"],
    }


print(evaluar_observacion_textual("MMD-DEMO-001", "No refiere disnea")["estado"].value)
```

Salida esperada:

```text
ausente
```

La diferencia es pequeña pero decisiva. Ya no clasificamos una frase aislada. Clasificamos una observación identificada dentro de un catálogo. El resultado conserva código, sistema, versión, estado, acción y razón. Esa estructura será compatible con decisiones posteriores: filtros, conteos, auditorías o migraciones.

## CODE CLEAN: nombres que exponen semántica

Comparemos dos nombres.

```python
def procesar(x):
    return x.lower()
```

El código funciona, pero no dice qué promete. `x` puede ser una nota, un diagnóstico, una unidad, una contraseña o un apellido. `procesar` no informa si normaliza, valida, codifica, resume o clasifica.

Una versión más honesta nombra la intención.

```python
def normalizar_texto_para_vocabulario(texto_original):
    texto = texto_original.strip().lower()
    return " ".join(texto.split())
```

No es solo estética. El nombre declara el límite: prepara texto para compararlo contra un vocabulario. No afirma que entiende lenguaje natural. No afirma que codifica una historia clínica. No afirma que produce diagnóstico. Código limpio, aquí, es precisión semántica.

## Límites y errores frecuentes

1. **Confundir texto normalizado con concepto.** Quitar acentos y minúsculas ayuda a comparar; no convierte una frase en significado clínico.
2. **Usar texto libre como categoría.** Si `positivo`, `pos`, `sí`, `si`, `presente` y `+` significan lo mismo, el sistema necesita un vocabulario.
3. **Borrar el texto original.** Normalizar sirve para comparar; el texto original puede ser necesario para auditoría, explicación o revisión.
4. **Codificar demasiado pronto.** Asignar códigos sin conocer el dominio puede producir falsa interoperabilidad.
5. **Mezclar vocabularios.** Un código local, un código LOINC, un concepto SNOMED CT y una etiqueta visual no son intercambiables si no se declara el sistema.
6. **Tratar negación como ausencia simple.** "Niega fiebre" no es lo mismo que "fiebre no evaluada" ni que "no aparece la palabra fiebre".

## Argumentos críticos

### Desacuerdo 1: texto libre contra estructura

Pregunta: ¿por qué no dejar todo como texto y usar búsqueda?

El argumento a favor del texto libre dice que preserva matices y evita forzar categorías pobres. Es correcto. El argumento a favor de la estructura responde que las decisiones automatizadas, la investigación y la interoperabilidad necesitan valores estables.

Consenso operativo: conservar el texto original cuando tenga valor narrativo, pero derivar campos estructurados cuando el sistema necesite contar, filtrar, comparar o decidir.

### Desacuerdo 2: vocabulario local contra estándar externo

Pregunta: ¿un sistema pequeño debe empezar directamente con SNOMED CT, LOINC u otro estándar?

Un estándar externo aporta interoperabilidad, mantenimiento comunitario y semántica compartida. Pero también exige licencia, aprendizaje, selección correcta del código, versionamiento y mapeos cuidadosos. Un vocabulario local puede ser útil para aprender y prototipar, siempre que no se presente como estándar.

Consenso operativo: iniciar con vocabularios locales explícitos en ejemplos pedagógicos; migrar o mapear hacia estándares cuando el caso de uso requiera intercambio real, investigación multicéntrica o integración con sistemas clínicos.

### Desacuerdo 3: reglas simples contra procesamiento de lenguaje natural

Pregunta: ¿por qué no usar directamente modelos de lenguaje o NLP clínico?

Las reglas simples son transparentes, baratas y verificables, pero cubren poco. El procesamiento de lenguaje natural puede capturar patrones más ricos, pero introduce incertidumbre, entrenamiento, validación, sesgos y mantenimiento.

Consenso operativo: usar reglas explícitas para contratos mínimos y trazabilidad inicial. Cuando se incorpore NLP, sus salidas deben entrar como resultados probabilísticos o revisables, no como verdad semántica automática.

## Puente hacia la frontera

El problema textual crece rápido. En historias clínicas reales aparecen negación, temporalidad, experienciador, incertidumbre, abreviaturas, idioma, errores de dictado, plantillas, copias previas y jerga local. Una frase como "madre con antecedente de asma; paciente niega disnea actual" contiene al menos tres trampas: familiar, negación y tiempo.

Más adelante, este libro podrá conectar esta sección con expresiones regulares, modelos de lenguaje, extracción de entidades, ontologías, embeddings, recuperación semántica y sistemas de interoperabilidad como HL7 FHIR. Pero el principio seguirá siendo el mismo: texto libre, categoría y código no son lo mismo.

Los sistemas avanzados no eliminan la necesidad de contrato. La aumentan. Si una herramienta extrae conceptos desde texto clínico, el sistema debe saber qué vocabulario usa, qué versión, qué confianza tiene, qué fragmento textual respalda la extracción y qué debe hacer cuando hay duda.

## Evaluar si entendiste

1. ¿Por qué `str` no equivale a vocabulario clínico?
2. ¿Qué diferencia hay entre texto libre, texto normalizado, categoría controlada y código?
3. ¿Por qué buscar `"disnea"` dentro de una nota puede producir falsos positivos?
4. ¿Por qué `None` no debe transformarse en `ausente`?
5. ¿Qué gana un sistema al usar `Enum` para estados?
6. ¿Por qué conviene conservar el texto original aunque exista una versión normalizada?
7. ¿Qué riesgo aparece al usar códigos locales sin declarar sistema y versión?
8. ¿Por qué una expresión fuera de vocabulario debe ir a revisión en vez de clasificarse por intuición?
9. ¿Cuándo tendría sentido mapear un vocabulario local a LOINC o SNOMED CT?
10. ¿Qué parte de este ejemplo sería insuficiente para una historia clínica real?

## Vacíos de comprensión que debes vigilar

1. Creer que normalizar texto es entender texto. La normalización prepara comparación; no resuelve semántica.
2. Pensar que un código es solo una abreviatura. Un código pertenece a un sistema y una versión; sin eso pierde trazabilidad.
3. Tratar vocabularios controlados como listas decorativas. Su función es gobernar decisiones, rechazos y equivalencias.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma diez expresiones reales o inventadas para un mismo hallazgo y separa afirmación, negación, duda y no evaluado.
2. **Segunda hora:** crea un vocabulario controlado mínimo con `Enum` y una función de normalización.
3. **Tercera hora:** escribe pruebas para expresión conocida, negación, texto ausente, expresión desconocida y variación de mayúsculas/acentos.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *Text Sequence Type — str*. Python 3.14.6 documentation. <https://docs.python.org/3/library/stdtypes.html>
- Python Software Foundation. (2026). *enum — Support for enumerations*. Python 3.12 documentation. <https://docs.python.org/3.12/library/enum.html>
- Regenstrief Institute. (2026). *LOINC: Get started with LOINC*. <https://loinc.org/start>
- Regenstrief Institute. (2026). *About LOINC*. <https://loinc.org/about/>
- SNOMED International. (2026). *Introduction to SNOMED CT*. <https://docs.snomed.org/snomed-ct-specifications/snomed-ct-editorial-guide/readme/snomed-ct-introduction>

## Siguiente paso

El texto mostró que un dato puede tener más estados que una cadena permite ver. La próxima sección continuará con un caso todavía más traicionero: booleanos y estados. En Python, `True` y `False` parecen suficientes. En biomedicina, muchas decisiones necesitan distinguir presente, ausente, desconocido, no evaluado, contraindicado, pendiente o no aplicable.
