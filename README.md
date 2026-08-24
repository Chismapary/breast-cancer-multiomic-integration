# Breast Cancer Multi-Omic Integration — mRNA & miRNA

Pipeline de bioinformática para integrar evidencia de expresión diferencial de **mRNA** y **miRNA**
en cáncer de mama, evaluar convergencia entre datasets y priorizar genes candidatos mediante
métricas de solapamiento, pruebas de enriquecimiento y corrección por múltiples comparaciones.

El proyecto utiliza tres datasets públicos de microarreglos:

- **GSE42568** — expresión mRNA en tejido mamario;
- **GSE73002-F** — expresión de miRNA sérico;
- **E-MTAB-6703** — meta-dataset mRNA independiente.

## Alcance y autoría

Este repositorio contiene **exclusivamente la rama de integración multi-ómica y enriquecimiento funcional desarrollada por el autor del repositorio**.

El trabajo de investigación asociado contiene además una rama predictiva de Machine Learning desarrollada dentro de un trabajo colaborativo. **Ese código no forma parte de este repositorio y no se atribuye aquí.**

El manuscrito asociado se encuentra actualmente bajo revisión.

## Nota de reproducibilidad

Este repositorio contiene una **ejecución actual del pipeline** realizada con los recursos externos disponibles al momento de dicha ejecución.

Algunas etapas dependen de fuentes externas de anotación y mapeo biológico, especialmente:

```text
probe → gene
MIMAT → mature miRNA
miRNA → target gene
```

Por ello, pueden existir pequeñas diferencias numéricas respecto de la ejecución canónica utilizada para generar los resultados del manuscrito asociado.

Estas diferencias pueden afectar principalmente:

- el número de identificadores mapeados;
- el número de interacciones miRNA–target;
- el tamaño de los conjuntos derivados;
- métricas como IoU, coverage y p-values.

La metodología, la progresión experimental y las conclusiones principales se mantienen consistentes.

La ejecución canónica asociada al manuscrito se conserva por separado con sus outputs originales.

## Objetivo

El objetivo es estudiar si existe una convergencia no aleatoria entre tres tipos de evidencia:

```text
A = genes diferencialmente expresados en GSE42568

B = genes diferencialmente expresados o priorizados
    en E-MTAB-6703

C = genes objetivo de miRNAs diferencialmente
    expresados en GSE73002-F
```

En lugar de asumir que un alto rendimiento predictivo implica interpretabilidad biológica,
el análisis evalúa directamente la convergencia entre evidencia mRNA y regulación asociada a miRNA.

## Flujo general

```text
GSE42568 mRNA
      │
      ├── probes Affymetrix → genes
      │
      └── expresión diferencial ──────────────┐
                                               │
E-MTAB-6703 mRNA                              ├── A / B / C
      │                                        │
      └── expresión diferencial / ranking ────┤
                                               │
GSE73002-F miRNA                              │
      │                                        │
      ├── MIMAT → miRNA maduro                 │
      ├── expresión diferencial                │
      └── miRNA → genes objetivo ──────────────┘
                                               │
                                               ▼
                                      IoU + Coverage
                                               │
                                               ▼
                                       Fisher exact test
                                               │
                                               ▼
                                  1,460 escenarios evaluados
                                               │
                                               ▼
                                   Bonferroni + BH-FDR
                                               │
                                               ▼
                                    genes candidatos robustos
                                               │
                                               ▼
                                   enriquecimiento funcional
```

## Procesamiento molecular

### GSE42568

La matriz original contiene probes Affymetrix GPL570.

El pipeline realiza:

```text
probe → gene symbol
```

y colapsa múltiples probes asociados a un mismo gen.

### GSE73002-F

Los identificadores MIMAT se convierten a nombres de miRNA humano maduro.

Posteriormente, los miRNAs diferencialmente expresados se relacionan con genes objetivo mediante
interacciones experimentalmente soportadas derivadas de recursos como miRTarBase/Harmonizome.

### E-MTAB-6703

Se utiliza como una fuente independiente de evidencia mRNA.

El análisis emplea tanto criterios estrictos de expresión diferencial como rankings de evidencia
diferencial para estudiar señales distribuidas de magnitud moderada.

## Expresión diferencial

Para cada feature se calcula la diferencia entre los grupos cáncer y normal:

```text
log2FC = media(CANCER) - media(NORMAL)
```

Se utiliza **Welch's t-test** y corrección **Benjamini–Hochberg FDR**.

El criterio estricto inicial es:

```text
FDR < 0.05
|log2FC| >= 1
```

## Experimentos

### Experimento 1 — IoU directo

Se construyen conjuntos estrictos A, B y C.

Resultados de la ejecución actual:

| Métrica | Resultado |
|---|---:|
| |A| | 2,501 |
| |B| | 19 |
| |C| | **11,533** |
| Intersección triple | **11 genes** |
| IoU | **0.000880** |
| Coverage de B | **57.9 %** |

El IoU es muy pequeño porque el tamaño de C domina la unión.

Sin embargo, **11 de los 19 genes** del conjunto B estricto aparecen en la intersección triple.

Este resultado motivó no utilizar IoU como única evidencia.

### Experimento 2 — IoU refinado y enriquecimiento

Se introduce:

- filtrado de C al universo comparable de genes;
- coverage;
- Fisher exact test;
- relaciones miRNA–mRNA direccionalmente consistentes;
- conjuntos Top-N de miRNAs.

Resultados principales de la ejecución actual:

```text
11 genes en la intersección
Coverage B = 57.9 %
|C| = 10,184
p ≈ 0.07257
```

El resultado no alcanza significancia estadística, por lo que se continúa con una búsqueda sistemática de escenarios.

### Experimento 3 — Scenario search

Se exploran combinaciones de:

- conjuntos A estrictos o rankeados;
- conjuntos B estrictos o Top-N;
- Top-N de miRNAs;
- número mínimo de miRNAs que soportan cada target;
- targets globales o direccionales.

En total se evalúan:

**1,460 escenarios**

El mejor escenario exploratorio de la ejecución actual obtuvo:

| Métrica | Resultado |
|---|---:|
| |C| | **3,672** |
| Triple intersection | **77 genes** |
| Coverage de B | **30.8 %** |
| Odds ratio | **5.039** |
| Fisher p-value | **8.60 × 10⁻²²** |

La búsqueda masiva introduce un problema de multiple testing, por lo que estos p-values no se utilizan como conclusión final sin corrección.

### Experimento 4 — Corrección múltiple y genes robustos

Los 1,460 tests se corrigen mediante:

- **Bonferroni**
- **Benjamini–Hochberg FDR**

Resultados de la ejecución actual:

```text
75 escenarios significativos después de Bonferroni
130 escenarios significativos después de BH-FDR
```

#### Mejor escenario global corregido

| Métrica | Resultado |
|---|---:|
| |C| | **10,184** |
| Triple intersection | **215 genes** |
| IoU | ≈ 0.0189 |
| Coverage B | **43.0 %** |
| Bonferroni p | **≈ 4.36 × 10⁻²⁹** |

#### Escenario direccional corregido

La ejecución actual conserva el mismo patrón de escenario direccional significativo, con pequeñas variaciones en el tamaño del conjunto C derivadas del mapeo externo actual.

#### Escenario direccional compacto

La ejecución actual conserva igualmente un escenario compacto direccional significativo tras corrección múltiple.

## Genes candidatos robustos

El Experimento 4 produce:

**220 genes candidatos robustos**

La estabilidad se evalúa contando en cuántos escenarios significativos aparece cada gen.

Entre los genes con evidencia recurrente se encuentran:

- `DMD`
- `TGFBR3`
- `PLSCR4`
- `NR3C1`
- `SPRY2`
- `HOXA5`
- `FOXO1`
- `LIFR`
- `IRS2`
- `CAV1`

La lista constituye una **priorización computacional** y no un panel clínicamente validado.

## Experimento 5 — Enriquecimiento funcional

Los genes candidatos robustos se utilizan como entrada para análisis funcional mediante Enrichr/gseapy.

Se analizan:

- GO Biological Process 2023;
- Reactome 2022;
- KEGG 2021 Human;
- MSigDB Hallmark 2020.

La ejecución actual evaluó:

```text
2,483 términos funcionales
293 términos significativos con FDR < 0.05
```

Entre los principales procesos enriquecidos se encontraron:

- **E2F Targets**
- **G2/M Checkpoint**
- **Cell Cycle**
- **Mitotic Cell Cycle**
- **Mitotic Spindle**

Algunos genes recurrentes en estos términos incluyen:

- `TOP2A`
- `PCNA`
- `CDCA8`
- `BUB1B`
- `AURKA`
- `KIF11`
- `SMC4`
- `LMNB1`

Esto aporta evidencia de coherencia funcional de los candidatos, pero **no demuestra causalidad ni validación experimental**.

## Comparación con la ejecución del manuscrito

La ejecución canónica utilizada para el manuscrito y la ejecución actual producen la misma narrativa experimental y las mismas conclusiones principales, aunque algunos valores derivados del mapeo externo cambian ligeramente.

Ejemplos:

| Resultado | Manuscrito | Ejecución actual |
|---|---:|---:|
| |C| — Exp. 1 | 11,685 | 11,533 |
| IoU — Exp. 1 | 0.000870 | 0.000880 |
| |C| — Exp. 2 | 10,224 | 10,184 |
| p — Exp. 2 | 0.0746 | 0.07257 |
| |C| — Exp. 3 | 3,698 | 3,672 |
| Intersección Exp. 3 | 77 | 77 |
| Escenarios Bonferroni | 74 | 75 |
| Escenarios FDR | 130 | 130 |
| Intersección global Exp. 4 | 215 | 215 |
| Coverage global | 43.0 % | 43.0 % |
| Genes robustos | 220 | 220 |
| Términos funcionales significativos | 293 | 293 |

La consistencia de los resultados principales es más importante que la coincidencia exacta de cada tamaño de conjunto derivado de fuentes externas.

## Por qué IoU no fue suficiente

Una de las conclusiones metodológicas principales del proyecto es que IoU puede ser engañoso cuando los conjuntos tienen tamaños muy asimétricos.

En el primer experimento:

```text
|B| = 19
|C| > 11,000
```

por lo que el denominador de:

```text
IoU = |A ∩ B ∩ C| / |A ∪ B ∪ C|
```

está dominado por C.

Así, un IoU muy pequeño coexistía con:

```text
11 / 19 = 57.9 %
```

de cobertura del conjunto B.

Por ello, el análisis evoluciona hacia:

```text
IoU
  ↓
Coverage
  ↓
Fisher exact test
  ↓
Directional consistency
  ↓
Scenario search
  ↓
Bonferroni / BH-FDR
  ↓
Gene stability
  ↓
Functional enrichment
```

## Estructura del repositorio

```text
breast-cancer-multiomic-integration/
├── README.md
├── multiomic_mrna_mirna_integration.ipynb
├── requirements.txt
├── .gitignore
├── data/
│   └── README.md
├── outputs/
└── figures/
```

## Datos

Los datasets no se redistribuyen en este repositorio.

Las fuentes utilizadas son públicas:

- GEO — GSE42568
- GEO — GSE73002
- ArrayExpress / BioStudies — E-MTAB-6703
- GEO Platform GPL570
- miRBase
- miRTarBase / Harmonizome

Consulta `data/README.md` para detalles sobre fuentes externas y reproducibilidad.

## Ejecución

### 1. Crear entorno

```bash
python -m venv .venv
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows:

```bash
.venv\Scripts\activate
```

### 2. Instalar dependencias

```bash
pip install -r requirements.txt
```

### 3. Preparar los datos

Consulta:

```text
data/README.md
```

### 4. Ejecutar

```bash
jupyter notebook multiomic_mrna_mirna_integration.ipynb
```

## Limitaciones

- GSE42568 y E-MTAB-6703 corresponden a mRNA de tejido, mientras GSE73002 utiliza miRNA sérico.
- Las bases de datos miRNA–target son incompletas y están sesgadas hacia interacciones más estudiadas.
- El mapeo probe→gene puede ser ambiguo.
- Parte de los resultados derivados depende de recursos externos que pueden evolucionar con el tiempo.
- La gran cantidad de escenarios exige corrección explícita por múltiples comparaciones.
- El enriquecimiento funcional también está condicionado por sesgos de anotación de las bases.
- Los genes candidatos requieren validación independiente y experimental.
- Este repositorio no constituye evidencia clínica ni propone biomarcadores validados.

## Tecnologías

- Python
- pandas
- NumPy
- SciPy
- statsmodels
- scikit-learn
- gseapy / Enrichr
- Matplotlib
- Jupyter Notebook

## Autor

**Ismael Gerson Ramos Alvarez**  
Ingeniería Informática y de Sistemas  
Universidad Nacional de San Antonio Abad del Cusco — UNSAAC

## Manuscrito asociado

Este pipeline forma parte de un trabajo de investigación colaborativo actualmente bajo revisión.

Este repositorio publica únicamente la parte de integración multi-ómica y enriquecimiento funcional desarrollada por el autor del repositorio.

La ejecución canónica utilizada para producir los resultados del manuscrito se preserva por separado para mantener trazabilidad.
