# Conectar tu Google Drive al sitio (sin programar)

Esto le da al sitio permiso para leer tu planilla `resiliencia_metadata` y las carpetas de tu Drive. Lo hacés una sola vez.

## Parte 1 — Crear las llaves de acceso en Google Cloud

1. Andá a https://console.cloud.google.com/ y entrá con la cuenta de Google que tiene el Drive del proyecto.
2. Arriba, al lado del logo "Google Cloud", hacé clic en el selector de proyectos → **Proyecto nuevo**. Ponele un nombre, ej. `Repositorio Resiliencia Urbana`, y creá.
3. Con ese proyecto seleccionado, andá al buscador de arriba y escribí **"Google Drive API"** → entrá y tocá **Habilitar**.
4. Repetí lo mismo buscando **"Google Sheets API"** → **Habilitar**.
5. En el menú de la izquierda: **APIs y servicios → Pantalla de consentimiento OAuth**.
   - Tipo de usuario: **Externo** → Crear.
   - Completá nombre de la app (ej. "Repositorio Geoespacial MOPC"), tu correo, y guardá/continuá en cada paso hasta terminar.
   - En "Usuarios de prueba", agregá los correos de tu equipo (los mismos que ya usás para descargas) para que puedan iniciar sesión mientras la app no está "publicada".
6. Menú izquierdo: **APIs y servicios → Credenciales → + Crear credenciales → ID de cliente de OAuth**.
   - Tipo de aplicación: **Aplicación web**.
   - Nombre: lo que quieras.
   - En **"Orígenes de JavaScript autorizados"**, agregá la URL donde vas a publicar el sitio, por ejemplo `https://tu-usuario.github.io` (sin barra al final). Si todavía no la tenés, completá esto después de publicar en GitHub Pages (Parte 2) y volvé a editar la credencial.
   - Creá. Google te muestra un **Client ID** (termina en `.apps.googleusercontent.com`) — copialo, lo vas a pegar en el sitio.

## Parte 2 — Publicar el sitio en GitHub Pages

1. Entrá a https://github.com y creá una cuenta si no tenés.
2. Arriba a la derecha, **+ → New repository**. Nombre, ej. `repositorio-geoespacial`. Público. Creá.
3. Dentro del repo vacío, hacé clic en **"uploading an existing file"**.
4. Arrastrá **todos** los archivos y carpetas que descargaste en el paquete (`Repositorio Geoespacial.html`, `.nojekyll`, `image-slot.js`, la carpeta `assets/`, la carpeta `_ds/`). Confirmá el commit.
5. Andá a **Settings → Pages** (menú izquierdo).
6. En "Build and deployment", elegí **Deploy from a branch**, branch **main**, carpeta **/(root)** → **Save**.
7. Esperá 1-2 minutos y arriba te va a aparecer la URL pública, algo como `https://tu-usuario.github.io/repositorio-geoespacial/`.
8. **Importante**: si en la Parte 1 paso 6 no tenías esta URL todavía, volvé a Google Cloud → Credenciales → tu Client ID → agregá esa URL exacta (sin la parte final del archivo) en "Orígenes de JavaScript autorizados" → Guardar.

## Parte 3 — Compartir tu Drive con la app

1. Abrí tu carpeta de Drive del proyecto y tu planilla `resiliencia_metadata`.
2. Confirmá que la cuenta de Google que usás para "Conectar con Google" en el sitio (Parte 4) tiene al menos acceso de **Lector** a ambas — si es tu propia cuenta, ya tenés acceso.

## Parte 4 — Pegar el Client ID en el archivo (una sola vez)

Esta configuración ya no se escribe en pantalla — vive fija en el código, dentro de `Repositorio Geoespacial.html`.

1. Abrí `Repositorio Geoespacial.html` con un editor de texto simple (Bloc de notas, TextEdit, o el editor de archivos de GitHub haciendo clic en el lápiz ✏️ arriba a la derecha del archivo).
2. Buscá (Ctrl+F / Cmd+F) el texto `DRIVE_CONFIG`. Vas a ver algo así:
   ```js
   const DRIVE_CONFIG = {
     clientId: "",   // Client ID de Google Cloud (termina en .apps.googleusercontent.com)
     sheetId: "1GJ_KuQXlODZZJNYonmx7xfSMkaaHhbGunzWHb73Zm9k",
     range: "Table1!A:I",
     rootFolder: "1wZRVnMicmCuvqWUDd5tcmABx5HfyV1Ne"
   };
   ```
3. Entre las comillas de `clientId: ""` pegá el Client ID que copiaste en la Parte 1. Debe quedar así (con tu propio ID): `clientId: "123456789-abc.apps.googleusercontent.com",`
4. Si en el futuro cambiás de planilla o de carpeta de Drive, este es el único lugar que tenés que tocar: reemplazá `sheetId` y/o `rootFolder` por los IDs nuevos (la parte larga de letras y números en la URL de cada uno).
5. Guardá el archivo. Si lo editaste en tu computadora, volvé a subirlo a GitHub (arrastralo al repositorio, reemplazando el anterior). Si lo editaste directo en GitHub, el sitio se actualiza solo en 1-2 minutos.

## Parte 5 — Usar la conexión

1. Abrí tu sitio publicado y andá a la pestaña **Administrar**.
2. Tocá **Conectar con Google** → elegí tu cuenta → aceptá los permisos.
3. Tocá **Sincronizar planilla**. El catálogo, el mapa y la descarga se actualizan con tus datos reales.

El permiso de acceso (el login de Google) se pide una vez por navegador — no hace falta repetirlo cada vez que entrás, salvo que cambies de computadora o borres los datos de navegación.

## Si algo falla

- **"Falta configurar el Client ID..."**: no completaste el paso de la Parte 4 — revisá que quedó pegado entre las comillas, sin espacios de más.
- **"Error de acceso" al conectar**: revisá que la URL del sitio esté agregada en "Orígenes de JavaScript autorizados" (Parte 1, paso 6) exactamente igual, sin barra final.
- **"No has verificado esta app"**: es normal mientras la app está en modo de prueba. Hacé clic en "Avanzado" → "Ir a [nombre de la app] (no seguro)" — es tu propia app, es seguro continuar.
- **La planilla no sincroniza**: confirmá que el `sheetId` en `DRIVE_CONFIG` es el ID completo (la parte larga de letras y números entre `/d/` y `/edit` en el link de la planilla), y que los encabezados de la hoja son: `id, titulo, tipologia, subproyecto, fuente, formato, referencia, ubicacion` (más la columna de coordenadas).
