# Esquemas mínimos y validación formal

La sección anterior mostró una tabla como filas, columnas, limpieza y validación. Pero el validador quedó escrito como código disperso: columnas requeridas en un conjunto, reglas en un diccionario, estados en un `Enum` y funciones que conocen demasiado de una prueba específica.

Eso sirve para aprender. No basta para crecer.

Cuando una fuente biomédica empieza a acumular tablas, formularios, endpoints, cohortes, mediciones, eventos y versiones, aparece una pregunta inevitable: ¿dónde vive el contrato del dato? Si el contrato está repartido entre funciones, comentarios y supuestos personales, cada cálculo nuevo puede reinterpretar la misma columna de forma distinta.

Esta sección introduce una idea central: un esquema mínimo es una representación explícita, revisable y reutilizable del contrato de una estructura de datos. No reemplaza el juicio de dominio. Lo obliga a escribirse.

## Origen técnico: el esquema como contrato de entrada

Un esquema declara qué campos espera una estructura, qué tipo debe tener cada campo, qué valores son aceptables, qué unidades son compatibles y qué hacer cuando algo falta.

En Python se puede empezar con un diccionario.

```python
ESQUEMA_CREATININA = {
    "version": "creatinina.v1",
    "campos_requeridos": ["paciente_id", "valor", "unidad"],
    "campos": {
        "paciente_id": {"tipo": "texto", "requerido": True},
        "valor": {"tipo": "numero", "minimo": 0, "maximo": 25, "requerido": True},
        "unidad": {"tipo": "categoria", "permitidos": ["mg/dL"], "requerido": True},
    },
}

print(ESQUEMA_CREATININA["version"])
print(ESQUEMA_CREATININA["campos"]["valor"]["tipo"])
```

Salida esperada:

```text
creatinina.v1
numero
```

El esquema todavía no valida nada. Solo declara una promesa. Esa separación es importante: el contrato debe poder leerse antes de ejecutarse.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un esquema mínimo biomédico es una descripción explícita de los campos esperados, tipos, obligatoriedad, unidades, valores permitidos, límites y versión de una estructura de datos. La validación formal es el proceso de comparar datos reales contra ese esquema y producir una salida trazable: válido, inválido, razón y versión de regla aplicada.
</div>

Esta definición contiene cuatro piezas.

**Campos esperados.** Qué nombres deben aparecer para que la estructura sea interpretable.

**Tipos.** Qué forma técnica debe tener cada valor: texto, número, categoría, fecha, booleano, lista, registro.

**Reglas de dominio.** Qué unidades, rangos, códigos o estados son aceptables.

**Versión.** Qué variante del contrato produjo la aceptación o el rechazo.

Sin versión, una validación puede ser correcta hoy e irreconstruible mañana.

## Versión ingenua: validar con condiciones sueltas

Supongamos una medición de creatinina.

```python
fila = {"paciente_id": "P001", "valor": "1.8", "unidad": "mg/dL"}

if "paciente_id" in fila and "valor" in fila and "unidad" in fila:
    valor = float(fila["valor"])
    if fila["unidad"] == "mg/dL" and valor >= 0 and valor <= 25:
        print("fila valida")
```

Salida esperada:

```text
fila valida
```

La fila pasa. El código parece razonable. Pero el contrato está escondido dentro del `if`.

¿Qué pasa si otra función usa `valor <= 20`? ¿Qué pasa si otra acepta `umol/L` y convierte? ¿Qué pasa si un archivo trae `paciente` en vez de `paciente_id`? ¿Qué versión de la regla produjo el resultado?

La validación funciona, pero no es gobernable.

## Crítica técnica: qué está mal

Primero, la regla no tiene nombre. Si falla, no sabemos si falló por tipo, unidad, rango o campo ausente.

Segundo, el esquema no puede inspeccionarse. Está mezclado con la ejecución.

Tercero, no hay versión de regla. Un resultado antiguo no puede reconstruirse si el rango cambia.

Cuarto, no hay salida estructurada. El programa imprime `"fila valida"` o no imprime nada. Eso no alcanza para auditar.

Quinto, la conversión ocurre antes de declarar qué se espera. Si `valor` viene vacío, la excepción técnica interrumpe el flujo antes de que el dominio produzca una razón útil.

Sexto, la función solo sirve para creatinina. Si mañana validamos hemoglobina, sodio o presión arterial, copiaremos y pegaremos condiciones.

## Esquema mínimo como dato

Un primer avance es tratar el esquema como dato. Esto permite imprimirlo, versionarlo, probarlo y reutilizarlo.

```python
from enum import Enum


class EstadoValidacion(Enum):
    VALIDO = "valido"
    FALTA_CAMPO = "falta_campo"
    TIPO_INVALIDO = "tipo_invalido"
    VALOR_AUSENTE = "valor_ausente"
    VALOR_NO_PERMITIDO = "valor_no_permitido"
    FUERA_DE_RANGO = "fuera_de_rango"


MARCADORES_AUSENCIA = {"", "NA", "N/A", "PENDIENTE", "SIN DATO"}


ESQUEMA_CREATININA = {
    "nombre": "medicion_creatinina",
    "version": "1.0.0",
    "campos": {
        "paciente_id": {
            "tipo": "texto",
            "requerido": True,
        },
        "valor": {
            "tipo": "numero",
            "requerido": True,
            "minimo": 0,
            "maximo": 25,
        },
        "unidad": {
            "tipo": "categoria",
            "requerido": True,
            "permitidos": ["mg/dL"],
        },
    },
}
```

El esquema no es una herramienta clínica. Es una miniatura pedagógica. Declara que, para este cálculo, aceptaremos creatinina en `mg/dL` y valores entre 0 y 25.

## Validador genérico pequeño

Ahora escribimos una función que no conozca creatinina. Solo conoce el esquema.

```python
def normalizar_texto(valor):
    """Convierte una celda cruda en texto comparable."""
    if valor is None:
        return ""
    return str(valor).strip()


def validar_tipo_texto(valor):
    texto = normalizar_texto(valor)
    if texto.upper() in MARCADORES_AUSENCIA:
        return EstadoValidacion.VALOR_AUSENTE, None
    return EstadoValidacion.VALIDO, texto


def validar_tipo_numero(valor):
    texto = normalizar_texto(valor)
    if texto.upper() in MARCADORES_AUSENCIA:
        return EstadoValidacion.VALOR_AUSENTE, None

    texto = texto.replace(",", ".")
    try:
        numero = float(texto)
    except ValueError:
        return EstadoValidacion.TIPO_INVALIDO, None

    return EstadoValidacion.VALIDO, numero


def validar_campo(nombre, valor, regla):
    """Valida un campo individual contra una regla declarativa."""
    tipo = regla["tipo"]

    if tipo == "texto":
        estado, valor_limpio = validar_tipo_texto(valor)
    elif tipo == "numero":
        estado, valor_limpio = validar_tipo_numero(valor)
    elif tipo == "categoria":
        estado, valor_limpio = validar_tipo_texto(valor)
    else:
        return {
            "campo": nombre,
            "estado": EstadoValidacion.TIPO_INVALIDO,
            "valor": None,
            "razon": f"tipo_no_soportado:{tipo}",
        }

    if estado != EstadoValidacion.VALIDO:
        return {
            "campo": nombre,
            "estado": estado,
            "valor": None,
            "razon": estado.value,
        }

    if tipo == "categoria" and valor_limpio not in regla.get("permitidos", []):
        return {
            "campo": nombre,
            "estado": EstadoValidacion.VALOR_NO_PERMITIDO,
            "valor": valor_limpio,
            "razon": "categoria_fuera_de_lista",
        }

    if tipo == "numero":
        minimo = regla.get("minimo")
        maximo = regla.get("maximo")
        if minimo is not None and valor_limpio < minimo:
            return {
                "campo": nombre,
                "estado": EstadoValidacion.FUERA_DE_RANGO,
                "valor": valor_limpio,
                "razon": "menor_que_minimo",
            }
        if maximo is not None and valor_limpio > maximo:
            return {
                "campo": nombre,
                "estado": EstadoValidacion.FUERA_DE_RANGO,
                "valor": valor_limpio,
                "razon": "mayor_que_maximo",
            }

    return {
        "campo": nombre,
        "estado": EstadoValidacion.VALIDO,
        "valor": valor_limpio,
        "razon": "cumple_regla",
    }
```

Este código tiene más líneas que el `if` ingenuo. Pero ahora las reglas viven en el esquema y el validador puede aplicarse a campos distintos.

## Validar un registro completo

El siguiente paso es recorrer todos los campos declarados por el esquema.

```python
def validar_registro(registro, esquema):
    """Valida un registro completo y conserva valores limpios y errores."""
    valores_limpios = {}
    errores = []

    for nombre, regla in esquema["campos"].items():
        requerido = regla.get("requerido", False)

        if nombre not in registro:
            if requerido:
                errores.append({
                    "campo": nombre,
                    "estado": EstadoValidacion.FALTA_CAMPO,
                    "razon": "campo_requerido_ausente",
                })
            continue

        resultado = validar_campo(nombre, registro[nombre], regla)
        if resultado["estado"] == EstadoValidacion.VALIDO:
            valores_limpios[nombre] = resultado["valor"]
        else:
            errores.append(resultado)

    estado_general = EstadoValidacion.VALIDO if not errores else EstadoValidacion.TIPO_INVALIDO

    return {
        "estado": estado_general,
        "valido": not errores,
        "valores": valores_limpios,
        "errores": errores,
        "esquema": esquema["nombre"],
        "version": esquema["version"],
    }
```

Probemos tres registros.

```python
registros = [
    {"paciente_id": "P001", "valor": "1.8", "unidad": "mg/dL"},
    {"paciente_id": "P002", "valor": "160", "unidad": "umol/L"},
    {"paciente_id": "P003", "valor": "", "unidad": "mg/dL"},
]

for registro in registros:
    resultado = validar_registro(registro, ESQUEMA_CREATININA)
    print(resultado["valido"], resultado["version"], resultado["errores"])
```

Salida esperada:

```text
True 1.0.0 []
False 1.0.0 [{'campo': 'valor', 'estado': <EstadoValidacion.FUERA_DE_RANGO: 'fuera_de_rango'>, 'valor': 160.0, 'razon': 'mayor_que_maximo'}, {'campo': 'unidad', 'estado': <EstadoValidacion.VALOR_NO_PERMITIDO: 'valor_no_permitido'>, 'valor': 'umol/L', 'razon': 'categoria_fuera_de_lista'}]
False 1.0.0 [{'campo': 'valor', 'estado': <EstadoValidacion.VALOR_AUSENTE: 'valor_ausente'>, 'valor': None, 'razon': 'valor_ausente'}]
```

La segunda fila falla por dos razones: el valor está fuera del rango pedagógico y la unidad no está permitida. La tercera falla por ausencia. Ninguna falla se pierde.

## Darle forma al esquema con `dataclass`

Un diccionario es flexible, pero puede volverse frágil. Es fácil escribir `"minimoo"` en vez de `"minimo"` y no enterarse hasta tarde. Una forma intermedia es usar `dataclass` para representar reglas.

```python
from dataclasses import dataclass, field


@dataclass
class ReglaCampo:
    nombre: str
    tipo: str
    requerido: bool = True
    permitidos: list[str] = field(default_factory=list)
    minimo: float | None = None
    maximo: float | None = None


@dataclass
class Esquema:
    nombre: str
    version: str
    campos: list[ReglaCampo]


esquema_sodio = Esquema(
    nombre="medicion_sodio",
    version="1.0.0",
    campos=[
        ReglaCampo(nombre="paciente_id", tipo="texto"),
        ReglaCampo(nombre="valor", tipo="numero", minimo=80, maximo=200),
        ReglaCampo(nombre="unidad", tipo="categoria", permitidos=["mmol/L"]),
    ],
)

print(esquema_sodio.nombre)
print(esquema_sodio.campos[1].minimo)
```

Salida esperada:

```text
medicion_sodio
80
```

`dataclass` no valida automáticamente el dominio. Su aporte aquí es estructural: reduce errores de forma, hace visibles los campos del contrato y permite crear objetos legibles para reglas.

## Convertir esquema estructurado a esquema ejecutable

Podemos adaptar el validador anterior para aceptar un `Esquema`.

```python
def esquema_a_diccionario(esquema):
    """Convierte un Esquema tipado a la forma usada por el validador."""
    campos = {}
    for campo in esquema.campos:
        regla = {
            "tipo": campo.tipo,
            "requerido": campo.requerido,
        }
        if campo.permitidos:
            regla["permitidos"] = campo.permitidos
        if campo.minimo is not None:
            regla["minimo"] = campo.minimo
        if campo.maximo is not None:
            regla["maximo"] = campo.maximo
        campos[campo.nombre] = regla

    return {
        "nombre": esquema.nombre,
        "version": esquema.version,
        "campos": campos,
    }


registro_sodio = {"paciente_id": "P010", "valor": "138", "unidad": "mmol/L"}
resultado = validar_registro(registro_sodio, esquema_a_diccionario(esquema_sodio))

print(resultado["valido"])
print(resultado["valores"]["valor"])
```

Salida esperada:

```text
True
138.0
```

Esta capa no es necesaria para todos los proyectos. Es útil cuando el esquema empieza a repetirse y conviene evitar diccionarios escritos a mano en cada función.

## Validación formal no significa rigidez ciega

Un esquema puede ser rígido en lo técnico y humilde en lo clínico. Debe rechazar lo que no puede interpretar, pero también debe declarar cuándo su alcance es pequeño.

Por ejemplo, este esquema de sodio no diagnostica hiponatremia ni hipernatremia. Solo declara que una medición de sodio para esta operación debe tener:

- `paciente_id`;
- `valor` numérico;
- `unidad` `mmol/L`;
- rango pedagógico entre 80 y 200.

El esquema valida entrada. No produce conducta médica.

Ese límite protege el libro. También protege al programador. La formalización no debe inflar el alcance de una miniatura.

## CODE CLEAN: datos, reglas y ejecución en capas

Una versión frágil mezcla regla y ejecución.

```python
if fila["unidad"] == "mg/dL" and float(fila["valor"]) <= 25:
    calcular(fila)
```

Una versión más limpia separa tres capas.

```python
esquema = ESQUEMA_CREATININA
validacion = validar_registro(fila, esquema)

if validacion["valido"]:
    calcular(validacion["valores"])
else:
    registrar_rechazo(validacion["errores"])
```

La separación importa porque cada capa responde una pregunta distinta.

**Esquema:** ¿qué contrato exige esta operación?

**Validación:** ¿este registro cumple el contrato?

**Cálculo:** ¿qué hacemos con registros ya validados?

Si una función intenta responder las tres preguntas al mismo tiempo, será más difícil probarla, auditarla y modificarla.

## Pruebas mínimas

```python
# Propiedad 1: una fila válida conserva versión del esquema.
resultado = validar_registro(
    {"paciente_id": "P001", "valor": "1.8", "unidad": "mg/dL"},
    ESQUEMA_CREATININA,
)
assert resultado["valido"] is True
assert resultado["version"] == "1.0.0"

# Propiedad 2: una unidad no permitida se rechaza con razón explícita.
resultado = validar_registro(
    {"paciente_id": "P002", "valor": "1.8", "unidad": "umol/L"},
    ESQUEMA_CREATININA,
)
assert resultado["valido"] is False
assert any(error["razon"] == "categoria_fuera_de_lista" for error in resultado["errores"])

# Propiedad 3: un valor ausente no se convierte a cero.
resultado = validar_registro(
    {"paciente_id": "P003", "valor": "", "unidad": "mg/dL"},
    ESQUEMA_CREATININA,
)
assert resultado["valido"] is False
assert resultado["errores"][0]["estado"] == EstadoValidacion.VALOR_AUSENTE

# Propiedad 4: un esquema estructurado puede convertirse y validar.
resultado = validar_registro(
    {"paciente_id": "P010", "valor": "138", "unidad": "mmol/L"},
    esquema_a_diccionario(esquema_sodio),
)
assert resultado["valido"] is True
assert resultado["valores"]["valor"] == 138.0
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas no garantizan que el esquema sea clínicamente completo. Garantizan que el contrato mínimo se aplica de forma estable.

## De esquema local a estándares externos

La miniatura anterior usa diccionarios y `dataclass` porque el objetivo es aprender el razonamiento. En proyectos reales pueden aparecer herramientas más formales.

**JSON Schema** permite describir estructuras JSON con tipos, campos requeridos y restricciones. Es útil cuando se comparten datos entre sistemas, APIs o archivos.

**Pydantic** permite declarar modelos de datos en Python y validar entradas contra esos modelos. Es útil cuando una aplicación necesita convertir datos crudos en objetos con tipos y reglas.

**DataFrames con validación externa** permiten trabajar con tablas grandes, pero no eliminan la necesidad de declarar columnas, tipos, unidades y reglas.

La herramienta concreta importa menos que la frontera conceptual: no basta con tener datos en memoria; hay que saber qué contrato cumplen.

## Esquema mínimo para una tabla biomédica

Antes de calcular sobre una tabla, el esquema debería responder al menos:

1. ¿Cuál es la unidad de observación de cada fila?
2. ¿Qué columnas son obligatorias?
3. ¿Qué columnas son opcionales?
4. ¿Qué tipo técnico tiene cada columna?
5. ¿Qué unidad usa cada medición?
6. ¿Qué valores categóricos están permitidos?
7. ¿Qué marcadores de ausencia se reconocen?
8. ¿Qué rangos son imposibles, no calculables o solo fuera de referencia?
9. ¿Qué versión de regla está activa?
10. ¿Qué salida se produce para filas inválidas?

Si una tabla no puede responder estas preguntas, todavía puede explorarse. Pero no debería alimentar una conclusión.

## Límites y errores frecuentes

1. **Confundir anotaciones de tipo con validación.** Escribir `valor: float` comunica intención, pero no convierte ni valida por sí solo una celda cruda.
2. **Creer que un esquema valida la verdad clínica.** Un esquema valida contrato de datos; la verdad clínica exige contexto, fuente, población y evidencia.
3. **No versionar reglas.** Sin versión, no se puede reconstruir por qué una fila fue aceptada.
4. **Aceptar unidades múltiples sin regla de conversión.** Si se aceptan varias unidades, debe existir conversión trazable.
5. **Mezclar validación con cálculo.** El cálculo debe recibir datos ya interpretables.
6. **Convertir todo lo que se pueda convertir.** Que `float()` acepte un texto no significa que el dato sea válido para el dominio.
7. **Usar rangos pedagógicos como rangos clínicos.** En este libro los rangos de ejemplo enseñan estructura, no reemplazan referencia clínica.
8. **No conservar errores.** Una validación que solo devuelve `False` no permite corregir la fuente.
9. **Repetir esquemas a mano.** Duplica divergencias.
10. **Validar demasiado tarde.** Si el error aparece después del cálculo, ya contaminó el flujo.

## Argumentos críticos

### Desacuerdo 1: esquema mínimo contra velocidad

Pregunta: ¿no es más lento escribir un esquema?

Al principio sí. Pero cuando el mismo dato entra a varios cálculos, el esquema ahorra ambigüedad. La velocidad inicial de un `if` suelto puede convertirse en deuda técnica y conceptual.

Consenso operativo: escribir esquema mínimo cuando el dato vaya a reutilizarse, compartirse o sustentar una salida.

### Desacuerdo 2: validación formal contra exploración libre

Pregunta: ¿un esquema no limita descubrir patrones inesperados?

La exploración puede empezar sin esquema completo. Pero la conclusión debe pasar por contrato. Explorar es conocer la fuente; validar es decidir qué parte de la fuente puede sostener una operación.

Consenso operativo: explorar primero si la fuente es desconocida; formalizar antes de calcular resultados defendibles.

### Desacuerdo 3: rangos estrictos contra casos raros

Pregunta: ¿un valor fuera de rango no puede ser real?

Puede serlo. Por eso el estado no debe llamarse automáticamente `falso`. Puede ser `fuera_de_rango`, `requiere_revision` o `no_calculable`. La clave es no dejarlo entrar silenciosamente al cálculo ordinario.

Consenso operativo: separar imposible técnico, raro clínico, error de captura y caso que requiere revisión.

### Desacuerdo 4: herramienta externa contra comprensión propia

Pregunta: ¿por qué no usar directamente una librería de validación?

Las librerías son útiles. Pero si el programador no entiende qué valida, cuándo convierte, cómo reporta errores y qué no cubre, la herramienta solo traslada la opacidad a otra capa.

Consenso operativo: aprender el contrato a mano en miniatura; luego usar herramientas formales con criterio.

## Puente hacia la frontera

Los esquemas son la antesala de bases de datos, APIs, modelos de datos clínicos, interoperabilidad, validación de cohortes y análisis reproducibles. Un sistema que registra pacientes, eventos, laboratorios o variantes genéticas necesita algo más que columnas visibles: necesita contratos.

En bases SQL, el esquema aparece como tablas, tipos, claves y restricciones. En APIs, aparece como modelos de entrada y salida. En ciencia de datos, aparece como diccionarios de datos, reglas de limpieza, validación de datasets y reportes de calidad. En bioinformática, aparece como formatos, campos obligatorios, versiones y convenciones de referencia.

El principio es el mismo: si no puedes declarar el contrato de tus datos, no puedes defender el cálculo que haces sobre ellos.

## Evaluar si entendiste

1. ¿Qué diferencia hay entre validar con un `if` suelto y validar contra un esquema?
2. ¿Por qué un esquema mínimo debe tener versión?
3. ¿Qué parte del contrato pertenece al tipo técnico y qué parte pertenece al dominio?
4. ¿Por qué `valor: float` no basta para validar una medición cruda?
5. ¿Qué salida debería producir una validación responsable cuando un campo falta?
6. ¿Cuándo conviene rechazar una unidad en vez de convertirla?
7. ¿Por qué una validación que solo devuelve `True` o `False` es insuficiente?
8. ¿Qué ventajas aporta representar reglas con `dataclass`?
9. ¿Qué riesgo aparece al usar rangos pedagógicos como si fueran clínicos?
10. ¿Qué tendría que cambiar para que el esquema de creatinina acepte `umol/L` de forma responsable?

## Vacíos de comprensión que debes vigilar

1. Pensar que un esquema es burocracia. En realidad es una forma de hacer visible el criterio.
2. Confundir corrección técnica con suficiencia biomédica. Un dato puede cumplir tipo y aun así ser insuficiente para decidir.
3. Creer que la herramienta de validación reemplaza la definición del dominio. La herramienta ejecuta reglas; no decide qué reglas son correctas.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** escribe un esquema mínimo para una medición distinta: hemoglobina, sodio, presión arterial o peso.
2. **Segunda hora:** crea cinco registros: válido, unidad incompatible, valor ausente, campo faltante y valor fuera de rango.
3. **Tercera hora:** adapta `validar_registro` para tu esquema y escribe cuatro pruebas con `assert`.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *dataclasses: Data Classes*. Python 3 documentation. <https://docs.python.org/3/library/dataclasses.html>
- Python Software Foundation. (2026). *typing: Support for type hints*. Python 3 documentation. <https://docs.python.org/3/library/typing.html>
- JSON Schema. (2026). *Creating your first schema*. <https://json-schema.org/learn/getting-started-step-by-step>
- Pydantic. (2026). *Pydantic Validation*. <https://pydantic.dev/docs/validation/latest/get-started/>
- Wickham, H., & Grolemund, G. (2017). *R for Data Science*. O'Reilly Media.

## Siguiente paso

Los esquemas mínimos convierten reglas dispersas en contratos explícitos. La siguiente sección avanzará hacia una introducción controlada a `pandas`: cómo usar una herramienta tabular potente sin olvidar que un `DataFrame` solo es confiable cuando sus columnas ya tienen significado, contrato y límites.
