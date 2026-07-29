# Pruebas y verificación mínima

Un programa puede pasar todas sus pruebas y seguir estando equivocado. No es una paradoja: es la consecuencia de escribir pruebas que recorren el código sin verificar nada. La suite muestra un verde tranquilizador, el equipo respira, y el defecto viaja hacia producción envuelto en esa confianza.

Las dos secciones anteriores enseñaron a gobernar el error y a separar las funciones que deciden de las que actúan sobre el mundo. Quedó pendiente la pregunta que sostiene a ambas: ¿cómo sabes que una pieza hace lo que promete? La respuesta corta es "probándola". La respuesta útil es saber qué significa probar.

## Origen técnico: la humildad de la verificación

En 1972, en su conferencia de aceptación del premio Turing, Edsger Dijkstra formuló una de las frases más citadas y menos obedecidas de la computación: las pruebas de un programa pueden mostrar la presencia de defectos, pero jamás su ausencia. La industria llevaba apenas unos años discutiendo la "ingeniería de software" y ya quedaba advertida: el verde no absuelve.

La medicina llegó a la misma conclusión por otro camino y con otro vocabulario. Una troponina negativa no elimina el infarto; lo hace menos probable. Una prueba diagnóstica se juzga por su sensibilidad —cuántos enfermos detecta— porque se sabe que ninguna captura todo. El clínico aprende pronto a preguntar no "¿qué dice el resultado?" sino "¿qué queda sin descartar después de este resultado?".

Esa es exactamente la actitud que la verificación mínima exige del programador. Un `assert` que pasa es una troponina negativa: información real, alcance limitado.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una prueba es una declaración ejecutable de una propiedad que un procedimiento debe cumplir. La verificación mínima es el conjunto más pequeño de pruebas que hace defendible una afirmación sobre ese procedimiento. Una propiedad es una afirmación general sobre la relación entre entradas y salidas, no un caso concreto.
</div>

La definición tiene tres consecuencias.

**Una prueba sin propiedad no verifica nada.** Recorrer el camino feliz demuestra que el código ejecuta, no que decide bien. Si no puedes completar la frase "esta prueba garantiza que...", la prueba es decoración.

**La prueba verifica el programa, no el dominio.** Un `assert` puede garantizar que la función clasifica según sus umbrales; no puede garantizar que esos umbrales sean clínicamente correctos. Confundir ambas capas es una fuente seria de falsa seguridad.

**La reproducibilidad es la primera propiedad.** Si misma entrada no produce misma salida, no existe procedimiento estable que verificar. Por eso la sección anterior trató la pureza como base: las funciones puras son el terreno donde las propiedades se pueden probar.

## Versión ingenua: la suite que no prueba nada

Supongamos que decidimos "agregar pruebas" a la función de IMC de las secciones anteriores.

```python
# Suite de "pruebas" de la función de IMC.
resultado = calcular_y_clasificar_imc(70, 1.75)

assert resultado["imc"] == resultado["imc"]      # comparación del valor consigo mismo
assert resultado["categoria"] == "normal"        # un solo caso, el más fácil

print("Las pruebas pasaron.")
```

Salida esperada:

```text
Las pruebas pasaron.
```

El mensaje es reconfortante. También es casi vacío. La primera línea sería verdadera para cualquier función del universo: compara el programa consigo mismo. La segunda verifica un único punto, el más amable, y calla sobre umbrales, errores y ausencias.

## Crítica técnica: qué está mal

Esta suite falla por cuatro razones.

Primero, contiene una prueba tautológica. `resultado["imc"] == resultado["imc"]` no puede fallar; su presencia solo infla el recuento de pruebas exitosas. Una suite crece en número y se encoge en señal.

Segundo, no toca ninguna frontera. Los defectos viven en los bordes: el umbral exacto de 25, la estatura cero, el dato que no llegó. Un caso central, elegido porque "se ve bien", evade deliberadamente el territorio donde el código se equivoca.

Tercero, el mensaje final confunde ejecución con verificación. "Las pruebas pasaron" significa aquí "el intérprete no se detuvo", que es lo mismo que ocurría sin suite.

Cuarto, no verifica reproducibilidad. Si mañana alguien introduce una variable global o una llamada al azar dentro de la función, esta suite seguirá pasando mientras la función deja de ser reproducible.

CODE CLEAN aquí no pide más pruebas. Pide propiedades: menos líneas, más afirmaciones.

## Versión mejorada: propiedades, no casos

La misma función se verifica con cuatro propiedades, cada una declarada antes de ejecutarse.

```python
# Propiedad 1: el caso documentado se clasifica correctamente.
resultado = calcular_y_clasificar_imc(70, 1.75)
assert resultado["imc"] == 22.86
assert resultado["categoria"] == "normal"

# Propiedad 2: el dato faltante se rechaza con vocabulario explícito, nunca con un número.
try:
    calcular_y_clasificar_imc(None, 1.75)
    raise AssertionError("debió lanzar ValueError")
except ValueError as error:
    assert str(error) == "dato_faltante"

# Propiedad 3: la frontera del umbral 25 decide la categoría.
assert calcular_y_clasificar_imc(97.5, 2.0)["categoria"] == "normal"    # imc = 24.375
assert calcular_y_clasificar_imc(100, 2.0)["categoria"] == "sobrepeso"  # imc = 25.0

# Propiedad 4: reproducibilidad — misma entrada, misma salida.
assert calcular_y_clasificar_imc(70, 1.75) == calcular_y_clasificar_imc(70, 1.75)
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Cada línea merece su comentario porque cada línea afirma algo distinto. La primera ancla el comportamiento documentado. La segunda garantiza que la ausencia se trata como ausencia y no se convierte en cifra: es la propiedad que separa un cálculo responsable de uno silencioso. La tercera vigila la frontera exacta donde la regla cambia de opinión. La cuarta convierte la reproducibilidad —una promesa del prefacio— en algo que se puede ejecutar.

Cuatro afirmaciones. Ninguna tautológica. La suite es más corta que antes y dice infinitamente más.

## Anatomía de una propiedad

La segunda propiedad tiene una forma que conviene memorizar.

```python
try:
    calcular_y_clasificar_imc(None, 1.75)
    raise AssertionError("debió lanzar ValueError")
except ValueError as error:
    assert str(error) == "dato_faltante"
```

La prueba espera que la función falle de una manera específica. Si la función acepta el `None` y devuelve un número, la línea del `AssertionError` se ejecuta y la prueba muere: exactamente lo que se quiere, porque una función que calcula sin datos es más peligrosa que una que se niega. Si la función lanza otra excepción distinta —un `TypeError`, por ejemplo—, la prueba también muere, porque el contrato prometía vocabulario de dominio. Solo sobrevive si ocurre lo prometido, con las palabras prometidas.

Nótese la asimetría con la versión ingenua: ahí se probaba que la función funciona; aquí se prueba además que falla correctamente. En sistemas que tocan personas, la segunda afirmación vale tanto como la primera.

## Ejemplo biomédico progresivo: verificar la regla de alerta

Volvamos sobre `evaluar_medicion`, la regla pedagógica de alertas de laboratorio de la sección anterior. Su contrato ofrece cuatro propiedades que una suite mínima puede defender.

```python
RANGO_K = (3.5, 5.1)      # rango pedagógico de potasio
CRITICO_K = (2.5, 6.0)    # límites pedagógicos de criticidad
SEVERIDAD = {"en_rango": 0, "fuera_de_rango": 1, "critico": 2}


def severidad(valor):
    """Atajo: severidad numérica del estado producido por la regla."""
    return SEVERIDAD[evaluar_medicion(valor, RANGO_K, CRITICO_K)["estado"]]


# Propiedad 1: la ausencia de dato se conserva como estado, no como cifra.
decision = evaluar_medicion(None, RANGO_K, CRITICO_K)
assert decision["estado"] == "sin_dato"
assert decision["accion"] == "solicitar_repeticion"

# Propiedad 2: la frontera crítica es exacta.
assert evaluar_medicion(6.0, RANGO_K, CRITICO_K)["estado"] == "fuera_de_rango"
assert evaluar_medicion(6.01, RANGO_K, CRITICO_K)["estado"] == "critico"

# Propiedad 3: el estado siempre pertenece al vocabulario del contrato.
for valor in [None, 2.0, 3.0, 4.2, 5.5, 6.4, 8.0]:
    estado = evaluar_medicion(valor, RANGO_K, CRITICO_K)["estado"]
    assert estado in {"en_rango", "fuera_de_rango", "critico", "sin_dato"}

# Propiedad 4: la gravedad no disminuye al alejarse del rango, en ninguna dirección.
cola_alta = [5.2, 5.5, 5.8, 6.0, 6.2, 6.5]
cola_baja = [3.4, 3.2, 3.0, 2.7, 2.4, 2.0]
assert all(severidad(a) <= severidad(b) for a, b in zip(cola_alta, cola_alta[1:]))
assert all(severidad(a) <= severidad(b) for a, b in zip(cola_baja, cola_baja[1:]))
```

Salida esperada: no imprime nada si las propiedades se cumplen.

La cuarta propiedad es la más interesante. No verifica casos: verifica una forma del mundo. El potasio es peligroso por dos colas —la hipopotasemia y la hiperpotasemia— y la suite declara que alejarse del rango nunca mejora el estado del paciente, en ninguna dirección. Podrían agregarse cincuenta valores a cada cola y la propiedad seguiría siendo una sola afirmación. Ese es el salto conceptual de esta sección: dejar de contar casos y empezar a declarar formas.

La regla sigue siendo una miniatura pedagógica. Lo que no es miniatura es el patrón: toda regla clínica automatizada debería poder defender sus propiedades con esta claridad.

## Límites y errores frecuentes

1. **Pruebas tautológicas.** Comparan el programa consigo mismo o repiten su lógica en la prueba. No pueden fallar; solo engordan el contador.
2. **Pruebas frágiles.** Verifican el formato del mensaje en vez de la decisión. Rompen con cambios inocuos —una tilde, un espacio— y enseñan al equipo a ignorar el rojo.
3. **Confundir cobertura con señal.** Cien líneas ejecutadas no dicen qué se verificó. La cobertura es un mapa de lo recorrido, no un certificado de lo correcto.
4. **Probar el ejecutor en lugar de la decisión.** Si la suite simula teclado, archivos y pantalla para llegar a la regla, la separación de la sección anterior quedó a medias.

## Argumentos críticos

### Desacuerdo 1: cobertura contra señal

Pregunta: ¿conviene medir el éxito de una suite por su cobertura de líneas?

El argumento de la cobertura es objetividad: un número que crece, visible en un informe. El argumento de la señal es que el número mide recorrido, no verificación; una línea ejecutada con una aserción débil es una línea desprotegida con buena asistencia.

Consenso operativo: primero las propiedades fuertes sobre las decisiones que importan; después, si se desea, la cobertura como mapa para encontrar lo olvidado. La cobertura como meta invierte el instrumento.

### Desacuerdo 2: exhaustividad contra velocidad

Pregunta: ¿cuántos casos bastan?

El argumento exhaustivo propone barrer el dominio: todos los enteros, todos los decimales plausibles. El argumento pragmático recuerda que una suite lenta deja de ejecutarse, y una suite que no se ejecuta es literatura.

Consenso operativo: los casos que mueren en la frontera —umbrales exactos, vacíos, extremos, el primer valor después del límite— más una propiedad de forma cuando exista, como la monotonicidad de las colas. El resto del dominio es ruido con buena presentación.

### Desacuerdo 3: prueba unitaria contra prueba de integración

Pregunta: ¿dónde debe vivir el esfuerzo de verificación?

El argumento unitario defiende probar cada pieza aislada, porque el fallo señala exactamente dónde está el defecto. El argumento de integración responde que los sistemas fallan en las costuras: la pieza probada se conecta mal con su vecina.

Consenso operativo: propiedades unitarias sobre las funciones puras primero —ahí vive la decisión— y una verificación mínima de integración en el punto donde el error sería costoso: que el ejecutor escriba exactamente lo que el plan declara, ni más ni menos.

## Puente hacia la frontera

La idea de declarar propiedades en vez de enumerar casos tiene una continuación natural: las herramientas de property-based testing, como Hypothesis, donde el programador escribe la propiedad y un generador explora cientos de entradas buscando el contraejemplo. No hace falta instalarlas para aprender de su disciplina: la propiedad 4 de la regla de alerta ya es ese gesto, escrito a mano.

Más adelante, la verificación cambiará de objeto. Un pipeline genómico se verifica con propiedades de conservación —las unidades no se pierden, el orden no se invierte, el filtro no inventa moléculas—. Y un modelo clínico no se valida con `assert`: se valida con calibración, sensibilidad y sesgo, que es tema de la unidad de razonamiento cuantitativo. El `assert` es el piso, no el techo.

La pregunta madura no es "¿pasaron las pruebas?". Es:

```text
¿Qué propiedad queda verificada y cuál sigue abierta?
```

Esa pregunta, hecha con honestidad, es la diferencia entre una suite y una superstición.

## Evaluar si entendiste

1. ¿Qué diferencia hay entre una prueba y una propiedad?
2. ¿Por qué una prueba tautológica no puede fallar, y qué efecto tiene en una suite?
3. ¿Qué afirma la prueba que espera una excepción con vocabulario de dominio?
4. ¿Por qué la reproducibilidad es la primera propiedad que conviene verificar?
5. ¿Qué relación hay entre la humildad de Dijkstra sobre las pruebas y la interpretación clínica de un resultado negativo?
6. ¿Por qué la frontera exacta de un umbral merece dos casos y no uno?
7. ¿Qué verifica una propiedad de monotonicidad que ningún caso aislado puede verificar?
8. ¿Por qué la cobertura de líneas no certifica corrección?
9. ¿Cuándo una verificación de integración está justificada si las piezas ya tienen propiedades probadas?
10. ¿Qué cambia cuando el objeto verificado deja de ser una función y se convierte en un modelo?

## Vacíos de comprensión que debes vigilar

1. Escribir primero el código y después las pruebas que el código ya cumple. Esa suite certifica lo que existe, no lo que se prometió; escribir la propiedad primero invierte la relación de poder.
2. Creer que una suite verde cierra la discusión. Cierra la ejecución; la discusión sobre qué propiedades faltan sigue abierta.
3. Probar la excepción sin mirar su mensaje. Que la función falle no basta: debe fallar con el vocabulario que el contrato prometió, o el error se vuelve indescifrable aguas arriba.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma una función propia y escribe las cuatro propiedades del patrón: caso documentado, dato faltante, frontera, reproducibilidad. Ejecuta hasta que pasen sin mensaje.
2. **Segunda hora:** escribe a propósito una prueba tautológica y una prueba frágil; observa cómo ambas pasan sin verificar nada; bórralas y reescribe una sola propiedad que las reemplace.
3. **Tercera hora:** elige una regla de tu dominio y declara una propiedad de forma —monotonicidad, conservación, vocabulario acotado—; implementa su verificación con un barrido de valores.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *The assert statement*. Python 3.14.4 documentation. <https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement>
- Dijkstra, E. W. (1972). The humble programmer. *Communications of the ACM, 15*(10), 859-866. <https://doi.org/10.1145/355604.361591>
- Myers, G. J. (1979). *The Art of Software Testing*. John Wiley & Sons.
- National Academies of Sciences, Engineering, and Medicine. (2019). *Reproducibility and Replicability in Science*. National Academies Press. <https://doi.org/10.17226/25303>
- Hypothesis developers. (2026). *Hypothesis: property-based testing for Python*. <https://hypothesis.readthedocs.io/>

## Siguiente paso

Con esta sección se cierra la primera unidad: formalizar decisiones, expresarlas en Python, gobernar sus errores y verificar sus promesas. La pregunta que abre la siguiente unidad es anterior a todo eso y fácil de pasar por alto: ¿qué promete un dato? Un peso no es un número, una fecha no es un texto y una ausencia no es un cero. Tipos de datos para problemas biomédicos.
