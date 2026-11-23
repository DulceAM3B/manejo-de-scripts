# Manejo de Scripts para la liberación y despliegue continuo de un panel de liberaciones

## Introducción

El objetivo de esta práctica es automatizar la creación de una versión candidata de un sitio estático, sus pruebas y su despliegue en GitHub Pages. Se emplean HTML y CSS para el sitio, Bash para los scripts y GitHub Actions para la integración y la publicación continua. El sitio muestra un panel de liberaciones continuas con el estado del pipeline y un historial de ejemplo.

## 1. Justificación del flujo de trabajo

El flujo definido en `.github/workflows/release.yml` se activa en cada `pull_request` dirigido a `main` y en cada `push` a `main`. En ambos casos, el trabajo `verificar` prepara `dist/` y ejecuta las pruebas. Por tanto, un cambio propuesto se revisa antes de integrarse. En un `push` a `main`, el trabajo `desplegar` depende de `verificar` mediante `needs: verificar`; GitHub Actions no lo inicia si falla la preparación o una prueba.

Las pruebas se realizan antes del despliegue para evitar errores concretos: una respuesta HTTP inexistente detecta que el sitio no puede servirse; la ausencia de `table`, `tbody` o del texto “Liberaciones” detecta una página incompleta o equivocada; y la ausencia o respuesta fallida del CSS detecta una referencia rota. La versión que supera esas pruebas se guarda como el artefacto `sitio-verificado`; el trabajo de despliegue descarga ese mismo artefacto, valida su estructura con `desplegar.sh` y lo entrega a Pages. Así, no se vuelve a construir un contenido diferente para publicar.

Este diseño justifica el pipeline porque:

- Reduce el riesgo de publicar una versión incompleta o rota.
- Garantiza que lo desplegado es exactamente lo que se probó.
- Ofrece feedback rápido en cada pull request sin publicar todavía.
- Separa la verificación (siempre) del despliegue (solo en push a `main`).

## 2. Entorno requerido para la liberación y el despliegue continuo

El ejecutor remoto es `ubuntu-latest`, que proporciona Bash, Python 3, `curl`, Git y el entorno de GitHub Actions. `scripts/preparar.sh` verifica Bash, `cp` y `find`, y crea una carpeta `dist/` limpia con `index.html` y `assets/styles.css`. `scripts/probar.sh` usa Python 3 para servir esta carpeta y `curl` para consultarla.

El destino de publicación es el entorno protegido `github-pages`. La publicación remota no la hace Bash: `scripts/desplegar.sh` únicamente confirma que la versión probada sigue completa. Los pasos `actions/upload-pages-artifact@v3` y `actions/deploy-pages@v4` suben el artefacto y realizan la publicación, respectivamente. En GitHub se debe configurar **Settings > Pages > Source > GitHub Actions** y permitir Actions en el repositorio.

Para la ejecución local se requiere:

- Git y Bash.
- Python 3 (`python3` o `py -3`) y `curl`.

## 3. Niveles de servicio acordados

Los siguientes valores son metas propuestas para un caso académico, no resultados medidos.

| Indicador | Objetivo | Forma de medición |
| --- | --- | --- |
| Disponibilidad mensual del sitio | ≥ 99 % | Sonda HTTP externa: `(mediciones correctas / mediciones totales) × 100` |
| Tiempo desde push a `main` hasta sitio visible | ≤ 10 minutos | Hora del push integrado − hora en que Pages informa la URL o una sonda observa el cambio |
| Tasa de éxito del pipeline | ≥ 90 % en 30 días | `(ejecuciones exitosas / ejecuciones totales) × 100` según el historial de Actions |
| Recuperación ante fallo de despliegue | ≤ 4 horas | Tiempo desde detección del fallo hasta un nuevo despliegue exitoso |

## 4. Métricas para el monitoreo de la aplicación

| Métrica | Forma de cálculo | Fuente de datos | Frecuencia |
| --- | --- | --- | --- |
| Disponibilidad del sitio | `(HTTP 2xx / solicitudes totales) × 100` | Sonda HTTP externa y registros de GitHub Pages | Cada 5 minutos; resumen mensual |
| Éxito del pipeline | `(ejecuciones exitosas / ejecuciones totales) × 100` | Historial de GitHub Actions | Por ejecución y resumen mensual |
| Duración de liberación | Hora final de despliegue − hora del push a `main` | Tiempos del workflow y del despliegue | Cada integración |
| Resultado local de preparación | Código de salida de `bash scripts/preparar.sh` | Consola local | Antes de entregar |
| Resultado local de prueba HTTP | Código de salida de `bash scripts/probar.sh` | Consola local | Antes de entregar |

La disponibilidad del sitio y el resultado del pipeline son métricas distintas: una URL puede responder aunque un pipeline posterior falle, y un pipeline puede pasar sin que una sonda externa haya medido la URL.

## 5. Parámetros de configuración de las herramientas utilizadas

| Parámetro | Valor | Archivo o lugar | Función |
| --- | --- | --- | --- |
| Rama | `main` | `release.yml` | Limita PR y push que activan el flujo |
| Eventos | `pull_request`, `push` | `release.yml` | Verifica propuestas e integra/publica cambios |
| Ejecutor | `ubuntu-latest` | `release.yml` | Aporta el entorno de CI |
| Versión candidata | `dist/` | `scripts/preparar.sh` | Contiene solo los archivos estáticos que se prueban |
| Preparación | `bash scripts/preparar.sh` | `release.yml` | Genera la versión candidata limpia |
| Pruebas | `bash scripts/probar.sh` | `release.yml` | Sirve y verifica HTML, tablas y CSS |
| Artefacto | `sitio-verificado` | `release.yml` | Conserva exactamente el contenido aprobado |
| Ruta del artefacto Pages | `dist` | `release.yml` | Indica el contenido que se carga a Pages |
| Entorno | `github-pages` | `release.yml` | Identifica el destino de despliegue |
| Acción de publicación | `actions/deploy-pages@v4` | `release.yml` | Publica remotamente el artefacto |

## Evidencia de los cuatro tipos de scripts

| Tipo solicitado | Archivo | Evidencia |
| --- | --- | --- |
| Flujo de trabajo (pipeline) | `.github/workflows/release.yml` | Eventos para `main`, trabajos `verificar` y `desplegar`, dependencia `needs: verificar` |
| Generación del entorno de liberación | `scripts/preparar.sh` | Crea `dist/` limpio con `index.html` y `assets/styles.css` |
| Pruebas en el entorno de liberación | `scripts/probar.sh` | Inicia servidor HTTP, verifica tablas/liberaciones/CSS y cierra el servidor con `trap` |
| Generación del despliegue | `scripts/desplegar.sh` | Valida `dist/`; el workflow usa `upload-pages-artifact` y `deploy-pages` para publicar |

## Registro de verificación local

Comandos esperados:

```bash
bash scripts/preparar.sh
bash scripts/probar.sh
```

La preparación debe generar `dist/index.html` y `dist/assets/styles.css`. Con `dist/` intacto, `bash scripts/probar.sh` debe terminar con código `0`.

Caso de fallo controlado:

```bash
cp -R dist /tmp/panel-sin-css
rm /tmp/panel-sin-css/assets/styles.css
DIST_DIR=/tmp/panel-sin-css bash scripts/probar.sh
```

Sobre la copia sin CSS, la prueba debe terminar con código `1` y el mensaje de que no existe el CSS referenciado. Eso demuestra que el script detecta la ausencia de la hoja de estilos.
