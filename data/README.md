# Data Sources

Este proyecto utiliza datasets y recursos públicos de transcriptómica, anotación génica y regulación miRNA.

Los archivos originales **no se incluyen en este repositorio**.

## 1. GSE42568 — mRNA

Repositorio:

**NCBI Gene Expression Omnibus (GEO)**

Accession:

```text
GSE42568
```

Contenido utilizado:

- biopsias de cáncer de mama;
- tejido mamario normal;
- plataforma Affymetrix GPL570.

El pipeline convierte probes Affymetrix a símbolos génicos utilizando la anotación de GPL570.

## 2. GSE73002 — miRNA

Repositorio:

**NCBI Gene Expression Omnibus (GEO)**

Accession:

```text
GSE73002
```

El proyecto utiliza una versión filtrada denominada `GSE73002-F`, restringida al problema:

```text
CANCER vs NORMAL / non-cancer
```

Los identificadores MIMAT se convierten a nombres de miRNA humano maduro antes de realizar el mapeo hacia genes objetivo.

## 3. E-MTAB-6703 — mRNA

Repositorio:

**ArrayExpress / BioStudies**

Accession:

```text
E-MTAB-6703
```

Se utiliza como fuente independiente de evidencia a nivel de genes.

## Recursos de anotación y mapeo

### GPL570

Plataforma:

```text
Affymetrix Human Genome U133 Plus 2.0 Array
```

Se utiliza para realizar:

```text
probe → gene symbol
```

### miRBase

Se utiliza para realizar:

```text
MIMAT accession → mature human miRNA
```

### miRTarBase / Harmonizome

Se utiliza para obtener interacciones:

```text
miRNA → target gene
```

con soporte experimental reportado en las fuentes originales.

## Estructura local esperada

Los nombres concretos de archivos pueden variar según la versión descargada o preprocesada.

El notebook permite configurar las rutas utilizadas durante el análisis.

Una estructura local típica es:

```text
data/
├── README.md
├── GSE42568/
├── GSE73002/
├── E-MTAB-6703/
└── annotations/
```

## Reproducibilidad y fuentes externas

Parte del pipeline depende de recursos externos de anotación y mapeo biológico.

En particular, etapas como:

```text
probe → gene
MIMAT → mature miRNA
miRNA → target gene
```

pueden depender del contenido disponible en las fuentes externas en el momento de la ejecución.

Por esta razón, una reejecución realizada posteriormente puede producir pequeñas variaciones en:

- número de identificadores mapeados;
- número de interacciones miRNA–target;
- tamaño de los conjuntos derivados;
- p-values y métricas de solapamiento asociadas.

La versión asociada al manuscrito conserva los outputs de la ejecución original utilizada para generar los resultados reportados.

Cuando estén disponibles, los artefactos intermedios de esa ejecución deben preservarse para facilitar una reproducción exacta sin depender nuevamente de servicios externos.