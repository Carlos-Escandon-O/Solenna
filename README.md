# Solenna · Probador virtual

Probador virtual de joyería que funciona en el navegador, sin instalar nada.
Usa la cámara o una foto subida + MediaPipe (gratis) para colocar aretes, collar, anillos y pulseras en tiempo real.

## 🌐 Sitio en vivo

**https://carlos-escandon-o.github.io/Solenna/**

Publicado con GitHub Pages (HTTPS, funciona desde el celular).

**Instagram de la marca:** https://www.instagram.com/solennamx/ (enlazado en el botón del probador y en el pie de página).

## 📁 Estructura del proyecto

```
Solenna/
├── index.html              # web principal (probador virtual)
├── admin.html              # panel de administración del inventario
├── preparar-fotos.html     # herramienta auxiliar para preparar fotos
├── catalogo.json           # inventario real (25 piezas, editable desde /admin.html)
├── assets/                 # PNGs de joyería con fondo transparente
├── fotos-crudas/           # tus fotos sin recortar (para preparar_fotos.py)
├── preparar_fotos.py       # recorte automático (rembg + BiRefNet) → assets/
├── fotos_app.py            # interfaz web local para el mismo flujo de recorte
├── generar_qr.py           # regenera el cartel QR del bazar
├── cartel-qr-solenna.png   # cartel listo para imprimir
├── favicon.svg
└── ESPECIFICACIONES_FOTOS.md  # cómo fotografiar y entregar las joyas
```

## ✨ Funciones

- **Dos modos:** cámara en vivo o subir una selfie guardada (ideal si no quieres abrir la cámara en el bazar).
- **Catálogo interno de 17 piezas de ejemplo**: 16 de ejemplo + 1 pieza real (Collar Vaquero, $850), con filtros **Oro / Plata / Perlas / Piedras** y orden por precio o nombre.
- **`catalogo.json` como fuente de verdad del inventario**: 25 piezas editables desde el panel Admin (nombre, tipo, categoría, precio, stock, imagen). Los precios del `catalogo.json` están en 0 hasta que los configures en el panel; la web muestra "Consultar precio" para las piezas con precio 0.
- **Ajuste manual** de posición y tamaño, dedo para anillos, muñeca para pulseras, y opción de subir tu propia joya en PNG.
- **Pedir por WhatsApp:** abre un chat con el nombre y precio de la pieza ya escritos.
- **Foto descargable** y botón compartir.
- **Contador social diario.**

## ⚙️ Configuración ( importante )

Al inicio del `<script>` en `index.html` hay dos valores que debes editar:

```js
const WHATSAPP = "525539753964";                  // configurado con tu número
const BASE_PRUEBAS_REALES_POR_PIEZA = {};         // solo con pruebas reales documentadas
```

1. **WHATSAPP:** ya está tu número (`52` + `55 3975 3964`). El botón abre el chat con el nombre y precio de la pieza ya escritos.
2. **Contador:** es **real y por pieza**: cuenta una prueba por sesión, se reinicia cada día y muestra qué pieza está siendo más probada (ej. *"💫 4 personas se lo probaron hoy · Collar Perla ×2"*). Las variaciones salen solas porque cada persona elige una joya distinta.
   Si llevas un registro **verdadero** de pruebas anteriores, puedes cargarlo como base, por ejemplo: `{ a1: 2, c3: 1, r2: 1 }`. Déjalo vacío si no: un cartel con números inventados se nota y rompe la confianza del cliente (además de ser publicidad engañosa).

## 📱 Cartel QR para el bazar

![Cartel QR de Solenna](cartel-qr-solenna.png)

`cartel-qr-solenna.png` está listo para imprimir. Si cambias la URL, regenera:

```bash
python3 generar_qr.py   # requiere: pip install qrcode pillow
```

## 🔄 Cómo actualizar el sitio

Cada cambio en `index.html` se publica solo con:

```bash
git add . && git commit -m "describe el cambio" && git push
```

GitHub Pages tarda ~1 minuto en reflejarlo.

## 🔐 Panel de administración

**https://carlos-escandon-o.github.io/Solenna/admin.html**

El inventario vive en `catalogo.json`. El panel permite cambiar **nombre, tipo, categoría, precio, stock e imagen** de cada pieza, agregar o borrar piezas, y guardar directamente en el repo.

### Configuración (una sola vez)

1. En GitHub: **Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
2. **Generate new token**: nombre `Solena Admin`, expiración la que prefieras.
3. Repository access → **Only select repositories** → `Solenna`.
4. Permissions → **Contents: Read and write**.
5. Copia el token (solo se muestra una vez).
6. Abre `/admin.html`, pega el token y tu repo (`Carlos-Escandon-O/Solenna`), y entra.

El token queda guardado **solo en ese navegador**. No se envía a ningún servidor propio: el panel habla directamente con la API de GitHub. Cualquier otra persona puede ver la página del panel, pero **sin tu token no puede guardar nada**.

> Si prefieres no usar tokens: edita `catalogo.json` directamente en GitHub y haz commit. La web lee ese archivo al cargar.

### Quitar fondo con IA (dentro del panel)

El panel incluye un apartado **«Quitar fondo con IA»**: subes una foto de la joya y se recorta con el mismo tipo de segmentación que usa Canva, pero **en tu navegador**, sin API key ni servidor propio.

- La librería se carga desde CDN y el modelo (~40 MB) se descarga la primera vez; luego queda en caché.
- La imagen nunca se envía a un servidor externo: todo el procesamiento es local.
- Puedes **descargar el PNG**, **reemplazar la imagen de una pieza existente** o **crear un producto nuevo**: el panel rellena nombre, categoría (tipo de joya), material y precio de forma provisional a partir del nombre del archivo, y tú los editas antes de publicar.
- Al crear un producto nuevo se sube el PNG a `assets/` y se agrega la entrada a `catalogo.json` (ID, stock `1`, `src` y `recortar: false` se generan solos). No hace falta editar ningún archivo a mano.
- Para publicar necesitas el token con permiso `Contents: read and write`.

> La librería `@imgly/background-removal` es de código abierto bajo licencia **AGPL**; para un uso comercial se requiere licencia de IMG.LY.

### Stock

- `stock: 0` → la pieza aparece como **Agotado** y el botón de WhatsApp se desactiva.
- Si el archivo JSON no carga (sin conexión), la web usa un catálogo interno de respaldo.

## 🧼 Preparar fotos automáticamente

Script que quita el fondo, separa piezas y ajusta medidas en un solo paso:

```bash
pip install -r requirements.txt
# pon tus fotos en fotos-crudas/ con prefijo según el tipo:
# aretes-...  collar-...  anillo-...  pulsera-...
python3 preparar_fotos.py
```

- Detecta **varias piezas en una misma foto** (islas de transparencia) y las guarda por separado.
- Escala a 600×1200 (aretes), 1200×1200 (collares), 1200×675 (anillos) o 1200×720 (pulseras).
- Guarda en `assets/` y escribe `catalogo-sugerido.json` con las entradas para el panel Admin.
- Para fotos con **varias pulseras juntas**, funciona siempre que no se toquen.
- Si la joya está sostenida por una mano o se toca con otra, recorta esa foto antes.
- Usa el modelo **BiRefNet** (`birefnet-general`), mucho mejor que `u2net` para joyas delgadas. La primera corrida descarga ~900 MB y luego usa caché.

### Alternativas gratis si el recorte sigue saliendo mal

- **BRIA RMBG 2.0** (demo gratis): https://huggingface.co/spaces/briaai/BRIA-RMBG-2.0 — modelo BiRefNet especializado en quitar fondo.
- **Google Gemini**, **Adobe Firefly**, **Leonardo AI**, **Krea** o **Magic Hour** tienen capas gratis de edición por chat parecida a ChatGPT.

Si ya tienes la joya con fondo transparente desde otra herramienta, puedes saltarte el paso de `rembg`: pon el PNG directamente en `assets/` y usa el panel Admin para darle nombre, precio y stock.

## 📸 Poner tus piezas reales

1. Foto de cada joya de frente, bien iluminada y centrada, con fondo liso o transparente (PNG). Ver `ESPECIFICACIONES_FOTOS.md`.
2. **La fuente de verdad del inventario es `catalogo.json`** (no `CATALOGO` dentro de `index.html`, que queda como respaldo si no carga el JSON). El modo recomendado: abre `/admin.html` y agrega/edita desde ahí; o edita el JSON a mano:

```json
{ "id": "c13", "nombre": "Collar Vaquero", "tipo": "collar", "cat": "oro", "precio": 850, "stock": 3, "src": "assets/collar-vaquero.png" }
```

3. Las imágenes van en `assets/` y se recortan con fondo transparente (ver `ESPECIFICACIONES_FOTOS.md` o usa `preparar_fotos.py`).
4. Con 5–10 piezas reales ya es usable como catálogo; hoy hay 25 piezas reales con `precio: 0` esperando que las personalices en el panel Admin.

## Notas

- **Oclusión:** el cuello tapa la parte trasera del collar, y la muñeca o el dedo tapan la parte de atrás de pulseras y anillos, para que se vean puestos y no dibujados encima.
- El collar es aproximado (la malla facial no cubre el cuello), por eso tiene controles de ajuste. Aretes, anillos y pulseras son más precisos.
- Funciona mejor con buena luz y la cara o mano bien visibles.
- Las fotos que sube cada persona no se envían a ningún servidor: se procesan en su propio navegador.
- Librerías: MediaPipe Tasks Vision v0.10.14 (CDN). Sin costos ni API keys.
