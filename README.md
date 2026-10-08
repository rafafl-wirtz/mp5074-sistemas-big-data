# MP5074 · Sistemas de Big Data

Web de materiales del módulo **MP5074 Sistemas de Big Data** (IES Fernando Wirtz Suárez, curso 2026-27), hecha con [MkDocs](https://www.mkdocs.org/) y [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/), y publicada en **GitHub Pages**.

## Contenido

```
.
├── mkdocs.yml                     ← configuración del sitio y menú
├── requirements.txt               ← dependencias (mkdocs + material)
├── .github/workflows/deploy.yml   ← publicación automática en GitHub Pages
└── docs/
    ├── index.md                   ← portada: índice de contenidos
    ├── assets/stylesheets/extra.css
    ├── ud1/
    │   └── manuales/
    │       ├── index.md
    │       ├── wsl/manual_wsl.md
    │       ├── miniconda/manual_miniconda.md
    │       ├── jupyterlab/manual_jupyterlab.md
    │       ├── git/manual_git.md
    │       └── docker/manual_docker.md
    └── ud2/
        └── index.md
```

## Publicar en GitHub Pages (solo la primera vez)

0. **Mueve `deploy.yml` a `.github/workflows/deploy.yml`.** Es el flujo que publica la web. Si `.github/workflows/` ya contiene `deploy.yml`, borra el de la raíz.

    ```powershell
    mkdir .github\workflows -Force; move deploy.yml .github\workflows\deploy.yml
    ```

1. Crea un repositorio **vacío** en GitHub llamado `mp5074-sistemas-big-data` (sin README).
2. En `mkdocs.yml`, cambia `USUARIO` por tu usuario u organización de GitHub en `site_url` y `repo_url`.
3. Sube el proyecto desde esta carpeta:

    ```bash
    git init -b main
    git add .
    git commit -m "Web del módulo: UD1 y UD2"
    git remote add origin https://github.com/USUARIO/mp5074-sistemas-big-data.git
    git push -u origin main
    ```

4. En GitHub: **Settings → Pages → Build and deployment → Source: GitHub Actions**.
5. En **Actions** verás el flujo «Publicar en GitHub Pages». Cuando termine (≈1 min), la web estará en `https://USUARIO.github.io/mp5074-sistemas-big-data/`.

A partir de ahí, **cada `git push` a `main` vuelve a publicar la web** automáticamente.

## Trabajar en local

```bash
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
mkdocs serve                       # http://127.0.0.1:8000 · se recarga al guardar
```

## Añadir contenido

- **Nueva página:** crea el `.md` dentro de `docs/` y añádelo a `nav:` en `mkdocs.yml`.
- **Nueva unidad (UD3, UD4):** crea `docs/ud3/index.md` y añade una sección en `nav:` siguiendo el modelo de la UD2.
- **Imágenes:** guárdalas junto al `.md`, por ejemplo `docs/ud1/manuales/jupyterlab/imagenes/`, y enlázalas con una ruta relativa.
- **Avisos y recuadros:** `!!! tip "Título"`, `!!! warning`, `!!! info` ([admonitions](https://squidfunk.github.io/mkdocs-material/reference/admonitions/)).
- **Diagramas:** bloques ` ```mermaid `.

Los enlaces entre manuales usan rutas relativas (`../wsl/manual_wsl.md`), así que funcionan igual en GitHub, en VS Code y en la web.

> `requirements.txt` limita MkDocs a la versión 1.x: MkDocs 2.0 no es compatible con Material for MkDocs.
