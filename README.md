[README.md](https://github.com/user-attachments/files/26583148/README.md)
#  Muestreo Complejo en R

**Ayudantía N°3 — Análisis Avanzado de Datos II**  


> Material educativo para el análisis de muestras complejas en R, aplicado a la Encuesta de Bienestar Social (EBS) 2023.

---

# Descripción

Este repositorio contiene los materiales de la tercera ayudantía del curso **Análisis Avanzado de Datos II**, donde se introduce el concepto de muestreo complejo y su correcta implementación en R mediante los paquetes `survey` y `srvyr`.

El material utiliza como caso práctico la **Encuesta de Bienestar Social (EBS) 2023**, que mide la calidad de vida de las personas en Chile incorporando dimensiones como satisfacción con la vida, salud mental y relaciones sociales.

---

## Contenidos

- ¿Qué son las muestras complejas?
- Estratificación y muestreo por conglomerados
- Ponderadores / Factores de expansión
- Creación del diseño muestral en R
- Cálculos estadísticos con diseño complejo
- Comparación entre resultados con y sin ponderadores
- Análisis de significancia con regresión (`svyglm`)
- Actividad práctica (10 minutos)

---

## Requisitos

```r
install.packages(c("tidyverse", "haven", "survey", "srvyr"))
```

| Paquete | Uso |
|---------|-----|
| `tidyverse` | Manipulación y visualización de datos |
| `haven` | Lectura de bases en formato SPSS/Stata |
| `survey` | Análisis con diseño muestral complejo |
| `srvyr` | Interfaz `tidyverse`-friendly para `survey` |

---

##  Datos

Se utiliza la **Encuesta de Bienestar Social 2023 (EBS)** en formato `.RData`.

- 📥 [Descargar base de datos](https://github.com/franimiau/Muestreo-Complejo)
- 📥 [Descargar libro de códigos](https://github.com/franimiau/Muestreo-Complejo)

### Variables clave del diseño muestral

| Variable | Descripción |
|----------|-------------|
| `fexp` | Factor de expansión nacional |
| `estrato_ebs` | Estratos de varianza |
| `varunit` | Conglomerados de varianza |

### Variables de análisis

| Variable | Descripción |
|----------|-------------|
| `a1` | Satisfacción con la vida (1–5) |
| `os1_recat` | Orientación sexual |
| `tramoebs2` | Tramo de edad |
| `sg01` | Sexo |

---


### 1. Cargar paquetes y datos

```r
library(tidyverse)
library(haven)
library(survey)
library(srvyr)

load("Base de datos EBS 2023.RData")
ebs <- EBS_2023_vp
```

### 2. Crear el diseño muestral

**Con `survey`:**

```r
ebs_survey <- svydesign(
  ids     = ~varunit,
  strata  = ~estrato_ebs,
  weights = ~fexp,
  data    = ebs
)
```

**Con `srvyr`:**

```r
ebs_srvyr <- as_survey_design(ebs,
  ids     = varunit,
  strata  = estrato_ebs,
  weights = fexp
)
```

### 3. Calcular estadísticas ponderadas

**Media ponderada (`survey`):**

```r
svymean(~a1, design = ebs_survey, na.rm = TRUE)
```

**Media ponderada por grupo (`srvyr`):**

```r
ebs_srvyr %>%
  group_by(os1_recat) %>%
  summarise(
    satis_media = survey_mean(a1, na.rm = TRUE)
  )
```

### 4. Regresión con diseño complejo

```r
modelo <- svyglm(as.numeric(a1) ~ os1_recat, design = ebs_survey)
summary(modelo)
```

---

## 📋 Referencia rápida de funciones

### `srvyr`

| Función | Descripción |
|---------|-------------|
| `as_survey_design()` | Crear el diseño muestral |
| `survey_mean()` | Media ponderada |
| `survey_total()` | Total poblacional estimado |
| `survey_prop()` | Proporciones ponderadas |
| `survey_quantile()` | Percentiles ponderados |
| `survey_median()` | Mediana ponderada |
| `survey_sd()` | Desviación estándar |

### `survey`

| Función | Descripción |
|---------|-------------|
| `svydesign()` | Crear el diseño muestral |
| `svymean()` | Media ponderada |
| `svytotal()` | Total poblacional |
| `svyby()` | Calcular por grupos |
| `svyquantile()` | Percentiles |
| `svytable()` | Tablas de frecuencia ponderadas |
| `svyglm()` | Regresión lineal o logística |
| `svychisq()` | Test chi-cuadrado |
| `svyttest()` | Test de medias (t-test) |

---

## 📝 Actividad práctica

Usando la base EBS y el objeto `ebs_srvyr`:

1. Calcular la media de satisfacción con la vida (`a1`) según sexo (`sg01`).
2. Calcular la proporción de personas satisfechas (categorías 4 y 5 de `a1`) según tramo de edad (`tramoebs2`).
3. Responder: ¿Qué grupo presenta mayor satisfacción? ¿Por qué es importante usar ponderadores?

**Opcional:** Comparar los resultados con y sin diseño muestral usando `mean()`.

---

## Autoría

Elaborado por **Francisca Hernández** (francisca.hernandez_c@mail.udp.cl), en base a los contenidos del curso y las clases del profesor **Gabriel Sotomayor**.

---

## 📚 Referencias

- Lumley, T. (2010). *Complex Surveys: A Guide to Analysis Using R*. Wiley.
- Documentación oficial: [`survey`](https://cran.r-project.org/package=survey) · [`srvyr`](https://cran.r-project.org/package=srvyr)
- Ministerio de Desarrollo Social y Familia. *Encuesta de Bienestar Social 2023*.
