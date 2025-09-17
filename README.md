# sistema-votacion

Mini app **0 €** para la votación del Concurso de Pintxos: **lista de bares** (foto, dirección, botón _Votar_) + pestaña de **Clasificación**.  
Frontend estático (GitHub Pages) que **lee datos de Google Sheets (GViz)** y usa **Google Forms** para recoger votos. Pensada para el barrio de **Uribarri (Bilbao)**, reutilizable cambiando el Sheet.

**Demo:** https://markelmartinez.github.io/sistema-votacion/

---

## 🧭 Cómo funciona (resumen)
1. Cada bar tiene un **enlace de voto** (Google Form pre-relleno con el nombre del bar).  
2. La web muestra tarjetas con **foto**, **dirección**, **Votar 1–10** y **Cómo llegar**.  
3. El Form guarda las respuestas en una hoja de cálculo.  
4. Una hoja llamada **Ranking** calcula **Puntos · Votos · Media** por bar.  
5. La web lee dos JSON (GViz) del Sheet: **Bares** y **Ranking**, y los pinta.

---

## 🧱 Arquitectura
- **Frontend:** `index.html` (HTML + CSS + JS plano).
- **Datos:** Google Sheets publicado → consultado como JSON via **GViz**.
- **Votación:** Google Forms (enlaces pre-rellenos por bar).
- **Hosting:** GitHub Pages (gratis).
- **Backend:** ninguno.

---

## 📦 Estructura del repo
<img width="379" height="95" alt="image" src="https://github.com/user-attachments/assets/92b5b110-2062-4ae7-8319-4033002cb2cf" />

> Imprescindible: **index.html** en la **raíz** del repo.

---

## 🗂️ Google Sheets: modelo de datos

### Hoja **Bares**
| Col | Contenido                                 | Uso en la app      |
|----:|-------------------------------------------|--------------------|
|  A  | **Bar** (nombre)                          | Título de la tarjeta |
|  B  | URL_base (no se usa)                      | —                  |
|  C  | **Enlace** (Form pre-relleno por bar)     | Botón **Votar 1–10** |
|  D  | Botón (texto)                             | —                  |
|  E  | **Dirección**                             | Dirección/MAPS     |
|  F  | Foto (no pública)                         | —                  |
|  G  | **Foto pública** (URL directa accesible)  | Imagen de la tarjeta |

**Importante:** las fotos deben ser **públicas**. En Drive usa:  
`https://drive.google.com/uc?export=view&id=ID_DEL_ARCHIVO`

**URL GViz – Bares** (A,C,E,G + `headers=1` para ignorar cabeceras):


> En este proyecto, tus valores actuales son:
>
> ```
> Bares (GViz):
> https://docs.google.com/spreadsheets/d/1o9-3b-GwN8Z8QEmTIBTAahP8sXPF8oMrn-AB7MQMTew/gviz/tq?tqx=out:json&gid=2039727451&tq=select%20A,C,E,G&headers=1
> ```

---

### Hoja **Ranking**
Debe exponer estas 4 columnas:
A = Bar
B = Votos
C = Media
D = Puntos

**Fórmula sugerida (ES) en `Ranking!A1`**  
Ajusta `Respuestas` al nombre real de tu pestaña de respuestas (p.ej. `Form Responses`)

=QUERY(
  {Respuestas!B2:B \ ARRAYFORMULA(N(Respuestas!C2:C))};
  "select Col1, count(Col2), avg(Col2), sum(Col2)
   where Col1 is not null and Col2 is not null
   group by Col1
   order by sum(Col2) desc, count(Col2) desc
   label Col1 'Bar', count(Col2) 'Votos', avg(Col2) 'Media', sum(Col2) 'Puntos'";
  0
)

URL GViz – Ranking:
https://docs.google.com/spreadsheets/d/SHEET_ID/gviz/tq?tqx=out:json&sheet=Ranking

valores actuales:
Ranking (GViz):
https://docs.google.com/spreadsheets/d/1o9-3b-GwN8Z8QEmTIBTAahP8sXPF8oMrn-AB7MQMTew/gviz/tq?tqx=out:json&sheet=Ranking

**⚙️ Configurar la app (index.html)**

Edita estas constantes con tus URLs GViz:

<script>
  const GVIZ_URL_BARES   = "https://docs.google.com/spreadsheets/d/1o9-3b-GwN8Z8QEmTIBTAahP8sXPF8oMrn-AB7MQMTew/gviz/tq?tqx=out:json&gid=2039727451&tq=select%20A,C,E,G&headers=1";
  const GVIZ_URL_RANKING = "https://docs.google.com/spreadsheets/d/1o9-3b-GwN8Z8QEmTIBTAahP8sXPF8oMrn-AB7MQMTew/gviz/tq?tqx=out:json&sheet=Ranking";
</script>


.

🔐 **Consejos de privacidad/antifraude**

Evita recoger datos personales.

Si quieres limitar duplicados: en el Form activa “Limitar a 1 respuesta” (requiere login Google).

Si recoges emails, notifícalo en el Form y establece un periodo de borrado (p. ej. 30–90 días).

🆘 **Problemas frecuentes**

404 en Pages: espera 1–5 min tras el commit o fuerza con un cambio menor.

CORS en local: no abras file://. Prueba en GitHub Pages o con servidor local.

No cargan fotos: usa URL pública directa (drive.google.com/uc?export=view&id=...) o imágenes públicas.

La cabecera aparece como fila: añade &headers=1 en la URL GViz (ya lo hace esta app).

Ranking vacío: mete 1 voto de prueba y revisa el nombre de la hoja de respuestas en la fórmula.
