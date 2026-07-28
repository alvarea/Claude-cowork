# Skills — Vacaciones Asturias 2026

Skills de Cowork usados en este proyecto.

## busqueda-alojamiento-asturias

Agente de búsqueda diaria de alojamiento en Asturias para agosto 2026.

- Busca en 5 zonas de la costa asturiana via Booking.com (10 llamadas paralelas)
- Filtra por proximidad a la A-8 (radio 20 km)
- Marca zona preferida: Villaviciosa, Caravia, Colunga
- Detecta novedades respecto al día anterior
- Incluye tarjeta fija con la casa ya reservada en Airbnb (Casa rústica L'alloru, Vibaño)
- Envía borrador HTML por Gmail y guarda informe Markdown en Google Drive

### Instalación

Abre el fichero `.skill` desde la app Cowork → "Save skill".

### Parámetros

Edita `Parametros Alojamiento.yaml` en esta misma carpeta para ajustar fechas, presupuesto, zonas preferidas, etc.

---

## plan-excursiones

Genera la web HTML del plan de viaje a Asturias 2026 (`Plan_Viaje_Asturias_2026.html`).

- Produce un HTML interactivo con 7 tabs CSS-only (sin JavaScript — compatible con Quick Look de iCloud)
- Tab **Semana**: resumen de los 5 días con badges de estado (Cerrado / Por cerrar) y rutas Sevilla ↔ Asturias
- Tabs **Mar 4 / Mié 5 / Jue 6 / Vie 7 / Sáb 8**: actividades, km, plan paso a paso y botón de ruta Google Maps
- Tab **Reservas**: 3 reservas confirmadas (VivePicos 4x4 · La Galana · ALSA Lagos) + pendientes
- Rutas Google Maps verificadas para cada día, con coordenadas exactas de la casa base
- Lee `Parametros_Excursiones.yaml` para verificar cambios en la casa base y asignaciones de días
- NO genera PDF — solo HTML

### Instalación

Invoca el skill desde Cowork con `/plan-excursiones` o selecciónalo en el listado de skills.

### Parámetros

Edita `Parametros_Excursiones.yaml` para:
- `base` — datos de la casa (nombre, dirección, check-in/out)
- `dias_excursion` — asignaciones de ruta por día (`asignado` + `bloqueado: true` para fijar)
- Las rutas Google Maps y el contenido detallado de cada día están embebidos en el SKILL.md

### Mantenimiento

Cuando cambies algo en la web del plan (sesión interactiva), actualiza también el template HTML en `plan-excursiones/SKILL.md` para que el skill regenere correctamente en futuras ejecuciones.
