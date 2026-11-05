# Geoestadística de la temperatura máxima en Antioquia: autocorrelación espacial y kriging

**EN** — Spatial statistics project on daily maximum temperature at weather-grid points in Antioquia, Colombia, motivated by heat in the Aburrá Valley. It tests normality and spatial autocorrelation (Mantel test), builds empirical and fitted semivariograms, and interpolates temperature over the department with ordinary and universal kriging, validated by cross-validation. Written in R with Quarto and published as a web report.

**ES** — Proyecto de estadística espacial sobre la temperatura máxima diaria en puntos de una malla climática de Antioquia, Colombia, motivado por el calor en el Valle de Aburrá. Se prueba la normalidad y la autocorrelación espacial (test de Mantel), se construyen semivariogramas empíricos y ajustados y se interpola la temperatura sobre el departamento con kriging ordinario y universal, validado por validación cruzada. Desarrollado en R con Quarto y publicado como informe web.

Informe publicado: <https://dvillaunal.github.io/geoestadistica-temperatura-valle-aburra/>

## Objetivo

Responder qué zonas de Antioquia presentan las mayores temperaturas y qué factores se asocian a ese patrón, usando la temperatura máxima de un día como variable regionalizada para modelar su estructura espacial y predecirla en todo el departamento. El documento contextualiza el problema con el caso del Valle de Aburrá (isla de calor urbana, temporadas secas, incendios forestales).

## Datos

- **Temperatura:** `databases/temperature.csv`, datos diarios de la malla climática de [Global Weather Data for SWAT](https://swat.tamu.edu/data/) con latitud, longitud, elevación, temperatura mínima y máxima: 32 puntos de malla con datos diarios del 2018-03-01 al 2019-12-31 (21.472 registros). Se analizan dos fechas, 2019-01-10 y 2019-10-21; tras excluir puntos y faltantes, el análisis de 2019-10-21 usa 22 ubicaciones. `databases/historicalData.zip` contiene la descarga original.
- **Cartografía:** Marco Geoestadístico Nacional del [DANE](https://geoportal.dane.gov.co/servicios/descarga-y-metadatos/descarga-mgn-marco-geoestadistico-nacional/) (`MGN_DPTO_POLITICO`, `MGN_MPIO_POLITICO`), de donde se extrae el contorno de Antioquia.
- **Otros archivos:** `base_cienaga.xls`, `cienaga_border.txt` y `borde_antioquia.csv` (bordes y datos auxiliares).

Todos los datos están incluidos en el repositorio. Los shapefiles del MGN se versionan solo con `.shp` y `.shx` (sin `.dbf` ni `.prj`).

## Métodos

- Selección de un día (análisis espacial, no espacio-temporal) y depuración de puntos.
- Normalidad: gráfico Q-Q y prueba de Shapiro-Wilk.
- Autocorrelación espacial: gráfico de distancia frente a disimilitud y test de Mantel (`vegan::mantel`, `ade4::mantel.rtest`).
- Transformación de coordenadas de EPSG:4326 a EPSG:9377 (MAGNA-SIRGAS origen nacional).
- Semivariogramas empíricos (nube y por clases, con distancia máxima por regla empírica), bandas de independencia con `sm.variogram` y ajuste de un modelo Matérn con `geoR::variofit`.
- Predicción sobre una malla de 10.000 puntos recortada al contorno de Antioquia:
  - kriging con `geoR::krige.conv` (tendencia de segundo orden en las coordenadas), mapa de la desviación estándar de predicción y validación cruzada con `xvalid`;
  - kriging universal con `gstat::krige` (tendencia lineal en X e Y, variograma exponencial) y mapa de su varianza.

## Resultados principales

- **2019-01-10:** la temperatura máxima no es normal (Shapiro-Wilk W = 0,909, p = 0,046) y el test de Mantel no detecta autocorrelación espacial (r = 0,099, p = 0,127).
- **2019-10-21:** no se rechaza la normalidad (W = 0,930, p = 0,122) y el test de Mantel sí indica autocorrelación espacial (r = 0,150, p = 0,042), por lo que esta fecha se usa para el modelado.
- **Validación cruzada del kriging con `geoR` (22 ubicaciones):** error medio 0,134 °C, desviación estándar del error 2,02 °C y errores entre -3,68 y 4,01 °C.
- Según las conclusiones del equipo (`razones kriging universal.txt`), el kriging universal fue el mejor modelo por su menor error en validación cruzada y su capacidad de incorporar la tendencia espacial; esa comparación no aparece cuantificada en las salidas.
- La conclusión del informe, apoyada en la literatura citada, asocia las temperaturas más altas a las subregiones bajas y cercanas al Caribe o al río Magdalena (Bajo Cauca, Magdalena Medio, Urabá) y las más bajas a las de mayor altitud (Suroeste, Oriente, Occidente).

## Estructura del repositorio

```
trabajo_geoestadistica.qmd    Informe final (fuente Quarto)
trabajo_geoestadistica.html   Informe final renderizado (también publicado en GitHub Pages)
data_pre.html                 Versión previa: presentación de los datos
razones kriging universal.txt Notas sobre la elección del kriging universal
databases/                    Temperatura (SWAT), shapefiles del DANE y bordes auxiliares
images/                       Figuras del informe (Gobernación de Antioquia, SIATA)
Proyecto_Espacial.Rproj       Proyecto de RStudio
```

La rama `gh-pages` contiene la copia publicada del informe (`index.html`).

## Cómo reproducir

- **Software:** R y Quarto (salida HTML). La versión de R no está registrada en el repositorio.
- **Paquetes:** `geoR`, `gstat`, `sf`, `raster`, `rgdal`, `vegan`, `ade4`, `tidyverse`, `magrittr`, `tmaptools`, `tidygeocoder`, `leaflet`, `sm`, `MASS`, `gt`. `rgdal` ya no está en CRAN; para ejecutar hoy el código puede ser necesario instalarlo desde el archivo de CRAN o reemplazar las transformaciones de `sp` por `sf`.
- **Ejecución:** abrir `Proyecto_Espacial.Rproj` y ejecutar `quarto render trabajo_geoestadistica.qmd`.

## Contexto y autoría

Proyecto del curso de Estadística Espacial, primer semestre de 2023 (marzo a mayo de 2023), del pregrado en Estadística de Daniel Felipe Villa Rengifo en la Universidad Nacional de Colombia, sede Medellín.

Autores: Daniel Villa, Raquel Latorre, Ivan Rojas y Juan Esteban Restrepo.
