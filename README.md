# Sistema de votación de pintxos — Arquitectura y funcionamiento

Proyecto 0 € para montar una votación popular de pintxos con **frontend estático** (GitHub Pages) y **backend “serverless”** sobre **Google Sheets + Google Forms**.  
Sin bases de datos ni servidores: todo vive en tu hoja y en una página HTML.

---

## 🧩 Piezas del sistema

**1) GitHub Pages (frontend)**
- `index.html`: aplicación completa (HTML+CSS+JS).
- Imágenes de los bares y del “hero” (`*.png`, `pintxos.jpg`, `urtiko.png`).
- Se sirve en `https://<usuario>.github.io/<repo>`.

**2) Google Sheets (datos)**
- **Hoja “Bares”**: catálogo que la web lista en la pestaña *Bares*.  
  - Columnas que se leen vía **GViz**:  
    **A**: *Bar*, **C**: *Link para votar* (URL al Form del bar), **F**: *Dirección*, **H**: *Foto pública*.
- **15 hojas de respuestas**: una por bar, con dos columnas:
  - **A**: marca temporal (no se usa), **B**: *nota* (1–10).
  - Cada hoja se llama exactamente como el bar (p. ej. `Bar el Final`, `Uribarri Elkartea`, etc.).

**3) Google Forms (votación)**
- Un formulario **por bar** (evita “Ya has respondido” global de un único form).
- Cada form escribe en su **propia hoja de respuestas** (la de su bar).

---

## 🔁 Flujo de datos

- Usuario → (clic en "Votar") → Form del bar (Google Forms)
- Form → escribe respuesta (columna B) → hoja de respuestas del bar (Google Sheets)
- index.html → pide a GViz las respuestas B de cada bar → calcula en el cliente:

• PUNTOS = suma de notas (∑B)
• VOTOS = nº de respuestas (count(B))
• MEDIA = PUNTOS / VOTOS

- Orden → por PUNTOS (desc), después VOTOS (desc) como desempate
- UI → muestra tabla, podio, barras de progreso, etc.
  
---

## 📡 Cómo se conecta la web con Sheets

### Catálogo de bares (pestaña *Bares*)
Se lee con una sola URL **GViz** (seleccionando columnas A,C,F,H):

- Petición:
https://docs.google.com/spreadsheets/d/<SHEET_ID>/gviz/tq?tqx=out:json&gid=<GID_BARES>&tq=select%20A,C,F,H&headers=1


### Respuestas (pestaña *Clasificación en vivo*)
Se define un array `RESP_SHEETS` con **una URL GViz por bar**, apuntando a su hoja de respuestas y **select B** (solo la nota):

**(js)**
const RESP_SHEETS = [
  { bar: "Bar el Final",
    url: "https://docs.google.com/spreadsheets/d/<SHEET_ID>/gviz/tq?tqx=out:json&gid=<GID_FINAL>&tq=select%20B%20where%20B%20is%20not%20null" },
  // … uno por bar …
];

- La app añade un parámetro anti-caché &_=${Date.now()} en cada fetch para que los cambios salgan en vivo.

## ✨ Funcionalidad de la web

### Dos pestañas

**Bares**: tarjetas con foto, nombre, dirección (columna **F**) y botones:
- **Votar 1–10** → abre el Google Form del bar (columna **C**).
- **Cómo llegar** → abre Maps con *nombre + dirección*.

**Clasificación en vivo**:
- Tabla con **Bar**, **Puntos**, **Votos**, **Media**.
- **Podio dinámico** (1º, 2º, 3º) con mini-fotos.
- **Barras de progreso** (proporcionales a los puntos).
- **Reglas de ganador**: *más puntos* (suma de notas). Desempate por *más votos*.
- Marca de tiempo **“Actualizado: …”**.

### Bilingüe CAS / EUS
Botones **CAS** y **EUS**, con textos traducidos y persistencia en `localStorage` y parámetro `?lang=` en la URL.

### Búsqueda rápida
Filtro por nombre/dirección en la pestaña **Bares**.

### Imágenes
Cargan desde el propio repositorio (evita CORS de Drive).

### Decoración / UX
- Hero con imagen (`pintxos.jpg`) y slogan **“Gora Uribarri eta Matikoko jaiak!”** con logo (`urtiko.png`).
- Micro-interacciones (hover en cards/botones).
- Opción de **modo oscuro** (opcional; ver sección de opciones).
- **Confeti** cuando cambia el líder (opcional).

---

## 🗂️ Estructura del repo (mínimo)
<img width="525" height="160" alt="image" src="https://github.com/user-attachments/assets/393d0661-5b00-4041-a140-9fd64befbbcc" />


---

## 🔧 Cómo adaptar a otro barrio/edición

1. **Copia** esta hoja de Google Sheets y esta web.
2. En tu hoja:
   - Rellena **“Bares”** con: *Bar*, *Link al Form*, *Dirección*, *Foto pública*.
   - Crea **un Form por bar** y conéctalo a una **hoja de respuestas** cuyo nombre sea exactamente el del bar.
   - Comprueba el **gid** (id de pestaña) de cada hoja de respuestas.
3. En `index.html`:
   - Sustituye `<SHEET_ID>` y los **gid** en:
     - La URL **GViz** del **catálogo de Bares**.
     - Cada entrada de `RESP_SHEETS` para `select B` de las respuestas.
4. Sube imágenes al repo y usa su URL absoluta: https://<usuario>.github.io/<repo>/<imagen>.png
5. Activa **GitHub Pages** → *Build from* `main` → `/ (root)`.

---

## 🧪 Reglas de la clasificación

- **PUNTOS** = suma de todas las notas recibidas por el bar.  
- **VOTOS** = número de respuestas.  
- **MEDIA** = PUNTOS / VOTOS.  
- **Ganador** = bar con **más puntos**. En empate, **más votos**. *(La media queda de referencia).*

> Si quieres cambiar las reglas (p. ej. **ganador por media** con un **mínimo de 10 votos**), edita la función que ordena los resultados en el JS.

---

## ❗️ Limitaciones y consideraciones

- **Privacidad**: todo debe ser público (imágenes y hoja) para que el navegador pueda leerlo.
- **Sin servidores**: para eventos de barrio, Google Sheets/Forms es suficiente; para grandes volúmenes considera una API intermedia.
- **Caché**: la app añade un parámetro anti-caché en cada `fetch`; al cambiar imágenes, GitHub Pages puede tardar ~1–2 min en refrescar.

---

## 🚀 Roadmap (sugerencias)

- **Códigos QR** por bar y verificación básica anti-multi-voto (por dispositivo).
- **Mapa** con marcadores de los bares (ya hay ejemplo de mini-mapa).
- **Exportar CSV** de resultados desde la web.
- **Histórico** por días/ediciones.

---

## Créditos

Hecho con cariño para **Uribarri**.  
**Auzotik auzoarentzat**

