# Handoff: Repositorio Geoespacial — MOPC / Franja Costera de Asunción

## Overview
Repositorio interno para centralizar todos los datos georreferenciados del proyecto **Resiliencia Urbana de la Franja Costera de Asunción** (MOPC). Tres módulos: un dashboard de estado, un explorador de mapa con capas agrupadas por eje temático (urbano / social / ambiental) que se prenden y apagan, y una descarga de cada dataset como ZIP (carpeta completa, porque un shapefile necesita sus 6 archivos acompañantes para abrirse). Incluye una pantalla de administración para subir nuevos datos.

## About the Design Files
Los archivos HTML de esta carpeta son **referencias de diseño**, no código de producción. Están construidos en HTML/CSS/JS plano con datos de ejemplo hardcodeados (21 capas ficticias, un ZIP generado en el navegador, "últimos archivos" simulados). La tarea es **reimplementar este diseño y su lógica en un stack real**, conectado a almacenamiento verdadero — no copiar el HTML tal cual a producción.

## Fidelity
**Alta fidelidad (hifi)**: colores, tipografía, espaciado y layout son finales (sistema de diseño "Nocturne", ver Design Tokens). Recreen la UI con esta fidelidad visual. La lógica de datos (carga de archivos, listado, ZIP) es una simulación funcional en el navegador — su comportamiento (qué pasos sigue, qué información pide, qué genera) es el que debe reproducirse contra datos reales, no su implementación interna (canvas de ZIP hecho a mano, arrays en JS).

## Arquitectura de datos recomendada — Google Drive como storage

**Unidad real del proyecto**: https://drive.google.com/drive/u/2/folders/1wZRVnMicmCuvqWUDd5tcmABx5HfyV1Ne (folder ID `1wZRVnMicmCuvqWUDd5tcmABx5HfyV1Ne`) — usar este ID como raíz para las llamadas a la Drive API.

El usuario ya tiene todos sus datos organizados en una unidad de Google Drive, con carpetas por dataset (cada carpeta contiene el `.shp` + sus sidecars `.shx .dbf .prj .cpg .qmd`, o el `.tif`, `.geojson`, etc.). La recomendación es **no migrar los archivos** — usar Drive como está:

1. **Storage → Google Drive** (gratis, 15GB en cuenta personal o el pool de Drive institucional).
   - Usar la **Google Drive API v3** (cuenta de servicio o OAuth) para:
     - listar carpetas/archivos de la unidad raíz del proyecto,
     - descargar un archivo puntual,
     - **generar el ZIP de una carpeta al vuelo** en un backend (Drive no zippea carpetas vía API — hay que descargar cada archivo del folder y comprimirlo en el servidor con una librería de ZIP, p. ej. `archiver` en Node o `zipfile` en Python, y devolver el stream al navegador).
   - Cachear el listado (los `list` de Drive son lentos si hay muchas carpetas) — refrescar cada N minutos o al detectar cambios vía `changes.watch`.

2. **Metadatos/índice → Google Sheets** (gratis, vía **Google Sheets API**), como sustituto liviano de una base de datos:
   - Una fila por dataset: `id, nombre, carpeta_id (Drive), eje (urb/soc/amb), formato, epsg, licencia, fecha_carga, autor, tamaño, geometry_type, feature_count, bbox (para centrar el mapa)`.
   - El equipo puede seguir editando esta hoja a mano o vía el panel de "Administrar".
   - Alternativa sin límite de filas si el catálogo crece mucho: Firestore free tier o Supabase free tier — pero para decenas/cientos de capas, Sheets alcanza y es gratis sin tarjeta.

3. **Geometría para el mapa**: no cargar el `.shp` binario en el navegador. Por cada capa, generar **una vez** (al publicar) un `.geojson` simplificado (usar `mapshaper` o `ogr2ogr -f GeoJSON` reproyectando a EPSG:4326) y guardarlo junto al dataset en Drive o cacheado en el backend. El mapa (Leaflet) consume ese GeoJSON liviano; el ZIP de descarga sigue entregando los archivos originales completos.

4. **Hosting gratuito del sitio**: Vercel o Netlify (funciones serverless gratis para el endpoint de ZIP) + GitHub para el código fuente. GitHub Pages también sirve si el ZIP se genera client-side contra archivos públicos de Drive, pero un backend serverless es más robusto para autenticar el acceso a Drive y no exponer credenciales.

## Screens / Views

### 1. Dashboard (`data-view="dashboard"`)
**Propósito**: vista de estado general al entrar.
**Layout**: grid de 3 columnas (áreas CSS grid nombradas `kpi/rec/pie/eje`), con un selector de 3 disposiciones alternativas (A/B/C) que reordena las mismas tarjetas vía `grid-template-areas`. Máx ancho 1560px, padding 32px.
**Componentes**:
- **4 tarjetas KPI** (`.kpi`): número grande (34px, Inter 500), etiqueta debajo (11.5px, gris), marca de acento de 22×2px arriba. Deben calcularse desde datos reales: total de capas, carpetas en la unidad, formatos distintos, peso total.
- **Tabla "Últimos 5 archivos subidos"**: ordenada por fecha de carga descendente, cada fila con ícono de formato, nombre, ruta de carpeta, tag de eje (color por eje), fecha/tamaño/autor, botón "ZIP" que dispara la descarga de esa carpeta.
- **Donut de composición por tipo de dato**: SVG generado a partir de conteo de formatos (Shapefile/GeoTIFF/DWG/CSV/GeoJSON/KML), con leyenda interactiva (hover resalta el gajo).
- **Tarjetas por eje** (urbano/social/ambiental): conteo de capas, descripción corta, chips con nombres de datasets, botón "Ver en el mapa" que navega al mapa con solo esas capas activas.

### 2. Explorar mapa (`data-view="mapa"`)
**Propósito**: explorar geoespacialmente y descargar por capa.
**Layout**: 2 columnas — sidebar fijo 326px + mapa a la derecha (`flex:1`), altura completa bajo el header (56px).
**Sidebar**:
- Buscador de capas (filtra por nombre o carpeta).
- Árbol agrupado por eje: header de grupo colapsable (nombre, contador "activas/total", toggle que prende/apaga todo el eje) + lista de capas.
- Cada fila de capa: toggle on/off, swatch de color del eje, nombre (trunca con ellipsis), botón de info (expande metadatos: formato, EPSG, licencia, carpeta, archivos del set) y botón de descarga individual.
- Pie del sidebar: contador de capas activas + botón "Descargar activas (ZIP)" que empaqueta todas las prendidas en un solo ZIP.
**Mapa**: Leaflet sobre tiles **CartoDB Dark Matter** (`https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png`, atribución CARTO + OSM obligatoria). Geometrías coloreadas por eje (urbano `#b5abfc`, social `#f0a79a`, ambiental `#7fd3a5`): puntos = `circleMarker`, líneas = `polyline`, polígonos = `polygon` con relleno 16% de opacidad. Popup al click con nombre, eje, formato, cantidad de registros, EPSG, carpeta y botón de descarga. Leyenda flotante de ejes abajo a la izquierda; nota flotante arriba a la derecha explicando que la geometría es una vista previa.
**Comportamiento crítico a preservar**: al activar la pestaña de mapa o navegar desde "Ver en el mapa", el mapa debe invalidar su tamaño y encuadrar (`fitBounds`) las capas activas — si el contenedor no tiene tamaño real en el momento del fit, Leaflet calcula mal el zoom (bug ya resuelto en el prototipo: esperar dos `requestAnimationFrame` tras mostrar la pestaña antes de fitear).

### 3. Administrar (`data-view="admin"`)
**Propósito**: subir nuevos datasets y ver el catálogo completo.
**Layout**: columna única, máx 1180px.
**Componentes**:
- **Zona de arrastre** (drag & drop + click-to-browse) que agrupa los archivos soltados por nombre base (todo lo que comparte nombre antes de la extensión es un dataset) y detecta si un shapefile está incompleto (le falta algún `.shp/.shx/.dbf/.prj/.cpg/.qmd`) mostrando un tag de advertencia.
- **Formulario de metadatos** por lote: eje temático (select), sistema de coordenadas/EPSG (select), licencia (select), nota libre. Botones "Descartar" / "Publicar en el repositorio".
- **Tabla de catálogo completo**: todas las capas con eje, formato, EPSG, licencia, carpeta de origen y botón de descarga individual.

### Modal de descarga (flujo de 3 pasos, se abre desde cualquier botón "ZIP")
1. **Datos de uso**: correo institucional, institución/dirección, uso previsto (select) — validación de campos obligatorios antes de continuar.
2. **Licencia y cita**: muestra la(s) licencia(s) de las capas incluidas, advertencia si hay alguna de "Uso interno MOPC", cita sugerida en formato APA-ish, lista del contenido que tendrá el ZIP, checkbox de aceptación obligatorio.
3. **Progreso + entrega**: lista de pasos con spinner→check secuencial ("Reuniendo carpetas…", "Verificando…", "Escribiendo metadatos…", "Comprimiendo…"), termina disparando la descarga del `.zip` y mostrando un resumen de archivos incluidos.

## Interactions & Behavior
- Cambiar de pestaña (`Dashboard` / `Explorar mapa` / `Administrar`) es un simple show/hide, sin recarga.
- El toggle "Prender/apagar" de una capa agrega/quita esa geometría del mapa en vivo; el toggle de "eje completo" prende/apaga todas las capas de ese grupo a la vez.
- El selector de disposición del dashboard (A/B/C) es solo reordenamiento visual — no cambia datos.
- Todas las descargas pasan por el modal de 3 pasos; no hay descarga directa sin ese flujo (registro de uso + aceptación de licencia).
- Publicar un lote en "Administrar" debe reflejarse de inmediato en: catálogo, recientes, KPIs, donut y árbol de capas del mapa (en el prototipo esto es un re-render en memoria; en producción sería una escritura a Sheets/Drive + invalidación de caché).
- Sin animaciones de entrada elaboradas — transiciones son opacidad/transform cortas (~150-200ms) en hovers, toggles y el toast de confirmación.

## State Management
Estado necesario en el frontend real:
- `catalog: Dataset[]` — el índice completo (viene de Sheets/API), cada item con `{id, name, folder, eje, fmt, geom, epsg, lic, size, date, author, feats, files[], bbox|geojsonUrl}`.
- `activeLayerIds: Set<string>` — qué capas están prendidas en el mapa (podría persistirse en la URL como query param para compartir vistas).
- `recent: Dataset[]` — derivado de `catalog` ordenado por fecha, top 5.
- `formatCounts` — derivado de `catalog` para el donut.
- Flujo de descarga: `downloadTargetIds[]`, `step (1|2|3)`, `form {email, institucion, uso}`, `agreed:boolean`.
- Flujo de carga: `stagedGroups[]` (archivos agrupados por nombre antes de publicar), `metaForm {eje, epsg, licencia, nota}`.
- Data fetching: listado de catálogo al montar (Sheets API o endpoint propio), generación de GeoJSON de capa on-demand o pre-generada, endpoint de ZIP que arma el archivo server-side desde Drive.

## Design Tokens
Sistema de diseño **Nocturne** (dark, Inter, radios 8px, densidad compacta). Tokens usados (ver `_ds/.../styles.css` incluido):
- `--color-bg: #161826` / `--color-surface: #232532` (panel usado en este diseño: `#1c1e2b`)
- `--color-text: #e9e9ed` · `--color-accent: #9184d9`
- Ramas neutrales 100–900 y accent 100–900 (ver archivo de tokens)
- Colores de eje (específicos de este diseño, no del DS base): urbano `#b5abfc`, social `#f0a79a`, ambiental `#7fd3a5`
- `--font-heading` / `--font-body`: Inter, peso 500 en headings
- `--space-1..8`: escala 0.7× (2.8px → 22.4px)
- `--radius-sm/md/lg`: 4px/8px/14px
- `--shadow-sm/md/lg`: bordes + sombra ambiental, sin sombras duras

## Assets
- Sin imágenes ni iconografía de marca — solo SVG inline (flechas, íconos de acción) dibujados a mano en el prototipo.
- Tiles de mapa: CartoDB Dark Matter (gratis, requiere atribución CARTO + OSM visible siempre).
- Fuente: Google Fonts Inter (400/500/600).

## Files
- `Repositorio Geoespacial.html` — el prototipo completo (dashboard, mapa, admin, modal de descarga), HTML/CSS/JS plano, sin build step. Ábranlo directo en el navegador para ver todos los estados e interacciones.
- `_ds/` — tokens y hoja de estilos del sistema Nocturne usados en el prototipo.
