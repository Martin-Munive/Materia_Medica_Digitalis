# Mantenimiento de GitHub Actions

## Motivo

El despliegue del libro usa GitHub Actions para construir Jupyter Book y publicar GitHub Pages. Estas actions dependen de runtimes de Node.js administrados por GitHub. Cuando GitHub depreca una versión de Node, los workflows pueden seguir funcionando durante un periodo de transición, pero empiezan a emitir advertencias y eventualmente pueden fallar.

El 2026-08-05 se observó una advertencia en `deploy-book`: varias actions declaraban Node.js 20 y GitHub las estaba forzando a Node.js 24. El despliegue no falló, pero la advertencia indicaba deuda técnica.

## Regla operativa

No cerrar un ciclo editorial como completamente sano si GitHub Actions publica con advertencias de runtime deprecado.

Cuando aparezca una advertencia de este tipo:

1. Consultar las releases oficiales de cada action en GitHub.
2. Actualizar solo a versiones estables publicadas.
3. Mantener el cambio limitado al workflow.
4. Ejecutar el workflow remoto después del push.
5. Documentar el run exitoso en el cierre del handoff.

## Versiones actualizadas

Actualización aplicada el 2026-08-05:

| Action | Version anterior | Version aplicada |
|---|---:|---:|
| `actions/checkout` | `v4` | `v7` |
| `actions/setup-python` | `v5` | `v7` |
| `actions/configure-pages` | `v5` | `v6` |
| `actions/upload-pages-artifact` | `v3` | `v5` |
| `actions/deploy-pages` | `v4` | `v5` |

Las versiones se verificaron contra las releases oficiales de los repositorios `actions/*` mediante GitHub API.

## Criterio de verificación

El mantenimiento queda aceptado solo si:

- el build local del libro sigue funcionando;
- el workflow remoto `deploy-book` termina en `success`;
- no aparecen advertencias nuevas que indiquen cambio obligatorio de runtime;
- la publicación de GitHub Pages queda actualizada.

## Nota para futuros sistemas

La versión del ejecutable `jupyter-book.exe` local puede fallar silenciosamente en este entorno. Si ocurre, usar la entrada interna validada:

```powershell
.\venv\Scripts\python.exe -c "from jupyter_book.cli.main import main; raise SystemExit(main(['build', '.']))"
```

No cambiar la arquitectura del libro por ese fallo del wrapper; el build por la CLI interna ya fue validado.
