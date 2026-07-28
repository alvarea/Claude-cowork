---
name: plan-excursiones
description: Genera la web HTML del plan de viaje a Asturias 2026 (Plan_Viaje_Asturias_2026.html) a partir de Parametros_Excursiones.yaml. Produce un HTML interactivo con tabs CSS-only, reservas confirmadas, rutas Google Maps por día y badges de estado. NO genera PDF.
---

Eres el generador de la web de plan de viaje de Asturias 2026. Tu única tarea es escribir el fichero `Plan_Viaje_Asturias_2026.html` en la carpeta del proyecto usando el template exacto de este skill.

---

## PASO 1 — Leer el YAML (opcional, para actualizaciones)

Lee `Parametros_Excursiones.yaml` del proyecto para verificar si hay cambios en:
- `base.nombre` y `base.direccion` — datos de la casa
- `dias_excursion` — días y asignaciones
- Estado de reservas (si hay campos `confirmado` añadidos)

Si el YAML no ha cambiado respecto a la versión de referencia de este skill, usa directamente el template embebido.

---

## PASO 2 — Escribir el HTML

Usa el Write tool para guardar el siguiente HTML COMPLETO como `Plan_Viaje_Asturias_2026.html` en la carpeta del proyecto (iCloud). No modifiques el contenido salvo que el usuario indique cambios explícitos.

### Características críticas a preservar:
- **Tabs CSS-only** (radio buttons, sin JavaScript) — imprescindible para Quick Look de iCloud
- **7 tabs**: Semana, Mar 4, Mié 5, Jue 6, Vie 7, Sáb 8, Reservas
- **Variables CSS** para theming consistente
- **Badges de estado** por día (Cerrado / Por cerrar)
- **Botones de ruta Google Maps** en cada tab de día
- **3 reservas confirmadas** (VivePicos, La Galana, ALSA) en tab Reservas
- **Plan paso a paso** en Mié 5 (aviso ALSA + lista numerada)
- **Avisos de confirmación verdes** en Mié 5, Jue 6 y Vie 7
- **Tarjeta de casa** con links Maps / Eranovum / Booking en tab Semana

### Rutas Google Maps por día (URLs verificadas):
- **Mar 4**: `https://www.google.com/maps/dir/43.4563165,-5.5307515/Mirador+del+Fitu,+Parres,+Asturias/Playa+de+Vega,+Ribadesella,+Asturias/Ribadesella,+Asturias/Bufones+de+Pr%C3%ADa,+Asturias`
- **Mié 5**: `https://www.google.com/maps/dir/43.4563165,-5.5307515/Cangas+de+Onis,+Asturias`
- **Jue 6**: `https://www.google.com/maps/dir/43.4563165,-5.5307515/Plaza+Mayor+10,+Gijon,+Asturias/Cudillero,+Asturias`
- **Vie 7**: `https://www.google.com/maps/dir/43.4563165,-5.5307515/La+Casona+de+Palmira,+Ca%C3%ADn,+Le%C3%B3n`
- **Sáb 8**: `https://www.google.com/maps/dir/43.4563165,-5.5307515/Niembro,+Llanes,+Asturias/Playa+de+Gulpiyuri,+Nueva,+Llanes/Llanes,+Asturias`

---

## TEMPLATE HTML COMPLETO

Escribe exactamente este HTML sin modificaciones:

```html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>Plan Asturias 2026</title>
<style>
  *{margin:0;padding:0;box-sizing:border-box}
  :root{
    --azul:#1a5276;--azul-c:#eaf2fb;
    --verde:#1e8449;--verde-c:#d5f5e3;
    --rojo:#c0392b;--rojo-c:#fde8e8;
    --naranja:#e67e22;--naranja-c:#fef5ec;
    --morado:#6c3483;--morado-c:#f4ecf7;
    --gris:#f5f5f7;--borde:#e0e0e0;
    --txt:#1c1c1e;--txt2:#636366;
  }
  body{font-family:-apple-system,'Helvetica Neue',Arial,sans-serif;background:var(--gris);color:var(--txt);font-size:14px;padding-bottom:30px}

  /* HEADER */
  .hdr{background:var(--azul);color:white;padding:20px 16px 16px}
  .hdr h1{font-size:19px;font-weight:800}
  .hdr p{font-size:11px;opacity:.75;margin-top:3px}
  .hdr-pills{display:flex;flex-wrap:wrap;gap:6px;margin-top:10px}
  .hp{background:rgba(255,255,255,.18);color:white;font-size:11px;font-weight:700;border-radius:12px;padding:4px 10px}

  /* CSS-ONLY TABS */
  input[type="radio"][name="tab"]{display:none}
  .tabs{display:flex;overflow-x:auto;background:white;border-bottom:2px solid var(--borde);position:sticky;top:0;z-index:50;scrollbar-width:none}
  .tabs::-webkit-scrollbar{display:none}
  .tabs label{flex-shrink:0;padding:11px 13px;font-size:11px;font-weight:700;color:var(--txt2);border-bottom:3px solid transparent;cursor:pointer;white-space:nowrap;-webkit-tap-highlight-color:transparent;user-select:none;margin-bottom:-2px}

  #r-resumen:checked  ~ .tabs label[for="r-resumen"],
  #r-mar:checked      ~ .tabs label[for="r-mar"],
  #r-mie:checked      ~ .tabs label[for="r-mie"],
  #r-jue:checked      ~ .tabs label[for="r-jue"],
  #r-vie:checked      ~ .tabs label[for="r-vie"],
  #r-sab:checked      ~ .tabs label[for="r-sab"],
  #r-reservas:checked ~ .tabs label[for="r-reservas"]{color:var(--azul);border-bottom-color:var(--azul)}

  .sec{display:none;padding:14px}
  #r-resumen:checked  ~ #s-resumen  {display:block}
  #r-mar:checked      ~ #s-mar      {display:block}
  #r-mie:checked      ~ #s-mie      {display:block}
  #r-jue:checked      ~ #s-jue      {display:block}
  #r-vie:checked      ~ #s-vie      {display:block}
  #r-sab:checked      ~ #s-sab      {display:block}
  #r-reservas:checked ~ #s-reservas {display:block}

  /* CARDS */
  .card{background:white;border-radius:12px;margin-bottom:10px;padding:12px 14px;box-shadow:0 1px 3px rgba(0,0,0,.07)}
  .card-azul{border-left:4px solid var(--azul)}
  .card-verde{border-left:4px solid var(--verde)}
  .card-naranja{border-left:4px solid var(--naranja)}
  .card-morado{border-left:4px solid var(--morado)}
  .card-rojo{border-left:4px solid var(--rojo)}

  /* RESUMEN */
  .dia-row{background:white;border-radius:10px;margin-bottom:8px;overflow:hidden;box-shadow:0 1px 3px rgba(0,0,0,.06)}
  .dia-row-hdr{display:flex;align-items:center;gap:10px;padding:10px 12px}
  .dia-badge{border-radius:8px;padding:4px 10px;font-size:11px;font-weight:800;color:white;flex-shrink:0}
  .bg-local  {background:#1e8449}
  .bg-montana{background:#6c3483}
  .bg-ciudad {background:#e67e22}
  .bg-costa  {background:#1a5276}
  .dia-nombre{font-size:13px;font-weight:700;flex:1}
  .dia-km{font-size:12px;font-weight:800;color:var(--azul);flex-shrink:0}
  .dia-detalle{font-size:11px;color:var(--txt2);padding:0 12px 10px 12px;line-height:1.5}

  /* ACTIVIDADES */
  .act-title{font-size:11px;font-weight:800;color:var(--azul);text-transform:uppercase;letter-spacing:.5px;margin:0 0 8px}
  .act-item{display:flex;gap:10px;padding:8px 0;border-bottom:1px solid var(--gris)}
  .act-item:last-child{border-bottom:none}
  .act-icon{font-size:16px;flex-shrink:0;width:24px;text-align:center}
  .act-body{}
  .act-nombre{font-size:13px;font-weight:700;color:var(--txt)}
  .act-desc{font-size:11px;color:var(--txt2);margin-top:2px;line-height:1.4}

  /* KM BANNER */
  .km-strip{background:var(--azul-c);border-radius:8px;padding:8px 12px;margin-bottom:10px;display:flex;justify-content:space-between;align-items:center}
  .km-num{font-size:16px;font-weight:800;color:var(--azul)}
  .km-info{font-size:10px;color:var(--txt2);text-align:right;line-height:1.5}
  .km-ruta{font-size:10px;color:var(--txt2);margin-top:4px;line-height:1.5}

  /* COMIDA */
  .comida-row{display:flex;gap:7px;flex-wrap:wrap;margin-top:10px}
  .cpill{font-size:10px;font-weight:700;border-radius:10px;padding:3px 10px}
  .cpill-com{background:#fef9e7;color:#b7770d}
  .cpill-cen{background:var(--azul-c);color:var(--azul)}

  /* AVISO */
  .aviso{background:#fef9e7;border:1px solid #f9d03b;border-radius:8px;padding:8px 10px;font-size:11px;color:#7d6608;margin:8px 0;line-height:1.5}
  .aviso strong{display:block;margin-bottom:2px}

  /* RESERVAS */
  .res-card{border-radius:10px;padding:12px;margin-bottom:10px}
  .res-urgente{background:#fdf2f2;border:2px solid var(--rojo)}
  .res-recom{background:#fffbeb;border:2px solid #f59e0b}
  .res-info{background:var(--verde-c);border:2px solid var(--verde)}
  .res-confirmado{background:#d5f5e3;border:2px solid var(--verde)}
  .tag-c{background:var(--verde);color:white}
  .res-link{display:inline-block;font-size:11px;color:var(--azul);font-weight:700;text-decoration:none;background:var(--azul-c);border-radius:6px;padding:3px 9px;margin-top:5px;margin-right:4px}
  .res-tag{font-size:10px;font-weight:800;border-radius:6px;padding:2px 8px;display:inline-block;margin-bottom:6px}
  .tag-r{background:var(--rojo);color:white}
  .tag-y{background:#f59e0b;color:white}
  .tag-g{background:var(--verde);color:white}
  .res-nombre{font-size:13px;font-weight:700;color:var(--txt);margin-bottom:3px}
  .res-desc{font-size:11px;color:var(--txt2);line-height:1.4;margin-bottom:4px}
  .res-web{font-size:11px;color:var(--azul);font-weight:700}
  .res-dia{font-size:10px;color:var(--txt2);margin-top:2px}

  /* CHECKLIST */
  .check-item{display:flex;align-items:flex-start;gap:10px;padding:8px 0;border-bottom:1px solid var(--gris);font-size:12px}
  .check-item:last-child{border-bottom:none}
  .check-box{width:18px;height:18px;border:2px solid var(--borde);border-radius:4px;flex-shrink:0;margin-top:1px}

  /* DÍA HEADER */
  .dia-hdr-full{border-radius:10px;padding:12px 14px;margin-bottom:10px;color:white}
  .dia-hdr-full.local  {background:var(--verde)}
  .dia-hdr-full.montana{background:var(--morado)}
  .dia-hdr-full.ciudad {background:var(--naranja)}
  .dia-hdr-full.costa  {background:var(--azul)}
  .dia-hdr-full h2{font-size:16px;font-weight:800}
  .dia-hdr-full p{font-size:11px;opacity:.85;margin-top:3px}

  /* MAPA LINK */
  .mapa-btn{display:block;background:var(--azul-c);color:var(--azul);font-size:11px;font-weight:700;text-decoration:none;border-radius:8px;padding:8px 12px;text-align:center;margin:8px 0}

  .section-lbl{font-size:10px;font-weight:800;color:var(--txt2);text-transform:uppercase;letter-spacing:.5px;margin:12px 0 6px 2px}
  .nota-imprescindible{background:var(--naranja-c);border:1.5px solid var(--naranja);border-radius:8px;padding:8px 10px;font-size:11px;color:#7d4200;margin-bottom:10px}
  .plan-status{font-size:10px;font-weight:800;border-radius:10px;padding:3px 9px;flex-shrink:0;margin-left:6px}
  .plan-ok{background:#d5f5e3;color:#1a5e35}
  .plan-wip{background:#fef9e7;color:#7d6608;border:1px solid #f9d03b}
</style>
</head>
<body>

<div class="hdr">
  <h1>Asturias 2026 — Plan del Viaje</h1>
  <p>6 adultos · 3–9 agosto · Base: Villaviciosa (El Solsuco)</p>
  <div class="hdr-pills">
    <span class="hp">5 excursiones</span>
    <span class="hp">625 km totales</span>
    <span class="hp">250 km/dia max</span>
    <span class="hp">Salida 09:00h</span>
    <span class="hp">60€/persona comida</span>
  </div>
</div>

<!-- RADIOS -->
<input type="radio" name="tab" id="r-resumen" checked>
<input type="radio" name="tab" id="r-mar">
<input type="radio" name="tab" id="r-mie">
<input type="radio" name="tab" id="r-jue">
<input type="radio" name="tab" id="r-vie">
<input type="radio" name="tab" id="r-sab">
<input type="radio" name="tab" id="r-reservas">

<div class="tabs">
  <label for="r-resumen">📋 Semana</label>
  <label for="r-mar">Mar 4</label>
  <label for="r-mie">Mie 5</label>
  <label for="r-jue">Jue 6</label>
  <label for="r-vie">Vie 7</label>
  <label for="r-sab">Sab 8</label>
  <label for="r-reservas">🟡 Reservas</label>
</div>

<!-- ══ SEMANA ══ -->
<div class="sec" id="s-resumen">

  <div class="section-lbl">Informacion de la casa</div>
  <div class="card card-azul" style="margin-bottom:14px">
    <div style="font-size:13px;font-weight:700;margin-bottom:6px">Apartamentos Rurales El Solsuco</div>
    <div style="font-size:11px;color:var(--txt2);line-height:1.6">
      Sampedrin s/n Rozaes, 33313 Villaviciosa, Asturias<br>
      Check-in: Lun 3 agosto · Check-out: Dom 9 agosto<br>
      <a href="https://www.google.com/maps/place/Apartamentos+Solsuco/@43.4563165,-5.5307515,17z" style="color:var(--azul);font-weight:700">Google Maps</a>
      &nbsp;·&nbsp;
      <a href="https://maps.google.com/?q=Eranovum+41+Aldea+San+Juan+Amandi+Villaviciosa+Asturias" style="color:var(--azul);font-weight:700">Eranovum Amandi</a>
      &nbsp;·&nbsp;
      <a href="https://www.booking.com/hotel/es/apartamentos-rurales-el-solsuco-la-galeria.html?aid=8132308&checkin=2026-08-03&checkout=2026-08-09&no_rooms=1&group_adults=6&selected_currency=EUR" style="color:var(--azul);font-weight:700">Booking.com</a>
    </div>
  </div>

  <div class="section-lbl">Ruta Sevilla a Asturias</div>
  <a class="mapa-btn" href="https://www.google.com/maps/dir/Sevilla,+Spain/43.4563165,-5.5307515" target="_blank">🚗 Ida: Sevilla → El Solsuco (Lun 3 agosto)</a>
  <a class="mapa-btn" href="https://www.google.com/maps/dir/43.4563165,-5.5307515/Sevilla,+Spain" target="_blank">🏠 Vuelta: El Solsuco → Sevilla (Dom 9 agosto)</a>

  <div class="section-lbl">Los 5 dias de excursion</div>

  <div class="dia-row">
    <div class="dia-row-hdr">
      <span class="dia-badge bg-local">Mar 4</span>
      <div class="dia-nombre">Zona Local + Costa Media</div>
      <span class="plan-status plan-wip">Por cerrar</span>
      <div class="dia-km">93 km</div>
    </div>
    <div class="dia-detalle">Mirador del Fitu · Playa de Vega · Ribadesella · Bufones de Pria</div>
  </div>

  <div class="dia-row">
    <div class="dia-row-hdr">
      <span class="dia-badge bg-montana">Mie 5</span>
      <div class="dia-nombre">Covadonga + Lagos + Cangas</div>
      <span class="plan-status plan-ok">Cerrado</span>
      <div class="dia-km">110 km</div>
    </div>
    <div class="dia-detalle">Santuario Covadonga · Lagos Enol y Ercina (bus) · Cangas de Onis · Helados Nacho</div>
  </div>

  <div class="dia-row">
    <div class="dia-row-hdr">
      <span class="dia-badge bg-ciudad">Jue 6</span>
      <div class="dia-nombre">Gijon + Cudillero</div>
      <span class="plan-status plan-ok">Cerrado</span>
      <div class="dia-km">160 km</div>
    </div>
    <div class="dia-detalle">Cimadevilla · Playa San Lorenzo · La Galana (fabada) · Cudillero</div>
  </div>

  <div class="dia-row">
    <div class="dia-row-hdr">
      <span class="dia-badge bg-montana">Vie 7</span>
      <div class="dia-nombre">Ruta del Cares + 4x4 + Naranjo</div>
      <span class="plan-status plan-ok">Cerrado</span>
      <div class="dia-km">160 km</div>
    </div>
    <div class="dia-detalle">Poncebos → Cain (12 km) · 4x4 vuelta · Naranjo desde Sotres · Queso Cabrales</div>
  </div>

  <div class="dia-row">
    <div class="dia-row-hdr">
      <span class="dia-badge bg-costa">Sab 8</span>
      <div class="dia-nombre">Playas de Llanes</div>
      <span class="plan-status plan-wip">Por cerrar</span>
      <div class="dia-km">102 km</div>
    </div>
    <div class="dia-detalle">Cluster Niembro · Gulpiyuri · Cuevas del Mar · Poo · Llanes pueblo</div>
  </div>

  <div class="section-lbl">Km totales de la semana</div>
  <div class="km-strip">
    <div class="km-num">625 km</div>
    <div class="km-info">Limite: 1.250 km<br>Uso: 50%</div>
  </div>

  <div class="nota-imprescindible">
    <strong style="font-size:11px">Lugares imprescindibles no incluidos:</strong>
    Lastres (excluido del plan) · Oviedo (1/3 votos) — ambos a menos de 30 min de casa, posibles visitas de tarde espontaneas.
  </div>

</div>

<!-- ══ MARTES 4 ══ -->
<div class="sec" id="s-mar">
  <div class="dia-hdr-full local">
    <h2>Martes 4 · Zona Local + Costa Media</h2>
    <p>Salida 09:00h · Regreso ~20:00h · 93 km</p>
  </div>
  <div class="km-strip">
    <div class="km-num">93 km</div>
    <div class="km-ruta">Base → Fitu (25) → Vega (22) → Ribadesella (30) → Bufones (40) → Base (40)</div>
  </div>
  <a class="mapa-btn" href="https://www.google.com/maps/dir/43.4563165,-5.5307515/Mirador+del+Fitu,+Parres,+Asturias/Playa+de+Vega,+Ribadesella,+Asturias/Ribadesella,+Asturias/Bufones+de+Pr%C3%ADa,+Asturias" target="_blank">🗺 Ver ruta del dia en Google Maps</a>
  <div class="act-title">Actividades del dia</div>
  <div class="act-item">
    <div class="act-icon">🏔️</div>
    <div class="act-body">
      <div class="act-nombre">Mirador del Fitu</div>
      <div class="act-desc">El mejor mirador de Asturias. Vista 360° de la costa y los Picos. Desviacion de ~7 km desde la carretera principal. Mañana.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🏖️</div>
    <div class="act-body">
      <div class="act-nombre">Playa de Vega</div>
      <div class="act-desc">Monumento Natural. Playa virgen de arena fina sin edificaciones. A 22 km de casa. Bano de media mañana.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">⚓</div>
    <div class="act-body">
      <div class="act-nombre">Ribadesella</div>
      <div class="act-desc">Casco antiguo marinero, ria, puente historico. Ambiente tranquilo. Comida/marisco en puerto.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">💧</div>
    <div class="act-body">
      <div class="act-nombre">Bufones de Pria</div>
      <div class="act-desc">Formaciones kársticas únicas: agujeros en el acantilado que expulsan chorros de agua del mar. Ruta costera ~4 km. Tarde.</div>
    </div>
  </div>
  <div class="aviso">
    <strong>Marea baja opcional:</strong> Playa de La Griega (huellas de dinosaurios) cerca de Colunga — solo visible con marea baja. Consultar tabla la noche anterior.
  </div>
  <div class="comida-row">
    <span class="cpill cpill-com">Comida: Marisco en Ribadesella</span>
    <span class="cpill cpill-cen">Cena: Sidreria La Marina (Ribadesella)</span>
  </div>
</div>

<!-- ══ MIERCOLES 5 ══ -->
<div class="sec" id="s-mie">
  <div class="dia-hdr-full montana">
    <h2>Miercoles 5 · Covadonga + Lagos + Cangas</h2>
    <p>Salida 08:30h · Regreso ~19:30h · 110 km</p>
  </div>
  <div class="km-strip">
    <div class="km-num">110 km</div>
    <div class="km-ruta">Base → Covadonga (55) → Lagos (bus, sin km) → Cangas de Onis (+5) → Base (50)</div>
  </div>
  <a class="mapa-btn" href="https://www.google.com/maps/dir/43.4563165,-5.5307515/Cangas+de+Onis,+Asturias" target="_blank">🗺 Ver ruta del dia en Google Maps</a>
  <div class="act-title">Plan del dia — Paso a paso</div>
  <div class="aviso" style="background:#d5f5e3;border-color:var(--verde);color:#1a5e35">
    <strong>Bus ALSA confirmado · Localizador: 1gh7rvs · Llevar DNI</strong>
    Ida 10:00h Cangas · Vuelta 16:50h Lagos · Billetes nominativos — presentar en movil
  </div>
  <div class="card" style="margin-top:8px">
    <ol style="padding-left:18px;line-height:2.1;font-size:12px;color:var(--txt)">
      <li><strong>09:00h</strong> Salida desde El Solsuco (Villaviciosa)</li>
      <li><strong>09:50h</strong> Llegada Cangas de Onis — aparcar en <strong>P1 Estacion de autobuses</strong> (3€/coche)</li>
      <li><strong>10:00h</strong> Bus ALSA Cangas → Lagos · llegada 10:50h</li>
      <li><strong>~11:00h</strong> Parada en Covadonga — visita <strong>Santuario y Cueva de la Virgen</strong> (~1h)</li>
      <li><strong>~12:00h</strong> Lanzadera gratuita Covadonga → Lagos (incluida en billete, cada 30 min)</li>
      <li><strong>~12:20h</strong> <strong>Paseo circular Lagos Enol y Ercina</strong> ~5 km · bocadillos al aire libre</li>
      <li><strong>16:50h</strong> Bus vuelta Lagos → Cangas · llegada 17:40h</li>
      <li><strong>~18:00h</strong> Paseo <strong>Cangas de Onis</strong> — puente romano s.VIII, casco historico</li>
      <li><strong>~18:30h</strong> <strong>Helados Nacho</strong> — imprescindible</li>
      <li><strong>~19:30h</strong> Vuelta a base</li>
    </ol>
  </div>
  <div class="comida-row">
    <span class="cpill cpill-com">Comida: Bocadillos en los Lagos</span>
    <span class="cpill cpill-cen">Cena: En casa</span>
  </div>
</div>

<!-- ══ JUEVES 6 ══ -->
<div class="sec" id="s-jue">
  <div class="dia-hdr-full ciudad">
    <h2>Jueves 6 · Gijon + Cudillero</h2>
    <p>Salida 09:00h · Regreso ~21:00h · 160 km</p>
  </div>
  <div class="km-strip">
    <div class="km-num">160 km</div>
    <div class="km-ruta">Base → Gijon (35) → Cudillero (35+45=80) → Base (80)</div>
  </div>
  <a class="mapa-btn" href="https://www.google.com/maps/dir/43.4563165,-5.5307515/Plaza+Mayor+10,+Gijon,+Asturias/Cudillero,+Asturias" target="_blank">🗺 Ver ruta del dia en Google Maps</a>
  <div class="act-title">Actividades del dia</div>
  <div class="act-item">
    <div class="act-icon">🏛️</div>
    <div class="act-body">
      <div class="act-nombre">Cimadevilla — Gijon</div>
      <div class="act-desc">Barrio historico sobre el cabo. Puerto deportivo, calles medievales, ambiente animado.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🏖️</div>
    <div class="act-body">
      <div class="act-nombre">Playa de San Lorenzo</div>
      <div class="act-desc">2 km de playa urbana con paseo maritimo. Bano de media mañana antes de comer.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🍺</div>
    <div class="act-body">
      <div class="act-nombre">La Galana — Fabada + Sidra Escanciada</div>
      <div class="act-desc">LA experiencia gastronomica asturiana. Fabada tradicional, sidra desde 1m de altura, oricios. c/ Santa Rosa 4, Cimadevilla. Reservar para 6.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🎨</div>
    <div class="act-body">
      <div class="act-nombre">Cudillero</div>
      <div class="act-desc">El pueblo mas fotogenico de Asturias. Casas de colores en anfiteatro sobre el puerto pesquero. Postal obligatoria. Tarde.</div>
    </div>
  </div>
  <div class="aviso" style="background:#d5f5e3;border-color:var(--verde);color:#1a5e35">
    <strong>RESERVADO — La Galana · Jueves 6 agosto 14:00h · Sala interior · 6 personas</strong>
    Pl. Mayor 10, Cimadevilla · 985 172 429
  </div>
  <div class="comida-row">
    <span class="cpill cpill-com">Comida: La Galana (fabada + sidra) — Gijon</span>
    <span class="cpill cpill-cen">Cena: El Remo (Cudillero) o en casa</span>
  </div>
</div>

<!-- ══ VIERNES 7 ══ -->
<div class="sec" id="s-vie">
  <div class="dia-hdr-full montana">
    <h2>Viernes 7 · Ruta del Cares + 4x4 + Naranjo</h2>
    <p>Salida 07:30h · Regreso ~21:00h · 160 km · Dia mas exigente</p>
  </div>
  <div class="km-strip">
    <div class="km-num">160 km</div>
    <div class="km-ruta">Base → Poncebos (65) → [Cares walk + 4x4] → Sotres (+15) → Arenas Cabrales (+20) → Base (60)</div>
  </div>
  <a class="mapa-btn" href="https://www.google.com/maps/dir/43.4563165,-5.5307515/La+Casona+de+Palmira,+Ca%C3%ADn,+Le%C3%B3n" target="_blank">🗺 Ver ruta del dia en Google Maps</a>
  <div class="act-title">Actividades del dia</div>
  <div class="act-item">
    <div class="act-icon">🥾</div>
    <div class="act-body">
      <div class="act-nombre">Ruta del Cares — Poncebos a Cain</div>
      <div class="act-desc">12 km por el desfiladero mas espectacular de Europa. Sendero excavado en la roca con el rio 400m abajo. ~4h ida. Dificultad media.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🚙</div>
    <div class="act-body">
      <div class="act-nombre">4x4 — Regreso Cain a Poncebos · VivePicos ✓</div>
      <div class="act-desc">Pistas de montana de vertigo en vehiculo 4x4. VivePicos (Landrover blancos). Encuentro 14:50h en Restaurante La Casona de Palmira, Cain. Res. #4442-4288276 · ~4h.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">⛰️</div>
    <div class="act-body">
      <div class="act-nombre">Naranjo de Bulnes — Mirador desde Sotres</div>
      <div class="act-desc">El Picu Urriellu, pico mas iconico de los Picos de Europa. Mirador desde Sotres (no es escalada). Vistas espectaculares.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🧀</div>
    <div class="act-body">
      <div class="act-nombre">Queso Cabrales DOP — Arenas de Cabrales</div>
      <div class="act-desc">El queso azul mas potente de España. En origen, en queseria artesanal. De vuelta hacia casa.</div>
    </div>
  </div>
  <div class="aviso" style="background:#d5f5e3;border-color:var(--verde);color:#1a5e35">
    <strong>RESERVADO — VivePicos · Res. #4442-4288276</strong>
    Encuentro 14:50h · Restaurante La Casona de Palmira, Cain · Landrovers blancos · 6 personas · 150€ pendiente en efectivo el dia · 984 151 207
  </div>
  <div class="comida-row">
    <span class="cpill cpill-com">Comida: Bocadillos en la ruta del Cares</span>
    <span class="cpill cpill-cen">Cena: Sidreria asturiana (Villaviciosa)</span>
  </div>
</div>

<!-- ══ SABADO 8 ══ -->
<div class="sec" id="s-sab">
  <div class="dia-hdr-full costa">
    <h2>Sabado 8 · Playas de Llanes</h2>
    <p>Salida 09:00h · Regreso ~20:30h · 102 km · Dia tranquilo</p>
  </div>
  <div class="km-strip">
    <div class="km-num">102 km</div>
    <div class="km-ruta">Base → Niembro (46) → Gulpiyuri (+2) → Cuevas del Mar (+3) → Poo (+3) → Llanes (+3) → Base (45)</div>
  </div>
  <a class="mapa-btn" href="https://www.google.com/maps/dir/43.4563165,-5.5307515/Niembro,+Llanes,+Asturias/Playa+de+Gulpiyuri,+Nueva,+Llanes/Llanes,+Asturias" target="_blank">🗺 Ver ruta del dia en Google Maps</a>
  <div class="act-title">Actividades del dia</div>
  <div class="act-item">
    <div class="act-icon">🏝️</div>
    <div class="act-body">
      <div class="act-nombre">Cluster Niembro — Torimbia, Borizu, Barro</div>
      <div class="act-desc">Aparcar en Niembro, acceder a pie a la serie de calas salvajes encadenadas. Torimbia es la mas famosa. Mañana entera (~3h).</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🌊</div>
    <div class="act-body">
      <div class="act-nombre">Playa de Gulpiyuri</div>
      <div class="act-desc">Unica en Europa. Playa interior sin acceso directo al mar — el agua llega por conductos subterraneos. Solo 50m de arena. Ir temprano.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🪨</div>
    <div class="act-body">
      <div class="act-nombre">Playa de Cuevas del Mar</div>
      <div class="act-desc">Cala triangular con cuevas excavadas por el mar en los acantilados. Arena blanca, aguas tranquilas.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🦢</div>
    <div class="act-body">
      <div class="act-nombre">Playa de Poo</div>
      <div class="act-desc">Laguna donde el rio se mezcla con el Cantabrico. Aguas muy calmadas. Bano tranquilo despues de las playas abiertas.</div>
    </div>
  </div>
  <div class="act-item">
    <div class="act-icon">🏰</div>
    <div class="act-body">
      <div class="act-nombre">Llanes pueblo</div>
      <div class="act-desc">Casco medieval, muralla, Los Cubos de la Memoria (Agustin Ibarrola). Paseo de tarde.</div>
    </div>
  </div>
  <div class="aviso">
    <strong>Gulpiyuri:</strong> llena muy rapido en agosto. Ir a primera hora de la tarde. Aparcamiento en Niembro para el Cluster.
  </div>
  <div class="comida-row">
    <span class="cpill cpill-com">Comida: Bocadillos en playa Niembro</span>
    <span class="cpill cpill-cen">Cena: En casa (ultimo dia completo)</span>
  </div>
</div>

<!-- ══ RESERVAS ══ -->
<div class="sec" id="s-reservas">

  <div class="res-card res-confirmado">
    <span class="res-tag tag-c">CONFIRMADO — Reserva realizada</span>
    <div class="res-nombre">1. VivePicos 4x4 Ruta Cares — Cain a Poncebos (Viernes 7)</div>
    <div class="res-desc" style="line-height:1.7">
      <strong>Reserva:</strong> #4442-4288276 &nbsp;·&nbsp; <strong>Fecha:</strong> 7 agosto 14:50h<br>
      <strong>Encuentro:</strong> Restaurante La Casona de Palmira, Cain<br>
      <strong>Vehiculo:</strong> Landrover blancos (uso compartido)<br>
      <strong>Participantes:</strong> 6 personas &nbsp;·&nbsp; 6 x 50€ = <strong>300€</strong><br>
      <strong>Pagado (anticipo 50%):</strong> 150€ via tarjeta (****5402)<br>
      <strong>Pendiente:</strong> 150€ en efectivo el dia de la actividad<br>
      <strong>Contacto:</strong> info@vivepicos.com &nbsp;·&nbsp; 984 151 207
    </div>
    <div style="margin-top:6px;font-size:10px;color:#1a5e35;font-weight:700">Recomiendan salir a caminar desde Poncebos antes de las 09:00h (cortan la carretera a partir de esa hora en verano)</div>
    <div style="margin-top:8px">
      <a class="res-link" href="https://www.reservaonline.support/vivepicos/confirmacion.html?tpv_hash=b63b4972ecaba5ee13d8918e7ec5acfdd8db47f2dece8479469b42cbc73f313f&ts=2654333-1785055894">Ver confirmacion</a>
      <a class="res-link" href="https://www.mrplan.es/scr/modulos/TCheckin/TCheckin_Gestion/?lpEnc=d150671c4083aff1ccc5ad57b8371a12-0-1-d16c5176795e6d00b3985e5490c1bdb664645d66f80daad60cc223e3b7313db7">Listado participantes</a>
    </div>
    <div class="res-dia">Dia: Viernes 7 agosto · Actividad: 8 horas totales</div>
  </div>

  <div class="res-card res-confirmado">
    <span class="res-tag tag-c">CONFIRMADO — Reserva realizada</span>
    <div class="res-nombre">2. La Galana — Gijón · Comida Jueves 6</div>
    <div class="res-desc" style="line-height:1.7">
      <strong>Fecha:</strong> Jueves 6 agosto · 14:00h<br>
      <strong>Personas:</strong> 6 · Zona: sala interior<br>
      <strong>Direccion:</strong> Pl. Mayor, 10, Cimadevilla, Gijón<br>
      <strong>Telefono:</strong> 985 172 429
    </div>
    <div style="margin-top:8px">
      <a class="res-link" href="https://restauranteasturianolagalana.es">Web oficial</a>
      <a class="res-link" href="https://carta.menu/restaurants/gijon/la-galana/menu">Ver carta</a>
      <a class="res-link" href="https://www.tripadvisor.com/Restaurant_Review-g187451-d1926012-Reviews-Restaurante_Sidreria_La_Galana-Gijon_Asturias.html">TripAdvisor</a>
    </div>
  </div>

  <div class="res-card res-confirmado">
    <span class="res-tag tag-c">CONFIRMADO — Reserva realizada</span>
    <div class="res-nombre">3. Bus ALSA — Cangas de Onis a Lagos de Covadonga (Miercoles 5)</div>
    <div class="res-desc" style="line-height:1.7">
      <strong>Localizador:</strong> 1gh7rvs<br>
      <strong>Ida:</strong> 10:00h P1 Estacion Cangas → Lagos 10:50h<br>
      <strong>Vuelta:</strong> 16:50h Lagos → Cangas 17:40h<br>
      <strong>Personas:</strong> 6 · 9€/persona (54€ total)<br>
      <strong>Tarifa Flex</strong> — se puede anular y cambiar<br>
      <strong>Importante:</strong> billetes nominativos — presentar en movil con DNI
    </div>
    <div style="margin-top:8px">
      <a class="res-link" href="https://www.icloud.com/iclouddrive/0a4lSL5GdDp0zreSuH8o1oVag#ALSA_Billetes_Cangas_-_Lagos_ticket_05.08">Ver billetes PDF</a>
    </div>
    <div style="margin-top:4px;font-size:10px;color:#1a5e35;font-weight:700">Parking P1 Estacion autobuses Cangas: 3€/coche/dia</div>
  </div>

  <div class="res-card res-recom">
    <span class="res-tag tag-y">RECOMENDADO — Reservar pronto</span>

    <div style="padding-bottom:10px;border-bottom:1px solid #f0d080;margin-bottom:10px">
      <div class="res-nombre">4. Sidreria La Marina — Ribadesella (Martes 4)</div>
      <div class="res-desc">Cena de 6 personas. Sidreria clasica de referencia en la zona oriental asturiana.</div>
      <div class="res-web">Ribadesella — buscar en Google Maps</div>
    </div>

    <div>
      <div class="res-nombre">5. Sidreria asturiana — Villaviciosa (Viernes 7 noche)</div>
      <div class="res-desc">La gran experiencia gastronomica del viaje: sidra escanciada, cachopo, oricios. Cena de vuelta del Cares.</div>
      <div class="res-web">Consultar sidrerias en Villaviciosa</div>
    </div>
  </div>

  <div class="res-card res-info">
    <span class="res-tag tag-g">INFORMATIVO</span>

    <div style="padding-bottom:10px;border-bottom:1px solid #a9dfbf;margin-bottom:10px">
      <div class="res-nombre">Playa de La Griega — huellas dinosaurios</div>
      <div class="res-desc">Solo visibles con marea baja. Cerca de Colunga, paso natural el Martes 4. Consultar tabla de mareas la noche anterior.</div>
    </div>

    <div>
      <div class="res-nombre">Lastres y Oviedo — posibles visitas espontaneas</div>
      <div class="res-desc">Ambos a menos de 30 min de casa. No estan en el plan pero son perfectos para una tarde libre o dia de lluvia (Oviedo — prerromanico UNESCO).</div>
    </div>
  </div>

</div>

</body>
</html>
```

---

## PASO 3 — Verificar

Tras escribir el fichero, confirma que:
- [ ] El fichero tiene 7 secciones de tab (s-resumen, s-mar, s-mie, s-jue, s-vie, s-sab, s-reservas)
- [ ] Cada tab de día tiene un `.mapa-btn` con la URL de Google Maps correcta
- [ ] Las 3 tarjetas confirmadas (VivePicos, La Galana, ALSA) están en s-reservas
- [ ] Los badges de estado son correctos: Mié/Jue/Vie = Cerrado, Mar/Sáb = Por cerrar

## PASO 4 — Sincronizar con GitHub (opcional)

Si el usuario tiene el script `skill_agente_vacaciones_to_github.ssh` en la carpeta del proyecto, recuérdale que puede ejecutarlo desde Terminal para subir los cambios al repositorio GitHub.

---

## NOTAS DE MANTENIMIENTO

Cuando el usuario confirme el plan de Mar 4 o Sáb 8, actualizar en este skill:
- El badge `plan-wip` → `plan-ok` en la tab Semana
- El contenido de la tab correspondiente con el plan detallado (similar al de Mié 5)
- La URL de Google Maps si es necesario actualizar paradas

Reservas pendientes de confirmar (actualizar aquí cuando se hagan):
- Sidreria La Marina Ribadesella — Martes 4 noche (6 personas)
- Sidreria Villaviciosa — Viernes 7 noche (6 personas)
