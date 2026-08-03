# Pensar en pasos

Programar no empieza escribiendo código. Empieza haciendo una operación mucho más exigente: convertir una situación confusa en una secuencia de decisiones examinables.

Un principiante cree que pensar en pasos es enumerar acciones. Un experto sabe que la secuencia es solo la superficie. Debajo hay una arquitectura: que se observa primero, que se descarta, que se conserva, que se mide, que se interpreta, que se hace si aparece una excepción y que evidencia autoriza pasar al siguiente estado.

En medicina esto es evidente. Un médico no evalúa a un paciente como una masa indiferenciada de datos. Ordena. Pregunta por motivo de consulta, identifica signos de alarma, ubica antecedentes relevantes, examina, formula hipótesis, decide pruebas, interpreta resultados y actúa. Ese orden no es decorativo. Reduce incertidumbre, protege memoria de trabajo y evita que una decisión se adelante a la evidencia.

Un algoritmo hace algo parecido: transforma intuición en procedimiento revisable.

## Origen histórico: del método al procedimiento verificable

La idea de pensar en pasos no nació con Python ni con los computadores personales. Es mucho más antigua. El algoritmo de Euclides, descrito en *Elementos*, ya mostraba una intuición poderosa: un problema matemático podía resolverse mediante una secuencia finita de operaciones repetibles. Siglos después, Al-Juarismi sistematizó procedimientos aritméticos y algebraicos cuyo nombre latinizado terminó asociado a la palabra "algoritmo".

En el siglo XX, la pregunta se volvió más radical. Turing no solo preguntó cómo resolver un cálculo, sino qué tipo de procedimiento podía considerarse mecánicamente ejecutable. Esa transición importa porque separa dos niveles: una explicación humana puede ser convincente, pero un procedimiento computable debe ser lo bastante explícito para ejecutarse sin intuición tácita.

En medicina ocurre algo paralelo. Las listas de verificación quirúrgica de la OMS y los esquemas de triage no intentan reemplazar el juicio clínico; intentan convertir momentos de alto riesgo en secuencias visibles, auditables y menos dependientes de memoria, fatiga o improvisación. El dato histórico importante es este: la formalización por pasos no empobrece necesariamente el pensamiento experto. Bien diseñada, lo protege de sus propios puntos ciegos.

## El paso no es una frase: es una unidad de decisión

Una instrucción como esta parece clara:

```text
Evalúa si el paciente está grave.
```

Pero para un algoritmo no significa casi nada. No define variables, umbrales, excepciones, salida, prioridad ni criterio de terminación. Un humano podría apoyarse en experiencia tácita. Un programa no.

Una formulacion más algorítmica seria:

```text
1. Medir estado de conciencia.
2. Medir presion arterial sistolica.
3. Medir frecuencia respiratoria.
4. Medir saturacion de oxigeno.
5. Identificar signos de choque, dificultad respiratoria o alteracion neurologica.
6. Si algún criterio de emergencia está presente, clasificar como alto riesgo.
7. Si no hay criterio de emergencia, evaluar criterios de prioridad.
8. Registrar clasificacion y motivo.
```

Todavía falta validar los umbrales, ajustar poblaciones, definir excepciones y decidir qué escala usar. Pero el pensamiento ya cambió. La gravedad dejó de ser una impresión global y empezó a convertirse en una estructura.

## Definición de trabajo

Pensar en pasos no significa partir una tarea en frases pequeñas. Significa construir una trayectoria controlada desde un estado inicial hasta un estado final, especificando qué información se observa, qué transformaciones se permiten, qué condiciones autorizan avanzar, qué excepciones interrumpen el proceso y qué evidencia debe quedar registrada.

En un contexto biomédico, esa definición exige más que orden. Exige prioridad. No todas las preguntas valen lo mismo cuando el tiempo, el riesgo y la incertidumbre son desiguales.

## Modelos mentales expertos

### 1. Separar observación, interpretación y acción

El experto no mezcla dato bruto con decisión final. Primero distingue qué se observó, luego qué significa y finalmente qué conducta se justifica. Este modelo aparece en el proceso diagnóstico descrito por las National Academies, donde la información se recolecta, integra e interpreta antes de tomar decisiones clínicas, y también en ETAT, donde los signos de emergencia preceden a la intervención.

Ejemplo: una saturación de 88% no es por sí sola "neumonía grave". Es un dato. Su interpretación depende de contexto, técnica de medición, edad, enfermedad previa y signos asociados. La acción puede ser oxígeno, traslado, monitorización o búsqueda de causa.

### 2. Convertir ambigüedad en estados

Un sistema experto no intenta resolver todo en una sola frase. Define estados intermedios: no evaluado, estable, prioridad, emergencia, pendiente de dato, excepción. Esta forma de pensar conecta con la teoría algorítmica clásica, que exige estados, entrada, salida y terminación, y con la práctica clínica, que distingue pacientes no evaluados de pacientes realmente estables.

Ejemplo: un paciente sin presión arterial registrada no debe entrar automáticamente en "bajo riesgo". Debe entrar en un estado diferente: `dato_pendiente` o `evaluacion_incompleta`.

### 3. Priorizar riesgos antes que comodidad computacional

En salud, el orden de los pasos no se decide solo por elegancia técnica. Se decide por consecuencias. ETAT clasifica rápido a los niños con signos de emergencia porque el retraso cambia el desenlace. En cirugía, las listas de verificación colocan puntos críticos antes de la incisión y antes de salir del quirófano porque el momento de la pregunta modifica su valor.

Ejemplo: en una interfaz clínica, preguntar alergias antes de recomendar un medicamento no es una preferencia de formulario. Es una restricción de seguridad.

### 4. Hacer visibles las excepciones

El pensamiento ingenuo diseña el caso promedio. El pensamiento experto pregunta qué rompe la regla. En programación, una excepción no es un estorbo: es información sobre los límites del modelo. En diagnóstico, Croskerry ha insistido en que muchos errores no provienen de falta de datos, sino de fallas cognitivas, sesgos y heurísticas mal controladas.

Ejemplo: una regla para fiebre basada en `temperatura >= 38` puede fallar en inmunosuprimidos, ancianos, pacientes con antipiréticos o mediciones poco confiables. El algoritmo debe permitir advertir ese límite.

### 5. Diseñar para revisión

Un procedimiento serio deja huellas. Si una clasificación aparece, debe poder responder: qué datos la produjeron, qué regla se aplicó, qué excepciones se ignoraron, qué paso faltó y qué versión del criterio estaba vigente.

Ejemplo: "alto riesgo" no basta. Un sistema útil debe registrar: `alto_riesgo_por = ["hipotension", "taquicardia", "fiebre"]`.

## Dónde se solapan

Estos modelos mentales se refuerzan mutuamente. Separar observación, interpretación y acción ayuda a convertir ambigüedad en estados. Definir estados permite capturar excepciones. Capturar excepciones facilita auditoría. Auditar obliga a priorizar riesgos de forma explícita.

El meta-modelo que los engloba es este:

> Pensar en pasos es diseñar una trayectoria verificable desde datos incompletos hasta una decisión responsable.

## Dónde crean tensión

También hay tensiones. Un libro serio no debe ocultarlas.

Un algoritmo muy detallado puede ser más seguro, pero más lento de usar. Un triage muy sensible puede detectar más emergencias, pero sobrecargar el sistema. Un procedimiento muy flexible puede respetar contexto, pero volverse difícil de auditar. Una regla simple puede ser rápida, pero pobre ante pacientes complejos.

La madurez consiste en reconocer que no existe "el paso correcto" en abstracto. Existe un orden defendible para un objetivo, un contexto, un riesgo y una población.

## De procedimiento humano a código

Podemos traducir una miniatura del razonamiento anterior a Python:

```python
# Entradas: observaciones iniciales.
estado_conciencia_alterado = False
presion_sistolica = 88
frecuencia_respiratoria = 32
saturacion_oxigeno = 89

# Lista de razones que explicarán la clasificación.
criterios_emergencia = []

if estado_conciencia_alterado:
    criterios_emergencia.append("alteración del estado de conciencia")

# Cada condición agrega una razón si el criterio está presente.
if presion_sistolica < 90:
    criterios_emergencia.append("hipotension")

if frecuencia_respiratoria > 30:
    criterios_emergencia.append("taquipnea severa")

if saturacion_oxigeno < 90:
    criterios_emergencia.append("hipoxemia")

# Si existe al menos una razón, el estado operativo cambia.
if criterios_emergencia:
    clasificacion = "alto riesgo"
else:
    clasificacion = "evaluación no urgente"

# Salidas visibles: estado y justificación.
print(clasificacion)
print(criterios_emergencia)
```

Salida esperada:

```text
alto riesgo
['hipotension', 'taquipnea severa', 'hipoxemia']
```

Este código sigue siendo una simplificación. No es una guía clínica. No reemplaza una escala validada. Pero ya muestra una diferencia decisiva: la salida no aparece sola. Viene acompañada de razones.

Ese cambio es enorme. Un algoritmo que da una respuesta sin razones puede servir para automatizar. Un algoritmo que conserva razones puede servir para aprender, auditar y mejorar.

## Argumentos críticos

### Desacuerdo 1: rapidez contra exhaustividad

Pregunta: ¿debe un algoritmo clínico pedir muchos datos para ser más preciso o pocos datos para actuar rápido?

El argumento por rapidez es fuerte en triage: muchas muertes tempranas se previenen identificando signos de emergencia de inmediato, como enfatiza la estrategia ETAT de la OMS para clasificar y tratar niños gravemente enfermos al ingreso. El argumento por exhaustividad es fuerte en diagnóstico: las National Academies describen el diagnóstico como un proceso complejo de recolección, integración e interpretación de información.

Importa porque un sistema demasiado lento puede fallar en urgencias, pero uno demasiado simple puede pasar por alto diagnósticos complejos.

Consenso emergente: no hay una única profundidad correcta. La granularidad debe depender de riesgo, tiempo disponible, consecuencias del falso negativo, carga operativa y posibilidad de reevaluación.

### Desacuerdo 2: reglas explícitas contra juicio experto

Pregunta: ¿cuánto debe formalizarse y cuánto debe dejarse al criterio clínico?

Las reglas explícitas permiten reproducibilidad, auditoría y programación. El juicio experto permite reconocer patrones, excepciones y contexto que la regla no captura. El conflicto no se resuelve eliminando uno de los lados. Se resuelve diseñando algoritmos que hagan explícito su alcance y permitan intervención humana cuando la situación excede el modelo.

Consenso emergente: los procedimientos explícitos son especialmente útiles en tareas repetibles, de alto riesgo y sensibles a omisiones; pero deben declarar sus límites y no fingir que todo juicio clínico cabe en una regla.

### Desacuerdo 3: algoritmo como respuesta contra algoritmo como instrumento

Pregunta: ¿el algoritmo debe entregar una decisión final o estructurar mejor el pensamiento?

En sistemas de bajo riesgo, puede bastar una salida automática. En medicina, con frecuencia el mayor valor del algoritmo no es sustituir al clínico, sino ordenar información, revelar omisiones y hacer visible la razón de una recomendación. Esta distinción será central en todo el libro.

Consenso emergente: en dominios biomédicos, el algoritmo más valioso no siempre es el que decide por el humano, sino el que mejora la calidad de la representación, reduce omisiones y permite revisar el razonamiento.

## Puente hacia la frontera

La capacidad de pensar en pasos será indispensable cuando el libro avance hacia problemas más exigentes.

En bioinformática, un alineamiento de secuencias no es una comparación intuitiva entre cadenas de ADN. Es una secuencia de decisiones sobre coincidencias, penalizaciones, brechas, costo y optimización. En neurología computacional, analizar una señal electrofisiológica exige decidir ventanas temporales, filtros, artefactos, rasgos y estados. En medicina de precisión, un sistema no puede limitarse a "predecir riesgo"; debe mostrar qué datos usó, qué supuestos hizo, qué incertidumbre conserva y qué acción es defendible.

Por eso esta sección no es un tema menor de introducción. Es la gramática inicial de todo procedimiento científico computable.

## Límites de esta miniatura

Pensar en pasos no garantiza que el procedimiento sea clínicamente suficiente. Una secuencia puede estar ordenada y aun así omitir datos relevantes, excepciones, sesgos o consecuencias. La utilidad de esta sección es entrenar descomposición y trazabilidad, no reemplazar validación de dominio.

## Evaluar si entendiste

Estas preguntas no buscan memoria. Buscan criterio. Están ordenadas de mayor a menor dificultad.

1. ¿Cómo diseñarías un algoritmo que distinga entre "paciente estable" y "evaluación incompleta" sin convertir la ausencia de datos en falsa tranquilidad?
2. ¿En qué circunstancias una regla más simple puede ser epistémicamente más honesta y clínicamente más segura que un modelo más complejo?
3. ¿Cómo cambiaría tu secuencia de pasos si el objetivo no fuera diagnosticar, sino evitar muertes prevenibles en los primeros minutos de atención?
4. ¿Qué información mínima necesitarías registrar para que una clasificación de alto riesgo sea auditable seis meses después?
5. ¿Dónde trazarías el límite entre una excepción que debe codificarse y una excepción que debe escalarse a juicio humano?
6. ¿Cómo distinguirías dato, interpretación y acción en un sistema de triage, y qué daño produce mezclarlos?
7. ¿Qué tensión aparece entre sensibilidad y carga operativa en un sistema de alerta hospitalario?
8. ¿Por qué una lista de verificación puede fracasar si se implementa como ritual y no como arquitectura de decisión?
9. ¿Qué excepciones debería contemplar una regla basada en temperatura corporal?
10. ¿Cómo sabrías si un ejemplo introductorio está simplificando de forma pedagógica o deformando el problema?

## Vacíos de comprensión que debes vigilar

1. Confundir orden narrativo con estructura algorítmica. Que una explicación suene ordenada no significa que sea ejecutable, auditable o segura.
2. Confundir dato faltante con dato normal. En medicina y ciencia, lo no medido no debe entrar silenciosamente como normalidad.
3. Confundir automatización con inteligencia. Un sistema que produce una salida sin razones puede acelerar una mala decisión.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** releer esta sección y reescribir el ejemplo de triage separando observación, interpretación y acción.
2. **Segunda hora:** estudiar el proceso diagnóstico de las National Academies y compararlo con ETAT: qué pasos aparecen en ambos y cuáles cambian por el contexto.
3. **Tercera hora:** implementar una versión pequeña del algoritmo con estados `dato_pendiente`, `alto_riesgo`, `prioridad` y `no_urgente`, registrando siempre la razón de la salida.

## Bibliografía y fuentes

- Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2022). *Introduction to algorithms* (4th ed.). MIT Press. <https://mitpress.mit.edu/9780262046305/introduction-to-algorithms/>
- Croskerry, P. (2003). Cognitive forcing strategies in clinical decisionmaking. *Annals of Emergency Medicine, 41*(1), 110-120. <https://pubmed.ncbi.nlm.nih.gov/12514691/>
- Croskerry, P. (2009). Clinical cognition and diagnostic error: Applications of a dual process model of reasoning. *Advances in Health Sciences Education, 14*(Suppl. 1), 27-35. <https://pubmed.ncbi.nlm.nih.gov/19669918/>
- Haynes, A. B., et al. (2009). A surgical safety checklist to reduce morbidity and mortality in a global population. *New England Journal of Medicine, 360*, 491-499. Reproducido en *WHO guidelines for safe surgery 2009*. <https://www.ncbi.nlm.nih.gov/books/NBK143241/>
- National Academies of Sciences, Engineering, and Medicine. (2015). *Improving diagnosis in health care*. National Academies Press. <https://www.ncbi.nlm.nih.gov/books/NBK338596/>
- National Academies of Sciences, Engineering, and Medicine. (2015). The diagnostic process. En *Improving diagnosis in health care*. National Academies Press. <https://www.ncbi.nlm.nih.gov/books/NBK338593/>
- Weiser, T. G., & Haynes, A. B. (2018). Ten years of the surgical safety checklist. *British Journal of Surgery, 105*(8), 927-929. <https://pubmed.ncbi.nlm.nih.gov/29770959/>
- World Health Organization. (2005). *Emergency triage assessment and treatment (ETAT) course*. <https://www.who.int/publications/i/item/9241546875>

## Siguiente paso

Ahora podemos estudiar variables, datos y decisiones con más precisión: no como nombres para valores, sino como una teoría práctica de representación. Nombrar una variable es decidir qué parte del mundo merece existir dentro del programa.
