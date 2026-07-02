# Glosario

Este glosario no reemplaza las explicaciones de cada capítulo. Su función es ofrecer una referencia rápida para términos que aparecerán de forma recurrente en el libro. Las definiciones son operativas: buscan ayudar a leer, programar y razonar, no cerrar debates filosóficos.

## Algoritmo

Especificación finita, ordenada y verificable de transformaciones y decisiones que convierte entradas en salidas bajo reglas, restricciones y criterios de terminación.

## Acumulador

Variable o estructura que conserva información producida durante un recorrido. Puede guardar conteos, sumas, elementos filtrados, razones, errores o estados intermedios.

## Bucle

Estructura de control que repite una operación bajo una regla de recorrido o permanencia. En este libro, un bucle también representa una forma de gobernar procesos y conservar estado.

## Complejidad computacional

Forma de estudiar cómo crece el costo de un algoritmo cuando aumenta el tamaño de la entrada. Ese costo puede medirse en tiempo, memoria, llamadas a disco, comunicaciones o energía.

## Condición

Expresión que puede evaluarse como verdadera o falsa y que permite tomar caminos diferentes dentro de un procedimiento.

## Condicional

Estructura de control que selecciona una rama de ejecución según una condición. En este libro, un condicional también representa una decisión sobre el significado operativo de los datos.

## Dato

Representación registrada de una observación, medición, evento, categoría o estado. Un dato no equivale automáticamente a conocimiento ni a decisión.

## Dato faltante

Valor ausente, no medido, no registrado o no disponible. En dominios biomédicos no debe tratarse silenciosamente como normalidad.

## Decisión

Elección de una acción, clasificación, interpretación o transformación a partir de datos, reglas, contexto y objetivos.

## Estado

Representación discreta de la situación actual de un sistema, paciente, procedimiento o dato. Un estado permite saber qué se conoce, qué falta y qué transiciones son posibles.

## Estructura de datos

Forma organizada de representar información para facilitar operaciones como búsqueda, acceso, actualización, recorrido, comparación o agregación.

## Excepción

Condición que interrumpe, modifica o degrada el flujo esperado de un procedimiento. Puede ser técnica, como un tipo incompatible, o del dominio, como un caso fuera de población.

## Función

Bloque reutilizable de código que recibe entradas, ejecuta una operación y puede devolver una salida. En este libro, una función también será tratada como unidad de diseño y verificación.

## Contrato

Promesa operacional de una función, regla o componente: qué entradas espera, qué salida entrega, qué casos cubre y qué límites no debe cruzar.

## Docstring

Texto interno de documentación ubicado al inicio de una función, clase o módulo en Python. Sirve para explicar propósito, contrato y límites de uso.

## Iteración

Cada repetición individual de un bucle. Durante una iteración pueden cambiar acumuladores, estados, salidas parciales o condiciones de parada.

## Parámetro

Nombre que representa una entrada esperada por una función. Un buen parámetro conserva la información necesaria sobre dominio, unidad o significado.

## Rama

Camino de ejecución posible dentro de un condicional. Una rama responsable debe tener un significado claro y no mezclar estados conceptualmente distintos.

## Modelo

Representación simplificada de un fenómeno. Un modelo decide qué aspectos del mundo conserva, cuáles omite y qué relaciones considera relevantes.

## Representación

Forma en que un fenómeno del mundo se convierte en algo manipulable por un sistema: variable, lista, tabla, matriz, grafo, texto, señal, imagen o secuencia.

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
