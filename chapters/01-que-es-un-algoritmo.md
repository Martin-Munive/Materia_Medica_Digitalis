# Qué es un algoritmo

La palabra algoritmo suele presentarse con una definición demasiado breve: una serie finita de pasos para resolver un problema. La frase no es falsa, pero es insuficiente. Sirve como puerta de entrada; no sirve como concepto rector para medicina, ciencia de datos, bioinformática, medicina digital o software clínico.

En un dominio trivial, un algoritmo puede parecer una receta. En un dominio serio, un algoritmo es una arquitectura de decisión: define qué información importa, cómo se representa, qué transformaciones son válidas, qué condiciones cambian el curso de acción, qué errores deben anticiparse y cuándo el procedimiento debe detenerse.

Este capítulo no busca que memorices una definición. Busca que aprendas a mirar cualquier algoritmo como una pieza de pensamiento formal: una forma de domesticar una pregunta hasta volverla ejecutable, verificable y criticable.

## Breve historia de una idea antigua

La computación moderna hizo famosa la palabra algoritmo, pero la idea es mucho más antigua que el computador.

Uno de los ejemplos clásicos es el algoritmo de Euclides para encontrar el máximo común divisor de dos números. Aparece en los *Elementos*, alrededor del 300 a. C., y sigue siendo una pieza viva de la matemática computacional. Su fuerza no está solo en que calcula un resultado; está en que muestra algo profundo: un problema puede resolverse por reducción sucesiva, transformando una pregunta difícil en otra más pequeña hasta que aparece una condición de parada.

Muchos siglos después, el término algoritmo quedó asociado al nombre de Muhammad ibn Musa al-Khwarizmi, matemático del siglo IX. La palabra procede de formas latinizadas de su nombre usadas en textos medievales sobre cálculo con numerales hindú-arábigos. El dato curioso importa: "algoritmo" no nació como una palabra de Silicon Valley, ni siquiera como una palabra de computación electrónica. Nació ligado a la transmisión cultural de técnicas de cálculo.

En el siglo XIX, Charles Babbage imaginó la Máquina Analítica, un dispositivo mecánico programable que nunca llegó a construirse completamente. Ada Lovelace, al comentar ese proyecto, entendió algo decisivo: una máquina de cálculo podía manipular símbolos de acuerdo con reglas, no solo números como cantidades. Sus notas sobre la Máquina Analítica incluyeron procedimientos para calcular números de Bernoulli y anticiparon la separación entre máquina, datos e instrucciones.

En 1936, Alan Turing llevó la pregunta a otro nivel. No se limitó a preguntar cómo calcular algo, sino que formalizó qué significa que un procedimiento sea mecánico. La máquina de Turing no fue importante por ser una máquina física, sino por convertir la intuición de "método efectivo" en un objeto matemático. Desde ahí, la pregunta por los algoritmos quedó conectada con una frontera: hay problemas que ningún procedimiento general puede decidir.

La historia deja una lección para este libro: un algoritmo no es simplemente una lista de pasos. Es una forma histórica, matemática y técnica de responder una pregunta: ¿qué parte del razonamiento puede hacerse explícita?

## Definición formal de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un algoritmo es una especificación finita, ordenada y verificable de transformaciones y decisiones que convierte entradas en salidas bajo un conjunto explícito de reglas, restricciones y criterios de terminación.
</div>

La definición tiene varias piezas.

**Entrada.** Algo ingresa al sistema: números, texto, signos vitales, imágenes, secuencias genéticas, resultados de laboratorio, grafos, síntomas, historiales o estados previos.

**Representacion.** La entrada debe adoptar una forma manipulable: variables, listas, tablas, matrices, diccionarios, grafos, tensores, cadenas de caracteres o estructuras especializadas.

**Transformacion.** El algoritmo opera sobre esa representación: compara, filtra, busca, ordena, suma, estima, clasifica, alinea, optimiza, infiere o simula.

**Control.** No basta con tener operaciones. Debe existir una arquitectura de flujo: condiciones, ramas, ciclos, recursión, excepciones y criterios de parada.

**Salida.** El procedimiento produce algo: una clasificación, alerta, recomendación, cálculo, reporte, ruta, predicción, visualización o acción.

**Terminación.** El algoritmo debe saber cuándo detenerse o declarar que no puede continuar bajo las condiciones dadas.

**Verificación.** La salida debe poder examinarse. Si no podemos revisar por qué apareció, el procedimiento puede funcionar como maquinaria, pero no como conocimiento confiable.

Estas capas impiden una confusión frecuente: creer que el código es el algoritmo. El código es una encarnación posible. El algoritmo es la estructura intelectual que el código ejecuta.

## Algoritmo, programa, modelo y decisión

Cuatro palabras se confunden con facilidad.

Un **algoritmo** define un procedimiento. Un **programa** implementa ese procedimiento en un lenguaje concreto. Un **modelo** decide qué aspectos del mundo se representan y cuáles se ignoran. Una **decisión** define por qué la transformación importa.

La diferencia no es académica. En medicina, un programa puede estar bien escrito y aun así representar un modelo pobre. Un modelo puede ser estadísticamente elegante y aun así apoyar una decisión equivocada. Una decisión puede ser razonable para población general y peligrosa para un subgrupo.

Por eso la pregunta central no es: "¿corre el código?". La pregunta madura es:

> ¿Qué decisión, bajo qué representación del mundo, está intentando ejecutar este algoritmo?

## Modelos mentales expertos

### 1. El algoritmo como contrato

Un algoritmo establece un contrato entre entrada, procedimiento y salida. Si la entrada cumple ciertas condiciones, el procedimiento promete una salida bajo reglas específicas. El contrato también define sus límites: qué no sabe manejar, qué datos requiere y cuándo debe detenerse.

En medicina, esto cambia una decisión real. Si un score de riesgo exige creatinina y edad, no se debe producir una recomendación completa cuando esos datos faltan. El contrato está incompleto.

### 2. El algoritmo como compresión de una decisión

Un buen algoritmo comprime una decisión compleja en una forma operable. Esa compresión no es neutral: conserva ciertas variables y descarta otras.

Ejemplo: clasificar fiebre por temperatura conserva un umbral y descarta contexto. Puede ser útil, pero no equivale a evaluar sepsis, inmunosupresión o trayectoria clínica.

### 3. El algoritmo como estructura de representación

Antes de calcular, el algoritmo decide cómo existe el problema dentro del sistema. Una secuencia genómica puede ser una cadena; una red metabólica puede ser un grafo; una imagen médica puede ser una matriz; una historia clínica puede ser texto estructurado.

El experto no pregunta solo "¿qué algoritmo uso?", sino "¿qué representación hace visible el fenómeno que quiero estudiar?".

### 4. El algoritmo como máquina de errores posibles

Cada algoritmo abre una familia de errores. Puede fallar por entrada incorrecta, variable mal definida, regla insuficiente, sesgo de datos, mala calibración, excepción no contemplada o salida mal interpretada.

En dominios biomédicos, esta idea es central: un algoritmo no se evalúa solo por aciertos promedio, sino por el tipo de error que puede producir y por las consecuencias de ese error.

### 5. El algoritmo como puente entre dominio y ejecución

Un algoritmo serio no vive solo en la matemática ni solo en el código. Vive entre un dominio y una máquina. Su calidad depende de traducir correctamente conocimiento del dominio a operaciones ejecutables.

Ese puente es la razón de este libro. El médico que aprende algoritmos no está aprendiendo "a programar por programar"; está aprendiendo a hacer explícito un razonamiento en un medio que exige precisión.

## Un ejemplo deliberadamente pequeño

Supongamos que queremos clasificar una temperatura corporal.

```python
# Entrada: una medición de temperatura en grados Celsius.
temperatura_celsius = 38.4

# Regla mínima: comparar la medición con un umbral.
if temperatura_celsius >= 38.0:
    estado_termico = "fiebre"
else:
    estado_termico = "sin fiebre"

# Salida visible del procedimiento.
print(estado_termico)
```

Salida esperada:

```text
fiebre
```

Este fragmento es simple, pero contiene la anatomía de un algoritmo.

Hay una entrada: `temperatura_celsius`. Hay una regla: comparar con `38.0`. Hay una bifurcación: fiebre o no fiebre. Hay una salida: `estado_termico`. Y hay una decisión conceptual escondida: aceptar un umbral.

El programa no sabe medicina. No sabe si la medición fue axilar, oral, timpánica o rectal. No sabe la edad del paciente, el contexto clínico, el uso de antipiréticos, el estado inmune ni la trayectoria temporal. Tampoco sabe si clasificar fiebre es suficiente o si la pregunta relevante era otra: riesgo de sepsis, indicación de aislamiento, necesidad de hemocultivos, prioridad de atención.

Ese es el punto. Un algoritmo no se vuelve inteligente por estar escrito en Python. Se vuelve útil cuando la estructura de decisión corresponde al problema real.

## De regla a modelo de decisión

Podemos volver el ejemplo menos ingenuo:

```python
# Entradas: signos vitales medidos.
temperatura_celsius = 38.4
frecuencia_cardiaca = 118
presion_sistolica = 88

# Variables intermedias: convierten números en criterios.
fiebre = temperatura_celsius >= 38.0
taquicardia = frecuencia_cardiaca >= 100
hipotension = presion_sistolica < 90

# Trazabilidad: conserva las razones que activan la decisión.
criterios_riesgo = []

if fiebre:
    criterios_riesgo.append("fiebre")

if taquicardia:
    criterios_riesgo.append("taquicardia")

if hipotension:
    criterios_riesgo.append("hipotension")

# Regla compuesta: combina criterios para producir una clasificación.
alto_riesgo = fiebre and (taquicardia or hipotension)

if alto_riesgo:
    conducta = "evaluacion_prioritaria"
else:
    conducta = "seguimiento_clinico"

# Salidas: conducta y razones.
print(conducta)
print(criterios_riesgo)
```

Salida esperada:

```text
evaluacion_prioritaria
['fiebre', 'taquicardia', 'hipotension']
```

Seguimos ante una miniatura, no ante una escala clínica validada. Pero la estructura ya cambió. El algoritmo no clasifica un dato aislado; integra variables, crea conceptos intermedios, conserva razones y produce una conducta.

Esta diferencia es fundamental. Muchos programas fallan no porque la sintaxis sea incorrecta, sino porque la decisión representada es pobre. El error profundo no está en el `if`; está en haber reducido un fenómeno complejo a una regla sin suficiente estructura.

## Correcto no significa suficiente

En ciencias de la computación, un algoritmo puede evaluarse por corrección: produce la salida esperada para las entradas definidas. En medicina y ciencias de la vida, esa evaluación es necesaria, pero incompleta.

Un algoritmo puede ser sintácticamente correcto y clínicamente irrelevante. Puede ser eficiente y biológicamente ingenuo. Puede ser reproducible y metodológicamente sesgado. Puede ser transparente y aun así operar sobre datos pobres. Puede acertar en promedio y fallar justo en los casos donde más importa.

Los informes sobre seguridad diagnóstica insisten en que diagnosticar no es una etiqueta aislada, sino un proceso complejo de recolección, integración e interpretación de información. Esta idea debe acompañar cualquier algoritmo clínico: una salida computacional no equivale a comprensión clínica.

## Tres tensiones críticas

### 1. Formalización vs juicio clínico

Formalizar permite auditar, programar y reproducir. Pero no todo juicio experto cabe en una regla simple. La tensión no se resuelve negando el juicio ni rechazando el algoritmo. Se resuelve definiendo dónde el algoritmo ayuda, dónde advierte y dónde debe dejar espacio a revisión humana.

### 2. Simplicidad vs fidelidad al fenómeno

Un algoritmo simple puede ser robusto, explicable y fácil de usar. Pero también puede borrar variables decisivas. Un algoritmo complejo puede capturar más estructura, pero volverse opaco, frágil o difícil de validar.

La pregunta no es si la simplicidad es buena o mala. La pregunta es qué costo paga el modelo por ser simple.

### 3. Automatización vs responsabilidad

Automatizar una decisión cambia quién ve el error, quién lo interpreta y quién responde por sus consecuencias. En biomedicina, el algoritmo no elimina responsabilidad; la redistribuye.

Por eso este libro no estudiará algoritmos como ornamentos matemáticos. Los tratará como instrumentos de conocimiento y acción.

## De aquí a la frontera

La misma idea de algoritmo que aparece en una regla de fiebre reaparece, con mayor sofisticación, en problemas de frontera.

En bioinformática, el alineamiento de secuencias usa programación dinámica para comparar ADN, ARN o proteínas. En genómica moderna, los volúmenes de datos exigen índices, grafos, modelos probabilísticos y aprendizaje automático. En neurología computacional, las señales y redes cerebrales requieren representaciones temporales, gráficas y dinámicas. En medicina de precisión, algoritmos y modelos convierten datos moleculares, clínicos e imagenológicos en hipótesis de estratificación y tratamiento.

La escalera del libro será esta: primero entenderemos qué es una decisión formalizable; después aprenderemos estructuras y algoritmos clásicos; luego veremos cómo esas herramientas se transforman cuando el objeto ya no es un número, sino un sistema vivo.

## Límites de esta miniatura

Los ejemplos de esta sección no son modelos clínicos ni sistemas de decisión asistencial. Sirven para distinguir algoritmo, programa, modelo y decisión. En capítulos posteriores, esa distinción deberá ampliarse con tipos de datos, validación, incertidumbre, pruebas y responsabilidad operacional.

## Evaluar si entendiste

Estas preguntas buscan distinguir comprensión de memorización.

1. ¿Por qué la definición "serie finita de pasos" es correcta pero insuficiente?
2. ¿En qué se diferencia un algoritmo de un programa?
3. ¿Qué parte del mundo queda fuera cuando representas fiebre como `temperatura >= 38.0`?
4. ¿Por qué una salida correcta puede ser clínicamente insuficiente?
5. ¿Cómo cambiaría el algoritmo de fiebre si el paciente fuera inmunosuprimido?
6. ¿Qué significa que un algoritmo tenga contrato?
7. ¿Qué errores nuevos aparecen cuando automatizas una decisión?
8. ¿Cómo se conecta el algoritmo de Euclides con la idea moderna de reducción iterativa de problemas?
9. ¿Por qué la máquina de Turing importa para entender los límites de los algoritmos?
10. ¿Cómo puede una misma idea algorítmica aparecer en triage, alineamiento genómico y redes biológicas?

## Bibliografía y fuentes

- Berger, B., Peng, J., & Singh, M. (2013). Computational solutions for omics data. *Nature Reviews Genetics, 14*, 333-346. <https://www.nature.com/articles/nrg3433>
- Britannica. (s. f.). *Al-Khwarizmi*. <https://www.britannica.com/biography/al-Khwarizmi>
- Britannica. (s. f.). *Algorithm*. <https://www.britannica.com/science/algorithm>
- Britannica. (s. f.). *Euclidean algorithm*. <https://www.britannica.com/science/Euclidean-algorithm>
- Computer History Museum. (s. f.). *A brief history: Babbage engine*. <https://www.computerhistory.org/babbage/history/>
- Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2022). *Introduction to algorithms* (4th ed.). MIT Press. <https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/>
- Eddy, S. R. (2004). What is dynamic programming? *Nature Biotechnology, 22*, 909-910. <https://doi.org/10.1038/nbt0704-909>
- Moor, M., et al. (2023). Foundation models for generalist medical artificial intelligence. *Nature, 616*, 259-265. <https://www.nature.com/articles/s41586-023-05881-4>
- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>
- Stanford Encyclopedia of Philosophy. (2021). *The Church-Turing thesis*. <https://plato.stanford.edu/archives/spr2021/entries/church-turing/>

## Siguiente paso

Ahora podemos estudiar cómo pensar en pasos. Si un algoritmo es una decisión formalizada, el siguiente problema es aprender a descomponer una situación compleja sin destruir lo importante.
