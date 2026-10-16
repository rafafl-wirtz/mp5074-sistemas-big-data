# Copia de seguridad: configuración con MkDocs + Material for MkDocs

El sitio se generaba con MkDocs y Material for MkDocs hasta el 16/10/2026. Desde entonces se genera con **Zensical** (configuración en `../zensical.toml`).

Esta carpeta guarda la configuración anterior, que sigue funcionando sobre los mismos contenidos de `../docs/`:

| Fichero | Qué es |
|---|---|
| `mkdocs.yml` | Configuración anterior (con `docs_dir: ../docs` para usar los mismos Markdown) |
| `requirements.txt` | Dependencias anteriores (MkDocs y Material for MkDocs) |
| `deploy-mkdocs.yml` | Workflow anterior de GitHub Actions (aquí **no** se ejecuta) |

## Probar la versión anterior en local

```bash
pip install -r back/requirements.txt
mkdocs serve -f back/mkdocs.yml
```

## Volver a publicar con MkDocs

1. Copia `back/deploy-mkdocs.yml` a `.github/workflows/deploy.yml` (sustituyendo al de Zensical).
2. Haz commit y push a `main`.

El estado exacto del repositorio antes de la migración también está en el historial de git (commit `bff3c34`).
