# Funciones puras, efectos y coordinación de procesos

Algunas piezas de un programa se pueden probar con una sola línea: se llaman con una entrada y se compara la salida. Otras exigen preparar el mundo antes de probarlas: un teclado que responda, un archivo disponible, una pantalla donde mirar, una variable global en el estado exacto. La diferencia no está en el lenguaje ni en la experiencia de quien programa. Está en la relación que cada pieza tiene con el mundo exterior.

En la sección anterior, las excepciones gobernaron lo que un procedimiento no puede evitar. Aquí la pregunta es previa y más estructural: ¿qué partes del procedimiento deciden y qué partes actúan sobre el mundo? Mientras esas dos cosas vivan mezcladas en la misma función, cada prueba será una negociación con el entorno. Separarlas es lo que convierte la verificación —el tema de la siguiente sección— en un acto sencillo.

## Origen técnico: separar el razonamiento del mundo

La distinción viene de una tradición larga. Las matemáticas trabajan con funciones que solo relacionan entradas con salidas: la raíz de 4 es 2, sin importar el día, el lugar ni cuántas veces se pregunte. La computación heredó ese ideal y tuvo que reconciliarlo con una realidad: los programas útiles leen teclados, escriben archivos, envían señales y modifican estados. Un programa sin efectos no produce nada observable; un programa donde todo es efecto no produce nada verificable.

La respuesta que consolidó la programación funcional —y que lenguajes como Python adoptan como disciplina voluntaria, no como obligación— es tratar la pureza como una propiedad diseñable: acercar la mayor parte del código al ideal matemático y concentrar lo inevitablemente mundano en piezas pequeñas y visibles. En 1989, John Hughes defendió que esa separación no es estética: es lo que permite razonar sobre un programa pieza por pieza y componer esas piezas sin sorpresas.

En un contexto biomédico, la misma idea tiene un nombre familiar: reproducibilidad. Un cálculo científico debe dar el mismo resultado cuando se repite con los mismos datos. Eso es exactamente lo que una función pura garantiza por construcción.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una función pura es aquella cuya salida depende solo de sus entradas y que no produce efectos observables fuera de sí misma: no lee teclado, no imprime, no escribe archivos, no consulta el reloj ni modifica estado externo. Una función con efectos es aquella que sí interactúa con el mundo exterior. Una función de coordinación es una función pura que, en lugar de ejecutar efectos, construye la descripción de lo que debe ocurrir, para que otra pieza —un ejecutor delgado— la lleve a cabo.
</div>

La definición tiene tres consecuencias.

**La pureza se reconoce con dos preguntas.** ¿La salida cambia si llamo la función dos veces con la misma entrada? ¿La llamada deja alguna huella fuera de la función? Si ambas respuestas son no, la función es pura.

**Los efectos no son el enemigo.** Un programa sin efectos no imprime, no guarda y no informa. El objetivo no es eliminar los efectos; es ubicarlos en piezas delgadas y explícitas, lejos de las decisiones.

**La coordinación es una función más, con un trabajo distinto.** No toca el mundo: recibe datos, llama funciones puras y devuelve un plan —qué mostrar, qué registrar, qué notificar— que otra pieza ejecutará. Decidir y hacer quedan en funciones diferentes.

## Versión ingenua: la función que hace todo

Volvamos al cálculo del índice de masa corporal, ahora como una función completa que atiende un flujo de pacientes.

```python
# Registro global donde se acumulan los resultados calculados.
historial_imc = []

def procesar_paciente():
    # Efecto de entrada: lee el teclado.
    peso_kg = float(input("Peso en kg: "))
    estatura_m = float(input("Estatura en metros: "))

    # Decisión: cálculo y clasificación pedagógica.
    imc = peso_kg / estatura_m ** 2
    if imc < 18.5:
        categoria = "bajo_peso"
    elif imc < 25:
        categoria = "normal"
    elif imc < 30:
        categoria = "sobrepeso"
    else:
        categoria = "obesidad"

    # Efecto: modifica el estado global compartido.
    historial_imc.append(imc)

    # Efecto: escribe una línea en el archivo de registro.
    with open("registro_imc.txt", "a") as archivo:
        archivo.write(f"{imc:.2f},{categoria}\n")

    # Efecto: imprime el resultado en pantalla.
    print("IMC:", round(imc, 2))
    print("Categoría:", categoria)
    print("Pacientes procesados:", len(historial_imc))
```

Salida esperada, con entradas `70` y `1.75`, en la primera llamada:

```text
IMC: 22.86
Categoría: normal
Pacientes procesados: 1
```

El programa funciona. Pide los datos, calcula, registra y reporta. Pero intenta responder estas preguntas sin ejecutarlo: ¿qué categoría produce un peso de 70 con una estatura de 1.75? ¿Qué queda escrito en el archivo? ¿Cuántos pacientes dice llevar la segunda vez que corre? Cada respuesta exige montar un teatro: simular el teclado, crear un archivo temporal, reiniciar la variable global. La decisión —el corazón del programa— no se puede ver sin atravesar tres efectos.

## Crítica técnica: qué está mal

La versión anterior falla por cuatro razones.

Primero, la función no se puede probar sin simular el mundo. Verificar la categoría exige fingir el teclado, interceptar el archivo y controlar el estado global. La prueba deja de medir la decisión y pasa a medir el montaje.

Segundo, el resultado depende del orden de ejecución. `historial_imc` hace que la salida de la décima llamada dependa de las nueve anteriores. Misma entrada ya no significa misma salida, y sin esa propiedad no hay reproducibilidad posible.

Tercero, el estado compartido queda oculto. La firma `procesar_paciente()` no recibe nada y no promete nada; sin embargo, depende de una lista global y la modifica. Todo lo que una función necesita debería entrar por sus parámetros, y todo lo que produce debería salir por su retorno.

Cuarto, la decisión queda mezclada con la presentación. La categoría y el texto que la muestra nacen en el mismo lugar. Cambiar el formato del mensaje obliga a tocar la misma función que decide la categoría, y cada toque es una oportunidad de romper la regla.

CODE CLEAN aquí no agrega pruebas ni comentarios. Cambia la forma: separa decidir de hacer.

## Versión mejorada: pura, coordinación y ejecutor delgado

La misma operación se reescribe en tres piezas con tres trabajos distintos.

```python
def calcular_y_clasificar_imc(peso_kg, estatura_m, version_regla="imc_pedagogico_v0"):
    """Función pura: misma entrada produce siempre la misma salida.

    No lee teclado, no imprime, no escribe archivos ni toca estado externo.
    """
    if peso_kg <= 0:
        raise ValueError("peso_fuera_de_rango")
    if estatura_m <= 0 or estatura_m > 2.5:
        raise ValueError("estatura_fuera_de_rango")
    imc = peso_kg / estatura_m ** 2
    if imc < 18.5:
        categoria = "bajo_peso"
    elif imc < 25:
        categoria = "normal"
    elif imc < 30:
        categoria = "sobrepeso"
    else:
        categoria = "obesidad"
    return {
        "imc": round(imc, 2),
        "categoria": categoria,
        "version_regla": version_regla,
    }


def coordinar_registro_imc(peso_kg, estatura_m):
    """Coordinación: llama a la pura y devuelve el plan de efectos pendientes.

    También es pura: construye datos, no ejecuta nada.
    """
    resultado = calcular_y_clasificar_imc(peso_kg, estatura_m)
    return {
        "resultado": resultado,
        "linea_registro": f"{resultado['imc']:.2f},{resultado['categoria']}",
        "mensaje_pantalla": f"IMC: {resultado['imc']} · Categoría: {resultado['categoria']}",
    }


def ejecutar_registro_imc(plan):
    """Ejecutor delgado: la única pieza que toca el mundo exterior."""
    print(plan["mensaje_pantalla"])
    with open("registro_imc.txt", "a") as archivo:
        archivo.write(plan["linea_registro"] + "\n")
```

Uso:

```python
plan = coordinar_registro_imc(70, 1.75)
ejecutar_registro_imc(plan)
```

Salida esperada:

```text
IMC: 22.86 · Categoría: normal
```

Además, el archivo `registro_imc.txt` recibe la línea `22.86,normal`. Ese efecto es intencional y está aislado en una sola pieza.

La estructura cambió en cinco puntos. La decisión vive en una función pura que se prueba con una línea. La coordinación arma un plan legible: qué mensaje mostrar, qué línea registrar. El ejecutor tiene tan poca lógica que se puede leer de una vez; apenas hay algo que probar en él. El estado global desapareció: el historial puede reconstruirse leyendo el archivo o acumulando planes, pero ya no es una dependencia oculta. Y la presentación se separó de la decisión: cambiar el mensaje no toca la regla.

## Anatomía de la coordinación

La pieza nueva merece una mirada lenta.

```python
def coordinar_registro_imc(peso_kg, estatura_m):
    resultado = calcular_y_clasificar_imc(peso_kg, estatura_m)
    return {
        "resultado": resultado,
        "linea_registro": f"{resultado['imc']:.2f},{resultado['categoria']}",
        "mensaje_pantalla": f"IMC: {resultado['imc']} · Categoría: {resultado['categoria']}",
    }
```

La coordinación no imprime ni escribe: devuelve un diccionario que describe lo que debería pasar. Ese diccionario es un dato. Y como dato, se puede imprimir, comparar, guardar y probar sin ejecutar nada. La pregunta "¿qué habría hecho el programa con esta entrada?" tiene una respuesta inspeccionable: el plan.

El ejecutor, en cambio, casi no tiene contenido que examinar. Imprime lo que el plan dice, escribe lo que el plan dice. Toda la inteligencia está aguas arriba, en funciones que se pueden probar; toda la mecánica queda en el borde, donde se puede leer de un vistazo. Esa es la asimetría buscada: decisiones profundas y verificables, efectos delgados y visibles.

## Ejemplo biomédico progresivo: revisión de mediciones de laboratorio

El patrón escala sin cambiar de forma. Supongamos un lote de mediciones de laboratorio que debe revisarse con una regla pedagógica de alertas: cada medición se clasifica y cada clasificación dispara una acción distinta.

```python
# Cada medición trae el analito, el valor y los rangos pedagógicos de referencia.
mediciones = [
    {"analito": "glucosa_mg_dl", "valor": 210, "rango": (70, 140), "critico": (40, 400)},
    {"analito": "potasio_meq_l", "valor": 6.4, "rango": (3.5, 5.1), "critico": (2.5, 6.0)},
    {"analito": "hemoglobina_g_dl", "valor": None, "rango": (12, 16), "critico": (7, 20)},
]


def evaluar_medicion(valor, rango, critico):
    """Función pura: clasifica una medición según rangos pedagógicos."""
    if valor is None:
        return {"estado": "sin_dato", "accion": "solicitar_repeticion"}
    if valor < critico[0] or valor > critico[1]:
        return {"estado": "critico", "accion": "notificar_inmediato"}
    if valor < rango[0] or valor > rango[1]:
        return {"estado": "fuera_de_rango", "accion": "marcar_para_revision"}
    return {"estado": "en_rango", "accion": "registrar"}


def coordinar_revision(mediciones):
    """Coordinación: aplica la regla pura y arma el plan de acciones sin ejecutarlo."""
    plan = []
    for medicion in mediciones:
        decision = evaluar_medicion(medicion["valor"], medicion["rango"], medicion["critico"])
        plan.append({"analito": medicion["analito"], **decision})
    return plan


def ejecutar_revision(plan):
    """Ejecutor delgado: recorre el plan y produce los efectos."""
    for item in plan:
        print(f"{item['analito']}: {item['estado']} -> {item['accion']}")


plan = coordinar_revision(mediciones)
ejecutar_revision(plan)
```

Salida esperada:

```text
glucosa_mg_dl: fuera_de_rango -> marcar_para_revision
potasio_meq_l: critico -> notificar_inmediato
hemoglobina_g_dl: sin_dato -> solicitar_repeticion
```

La regla de alerta es una miniatura pedagógica, no una guía clínica. Lo que importa es la forma: la decisión clínica simulada es pura y se prueba con `assert` sin teclado, sin archivo y sin pantalla. El plan completo —qué analito quedó crítico, cuál sin dato, qué acción corresponde a cada uno— se verifica antes de que se ejecute ninguna notificación. En un sistema real, esa propiedad no es un lujo: es la diferencia entre auditar una decisión y adivinarla.

## Cinco principios para reconocer la forma

Con la práctica, la separación deja de verse como regla y se vuelve lectura directa del código. Cinco principios orientan esa lectura.

1. **Misma entrada, misma salida.** Si el resultado puede cambiar entre dos llamadas idénticas, algo externo está entrando por la puerta de atrás.
2. **Dependencia explícita.** Todo lo que la función necesita entra por los parámetros; todo lo que produce sale por el retorno.
3. **Decidir y hacer son trabajos distintos.** La función que decide no ejecuta; la que ejecuta no decide.
4. **Componer en vez de compartir.** Funciones pequeñas encadenadas por datos, no variables globales compartidas por convenio.
5. **El borde con efectos es delgado.** Los efectos se concentran en el perímetro del programa —entrada, salida, registro— y ese perímetro se mantiene tan simple que se verifica leyéndolo.

## Prueba mínima: probar la decisión sin tocar el mundo

La recompensa de la separación aparece aquí. Con las funciones de la versión mejorada:

```python
def calcular_y_clasificar_imc(peso_kg, estatura_m, version_regla="imc_pedagogico_v0"):
    if peso_kg <= 0:
        raise ValueError("peso_fuera_de_rango")
    if estatura_m <= 0 or estatura_m > 2.5:
        raise ValueError("estatura_fuera_de_rango")
    imc = peso_kg / estatura_m ** 2
    if imc < 18.5:
        categoria = "bajo_peso"
    elif imc < 25:
        categoria = "normal"
    elif imc < 30:
        categoria = "sobrepeso"
    else:
        categoria = "obesidad"
    return {"imc": round(imc, 2), "categoria": categoria, "version_regla": version_regla}


def coordinar_registro_imc(peso_kg, estatura_m):
    resultado = calcular_y_clasificar_imc(peso_kg, estatura_m)
    return {
        "resultado": resultado,
        "linea_registro": f"{resultado['imc']:.2f},{resultado['categoria']}",
        "mensaje_pantalla": f"IMC: {resultado['imc']} · Categoría: {resultado['categoria']}",
    }


# Caso normal: la decisión se verifica con una línea.
assert calcular_y_clasificar_imc(70, 1.75)["categoria"] == "normal"

# Borde: el umbral 25 separa "normal" de "sobrepeso".
assert calcular_y_clasificar_imc(100, 2.0)["categoria"] == "sobrepeso"  # imc = 25.0
assert calcular_y_clasificar_imc(97.5, 2.0)["categoria"] == "normal"    # imc = 24.375

# El plan de la coordinación se prueba sin ejecutar ningún efecto.
plan = coordinar_registro_imc(70, 1.75)
assert plan["linea_registro"] == "22.86,normal"
assert plan["mensaje_pantalla"] == "IMC: 22.86 · Categoría: normal"

# Reproducibilidad: misma entrada, misma salida.
assert calcular_y_clasificar_imc(70, 1.75) == calcular_y_clasificar_imc(70, 1.75)
```

Salida esperada: no imprime nada si las pruebas pasan.

Ninguna de estas pruebas simula teclado, archivo o pantalla, porque la decisión ya no depende de ellos. La prueba del plan merece una pausa: verifica que el mensaje y la línea de registro serán correctos *antes* de que exista el archivo. Eso es lo que la coordinación compra: el comportamiento del sistema se examina como un dato.

## Límites y errores frecuentes

1. **Confundir "no tiene `return`" con "no tiene efectos".** Una función que imprime y devuelve `None` sigue tocando el mundo; la impureza no depende de lo que retorna.
2. **Abusar de variables globales para compartir estado.** La firma parece inocente —no recibe nada— y la dependencia es total: la función lee y modifica algo que no se ve.
3. **Pureza de fachada.** Una función "pura" que llama a otra que imprime no es pura: la impureza se hereda por la cadena de llamadas.
4. **Creer que todo debe ser puro.** El programa útil necesita efectos. La disciplina consiste en ubicarlos, no en eliminarlos; un ejecutor delgado es una pieza legítima, no una falla de diseño.

## Argumentos críticos

### Desacuerdo 1: pureza estricta contra pragmatismo

Pregunta: ¿vale la pena extraer una función pura incluso para operaciones triviales?

El argumento estricto es que toda decisión merece ser probable, porque lo trivial deja de serlo cuando el sistema crece. El argumento pragmático es que extraer cuesta: un script de tres líneas no necesita arquitectura.

Consenso operativo: extraer cuando la decisión necesite prueba, trazabilidad o reutilización. Si la salida puede influir en una acción de cuidado, una investigación o una publicación, la extracción no es opcional.

### Desacuerdo 2: extracción contra complejidad

Pregunta: ¿cuántas capas justifica la separación?

Tres roles no significan diez archivos. En Python, un solo módulo con funciones claras suele bastar. Crear clases e interfaces para imitar la ceremonia de otros lenguajes agrega peso sin agregar verificación.

Consenso operativo: la medida correcta es la legibilidad de la cadena. `pura → coordinación → ejecutor` debe poder leerse de arriba abajo sin saltar entre archivos.

### Desacuerdo 3: efectos centralizados contra efectos distribuidos

Pregunta: ¿conviene concentrar los efectos en el borde o dejarlos cerca de donde se necesitan?

Distribuir parece cómodo: cada función se las arregla sola. Pero diluye la trazabilidad: deja de existir un solo lugar donde mirar qué salió del sistema.

Consenso operativo: centralizar los efectos en el borde y dejar puro el centro del programa. La auditoría de "qué escribió el sistema" se reduce a leer el ejecutor.

## Puente hacia la frontera

La separación entre decidir y hacer habilita tres capacidades que aparecerán después. Las pruebas automáticas —la siguiente sección— existen porque la decisión es pura: se verifica sin montar el mundo. La paralelización se vuelve segura cuando las funciones no comparten estado: aplicar la misma regla pura a un millón de secuencias genómicas no exige candados ni orden de llegada. Y la reproducibilidad científica —misma entrada, misma salida, misma versión de regla— deja de ser un deseo declarado y se convierte en una propiedad que se puede probar.

Un pipeline de genómica serio tiene esta forma: etapas puras que transforman datos, separadas por bordes delgados que leen y escriben. La pregunta madura no es "¿mi código tiene efectos?". Es:

```text
¿Qué parte de mi programa decide, qué parte actúa sobre el mundo y cómo se nota la diferencia?
```

La pureza no es pureza moral. Es la forma en que un programa declara dónde termina el razonamiento y dónde empieza el mundo.

## Evaluar si entendiste

1. ¿Qué dos preguntas permiten reconocer una función pura?
2. ¿Por qué una función que solo imprime y devuelve `None` no es pura?
3. ¿Qué diferencia hay entre una función de coordinación y un ejecutor?
4. ¿Por qué el plan que devuelve la coordinación se puede probar sin ejecutar efectos?
5. ¿Qué problema introduce una variable global compartida en una función que parece no recibir nada?
6. ¿Por qué "misma entrada, misma salida" es también una exigencia de la ciencia reproducible?
7. ¿Cuándo está justificado extraer una función pura y cuándo es ceremonia?
8. ¿Por qué la cadena de llamadas hereda la impureza?
9. ¿Qué ventaja tiene centralizar los efectos en el borde del programa?
10. ¿Cómo conecta esta separación con las pruebas de la siguiente sección?

## Vacíos de comprensión que debes vigilar

1. Creer que pura significa "que sí retorna algo". La pureza es ausencia de efectos y dependencia exclusiva de las entradas; el retorno es solo su forma habitual.
2. Probar la decisión a través del ejecutor. Si la prueba necesita simular el mundo, la extracción quedó a medias.
3. Dejar "solo un `print` de depuración" dentro de la función pura. Ese `print` convierte cada ejecución en una lectura de pantalla y rompe la promesa de reproducibilidad.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma una función propia que imprima, escriba archivos o use una variable global; sepárala en pura más ejecutor delgado sin cambiar su comportamiento observable.
2. **Segunda hora:** escribe tres `assert` sobre la función pura: un caso normal, un borde de umbral y una doble llamada que verifique reproducibilidad.
3. **Tercera hora:** implementa `coordinar_revision` con mediciones propias y prueba el plan completo —cada estado, cada acción— sin ejecutar ningún efecto.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *Functional Programming HOWTO*. Python 3.14.4 documentation. <https://docs.python.org/3/howto/functional.html>
- Python Software Foundation. (2026). *The global statement*. Python 3.14.4 documentation. <https://docs.python.org/3/reference/simple_stmts.html#the-global-statement>
- Hughes, J. (1989). Why Functional Programming Matters. *The Computer Journal, 32*(2), 98-107. <https://doi.org/10.1093/comjnl/32.2.98>
- National Academies of Sciences, Engineering, and Medicine. (2019). *Reproducibility and Replicability in Science*. National Academies Press. <https://doi.org/10.17226/25303>

## Siguiente paso

El siguiente tema es la verificación mínima del cálculo: qué propiedad verifica cada prueba, por qué un `assert` es el piso y no el techo, y cómo reconocer una prueba que pasa sin demostrar nada. Las funciones puras dieron algo que probar; las pruebas enseñan cómo saber si esa pieza funciona antes de que importe de verdad.
