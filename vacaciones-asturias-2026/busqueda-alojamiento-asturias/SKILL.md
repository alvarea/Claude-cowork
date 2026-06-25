---
name: busqueda-alojamiento-asturias
description: Búsqueda diaria de alojamiento en Asturias para agosto 2026
---

Eres un agente de búsqueda de alojamiento vacacional. Tu tarea es ejecutar una búsqueda diaria en Booking.com con los parámetros del usuario, aplicar filtros de calidad y proximidad a autopista, y enviar un informe HTML completo por correo Gmail.

## PASO 0 — Alojamientos ya reservados (SIEMPRE PRESENTES EN EL INFORME)

Este paso NO es opcional. Independientemente de lo que encuentres en Booking, el correo final SIEMPRE debe incluir **las dos tarjetas** de los alojamientos ya reservados. Son datos fijos — no necesitas buscarlos en ningún sitio.

### Reserva 1 — Airbnb (noches 1–6: 3–9 agosto)

```
Nombre:         Casa rústica L'alloru en aldea de Llanes
Plataforma:     Airbnb
URL:            https://www.airbnb.es/rooms/1700803072233117740
Ubicación:      Vibaño, Asturias (zona Llanes — Costa Oriental)
Capacidad:      6 viajeros · 3 dormitorios · 6 camas · 2,5 baños
Precio:         1.950 € / 6 noches (3–9 agosto 2026) — YA RESERVADA ✅
Comodidades:    Cocina ✅ · Wifi ✅ · Parking gratuito ✅ · TV · Lavadora
Anfitriona:     Belén — Superanfitriona · 9 años experiencia · 4,57/5 (241 reseñas)
Reseñas:        Anuncio nuevo (sin reseñas propias aún)
Registro:       VV-3953-AS
Cancelación:    ⚠️ Fecha límite cancelación gratuita: 02/08/2026
Notas:          ⚠️ Sin detector de CO ni de humo indicados. Cámaras en exteriores.
```

### Reserva 2 — Booking.com (noches 1–6: 3–9 agosto)

```
Nombre:         Apartamentos Rurales El Solsuco (La Galería)
Plataforma:     Booking.com
URL:            https://www.booking.com/hotel/es/apartamentos-rurales-el-solsuco-la-galeria.html?aid=8132308&checkin=2026-08-03&checkout=2026-08-09&no_rooms=1&group_adults=6&selected_currency=EUR
Ubicación:      Sampedrin s/n Rozaes, Villaviciosa, Asturias (Zona Preferida ⭐)
Capacidad:      6 viajeros
Precio:         1.920 € / 6 noches (3–9 agosto 2026) — YA RESERVADA ✅
Comodidades:    Parking ✅ · Wifi ✅ · Jardín ✅ · Terraza ✅ · BBQ ✅ · Playa cercana
Valoración:     Sin puntuación (anuncio nuevo)
Cancelación:    ⚠️ Fecha límite cancelación gratuita: 19/07/2026
Notas:          Zona preferida del grupo · 8,7 km de la A-8
```

Las dos tarjetas se posicionan en el correo **justo después de la cabecera** y **antes de cualquier resultado de Booking**. Diseño para ambas: borde azul-índigo (#2c3e7a), etiqueta "✅ YA RESERVADA", con la plataforma indicada en el encabezado de la tarjeta. Botón "Ver en Airbnb →" (Reserva 1) y "Ver en Booking →" (Reserva 2), ambos con bgcolor="#2c3e7a".

La fecha límite de cancelación debe destacarse visualmente: si está dentro de los próximos 7 días, muéstrala en rojo; si está entre 8–14 días, en naranja; si es más lejana, en verde. Si el campo dice "[COMPLETAR]", muéstralo en gris.

Debajo de las dos tarjetas, en texto gris: "Los resultados de Booking a continuación se muestran como posibles alternativas para ampliar el viaje o como referencia de precios."

## PASO 1 — Leer parámetros desde Google Drive

Busca en Google Drive el fichero llamado "Parametros Alojamiento.yaml" usando la herramienta de búsqueda de Drive. Lee su contenido y extrae todos los parámetros.

Parámetros clave a extraer:
- destino.comunidad_autonoma y destino.pais
- destino.ciudad — lista de ciudades o zonas preferidas (puede aparecer como string con comas o como lista YAML). Normaliza siempre a una lista Python de strings en minúsculas sin comillas extra.
- destino.radio_km — radio en km alrededor de las ciudades preferidas para marcarlas como zona preferente (default 20 si no está definido)
- fechas.checkin y fechas.checkout (calcula el número de noches = checkout - checkin en días)
- grupo.adultos
- alojamiento.habitaciones
- presupuesto.maximo_total y presupuesto.maximo_noche (si maximo_noche es null, calcula: maximo_noche = maximo_total / número_de_noches)
- calidad.puntuacion_minima
- autovia.activar_filtro, autovia.radio_max_km, autovia.A8_cantabrico (lista de waypoints [lat,lon]), autovia.A66_plata (lista de waypoints [lat,lon])
- correo.destinatarios, correo.asunto
- notas (texto libre con preferencias)


## PASO 2 — Buscar en Booking.com (10 búsquedas por zonas)

La costa asturiana se divide en **5 zonas geográficas** para garantizar cobertura completa. Una búsqueda genérica por "Asturias, Spain" solo devuelve ~10 resultados y pierde propiedades relevantes en zonas específicas. Con 5 zonas obtenemos hasta ~50 candidatos antes de filtrar.

Las 5 zonas cubren el eje A-8 de oeste a este:

| Zona | Destination para la API | Ciudades que cubre |
|------|------------------------|-------------------|
| 1 — Costa Occidental | "Luarca, Asturias, Spain" | Ribadeo, Tapia, Navia, Luarca |
| 2 — Centro-Oeste | "Cudillero, Asturias, Spain" | Cudillero, Soto del Barco, Muros de Nalón |
| 3 — Avilés/Gijón | "Gijón, Asturias, Spain" | Avilés, Gijón, Luanco, Candás |
| 4 — Zona Preferida ⭐ | "Colunga, Asturias, Spain" | Villaviciosa, Caravia, Colunga, Lastres, Ribadesella |
| 5 — Costa Oriental | "Llanes, Asturias, Spain" | Ribadesella, Llanes, Cue, La Franca |

Para cada zona ejecuta **dos llamadas** a accommodations_search: una de tipo A (alojamiento completo) y una de tipo B (hotel/pensión). Total: 10 llamadas en paralelo.

### Tipo A — Alojamientos completos (apartamentos, villas, casas)

Toda la propiedad se reserva como una sola unidad → number_of_rooms = 1.

Parámetros comunes para todas las zonas de tipo A:
- checkin_date y checkout_date del YAML
- number_of_adults: grupo.adultos
- number_of_rooms: 1
- accommodation_types: ["APARTMENT", "VILLA", "HOLIDAY_HOME"]
- price: {"maximum": maximo_noche calculado}
- minimum_review_score: puntuacion_minima como integer
- image_themes: ["SEA_VIEW", "MOUNTAIN_VIEW", "KITCHEN_OR_KITCHENETTE", "NATURAL_LANDSCAPE"]
- currency: "EUR"
- user_locale: "es-es"
- user_country_code: "es"
- user_query: "complete vacation apartment, villa or holiday home in [zona] for [N] adults, [fechas], with kitchen, quiet area, preferably near [ciudades de la zona], good cleanliness, [preferencias de notas]"

### Tipo B — Hoteles y pensiones (habitaciones separadas)

El precio máximo por habitación = round(maximo_noche / alojamiento.habitaciones).

Parámetros comunes para todas las zonas de tipo B:
- checkin_date y checkout_date del YAML
- number_of_adults: grupo.adultos
- number_of_rooms: alojamiento.habitaciones
- accommodation_types: ["HOTEL", "GUEST_HOUSE"]
- price: {"maximum": round(maximo_noche / alojamiento.habitaciones)}
- minimum_review_score: puntuacion_minima como integer
- image_themes: ["SEA_VIEW", "MOUNTAIN_VIEW", "KITCHEN_OR_KITCHENETTE", "NATURAL_LANDSCAPE"]
- currency: "EUR"
- user_locale: "es-es"
- user_country_code: "es"
- user_query: "hotel or guest house in [zona] for [N] adults in [N] rooms, [fechas], preferably near [ciudades de la zona], quiet area, [preferencias de notas]"

### Combinar y deduplicar resultados

Une los resultados de las 10 búsquedas en una sola lista. Para deduplicar:
1. Primero por id numérico del alojamiento (si dos resultados tienen el mismo id, conserva uno solo).
2. Si el id no está disponible, deduplica por nombre exacto.

Al conservar un duplicado, prioriza la entrada con más información (rating, facilities, etc.). Etiqueta cada resultado con su tipo: "🏠 Alojamiento completo" o "🏨 Hotel".

## PASO 3 — Filtro de proximidad a autopista

Si autovia.activar_filtro es true, filtra cada resultado usando sus coordenadas (location.coordinates.latitude, location.coordinates.longitude).

Calcula la distancia mínima de cada alojamiento a cualquier waypoint de A8_cantabrico y A66_plata usando Haversine:

```python
import math
def dist_km(lat1, lon1, lat2, lon2):
    R = 6371
    dlat = math.radians(lat2 - lat1)
    dlon = math.radians(lon2 - lon1)
    a = math.sin(dlat/2)**2 + math.cos(math.radians(lat1)) * math.cos(math.radians(lat2)) * math.sin(dlon/2)**2
    return R * 2 * math.asin(math.sqrt(a))
```

Descarta los alojamientos cuya distancia mínima supere autovia.radio_max_km.

## PASO 3b — Marcar zonas preferidas

**Este paso NO descarta ningún resultado.** Únicamente añade un badge informativo a los alojamientos cercanos a las ciudades preferidas del YAML.

Para cada alojamiento que haya pasado el filtro de autopista (o todos, si el filtro está desactivado):

1. Usa esta tabla de coordenadas aproximadas de ciudades asturianas frecuentes:
```python
COORDS_ASTURIAS = {
    "villaviciosa": (43.484, -5.430), "caravia": (43.462, -5.330),
    "colunga": (43.474, -5.267), "ribadesella": (43.463, -5.063),
    "llanes": (43.421, -4.757), "gijon": (43.537, -5.680),
    "aviles": (43.555, -5.925), "oviedo": (43.362, -5.850),
    "cudillero": (43.562, -6.148), "luarca": (43.540, -6.528),
    "cangas de onis": (43.351, -5.129), "arriondas": (43.389, -5.183),
    "navia": (43.547, -6.726), "tapia de casariego": (43.571, -6.934),
    "lastres": (43.514, -5.282), "luanco": (43.613, -5.790),
    "candas": (43.588, -5.778), "pola de siero": (43.394, -5.655),
    "norena": (43.416, -5.706), "grado": (43.388, -6.076),
    "pravia": (43.494, -6.102),
}
```

2. Para cada ciudad en destino.ciudad (lista extraída del YAML):
   - Normaliza a minúsculas y sin acentos para buscar en la tabla
   - Si está en la tabla, obtén sus coordenadas; si no, omite esa ciudad para el cálculo

3. Calcula la distancia mínima del alojamiento a cualquiera de las ciudades preferidas que tengan coordenadas.

4. Además, comprueba por nombre: si location.city_name del resultado contiene alguna ciudad preferida (comparación case-insensitive, sin acentos), cuenta como zona preferida independientemente de la distancia.

5. Marca zona_preferida = True si: distancia mínima ≤ destino.radio_km O coincide por nombre de ciudad.

6. Los alojamientos con zona_preferida = False se muestran igualmente en el informe, simplemente sin el badge.

## PASO 4 — Detectar novedades respecto al día anterior

Busca en Google Drive el informe del día anterior (fichero con nombre "resultados_AAAA-MM-DD.md" de la fecha de ayer). Si existe, extrae los nombres de los alojamientos que aparecían.

Para cada resultado de hoy, añade una etiqueta:
- 🆕 Nuevo — si el alojamiento NO aparecía en el informe de ayer
- ↩️ Ya conocido — si ya aparecía ayer

Si no existe informe de ayer, todos los resultados se marcan como 🆕 Nuevo.

## PASO 5 — Ordenar resultados

Ordena todos los resultados con el siguiente criterio:
1. Primero los que tienen zona_preferida = True, ordenados por review_score descendente
2. Después los que tienen zona_preferida = False, ordenados por review_score descendente

Identifica los 2-3 mejores como "Destacados" priorizando en este orden: zona_preferida, 🆕 Nuevo, puntuación alta, precio bajo.

## PASO 6 — Generar el HTML del correo

Construye el HTML siguiendo esta estructura en orden exacto. No omitas ninguna sección.

### 6.1 — Cabecera

Fondo con gradiente verde, título "Alojamiento [destino] · [mes año]" en texto NEGRO, negrita, font-size 28px, subtítulo con fechas/adultos/presupuesto también en negro. NO usar texto verde ni blanco en la cabecera.

### 6.2 — Tarjetas de alojamientos ya reservados — OBLIGATORIAS, VAN AQUÍ SIEMPRE

Inserta aquí las **dos tarjetas** de alojamientos ya reservados usando los datos del PASO 0. Primero Reserva 1 (Airbnb · L'alloru · Vibaño), luego Reserva 2 (Booking · El Solsuco · Villaviciosa). Ambas con borde azul-índigo (#2c3e7a) y etiqueta "✅ YA RESERVADA". La fecha límite de cancelación con color según urgencia (rojo/naranja/verde/gris según PASO 0).

### 6.3 — Nota de presupuesto

Banner amarillo si ningún resultado de Booking está dentro del presupuesto máximo.

### 6.4 — Destacados del día

Los 2-3 mejores alojamientos de Booking con tarjeta expandida: nombre, ubicación, puntuación, precio total, badges (🆕/↩️, tipo 🏠/🏨, AC✅/❌, VE✅/❌), botón "Ver en Booking".

### 6.5 — Lista completa zona preferida

Tabla con todos los alojamientos zona_preferida = True, ordenados por puntuación.

### 6.6 — Lista completa otros resultados

Separador visual + tabla con alojamientos zona_preferida = False.

### 6.7 — Footer

Fecha del informe, fuente Booking.com, nota sobre precios variables.

---

**Regla fondos de color Gmail:** Gmail ignora background-color en CSS inline en div/span. Cualquier elemento con fondo de color y texto blanco DEBE implementarse con table/td bgcolor="...". Patrón para botones:

```html
<table cellpadding="0" cellspacing="0" border="0" style="margin-top:14px;">
  <tr>
    <td bgcolor="#1a6b3c" style="background-color:#1a6b3c;border-radius:6px;mso-padding-alt:9px 20px;">
      <a href="[URL]" target="_blank" style="display:inline-block;color:#ffffff;text-decoration:none;font-size:13px;font-weight:bold;padding:9px 20px;border-radius:6px;font-family:Arial,sans-serif;">Ver en Booking &#8594;</a>
    </td>
  </tr>
</table>
```

Diseño: fondo gris claro, tarjetas blancas con border-radius, verde (#1a6b3c) para Booking, azul-índigo (#2c3e7a) para la tarjeta Airbnb, max-width 620px.

## PASO 7 — Checklist de verificación ANTES de llamar a create_draft

Antes de enviar, confirma que el HTML cumple todos estos puntos:

- [ ] Tarjeta 1 "Casa rústica L'alloru · Vibaño" (Airbnb) aparece DESPUÉS de la cabecera y ANTES de los resultados de Booking
- [ ] Tarjeta 1 tiene el botón "Ver en Airbnb →" con bgcolor="#2c3e7a"
- [ ] Tarjeta 2 "Apartamentos Rurales El Solsuco · Villaviciosa" (Booking) aparece INMEDIATAMENTE después de la Tarjeta 1
- [ ] Tarjeta 2 tiene el botón "Ver en Booking →" con bgcolor="#2c3e7a"
- [ ] Ambas tarjetas muestran la etiqueta "✅ YA RESERVADA"
- [ ] Ambas tarjetas muestran el campo de fecha de cancelación (con color según urgencia)
- [ ] Los resultados de Booking vienen DESPUÉS de las dos tarjetas de reservas
- [ ] Footer presente

Si alguno no se cumple, regenera esa sección antes de continuar.

## PASO 8 — Crear borrador Gmail

Usa create_draft con:
- to: lista completa de correo.destinatarios del YAML
- subject: correo.asunto + " · " + fecha de hoy (DD/MM)
- htmlBody: el HTML generado
- body: resumen en texto plano con los 3 destacados + mención de la casa Airbnb ya reservada

## PASO 9 — Guardar informe en Google Drive

Genera el informe en Markdown con la misma información (columnas 🆕/↩️ y tipo 🏠/🏨) y guárdalo en Google Drive con nombre "resultados_AAAA-MM-DD.md" (fecha de hoy) en la misma carpeta que el fichero de parámetros.

## NOTAS GENERALES
- Envía el correo SIEMPRE, independientemente de si hay novedades.
- Si Booking devuelve 0 resultados, envía igualmente el correo con la tarjeta Airbnb + aviso de sin alternativas hoy.
- No uses Expedia (error de geofencing desde España).
- El precio en los resultados de Booking es el precio TOTAL de la estancia.
- Las notas del YAML son preferencias cualitativas, no filtros duros.
