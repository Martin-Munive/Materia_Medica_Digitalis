# Apéndice A. Entorno de trabajo

Este libro usa Python como lenguaje principal.

La edición web está construida con Jupyter Book y publicada como sitio estático en GitHub Pages.

## Requisitos locales

Para construir el libro:

```bash
pip install -r requirements.txt
jupyter-book build .
```

Salida esperada:

```text
El primer comando instala las dependencias declaradas.
El segundo comando construye el sitio HTML del libro.
```

La salida HTML se genera en:

```text
_build/html
```

## Nota

No todos los capítulos necesitan ser notebooks. La regla editorial es usar notebooks cuando ejecutar código sea parte esencial del aprendizaje, y Markdown cuando el objetivo sea explicar, ordenar o argumentar.
