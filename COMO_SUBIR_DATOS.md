# Cómo subir y clasificar tus datos

## 1. Instalar / abrir el paquete
1. Descomprimí el ZIP descargado.
2. Abrí `Repositorio Geoespacial.html` directo con doble click en tu navegador (Chrome o Edge). No necesita instalación ni servidor — funciona local.
3. Para que tu equipo lo use desde un link, subí esa misma carpeta a Vercel, Netlify o GitHub Pages (ver `README.md`, sección de hosting).

## 2. Preparar tus archivos antes de subir
- Un shapefile SIEMPRE va en su propia carpeta con sus 6 archivos juntos: `.shp .shx .dbf .prj .cpg .qmd`. No subas el `.shp` solo.
- Otros formatos válidos: `.geojson`, `.tif` (+ `.tfw` si lo tenés), `.csv` con columnas de coordenadas, `.kml`/`.kmz`, `.dwg`.
- Nombre de archivo claro y sin espacios raros (ej: `red_pluvial_franja.shp`), porque ese nombre es el que va a mostrar el catálogo.

## 3. Subir en la pestaña "Administrar"
1. Andá a la pestaña **Administrar**.
2. Arrastrá los archivos (o carpeta) a la zona de carga, o hacé click para buscarlos.
3. El sistema agrupa automáticamente todo lo que comparte nombre base como un solo dataset, y te avisa si a un shapefile le falta algún archivo (`falta .prj`, por ejemplo) — completalo antes de publicar.

## 4. Clasificar por eje
En el formulario que aparece debajo de la carga, elegí el **Eje temático** para ese lote:
- **Datos urbanos** → usalo para infraestructura: vialidad, redes, catastro, planos de proyecto, topografía.
- **Datos sociales** → censo, vivienda, relocalización, equipamientos comunitarios.
- **Datos ambientales** → humedales, cuerpos de agua, riesgo de inundación, arbolado.

Si preferís que la etiqueta diga literalmente "Infraestructura" en vez de "Datos urbanos", decímelo y renombro esa categoría en toda la interfaz (dashboard, mapa, catálogo) — es un cambio de un minuto.

Completá también:
- **Sistema de coordenadas (EPSG)** — el que traiga tu archivo (usualmente EPSG:32721 para UTM 21S Paraguay).
- **Licencia** — CC BY 4.0, Uso interno MOPC, o Dominio público.
- **Nota** (opcional) — contexto para el equipo.

## 5. Publicar
Click en **"Publicar en el repositorio"**. El dataset aparece al instante en:
- el catálogo de Administrar,
- "Últimos 5 archivos subidos" del Dashboard,
- la torta de composición por tipo,
- el árbol de capas del mapa, dentro de su eje, listo para prenderse/apagarse.

## 6. Conectar a tu Drive real (para producción)
Este HTML es un prototipo funcional con datos de ejemplo — la carga en "Administrar" simula la publicación en memoria del navegador, no escribe todavía en tu Google Drive. Para que sea el repositorio real:
1. Pasale la carpeta `design_handoff_repositorio_geoespacial/` a un desarrollador (o a Claude Code).
2. El `README.md` adentro ya tiene la arquitectura: tu Drive (`https://drive.google.com/drive/u/2/folders/1wZRVnMicmCuvqWUDd5tcmABx5HfyV1Ne`) como almacenamiento de archivos, Google Sheets como índice de metadatos, y un backend chico que arma el ZIP real al descargar.
3. Con eso conectado, subir un archivo acá va a guardarlo de verdad en tu Drive, en la carpeta del eje correspondiente.
