# De los datos a los algoritmos

El Capítulo II terminó con un lote biomédico capaz de atravesar validación, análisis y exportación sin perder sus rechazos ni su trazabilidad. Esa capacidad era necesaria, pero todavía dejaba una parte del trabajo sin examinar.

Cuando una función buscaba una fila, recorría una lista. Cuando resumía estados, acumulaba conteos. Cuando recuperaba una entidad, consultaba una clave. Esas operaciones ya eran algoritmos, aunque el libro las tratara como mecanismos auxiliares.

Ahora pasan al primer plano.

La pregunta deja de ser solo si los datos están bien representados. También debemos preguntar qué procedimiento transforma esa representación, cuánto trabajo realiza y bajo qué condiciones sigue siendo una buena elección.

## Una segunda lectura de algoritmo

En el Capítulo I definimos un algoritmo como una forma de hacer explícita una decisión. Esa definición sigue vigente. El avance consiste en observar que una misma decisión puede admitir varios procedimientos.

Supongamos que necesitamos localizar un evento por su identificador. Podemos:

- revisar los eventos uno por uno;
- ordenar los identificadores y dividir repetidamente el espacio de búsqueda;
- construir antes un diccionario que relacione cada identificador con su evento;
- delegar la recuperación a un índice de base de datos.

Las cuatro estrategias pueden encontrar el mismo evento. No hacen el mismo trabajo, no exigen las mismas precondiciones y no pagan el costo en el mismo momento.

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un problema computacional declara una familia de entradas, una salida esperada y las propiedades que conectan ambas. Un algoritmo es un procedimiento finito para resolver ese problema bajo precondiciones y costos que deben permanecer visibles.
</div>

La definición separa tres cosas que suelen mezclarse:

1. **El problema:** localizar un evento con un identificador dado.
2. **La instancia:** localizar `EV-104` dentro de cinco eventos concretos.
3. **El algoritmo:** recorrer desde el inicio hasta encontrarlo o agotar la colección.

Una ejecución resuelve una instancia. El algoritmo describe cómo resolver toda instancia compatible con su contrato.

## El registro biomédico sintético

Usaremos una cohorte oncológica educativa. Cada registro representa un evento, no una persona completa. Los identificadores son ficticios y las categorías no autorizan interpretación asistencial.

```python
eventos = [
    {
        "evento_id": "EV-101",
        "paciente_id": "PX-001",
        "dia": 1,
        "tipo": "administracion_tratamiento",
        "estado_dato": "validado",
    },
    {
        "evento_id": "EV-102",
        "paciente_id": "PX-002",
        "dia": 8,
        "tipo": "laboratorio_control",
        "estado_dato": "validado",
    },
    {
        "evento_id": "EV-103",
        "paciente_id": "PX-001",
        "dia": 12,
        "tipo": "sintoma_reportado",
        "estado_dato": "pendiente_revision",
    },
    {
        "evento_id": "EV-104",
        "paciente_id": "PX-003",
        "dia": 14,
        "tipo": "laboratorio_control",
        "estado_dato": "validado",
    },
    {
        "evento_id": "EV-105",
        "paciente_id": "PX-002",
        "dia": 21,
        "tipo": "seguimiento",
        "estado_dato": "validado",
    },
]
```

La lista conserva el orden de los eventos. Cada diccionario conserva campos nombrados. El contrato de datos viene antes del algoritmo:

- `evento_id` debe ser único;
- `paciente_id` es un identificador sintético, no un dato personal;
- `dia` es un desplazamiento pedagógico, no una fecha real;
- `tipo` pertenece a un vocabulario del ejercicio;
- `estado_dato` distingue validación de revisión pendiente.

Si `evento_id` no fuera único, la pregunta "encuentra el evento" estaría mal especificada. El algoritmo podría devolver la primera coincidencia, la última, todas o un error. Ninguna conducta es correcta hasta que el contrato la declare.

## Antes del código: formular el problema

Podemos escribir el contrato de búsqueda en lenguaje natural:

```text
Entrada:
  una lista de eventos con identificadores únicos
  un identificador objetivo no vacío

Salida:
  el evento y su posición si existe
  una ausencia explícita si no existe
  el número de comparaciones realizadas

Propiedades:
  no modifica la lista original
  nunca fabrica un evento
  distingue encontrado de no encontrado
  ante la misma entrada produce la misma salida
```

Esta formulación parece más lenta que empezar con un `for`. En realidad evita que decisiones importantes queden escondidas dentro del bucle.

## Versión ingenua: una respuesta sin contrato

```python
def buscar(eventos, objetivo):
    for evento in eventos:
        if evento["evento_id"] == objetivo:
            return evento
    return None


print(buscar(eventos, "EV-104"))
```

Salida esperada:

```text
{'evento_id': 'EV-104', 'paciente_id': 'PX-003', 'dia': 14, 'tipo': 'laboratorio_control', 'estado_dato': 'validado'}
```

La función es pequeña y, para el caso mostrado, correcta. Sin embargo, oculta preguntas que este capítulo necesita hacer visibles:

- no informa cuánto trabajo realizó;
- no valida si el objetivo está vacío;
- usa `None` sin declarar si significa ausencia, error o entrada inválida;
- no informa la posición;
- no declara su conducta ante identificadores duplicados;
- no permite comparar su costo con otra estrategia.

El problema no es que la función sea corta. El problema es que todavía no produce la evidencia que queremos estudiar.

## Versión trazable: resultado y trabajo realizado

```python
def localizar_evento(eventos, evento_id):
    """Localiza un evento y hace visible el trabajo de la búsqueda."""
    if not isinstance(evento_id, str) or not evento_id.strip():
        return {
            "estado": "entrada_invalida",
            "evento": None,
            "posicion": None,
            "comparaciones": 0,
        }

    comparaciones = 0

    for posicion, evento in enumerate(eventos):
        comparaciones += 1
        if evento["evento_id"] == evento_id:
            return {
                "estado": "encontrado",
                "evento": evento,
                "posicion": posicion,
                "comparaciones": comparaciones,
            }

    return {
        "estado": "no_encontrado",
        "evento": None,
        "posicion": None,
        "comparaciones": comparaciones,
    }


resultado = localizar_evento(eventos, "EV-104")
print(resultado["estado"])
print(resultado["posicion"])
print(resultado["comparaciones"])
```

Salida esperada:

```text
encontrado
3
4
```

La función sigue recorriendo uno por uno. Aún no estamos estudiando en profundidad la búsqueda lineal; estamos aprendiendo a observarla.

El evento está en la posición `3`, porque Python indexa desde cero. La función realizó cuatro comparaciones: `EV-101`, `EV-102`, `EV-103` y `EV-104`. No necesitó revisar `EV-105` porque ya había satisfecho su condición de parada.

## Las cinco capas de una elección algorítmica

### 1. Pregunta

La pregunta debe especificar qué se busca. "Revisar los eventos" no alcanza. "Localizar el evento cuyo identificador es igual al objetivo" sí define una relación comprobable.

### 2. Representación

Los eventos viven en una lista de registros. Esa representación permite recorrido secuencial y acceso por posición, pero no promete recuperación inmediata por identificador.

### 3. Precondiciones

El procedimiento supone que cada evento tiene `evento_id` y que los identificadores son únicos. Si esas condiciones no se validaron antes, la salida puede ser ambigua o provocar una excepción técnica.

### 4. Propiedad de corrección

Si la función devuelve `encontrado`, el evento retornado debe tener exactamente el identificador solicitado. Si devuelve `no_encontrado`, ningún elemento de la lista debe tenerlo.

### 5. Costo

El número de comparaciones depende de la posición del objetivo. Puede ser una si está al inicio, cinco si está al final o cinco si no existe. Más adelante expresaremos este crecimiento mediante complejidad; por ahora basta con medir el trabajo real.

## CODE CLEAN: hacer visible sin contaminar

Agregar trazabilidad no significa imprimir desde cada iteración.

```python
def buscar_con_impresiones(eventos, objetivo):
    for evento in eventos:
        print("comparando", evento["evento_id"])
        if evento["evento_id"] == objetivo:
            return evento
    return None
```

Esta variante mezcla dos responsabilidades: buscar y decidir cómo mostrar el proceso. Las impresiones dificultan las pruebas y obligan a capturar consola si otra función necesita el conteo.

La versión mejorada devuelve `comparaciones` como dato. Una interfaz puede imprimirlo, una prueba puede comprobarlo y un experimento puede compararlo con otro algoritmo. La función decide; otra capa presenta.

## Pruebas mínimas

```python
# Propiedad 1: si encuentra, devuelve el identificador solicitado.
hallado = localizar_evento(eventos, "EV-102")
assert hallado["estado"] == "encontrado"
assert hallado["evento"]["evento_id"] == "EV-102"

# Propiedad 2: una ausencia no fabrica un registro.
ausente = localizar_evento(eventos, "EV-999")
assert ausente["estado"] == "no_encontrado"
assert ausente["evento"] is None

# Propiedad 3: una entrada inválida no recorre la colección.
invalido = localizar_evento(eventos, "")
assert invalido["estado"] == "entrada_invalida"
assert invalido["comparaciones"] == 0

# Propiedad 4: la función no modifica los eventos.
copia_antes = [evento.copy() for evento in eventos]
localizar_evento(eventos, "EV-104")
assert eventos == copia_antes

# Propiedad 5: un objetivo al final exige revisar toda la lista.
ultimo = localizar_evento(eventos, "EV-105")
assert ultimo["comparaciones"] == len(eventos)
```

Salida esperada: no imprime nada si todas las propiedades se cumplen.

Estas pruebas no demuestran que la estrategia sea la más eficiente para cualquier escala. Demuestran que la implementación cumple el contrato pedagógico declarado.

## Correcto, eficiente y apropiado no son sinónimos

**Correcto** significa que el procedimiento satisface sus propiedades para las entradas admitidas.

**Eficiente** significa que usa tiempo y memoria de manera adecuada para la escala considerada.

**Apropiado** agrega contexto: costo de construcción, frecuencia de uso, facilidad de mantenimiento, auditabilidad y consecuencias del error.

Para cinco eventos, recorrer la lista es simple y suficiente. Si la lista cambia una vez y se consulta una vez, construir un índice adicional puede costar más de lo que ahorra. Si el sistema recibe miles de consultas sobre millones de eventos, repetir el recorrido completo deja de ser razonable.

El mejor algoritmo no existe fuera de una situación. Existe una elección defendible para una representación, una escala y un patrón de uso.

## Costo computacional y costo biomédico

El costo computacional incluye comparaciones, tiempo, memoria, acceso a disco, transferencia de red y preparación de estructuras auxiliares.

El costo biomédico puede incluir retraso, omisión, clasificación errónea, pérdida de trazabilidad o una prioridad mal interpretada. Estos costos no se convierten automáticamente en una misma unidad.

Una búsqueda que tarda un segundo puede ser irrelevante en un informe mensual y excesiva en una interfaz usada miles de veces. Una optimización que elimina estados de error para ahorrar espacio puede ser técnicamente compacta y epistemológicamente peligrosa.

Por eso el capítulo no usará la velocidad como único criterio. Preguntará también qué supuesto se pagó para obtenerla.

## Errores frecuentes

1. **Programar antes de formular la pregunta.** El bucle termina definiendo accidentalmente el comportamiento.
2. **Confundir instancia con problema.** Que `EV-104` aparezca no demuestra cómo se comporta la función ante ausencia o duplicados.
3. **Optimizar un lote diminuto sin patrón de uso.** La estructura adicional puede aumentar complejidad sin beneficio real.
4. **Ocultar precondiciones.** La búsqueda binaria no es simplemente "más rápida"; necesita orden.
5. **Medir solo tiempo de reloj.** Una ejecución aislada depende del equipo, la carga y el tamaño concreto.
6. **Tratar `None` como explicación universal.** Entrada inválida y objetivo ausente son estados distintos.
7. **Confundir rapidez con seguridad.** Reducir operaciones no valida el dato ni la finalidad.

## Argumentos críticos

### Desacuerdo 1: usar bibliotecas o implementar desde cero

Pregunta: si Python ya ofrece búsqueda, ordenamiento, diccionarios y colas, ¿por qué implementar algoritmos?

Implementar una versión pequeña permite observar invariantes, condiciones de parada y costo. En producción, las implementaciones estándar suelen ser más probadas y optimizadas.

Consenso operativo: implementar para comprender; usar herramientas maduras cuando el objetivo sea construir software confiable, salvo que exista una razón medida para otra elección.

### Desacuerdo 2: devolver solo el resultado o también la traza

Pregunta: ¿no vuelve ruidosa la interfaz devolver estado, posición y comparaciones?

Una función de aplicación puede necesitar solo el evento. Una función pedagógica o de evaluación necesita evidencia del procedimiento.

Consenso operativo: devolver la trazabilidad proporcional al uso. No llenar toda función con telemetría, pero tampoco esconder la propiedad que se intenta estudiar.

### Desacuerdo 3: optimizar temprano o esperar evidencia

Pregunta: ¿debe construirse un índice desde el principio porque probablemente habrá muchos datos?

Anticipar escala evita rediseños obvios. Optimizar sin conocer volumen, frecuencia de consulta y costo de actualización puede fijar complejidad innecesaria.

Consenso operativo: diseñar contratos que permitan evolucionar; medir el patrón real; optimizar donde el costo sea material.

## Puente hacia frontera

La diferencia entre datos y algoritmos reaparece en toda la ciencia computacional:

- una secuencia genómica puede representarse como texto, pero alinearla exige definir similitud, penalizaciones y un procedimiento de optimización;
- una imagen puede representarse como matriz, pero segmentarla exige reglas, vecindades o modelos;
- una red de interacciones puede representarse como grafo, pero encontrar rutas o nodos centrales exige recorridos y métricas;
- una cohorte puede vivir en una tabla, pero recuperar, agrupar y priorizar eventos exige algoritmos e índices;
- un modelo fundacional puede ocultar algoritmos complejos, pero sigue dependiendo de representaciones, optimización, búsqueda y costo.

La frontera no elimina los fundamentos. Los apila.

## Preguntas de comprensión profunda

1. ¿Qué diferencia existe entre un problema, una instancia y un algoritmo?
2. ¿Por qué dos procedimientos correctos pueden no ser igualmente apropiados?
3. ¿Qué precondiciones asume `localizar_evento` aunque no las valide dentro del bucle?
4. ¿Qué demuestra el conteo de comparaciones que no demuestra el tiempo de reloj?
5. ¿Por qué devolver `None` puede borrar una distinción importante?
6. ¿Cuándo construir un índice podría costar más de lo que ahorra?
7. ¿Qué propiedad comprobaría que una búsqueda no fabrica resultados?
8. ¿Cómo se diferencia costo computacional de costo biomédico?
9. ¿Por qué una optimización puede reducir tiempo y empeorar trazabilidad?
10. ¿Qué cambia en la elección del algoritmo cuando la consulta se repite millones de veces?

## Vacíos de comprensión que debes vigilar

1. Creer que un algoritmo empieza en el `for`. Empieza en la formulación del problema y sus precondiciones.
2. Creer que medir comparaciones ya equivale a dominar complejidad. Es solo la primera observación del crecimiento.
3. Creer que una estructura más sofisticada siempre es mejor. También cuesta construirla, actualizarla y explicarla.
4. Creer que un resultado correcto valida el uso biomédico. Solo valida una propiedad técnica dentro del contrato probado.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** reescribe tres tareas vagas como problemas con entrada, salida, precondiciones y propiedades.
2. **Segunda hora:** ejecuta `localizar_evento` con objetivos al inicio, al final y ausentes; registra las comparaciones.
3. **Tercera hora:** crea una lista de cien eventos sintéticos y predice, antes de ejecutar, el mejor y el peor caso del recorrido.

## Bibliografía y fuentes

- Black, P. E. (ed.). (s. f.). *Dictionary of Algorithms and Data Structures*. National Institute of Standards and Technology. <https://www.nist.gov/dads/>.
- Black, P. E. (2016). *Search*. En *Dictionary of Algorithms and Data Structures*. National Institute of Standards and Technology. <https://www.nist.gov/dads/HTML/search.html>.
- Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2022). *Introduction to Algorithms* (4th ed.). MIT Press.
- Python Software Foundation. (s. f.). *Built-in Functions: enumerate*. <https://docs.python.org/3/library/functions.html#enumerate>.
- Python Software Foundation. (s. f.). *Data Structures*. <https://docs.python.org/3/tutorial/datastructures.html>.

## Siguiente paso

La próxima sección estudiará la **búsqueda lineal** con mayor precisión: mejor caso, peor caso, objetivo ausente, condición de parada, duplicados y propiedades que deben conservarse al recorrer registros biomédicos.
