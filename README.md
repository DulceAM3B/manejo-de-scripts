# Manejo de Scripts

Actividad escolar: un sitio estático del panel de liberaciones se prepara en `dist/`, se prueba por HTTP y, al integrar en `main`, GitHub Actions lo publica con GitHub Pages.

## Requisitos

- Git y Bash.
- Python 3 disponible como `python3` y `curl`. En Linux: `sudo apt install python3 curl`.
- En Windows, instala Git for Windows y ejecuta los comandos desde **Git Bash**. Si el comando de Python se llama `python`, usa `PYTHON_BIN=python bash scripts/probar.sh`.

## Ejecución local

```bash
bash scripts/preparar.sh
bash scripts/probar.sh
```

Para comprobar el caso fallido sin dañar `dist/`, crea una copia temporal, elimina su CSS y dirige la prueba a ella:

```bash
cp -R dist /tmp/panel-sin-css
rm /tmp/panel-sin-css/assets/styles.css
DIST_DIR=/tmp/panel-sin-css bash scripts/probar.sh
```

## Publicación en GitHub Pages

1. Repositorio en GitHub con archivos en rama `main`.
2. En el repositorio, abrir **Settings > Pages** y seleccionar **Source: GitHub Actions**.
3. Abrir la pestaña **Actions** para consultar “Liberar y desplegar panel”. Los pull requests a `main` solo preparan y prueban; los push a `main` también publican si todo pasa.
4. Cuando termine un despliegue exitoso, la URL real aparece en el resumen del trabajo “Publicar en GitHub Pages” y en **Settings > Pages**.

## Documentación del caso de estudio

Ver [docs/manejo-de-scripts.md](docs/manejo-de-scripts.md).
