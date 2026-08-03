# Variables, datos y decisiones

Una variable no es solamente un nombre para un valor. Esa definición sirve para comenzar, pero se queda corta. En un programa biomédico, una variable es una decisión de representación: selecciona una parte del mundo, la recorta, le asigna una unidad, la vuelve manipulable y permite que participe en una regla.

El principiante pregunta: "¿cómo nombro esta variable?". El experto pregunta algo más difícil: "¿qué estoy autorizando a existir dentro del sistema cuando nombro esto así?".

Esa diferencia parece sutil hasta que una variable alimenta una decisión. `temperatura = 38.4` puede parecer inocente. Pero antes de usarla para clasificar fiebre hay preguntas inevitables: ¿en qué unidad está?, ¿cómo fue medida?, ¿en qué sitio anatómico?, ¿cuándo?, ¿con qué instrumento?, ¿en qué contexto clínico?, ¿qué pasa si el dato falta?, ¿qué umbral se aplicará?, ¿a qué conducta conduce?

La variable es el punto donde el mundo entra al algoritmo. Si entra mal, el algoritmo puede ser elegante y aun así decidir pobremente.

## Breve historia: medir para poder razonar

La historia de las variables no empieza en Python. Empieza en una necesidad más antigua: convertir fenómenos en símbolos manipulables.

En álgebra, una letra permitió razonar sobre cantidades sin conocer todavía su valor específico. En estadística, una variable permitió estudiar propiedades que cambian entre individuos, mediciones o eventos. En ciencias de la medición, S. S. Stevens propuso en 1946 una clasificación influyente de escalas nominales, ordinales, de intervalo y de razón. Esa clasificación no es un detalle académico: recuerda que no todas las variables admiten las mismas operaciones. No se promedia igual una categoría diagnóstica que una presión arterial.

Con la computación, la variable adquirió otro papel: dejó de ser solo símbolo matemático y se convirtió en posición nombrada dentro de un procedimiento ejecutable. En bases de datos y ciencia de datos, la variable también se volvió columna, atributo, campo, característica, predictor o rasgo. En medicina digital, una variable puede ser un signo vital, un diagnóstico codificado, una imagen, una secuencia genética, una nota clínica o una señal fisiológica.

El dato curioso es que la misma palabra, variable, puede significar cosas distintas según el campo: en álgebra puede representar una incógnita; en estadística, una característica que varía; en programación, un nombre asociado a un objeto; en machine learning, una característica de entrada; en epidemiología, una exposición, desenlace o confusor. Un lector serio debe aprender a moverse entre esos sentidos sin confundirlos.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Una variable es una representación nombrada de un valor, estado, medición o categoría que un procedimiento puede almacenar, transformar, comparar o usar para decidir.
</div>

La definición tiene varias capas.

**Nombre.** La variable debe poder referirse con claridad. `pas` puede ser rápido de escribir, pero `presion_arterial_sistolica_mmHg` reduce ambigüedad.

**Valor.** La variable conserva algo: número, texto, booleano, lista, diccionario, tabla, matriz, señal o ausencia explícita.

**Tipo.** El valor pertenece a una forma computacional. Un entero, un decimal, una cadena de texto y un booleano no se comportan igual.

**Unidad.** Muchas variables biomédicas pierden sentido sin unidad: mg/dL, mmol/L, mmHg, latidos por minuto, años, días, mililitros por minuto por 1.73 m².

**Contexto.** La misma cifra puede significar cosas distintas según edad, método de medición, tiempo, condición basal, población y objetivo.

**Uso.** Una variable se vuelve importante cuando participa en una decisión: comparación, filtro, cálculo, clasificación, alerta, modelo o reporte.

## De dato a decisión

Observa esta miniatura:

```python
# Entradas: dos signos vitales representados como variables.
presion_sistolica = 88
frecuencia_respiratoria = 32

# Decisión derivada: combina dos comparaciones.
alto_riesgo = presion_sistolica < 90 or frecuencia_respiratoria > 30
```

Salida esperada: no imprime nada. `alto_riesgo` queda con valor `True`.

El código parece simple. Pero ya contiene una cadena conceptual:

1. se midieron dos signos vitales;
2. se representaron como variables;
3. se compararon con umbrales;
4. se combinaron con una operación lógica;
5. se produjo una clasificación.

La clasificación `alto_riesgo` no estaba dentro del dato. Apareció por una arquitectura de decisión. Esa arquitectura puede ser razonable, incompleta, peligrosa, útil o equivocada según el contexto.

## Modelos mentales expertos

### 1. La variable como frontera entre mundo y modelo

Una variable no copia el mundo; lo representa. Toda representación conserva algo y descarta algo. Cuando guardamos `temperatura_celsius = 38.4`, conservamos una medición numérica y descartamos método, sitio, hora, trayectoria, fármacos y contexto clínico, salvo que los modelemos por separado.

Este modelo mental cambia una decisión médica real: una alerta de fiebre en un paciente neutropénico no debería depender solo de una cifra aislada. Debe incorporar contexto inmunológico, tiempo, síntomas y confiabilidad de la medición.

### 2. El tipo de dato limita el razonamiento posible

No todo valor permite las mismas operaciones. Una categoría ordinal puede ordenarse, pero no necesariamente promediarse de forma significativa. Una variable booleana puede simplificar una decisión, pero también destruir gradaciones importantes.

Ejemplo: convertir dolor en `dolor_presente = True` puede ser suficiente para activar una pregunta adicional, pero es pobre para seguimiento. Una escala de 0 a 10 conserva más información, aunque también exige interpretación y contexto.

### 3. El dato faltante es un estado, no un cero

En dominios biomédicos, ausencia de dato no equivale a normalidad. Un sistema que interpreta creatinina faltante como creatinina normal puede producir una recomendación farmacológica peligrosa.

El modelo experto exige representar explícitamente estados como `no_medido`, `pendiente`, `no_aplica`, `dato_invalido` o `evaluacion_incompleta`. Estos estados no son molestias técnicas. Son protección epistemológica.

### 4. El nombre de una variable es una hipótesis de lectura

Nombrar no es decorar. El nombre guía la interpretación del lector y del futuro programador. `riesgo = True` es débil porque no dice riesgo de qué, en qué horizonte ni bajo qué criterio. `alto_riesgo_deterioro_24h = True` comunica mucho más.

En software clínico, el nombre puede ser parte de la seguridad. Un nombre ambiguo aumenta carga cognitiva y facilita errores.

### 5. Una variable útil debe poder auditarse

Una variable que alimenta una decisión debería poder responder: de dónde vino, cuándo se midió, cómo se transformó, qué unidad usa, qué versión del cálculo la produjo y qué límites tiene.

Ejemplo: `egfr = 52` es menos auditable que `filtracion_glomerular_estimacion_ckd_epi_2021 = 52`. El segundo nombre no resuelve todos los problemas, pero deja una pista sobre el método.

## Dónde se solapan

Estos modelos se refuerzan mutuamente. Entender la variable como frontera entre mundo y modelo obliga a cuidar tipo, unidad y contexto. Reconocer el dato faltante como estado exige nombrarlo. Nombrar bien facilita auditoría. Auditar muestra si la representación fue suficiente para la decisión.

El meta-modelo es este:

> Una variable biomédica no es un dato aislado; es una promesa de representación bajo condiciones explícitas.

## Dónde crean tensión

También hay tensiones reales.

Una variable muy precisa puede ser más segura, pero más incómoda. Un nombre largo puede reducir ambigüedad y al mismo tiempo volver el código más pesado. Una representación simple puede ser más robusta y explicable, pero perder matices clínicos. Una representación rica puede preservar contexto, pero volver el sistema difícil de validar.

La madurez no consiste en hacer variables largas por orgullo técnico. Consiste en escoger la representación mínima que conserva lo necesario para decidir sin fingir precisión que no existe.

## Ejemplo: dato faltante, razón y trazabilidad

Comparemos dos formas de clasificar riesgo.

Versión pobre:

```python
# Entradas: una medición disponible y una ausente.
presion_sistolica = 88
saturacion_oxigeno = None

# Esta línea falla porque None no puede compararse con 90.
alto_riesgo = presion_sistolica < 90 or saturacion_oxigeno < 90
```

Salida esperada:

```text
TypeError: '<' not supported between instances of 'NoneType' and 'int'
```

Este código falla porque intenta comparar `None` con un número. Pero incluso si el lenguaje lo permitiera, habría un problema conceptual: la saturación no es normal; está ausente.

Una versión más responsable:

```python
# Entradas: una medición disponible y una ausente.
presion_sistolica = 88
saturacion_oxigeno = None

# Razones y pendientes se guardan por separado.
razones = []
datos_pendientes = []

if presion_sistolica is None:
    datos_pendientes.append("presion_sistolica")
elif presion_sistolica < 90:
    razones.append("hipotension")

if saturacion_oxigeno is None:
    datos_pendientes.append("saturacion_oxigeno")
elif saturacion_oxigeno < 90:
    razones.append("hipoxemia")

# La clasificación distingue riesgo, incompletitud y ausencia de criterios.
if razones:
    clasificacion = "alto_riesgo"
elif datos_pendientes:
    clasificacion = "evaluacion_incompleta"
else:
    clasificacion = "sin_criterios_de_alto_riesgo"

print(clasificacion)
print("razones:", razones)
print("pendientes:", datos_pendientes)
```

Salida esperada:

```text
alto_riesgo
razones: ['hipotension']
pendientes: ['saturacion_oxigeno']
```

La diferencia no es solo técnica. La segunda versión distingue tres cosas:

- razones presentes;
- datos pendientes;
- ausencia de criterios detectados.

Esa distinción será fundamental en sistemas clínicos, investigación reproducible y análisis biomédico.

## Variables, tablas y ciencia de datos

Cuando pasamos de un paciente a muchos, las variables suelen aparecer como columnas.

```text
paciente_id | edad | presion_sistolica | saturacion_oxigeno | clasificacion
001         | 67   | 88                | 89                 | alto_riesgo
002         | 54   | 122               |                    | evaluacion_incompleta
003         | 80   | 135               | 96                 | sin_criterios
```

La ciencia de datos moderna insiste en que la organización de datos no es secundaria. En la formulación de *tidy data* propuesta por Wickham, la disciplina inicial consiste en separar variables, observaciones y unidades observacionales de manera sistemática. No siempre será suficiente para problemas complejos, pero es una regla de organización poderosa para empezar a pensar datos tabulares.

En biomedicina, sin embargo, las tablas simples se quedan cortas rápidamente. Una imagen es una matriz o tensor. Una señal electrofisiológica es una serie temporal. Una red de interacciones puede ser un grafo. Una secuencia genética es una cadena con estructura, índices y anotaciones. Por eso este libro no se quedará en variables escalares: las usará como puerta de entrada a estructuras de datos.

## Argumentos críticos

### Desacuerdo 1: variables simples contra representaciones ricas

Pregunta: ¿conviene representar un fenómeno complejo con pocas variables simples o con estructuras más ricas?

El argumento por simplicidad es fuerte: las variables simples son más fáciles de auditar, enseñar, validar y mantener. El argumento por riqueza también es fuerte: muchos fenómenos biomédicos pierden su estructura al comprimirse demasiado.

Este debate importa porque un modelo simple puede ser honesto y útil, pero también puede ser ciego. Un modelo rico puede capturar más contexto, pero volverse frágil, opaco o difícil de validar.

Consenso operativo: comenzar simple no significa quedarse simple. La representación debe crecer cuando el problema lo exige y cuando hay capacidad de validarla.

### Desacuerdo 2: dato faltante como exclusión contra dato faltante como información

Pregunta: ¿qué debe hacer un sistema con datos faltantes?

Una posición elimina registros incompletos para simplificar el análisis. Otra trata la ausencia como información: si un laboratorio no fue solicitado, si no se registró una presión arterial o si un dato no aplica, esa ausencia puede reflejar flujo clínico, gravedad, recursos, sesgo o error de captura.

El debate importa porque las decisiones automatizadas pueden castigar poblaciones con registros incompletos o interpretar silenciosamente la ausencia como normalidad.

Consenso emergente: el dato faltante debe ser explícito, analizado y justificado. No debe desaparecer por comodidad.

### Desacuerdo 3: nombre breve contra nombre semántico

Pregunta: ¿deben las variables ser cortas para escribir rápido o largas para comunicar mejor?

La brevedad reduce ruido visual en algoritmos matemáticos y bucles pequeños. La semántica explícita reduce ambigüedad en dominios de alto riesgo. En software biomédico, el costo de un nombre largo suele ser menor que el costo de una interpretación incorrecta.

Consenso operativo: usar nombres breves solo cuando el contexto es local y obvio. Usar nombres semánticos cuando la variable cruza funciones, archivos, decisiones o dominios.

## Puente hacia la frontera

La discusión sobre variables parece básica hasta que llegamos a la frontera.

En bioinformática, representar una secuencia como texto plano puede servir para comenzar, pero los algoritmos de alineamiento, búsqueda e indexación necesitan estructuras más sofisticadas. En genética, una variante no es solo una posición y una letra: implica referencia, coordenada, gen, transcrito, efecto, frecuencia poblacional y evidencia clínica. En neurología computacional, una señal no es una lista de números: tiene frecuencia de muestreo, filtros, artefactos, ventanas temporales y estados fisiológicos.

La pregunta será siempre la misma:

> ¿Qué representación conserva la estructura relevante del fenómeno sin volver imposible el análisis?

## Límites de esta miniatura

Las variables usadas aquí son simplificaciones pedagógicas. En sistemas reales, un dato biomédico necesita fuente, unidad, fecha, contexto, método de medición y límites de interpretación. La sección enseña el acto de nombrar y representar, no un esquema completo de historia clínica.

## Evaluar si entendiste

Estas preguntas buscan criterio, no memoria. Están ordenadas de mayor a menor dificultad.

1. ¿Cómo diseñarías variables para representar "riesgo de deterioro" sin confundir dato observado, interpretación y conducta?
2. ¿Qué daño puede producir un sistema que convierte datos faltantes en valores normales?
3. ¿Cuándo una variable booleana es una simplificación legítima y cuándo destruye información importante?
4. ¿Qué diferencia hay entre una variable, una columna de tabla y una característica de modelo?
5. ¿Cómo representarías la fiebre en un paciente inmunosuprimido sin depender solo de `temperatura >= 38`?
6. ¿Qué información mínima debería acompañar una variable de laboratorio para ser auditable?
7. ¿Por qué una categoría ordinal no debe tratarse automáticamente como número continuo?
8. ¿Cómo cambia el diseño de variables cuando pasas de un paciente individual a una cohorte?
9. ¿Qué nombre sería más seguro: `riesgo`, `score`, `alto_riesgo` o `alto_riesgo_deterioro_24h`, y por qué?
10. ¿Por qué una imagen médica, una secuencia genética y una señal neurológica no deberían representarse como si fueran simples listas sin contexto?

## Vacíos de comprensión que debes vigilar

1. Confundir dato con decisión. Guardar una cifra no significa haber definido qué hacer con ella.
2. Confundir ausencia con normalidad. Lo no medido no debe entrar silenciosamente como resultado tranquilizador.
3. Confundir nombre con decoración. En dominios de consecuencias, nombrar bien es una forma de control.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma un conjunto pequeño de signos vitales y escribe variables con nombres explícitos, unidad y contexto.
2. **Segunda hora:** reescribe el ejemplo de riesgo agregando tres estados: dato presente, dato faltante y dato inválido.
3. **Tercera hora:** convierte los datos de tres pacientes en una tabla y define qué columnas son observaciones, cuáles son interpretaciones y cuáles son decisiones.

## Bibliografía y fuentes

- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://doi.org/10.17226/21794>
- Obermeyer, Z., Powers, B., Vogeli, C., & Mullainathan, S. (2019). Dissecting racial bias in an algorithm used to manage the health of populations. *Science, 366*(6464), 447-453. <https://doi.org/10.1126/science.aax2342>
- Sittig, D. F., & Singh, H. (2010). A new sociotechnical model for studying health information technology in complex adaptive healthcare systems. *Quality & Safety in Health Care, 19*(Suppl. 3), i68-i74. <https://doi.org/10.1136/qshc.2010.042085>
- Stevens, S. S. (1946). On the theory of scales of measurement. *Science, 103*(2684), 677-680. <https://doi.org/10.1126/science.103.2684.677>
- Wickham, H. (2014). Tidy data. *Journal of Statistical Software, 59*(10), 1-23. <https://doi.org/10.18637/jss.v059.i10>
- Wilkinson, M. D., et al. (2016). The FAIR Guiding Principles for scientific data management and stewardship. *Scientific Data, 3*, 160018. <https://doi.org/10.1038/sdata.2016.18>

## Siguiente paso

Ahora podemos avanzar hacia condiciones, umbrales y estados. Si una variable permite representar una parte del mundo, una condición decide cuándo esa representación cambia el curso del procedimiento.
