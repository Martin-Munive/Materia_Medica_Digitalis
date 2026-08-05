# Fechas, tiempos, intervalos y granularidad clínica

Una fecha parece un dato tranquilo. Tiene forma reconocible, se puede escribir en un calendario y muchas veces cabe en una cadena como `"2026-08-04"`. Esa familiaridad engaña. En medicina y ciencias de la vida, el tiempo no solo responde cuándo ocurrió algo. También define orden, duración, ventana de validez, seguimiento, edad, exposición, latencia, inicio, finalización, precisión y comparabilidad.

La sección anterior mostró que `True` y `False` rara vez alcanzan cuando hay incertidumbre. Esta sección agrega una regla complementaria: una fecha sin contrato temporal puede parecer exacta aunque no lo sea.

Un paciente puede decir "empecé hace tres días", "desde la semana pasada", "en junio", "ayer en la noche" o "no recuerdo bien". Un laboratorio puede reportar una hora de toma de muestra, una hora de procesamiento y una hora de liberación. Un tratamiento puede tener fecha de inicio, fecha de suspensión, dosis omitidas y una ventana de seguimiento. Si todo eso se guarda como texto o como fechas sueltas, el programa podrá imprimirlo, pero no necesariamente podrá razonar de forma responsable.

## Origen técnico: calendario no es significado temporal

Python ofrece tipos específicos para trabajar con tiempo en el módulo `datetime`. Los más importantes para esta sección son:

- `date`: una fecha de calendario, sin hora;
- `datetime`: una fecha con hora;
- `timedelta`: una duración o diferencia entre momentos;
- `zoneinfo.ZoneInfo`: una forma estándar de asociar zonas horarias IANA a instantes.

Esos tipos son mejores que cadenas para calcular, comparar y validar. Pero no resuelven solos el problema biomédico. `date(2026, 8, 4)` dice día, mes y año. No dice si es fecha de nacimiento, inicio de síntomas, toma de muestra, consulta, exposición, cirugía, muerte, seguimiento o captura administrativa. Tampoco dice si la fecha fue exacta, aproximada, estimada por el paciente, importada de otro sistema o inferida por una regla.

El tipo técnico permite operar. El contrato temporal permite interpretar.

## Definición de trabajo

<div class="definition-block">
<strong>Definición de trabajo.</strong><br />
Un dato temporal biomédico es una representación de tiempo acompañada por tipo de evento, precisión, estado, fuente y regla de cálculo. Puede ser una fecha, un instante, un intervalo, una duración o una ventana, pero no debe fingir más exactitud de la que el dato realmente tiene.
</div>

Esta definición separa cinco objetos que suelen mezclarse.

**Fecha.** Día de calendario. Puede servir para nacimiento, consulta, inicio de síntomas o una medición fechada. No contiene hora ni zona.

**Instante.** Momento más específico, con fecha, hora y, cuando importa, zona horaria. Es necesario para eventos comparables entre lugares, sistemas o equipos.

**Intervalo.** Espacio entre dos puntos temporales. Puede estar cerrado, abierto, incompleto o no calculable.

**Duración.** Cantidad de tiempo transcurrido, expresada con unidad. No equivale a una fecha.

**Granularidad.** Precisión con la que se conoce el dato: año, mes, día, hora, minuto o segundo. La granularidad gobierna qué cálculos son legítimos.

La regla práctica es esta: antes de restar fechas, declarar qué representan y con qué precisión se conocen.

## Versión ingenua: fechas como cadenas

Supongamos que queremos calcular los días desde inicio de síntomas hasta consulta.

```python
def dias_desde_inicio(inicio, consulta):
    return consulta - inicio


inicio_sintomas = "2026-08-01"
fecha_consulta = "2026-08-04"

print(dias_desde_inicio(inicio_sintomas, fecha_consulta))
```

Salida esperada:

```text
TypeError
```

La función falla porque dos cadenas no se restan. Podemos intentar una versión que parezca más inteligente.

```python
from datetime import date


inicio_sintomas = date(2026, 8, 1)
fecha_consulta = date(2026, 8, 4)

print((fecha_consulta - inicio_sintomas).days)
```

Salida esperada:

```text
3
```

Ahora el cálculo funciona. Pero todavía hay un problema de representación. El número `3` no dice unidad, tipo de eventos, precisión, fuente, si el inicio era exacto o estimado, ni qué hacer si la fecha de inicio está ausente o cae después de la consulta.

El código dejó de ser sintácticamente frágil, pero todavía es semánticamente pobre.

## Crítica técnica: qué está mal

Primero, el cálculo no declara dominio. `fecha_consulta - inicio_sintomas` produce una duración, pero no conserva que se trata de un intervalo clínico entre inicio de síntomas y consulta.

Segundo, el resultado pierde unidad. `3` puede ser días, horas, semanas, visitas, ciclos o puntos. El lector humano infiere días porque conoce el ejemplo; el sistema no.

Tercero, no hay manejo de ausencia. Si el paciente no recuerda el inicio, el algoritmo debe devolver `no_calculable`, no inventar cero ni asumir que faltante significa hoy.

Cuarto, no hay control de orden. Una fecha de inicio posterior a la consulta puede ser error de captura, evento futuro, reprogramación o una pregunta mal formulada. No debe pasar como duración negativa sin explicación.

Quinto, no hay granularidad. "Agosto de 2026" no puede tratarse como `"2026-08-01"` solo porque el sistema necesita un día. Eso crea precisión falsa.

## Versión mejorada: evento temporal explícito

Una mejora mínima consiste en separar el valor temporal de sus metadatos. No construiremos todavía una clase completa de historia clínica. Usaremos diccionarios y enumeraciones para hacer visible el contrato.

```python
from datetime import date
from enum import Enum


class PrecisionTemporal(Enum):
    DIA = "dia"
    MES = "mes"
    ANIO = "anio"
    DESCONOCIDA = "desconocida"


class EstadoTemporal(Enum):
    EXACTO = "exacto"
    APROXIMADO = "aproximado"
    DESCONOCIDO = "desconocido"
    FUERA_DE_CONTRATO = "fuera_de_contrato"


def crear_evento_fecha(tipo_evento, fecha, precision=PrecisionTemporal.DIA, fuente="no_documentada"):
    """Crea una representación temporal mínima para eventos clínicos pedagógicos."""
    if fecha is None:
        return {
            "tipo_evento": tipo_evento,
            "fecha": None,
            "precision": PrecisionTemporal.DESCONOCIDA,
            "estado": EstadoTemporal.DESCONOCIDO,
            "fuente": fuente,
        }

    if not isinstance(fecha, date):
        return {
            "tipo_evento": tipo_evento,
            "fecha": None,
            "precision": PrecisionTemporal.DESCONOCIDA,
            "estado": EstadoTemporal.FUERA_DE_CONTRATO,
            "fuente": fuente,
        }

    return {
        "tipo_evento": tipo_evento,
        "fecha": fecha,
        "precision": precision,
        "estado": EstadoTemporal.EXACTO,
        "fuente": fuente,
    }


def calcular_dias_entre_eventos(evento_inicial, evento_final):
    """Calcula días solo cuando ambos eventos tienen fecha exacta con precisión de día."""
    eventos = [evento_inicial, evento_final]

    if any(evento["estado"] != EstadoTemporal.EXACTO for evento in eventos):
        return {
            "estado": "no_calculable",
            "valor": None,
            "unidad": "dias",
            "razon": "evento_temporal_no_exacto",
        }

    if any(evento["precision"] != PrecisionTemporal.DIA for evento in eventos):
        return {
            "estado": "no_calculable",
            "valor": None,
            "unidad": "dias",
            "razon": "granularidad_insuficiente",
        }

    dias = (evento_final["fecha"] - evento_inicial["fecha"]).days

    if dias < 0:
        return {
            "estado": "error_temporal",
            "valor": dias,
            "unidad": "dias",
            "razon": "evento_inicial_posterior_al_evento_final",
        }

    return {
        "estado": "calculable",
        "valor": dias,
        "unidad": "dias",
        "razon": "fechas_exactas_con_precision_dia",
    }


inicio = crear_evento_fecha(
    "inicio_sintomas",
    date(2026, 8, 1),
    fuente="relato_paciente",
)
consulta = crear_evento_fecha(
    "consulta",
    date(2026, 8, 4),
    fuente="registro_clinico",
)

resultado = calcular_dias_entre_eventos(inicio, consulta)
print(resultado["estado"])
print(resultado["valor"], resultado["unidad"])
print(resultado["razon"])
```

Salida esperada:

```text
calculable
3 dias
fechas_exactas_con_precision_dia
```

La función no es larga por capricho. Es más extensa porque conserva una promesa: solo calcula días cuando los eventos son fechas exactas con precisión de día. Si no puede calcular, devuelve razón.

## Anatomía del contrato

La primera pieza es el tipo de evento.

```python
"tipo_evento": "inicio_sintomas"
```

Una fecha aislada no basta. La misma fecha puede representar nacimiento, inicio de fiebre, toma de muestra, consulta, cirugía o alta. El tipo de evento permite que el intervalo tenga sentido.

La segunda pieza es la precisión.

```python
"precision": PrecisionTemporal.DIA
```

Si el inicio se conoce solo por mes, no se debe calcular una duración exacta en días. Se puede calcular una ventana aproximada o pedir revisión, pero no fingir precisión.

La tercera pieza es el estado.

```python
"estado": EstadoTemporal.EXACTO
```

La fecha exacta, aproximada, desconocida o fuera de contrato no debe entrar igual al cálculo. Esta idea viene de la sección anterior: no todo lo que falta equivale a falso, y no todo lo que parece fecha equivale a exactitud.

La cuarta pieza es la fuente.

```python
"fuente": "relato_paciente"
```

La fuente no cambia necesariamente el cálculo, pero sí la trazabilidad. Un inicio de síntomas por relato, una fecha de laboratorio importada automáticamente y una fecha estimada por reconstrucción no tienen la misma fuerza documental.

## Pruebas mínimas

```python
# Propiedad 1: dos fechas exactas con precisión de día producen duración calculable.
inicio = crear_evento_fecha("inicio_sintomas", date(2026, 8, 1))
consulta = crear_evento_fecha("consulta", date(2026, 8, 4))
assert calcular_dias_entre_eventos(inicio, consulta)["valor"] == 3

# Propiedad 2: fecha faltante no se transforma en duración cero.
inicio_faltante = crear_evento_fecha("inicio_sintomas", None)
assert calcular_dias_entre_eventos(inicio_faltante, consulta)["estado"] == "no_calculable"

# Propiedad 3: un evento inicial posterior al final se marca como error temporal.
inicio_posterior = crear_evento_fecha("inicio_sintomas", date(2026, 8, 8))
assert calcular_dias_entre_eventos(inicio_posterior, consulta)["estado"] == "error_temporal"

# Propiedad 4: una granularidad mensual no produce duración diaria exacta.
inicio_mensual = crear_evento_fecha(
    "inicio_sintomas",
    date(2026, 8, 1),
    precision=PrecisionTemporal.MES,
)
assert calcular_dias_entre_eventos(inicio_mensual, consulta)["razon"] == "granularidad_insuficiente"

# Propiedad 5: un valor textual fuera de contrato no se interpreta como fecha.
inicio_textual = crear_evento_fecha("inicio_sintomas", "2026-08-01")
assert calcular_dias_entre_eventos(inicio_textual, consulta)["estado"] == "no_calculable"
```

Salida esperada: no imprime nada si las propiedades se cumplen.

Estas pruebas no validan una escala clínica. Validan una propiedad de representación: ausencia, error de orden, granularidad insuficiente y fecha exacta no son lo mismo.

## Instantes, horas y zonas: cuándo `datetime` importa

Muchos datos clínicos se resuelven con `date`. Otros no. Una toma de muestra, una dosis intravenosa, una transfusión, un traslado, una monitorización o una señal fisiológica pueden depender de hora y zona.

Python permite crear `datetime` sin zona horaria:

```python
from datetime import datetime


toma_muestra = datetime(2026, 8, 4, 7, 30)
print(toma_muestra)
```

Salida esperada:

```text
2026-08-04 07:30:00
```

Ese valor tiene hora, pero no dice zona. En un sistema local puede ser suficiente si todo ocurre en el mismo hospital, en la misma zona y bajo una regla explícita. En intercambio de datos, telemedicina, laboratorios externos o auditoría longitudinal, la ausencia de zona puede ser peligrosa.

```python
from datetime import datetime
from zoneinfo import ZoneInfo


toma_muestra = datetime(2026, 8, 4, 7, 30, tzinfo=ZoneInfo("America/Bogota"))
procesamiento = datetime(2026, 8, 4, 10, 15, tzinfo=ZoneInfo("America/Bogota"))

delta = procesamiento - toma_muestra
print(delta)
print(delta.total_seconds() / 3600)
```

Salida esperada:

```text
2:45:00
2.75
```

Aquí el intervalo expresa dos horas y cuarenta y cinco minutos. El resultado puede ser importante para estabilidad de muestra, ventana preanalítica o trazabilidad de proceso. Pero, otra vez, el cálculo no decide por sí solo si la muestra es válida. Para eso se necesitaría una regla de dominio.

## Ventanas temporales: no todo intervalo está cerrado

Un intervalo puede tener inicio y fin. También puede tener inicio conocido y fin pendiente: tratamiento activo, seguimiento abierto, hospitalización en curso, incubación estimada, ventana de observación no cerrada.

```python
def describir_intervalo_tratamiento(inicio, fin):
    """Describe un intervalo de tratamiento sin cerrar artificialmente eventos abiertos."""
    if inicio is None:
        return {
            "estado": "no_evaluable",
            "duracion": None,
            "unidad": "dias",
            "razon": "inicio_no_documentado",
        }

    if fin is None:
        return {
            "estado": "intervalo_abierto",
            "duracion": None,
            "unidad": "dias",
            "razon": "tratamiento_sin_fecha_final",
        }

    dias = (fin - inicio).days
    if dias < 0:
        return {
            "estado": "error_temporal",
            "duracion": dias,
            "unidad": "dias",
            "razon": "fin_anterior_al_inicio",
        }

    return {
        "estado": "intervalo_cerrado",
        "duracion": dias,
        "unidad": "dias",
        "razon": "inicio_y_fin_documentados",
    }


print(describir_intervalo_tratamiento(date(2026, 8, 1), None)["estado"])
print(describir_intervalo_tratamiento(date(2026, 8, 1), date(2026, 8, 11))["duracion"])
```

Salida esperada:

```text
intervalo_abierto
10
```

El intervalo abierto es un estado legítimo, no un error. El error sería cerrarlo con la fecha actual sin declararlo, o registrar duración cero porque no existe fecha final.

## Edad: un caso donde el detalle importa

La edad parece una resta simple entre fecha actual y fecha de nacimiento. Pero según el uso, puede necesitar años cumplidos, meses, días, edad gestacional, edad corregida o edad al evento.

Una función pedagógica para años cumplidos puede ser así:

```python
def calcular_edad_anios_cumplidos(fecha_nacimiento, fecha_referencia):
    """Calcula años cumplidos en una fecha de referencia."""
    if fecha_nacimiento > fecha_referencia:
        return {
            "estado": "error_temporal",
            "edad": None,
            "unidad": "anios",
            "razon": "nacimiento_posterior_a_referencia",
        }

    edad = fecha_referencia.year - fecha_nacimiento.year
    no_ha_cumplido = (fecha_referencia.month, fecha_referencia.day) < (
        fecha_nacimiento.month,
        fecha_nacimiento.day,
    )

    if no_ha_cumplido:
        edad -= 1

    return {
        "estado": "calculable",
        "edad": edad,
        "unidad": "anios",
        "razon": "anios_cumplidos",
    }


print(calcular_edad_anios_cumplidos(date(2000, 12, 10), date(2026, 8, 4))["edad"])
print(calcular_edad_anios_cumplidos(date(2000, 2, 1), date(2026, 8, 4))["edad"])
```

Salida esperada:

```text
25
26
```

Esta función sirve para años cumplidos. No sirve para neonatología, edad gestacional, dosis pediátrica precisa, farmacocinética ni decisiones que dependan de meses o días. El contrato debe decirlo.

## CODE CLEAN: nombres que declaran tiempo y dominio

Comparemos dos nombres.

```python
def dias(a, b):
    return (b - a).days
```

El código es corto, pero no dice qué representa `a`, qué representa `b`, qué unidad promete, qué pasa si falta una fecha ni qué significa una duración negativa.

Una versión más honesta nombra el dominio.

```python
def calcular_dias_desde_inicio_sintomas(inicio_sintomas, fecha_consulta):
    return calcular_dias_entre_eventos(inicio_sintomas, fecha_consulta)
```

El nombre no lo resuelve todo, pero reduce una ambigüedad: no estamos restando fechas cualquiera. Estamos calculando un intervalo clínico específico. Código limpio no consiste en abreviar el tiempo; consiste en no esconderlo.

También conviene evitar nombres como:

```python
fecha1 = date(2026, 8, 1)
fecha2 = date(2026, 8, 4)
```

Preferir:

```python
fecha_inicio_sintomas = date(2026, 8, 1)
fecha_consulta = date(2026, 8, 4)
```

La segunda versión permite leer el cálculo sin mirar una tabla externa de significados.

## Límites y errores frecuentes

1. **Guardar fechas como cadenas sin contrato.** `"2026-08-04"` parece fecha, pero puede fallar en formato, zona, precisión o significado.
2. **Fingir precisión.** Si solo se conoce mes o año, no se debe inventar día para calcular una duración exacta.
3. **Confundir fecha con instante.** `date` no tiene hora; `datetime` puede tenerla, pero puede carecer de zona.
4. **Usar la fecha actual como reemplazo silencioso.** `date.today()` puede ser correcto para una consulta activa, pero no para completar datos faltantes sin declararlo.
5. **Aceptar duraciones negativas sin explicación.** Pueden revelar error de captura, evento futuro, regla invertida o dato fuera de contexto.
6. **Confundir intervalo abierto con error.** Un tratamiento activo o seguimiento en curso puede no tener fecha final.
7. **Olvidar unidad.** Un número temporal sin unidad pierde significado operacional.
8. **Restar datetimes de zonas distintas sin entender el resultado.** Python puede manejar instantes conscientes de zona, pero el dominio debe decidir qué comparación tiene sentido.
9. **Calcular edad sin declarar regla.** Años cumplidos, meses, días, edad gestacional y edad corregida son objetos distintos.
10. **Convertir aproximado en exacto por comodidad.** La comodidad técnica puede fabricar certeza documental.

## Argumentos críticos

### Desacuerdo 1: cadena ISO contra tipo temporal

Pregunta: ¿si una fecha está en ISO 8601, por qué no dejarla como texto?

El texto ISO es excelente para intercambio, almacenamiento y lectura humana. Pero para calcular, comparar y validar, conviene convertirlo a un tipo temporal. La cadena puede ser formato de transporte; el tipo temporal debe gobernar la operación.

Consenso operativo: usar formatos estables para almacenar o intercambiar; usar tipos temporales y validadores para decidir.

### Desacuerdo 2: fecha local contra instante con zona

Pregunta: ¿si el sistema se usa en un solo país, hace falta zona horaria?

No siempre. Una fecha de nacimiento o una fecha de consulta puede ser local por definición. Pero una hora de muestra, un evento de telemedicina, una transacción entre sistemas o una señal registrada por dispositivos puede requerir zona para ser comparable.

Consenso operativo: usar `date` cuando el día local es el dato; usar `datetime` con zona cuando el instante y la comparación horaria importan.

### Desacuerdo 3: cálculo exacto contra rango aproximado

Pregunta: ¿qué hacer si el paciente solo recuerda "hace una semana"?

Forzar una fecha exacta simplifica el código y deteriora el dato. Una alternativa es registrar aproximación, fuente y ventana posible. Otra es devolver `no_calculable` para cálculos que exigen precisión de día.

Consenso operativo: si la decisión depende de precisión exacta, no calcular. Si admite aproximación, representar rango y declarar incertidumbre.

### Desacuerdo 4: cerrar intervalos con hoy

Pregunta: ¿se puede calcular duración de un tratamiento activo usando la fecha de hoy?

Sí, si el resultado se nombra como duración hasta fecha de corte. No es lo mismo que duración total del tratamiento. El campo debe decir `fecha_corte`, `intervalo_abierto` o `duracion_hasta_corte`.

Consenso operativo: no cerrar intervalos abiertos de forma silenciosa.

## Puente hacia la frontera

El tiempo crece hasta convertirse en una estructura central de la medicina digital. Un paciente no es solo un conjunto de valores; es una secuencia de eventos. Un laboratorio aislado puede ser normal o alarmante según su trayectoria. Una exposición puede ser relevante según latencia. Una señal fisiológica puede depender de frecuencia de muestreo, ventanas móviles, sincronización y artefactos. Un modelo predictivo puede perder validez si usa información registrada después del evento que pretende predecir.

Más adelante, el libro tendrá que tratar series temporales, datos longitudinales, ventanas de observación, censura, supervivencia, leakage temporal, cohortes dinámicas, señales biomédicas y secuencias clínicas. Esta sección no resuelve esos temas. Establece una disciplina mínima: el tiempo debe representarse con precisión honesta antes de entrar a un algoritmo.

El principio seguirá vigente aunque el sistema sea más avanzado: una predicción longitudinal, una alerta clínica o un análisis de supervivencia dependen de saber qué ocurrió, cuándo ocurrió, con qué precisión se conoce, qué faltó y qué información estaba disponible en ese momento.

## Evaluar si entendiste

1. ¿Por qué una cadena con forma de fecha no basta para calcular de forma responsable?
2. ¿Qué diferencia hay entre fecha, instante, duración e intervalo?
3. ¿Por qué una fecha aproximada no debe convertirse en fecha exacta sin declararlo?
4. ¿Cuándo basta con `date` y cuándo conviene `datetime` con zona?
5. ¿Qué significa que un intervalo esté abierto?
6. ¿Por qué una duración negativa no debería aceptarse silenciosamente?
7. ¿Qué unidad debe conservar una diferencia temporal?
8. ¿Qué diferencia hay entre duración total y duración hasta fecha de corte?
9. ¿Por qué edad en años cumplidos no sirve para todos los usos biomédicos?
10. ¿Qué prueba escribirías para asegurar que una fecha faltante no produce duración cero?

## Vacíos de comprensión que debes vigilar

1. Creer que usar `datetime` resuelve automáticamente el significado clínico. Resuelve operaciones temporales, no contrato de dominio.
2. Pensar que toda fecha necesita hora y zona. Algunas fechas son deliberadamente locales y granulares al día.
3. Confundir aproximación con error. Una fecha aproximada puede ser el mejor dato disponible; el error aparece cuando se presenta como exacta.

## Orden de estudio para las próximas 3 horas

1. **Primera hora:** toma diez eventos clínicos o científicos y clasifícalos como fecha, instante, intervalo, duración o ventana.
2. **Segunda hora:** implementa un validador que rechace cálculos diarios cuando la precisión sea mensual o desconocida.
3. **Tercera hora:** escribe pruebas para fecha faltante, fecha futura, intervalo abierto, duración negativa y edad en años cumplidos.

## Bibliografía y fuentes

- Python Software Foundation. (2026). *datetime — Basic date and time types*. Python 3 documentation. <https://docs.python.org/3/library/datetime.html>
- Python Software Foundation. (2026). *zoneinfo — IANA time zone support*. Python 3 documentation. <https://docs.python.org/3/library/zoneinfo.html>
- International Organization for Standardization. (2019). *ISO 8601-1:2019 Date and time — Representations for information interchange — Part 1: Basic rules*. <https://www.iso.org/standard/70907.html>
- Health Level Seven International. (2026). *FHIR: Datatypes*. <https://hl7.org/fhir/datatypes.html>
- Health Level Seven International. (2026). *FHIR: Period*. <https://hl7.org/fhir/datatypes.html#Period>
- Health Level Seven International. (2026). *FHIR: Timing*. <https://hl7.org/fhir/datatypes.html#Timing>

## Siguiente paso

Las fechas muestran que un dato puede fallar por exceso de confianza: parecer exacto sin serlo. La próxima sección continuará con un problema todavía más común en bases biomédicas: ausencia de datos, valores centinela y marcadores especiales. Cuando un sistema usa `0`, `""`, `"NA"`, `999` o `None`, no siempre está diciendo lo mismo.
