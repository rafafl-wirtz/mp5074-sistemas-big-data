# MP5074 · Sistemas de Big Data

Web de materiales del módulo **MP5074 Sistemas de Big Data** (IES Fernando Wirtz Suárez, curso 2026-27), hecha con [Zensical](https://zensical.org/) y publicada en **GitHub Pages**.

## Trabajar en local

```bash
pip install -r requirements.txt
zensical serve      # vista previa en http://localhost:8000
zensical build      # genera la web en site/
```

- Los contenidos están en `docs/` (Markdown).
- La configuración y el menú lateral están en `zensical.toml`.
- Cada `push` a `main` publica la web automáticamente (`.github/workflows/deploy.yml`).

La configuración anterior con MkDocs + Material for MkDocs está guardada en [`back/`](back/README.md).
