# Índice ERU — San Isidro Club

Dashboard interactivo de rendimiento GPS para el plantel de San Isidro Club (SIC),
construido alrededor de un indicador propio de carga/esfuerzo por sesión: el
**Índice ERU**.

El sitio publicado es un único archivo HTML autocontenido (sin dependencias
externas en tiempo de ejecución): filtros en cascada, tarjetas de máximas
prestaciones, un scatter Índice ERU vs. Duración con escudos por rival, y una
tabla ordenable con escala de color.

## Cómo se calcula el Índice ERU

Para cada sesión (`Periodo = "Session"`) se ponderan cinco métricas —
Duración, Distancia Explosiva, RHIE (bouts totales), BiG (Back in Game, en
segundos) y Contactos— con pesos distintos según el grupo posicional del
jugador:

| Grupo posicional         | Duración | Dist Exp | RHIE | BiG | Contactos |
|---------------------------|:--:|:--:|:--:|:--:|:--:|
| Primera Línea              | 10% | 20% | 20% | 25% | 25% |
| Segunda/Tercera Línea       | 10% | 20% | 15% | 25% | 30% |
| Medio Scrum                 | 20% | 20% | 25% | 15% | 10% |
| Backs Internos (Apertura, Centro) | 20% | 25% | 25% | 15% | 15% |
| Backs Externos (Wing, Fullback)   | 20% | 35% | 25% | 10% | 10% |

Cada métrica se normaliza min–max **dentro de la temporada correspondiente**
(considerando todos los puestos y actividades de esa temporada). BiG se
invierte antes de ponderar, porque un valor menor indica mejor desempeño.

El resultado se expresa de dos formas:
- **Unidades arbitrarias** (0–100 aprox.): la suma ponderada de las cinco
  métricas normalizadas.
- **% del máximo**: el 100% de cada jugador corresponde a su propio partido
  de mayor Índice ERU dentro de la temporada seleccionada. Si un jugador no
  disputó partidos esa temporada, se usa su mejor registro de entrenamiento
  como referencia. Esta referencia se fija una sola vez por jugador y
  temporada, y no cambia al aplicar filtros de puesto o actividad.

## Estructura del repo

```
data/SIC_Carga.xlsx        Planilla fuente (GPS, hoja "GPS")
assets/logos/               Escudos de SIC y rivales (el ícono de pelota para
                             entrenamientos y rivales sin escudo se genera en
                             app.js, no es un archivo)
crest_map.json              Mapeo código de rival → archivo de escudo
dashboard/template.html     Estructura, CSS y placeholders del dashboard
dashboard/app.js            Lógica de filtros, tarjetas, tabla y gráfico
vendor/chart.umd.js         Chart.js 4.4.1 vendorizado (sin CDN)
scripts/build_dashboard.py  Calcula el Índice ERU e inyecta todo en el HTML
dist/index.html             Salida del build (generada, no versionada)
```

## Build local

```bash
pip install -r requirements.txt
python scripts/build_dashboard.py
# genera dist/index.html — abrilo directo en el navegador
```

## Actualizar datos o escudos

1. Reemplazá `data/SIC_Carga.xlsx` por la planilla actualizada (misma
   estructura de columnas, hoja `GPS`).
2. Si aparece un rival nuevo, sumá su escudo en `assets/logos/` y su entrada
   en `crest_map.json` (código de rival tal como aparece después de
   `"SIC vs "` en la columna Actividad, sin sufijos `(Int)`/`2`/`Semi`/etc.).
3. Corré `python scripts/build_dashboard.py` para regenerar `dist/index.html`,
   o simplemente hacé push a `main`: el workflow de GitHub Actions
   (`.github/workflows/deploy.yml`) reconstruye y publica el sitio en
   GitHub Pages automáticamente.

## Notas de datos y supuestos

- Se usa exclusivamente `Periodo = "Session"`, para evitar doble conteo de
  bloques/tramos parciales dentro de una misma sesión.
- "RHIE" corresponde a la columna `RHIE Total Bouts`; "BiG" corresponde a
  `# BiG` (segundos, decimal) — se descartaron BiG (Largo/Medio/Corto) por
  ser conteos, no tiempos.
- Se excluye 1 registro con Puesto en blanco y 1 con Duración vacía.
- 176 registros con Dist Exp/RHIE/BiG/Contactos en cero se conservan
  (mayormente entrada en calor): son el piso natural de la normalización.
- 30 combinaciones jugador+actividad+fecha aparecen duplicadas en el archivo
  original; se conservan ambos registros tal como están en la fuente.
- Los rivales sin escudo provisto (ver `_sin_logo` en `crest_map.json`) usan
  el ícono genérico de pelota, igual que todos los entrenamientos.

## Stack

Python (pandas/openpyxl) para el build · Chart.js 4.4.1 vendorizado · HTML/CSS/JS
sin frameworks · GitHub Actions + GitHub Pages para CI/CD.
