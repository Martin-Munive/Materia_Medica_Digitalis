# Errores, excepciones y seguridad del cálculo

Un programa puede ejecutar sin fallar y aun así producir un resultado inseguro. Esa idea no es derrotista; es la base de cualquier sistema que toma decisiones sobre personas, datos clínicos o resultados de laboratorio. Si un algoritmo confunde "el código corrió" con "el resultado es correcto", está confundiendo dos cosas que la medicina, la bioinformática y la ciencia de datos han aprendido a separar hace tiempo.

En las secciones anteriores aparecieron errores, datos faltantes, excepciones y trazabilidad. Aquí esos temas se vuelven arquitectura. La pregunta ya no es cómo capturar un error cuando ocurre, sino cómo diseñar el procedimiento de modo que el error tenga un lugar previsto, una razón explícita y un efecto limitado.

Esta sección cierra la primera unidad de Python como instrumento de cálculo. Si las funciones encapsulan criterio y los bucles controlan procesos repetidos, las excepciones y el manejo de errores enseñan algo distinto: cómo gobernar lo que un procedimiento no puede evitar.

## Origen técnico: pensar el error como objeto, no como accidente

La historia del manejo de errores en software es corta pero densa. Los lenguajes tempranos solían tratar los errores como accidentes terminales: si algo fallaba, el programa se detenía. La computación moderna aprendió que esa postura no escala. Un sistema clínico, un pipeline de genómica o una rutina de análisis estadístico no pueden colapsar por una fila mal formateada en una tabla.

El modelo que se consolidó en lenguajes como Python se llama manejo de excepciones. Una excepción no es un fallo del programa; es una señal de que algo que el programador previó como posible acaba de ocurrir. Esa distinción es filosófica y operativa a la vez. Si el programador pensó en la condición, puede decidir qué hacer. Si no la pensó, el sistema la trata como inesperada y, según el lenguaje, puede propagarla o detener todo.

En medicina, este paralelismo es directo. Una excepción técnica es equivalente a un resultado fuera de rango en un laboratorio. Una excepción del dominio es equivalente a un paciente fuera de la población para la cual se diseñó un protocolo. Ambas merecen una respuesta explícita, no un silencio.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un error es una condición que impide que un procedimiento produzca una salida confiable. Una excepción es la representación explícita de esa condición como un objeto que el programa puede capturar, inspeccionar y responder. La seguridad del cálculo es el conjunto de prácticas que limitan la propagación del error, lo hacen trazable y evitan que se confunda con una salida válida.
</div>

La definición tiene tres consecuencias.

**Un error no siempre debe detener el programa.** Algunos errores detienen el flujo. Otros se registran, se descartan o se traducen en una salida degradada. La elección depende del riesgo.

**Una excepción no es un mensaje genérico.** Debe contener suficiente información para que el código que la captura sepa qué hacer. Una excepción sin contexto es ruido.

**La seguridad del cálculo no es lo mismo que la corrección sintáctica.** Un programa puede tener su sintaxis correcta, sus tipos correctos y sus pruebas pasando, y aun así ser inseguro. La seguridad es una propiedad del sistema completo, no de una función aislada.

## Versión ingenua: el cálculo que parece funcionar

Supongamos que queremos calcular el índice de masa corporal a partir de peso y estatura, y clasificar el resultado en una categoría pedagógica.

```python
# Entradas del usuario, tomadas como texto desde una interfaz.
peso_kg = float(input("Peso en kg: "))
estatura_m = float(input("Estatura en metros: "))

# Cálculo directo.
imc = peso_kg / estatura_m ** 2

# Clasificación con umbrales pedagógicos.
if imc < 18.5:
    categoria = "bajo_peso"
elif imc < 25:
    categoria = "normal"
elif imc < 30:
    categoria = "sobrepeso"
else:
    categoria = "obesidad"

print("IMC:", round(imc, 2))
print("Categoría:", categoria)
```

Salida esperada, con entradas `70` y `1.75`:

```text
IMC: 22.86
Categoría: normal
```

El programa ejecuta, entrega una cifra, entrega una categoría. Si las entradas son razonables, no hay error visible. Esa es exactamente la fragilidad. El programa no sabe qué hacer si el usuario escribe letras en lugar de números, si la estatura es cero, si el peso es negativo o si la categoría sale del contrato pedagógico. Cada uno de esos casos producirá una falla abrupta o, peor, una salida numérica absurda que el programa presentará con la misma confianza que un resultado legítimo.

## Crítica técnica: qué está mal

La versión anterior falla por cuatro razones.

Primero, no diferencia entre entradas que el usuario no pudo proporcionar y entradas inválidas. Una estatura de `0` y una estatura de `cero` son cosas distintas. La primera es una cifra imposible; la segunda es un error de captura. Tratarlas igual destruye información.

Segundo, divide por una cantidad que puede ser cero. La línea `peso_kg / estatura_m ** 2` lanza una excepción no controlada si la estatura es cero. En un cuaderno de exploración es aceptable; en un sistema que interactúa con personas, no.

Tercero, las categorías no incluyen estados como `dato_invalido` o `fuera_de_poblacion_pedagogica`. La rama final `else` recibe tanto el sobrepeso como la obesidad, y nada obliga al programa a distinguir lo que el clínico sí distinguiría.

Cuarto, la salida numérica no tiene trazabilidad. El programa no registra qué versión de la fórmula usó, qué umbrales aplicó ni bajo qué suposiciones construyó la categoría. Si el criterio cambia, nadie puede saber qué respuestas antiguas son comparables con las nuevas.

CODE CLEAN no consiste en envolver el código en `try` y `except`. Consiste en diseñar el procedimiento de modo que el error tenga un lugar previsto y la respuesta sea defendible.

## Versión mejorada: validación, excepciones explícitas y trazabilidad

Una versión responsable separa captura, validación, cálculo, clasificación y registro.

```python
def convertir_a_float(texto, nombre_campo):
    """Convierte texto a float o lanza una excepción con contexto."""
    try:
        return float(texto)
    except (TypeError, ValueError):
        raise ValueError(f"{nombre_campo}_no_numerico") from None


def calcular_imc(peso_kg, estatura_m):
    """Calcula el IMC con validación de entradas y dominio pedagógico."""
    if peso_kg is None or estatura_m is None:
        raise ValueError("dato_faltante")
    if peso_kg <= 0:
        raise ValueError("peso_fuera_de_rango")
    if estatura_m <= 0 or estatura_m > 2.5:
        raise ValueError("estatura_fuera_de_rango")
    return peso_kg / estatura_m ** 2


def clasificar_imc_pedagogico(imc):
    """Clasifica un IMC en una categoría pedagógica."""
    if imc < 18.5:
        return "bajo_peso"
    if imc < 25:
        return "normal"
    if imc < 30:
        return "sobrepeso"
    return "obesidad"


def evaluar_peso_y_estatura(peso_texto, estatura_texto, version_regla="imc_pedagogico_v0"):
    """Punto de entrada: convierte, valida, calcula, clasifica y registra."""
    try:
        peso_kg = convertir_a_float(peso_texto, "peso")
        estatura_m = convertir_a_float(estatura_texto, "estatura")
        imc = calcular_imc(peso_kg, estatura_m)
        categoria = clasificar_imc_pedagogico(imc)
        return {
            "estado": "calculado",
            "imc": round(imc, 2),
            "categoria": categoria,
            "version_regla": version_regla,
        }
    except ValueError as error:
        return {
            "estado": "no_calculable",
            "razon": str(error),
            "version_regla": version_regla,
        }


resultado = evaluar_peso_y_estatura("70", "1.75")
print(resultado)
```

Salida esperada:

```text
{'estado': 'calculado', 'imc': 22.86, 'categoria': 'normal', 'version_regla': 'imc_pedagogico_v0'}
```

La estructura cambió en cinco puntos. La conversión de texto a número produce una excepción con nombre de campo. La validación de rango se hace en una función dedicada, no dispersa en condicionales. El cálculo y la clasificación están en funciones separadas con responsabilidad clara. El punto de entrada captura errores, los traduce en un estado explícito y conserva la versión de la regla. La salida nunca es un número suelto; siempre es un registro.

## Anatomía del manejo de errores

Una pieza vale la pena mirarla con cuidado.

```python
def convertir_a_float(texto, nombre_campo):
    try:
        return float(texto)
    except (TypeError, ValueError):
        raise ValueError(f"{nombre_campo}_no_numerico") from None
```

La función intenta la conversión. Si falla, captura solo las excepciones que sabe manejar. Luego lanza una excepción nueva con un mensaje que incluye el nombre del campo, lo que permite al código que la captura distinguir entre peso y estatura sin reinterpretar la causa original. La cláusula `from None` evita arrastrar la traza de la excepción previa, que ensucia la lectura sin aportar información útil al receptor.

Esta disciplina tiene un nombre implícito: traducir excepciones. La idea es que las excepciones técnicas (`TypeError`, `ValueError`) son detalles de implementación. El código de dominio debe operar con excepciones que expresen su propio vocabulario: `dato_faltante`, `peso_fuera_de_rango`, `estatura_no_numerico`.

## Captura específica y captura genérica

Python permite capturar varias excepciones a la vez y dejar que el resto se propague. Esa jerarquía es importante.

```python
def dividir_clasificacion(numerador, denominador):
    if denominador is None:
        return {"estado": "no_calculable", "razon": "denominador_faltante"}
    try:
        resultado = numerador / denominador
    except ZeroDivisionError:
        return {"estado": "no_calculable", "razon": "division_por_cero"}
    except TypeError:
        return {"estado": "no_calculable", "razon": "tipo_incompatible"}
    return {"estado": "calculado", "resultado": resultado}
```

Salida esperada, con `dividir_clasificacion(10, 0)`:

```text
{'estado': 'no_calculable', 'razon': 'division_por_cero'}
```

La función captura excepciones específicas. No atrapa `Exception` en general. Una captura amplia convierte el procedimiento en una caja negra: cualquier error, venga de donde venga, se convierte en la misma respuesta. Eso destruye información que el código superior podía usar para decidir.

La regla práctica es esta: capturar solo lo que se sabe responder. Si una excepción cae fuera del contrato de la función, propagarla es más honesto que traducirla.

## Excepciones como contrato

Una excepción puede ser parte del contrato de una función, no solo un error inesperado. Si una función espera una edad entre 0 y 130 años, la excepción `EdadFueraDeRango` no es un accidente: es la forma en que la función dice "este caso no me corresponde".

```python
class EdadFueraDeRango(ValueError):
    """Excepción para edades fuera del contrato pedagógico."""


def clasificar_edad_pedagogica(edad_anios):
    """Clasifica edad en grupo pedagógico; lanza si está fuera de rango."""
    if edad_anios is None:
        raise ValueError("edad_faltante")
    if edad_anios < 0 or edad_anios > 130:
        raise EdadFueraDeRango(f"edad={edad_anios}")
    if edad_anios < 18:
        return "pediatrica"
    if edad_anios < 65:
        return "adulta"
    return "adulta_mayor"


try:
    grupo = clasificar_edad_pedagogica(150)
except EdadFueraDeRango as error:
    grupo = "fuera_de_rango"
    motivo = str(error)
else:
    motivo = "edad_dentro_de_contrato"

print(grupo, motivo)
```

Salida esperada:

```text
fuera_de_rango edad=150
```

Definir una clase de excepción propia cuesta poco y comunica mucho. La función que llama sabe que esa excepción existe y puede decidir qué hacer con ella sin importar la causa interna. Si más adelante la regla cambia y la edad límite sube, las funciones que usaban `EdadFueraDeRango` siguen funcionando; el contrato sigue siendo explícito.

## Prueba mínima del manejo de errores

Un manejo de errores que no se prueba es un manejo de errores que falla en producción. La prueba mínima cubre tres casos: entrada válida, entrada malformada y entrada fuera de rango.

```python
def clasificar_edad_pedagogica(edad_anios):
    if edad_anios is None:
        raise ValueError("edad_faltante")
    if edad_anios < 0 or edad_anios > 130:
        raise EdadFueraDeRango(f"edad={edad_anios}")
    if edad_anios < 18:
        return "pediatrica"
    if edad_anios < 65:
        return "adulta"
    return "adulta_mayor"


assert clasificar_edad_pedagogica(10) == "pediatrica"
assert clasificar_edad_pedagogica(40) == "adulta"
assert clasificar_edad_pedagogica(70) == "adulta_mayor"

try:
    clasificar_edad_pedagogica(200)
except EdadFueraDeRango:
    paso = True
else:
    paso = False
assert paso
```

Salida esperada: no imprime nada si las pruebas pasan.

La primera parte valida la lógica. La segunda valida que la excepción se lanza cuando corresponde. Sin esa segunda parte, una refactorización podría reemplazar la excepción por un `return "fuera_de_rango"` silencioso y el cambio pasaría desapercibido en producción.

## Argumentos críticos

### Desacuerdo 1: capturar amplio contra capturar específico

Pregunta: ¿conviene capturar todas las excepciones con un solo `except Exception` o ser específico?

El argumento por la captura amplia es la simplicidad: una sola línea protege contra cualquier error. El argumento por la captura específica es la trazabilidad: cada tipo de error merece una respuesta documentada.

En dominios biomédicos, la captura amplia es peligrosa. Convierte errores técnicos en errores de dominio sin que el lector lo sepa. Una `KeyError` por un campo faltante y una `ValueError` por un dato fuera de rango pueden terminar apareciendo como el mismo estado en el registro, y eso destruye la auditoría.

Consenso operativo: capturar lo específico que se sabe responder. Dejar propagar lo demás. Si una excepción inesperada se vuelve frecuente, vale la pena capturarla con un nombre explícito.

### Desacuerdo 2: fallar rápido contra degradar con cuidado

Pregunta: ¿un procedimiento debe detenerse al primer error o intentar una salida degradada?

Fallar rápido protege contra decisiones construidas sobre datos corruptos. Degradar con cuidado permite que una parte del sistema siga operando cuando otra falla. La elección depende de qué es peor: una salida parcial incorrecta o una ausencia de salida.

Consenso operativo: si la salida parcial puede guiar una acción de alto impacto, fallar. Si la salida parcial solo informa, degradar con registro explícito puede ser razonable.

### Desacuerdo 3: mensajes genéricos contra mensajes específicos

Pregunta: ¿qué tan específico debe ser el mensaje de una excepción?

Un mensaje genérico (`"dato inválido"`) es fácil de escribir. Un mensaje específico (`"estatura_fuera_de_rango: valor=0"`) requiere que el programador piense en el contexto.

Consenso operativo: el mensaje es parte de la trazabilidad. Un mensaje específico acelera la depuración, facilita la auditoría y obliga al programador a explicitar qué no estaba bien. La especificidad no cuesta mucho y paga pronto.

## Puente hacia la frontera

El manejo de errores separa los sistemas serios de los frágiles. En genómica, una lectura de baja calidad debe marcarse como tal, no procesarse como si fuera confiable. En pipelines de datos clínicos, una fila mal formateada no debe detener todo el lote; debe aislarse y registrarse. En modelos de soporte a la decisión, una inferencia con confianza insuficiente no debe presentarse como recomendación; debe reportarse como `requiere_revision`.

La pregunta madura no es "¿cómo evito que el programa falle?". Es:

```text
¿Qué errores son posibles, qué debe hacer el sistema cuando aparecen y cómo se registra lo que se decidió?
```

Diseñar el error es diseñar el sistema.

## Evaluar si entendiste

1. ¿Por qué un programa que ejecuta sin fallar puede aun así producir un resultado inseguro?
2. ¿Qué diferencia hay entre una excepción técnica y una excepción del dominio?
3. ¿Cuándo conviene traducir una excepción técnica en una excepción con vocabulario de dominio?
4. ¿Por qué una captura amplia con `except Exception` puede ser peligrosa en sistemas biomédicos?
5. ¿Qué ventajas tiene definir una clase de excepción propia para el contrato de una función?
6. ¿Qué elementos mínimos debería contener un registro de error para ser auditable?
7. ¿Cómo distinguirías una salida parcial correcta de una salida parcial que induce a error?
8. ¿Por qué una prueba que solo valida el camino feliz no es suficiente para un sistema clínico?
9. ¿Qué versión de regla debería registrar un sistema cada vez que produce una salida?
10. ¿Cómo se conecta el manejo de errores con la trazabilidad, la validación y la seguridad del cálculo?

## Vacíos de comprensión que debes vigilar

1. Creer que `try` y `except` resuelven el problema del error. El problema del error es de diseño; el manejo es solo una pieza.
2. Capturar excepciones por reflejo. Cada captura debe justificarse.
3. Devolver `None` como forma de manejar un error. Un error no es un valor; es un estado con información.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma una función del libro que reciba datos y agrega validación de rango con excepciones específicas.
2. **Segunda hora:** escribe tres `assert` que cubran el camino feliz, la entrada malformada y la entrada fuera de rango.
3. **Tercera hora:** convierte el manejo de errores en un registro: incluye estado, razón, versión de regla y timestamp si está disponible.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *Errors and exceptions*. Python 3.14.4 documentation. <https://docs.python.org/3/tutorial/errors.html>
- Python Software Foundation. (2026). *Defining clean-up actions*. Python 3.14.4 documentation. <https://docs.python.org/3/tutorial/errors.html#defining-clean-up-actions>
- Python Software Foundation. (2026). *PEP 8: Style guide for Python code*. <https://peps.org/pep-0008/>
- Python Software Foundation. (2026). *PEP 20: The Zen of Python*. <https://peps.org/pep-0020/>
- Sittig, D. F., & Singh, H. (2010). A new sociotechnical model for studying health information technology in complex adaptive healthcare systems. *Quality & Safety in Health Care, 19*(Suppl. 3), i68-i74. <https://doi.org/10.1136/qshc.2010.042085>
- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>

## Siguiente paso

El siguiente tema natural es la verificación mínima del cálculo: pruebas, asserts, y separación entre pruebas unitarias y pruebas de integración. Si las funciones encapsulan criterio y las excepciones lo hacen explícito cuando el criterio falla, las pruebas enseñan cómo saber si el criterio funciona antes de que importe de verdad.
