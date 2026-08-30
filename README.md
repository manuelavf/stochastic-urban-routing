# Stochastic Urban Routing

Modelo de **optimización estocástica de dos etapas** para el ruteo de una cuadrilla municipal en Manhattan, con tiempos de viaje inciertos estimados a partir de datos reales de la NYC Taxi & Limousine Commission (TLC).

- **Primera etapa:** decisión binaria de ruteo (qué arcos usar) antes de conocer el tráfico del día.
- **Segunda etapa:** decisiones continuas de recurso (trabajo directo, tercerización, tiempo adicional/emergencia) una vez se revelan los tiempos de viaje.

El proyecto construye una aproximación por promedio muestral (**SAA**), valida una formulación extensa como **MILP**, implementa el algoritmo **Integer L-shaped** (descomposición de Benders con recurso entero), y compara la solución estocástica contra una aproximación determinista basada en el perfil promedio de tiempos.

> Proyecto desarrollado para el curso de Optimización Estocástica — Maestría en Matemáticas Aplicadas, Universidad EAFIT (2026-2).

---

## Arquitectura del repositorio

```
stochastic-urban-routing/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   ├── raw/            # Parquet original de la TLC (NO se versiona, ver .gitignore)
│   └── processed/      # Pools, escenarios y demás datos derivados (sí se versionan)
├── notebooks/
│   ├── 01_pools_and_qc.ipynb       # Descarga, filtros, pools por arco, control de calidad
│   ├── 02_scenarios.ipynb          # Generación de escenarios ξ (SAA)
│   ├── 03_extensive_form.ipynb     # Formulación extensa MILP
│   ├── 04_lshaped.ipynb            # Algoritmo Integer L-shaped
│   ├── 05_main_experiment.ipynb    # Experimento principal K=200, perfil promedio, VSS
│   └── 06_out_of_sample.ipynb      # Validación temporal con datos de febrero
├── src/
│   └── utils.py                    # Funciones compartidas entre notebooks
└── report/
    └── informe_tecnico.pdf         # Entregable principal
```

Cada notebook lee las salidas del anterior desde `data/processed/`, así que deben ejecutarse **en orden** la primera vez.

---

## Cómo reproducir el análisis

### 1. Clonar el repositorio

```bash
git clone https://github.com/<tu-usuario>/stochastic-urban-routing.git
cd stochastic-urban-routing
```

### 2. Crear el entorno virtual

```bash
python3 -m venv .venv
source .venv/bin/activate        # Linux/Mac
# .venv\Scripts\activate         # Windows
```

### 3. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 4. Registrar el kernel de Jupyter

```bash
python -m ipykernel install --user --name=stochastic-urban-routing --display-name "Python (stochastic-urban-routing)"
```

Al abrir cualquier notebook (en Jupyter o VS Code), selecciona el kernel **"Python (stochastic-urban-routing)"**.

### 5. Ejecutar los notebooks en orden

```
notebooks/01_pools_and_qc.ipynb        →  data/processed/pools.parquet, arc_costs.csv, sites.csv
notebooks/02_scenarios.ipynb           →  data/processed/scenarios_K50.parquet, scenarios_K200.parquet
notebooks/03_extensive_form.ipynb      →  validación con K=50
notebooks/04_lshaped.ipynb             →  algoritmo Integer L-shaped
notebooks/05_main_experiment.ipynb     →  experimento principal, perfil promedio, VSS_K
notebooks/06_out_of_sample.ipynb       →  validación con datos de febrero
```

El primer notebook descarga automáticamente el archivo oficial de la TLC (~175 MB) si no lo encuentra en `data/raw/`. La descarga solo se hace una vez.

---

## Datos

Los datos crudos (`data/raw/`) **no se versionan** en este repositorio, ya que corresponden a archivos públicos de gran tamaño publicados por la TLC:

- Enero 2015: https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2015-01.parquet
- Febrero 2015: https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2015-02.parquet

Los datos procesados (pools, escenarios, costos por arco) sí se versionan en `data/processed/` para facilitar la reproducibilidad sin tener que volver a descargar ni filtrar el Parquet original.

---

## Metodología (resumen)

1. **Datos y pools:** se filtran viajes de taxi entre las 11 ubicaciones del problema (depósito + 10 sitios), en horario laboral entre semana, y se construye un pool empírico de tiempos de viaje por cada uno de los 110 arcos dirigidos.
2. **Escenarios:** se muestrean $K$ escenarios sintéticos de tiempos de viaje, con reemplazo, de forma independiente entre arcos.
3. **Modelo SAA:** ruta óptima (primera etapa, binaria) + subproblema de recurso continuo por escenario (segunda etapa).
4. **Validación algorítmica:** se comparan la formulación extensa (MILP monolítico) y el algoritmo Integer L-shaped sobre la misma muestra (K=50), verificando que los valores objetivo coincidan.
5. **Experimento principal (K=200):** se obtiene la ruta óptima estocástica $x^\star$ y se compara contra una ruta determinista $x^{prom}$ basada en el perfil promedio de tiempos, calculando el valor de la solución estocástica (VSS).
6. **Validación fuera de muestra:** ambas rutas se evalúan, sin reoptimizar, sobre 1000 escenarios sintéticos generados a partir de datos de febrero de 2015.

Detalles completos de la formulación matemática, el algoritmo y los resultados en [`report/informe_tecnico.pdf`](report/informe_tecnico.pdf).

---

## Referencias

- Miller, C. E., Tucker, A. W., & Zemlin, R. A. (1960). Integer programming formulation of traveling salesman problems. *Journal of the ACM*, 7(4), 326-329.
- Laporte, G., & Louveaux, F. V. (1993). The integer L-shaped method for stochastic integer programs with complete recourse. *Operations Research Letters*, 13(3), 133-142.
- Kleywegt, A. J., Shapiro, A., & Homem-de-Mello, T. (2002). The sample average approximation method for stochastic discrete optimization. *SIAM Journal on Optimization*, 12(2), 479-502.
- Birge, J. R., & Louveaux, F. (2011). *Introduction to Stochastic Programming* (2nd ed.). Springer.
- NYC Taxi & Limousine Commission. [TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page).

## Licencia

Este proyecto se distribuye bajo la licencia especificada en [`LICENSE`](LICENSE).