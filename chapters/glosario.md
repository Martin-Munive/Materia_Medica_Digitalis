# Glosario

Este glosario no reemplaza las explicaciones de cada capítulo. Su función es ofrecer una referencia rápida para términos que aparecerán de forma recurrente en el libro. Las definiciones son operativas: buscan ayudar a leer, programar y razonar, no cerrar debates filosóficos.

## Algoritmo

Especificación finita, ordenada y verificable de transformaciones y decisiones que convierte entradas en salidas bajo reglas, restricciones y criterios de terminación.

## Análisis reproducible

Cálculo cuya salida puede reconstruirse a partir de datos versionados, parámetros explícitos, consultas o reglas identificables, denominadores y resultado registrado.

## API

Frontera contractual que permite a un componente solicitar una operación a otro. Puede ser una función pública, un comando o un endpoint HTTP; lo esencial es que declare entrada, salida, errores y límites.

## Artefacto

Salida intermedia o final producida por una etapa de trabajo: filas crudas, tabla normalizada, rechazos, resultado, reporte o archivo. Debe tener nombre y propósito claros dentro del flujo.

## Artefacto compartible

Salida persistente cuyo formato, propósito, versión y contrato permiten que otra persona o proceso la inspeccione o consuma. Que sea técnicamente compartible no significa que sea pública ni que carezca de información sensible.

## Auditoría de artefactos

Comparación entre los archivos actuales y un contrato o manifiesto previo para detectar ausencias, cambios de tamaño, diferencias de contenido y otras rupturas de integridad. No demuestra por sí sola validez clínica ni autorización.

## Huella de contenido

Resumen de longitud fija calculado a partir de los bytes de un artefacto, por ejemplo mediante SHA-256. Permite detectar cambios respecto a una huella conocida, pero no prueba autoría, identidad ni corrección del contenido.

## Manifiesto de exportación

Archivo que describe un paquete exportado mediante identificador de lote, versiones, conteos, instante de creación y lista de artefactos con tamaño y huella. Funciona como índice verificable de la exportación.

## Acumulador

Variable o estructura que conserva información producida durante un recorrido. Puede guardar conteos, sumas, elementos filtrados, razones, errores o estados intermedios.

## Bucle

Estructura de control que repite una operación bajo una regla de recorrido o permanencia. En este libro, un bucle también representa una forma de gobernar procesos y conservar estado.

## Booleano

Tipo de dato con dos valores posibles: verdadero o falso. Es útil para distinciones estrictamente binarias, pero puede ser insuficiente cuando un dato biomédico admite ausencia, incertidumbre, no evaluación o no aplicabilidad.

## Complejidad computacional

Forma de estudiar cómo crece el costo de un algoritmo cuando aumenta el tamaño de la entrada. Ese costo puede medirse en tiempo, memoria, llamadas a disco, comunicaciones o energía.

## Condición

Expresión que puede evaluarse como verdadera o falsa y que permite tomar caminos diferentes dentro de un procedimiento.

## Condicional

Estructura de control que selecciona una rama de ejecución según una condición. En este libro, un condicional también representa una decisión sobre el significado operativo de los datos.

## Consulta reproducible

Pregunta escrita de forma explícita, parametrizada y repetible sobre una base de datos o tabla. Permite reconstruir criterios de selección, denominadores y resúmenes.

## Ejecución de análisis

Registro concreto de un análisis realizado: especificación usada, versión de datos, parámetros, resultado, denominador y metadatos suficientes para auditar o comparar una repetición.

## Dato

Representación registrada de una observación, medición, evento, categoría o estado. Un dato no equivale automáticamente a conocimiento ni a decisión.

## Dato faltante

Valor ausente, no medido, no registrado o no disponible. En dominios biomédicos no debe tratarse silenciosamente como normalidad.

## Dato ausente

Valor no disponible para una operación concreta. Debe distinguirse de cero, normalidad, no aplicabilidad, resultado pendiente, dato inválido o dato censurado.

## Dato temporal

Representación de tiempo acompañada por tipo de evento, precisión, estado, fuente y regla de cálculo. Puede ser fecha, instante, intervalo, duración o ventana.

## Campo

Parte nombrada de un registro, como `valor`, `unidad`, `estado` o `fuente`. Un campo debe tener significado estable dentro del contrato de la observación.

## Calidad de datos

Grado en que un conjunto de datos es suficiente para una operación concreta según completitud, consistencia, validez, trazabilidad, unidad, estado y reglas del dominio.

## Clave

Identificador usado en un diccionario para acceder a un valor. En registros biomédicos debe ser estable, legible y coherente con el dominio.

## Clave foránea

Campo que enlaza una fila con una entidad registrada en otra tabla. Permite declarar que una medición pertenece a un paciente, una muestra o un evento existente.

## Clave primaria

Identificador que distingue de forma única una fila dentro de una tabla. Protege identidad local y permite que otras tablas apunten a esa entidad.

## Colección

Estructura que agrupa varios elementos para recorrerlos, buscarlos, contarlos, transformarlos o validarlos. Una colección no define por sí sola el significado de sus elementos.

## Medición

Valor observado acompañado por unidad, contexto y regla de validez. En dominios biomédicos, una medición no debe reducirse al número que la representa.

## Decisión

Elección de una acción, clasificación, interpretación o transformación a partir de datos, reglas, contexto y objetivos.

## Estado

Representación discreta de la situación actual de un sistema, paciente, procedimiento o dato. Un estado permite saber qué se conoce, qué falta y qué transiciones son posibles.

## Estado controlado

Valor permitido dentro de un conjunto explícito de estados. Evita que condiciones como presente, ausente, desconocido, no evaluado o no aplicable se dispersen como textos o booleanos ambiguos.

## Entidad

Cosa del dominio que necesita identidad propia dentro de un sistema, como paciente, muestra, medición, evento, fármaco, regla o fuente.

## Etapa

Paso delimitado dentro de un pipeline. Recibe una entrada definida, aplica una responsabilidad concreta y produce una salida inspeccionable.

## Marcador especial

Símbolo, texto o número usado para indicar una condición distinta al valor ordinario, como pendiente, inválido, no aplica o no medido. Debe traducirse antes de calcular.

## Estructura de datos

Forma organizada de representar información para facilitar operaciones como búsqueda, acceso, actualización, recorrido, comparación o agregación.

## Excepción

Condición que interrumpe, modifica o degrada el flujo esperado de un procedimiento. Puede ser técnica, como un tipo incompatible, o del dominio, como un caso fuera de población.

## Función

Bloque reutilizable de código que recibe entradas, ejecuta una operación y puede devolver una salida. En este libro, una función también será tratada como unidad de diseño y verificación.

## Flujo verificable

Proceso compuesto por etapas con contratos explícitos, artefactos identificables y controles que permiten reconstruir cómo una entrada llegó a convertirse en una salida. Conserva valores aceptados, rechazos, versiones, denominadores y límites proporcionales a su uso.

## Contrato

Promesa operacional de una función, regla o componente: qué entradas espera, qué salida entrega, qué casos cubre y qué límites no debe cruzar.

## Contrato de entrada

Descripción explícita de los campos, tipos, valores permitidos y condiciones que una solicitud debe cumplir antes de ejecutar una operación.

## Contrato de salida

Descripción explícita de la respuesta que una operación puede devolver, incluyendo estado, datos, identificadores, errores, razones y versión del contrato aplicado.

## Código

Identificador estable asignado a un concepto, observación, procedimiento, documento o categoría dentro de un sistema definido. Debe conservar sistema y versión para ser trazable.

## Docstring

Texto interno de documentación ubicado al inicio de una función, clase o módulo en Python. Sirve para explicar propósito, contrato y límites de uso.

## Iteración

Cada repetición individual de un bucle. Durante una iteración pueden cambiar acumuladores, estados, salidas parciales o condiciones de parada.

## Índice

Estructura auxiliar de una base de datos que acelera búsquedas, ordenamientos o uniones sobre una o varias columnas. No cambia el contenido del dato, pero cambia el costo de recuperarlo.

## Granularidad

Precisión con la que se conoce o representa un dato. En datos temporales puede ser año, mes, día, hora, minuto o segundo, y gobierna qué cálculos son legítimos.

## Instante

Momento temporal específico, idealmente con fecha, hora y zona horaria cuando se requiere comparación entre sistemas, lugares o registros.

## Lista

Colección ordenada de elementos. Es útil para representar secuencias, conjuntos de observaciones o resultados, pero no debe usarse sola para ocultar campos con significado distinto.

## Intervalo

Espacio temporal entre un inicio y un fin. Puede estar cerrado, abierto, incompleto o no calculable según la disponibilidad y precisión de sus límites.

## Parámetro

Nombre que representa una entrada esperada por una función. Un buen parámetro conserva la información necesaria sobre dominio, unidad o significado.

## Pipeline

Secuencia ordenada de etapas que transforma entradas en salidas, conservando contratos, conteos, errores y artefactos suficientes para auditar el flujo.

## Resultado derivado

Salida producida por una transformación, consulta, regla o análisis a partir de datos previos. Debe conservar trazabilidad proporcional a su uso posterior.

## Reporte de calidad

Resumen estructurado de la evaluación de un lote: conteos, proporciones, errores, advertencias, completitud y estado operativo resultante.

## Rama

Camino de ejecución posible dentro de un condicional. Una rama responsable debe tener un significado claro y no mezclar estados conceptualmente distintos.

## Modelo

Representación simplificada de un fenómeno. Un modelo decide qué aspectos del mundo conserva, cuáles omite y qué relaciones considera relevantes.

## Normalización

Transformación técnica que prepara un valor para comparación o validación, como convertir texto a minúsculas, controlar espacios o remover acentos. No equivale por sí sola a interpretación semántica.

## No aplicable

Estado que indica que una pregunta, regla o campo no corresponde al caso evaluado. No debe usarse como sinónimo de dato faltante, desconocido o no diligenciado.

## Relación

Vínculo explícito entre entidades. En una base relacional suele expresarse mediante claves, por ejemplo una medición asociada a un paciente.

## Representación

Forma en que un fenómeno del mundo se convierte en algo manipulable por un sistema: variable, lista, tabla, matriz, grafo, texto, señal, imagen o secuencia.

## Registro

Observación representada por campos nombrados. En dominios biomédicos debe declarar campos mínimos, tipos, unidades, estado y trazabilidad proporcional al uso posterior.

## Restricción

Regla declarada en una base de datos para impedir estados estructuralmente inválidos, como claves duplicadas, campos obligatorios ausentes, referencias huérfanas o valores fuera de un conjunto permitido.

## Retorno

Salida que una función entrega al terminar. Puede ser un valor simple o una estructura con estado, razón y metadatos.

## Regla de decisión

Criterio explícito que conecta datos y condiciones con una clasificación, acción, transición o salida.

## Sesgo

Desviación sistemática que afecta observaciones, datos, modelos, decisiones o resultados. Puede originarse en medición, selección, codificación, contexto social, diseño del sistema o interpretación.

## Trazabilidad

Capacidad de reconstruir cómo se llegó a una salida: qué datos entraron, qué reglas se aplicaron, qué versión del procedimiento se usó y qué excepciones aparecieron.

## Transición

Cambio de un estado a otro cuando una condición se cumple. En un algoritmo responsable, la transición debe conservar razones y límites.

## Umbral

Valor que separa estados o conductas. Un umbral puede ser útil, pero casi siempre simplifica un fenómeno continuo o contextual.

## Validación

Proceso de evaluar si un procedimiento, modelo o algoritmo funciona como se espera en los datos, condiciones y poblaciones para los que será usado.

## Versión de regla

Identificador que permite saber qué variante de un criterio, algoritmo o modelo produjo una salida. Es parte mínima de la trazabilidad cuando una decisión puede cambiar con el tiempo.

## Variable

Nombre que conserva un valor para ser usado por un programa. En este libro, una variable se entiende también como una decisión de representación.

## Excepción técnica

Interrupción del flujo del programa causada por condiciones que el lenguaje puede detectar, como tipos incompatibles, valores fuera de rango o divisiones imposibles. Debe distinguirse de las excepciones del dominio.

## Excepción del dominio

Interrupción o degradación del flujo causada por una condición que el lenguaje no puede detectar por sí solo, como un dato fuera de población, una unidad equivocada o un valor fisiológicamente imposible. Su tratamiento es parte del diseño del algoritmo.

## Error

Condición que impide que un procedimiento produzca una salida confiable. Un error puede ser técnico o del dominio, y puede traducirse en una excepción explícita o en una salida degradada.

## Manejo de excepciones

Conjunto de prácticas que capturan, traducen y registran excepciones para evitar que el procedimiento falle de forma silenciosa o abrupta.

## Seguridad del cálculo

Propiedad de un sistema que limita la propagación del error, lo hace trazable y evita que una salida inválida se confunda con una salida válida. No se reduce a la corrección sintáctica del código.

## Tipo de dato

Contrato técnico y semántico que define cómo se representa un valor, qué operaciones admite, qué errores debe rechazar y qué significado conserva para una decisión posterior.

## Texto libre

Texto narrativo no restringido por un conjunto cerrado de valores. Conserva matices, pero no promete uniformidad para contar, filtrar, comparar o decidir.

## Unidad

Referencia que da escala y significado operacional a una medición. Sin unidad explícita, un número puede ser ejecutable y aun así ser ambiguo o incompatible con la decisión.

## Validador

Procedimiento que revisa si un dato cumple el contrato mínimo requerido antes de entrar a una transformación, clasificación o decisión.

## Valor centinela

Valor usado para representar un estado especial, como ausencia o no disponibilidad. Es peligroso cuando puede confundirse con un valor real del dominio.

## Vocabulario controlado

Conjunto explícito de valores permitidos, cada uno con significado definido y, cuando corresponde, código estable. Reduce ambigüedad sin reemplazar necesariamente el texto narrativo original.

## Función pura

Función cuya salida depende solo de sus entradas y que no produce efectos observables fuera de sí misma: no lee teclado, no imprime, no escribe archivos ni modifica estado externo. Misma entrada produce siempre misma salida, lo que la hace verificable con una sola llamada.

## Efecto secundario

Cualquier interacción de una función con el mundo exterior: leer entrada, imprimir, escribir archivos, consultar el reloj o modificar estado compartido. Los efectos no son errores; son necesidades que deben ubicarse en piezas delgadas y explícitas.

## Coordinación de procesos

Función pura que, en lugar de ejecutar efectos, construye la descripción de lo que debe ocurrir —el plan: qué mostrar, qué registrar, qué notificar— para que un ejecutor delgado lo lleve a cabo. Separa decidir de hacer.

## Prueba (test)

Declaración ejecutable de una propiedad que un procedimiento debe cumplir. Una prueba que no declara propiedad recorre el código sin verificar nada; puede mostrar la presencia de defectos, nunca su ausencia.

## Propiedad (de una prueba)

Afirmación general sobre la relación entre entradas y salidas de un procedimiento, en contraste con un caso concreto. Ejemplos: la frontera de un umbral, la conservación de la ausencia de dato, la monotonicidad de una regla.

## Reproducibilidad

Propiedad de un procedimiento que produce la misma salida ante la misma entrada. Es la primera propiedad que conviene verificar, porque sin ella no existe cálculo estable que probar.

## Denominador

Conjunto de casos sobre los que se calcula una proporción. Cuando hay datos faltantes, el denominador debe declararse de forma explícita: total, documentados, medidos, elegibles u otra población.

## Columna

Variable o campo compartido por las filas de una tabla. En datos biomédicos debe tener nombre estable, tipo esperado, unidad cuando corresponda y regla de interpretación.

## DataFrame

Estructura tabular de `pandas` organizada por filas y columnas etiquetadas. En datos biomédicos debe tratarse como una mesa de trabajo, no como garantía automática de limpieza, validez o significado.

## Almacenamiento persistente

Conservación estructurada de datos más allá de una ejecución del programa. Puede ser un archivo, una base de datos u otro sistema de registro, pero debe mantener reglas de interpretación y trazabilidad proporcionales al uso.

## Esquema

Contrato que declara columnas, tipos, unidades, valores permitidos, campos requeridos y reglas de validez de una tabla o estructura de datos.

## Esquema mínimo

Contrato reducido pero explícito que declara los campos, tipos, obligatoriedad, unidades, valores permitidos, límites y versión necesarios para una operación concreta. No pretende cubrir todo el dominio, sino hacer defendible una transformación o cálculo específico.

## Fila

Observación individual dentro de una tabla. Puede representar un paciente, una muestra, una medición, un evento, una variante o cualquier unidad analítica definida.

## SQLite

Motor de base de datos relacional embebido. Es útil para aprendizaje, prototipos, herramientas locales y persistencia ligera, pero no elimina la necesidad de diseñar tablas, claves, restricciones y reglas de validación.

## Máscara booleana

Serie de valores verdadero/falso usada para seleccionar filas de una tabla. En datos biomédicos debe expresar una regla de dominio explícita, como fila medida, unidad compatible o valor calculable.

## Limpieza de datos

Transformación documentada de valores crudos para hacerlos comparables e interpretables sin borrar su origen, estado o razón de cambio.

## Lote

Conjunto de registros procesados como unidad operativa. Un lote puede contener filas válidas, rechazos, advertencias y métricas agregadas de calidad.

## Tabla

Colección de filas y columnas que organiza observaciones y variables. Una tabla biomédica no es confiable solo por tener forma tabular; necesita esquema, validación, manejo de ausencias y trazabilidad.

## Tabla de trabajo

Estructura temporal usada para inspeccionar, limpiar, validar y transformar datos antes de persistirlos o analizarlos. No debe confundirse automáticamente con la fuente de verdad.

## Serie

Estructura unidimensional de `pandas` con etiquetas. Puede representar una columna, un vector de estados, una máscara o un resultado derivado.

## Validación formal

Comparación explícita de un dato, registro o tabla contra un esquema declarado. Debe producir una salida trazable: estado, valores limpios cuando existan, errores, razones y versión de regla aplicada.

## Versión de esquema

Identificador de la variante del contrato de datos que se usó para validar una estructura. Permite reconstruir por qué una fila fue aceptada o rechazada cuando las reglas cambian con el tiempo.
