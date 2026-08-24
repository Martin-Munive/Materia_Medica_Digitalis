# Auditorías GLOBAL de integridad

## Auditoría inaugural

Fecha: 2026-08-02

Alcance auditado:

- front matter;
- Unidad I completa: `El lenguaje de las decisiones`;
- apertura de Unidad II: `Tipos de datos para problemas biomédicos`;
- secciones 13 y 14 de Unidad II;
- apéndice de entorno;
- glosario vivo;
- documentación interna de tesis, estado y roadmap.

## Tesis de control

La auditoria se realiza contra una tesis editorial amplia: el libro debe construir una arquitectura intelectual entre medicina, ciencias de la vida y computacion, usando Python para volver explicitos datos, representaciones, reglas, incertidumbre, excepciones, trazabilidad, verificacion y limites.

La pregunta operativa es:

```text
Esta parte del libro educa al autor y al lector hacia la promesa maxima, o solo agrega contenido correcto pero aislado?
```

La respuesta global es favorable: el libro mantiene una promesa reconocible y acumulativa. No se comporta como apuntes sueltos ni como tutorial genérico de Python. La Unidad I construye un lenguaje de decisión; la Unidad II inicia el paso lógico hacia representación de datos.

## Matriz de congruencia

| Bloque | Funcion en la obra | Tesis | Progresion | Terminologia | Fuentes | Codigo | Limites | Riesgo | Accion |
|---|---|---|---|---|---|---|---|---|---|
| Front matter | Orientar lector y posicionar obra | OK | OK | OK | NO_APLICA | PARCIAL | NO_APLICA | MENOR | Mantener; revisar promesa pública antes de próxima publicación mayor |
| Unidad I contenedor | Enmarcar pensamiento algorítmico como lenguaje de decisiones | OK | OK | OK | OK | NO_APLICA | NO_APLICA | RESUELTO | Actualizado mapa de 5 a 11 secciones |
| Secciones 01-05 | Construir algoritmo, pasos, variables, estados, excepciones y trazabilidad | OK | OK | OK | OK | OK | OK | RESUELTO | Agregados límites explícitos |
| Secciones 06-11 | Convertir conceptos en arquitectura ejecutable con condicionales, bucles, funciones y pruebas | OK | OK | OK | OK | OK | OK | MENOR | Mantener línea CODE CLEAN |
| Unidad II contenedor | Abrir tipos de datos como promesas operacionales | OK | OK | OK | OK | OK | NO_APLICA | RESUELTO | Agregada bibliografía |
| Secciones 13-14 | Desarrollar medición, texto, códigos y vocabularios | OK | OK | OK | OK | OK | OK | MENOR | Buen patrón para la siguiente sección |
| Apéndice A | Explicar entorno mínimo | OK | OK | OK | NO_APLICA | OK | NO_APLICA | RESUELTO | Agregada salida esperada |
| Glosario | Normalizar términos recurrentes | PARCIAL | OK | PARCIAL | NO_APLICA | NO_APLICA | NO_APLICA | MAYOR | Requiere revisión cíclica antes o durante la próxima sección |

## Hallazgos

### Resueltos en este ciclo

- El contenedor de Unidad I estaba desactualizado: hablaba de cinco secciones aunque la unidad ya tenía once.
- Faltaba transición explícita desde el contenedor de Unidad I hacia la Unidad II.
- La sección 05 cerraba la unidad sin usar el encabezado operativo `Siguiente paso`.
- El contenedor de Unidad II no tenía bibliografía proporcional a su función conceptual.
- El apéndice tenía comandos sin declarar salida esperada.
- Las secciones 01-09 no separaban de forma explícita los límites de sus miniaturas pedagógicas.

### Pendientes editoriales

- Revisar el glosario como sistema, no solo como lista acumulada.
- Evaluar si el README público debe incorporar una nota breve sobre la función formativa del libro sin hacerlo sonar informal.
- Antes de publicar una siguiente unidad completa, revisar que la tesis de `docs/editorial_thesis_scope.md` esté reflejada en portada, prefacio y README.

## Estado reputacional

La obra proyecta una identidad defendible: médico-programador que estudia, formaliza, ejecuta y explica conceptos computacionales desde problemas biomédicos.

El principal riesgo reputacional no es la falta de editorial ni de revisión por pares. Ese límite está declarado y puede manejarse. El riesgo real sería que el libro prometa frontera científica pero se estanque en ejemplos básicos. Por ahora la progresión evita ese problema: los ejemplos simples están funcionando como escalones, no como techo.

## Estado del glosario

El glosario está activo y contiene términos relevantes, pero requiere auditoría propia antes de que crezca más la Unidad II.

Acción recomendada:

- revisar duplicados conceptuales;
- confirmar que términos de Unidad I mantengan significado estable;
- agregar solo términos transversales de Unidad II;
- evitar que todo concepto local entre al glosario maestro.

## Siguiente cuota GLOBAL

La próxima revisión GLOBAL debe activarse cuando ocurra una de estas condiciones:

- se agreguen tres secciones nuevas desde esta auditoría;
- se cierre la Unidad II;
- se prepare una publicación pública importante o uso reputacional del libro;
- aparezca deriva entre promesa formativa, ejemplos de código y alcance biomédico.

## Siguiente paso editorial

Continuar con la sección:

```text
Booleanos, estados e incertidumbre
```

Condiciones para esa sección:

- mantener patrón `tipo + validador`;
- evitar reducir booleanos a `True/False`;
- mostrar por qué medicina necesita estados como `desconocido`, `no_aplica`, `pendiente`, `indeterminado`;
- conservar dato, interpretación, decisión, razón y límite;
- actualizar glosario al final;
- ejecutar ejemplos y declarar salida esperada.

---

## Auditoría de cierre del Capítulo II

**Fecha:** 2026-08-21
**Alcance:** 18 secciones del Capítulo II, desde la representación de datos biomédicos hasta el flujo verificable.

### Pregunta de auditoría

¿El capítulo construye una progresión coherente y reproducible desde un dato clínico aislado hasta un resultado que conserva contexto, rechazos, denominadores y trazabilidad?

### Matriz de comprobación

| Dimensión | Evidencia | Resultado |
|---|---|---|
| Progresión pedagógica | Tipos, estructuras, funciones, archivos, errores, pruebas, validación por lotes e integración final | Cumple |
| Continuidad técnica | Los conceptos reaparecen como contratos explícitos dentro del flujo final | Cumple |
| Reproducibilidad | El cierre fija entradas, transformaciones, rechazos, denominadores y artefactos | Cumple |
| Código ejecutable | Los 11 bloques de Python del cierre se ejecutaron en secuencia sin errores | Cumple |
| Seguridad conceptual | Se distingue un ejemplo educativo de una regla de decisión clínica | Cumple |
| Navegación | Tabla de contenido, ruta del capítulo, README, roadmap y estado fueron actualizados | Cumple |
| Vocabulario | Se incorporó al glosario la definición de flujo verificable | Cumple |

### Hallazgos

1. La secuencia del capítulo ya no termina en operaciones aisladas: la sección 18 demuestra cómo coordinarlas sin ocultar errores ni pérdidas de datos.
2. La distinción entre registros aceptados y rechazados permite explicar calidad sin confundir ausencia, invalidez y exclusión.
3. El denominador queda tratado como parte del resultado y no como un detalle implícito del cálculo.
4. La compuerta recomendó resolver el nombre y la promesa pedagógica antes de abrir contenido nuevo; ambas decisiones quedaron cerradas el 2026-08-24.

### Dictamen

El Capítulo II queda **cerrado y aprobado para publicación**. No existen bloqueos técnicos o pedagógicos para conservarlo como una unidad completa. El Capítulo III se abrió después de adoptar el nombre `Algoritmos fundamentales con lectura biomédica` y definir su artefacto acumulativo.

### Próxima compuerta

1. Publicar y verificar el inicio del Capítulo III en el sitio.
2. Mantener problema biomédico, costo, propiedad y límite en cada algoritmo.
3. Ejecutar la siguiente cuota GLOBAL al cerrar el Capítulo III o si aparece una desviación estructural fuerte.
