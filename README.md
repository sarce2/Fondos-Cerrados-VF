# Panel de Fondos

Dashboard estático (un solo `index.html`, sin build ni dependencias) que muestra
el resumen de bancos, inversiones y deuda financiera por empresa. Los datos se
leen en vivo desde la hoja de Google Sheets publicada como CSV — no hay que
tocar el código para actualizar números, solo editar la planilla.

## 1. Subir esto a GitHub

1. Creá un repositorio nuevo (público o privado, cualquiera funciona con
   GitHub Pages en un plan normal; si es privado necesitás GitHub Pro/Team/Enterprise
   para publicarlo con Pages).
2. Subí estos dos archivos a la raíz del repo:
   - `index.html`
   - `.nojekyll`
3. Hacé commit y push (o subilos desde la interfaz web de GitHub con
   "Add file → Upload files").

No hace falta ningún paso de build: son archivos estáticos, listos para servir.

## 2. Activar GitHub Pages

1. En el repo, ir a **Settings → Pages**.
2. En "Source" elegir **Deploy from a branch**.
3. Branch: `main` (o la que corresponda), carpeta `/ (root)`.
4. Guardar. GitHub te va a dar una URL del tipo
   `https://<tu-usuario>.github.io/<repo>/` en uno o dos minutos.

Listo — esa URL ya muestra el dashboard actualizado en tiempo real.

## 3. Cómo se actualizan los datos

El `index.html` no tiene ningún número hardcodeado: al cargar la página hace
`fetch` al link CSV publicado de la planilla:

```
https://docs.google.com/spreadsheets/d/e/2PACX-1vTcpCTzLKNkGf9YikuvFh44yyf0bV0N0xqYuy6hXKAuW8Gv6mM6q1cueIairVRxTg/pub?output=csv
```

Esa URL está guardada en la constante `DATA_CSV_URL` dentro del `<script>` del
`index.html`. Para que el dashboard siga funcionando:

- **No revoques** la publicación de esa hoja (Google Sheets → Archivo →
  Compartir → Publicar en la web). Si la desactivás o cambiás el documento
  publicado, el dashboard deja de poder leer datos.
- Para actualizar los números, simplemente editá la pestaña **DATOS** de la
  planilla (que a su vez se arma sola desde las pestañas de cada fondo, según
  la nota que ya tenía la hoja). Los cambios tardan normalmente unos segundos
  a un par de minutos en reflejarse, porque Google cachea un poco la
  publicación.
- Si en algún momento cambiás el link publicado (por ejemplo porque lo volvés
  a publicar desde cero), hay que actualizar el valor de `DATA_CSV_URL` en
  `index.html` y volver a subir el archivo al repo.

### Estructura que espera la hoja DATOS

El parser busca estas columnas por nombre de encabezado, así que si se
mantiene el formato actual de la hoja no hay que tocar nada:

**Bloque de bancos e inversiones**
```
fondo          moneda_fondo   seccion   nombre        moneda   monto
Link Center (USD)  USD        banco     GNB Banco     US$      6,695,786
...
```
- `fondo`: nombre de la empresa seguido de su moneda entre paréntesis, p. ej.
  `Eco 1 (Gs.)`. Esta columna es la clave que une este bloque con el de deuda.
- `seccion`: `banco` o `inversion`.
- Las filas con `nombre` vacío (plantilla sin cargar) se ignoran automáticamente.

**Bloque de deuda financiera** (debajo de una fila que diga literalmente
`DEUDA FINANCIERA`)
```
fondo              saldo        fecha_pago   capital   interes   cuota
FINI (USD)         3,964,041    30/11/2026   36,191    25,739    61,930
```
- `fondo` debe coincidir exactamente con el mismo texto usado en el bloque de
  arriba (p. ej. `FINI (USD)`), para que el dashboard sepa a qué empresa
  corresponde la deuda.
- Una empresa sin fila en este bloque simplemente se muestra como "sin deuda
  registrada" / "pendiente de carga".

La fila `actualizado` (celda B3 de la hoja) es opcional: si le ponés texto
ahí, ese texto se muestra tal cual junto a "Actualizado" en el encabezado del
dashboard; si la dejás vacía, se muestra la fecha/hora en que el navegador
cargó la página.

## 4. Si el navegador no logra cargar los datos (CORS / bloqueo)

El link `pub?output=csv` de Google Sheets normalmente permite ser leído desde
cualquier página con `fetch()`. Si alguna vez ves el mensaje de error
"No se pudo cargar la planilla publicada" dentro del dashboard:

- Confirmá que el link abre bien en una pestaña nueva del navegador y
  devuelve un CSV (texto plano con comas).
- Confirmá que la hoja sigue "Publicada en la web" (no solo "Compartida").
- Probá recargar — a veces es un corte temporal de Google, no del dashboard.

## Archivos de este repo

- `index.html` — el dashboard completo (HTML + CSS + JS en un solo archivo).
- `.nojekyll` — le dice a GitHub Pages que sirva los archivos tal cual, sin
  pasarlos por el procesador Jekyll (no es estrictamente necesario con un solo
  `index.html`, pero evita sorpresas si más adelante agregás archivos que
  empiecen con `_`).
