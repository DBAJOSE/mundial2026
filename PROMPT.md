# Prompt para reconstruir el "Simulador Mundial 2026"

> Copia y pega TODO lo que está debajo de la línea a una IA capaz de generar código.
> El resultado debe ser **un único archivo `index.html`** autónomo (HTML + CSS + JS en línea, sin
> dependencias, sin build, que funcione abriéndolo directo en el navegador y en móvil).

---

## ROL Y OBJETIVO

Eres un desarrollador front-end. Construye **un solo archivo `index.html`** (todo el CSS y el JS embebidos, sin
librerías externas, sin frameworks, sin CDN, en español) que sea un **simulador interactivo del cuadro de
eliminación del Mundial 2026** (formato de 48 selecciones → 32 clasificados a "16avos de final"), desde 16avos
hasta la Final, jugable y bonito, optimizado para abrir como archivo local y para verse bien en celular.

El contexto temporal es: **domingo 28 de junio de 2026, la fase de grupos YA terminó** y los 16avos están
confirmados. (La fase de grupos existió en una versión anterior; debe quedar **comentada y reutilizable**, ver
más abajo.)

## ESTÉTICA GENERAL

- Tema oscuro deportivo. Usa variables CSS en `:root`:
  `--bg:#0a1124; --bg2:#0e1a39; --card:#13224a; --card-hi:#1b3066; --line:#33508f;
   --accent:#22d3a6; --accent2:#37c2ff; --gold:#ffcf3f; --txt:#eaf1ff; --muted:#8fa3cf;`
- Fondo del `body`: dos `radial-gradient` azulados/verdosos superpuestos sobre `--bg`.
- Fuente: `"Segoe UI", system-ui, -apple-system, sans-serif`.
- Encabezado centrado:
  - kicker en mayúsculas: `Copa del Mundo · USA · México · Canadá` (color accent, letter-spacing).
  - `<h1>` "Simulador Mundial 2026" con relleno de texto en degradado (accent → accent2 → gold) usando
    `background-clip:text; color:transparent`.
  - subtítulo: `Domingo 28 de junio · 16avos de final ya definidos · arma tu cuadro hasta la Final`.
- Barra de botones tipo "pill": **⚡ Simular por probabilidad** (primario, degradado accent→accent2),
  **🎲 Azar** y **↺ Reiniciar**.
- Pie de página discreto (ver sección FOOTER).

## MODELO DE PROBABILIDAD (no azar puro)

Cada selección tiene una **fuerza tipo Elo** (según ranking FIFA y forma). Define un mapa `RATING` (nombre→número)
con estos valores exactos:

```
Argentina 2095, Francia 2080, España 2070, Brasil 2040, Portugal 2015, Inglaterra 2010,
Países Bajos 1985, Alemania 1955, Croacia 1895, Marruecos 1900, Colombia 1875, Suiza 1855,
Senegal 1845, Japón 1835, Noruega 1825, Austria 1815, Estados Unidos 1810, México 1800,
Ecuador 1800, Corea del Sur 1785, Irán 1775, Argelia 1770, Suecia 1770, Canadá 1760,
Costa de Marfil 1755, Egipto 1750, Australia 1740, Ghana 1735, Bosnia y H. 1720, RD Congo 1710,
Sudáfrica 1710, Paraguay 1690, Panamá 1675, Uzbekistán 1670, Cabo Verde 1640, Jordania 1610
```

- `rating(n) = RATING[n] || 1700`.
- Probabilidad de que A gane a B (eliminatoria, sin empate):
  `pWin(a,b) = 1 / (1 + 10^((rating(b)-rating(a))/400))`.
- Simulación por probabilidad de un partido KO: `sampleKO(a,b) = Math.random() < pWin(a,b) ? 0 : 1`.
- (Para la fase de grupos reciclable) `matchProbs(a,b)`: con empate. `e=pWin(a,b)`,
  `pd = max(0.08, 0.28 - 0.12*|2e-1|)`, devuelve `{a:e*(1-pd), d:pd, b:(1-e)*(1-pd)}`.
  `sampleGroup` muestrea A/D/B con esas probabilidades.

## DATOS DEL CUADRO (16avos confirmados)

Helper `T(nombre, bandera_emoji, codigo3letras)`. Para Inglaterra usa la bandera con tag-flag:
`const ENG_FLAG = "🏴󠁧󠁢󠁥󠁮󠁧󠁿";`

Define `R32DEF` = arreglo de 16 partidos en **orden de cuadro** (OF1..OF16). Cada partido es `[{t:T(...)},{t:T(...)}]`.
**Llave izquierda OF1–OF8**, **llave derecha OF9–OF16**. Equipos y banderas exactos:

| # | Local | Visitante |
|----|-------|-----------|
| OF1 | Alemania 🇩🇪 ALE | Paraguay 🇵🇾 PAR |
| OF2 | Francia 🇫🇷 FRA | Suecia 🇸🇪 SWE |
| OF3 | Sudáfrica 🇿🇦 RSA | Canadá 🇨🇦 CAN |
| OF4 | Países Bajos 🇳🇱 NED | Marruecos 🇲🇦 MAR |
| OF5 | Portugal 🇵🇹 POR | Croacia 🇭🇷 CRO |
| OF6 | España 🇪🇸 ESP | Austria 🇦🇹 AUT |
| OF7 | Estados Unidos 🇺🇸 USA | Bosnia y H. 🇧🇦 BIH |
| OF8 | Egipto 🇪🇬 EGY | Corea del Sur 🇰🇷 KOR |
| OF9 | Brasil 🇧🇷 BRA | Japón 🇯🇵 JPN |
| OF10 | Costa de Marfil 🇨🇮 CIV | Noruega 🇳🇴 NOR |
| OF11 | México 🇲🇽 MEX | Ecuador 🇪🇨 ECU |
| OF12 | Inglaterra 🏴 ENG | RD Congo 🇨🇩 COD |
| OF13 | Argentina 🇦🇷 ARG | Cabo Verde 🇨🇻 CPV |
| OF14 | Australia 🇦🇺 AUS | Irán 🇮🇷 IRN |
| OF15 | Suiza 🇨🇭 SUI | Argelia 🇩🇿 ALG |
| OF16 | Colombia 🇨🇴 COL | Ghana 🇬🇭 GHA |

`const ROUND_NAMES = ["16avos","Octavos","Cuartos","Semifinal","Final"];`

(Conserva el soporte para cupos dinámicos `{ref:[grupo,posición]}` en `resolveSlot`, aunque hoy todos sean fijos:
si `s.t` existe devuélvelo; si no, devuelve un placeholder `{name:"Por definir", flag:"⚪", code:"—", placeholder:true}`.)

## DATOS DE TRANSMISIÓN (tooltip)

Define `MATCHINFO` = 16 objetos `{d:fecha, h:hora, iso:fechaHoraISO|null, s:sede, tv:canal}` alineados a `R32DEF`
(OF1..OF16). `d`/`h` son texto para mostrar; `iso` es la **fecha-hora real** (formato `"AAAA-MM-DDTHH:MM:SS"`, en
**hora de Colombia, UTC-5**) que usa la agenda en vivo — debe ser `null` cuando la fecha u hora está por confirmar.
Horas en hora de Colombia, calendario provisional; `"—"` = por confirmar. `tv` siempre `"Caracol/RCN · DGO"`.
Valores exactos:

```
OF1  Lun 29 jun  15:30  iso 2026-06-29T15:30:00  Gillette Stadium, Boston
OF2  Mar 30 jun  —      iso null                 MetLife Stadium, Nueva Jersey
OF3  Dom 28 jun  14:00  iso 2026-06-28T14:00:00  SoFi Stadium, Los Ángeles
OF4  Lun 29 jun  20:00  iso 2026-06-29T20:00:00  Estadio BBVA, Monterrey
OF5  Jue 2 jul   18:00  iso 2026-07-02T18:00:00  BMO Field, Toronto
OF6  Mar 30 jun  14:00  iso 2026-06-30T14:00:00  SoFi Stadium, Los Ángeles
OF7  Mié 1 jul   19:00  iso 2026-07-01T19:00:00  Levi's Stadium, San Francisco
OF8  —           —      iso null                 Por confirmar
OF9  Lun 29 jun  12:00  iso 2026-06-29T12:00:00  NRG Stadium, Houston
OF10 Mar 30 jun  —      iso null                 AT&T Stadium, Dallas
OF11 Mié 1 jul   —      iso null                 Estadio Azteca, Ciudad de México
OF12 Mié 1 jul   11:00  iso 2026-07-01T11:00:00  Mercedes-Benz Stadium, Atlanta
OF13 Vie 3 jul   17:00  iso 2026-07-03T17:00:00  Hard Rock Stadium, Miami
OF14 —           —      iso null                 Por confirmar
OF15 Mié 1 jul   22:00  iso 2026-07-01T22:00:00  BC Place, Vancouver
OF16 Vie 3 jul   20:30  iso 2026-07-03T20:30:00  Arrowhead Stadium, Kansas City
```

## ESTADO Y LÓGICA DEL CUADRO

- `rounds`: arreglo de 5 rondas. Ronda 0 = 16 partidos (desde `R32DEF` vía `resolveSlot`), luego 8, 4, 2, 1.
  Cada partido `= {teams:[t0,t1], winner:null}`.
- `initState()`: crea `OUT={J:[null,null],K:[null,null],L:[null,null]}` (sólo lo usa la fase reciclable) y arma
  `rounds`.
- `winnerTeam(r,m)`: equipo ganador o null.
- `propagate()`: para r=1..4, cada partido toma como equipos los ganadores de sus dos partidos hijos
  (`m*2`, `m*2+1`); si el ganador previamente elegido sigue presente, lo conserva.
- `pick(r,m,slot)`: marca ganador del slot, `propagate()`, `render()`.
- Avance interactivo: hacer **clic en una selección** la marca ganadora y propaga hacia la final.

## RENDER DEL BRACKET (¡alineación correcta y obligatoria!)

Disposición en columnas dentro de `.bracket` (flex, `gap:26px`, `min-width:1500px`, `align-items:stretch`):

```
L:16avos(0–7) · L:Octavos(0–3) · L:Cuartos(0–1) · L:Semi(0) · [FINAL] · R:Semi(1) · R:Cuartos(2–3) · R:Octavos(4–7) · R:16avos(8–15)
```

La llave **derecha** lleva clase `.side-r` (fila invertida `row-reverse`, texto a la derecha, número del partido a
la derecha).

**MUY IMPORTANTE — evitar que las líneas/recuadros queden desalineados:**
- Cada columna `.round` es `display:flex; flex-direction:column`. El **título de la ronda** va arriba como elemento
  fijo (NO debe participar del reparto vertical).
- Los partidos van dentro de un contenedor hijo **`.round-body`** con
  `flex:1 1 auto; display:flex; flex-direction:column; justify-content:space-around`. Así, en TODAS las columnas, los
  partidos quedan centrados con la misma fórmula `(k+0.5)·alto/n` y cada partido de una ronda cae en el **punto medio
  exacto** de sus dos partidos hijos. (Si el título se deja dentro del space-around, se desalinea: NO lo hagas.)
- La columna **Final** (`.final-col`, ancho fijo `flex:0 0 200px`) también mete su único partido en un `.round-body`
  (queda a 0.5·alto, alineado con las semifinales). El bloque del **Campeón** (`.champion-wrap`) va
  **fuera del flujo**: `position:absolute; left:0; right:0; top:50%; margin-top:50px`, para no descentrar la Final.

**Cada partido (`.match`)**: dos filas `.team` (bandera + código de 3 letras + nombre + `%`). El `%` se muestra solo
cuando ambos equipos son reales (no placeholder) y es `round(pWin(equipo, rival)*100)`. Estados visuales:
`.win` (degradado verde, resaltado), `.lose` (atenuado + tachado). En ronda 0 cada `.match` muestra un número
(1..16). Placeholder/empty: bandera ⚪ y texto "Por definir".

**Conectores (líneas):** un overlay `#lines` (absolute, detrás de los partidos, `z-index:0`; los partidos
`z-index:1`). Función `drawConnectors()` que, leyendo `getBoundingClientRect()` de cada partido y su contenedor,
dibuja para r=1..4 e i en cada ronda **3 segmentos en codo** (div `.conn` con `background:var(--line)`): desde el
borde interno de cada hijo (`2i`, `2i+1`) horizontal hasta el x medio del gap, vertical hasta el centro-y del
partido destino, y horizontal hasta el borde interno del destino. Detecta el lado comparando centros-x (si el hijo
está a la izquierda usa su borde derecho, si está a la derecha usa el izquierdo). Redibuja en `requestAnimationFrame`
tras render y en el evento `resize`.

## PANEL DE INFO AL PASAR EL MOUSE (no debe tapar el cuadro)

- Un panel **`#mtip`** que aparece **DEBAJO del cuadro** (insertado por JS justo después de `.scroller`, centrado,
  `max-width:680px`, borde `--accent2`, fondo casi negro). NO sigue al cursor.
- Al hacer **`mouseenter`** sobre un `.match`, el panel se actualiza con:
  - Título: `bandera Nombre  vs  bandera Nombre`.
  - Si es ronda 0 (16avos): línea meta con `🗓 fecha · hora (hora Col.) · 📍 sede · 📺 canal` (usa `MATCHINFO`;
    si falta fecha/hora muestra "Fecha por confirmar" / "hora por confirmar").
  - Si es ronda posterior: `🗓 Se define al avanzar el cuadro`.
  - Línea favorito (si ambos reales): `⭐ Favorito: bandera Nombre (XX%)`, donde el favorito es el de mayor `rating`
    y XX = `round(pWin(fav, otro)*100)`, en color dorado.
- Al salir el mouse, **mantiene el último partido mostrado** (no parpadea). Al cargar, muestra una pista breve
  ("Pasa el mouse por un partido para ver fecha, hora de transmisión y favorito.").

## BOTONES

- **⚡ Simular por probabilidad**: `initState()`, recorre rondas 0..4 y en cada partido con ambos equipos reales
  fija ganador con `sampleKO`; propaga ronda a ronda; `render()`.
- **🎲 Azar**: igual pero ganador 50/50 (`Math.random()<.5`).
- **↺ Reiniciar**: `initState(); render()`.

## FASE DE GRUPOS — DEBE QUEDAR COMENTADA PERO RECICLABLE

La versión previa tenía, **arriba del cuadro**, una fase de grupos del sábado 27 (Grupos J, K, L) que hoy ya no
aplica. Inclúyela **comentada** (HTML entre `<!-- ... -->`, JS entre `/* ... */`) con etiquetas claras
tipo `RECICLABLE`, de modo que se pueda reactivar en un futuro Mundial descomentando y volviendo a llamar
`renderGroups()` / `renderScorers()` en `render()`/init. Conténtido a dejar comentado (no se ejecuta, no debe romper
el JS vivo):

1. **Datos `GROUPS`** (J/K/L) con `matches`, `teams` y `stats` (pts/gd/gf tras 2 fechas):
   - J: Argentina(6,+3,3), Austria(3,0,2), Argelia(3,0,2), Jordania(0,-3,1). Partidos: Argentina-Jordania, Argelia-Austria.
   - K: Colombia(6,+3,4), Portugal(4,+5,6), RD Congo(1,-1,1), Uzbekistán(0,-7,1). Partidos: Colombia-Portugal, RD Congo-Uzbekistán.
   - L: Inglaterra(4,+2,4), Ghana(4,+1,1), Croacia(3,-1,3), Panamá(0,-2,0). Partidos: Panamá-Inglaterra, Croacia-Ghana.
   - `teamByName`.
2. **Tablas de grupo**: `computeGroup(g,outs)` (suma 3/1/0, ordena por `P, GD, GF, seed`), `lockedTeam(grp,pos)`
   (devuelve el equipo de una posición solo si está **matemáticamente asegurado** probando por fuerza bruta los
   partidos sin jugar, 3^n), `groupDecided`.
3. **UI de grupos** `renderGroups()`: tarjetas por grupo con tabla (resalta 1°/2° clasificados y 3° como mejor
   tercero), y por partido **3 botones** (gana/empate/gana) con **tooltip** del nombre en hover
   (`[data-tip]:hover::after`), una línea de **probabilidad %** y una línea de **cuota tipo BetPlay** en cajas
   doradas (`cuota = 1/(prob*1.07)`, mínimo 1.04). Clic en un botón llama `setOutcome` y, vía `lockedTeam`, completa
   los cupos del cuadro en cuanto quedan asegurados.
4. **Goleadores** `SCORERS` (20 jugadores de J/K/L con `pct`) y `renderScorers()` (barra + %).
5. **Contador de simulaciones de grupos** `SIM`/`recordSim`/`renderSimStats`: tabla que acumula por clic los
   resultados (Gana/Empate/Gana por partido), con botón "limpiar"; se reinicia al recargar la página.

Mantén también el CSS de esas secciones (clases `.groups/.groupcard/.gtable/.gmatch/.gprob/.godds/.gnote/.simstats/
.ss-*/.scorers/.scorer`) aunque queden sin uso, para el reciclaje.

## AGENDA EN VIVO — "PRÓXIMO PARTIDO / TRANSMITIENDO" (esquina inferior izquierda)

Un panel **fijo, pequeño y discreto** en la **esquina inferior izquierda** (`.ticker`: `position:fixed; left:8px;
bottom:10px; max-width:215px; font-size:10px; fondo translúcido oscuro con borde --line y blur; en móvil aún más
pequeño`). Es **independiente** del panel `#mtip` y del cuadro.

Función `renderTicker()` que **lee la fecha-hora ACTUAL del sistema** (`Date.now()`) y, usando los `iso` de
`MATCHINFO` (sólo los partidos con `iso` no nulo, interpretados como UTC-5: `new Date(iso+"-05:00")`):

- Ordena los partidos por hora de inicio.
- **En vivo / Transmitiendo**: si la hora actual está dentro de `[inicio, inicio + 120 min)`, muestra
  `🔴 Transmitiendo · A vs B · 📺 canal` (si hay varios en vivo, los muestra todos).
- **Próximo**: muestra el/los partido(s) futuros con la hora de inicio más cercana:
  `⏭ Próximo · A vs B · 🗓 fecha · ⏰ hora (hora Col.) · 📺 canal`. **Si dos parten a la misma hora, muestra los dos.**
- Si ya no quedan partidos programados ni en vivo: `🏁 16avos finalizados`.
- Encabezado pequeño `Agenda · 16avos`. Equipos mostrados como `bandera CÓDIGO`.

Se llama al iniciar y se **autoactualiza con `setInterval(renderTicker, 30000)`** (cada 30 s, sin recargar). Como
depende del reloj real, al abrir la página el contenido cambia según la hora del día.

## FOOTER

- Texto: `16avos de final confirmados tras la fase de grupos (28-jun-2026) · Probabilidades estimadas con modelo Elo.`
- Debajo, **firma discreta** (clase `.sign`, fuente ~10.5px, color tenue `--line`, opacidad ~.85, `inline-flex`,
  gap 5px) con un **ícono SVG de base de datos** (cilindro: una `ellipse` arriba + dos `path` de cuerpos apilados,
  `stroke=currentColor`, en color `--accent2` con opacidad ~.7) seguido del texto:
  **`Jose Miguel Causado · DBA`**.

## INICIALIZACIÓN

Al final del script: `initState(); render(); renderTicker(); setInterval(renderTicker, 30000);`
Listeners: clic en cada `.team` → `pick`; `mouseenter` en cada `.match` → actualizar `#mtip`; `resize` → redibujar
conectores; los 3 botones de la barra.

## ENTREGABLE

Un solo `index.html` válido, sin errores de consola, que:
1. Muestre el cuadro de 16avos a Final con líneas perfectamente alineadas (izquierda y derecha simétricas).
2. Permita elegir ganadores con clic y propagar hasta el campeón (con copa 🏆).
3. Simule todo el cuadro por probabilidad o por azar.
4. Muestre el panel de info (fecha/hora/sede/canal/favorito) debajo del cuadro al pasar el mouse.
5. Tenga la agenda en vivo (⏭ Próximo / 🔴 Transmitiendo) discreta abajo-izquierda, según la hora real, autoactualizada.
6. Tenga la fase de grupos comentada y reutilizable.
7. Se vea bien en escritorio y celular.
