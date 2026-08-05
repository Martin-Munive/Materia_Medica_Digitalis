# Ausencia de datos, valores centinela y marcadores especiales

Un dato ausente no es un espacio vacío sin consecuencias. En medicina y ciencias de la vida, la ausencia puede significar muchas cosas: no se preguntó, no se midió, no aplica, el paciente no respondió, el equipo falló, el resultado está pendiente, el valor fue censurado, el dato se perdió durante una migración o el sistema heredado usó un número especial para representar que no había número.

La sección anterior mostró que el tiempo puede fallar por precisión falsa. Esta sección trata una trampa todavía más cotidiana: convertir ausencias distintas en un mismo valor cómodo.

En bases biomédicas aparecen con frecuencia valores como `None`, `""`, `"NA"`, `"N/A"`, `"desconocido"`, `0`, `-1`, `999`, `9999`, `"sin dato"` o `"no aplica"`. A veces son inevitables al importar datos de formularios, hojas de cálculo, equipos o sistemas antiguos. El problema no es que existan. El problema es dejarlos entrar al cálculo como si fueran valores reales.

## Origen técnico: Python tiene `None`, el dominio tiene razones

Python usa `None` para representar ausencia de valor. No es cero, no es cadena vacía, no es falso clínico y no es una categoría biomédica. Es un objeto especial que suele indicar que algo no tiene valor asignado.

```python
valor = None

print(valor is None)
print(valor == 0)
print(valor == "")
```

Salida esperada:

```text
True
False
False
```

Esto es útil, pero limitado. `None` puede decir que no hay valor; no dice por qué no hay valor. En un programa pequeño esa diferencia puede parecer excesiva. En un sistema biomédico, la razón de la ausencia puede cambiar la interpretación.

No es lo mismo:

- presión arterial no tomada;
- presión arterial imposible por error de captura;
- presión arterial pendiente de cargar;
- pregunta no aplicable;
- valor oculto por privacidad;
- resultado inválido por muestra hemolizada;
- medición omitida por decisión clínica.

Todos esos casos pueden terminar como `None`, pero no significan lo mismo.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un dato ausente es un valor no disponible para una operación concreta. Un valor centinela es un marcador especial usado para representar ausencia, error, no aplicabilidad u otro estado. En dominios biomédicos, todo centinela debe traducirse a un estado explícito antes de calcular, comparar o decidir.
</div>

Esta definición separa cuatro objetos.

**Ausencia técnica.** El programa no tiene valor: `None`, cadena vacía, campo omitido.

**Ausencia de dominio.** El valor no existe, no aplica, no se midió o no está disponible por una razón que importa.

**Valor centinela.** Valor especial que parece dato ordinario, pero significa otra cosa: `999`, `-1`, `"NA"`, `"no aplica"`.

**Estado explícito.** Representación controlada de la razón: `no_medido`, `no_aplica`, `pendiente`, `invalido`, `fuera_de_contrato`.

La regla práctica es esta: antes de calcular, traducir centinelas a estados. No permitir que un marcador especial pase como dato real.

## Versión ingenua: promediar con centinelas

Supongamos una lista pedagógica de presiones sistólicas. El sistema heredado usa `999` para indicar que la presión no fue tomada.

```python
presiones_sistolicas = [120, 128, 999, 132]

promedio = sum(presiones_sistolicas) / len(presiones_sistolicas)
print(promedio)
```

Salida esperada:

```text
344.75
```

El código funciona y el resultado es absurdo. No hubo error técnico. Python sumó números. El error está en permitir que `999` entrara al promedio como si fuera una presión real.

Podemos intentar una mejora rápida:

```python
presiones_validas = [valor for valor in presiones_sistolicas if valor != 999]
promedio = sum(presiones_validas) / len(presiones_validas)
print(promedio)
```

Salida esperada:

```text
126.66666666666667
```

El promedio mejora, pero todavía falta algo: el sistema perdió que hubo una medición no tomada. Para un análisis de calidad, una auditoría o una regla de completitud, esa ausencia importa.

## Crítica técnica: qué está mal

Primero, el centinela está escondido dentro de una lista de números. `999` parece entero y por eso participa en operaciones aritméticas.

Segundo, la limpieza rápida elimina información. Quitar `999` permite calcular, pero borra que el registro tenía un dato ausente.

Tercero, no se distingue razón de ausencia. `999` podría significar no tomado, pendiente, no aplica, error de equipo o valor censurado. Si no se documenta, el sistema solo aprende a ignorar.

Cuarto, no hay trazabilidad. El promedio final no informa cuántos valores fueron válidos, cuántos fueron excluidos ni por qué.

Quinto, no hay frontera fisiológica. Aunque `999` sea el centinela obvio, otros valores pueden ser técnicamente numéricos y biomédicamente imposibles.

## Versión mejorada: traducir antes de calcular

Una representación mínima debe separar valor, estado y razón. Usaremos `Enum` para gobernar estados de medición.

```python
from enum import Enum


class EstadoMedicion(Enum):
    VALIDA = "valida"
    NO_MEDIDA = "no_medida"
    NO_APLICA = "no_aplica"
    PENDIENTE = "pendiente"
    INVALIDA = "invalida"
    FUERA_DE_CONTRATO = "fuera_de_contrato"


CENTINELAS_PRESION = {
    999: EstadoMedicion.NO_MEDIDA,
    -1: EstadoMedicion.INVALIDA,
    "NA": EstadoMedicion.NO_APLICA,
    "PENDIENTE": EstadoMedicion.PENDIENTE,
}


def validar_presion_sistolica(valor):
    """Traduce valores crudos a mediciones con estado antes de calcular."""
    if isinstance(valor, (int, float, str)) and valor in CENTINELAS_PRESION:
        return {
            "valor": None,
            "unidad": "mmHg",
            "estado": CENTINELAS_PRESION[valor],
            "razon": "centinela_traducido",
        }

    if valor is None or valor == "":
        return {
            "valor": None,
            "unidad": "mmHg",
            "estado": EstadoMedicion.NO_MEDIDA,
            "razon": "ausencia_tecnica",
        }

    if not isinstance(valor, (int, float)):
        return {
            "valor": None,
            "unidad": "mmHg",
            "estado": EstadoMedicion.FUERA_DE_CONTRATO,
            "razon": "tipo_no_numerico",
        }

    if valor < 40 or valor > 260:
        return {
            "valor": valor,
            "unidad": "mmHg",
            "estado": EstadoMedicion.INVALIDA,
            "razon": "fuera_de_rango_pedagogico",
        }

    return {
        "valor": valor,
        "unidad": "mmHg",
        "estado": EstadoMedicion.VALIDA,
        "razon": "medicion_aceptada",
    }


def resumir_presiones_sistolicas(valores_crudos):
    """Calcula promedio solo con valores válidos y conserva trazabilidad de exclusiones."""
    mediciones = [validar_presion_sistolica(valor) for valor in valores_crudos]
    valores_validos = [
        medicion["valor"]
        for medicion in mediciones
        if medicion["estado"] == EstadoMedicion.VALIDA
    ]

    conteo_por_estado = {}
    for medicion in mediciones:
        estado = medicion["estado"].value
        conteo_por_estado[estado] = conteo_por_estado.get(estado, 0) + 1

    if not valores_validos:
        return {
            "estado": "no_calculable",
            "promedio": None,
            "unidad": "mmHg",
            "conteo_por_estado": conteo_por_estado,
            "razon": "sin_valores_validos",
        }

    return {
        "estado": "calculable",
        "promedio": sum(valores_validos) / len(valores_validos),
        "unidad": "mmHg",
        "conteo_por_estado": conteo_por_estado,
        "razon": "promedio_de_mediciones_validas",
    }


resultado = resumir_presiones_sistolicas([120, 128, 999, 132])
print(resultado["estado"])
print(round(resultado["promedio"], 1), resultado["unidad"])
print(resultado["conteo_por_estado"])
```

Salida esperada:

```text
calculable
126.7 mmHg
{'valida': 3, 'no_medida': 1}
```

El promedio ya no queda contaminado por el centinela, pero tampoco se borra la ausencia. El resultado conserva cuántos datos fueron válidos y cuántos fueron no medidos.

## Anatomía del contrato

La primera pieza es el vocabulario de estados.

```python
class EstadoMedicion(Enum):
    VALIDA = "valida"
    NO_MEDIDA = "no_medida"
    NO_APLICA = "no_aplica"
    PENDIENTE = "pendiente"
    INVALIDA = "invalida"
    FUERA_DE_CONTRATO = "fuera_de_contrato"
```

Cada estado cambia algo. `NO_MEDIDA` puede activar completitud. `NO_APLICA` puede excluir legítimamente de un denominador. `PENDIENTE` puede requerir espera. `INVALIDA` puede indicar error de calidad. `FUERA_DE_CONTRATO` puede revelar problema de importación.

La segunda pieza es la tabla de centinelas.

```python
CENTINELAS_PRESION = {
    999: EstadoMedicion.NO_MEDIDA,
    -1: EstadoMedicion.INVALIDA,
    "NA": EstadoMedicion.NO_APLICA,
}
```

El centinela no se deja disperso en condiciones sueltas por todo el código. Se declara en un lugar visible. Esto permite auditarlo, cambiarlo o eliminarlo durante una migración.

La tercera pieza es la traducción antes del cálculo.

```python
if valor in CENTINELAS_PRESION:
    return {
        "valor": None,
        "estado": CENTINELAS_PRESION[valor],
        "razon": "centinela_traducido",
    }
```

El valor crudo se convierte en una estructura con estado. Después de esta frontera, el cálculo trabaja con datos validados, no con símbolos heredados.

La cuarta pieza es la trazabilidad.

```python
"conteo_por_estado": conteo_por_estado
```

El resultado no solo entrega promedio. Entrega qué ocurrió con los datos. En biomedicina, esa diferencia puede separar un cálculo útil de una salida engañosa.

## Pruebas mínimas

```python
# Propiedad 1: el centinela 999 no entra como presión válida.
assert validar_presion_sistolica(999)["estado"] == EstadoMedicion.NO_MEDIDA

# Propiedad 2: una presión válida conserva valor y unidad.
presion = validar_presion_sistolica(120)
assert presion["estado"] == EstadoMedicion.VALIDA
assert presion["unidad"] == "mmHg"

# Propiedad 3: un valor imposible se marca como inválido.
assert validar_presion_sistolica(400)["estado"] == EstadoMedicion.INVALIDA

# Propiedad 4: el promedio excluye centinelas pero conserva el conteo de ausencia.
resultado = resumir_presiones_sistolicas([120, 128, 999, 132])
assert round(resultado["promedio"], 1) == 126.7
assert resultado["conteo_por_estado"]["no_medida"] == 1

# Propiedad 5: sin valores válidos no se inventa promedio.
sin_validos = resumir_presiones_sistolicas([999, "NA", -1])
assert sin_validos["estado"] == "no_calculable"
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Las pruebas validan una propiedad de representación: un marcador de ausencia no debe contaminar el cálculo ni desaparecer sin rastro.

## `None`, cero y cadena vacía no son equivalentes

Una trampa frecuente consiste en tratar todos los valores falsy de Python como si fueran ausencia biomédica.

```python
valores = [None, 0, "", [], "0"]

for valor in valores:
    print(bool(valor))
```

Salida esperada:

```text
False
False
False
False
True
```

Python está evaluando valor de verdad técnico, no significado de dominio. `0` puede ser una medición real: cero crisis epilépticas, cero colonias, cero eventos, cero días de retraso. Una cadena vacía puede significar campo sin diligenciar. Una lista vacía puede significar que no se encontraron eventos o que no se buscó.

Por eso una condición como esta es peligrosa:

```python
if not valor:
    estado = "faltante"
```

Puede convertir un cero real en faltante. Una versión más responsable pregunta por casos específicos.

```python
def clasificar_conteo_eventos(valor):
    """Distingue cero real de dato no medido en un conteo pedagógico."""
    if valor is None or valor == "":
        return {
            "estado": "no_medido",
            "valor": None,
            "razon": "ausencia_tecnica",
        }

    if not isinstance(valor, int):
        return {
            "estado": "fuera_de_contrato",
            "valor": None,
            "razon": "conteo_no_entero",
        }

    if valor < 0:
        return {
            "estado": "invalido",
            "valor": valor,
            "razon": "conteo_negativo",
        }

    return {
        "estado": "valido",
        "valor": valor,
        "razon": "conteo_aceptado",
    }


print(clasificar_conteo_eventos(0)["estado"])
print(clasificar_conteo_eventos(None)["estado"])
```

Salida esperada:

```text
valido
no_medido
```

El cero conserva su significado. La ausencia conserva el suyo.

## Denominadores: el lugar donde los faltantes cambian conclusiones

El manejo de datos ausentes no solo afecta promedios. También afecta proporciones.

Supongamos cuatro registros de una variable pedagógica: vacunación documentada. Usaremos tres estados: sí, no y no documentado.

```python
registros = ["si", "no", "no_documentado", "si"]

vacunados = sum(1 for registro in registros if registro == "si")
total_registros = len(registros)
registros_documentados = sum(1 for registro in registros if registro in {"si", "no"})

print(vacunados / total_registros)
print(vacunados / registros_documentados)
```

Salida esperada:

```text
0.5
0.6666666666666666
```

Ambos cálculos pueden ser correctos si declaran su denominador. El primero responde: proporción sobre todos los registros. El segundo responde: proporción entre registros documentados. Son preguntas distintas.

```python
def calcular_proporcion_documentada(registros):
    """Calcula proporción positiva solo entre registros con respuesta documentada."""
    positivos = sum(1 for registro in registros if registro == "si")
    documentados = sum(1 for registro in registros if registro in {"si", "no"})
    no_documentados = len(registros) - documentados

    if documentados == 0:
        return {
            "estado": "no_calculable",
            "proporcion": None,
            "denominador": "documentados",
            "no_documentados": no_documentados,
            "razon": "sin_registros_documentados",
        }

    return {
        "estado": "calculable",
        "proporcion": positivos / documentados,
        "denominador": "documentados",
        "no_documentados": no_documentados,
        "razon": "excluye_no_documentados_del_denominador",
    }


resultado = calcular_proporcion_documentada(["si", "no", "no_documentado", "si"])
print(round(resultado["proporcion"], 2))
print(resultado["denominador"])
print(resultado["no_documentados"])
```

Salida esperada:

```text
0.67
documentados
1
```

La proporción ya no viaja sola. Lleva denominador y conteo de no documentados.

## CODE CLEAN: no esconder centinelas en números mágicos

Comparemos dos estilos.

```python
if presion == 999:
    continue
```

El lector no sabe si `999` significa no medido, no aplica, error, pendiente o valor imposible. Esa condición puede repetirse en diez funciones y cambiar de significado sin control.

Una versión más limpia declara el contrato.

```python
CENTINELAS_PRESION = {
    999: EstadoMedicion.NO_MEDIDA,
    -1: EstadoMedicion.INVALIDA,
}
```

Y luego traduce:

```python
medicion = validar_presion_sistolica(valor_crudo)
```

Código limpio no significa ocultar los datos sucios. Significa crear una frontera explícita donde lo sucio se reconoce, se traduce y se registra.

También conviene evitar nombres genéricos:

```python
def limpiar(x):
    ...
```

Preferir:

```python
def validar_presion_sistolica(valor_crudo):
    ...
```

El segundo nombre declara dominio, tipo de dato y momento del proceso: todavía es valor crudo, todavía debe validarse.

## Límites y errores frecuentes

1. **Usar `if not valor` para detectar faltantes.** Convierte ceros reales en ausencias.
2. **Promediar centinelas.** `999`, `-1` o `0` pueden contaminar cálculos si son marcadores heredados.
3. **Eliminar filas sin contar qué se eliminó.** Una limpieza sin trazabilidad puede cambiar conclusiones sin dejar evidencia.
4. **Mezclar no medido con no aplicable.** `NO_MEDIDA` indica ausencia de medición; `NO_APLICA` indica que la pregunta no corresponde.
5. **Tratar inválido como faltante simple.** Un resultado inválido puede revelar problema de calidad, equipo, muestra o captura.
6. **Ocultar centinelas en condiciones sueltas.** Los números mágicos deben concentrarse en una tabla o validador.
7. **No declarar denominador.** Una proporción con faltantes puede cambiar según se use total, documentados, elegibles o medidos.
8. **Convertir texto `"0"` en cero sin validar.** La conversión puede ser correcta o puede esconder un error de importación.
9. **Confundir dato pendiente con dato ausente.** Pendiente puede resolverse; ausente puede no existir o no haberse medido.
10. **Usar un solo estado `desconocido` para todo.** Simplifica, pero puede borrar acciones diferentes.

## Argumentos críticos

### Desacuerdo 1: eliminar faltantes contra conservar estados

Pregunta: ¿por qué no eliminar todos los registros incompletos?

Eliminar puede ser razonable para un cálculo concreto si se declara. Pero si se elimina sin contar, el sistema pierde información de calidad. En medicina, el patrón de faltantes también puede ser información: una variable puede faltar más en un grupo, una sede, una hora, una condición o una etapa del proceso.

Consenso operativo: se puede excluir para calcular, pero se debe conservar conteo, razón y denominador.

### Desacuerdo 2: centinelas contra `None`

Pregunta: ¿por qué no reemplazar todos los centinelas por `None`?

`None` es mejor que un número mágico para evitar cálculos accidentales, pero puede borrar la razón. Si `999` significaba no medido y `"NA"` significaba no aplica, ambos no deberían terminar como un mismo `None` sin estado adicional.

Consenso operativo: traducir centinelas a `valor=None` más `estado` y `razon`.

### Desacuerdo 3: rango fisiológico contra regla técnica

Pregunta: ¿un valor fuera de rango debe tratarse como faltante?

No automáticamente. Un valor fuera de rango puede ser error de captura, unidad equivocada, caso extremo o dato real raro. Una miniatura pedagógica puede marcarlo como inválido, pero un sistema real necesitaría reglas de revisión y contexto.

Consenso operativo: no convertir imposibles o extremos en faltantes silenciosos. Marcar como inválido o requiere revisión.

### Desacuerdo 4: imputar contra no calcular

Pregunta: ¿cuándo se pueden rellenar datos faltantes?

La imputación puede ser útil en análisis estadístico, investigación o modelos predictivos, pero introduce supuestos. No debe usarse como relleno invisible para decisiones individuales. En este punto del libro, la regla pedagógica es más conservadora: si falta lo necesario, devolver `no_calculable` o `evaluacion_incompleta`.

Consenso operativo: no imputar sin declarar método, propósito, población y límite.

## Puente hacia la frontera

El tratamiento de datos ausentes abre una frontera enorme. En análisis biomédico real aparecen mecanismos de ausencia: datos faltantes completamente al azar, al azar condicionados por otras variables o no al azar. También aparecen censura, pérdida de seguimiento, sesgo de selección, imputación múltiple, modelos de supervivencia, auditoría de calidad de datos y evaluación de completitud.

En sistemas clínicos, los faltantes no son solo inconvenientes estadísticos. Pueden afectar seguridad. Una contraindicación no documentada no equivale a contraindicación ausente. Un resultado pendiente no equivale a resultado normal. Una variable no aplicable no debe castigar calidad de registro. Un cero real no debe desaparecer por una condición técnica.

Más adelante, el libro podrá conectar esta sección con tablas, `pandas`, validación de esquemas, imputación, cohortes, sesgo, análisis longitudinal y modelos predictivos. El principio mínimo seguirá igual: los datos ausentes deben tener estado antes de convertirse en cálculo.

## Evaluar si entendiste

1. ¿Por qué `999` puede ser más peligroso que `None` en una columna numérica?
2. ¿Qué diferencia hay entre ausencia técnica y ausencia de dominio?
3. ¿Por qué `0` no debe tratarse automáticamente como dato faltante?
4. ¿Qué gana un sistema al traducir centinelas antes de calcular?
5. ¿Cuál es la diferencia entre `NO_MEDIDA`, `NO_APLICA`, `PENDIENTE` e `INVALIDA`?
6. ¿Por qué eliminar registros incompletos puede cambiar una conclusión?
7. ¿Qué debe acompañar una proporción cuando hay datos faltantes?
8. ¿Cuándo un valor fuera de rango debe detener el cálculo?
9. ¿Por qué `None` solo no basta para explicar una ausencia biomédica?
10. ¿Qué prueba escribirías para asegurar que un cero real no se convierte en faltante?

## Vacíos de comprensión que debes vigilar

1. Creer que limpiar datos es borrar lo incómodo. Limpiar responsablemente es traducir, validar y conservar trazabilidad.
2. Pensar que `None` resuelve todos los faltantes. Resuelve ausencia técnica, no razón de dominio.
3. Confundir cálculo posible con cálculo honesto. Un promedio puede ejecutarse y seguir siendo inválido si entraron centinelas.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma una tabla pequeña e identifica todos los símbolos que podrían significar ausencia, no aplicabilidad, pendiente o error.
2. **Segunda hora:** escribe un diccionario de centinelas y una función que traduzca cada valor crudo a `valor`, `estado`, `unidad` y `razon`.
3. **Tercera hora:** escribe pruebas para cero real, `None`, cadena vacía, centinela numérico, valor imposible y ausencia total de valores válidos.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *The Python Standard Library: Constants*. Python 3 documentation. <https://docs.python.org/3/library/constants.html#None>
- Python Software Foundation. (2026). *Built-in Types: Truth Value Testing*. Python 3 documentation. <https://docs.python.org/3/library/stdtypes.html#truth-value-testing>
- pandas development team. (2026). *Working with missing data*. pandas documentation. <https://pandas.pydata.org/docs/user_guide/missing_data.html>
- NumPy Developers. (2026). *Constants: numpy.nan*. NumPy documentation. <https://numpy.org/doc/stable/reference/constants.html#numpy.nan>
- Little, R. J. A., & Rubin, D. B. (2019). *Statistical Analysis with Missing Data* (3rd ed.). Wiley.
- Sterne, J. A. C., White, I. R., Carlin, J. B., et al. (2009). Multiple imputation for missing data in epidemiological and clinical research: potential and pitfalls. *BMJ*, 338, b2393. <https://doi.org/10.1136/bmj.b2393>

## Siguiente paso

Los faltantes muestran que un dato necesita estado antes de entrar al cálculo. La próxima sección seguirá con colecciones: listas, diccionarios y registros. Allí el problema ya no será un valor aislado, sino cómo agrupar observaciones, conservar estructura, recorrer datos y evitar que una colección parezca más ordenada de lo que realmente está.
