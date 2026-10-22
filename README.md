# calendar-final-boss

Calendario de cuenta atrás hasta el **2 de diciembre de 2026**. Los días pasados se tachan solos, y el día de hoy y los futuros se pueden tachar tocándolos. Todo se guarda en el navegador. Al tachar suena un pequeño "ding" y aparece una frase motivacional.

Es una sola página, sin dependencias ni build: todo vive en `index.html`.

## Qué hace

- Cuenta atrás con el número de días que quedan, barra de progreso y total de días tachados.
- **Días pasados**: se tachan automáticamente y no se pueden destachar.
- **Hoy y días futuros**: se tachan al tocarlos y se pueden destachar. Las marcas se guardan al instante.
- **Sonido sintetizado** con Web Audio (sin archivos): un ding al tachar, un arpegio en los hitos y un pequeño fanfarre al completar el calendario. El botón 🔔/🔇 silencia y recuerda la preferencia.
- **Frases motivacionales** en un toast, sin repetir la misma dos veces seguidas.
- **Hitos** al llegar a 5, 10, 22, 32 y 40 días tachados, con sonido y mensaje especial.
- **Cambio de día sin recargar**: si la página está abierta a medianoche, el nuevo día se tacha solo y avisa sin sonido.
- **"Borrar mis marcas"** para empezar de cero (los días pasados siguen tachados).
- Diseño mobile-first, navegable con teclado, con `prefers-reduced-motion` respetado.

## Uso en local

Abre `index.html` con doble clic. No hace falta servidor ni instalar nada. Funciona también sin conexión.

## Desplegar en Vercel

### Opción A: desde GitHub (recomendada)

1. Sube este repositorio a GitHub.
2. En [vercel.com](https://vercel.com), **Add New → Project** e importa el repo.
3. Framework preset: **Other**. Sin build command, sin output directory. Vercel sirve `index.html` de la raíz.
4. Deploy. Cada push a la rama principal vuelve a desplegar.

### Opción B: desde la terminal

```bash
npx vercel        # despliegue de prueba
npx vercel --prod # despliegue de producción
```

La primera vez pide iniciar sesión y confirmar el proyecto. Acepta los valores por defecto: es un sitio estático.

En el móvil, abre el enlace y usa **"Añadir a pantalla de inicio"** para tenerlo como una app.

## Personalizar

### Fechas

Al principio del `<script>` principal:

```js
const START_ISO = '2026-10-21';
const END_ISO   = '2026-12-02';
```

Formato `YYYY-MM-DD`. La cuenta atrás se calcula siempre contra `END_ISO` usando la fecha real del dispositivo.

> Si cambias el rango de fechas, revisa las claves de `hitos` en el JSON (5, 10, 22, 32 y 40): están pensadas para un calendario de 43 días.

### Frases

Dentro de `index.html` hay un bloque JSON editable:

```html
<script type="application/json" id="frases">
{
  "general": ["..."],
  "hitos": { "5": "...", "10": "...", "22": "...", "32": "...", "40": "..." },
  "ultimoDia": ["..."],
  "deshecho": ["..."],
  "nuevoDia": ["..."]
}
</script>
```

| Categoría | Cuándo se usa |
|---|---|
| `general` | Al tachar un día normal. |
| `hitos` | Al alcanzar ese número de días tachados. La clave es el número, como texto. |
| `ultimoDia` | Al completar el calendario (o llegar al último día). |
| `deshecho` | Al destachar un día. Sin sonido. |
| `nuevoDia` | Al cruzar la medianoche con la página abierta. Sin sonido. |

Reglas para editarlo:

- Es JSON estricto: comillas dobles y **sin comas finales**.
- No escribas la secuencia `</script>` dentro de una frase.
- Si el JSON queda roto, la app no se rompe: usa unas frases de reserva y avisa en la consola.
- Un array vacío también cae en las frases de reserva.

### Sonido

Los sonidos están generados con osciladores en `playTick()`, `playMilestone()` y `playFinish()`. Si quieres cambiarlos, toca las frecuencias y duraciones de esas funciones. El volumen está bajo a propósito.

Los navegadores (sobre todo Safari en iOS) bloquean el audio hasta el primer toque: por eso el contexto se desbloquea con el primer clic o tecla.

### Colores y estilo

Las variables CSS están al principio del archivo:

```css
:root {
  --accent: #8b1e31;
  --accent-2: #b23b48;
  --accent-soft: #f8e2e5;
  --ink: #2e2225;
  --muted: #7f6a6e;
}
```

La paleta base es granate (rojo profundo). Al completar el calendario, el tema pasa a verde automáticamente (`body.is-done`).

## Dónde se guardan los datos

En `localStorage`, bajo la clave `finalboss.calendar.v1`:

```json
{
  "version": 1,
  "crossed": ["2026-10-25", "2026-11-02"],
  "sonido": true
}
```

- Son datos **por navegador y dispositivo**: si abre el enlace en otro móvil, empieza sin marcas.
- Borrar los datos del navegador o usar una ventana privada con almacenamiento bloqueado elimina o impide guardar las marcas. En ese caso la app lo avisa y sigue funcionando en memoria.
- El botón "Borrar mis marcas" solo vacía `crossed`; conserva la preferencia de sonido.

## Notas técnicas

- Sin frameworks, sin bundler, sin peticiones de red. `index.html` es autosuficiente.
- Fechas tratadas como locales a medianoche y claves `YYYY-MM-DD`, para evitar desfases de un día por UTC.
- Los días se generan a partir de las dos constantes de fecha, con un tope de seguridad de 400 días.
- El estado se escribe en cada cambio, no al cerrar la pestaña.
- Tachar un día actualiza solo esa celda y la cabecera: el calendario no se reconstruye ni parpadea.
- Si el día cambia con la pestaña abierta, se detecta por intervalo, `visibilitychange` y `focus`.

## Comprobado

- Calendario de 43 días (21-oct → 2-dic), con 42 días restantes el 21-oct.
- Días pasados bloqueados y tachados; hoy y días futuros tachables (hoy empieza sin tachar).
- Tachar y destachar hoy y un día futuro, con persistencia tras recargar.
- Frase y toast al tachar; mensaje neutro y silencio al destachar.
- Hito a los 5 días con su frase y estilo propio.
- Silenciar/activar sonido y que la preferencia se guarde.
- Reinicio de marcas conservando los días pasados.
- Estados límite: último día, servicio terminado y calendario aún sin empezar.
- Fallback con el JSON de frases roto (la app sigue funcionando).
- Sin desbordamiento horizontal a 390 px de ancho (móvil) y sin errores de JavaScript en consola.
