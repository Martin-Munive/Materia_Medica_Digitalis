<div align="center">

![Materia Médica Digitalis](assets/materia-medica-digitalis-banner.svg)

# Materia Médica Digitalis

**Algoritmos en Python para Medicina y Ciencias de la Vida**

**Martin Munive**<br>
Médico General<br>
Analista y programador de software

[![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter Book](https://img.shields.io/badge/Jupyter%20Book-Sphinx-F37626?logo=jupyter&logoColor=white)](https://jupyterbook.org/)
[![Libro web](https://img.shields.io/badge/libro%20web-GitHub%20Pages-222222?logo=githubpages)](https://martin-munive.github.io/Materia_Medica_Digitalis/)
[![Licencia](https://img.shields.io/badge/licencia-MIT-16A34A)](LICENSE)
[![Estado](https://img.shields.io/badge/estado-reconstruccion%20editorial-f59e0b)](#estado-del-proyecto)

</div>

> Libro web técnico-científico para estudiar algoritmos desde problemas biomédicos: datos clínicos, decisiones, riesgo, estructuras de información, seguimiento de pacientes y sistemas computacionales para ciencias de la vida.

## Leer el libro

- **Sitio web:** <https://martin-munive.github.io/Materia_Medica_Digitalis/>
- **Portada editorial:** <https://martin-munive.github.io/Materia_Medica_Digitalis/chapters/00-portada.html>
- **Prefacio:** <https://martin-munive.github.io/Materia_Medica_Digitalis/chapters/00-prefacio.html>
- **Presaberes mínimos:** <https://martin-munive.github.io/Materia_Medica_Digitalis/chapters/00-presaberes-minimos.html>

## Qué es

`Materia Médica Digitalis` es un libro web sobre algoritmos, estructuras de datos y pensamiento computacional aplicado a medicina y ciencias de la vida.

No es una colección de apuntes ni un curso rápido de sintaxis. La obra usa Python como lenguaje de trabajo, pero su objetivo central es enseñar a razonar con algoritmos: traducir un fenómeno biomédico a datos, reglas, estados, excepciones, estructuras verificables y decisiones responsables.

El libro se ubica en la intersección entre:

- medicina clínica y razonamiento bajo incertidumbre;
- programación en Python como instrumento de pensamiento;
- estructuras de datos, algoritmos clásicos y complejidad;
- ciencia de datos biomédicos, bioinformática, genética computacional y medicina digital.

## Por qué importa

La medicina contemporánea y las ciencias de la vida producen datos, señales, texto, imágenes, secuencias y decisiones. Sin pensamiento algorítmico, esos materiales se convierten en listas dispersas, automatizaciones frágiles o modelos que parecen inteligentes pero ocultan sus supuestos.

Este libro parte de una idea simple:

> Un algoritmo no es una receta vacía. Es una forma de hacer explícita una decisión.

En un dominio biomédico, esa decisión rara vez vive sola. Aparece dentro de datos incompletos, umbrales, sesgos, trazabilidad, consecuencias clínicas, límites regulatorios y responsabilidad técnica.

## Qué aprenderás

- Formalizar decisiones biomédicas como procedimientos verificables.
- Representar datos, estados, reglas, condiciones, excepciones y trazabilidad.
- Usar Python para construir ejemplos claros, ejecutables y auditables.
- Entender algoritmos clásicos con lectura biomédica, no como ejercicios genéricos.
- Conectar fundamentos con estructuras de datos, análisis cuantitativo, bioinformática, genética computacional, neurología computacional y medicina de precisión.
- Reconocer por qué un algoritmo correcto puede ser clínicamente insuficiente si ignora contexto, sesgo o seguridad.

## Para quién es

Este libro está escrito para:

- estudiantes y profesionales de medicina que quieren entrar a programación, ciencia de datos o inteligencia artificial;
- investigadores en ciencias de la vida que necesitan automatizar análisis, ordenar datos y construir herramientas reproducibles;
- programadores interesados en aplicaciones biomédicas con sentido clínico y científico;
- lectores que quieren aprender algoritmos desde casos concretos, no desde ejemplos abstractos.

No se asume formación avanzada en ciencias de la computación. Sí se espera curiosidad, paciencia y voluntad de pensar con precisión.

## Arquitectura del libro

La proyección editorial completa está organizada en diez grandes unidades:

1. **El lenguaje de las decisiones:** algoritmos, pasos, variables, estados, condiciones, excepciones, bucles, funciones y seguridad del cálculo.
2. **Python como instrumento clínico-científico:** tipos de datos, tablas simples, limpieza, cálculo reproducible, automatización y validación.
3. **Algoritmos fundamentales con lectura biomédica:** búsqueda, ordenamiento, hashing, conteo, complejidad, recursividad, algoritmos voraces, programación dinámica y grafos.
4. **Datos médicos y razonamiento cuantitativo:** sensibilidad, especificidad, valores predictivos, riesgo, scores, triage, series temporales, calibración y sesgo.
5. **Estructuras de datos para ciencias de la vida:** arrays, matrices, tablas clínicas, grafos, árboles, pilas, colas, índices y representación de conocimiento médico.
6. **Algoritmos para datos biomédicos:** riesgo cardiovascular, urgencias, laboratorio longitudinal, contraindicaciones, sistemas de priorización, reglas clínicas, señales, texto e imagen biomédica.
7. **Bioinformática y genética computacional:** secuencias, similitud, alineamiento, programación dinámica, genomas, variantes, redes génicas y datos ómicos.
8. **Neurología computacional y sistemas complejos:** señales neurológicas, grafos cerebrales, modelos dinámicos, redes neuronales, aprendizaje y plasticidad.
9. **Del algoritmo al sistema:** organización de proyectos, pruebas, documentación, interfaces, ética, seguridad y responsabilidad.
10. **Frontera, modelos y responsabilidad:** medicina de precisión, modelos fundacionales, soporte de decisión, agentes, explicabilidad, incertidumbre y límites de automatización.

## Estado del proyecto

Estado actual: **libro web en reconstrucción editorial activa**.

Implementado:

- Jupyter Book clásico con `jupyter-book<2`.
- Sphinx Book Theme.
- Publicación objetivo en GitHub Pages.
- Portada editorial.
- Prefacio.
- Guía de lectura.
- Presaberes mínimos.
- Primera unidad conceptual: `El lenguaje de las decisiones`.
- Secciones publicables sobre algoritmos, pasos, variables, estados, excepciones, condicionales, bucles, funciones y seguridad del cálculo.
- Glosario vivo.
- Apéndice de entorno.
- Documentación interna de roadmap, auditoría editorial, estrategia de código limpio y estructura heredada.

Siguiente dirección editorial:

- cerrar la primera unidad con funciones puras, efectos, coordinación de procesos, pruebas y verificación mínima;
- abrir la segunda unidad sobre tipos de datos para problemas biomédicos;
- mantener la línea transversal `CODE CLEAN`: versión ingenua, crítica técnica, versión mejorada, salida esperada y prueba mínima.

## Instalación local

Requisitos:

- Python 3.12 o compatible.
- PowerShell, Windows Terminal o una terminal equivalente.

Crear entorno:

```powershell
python -m venv venv
.\venv\Scripts\python.exe -m pip install -r requirements.txt
```

Construir el libro:

```powershell
.\venv\Scripts\jupyter-book.exe build .
```

Abrir el resultado local:

```text
_build/html/index.html
```

## Estructura del repositorio

```text
README.md
  -> portada pública del repositorio

chapters/
  -> contenido publicable del libro

docs/
  -> documentación editorial interna del proyecto

_toc.yml
  -> navegación del libro web

_config.yml
  -> configuración de Jupyter Book / Sphinx

_static/custom.css
  -> estilo visual de la portada web y componentes editoriales
```

## Principios editoriales

- El libro debe enseñar Python sin reducirlo a sintaxis.
- Los ejemplos biomédicos son miniaturas pedagógicas, no escalas clínicas validadas.
- Cada tema importante debe conectar definición, intuición, límites, código, verificación y puente hacia frontera.
- La bibliografía y las fuentes sirven para sostener afirmaciones, no para transferir texto al libro.
- El contenido publicable debe mantener voz propia, trazabilidad conceptual y responsabilidad técnica.

## Límites

- Este libro no reemplaza juicio clínico, guías médicas, validación institucional ni revisión por especialistas.
- Los ejemplos médicos son educativos y no deben usarse como herramientas asistenciales.
- El repositorio público no debe mezclar documentación interna de planeación, notas de proceso ni material no publicable dentro del libro.
- Las fuentes privadas o protegidas solo pueden orientar estudio; no deben copiarse ni parafrasearse de forma cercana.

## Roadmap público

1. Cerrar la unidad I con funciones puras, efectos, coordinación y verificación mínima.
2. Abrir la unidad II con tipos de datos para problemas biomédicos.
3. Construir capítulos sobre listas, diccionarios, tablas simples, limpieza y validación.
4. Reintroducir complejidad, estructuras de datos y pruebas desde la nueva arquitectura editorial.
5. Avanzar hacia algoritmos clásicos con casos biomédicos concretos.
6. Conectar datos clínicos, bioinformática, genética computacional, señales, texto e imagen biomédica.
7. Consolidar la progresión hacia sistemas responsables, frontera científica y medicina digital.

## Autor

**Martin Munive**  
Médico General. Analista y programador de software.

## Licencia

El código y los materiales del repositorio se distribuyen bajo licencia MIT, salvo que un archivo específico indique otra cosa.
