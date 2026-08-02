# Números, unidades y mediciones

Un número biomédico rara vez viaja solo. `70` puede ser peso, edad, frecuencia cardíaca, glucosa, dosis, porcentaje, mililitros o años. Python no sabe eso. Para el intérprete, `70` es un entero. Para un sistema biomédico, puede ser una medición, una unidad omitida, un valor imposible, un dato incompleto o una decisión pendiente.

La diferencia importa porque los números se dejan operar con facilidad. Se suman, se dividen, se comparan y se ordenan. Esa obediencia técnica puede ocultar una falta de sentido. Un programa puede calcular sin protestar que `70 kg + 175 cm = 245`. El resultado existe; el significado no.

La primera regla de esta unidad es simple: no todo número es una cantidad segura para decidir.

## Origen técnico: medir no es contar

Contar y medir parecen operaciones hermanas, pero no prometen lo mismo. Contar leucocitos, pacientes o eventos adversos produce cantidades discretas. Medir peso, temperatura o concentración produce una aproximación dentro de una unidad, con instrumento, precisión y contexto. En ambos casos hay números; en ninguno el número agota el dato.

La informática hereda esa diferencia como tipos y contratos. Un entero puede representar conteos. Un decimal puede representar mediciones. Un texto puede conservar una unidad. Pero la responsabilidad de conectar esos elementos es del diseño del programa. Python ofrece herramientas; el dominio decide qué significan.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una medición biomédica es un valor numérico acompañado, como mínimo, por una unidad, una regla de validez y un significado de dominio. El número permite operar; la unidad y la validación permiten decidir si la operación tiene sentido.
</div>

La definición tiene cuatro consecuencias.

**La unidad no es comentario.** Si un valor puede estar en miligramos o microgramos, escribir la unidad en una nota no basta. La unidad debe estar disponible para el código que valida o transforma.

**El rango posible no es el rango normal.** Una estatura de `0` no es "baja"; es inválida para calcular IMC. Un potasio de `6.1` puede ser crítico o no según el contexto, pero sigue siendo una medición fisiológicamente posible. El validador debe distinguir imposibilidad, fuera de rango y alerta.

**La precisión también es información.** `37`, `37.0` y `37.04` no comunican exactamente lo mismo. En esta etapa no haremos teoría de error de medición, pero sí evitaremos borrar precisión por descuido.

**El dato faltante no es número.** Usar `0` para representar "no medido" destruye la semántica. El programa podrá calcular; el cálculo mentirá.

## Versión ingenua: números desnudos

Supongamos que una función decide si una concentración de potasio está fuera de rango.

```python
def clasificar_potasio(valor):
    # Versión frágil: recibe un número sin unidad ni validación de dominio.
    if valor < 3.5:
        return "bajo"
    if valor > 5.1:
        return "alto"
    return "en_rango"


print(clasificar_potasio(4.2))
print(clasificar_potasio(4200))
print(clasificar_potasio(0))
```

Salida esperada:

```text
en_rango
alto
bajo
```

La función ejecuta. Ese es el problema: ejecuta demasiado. Acepta `4200` sin preguntar si son micromoles por litro, una unidad mal convertida o un error de captura. Acepta `0` como potasio bajo, aunque para este ejemplo pedagógico debería tratarse como valor incompatible con una medición real.

El error no está en el `if`. Está en haber permitido que un número desnudo entrara a una decisión.

## Crítica técnica: qué está mal

Primero, la función mezcla cálculo con contrato implícito. Parece recibir "potasio", pero en realidad recibe cualquier número comparable. No hay rastro de unidad, método, población ni validez mínima.

Segundo, la salida confunde estados. `bajo` puede significar hipopotasemia real, dato faltante codificado como cero, unidad equivocada o error de digitación. Son situaciones diferentes que exigirían acciones diferentes.

Tercero, el nombre del parámetro `valor` no ayuda. En ejemplos pequeños parece suficiente; en sistemas reales se vuelve una invitación al abuso. Un nombre como `potasio_mmol_l` ya no resuelve todo, pero hace visible una promesa.

Cuarto, no hay frontera entre lo imposible y lo preocupante. Un valor imposible debe detener o degradar la decisión; un valor preocupante debe producir una alerta. Tratar ambos como "alto" o "bajo" borra trazabilidad.

## Versión mejorada: tipo mínimo de medición

Sin instalar librerías externas, podemos representar una medición como un diccionario con tres piezas: nombre, valor y unidad. Después se valida antes de clasificar.

```python
def validar_medicion(medicion, unidad_esperada, minimo_posible, maximo_posible):
    """Valida que una medición tenga unidad correcta y valor físicamente plausible."""
    if medicion is None:
        return {"estado": "sin_dato", "razon": "medicion_ausente"}

    if medicion["unidad"] != unidad_esperada:
        return {
            "estado": "unidad_incompatible",
            "razon": f"se esperaba {unidad_esperada}",
        }

    valor = medicion["valor"]
    if valor < minimo_posible or valor > maximo_posible:
        return {"estado": "valor_imposible", "razon": "fuera_de_limites_fisiologicos"}

    return {"estado": "valida", "razon": "medicion_aceptada"}


def clasificar_potasio(medicion):
    """Clasifica una medición pedagógica de potasio después de validar su contrato."""
    validacion = validar_medicion(
        medicion=medicion,
        unidad_esperada="mmol/L",
        minimo_posible=1.0,
        maximo_posible=10.0,
    )

    if validacion["estado"] != "valida":
        return {
            "estado": validacion["estado"],
            "accion": "revisar_dato",
            "razon": validacion["razon"],
        }

    valor = medicion["valor"]
    if valor < 3.5:
        estado = "bajo"
    elif valor > 5.1:
        estado = "alto"
    else:
        estado = "en_rango"

    return {
        "estado": estado,
        "accion": "interpretar_con_contexto",
        "razon": "potasio_validado_en_mmol_l",
    }


potasio = {"nombre": "potasio", "valor": 4.2, "unidad": "mmol/L"}
print(clasificar_potasio(potasio))
```

Salida esperada:

```text
{'estado': 'en_rango', 'accion': 'interpretar_con_contexto', 'razon': 'potasio_validado_en_mmol_l'}
```

La versión mejorada no pretende ser una historia clínica electrónica. Es una miniatura con una disciplina nueva: antes de decidir, verifica qué está decidiendo. El dato ya no es `4.2`; es una medición con unidad y contrato.

## Anatomía del contrato

La función `validar_medicion` separa tres preguntas.

```python
if medicion is None:
    return {"estado": "sin_dato", "razon": "medicion_ausente"}
```

Primero, pregunta si hay medición. Ausencia no es normalidad ni anormalidad; es otro estado.

```python
if medicion["unidad"] != unidad_esperada:
    return {
        "estado": "unidad_incompatible",
        "razon": f"se esperaba {unidad_esperada}",
    }
```

Segundo, pregunta si la unidad permite comparar. Una concentración en otra unidad no es necesariamente incorrecta, pero no debe entrar a la regla sin conversión explícita.

```python
if valor < minimo_posible or valor > maximo_posible:
    return {"estado": "valor_imposible", "razon": "fuera_de_limites_fisiologicos"}
```

Tercero, pregunta si el valor pertenece al dominio plausible. Este límite no es el rango normal; es una barrera de seguridad para no clasificar errores de captura como hallazgos clínicos.

La clasificación solo aparece después. Ese orden es el núcleo técnico de la sección.

## Pruebas mínimas

La sección anterior propuso cuatro propiedades: caso documentado, dato faltante, frontera y reproducibilidad. Apliquémoslas aquí.

```python
# Propiedad 1: caso documentado.
potasio = {"nombre": "potasio", "valor": 4.2, "unidad": "mmol/L"}
assert clasificar_potasio(potasio)["estado"] == "en_rango"

# Propiedad 2: dato faltante se conserva como ausencia.
assert clasificar_potasio(None)["estado"] == "sin_dato"

# Propiedad 3: unidad incompatible no se clasifica como alto o bajo.
potasio_micro = {"nombre": "potasio", "valor": 4200, "unidad": "umol/L"}
assert clasificar_potasio(potasio_micro)["estado"] == "unidad_incompatible"

# Propiedad 4: frontera superior del rango de referencia pedagógico.
assert clasificar_potasio({"nombre": "potasio", "valor": 5.1, "unidad": "mmol/L"})["estado"] == "en_rango"
assert clasificar_potasio({"nombre": "potasio", "valor": 5.11, "unidad": "mmol/L"})["estado"] == "alto"
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas no validan medicina clínica. Validan el contrato de representación. Esa diferencia es central: el libro enseña a no confundir corrección del código con suficiencia clínica del criterio.

## Ejemplo biomédico progresivo: dosis por peso

La dosis por peso muestra por qué la unidad importa antes de multiplicar. El cálculo es elemental; el contrato no.

```python
def calcular_dosis_por_peso(peso, dosis_mg_kg):
    """Calcula dosis total solo si el peso está expresado en kilogramos."""
    validacion = validar_medicion(
        medicion=peso,
        unidad_esperada="kg",
        minimo_posible=0.5,
        maximo_posible=300,
    )

    if validacion["estado"] != "valida":
        return {"estado": validacion["estado"], "dosis_mg": None}

    dosis_total = peso["valor"] * dosis_mg_kg
    return {"estado": "calculada", "dosis_mg": round(dosis_total, 2)}


peso = {"nombre": "peso", "valor": 70, "unidad": "kg"}
print(calcular_dosis_por_peso(peso, dosis_mg_kg=10))
```

Salida esperada:

```text
{'estado': 'calculada', 'dosis_mg': 700}
```

El ejemplo es deliberadamente incompleto: no revisa edad, función renal, dosis máxima, indicación, vía ni contraindicaciones. Esa omisión está declarada porque el objetivo de la sección no es dosificar un medicamento. Es mostrar que incluso el cálculo más simple necesita una representación honesta.

## CODE CLEAN: nombre semántico contra magia numérica

La versión frágil escondía constantes sin nombre: `3.5`, `5.1`, `1.0`, `10.0`. Esas cifras pueden ser pedagógicas, pero no deben parecer magia.

```python
UNIDAD_POTASIO = "mmol/L"
POTASIO_POSIBLE = (1.0, 10.0)
POTASIO_REFERENCIA = (3.5, 5.1)
```

Nombrar constantes no vuelve clínicamente válida una regla. Vuelve auditable el programa. Si mañana cambia un rango, si se adapta a otra población o si se decide usar otro analito, el lector sabe dónde vive la decisión.

Código limpio, en este libro, no significa código bonito. Significa que el supuesto importante tiene un lugar visible.

## Límites y errores frecuentes

1. **Guardar unidades en comentarios.** El comentario ayuda al lector, pero el programa no puede validarlo.
2. **Usar `0` como dato faltante.** Convierte ausencia en fisiología y contamina promedios, alertas y decisiones.
3. **Confundir plausible con normal.** Un valor puede ser fisiológicamente posible y estar fuera de rango.
4. **Comparar unidades incompatibles.** Si la unidad cambia, la regla debe convertir o rechazar; nunca adivinar.
5. **Redondear demasiado pronto.** Redondear para mostrar no es lo mismo que redondear para calcular. La presentación puede perder precisión; el cálculo debe conservarla mientras sea útil.

## Argumentos críticos

### Desacuerdo 1: estructura explícita contra simplicidad

Pregunta: ¿no es demasiado escribir un diccionario para una medición simple?

El argumento de la simplicidad dice que `4.2` basta si todos saben que se habla de potasio. El argumento de la estructura responde que los sistemas fallan precisamente cuando "todos saben" deja de ser cierto: otro archivo, otro laboratorio, otra unidad, otro programador.

Consenso operativo: en ejercicios mínimos puede aparecer el número desnudo para enseñar sintaxis; en decisiones biomédicas debe aparecer la estructura mínima que preserve unidad y validez.

### Desacuerdo 2: rechazar contra convertir

Pregunta: si llega una unidad incompatible, ¿debe rechazarse o convertirse?

Convertir puede ser útil cuando la equivalencia es clara, la unidad está bien identificada y la conversión está probada. Rechazar es más seguro cuando la procedencia es dudosa o la conversión exige contexto.

Consenso operativo: primero rechazar de forma explícita; después, cuando el sistema madure, agregar conversiones nombradas, probadas y trazables.

### Desacuerdo 3: rango fisiológico contra rango de referencia

Pregunta: ¿cuántos rangos necesita una medición?

El rango fisiológico posible protege contra errores de captura. El rango de referencia ayuda a interpretar. El rango crítico guía alerta. Mezclarlos en una sola pareja de números produce decisiones opacas.

Consenso operativo: separar los rangos por función. Si solo hay uno en un ejemplo pedagógico, declarar cuál es.

## Puente hacia la frontera

Los sistemas biomédicos reales no se conforman con diccionarios escritos a mano. Usan unidades estandarizadas, codificaciones, esquemas, validadores, tablas, modelos de datos y contratos de interoperabilidad. Más adelante aparecerán bibliotecas y formatos que ayudan con esas tareas. Pero la idea no cambia: ningún estándar salva un diseño que no sabe qué promete su dato.

En bioinformática, una cobertura de lectura no es "un número"; depende de profundidad, región, método y calidad. En señales fisiológicas, una frecuencia de muestreo no es "un entero"; gobierna qué fenómenos pueden verse. En medicina de precisión, una variante genética no es "texto"; es una representación con coordenadas, referencia, alelo, calidad y consecuencia.

La frontera empieza en la misma disciplina de esta sección: no dejar que el número finja ser todo el dato.

## Evaluar si entendiste

1. ¿Por qué `70` no es suficiente para representar un peso?
2. ¿Qué diferencia hay entre rango posible, rango de referencia y rango crítico?
3. ¿Por qué una unidad no debe vivir solo en un comentario?
4. ¿Qué error aparece cuando se usa `0` como dato faltante?
5. ¿Por qué una unidad incompatible debe rechazarse antes de clasificar?
6. ¿Qué propiedad verifican las pruebas de frontera?
7. ¿Por qué redondear para mostrar no equivale a redondear para calcular?
8. ¿Qué gana el código al nombrar constantes como `POTASIO_REFERENCIA`?
9. ¿Cuándo sería razonable convertir unidades en vez de rechazarlas?
10. ¿Qué parte del ejemplo de dosis por peso está deliberadamente incompleta y por qué?

## Vacíos de comprensión que debes vigilar

1. Creer que el tipo técnico de Python basta. `float` dice cómo opera el intérprete; no dice qué significa la medición.
2. Tratar todo valor fuera de rango como el mismo error. Unidad incompatible, valor imposible y alerta clínica son estados diferentes.
3. Pensar que más estructura siempre es mejor. La estructura correcta es la mínima que protege el significado necesario para decidir.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma cinco mediciones biomédicas y escribe para cada una: valor, unidad, rango posible y rango de referencia. No programes todavía.
2. **Segunda hora:** implementa una función `validar_medicion` para una de ellas y escribe pruebas de ausencia, unidad incompatible y frontera.
3. **Tercera hora:** revisa un código propio donde hayas usado números desnudos; reemplaza al menos uno por una estructura con unidad y razón.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *Built-in Types*. Python 3.14.4 documentation. <https://docs.python.org/3/library/stdtypes.html>
- Python Software Foundation. (2026). *Data Classes*. Python 3.14.4 documentation. <https://docs.python.org/3/library/dataclasses.html>
- National Academies of Sciences, Engineering, and Medicine. (2019). *Reproducibility and Replicability in Science*. National Academies Press. <https://doi.org/10.17226/25303>

## Siguiente paso

Los números muestran que un dato necesita unidad y validación. El siguiente problema parece más inocente: texto. Pero en biomedicina, texto libre puede significar nota clínica, categoría diagnóstica, código, sinónimo, error ortográfico o vocabulario controlado. La siguiente sección estudiará textos y vocabularios: cuándo una cadena alcanza y cuándo se convierte en una fuente de ambigüedad.

