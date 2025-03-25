

---
author:
- R. Strandberg, Y. Shang, J. Boursier, E. Bugianesi, CW. Kheong, J.
  Fan, GBB. Goh, VDE. Ledinghen, A. Nakajima, PN. Newsome, S. Petta, M.
  Romero-Gómez, AJ. Sanyal, MH. Zheng, JL. Calleja Panero, E.
  Tsochatzis, C. Fournier-Poizat, M. Sau-Wai Chan, G. Fong, L. Castera,
  M. Lai, V. Wai-Sun Wong, Hannes Hagström
authors:
- R. Strandberg, Y. Shang, J. Boursier, E. Bugianesi, CW. Kheong, J.
  Fan, GBB. Goh, VDE. Ledinghen, A. Nakajima, PN. Newsome, S. Petta, M.
  Romero-Gómez, AJ. Sanyal, MH. Zheng, JL. Calleja Panero, E.
  Tsochatzis, C. Fournier-Poizat, M. Sau-Wai Chan, G. Fong, L. Castera,
  M. Lai, V. Wai-Sun Wong, Hannes Hagström
header-includes:
- |
  <script src="LiverRisk_val_files/libs/htmlwidgets-1.6.4/htmlwidgets.js"></script>
  <script src="LiverRisk_val_files/libs/jquery-3.5.1/jquery.min.js"></script>
  <link href="LiverRisk_val_files/libs/jquery-sparkline-2.1.2/jquery.sparkline.css" rel="stylesheet" />
  <script src="LiverRisk_val_files/libs/jquery-sparkline-2.1.2/jquery.sparkline.js"></script>
  <script src="LiverRisk_val_files/libs/sparkline-binding-2.0/sparkline.js"></script>
  <script src="LiverRisk_val_files/libs/plotly-binding-4.10.4/plotly.js"></script>
  <script src="LiverRisk_val_files/libs/typedarray-0.1/typedarray.min.js"></script>
  <link href="LiverRisk_val_files/libs/crosstalk-1.2.1/css/crosstalk.min.css" rel="stylesheet" />
  <script src="LiverRisk_val_files/libs/crosstalk-1.2.1/js/crosstalk.min.js"></script>
  <link href="LiverRisk_val_files/libs/plotly-htmlwidgets-css-2.11.1/plotly-htmlwidgets.css" rel="stylesheet" />
  <script src="LiverRisk_val_files/libs/plotly-main-2.11.1/plotly-latest.min.js"></script>
title: LiverRisk validation in an international VCTE cohort of patients
  with MASLD
toc-title: Table of contents
---

<script src="LiverRisk_val_files/libs/htmlwidgets-1.6.4/htmlwidgets.js"></script>
<script src="LiverRisk_val_files/libs/jquery-3.5.1/jquery.min.js"></script>
<link href="LiverRisk_val_files/libs/jquery-sparkline-2.1.2/jquery.sparkline.css" rel="stylesheet" />
<script src="LiverRisk_val_files/libs/jquery-sparkline-2.1.2/jquery.sparkline.js"></script>
<script src="LiverRisk_val_files/libs/sparkline-binding-2.0/sparkline.js"></script>
<script src="LiverRisk_val_files/libs/plotly-binding-4.10.4/plotly.js"></script>
<script src="LiverRisk_val_files/libs/typedarray-0.1/typedarray.min.js"></script>
<link href="LiverRisk_val_files/libs/crosstalk-1.2.1/css/crosstalk.min.css" rel="stylesheet" />
<script src="LiverRisk_val_files/libs/crosstalk-1.2.1/js/crosstalk.min.js"></script>
<link href="LiverRisk_val_files/libs/plotly-htmlwidgets-css-2.11.1/plotly-htmlwidgets.css" rel="stylesheet" />
<script src="LiverRisk_val_files/libs/plotly-main-2.11.1/plotly-latest.min.js"></script>


-   [Introduction](#introduction){#toc-introduction}
-   [Describe the data](#describe-the-data){#toc-describe-the-data}
-   [Inspect missing
    data](#inspect-missing-data){#toc-inspect-missing-data}
    -   [Impute missing predictor values using `aregImpute` and
        calculate the average LiverRisk per patient over imputations.
        For later comparison, also calculate
        FIB-4.](#impute-missing-predictor-values-using-aregimpute-and-calculate-the-average-liverrisk-per-patient-over-imputations.-for-later-comparison-also-calculate-fib-4.){#toc-impute-missing-predictor-values-using-aregimpute-and-calculate-the-average-liverrisk-per-patient-over-imputations.-for-later-comparison-also-calculate-fib-4.}
-   [Results](#results){#toc-results}

# Introduction

Despite the authors not publishing the formula for LiverRisk, it can
nonetheless be reverse engineered from its online calculator
(liverriskscore.com). For this study we use the deduced LiverRisk
formula: $$
\begin{align*}
    LiverRisk =\quad&
3.75 + 0.265[\textrm{sex=male}] +0.0147\:[\textrm{age}] \\
&+0.47\:[\textrm{glucose (mmol/L)}] -0.357\:[\textrm{cholesterol (mmol/L)}] \\
&+0.021\:[\textrm{AST (IU/L)}] +0.00178\:[\textrm{ALT (IU/L)}] \\
& +0.0164\:[\textrm{GGT (IU/L)}] -0.00495\:[\textrm{platelets (x1000/microL)}] \\
\end{align*}
$$ which is accurate up to possible rounding errors in the calculator
output.

::: cell
`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
ex_data <- as.data.frame(matrix(c(
  "female", 55, 5.75, 5.39, 25, 26, 46, 245, 5.45,
  "male"  , 55, 5.75, 5.39, 25, 26, 46, 245, 5.72,
  "female", 19, 5.75, 5.39, 25, 26, 46, 245, 4.93,
  "female", 92, 5.75, 5.39, 25, 26, 46, 245, 6.00,
  "female", 55, 2.28, 5.39, 25, 26, 46, 245, 3.82,
  "female", 55, 25.5, 5.39, 25, 26, 46, 245, 14.74,
  "male"  , 55, 5.75, 1.60, 25, 26, 46, 245, 7.07,
  "male"  , 55, 5.75, 11.1, 25, 26, 46, 245, 3.68,
  "male"  , 55, 5.75, 5.39, 07, 26, 46, 245, 5.34,
  "female", 55, 5.75, 5.39, 331, 26, 46, 245, 11.87,
  "female", 55, 5.75, 5.39, 25, 3  , 46, 245, 5.41,
  "male"  , 55, 5.75, 5.39, 25, 387, 46, 245, 6.36,
  "female", 55, 5.75, 5.39, 25, 26, 4, 245, 4.76,
  "female", 55, 5.75, 5.39, 25, 26, 950, 245, 20.32,
  "male"  , 55, 5.75, 5.39, 25, 26, 46, 50, 6.68,
  "female", 55, 5.75, 5.39, 25, 26, 46, 716, 3.12), , 9, T))
colnames(ex_data) <- c("sex", "age", "gluc", "chol", "ast", "alt", "ggt", "plat", "liverrisk")

for (i in 2:9) {
  ex_data[, i] <- as.numeric(unlist(ex_data[, i]))
}

lr <- lm(liverrisk ~ sex + age + gluc + chol + ast + alt + ggt + plat, ex_data)
summary(lr)
```

```{=html}
</details>
```
::: {.cell-output .cell-output-stdout}

    Call:
    lm(formula = liverrisk ~ sex + age + gluc + chol + ast + alt + 
        ggt + plat, data = ex_data)

    Residuals:
           Min         1Q     Median         3Q        Max 
    -0.0028860 -0.0013506 -0.0001117  0.0001260  0.0048499 

    Coefficients:
                  Estimate Std. Error t value Pr(>|t|)    
    (Intercept)  3.752e+00  5.426e-03   691.5  < 2e-16 ***
    sexmale      2.645e-01  2.119e-03   124.8 5.58e-13 ***
    age          1.466e-02  6.535e-05   224.3 9.24e-15 ***
    gluc         4.702e-01  1.765e-04  2663.9  < 2e-16 ***
    chol        -3.568e-01  4.963e-04  -719.0  < 2e-16 ***
    ast          2.097e-02  1.183e-05  1772.9  < 2e-16 ***
    alt          1.780e-03  1.027e-05   173.3 5.61e-14 ***
    ggt          1.645e-02  3.978e-06  4133.8  < 2e-16 ***
    plat        -4.951e-03  7.166e-06  -690.9  < 2e-16 ***
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Residual standard error: 0.003374 on 7 degrees of freedom
    Multiple R-squared:      1, Adjusted R-squared:      1 
    F-statistic: 3.478e+06 on 8 and 7 DF,  p-value: < 2.2e-16
:::

`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
# Prediction function for generating LiverRisk values
liverrisk <- function(newdata) {
  M <- model.matrix.lm(~ sex + age + gluc + chol + ast + alt + ggt + plat, 
                    data = newdata, na.action = "na.pass")
  c(M %*% lr$coefficients)
}
liverrisk(ex_data)
```

```{=html}
</details>
```
::: {.cell-output .cell-output-stdout}
     [1]  5.452886  5.717423  4.925150  5.995281  3.821140 14.740200  7.069831
     [8]  3.679888  5.339945 11.869997  5.411938  6.360123  4.762153 20.320100
    [15]  6.682789  3.121155
:::
:::

::: cell
`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
library(rms)
library(tidyverse)
library(qreport)
options(prType="html")
knitr::set_alias(w = 'fig.width', h = 'fig.height', cap = 'fig.cap', scap ='fig.scap')
sparkline::sparkline(0)
```

```{=html}
</details>
```
::: cell-output-display
```{=html}
<span id="htmlwidget-0a22da7f68144b1bc177" class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-0a22da7f68144b1bc177">{"x":{"values":0,"options":{"height":20,"width":60},"width":60,"height":20},"evals":[],"jsHooks":[]}</script>
```
:::

`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
lsm_max <- 75

vcte <- readxl::read_xlsx("VCTE-Prognosis Database 230731.xlsx")

rem1 <- as.logical(rowSums(sapply(2:8, function(k) grepl(paste0("(", k, ")"), colnames(vcte), fixed = TRUE))))
rem2 <- as.logical(rowSums(sapply(2:8, function(k) grepl(paste0("Date of ", k), colnames(vcte), fixed = TRUE))))

vcte0 <- vcte[, -which(as.logical(rem1 + rem2))] %>% 
         select(id = "VCTE-ID", 
                Center, 
                sex = "Sex (1=male, 0=female)", 
                birth_year = "Year of birth (YYYY)", 
                diabetes0 = "Diabetes at baseline (1=yes, 0=no)", 
                hypertension0 = "Hypertension at baseline (1=yes, 0=no)",
                biopsy_date = "Date of liver biopsy closest to baseline VCTE (DD/MM/YYYY)",
                fstage = "Fibrosis stage NASH CRN (0-4)",
                vcte_date = "Date of baseline VCTE (DD/MM/YYYY)",
                age = "Age at baseline VCTE (skip if year of birth provided)",
                probe = "Probe (1) (1=M, 2=XL)",
                lsm = "LSM (1)",
                iqr = "IQR of LSM (1)",
                cap = "CAP (1)",
                iqr_cap = "IQR of CAP (1)",
                baseline_date = "Date of baseline clinical and labs (DD/MM/YYYY)",
                weight = "Body weight (1) (kg)" ,
                height = "Body height (1) (cm)",
                albumin = "Albumin (1) (g/L)",
                bilirubin = "Bilirubin (1) (µmol/L)",
                alt = "ALT (1) (U/L)",
                ast = "AST (1) (U/L)",
                ggt = "GGT (1) (U/L)",
                creatin = "Creatinine (1) (µmol/L)",
                gluc = "Fasting glucose (1) (mmol/L)",
                hba1c = "HbA1c (1) (%)",
                chol = "Total cholesterol (1) (mmol/L)",
                hdl = "HDL (1) (mmol/L)",
                ldl = "LDL (1) (mmol/L)",
                trig = "Triglycerides (1) (mmol/L)",
                plat = "Platelet (1) (x10e9/L)",
                end_date = "Date of last follow-up (DD/MM/YYYY)",
                hcc = "HCC (1=yes, 0=no)",
                hcc_date = "Date of HCC (DD/MM/YYYY)",
                asc =  "Ascites (1=yes, 0=no)",
                asc_date = "Date of ascites (DD/MM/YYYY)",
                sbp = "Spontaneous bacterial peritonitis (1=yes, 0=no)",
                sbp_date = "Date of SBP (DD/MM/YYYY)",
                var = "Variceal haemorrhage (1=yes, 0=no)",
                var_date = "Date of variceal haemorrhage (DD/MM/YYYY)",
                henc = "Hepatic encephalopathy (1=yes, 0=no)",
                henc_date = "Date of hepatic encephalopathy (DD/MM/YYYY)",
                hrs =  "Hepatorenal syndrome (1=yes, 0=no)",
                ltx = "Liver transplantation (1=yes, 0=no)",
                death = "Death (1=yes, 0=no)",
                death_date = "Date of death (DD/MM/YYYY)",
                death_cause = "Cause of death (1=liver-related, 2=cardiovascular, 3=cancers other than HCC, 4=others)",
                decomp = "Decompensation (1=yes, 0=no)",
                decomp_date = "Date of decompensation",
                decomp_type = "Type of decompensation",
                lre = "LRE (1=yes, 0=no)",
                lre_date = "Date of LRE",
                lre_type = "Type of LRE"
                ) %>% 
  filter(age >= 18, age <= 80) %>%
  mutate(biopsy_date = as.Date(biopsy_date),
         vcte_date = as.Date(vcte_date),
         baseline_date = as.Date(baseline_date),
         end_date = as.Date(end_date),
         lre_date = as.Date(lre_date),
         lsm_cat = cut2(lsm, c(8, 12)), 
         sex = ifelse(sex == 0, "female", "male"), 
         death_cause = factor(death_cause, 1:4, labels = c("liver", "cv", "nlcancer", "other")),
         death_time = as.numeric(as.Date(death_date) - as.Date(baseline_date))/365.25,
         lre_time = as.numeric(as.Date(lre_date) - as.Date(baseline_date))/365.25,
         end_time = as.numeric(as.Date(end_date) - as.Date(baseline_date))/365.25, 
         year = as.numeric(substr(vcte_date, 1, 4)) 
         )  %>%
  arrange(id) %>%
  filter(#abs(as.numeric(as.Date(biopsy_date) - as.Date(baseline_date))) <= 30,
               abs(as.numeric(as.Date(vcte_date) - as.Date(baseline_date))) <= 30,
               is.na(lre_time) | (lre_time > 30/365.25)) %>% 
  mutate(event = ifelse(!is.na(lre_time), 1, ifelse(!is.na(death_time), 2, 0)),
  time = ifelse(event == 1, lre_time, ifelse(event == 2, death_time, end_time)),
  event = factor(event, 0:2),
               subs = abs(as.numeric(as.Date(biopsy_date) - as.Date(baseline_date))) <= 30,
               fstage = factor(fstage, 0:4)#, c("Censored", "Liver", "Death"))
  ) %>% select(id, sex, age, ast, alt, ggt, gluc, chol, plat, lsm, iqr, cap, iqr_cap)

{
  label(vcte0$id) <- "Study ID"
  label(vcte0$sex) <- "Sex (male = 1)"
  label(vcte0$lsm) <- "Liver Stiffness Measure (kPa)" 
  label(vcte0$iqr) <- "Inter-quartile range of LSM" 
  label(vcte0$cap) <- "Controlled attenuation parameter (CAP)"
  label(vcte0$age) <- "Age"
  label(vcte0$iqr_cap) <- "Inter-quartile range of CAP"
  label(vcte0$alt) <- "ALT"
  label(vcte0$ast) <- "AST"
  label(vcte0$ggt) <- "GGT"
  label(vcte0$gluc) <- "Glucose"
  label(vcte0$chol) <- "Cholesterol"
  label(vcte0$plat) <- "Platelets"
}
```

```{=html}
</details>
```
:::

# Describe the data

`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
maketabs(print(describe(vcte0), 'both'), initblank = TRUE, 
         cwidth='column-screen-inset-shaded')
```

```{=html}
</details>
```
::: {.panel-tabset .column-screen-inset-shaded}
## 

## Continuous

<div>

```{=html}
<div id="qnwtopvoqu" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#qnwtopvoqu table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#qnwtopvoqu thead, #qnwtopvoqu tbody, #qnwtopvoqu tfoot, #qnwtopvoqu tr, #qnwtopvoqu td, #qnwtopvoqu th {
  border-style: none;
}

#qnwtopvoqu p {
  margin: 0;
  padding: 0;
}

#qnwtopvoqu .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#qnwtopvoqu .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#qnwtopvoqu .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#qnwtopvoqu .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#qnwtopvoqu .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#qnwtopvoqu .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qnwtopvoqu .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#qnwtopvoqu .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#qnwtopvoqu .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#qnwtopvoqu .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#qnwtopvoqu .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#qnwtopvoqu .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#qnwtopvoqu .gt_spanner_row {
  border-bottom-style: hidden;
}

#qnwtopvoqu .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#qnwtopvoqu .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#qnwtopvoqu .gt_from_md > :first-child {
  margin-top: 0;
}

#qnwtopvoqu .gt_from_md > :last-child {
  margin-bottom: 0;
}

#qnwtopvoqu .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#qnwtopvoqu .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#qnwtopvoqu .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#qnwtopvoqu .gt_row_group_first td {
  border-top-width: 2px;
}

#qnwtopvoqu .gt_row_group_first th {
  border-top-width: 2px;
}

#qnwtopvoqu .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qnwtopvoqu .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#qnwtopvoqu .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#qnwtopvoqu .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qnwtopvoqu .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#qnwtopvoqu .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#qnwtopvoqu .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#qnwtopvoqu .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#qnwtopvoqu .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#qnwtopvoqu .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#qnwtopvoqu .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qnwtopvoqu .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#qnwtopvoqu .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#qnwtopvoqu .gt_left {
  text-align: left;
}

#qnwtopvoqu .gt_center {
  text-align: center;
}

#qnwtopvoqu .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#qnwtopvoqu .gt_font_normal {
  font-weight: normal;
}

#qnwtopvoqu .gt_font_bold {
  font-weight: bold;
}

#qnwtopvoqu .gt_font_italic {
  font-style: italic;
}

#qnwtopvoqu .gt_super {
  font-size: 65%;
}

#qnwtopvoqu .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#qnwtopvoqu .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#qnwtopvoqu .gt_indent_1 {
  text-indent: 5px;
}

#qnwtopvoqu .gt_indent_2 {
  text-indent: 10px;
}

#qnwtopvoqu .gt_indent_3 {
  text-indent: 15px;
}

#qnwtopvoqu .gt_indent_4 {
  text-indent: 20px;
}

#qnwtopvoqu .gt_indent_5 {
  text-indent: 25px;
}
</style>
```
<table class="gt_table" data-quarto-postprocess="true"
style="table-layout: fixed;" data-quarto-disable-processing="false"
data-quarto-bootstrap="false">
<thead>
<tr class="header gt_heading">
<th colspan="10"
class="gt_heading gt_title gt_font_normal"><strong><code>vcte0</code>
Descriptives</strong></th>
</tr>
<tr class="odd gt_heading">
<th colspan="10"
class="gt_heading gt_subtitle gt_font_normal gt_bottom_border">11
Continous Variables of 13 Variables, 15984 Observations</th>
</tr>
<tr class="header gt_col_headings">
<th id="Variable"
class="gt_col_heading gt_columns_bottom_border gt_left"
data-quarto-table-cell-role="th" scope="col">Variable</th>
<th id="Label" class="gt_col_heading gt_columns_bottom_border gt_left"
data-quarto-table-cell-role="th" scope="col">Label</th>
<th id="n" class="gt_col_heading gt_columns_bottom_border gt_right"
style="text-align: center;" data-quarto-table-cell-role="th"
scope="col">n</th>
<th id="Missing"
class="gt_col_heading gt_columns_bottom_border gt_right"
style="text-align: center;" data-quarto-table-cell-role="th"
scope="col">Missing</th>
<th id="Distinct"
class="gt_col_heading gt_columns_bottom_border gt_right"
style="text-align: center;" data-quarto-table-cell-role="th"
scope="col">Distinct</th>
<th id="Info" class="gt_col_heading gt_columns_bottom_border gt_right"
style="text-align: center;" data-quarto-table-cell-role="th"
scope="col">Info</th>
<th id="Mean" class="gt_col_heading gt_columns_bottom_border gt_right"
style="text-align: center;" data-quarto-table-cell-role="th"
scope="col">Mean</th>
<th
id="Gini &lt;span style=&quot;text-decoration: overline&quot;&gt;|Δ|&lt;/span&gt;"
class="gt_col_heading gt_columns_bottom_border gt_right"
style="text-align: center;" data-quarto-table-cell-role="th"
scope="col">Gini <span style="text-decoration: overline">|Δ|</span></th>
<th id=" " class="gt_col_heading gt_columns_bottom_border gt_right"
data-quarto-table-cell-role="th" scope="col"></th>
<th
id="Quantiles&lt;br&gt;&lt;font size=&quot;1&quot;&gt; .05 &lt;/font&gt;&lt;font size=&quot;2&quot;&gt; .10 &lt;/font&gt;&lt;font size=&quot;3&quot;&gt; .25 &lt;/font&gt;&lt;font size=&quot;4&quot;&gt;&lt;strong&gt; .50 &lt;/strong&gt;&lt;/font&gt;&lt;font size=&quot;3&quot;&gt; .75 &lt;/font&gt;&lt;font size=&quot;2&quot;&gt; .90 &lt;/font&gt;&lt;font size=&quot;1&quot;&gt; .95 &lt;/font&gt;"
class="gt_col_heading gt_columns_bottom_border gt_center"
data-quarto-table-cell-role="th" scope="col">Quantiles<br />
<font size="1"> .05 </font><font size="2"> .10 </font><font size="3">
.25 </font><font size="4"> <strong>.50</strong> </font><font size="3">
.75 </font><font size="2"> .90 </font><font size="1"> .95 </font></th>
</tr>
</thead>
<tbody class="gt_table_body">
<tr class="odd">
<td class="gt_row gt_left" headers="Variable">age</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">Age</td>
<td class="gt_row gt_right" headers="n">15984</td>
<td class="gt_row gt_right" headers="Missing">0</td>
<td class="gt_row gt_right" headers="Distinct">63</td>
<td class="gt_row gt_right" headers="Info">0.999</td>
<td class="gt_row gt_right" headers="Mean">52.25</td>
<td class="gt_row gt_right" headers="Gmd">15.14</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-2c9190820bd01b764c87"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-2c9190820bd01b764c87">{"x":{"values":[176,89,67,64,82,81,75,111,89,113,127,143,148,132,154,145,205,211,229,213,242,252,261,267,268,315,331,317,392,367,398,394,420,431,489,524,510,520,525,527,480,459,430,432,396,424,388,357,305,286,248,254,172,177,153,145,235,239],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:19<br>Bin: [18, 20)<br>Observed:<br>18 (79)<br>19 (97)<br>Proportion:0.011 (n=176)<br>Cumulative:0.011\",\"Bin: [20, 21)<br>Observed:<br>20 (89)<br>Proportion:0.0056 (n=89)<br>Cumulative:0.0166\",\"Bin: [21, 22)<br>Observed:<br>21 (67)<br>Proportion:0.0042 (n=67)<br>Cumulative:0.0208\",\"Bin: [22, 23)<br>Observed:<br>22 (64)<br>Proportion:0.004 (n=64)<br>Cumulative:0.0248\",\"Bin: [23, 24)<br>Observed:<br>23 (82)<br>Proportion:0.0051 (n=82)<br>Cumulative:0.0299\",\"Bin: [24, 25)<br>Observed:<br>24 (81)<br>Proportion:0.0051 (n=81)<br>Cumulative:0.035\",\"Bin: [25, 26)<br>Observed:<br>25 (75)<br>Proportion:0.0047 (n=75)<br>Cumulative:0.0397\",\"Bin: [26, 27)<br>Observed:<br>26 (111)<br>Proportion:0.0069 (n=111)<br>Cumulative:0.0466\",\"Bin: [27, 28)<br>Observed:<br>27 (89)<br>Proportion:0.0056 (n=89)<br>Cumulative:0.0522\",\"Bin: [28, 29)<br>Observed:<br>28 (113)<br>Proportion:0.0071 (n=113)<br>Cumulative:0.0592\",\"Bin: [29, 30)<br>Observed:<br>29 (127)<br>Proportion:0.0079 (n=127)<br>Cumulative:0.0672\",\"Bin: [30, 31)<br>Observed:<br>30 (143)<br>Proportion:0.0089 (n=143)<br>Cumulative:0.0761\",\"Bin: [31, 32)<br>Observed:<br>31 (148)<br>Proportion:0.0093 (n=148)<br>Cumulative:0.0854\",\"Bin: [32, 33)<br>Observed:<br>32 (132)<br>Proportion:0.0083 (n=132)<br>Cumulative:0.0937\",\"Bin: [33, 34)<br>Observed:<br>33 (154)<br>Proportion:0.0096 (n=154)<br>Cumulative:0.1033\",\"Bin: [34, 35)<br>Observed:<br>34 (145)<br>Proportion:0.0091 (n=145)<br>Cumulative:0.1124\",\"Bin: [35, 36)<br>Observed:<br>35 (205)<br>Proportion:0.0128 (n=205)<br>Cumulative:0.1252\",\"Bin: [36, 37)<br>Observed:<br>36 (211)<br>Proportion:0.0132 (n=211)<br>Cumulative:0.1384\",\"Bin: [37, 38)<br>Observed:<br>37 (229)<br>Proportion:0.0143 (n=229)<br>Cumulative:0.1527\",\"Bin: [38, 39)<br>Observed:<br>38 (213)<br>Proportion:0.0133 (n=213)<br>Cumulative:0.166\",\"Bin: [39, 40)<br>Observed:<br>39 (242)<br>Proportion:0.0151 (n=242)<br>Cumulative:0.1812\",\"Bin: [40, 41)<br>Observed:<br>40 (252)<br>Proportion:0.0158 (n=252)<br>Cumulative:0.1969\",\"Bin: [41, 42)<br>Observed:<br>41 (261)<br>Proportion:0.0163 (n=261)<br>Cumulative:0.2133\",\"Bin: [42, 43)<br>Observed:<br>42 (267)<br>Proportion:0.0167 (n=267)<br>Cumulative:0.23\",\"Bin: [43, 44)<br>Observed:<br>43 (268)<br>Proportion:0.0168 (n=268)<br>Cumulative:0.2467\",\"Bin: [44, 45)<br>Observed:<br>44 (315)<br>Proportion:0.0197 (n=315)<br>Cumulative:0.2665\",\"Bin: [45, 46)<br>Observed:<br>45 (331)<br>Proportion:0.0207 (n=331)<br>Cumulative:0.2872\",\"Bin: [46, 47)<br>Observed:<br>46 (317)<br>Proportion:0.0198 (n=317)<br>Cumulative:0.307\",\"Bin: [47, 48)<br>Observed:<br>47 (392)<br>Proportion:0.0245 (n=392)<br>Cumulative:0.3315\",\"Bin: [48, 49)<br>Observed:<br>48 (367)<br>Proportion:0.023 (n=367)<br>Cumulative:0.3545\",\"Bin: [49, 50)<br>Observed:<br>49 (398)<br>Proportion:0.0249 (n=398)<br>Cumulative:0.3794\",\"Bin: [50, 51)<br>Observed:<br>50 (394)<br>Proportion:0.0246 (n=394)<br>Cumulative:0.404\",\"Bin: [51, 52)<br>Observed:<br>51 (420)<br>Proportion:0.0263 (n=420)<br>Cumulative:0.4303\",\"Bin: [52, 53)<br>Observed:<br>52 (431)<br>Proportion:0.027 (n=431)<br>Cumulative:0.4573\",\"Bin: [53, 54)<br>Observed:<br>53 (489)<br>Proportion:0.0306 (n=489)<br>Cumulative:0.4879\",\"Bin: [54, 55)<br>Observed:<br>54 (524)<br>Proportion:0.0328 (n=524)<br>Cumulative:0.5206\",\"Bin: [55, 56)<br>Observed:<br>55 (510)<br>Proportion:0.0319 (n=510)<br>Cumulative:0.5526\",\"Bin: [56, 57)<br>Observed:<br>56 (520)<br>Proportion:0.0325 (n=520)<br>Cumulative:0.5851\",\"Bin: [57, 58)<br>Observed:<br>57 (525)<br>Proportion:0.0328 (n=525)<br>Cumulative:0.6179\",\"Bin: [58, 59)<br>Observed:<br>58 (527)<br>Proportion:0.033 (n=527)<br>Cumulative:0.6509\",\"Bin: [59, 60)<br>Observed:<br>59 (480)<br>Proportion:0.03 (n=480)<br>Cumulative:0.6809\",\"Bin: [60, 61)<br>Observed:<br>60 (459)<br>Proportion:0.0287 (n=459)<br>Cumulative:0.7096\",\"Bin: [61, 62)<br>Observed:<br>61 (430)<br>Proportion:0.0269 (n=430)<br>Cumulative:0.7365\",\"Bin: [62, 63)<br>Observed:<br>62 (432)<br>Proportion:0.027 (n=432)<br>Cumulative:0.7636\",\"Bin: [63, 64)<br>Observed:<br>63 (396)<br>Proportion:0.0248 (n=396)<br>Cumulative:0.7884\",\"Bin: [64, 65)<br>Observed:<br>64 (424)<br>Proportion:0.0265 (n=424)<br>Cumulative:0.8149\",\"Bin: [65, 66)<br>Observed:<br>65 (388)<br>Proportion:0.0243 (n=388)<br>Cumulative:0.8392\",\"Bin: [66, 67)<br>Observed:<br>66 (357)<br>Proportion:0.0223 (n=357)<br>Cumulative:0.8615\",\"Bin: [67, 68)<br>Observed:<br>67 (305)<br>Proportion:0.0191 (n=305)<br>Cumulative:0.8806\",\"Bin: [68, 69)<br>Observed:<br>68 (286)<br>Proportion:0.0179 (n=286)<br>Cumulative:0.8985\",\"Bin: [69, 70)<br>Observed:<br>69 (248)<br>Proportion:0.0155 (n=248)<br>Cumulative:0.914\",\"Bin: [70, 71)<br>Observed:<br>70 (254)<br>Proportion:0.0159 (n=254)<br>Cumulative:0.9299\",\"Bin: [71, 72)<br>Observed:<br>71 (172)<br>Proportion:0.0108 (n=172)<br>Cumulative:0.9406\",\"Bin: [72, 73)<br>Observed:<br>72 (177)<br>Proportion:0.0111 (n=177)<br>Cumulative:0.9517\",\"Bin: [73, 74)<br>Observed:<br>73 (153)<br>Proportion:0.0096 (n=153)<br>Cumulative:0.9613\",\"Bin: [74, 75)<br>Observed:<br>74 (145)<br>Proportion:0.0091 (n=145)<br>Cumulative:0.9703\",\"Bin: [75, 76]<br>Observed:<br>75 (123)<br>76 (112)<br>Proportion:0.0147 (n=235)<br>Cumulative:0.985\",\"Q<sub>0.99<\\/sub>:77<br>Bin: (76, 80]<br>Observed:<br>77 (80)<br>78 (58)<br>79 (56)<br>80 (45)<br>Proportion:0.015 (n=239)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 27
</font><font size="2"> 33 </font><font size="3"> 44
</font><font size="4"> <strong>54</strong> </font><font size="3"> 62
</font><font size="2"> 69 </font><font size="1"> 72 </font></td>
</tr>
<tr class="even">
<td class="gt_row gt_left" headers="Variable">ast</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">AST</td>
<td class="gt_row gt_right" headers="n">12652</td>
<td class="gt_row gt_right" headers="Missing">3332</td>
<td class="gt_row gt_right" headers="Distinct">509</td>
<td class="gt_row gt_right" headers="Info">1.000</td>
<td class="gt_row gt_right" headers="Mean">40.2</td>
<td class="gt_row gt_right" headers="Gmd">26.78</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-159152007ed94043ebb8"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-159152007ed94043ebb8">{"x":{"values":[178,283,583,781,853,877,807,822,713,653,540,492,441,378,351,310,311,283,222,235,189,171,165,131,148,120,111,98,90,89,81,69,61,71,54,62,49,44,43,46,37,36,36,29,32,24,35,23,18,12,17,17,16,13,13,9,18,12,10,3,6,13,9,13,6,5,7,7,8,8,5,6,5,12,127],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:13<br>Bin: [1, 14)<br>Observed:<br>1 - 13.8 (12 distinct)<br>Proportion:0.0141 (n=178)<br>Cumulative:0.0141\",\"Bin: [14, 16)<br>Observed:<br>14 (118)<br>14.5<br>15 (161)<br>15.2941<br>15.7<br>15.8824<br>Proportion:0.0224 (n=283)<br>Cumulative:0.0364\",\"Bin: [16, 18)<br>Observed:<br>16 (256)<br>16.3<br>16.4706<br>16.6<br>17 (321)<br>17.0588 (2)<br>17.7<br>Proportion:0.0461 (n=583)<br>Cumulative:0.0825\",\"Bin: [18, 20)<br>Observed:<br>18 - 19.7 (12 distinct)<br>Proportion:0.0617 (n=781)<br>Cumulative:0.1442\",\"Bin: [20, 22)<br>Observed:<br>20 - 21.8 (16 distinct)<br>Proportion:0.0674 (n=853)<br>Cumulative:0.2117\",\"Bin: [22, 24)<br>Observed:<br>22 - 23.9 (16 distinct)<br>Proportion:0.0693 (n=877)<br>Cumulative:0.281\",\"Bin: [24, 26)<br>Observed:<br>24 - 25.8824 (17 distinct)<br>Proportion:0.0638 (n=807)<br>Cumulative:0.3448\",\"Bin: [26, 28)<br>Observed:<br>26 - 27.7 (16 distinct)<br>Proportion:0.065 (n=822)<br>Cumulative:0.4097\",\"Bin: [28, 30)<br>Observed:<br>28 - 29.9 (14 distinct)<br>Proportion:0.0564 (n=713)<br>Cumulative:0.4661\",\"Bin: [30, 32)<br>Observed:<br>30 - 31.9 (12 distinct)<br>Proportion:0.0516 (n=653)<br>Cumulative:0.5177\",\"Bin: [32, 34)<br>Observed:<br>32 - 33.9 (11 distinct)<br>Proportion:0.0427 (n=540)<br>Cumulative:0.5604\",\"Bin: [34, 36)<br>Observed:<br>34 - 35.9 (13 distinct)<br>Proportion:0.0389 (n=492)<br>Cumulative:0.5993\",\"Bin: [36, 38)<br>Observed:<br>36 (218)<br>36.2<br>36.4706 (2)<br>37 (210)<br>37.0588 (3)<br>37.1<br>37.3<br>37.7 (3)<br>37.8<br>37.9<br>Proportion:0.0349 (n=441)<br>Cumulative:0.6341\",\"Bin: [38, 40)<br>Observed:<br>38 (182)<br>38.1<br>38.2<br>38.2353 (5)<br>38.8235 (4)<br>39 (182)<br>39.4<br>39.9 (2)<br>Proportion:0.0299 (n=378)<br>Cumulative:0.664\",\"Bin: [40, 42)<br>Observed:<br>40 (185)<br>40.3<br>40.5882 (3)<br>40.7 (3)<br>41 (151)<br>41.1765 (4)<br>41.2<br>41.7647 (3)<br>Proportion:0.0277 (n=351)<br>Cumulative:0.6917\",\"Bin: [42, 44)<br>Observed:<br>42 (146)<br>42.3<br>42.3529 (4)<br>42.4<br>42.6<br>42.9412 (5)<br>43 (143)<br>43.3<br>43.5<br>43.5294 (7)<br>Proportion:0.0245 (n=310)<br>Cumulative:0.7163\",\"Bin: [44, 46)<br>Observed:<br>44 - 45.8824 (12 distinct)<br>Proportion:0.0246 (n=311)<br>Cumulative:0.7408\",\"Bin: [46, 48)<br>Observed:<br>46 (135)<br>46.4706 (6)<br>46.9<br>47 (134)<br>47.0588<br>47.6471 (4)<br>47.7 (2)<br>Proportion:0.0224 (n=283)<br>Cumulative:0.7632\",\"Bin: [48, 50)<br>Observed:<br>48 (101)<br>48.2353<br>48.8235 (3)<br>49 (115)<br>49.3<br>49.4118<br>Proportion:0.0175 (n=222)<br>Cumulative:0.7807\",\"Bin: [50, 52)<br>Observed:<br>50 (115)<br>50.5882<br>50.8<br>51 (109)<br>51.1<br>51.1765 (3)<br>51.2<br>51.7647 (3)<br>51.9<br>Proportion:0.0186 (n=235)<br>Cumulative:0.7993\",\"Bin: [52, 54)<br>Observed:<br>52 (94)<br>52.3529 (3)<br>52.9412<br>53 (88)<br>53.5294 (2)<br>53.7<br>Proportion:0.0149 (n=189)<br>Cumulative:0.8143\",\"Bin: [54, 56)<br>Observed:<br>54 (79)<br>54.1176 (2)<br>54.5<br>54.7059 (2)<br>55 (79)<br>55.2941 (4)<br>55.5<br>55.8824 (3)<br>Proportion:0.0135 (n=171)<br>Cumulative:0.8278\",\"Bin: [56, 58)<br>Observed:<br>56 (75)<br>56.4706 (4)<br>56.5<br>56.7<br>57 (79)<br>57.0588<br>57.6471 (3)<br>57.8<br>Proportion:0.013 (n=165)<br>Cumulative:0.8408\",\"Bin: [58, 60)<br>Observed:<br>58 (59)<br>58.2353 (2)<br>58.5<br>58.8<br>58.8235 (2)<br>59 (64)<br>59.4118 (2)<br>Proportion:0.0104 (n=131)<br>Cumulative:0.8512\",\"Bin: [60, 62)<br>Observed:<br>60 (68)<br>60.5<br>60.5882<br>61 (75)<br>61.1<br>61.1765<br>61.7647<br>Proportion:0.0117 (n=148)<br>Cumulative:0.8629\",\"Bin: [62, 64)<br>Observed:<br>62 (63)<br>62.1<br>62.3529<br>62.9<br>62.9412<br>63 (53)<br>Proportion:0.0095 (n=120)<br>Cumulative:0.8724\",\"Bin: [64, 66)<br>Observed:<br>64 (58)<br>64.1176 (3)<br>65 (46)<br>65.2941<br>65.5<br>65.8<br>65.8824<br>Proportion:0.0088 (n=111)<br>Cumulative:0.8811\",\"Bin: [66, 68)<br>Observed:<br>66 (45)<br>66.4706<br>67 (47)<br>67.0588 (3)<br>67.6471<br>67.9<br>Proportion:0.0077 (n=98)<br>Cumulative:0.8889\",\"Bin: [68, 70)<br>Observed:<br>68 (42)<br>68.2353 (2)<br>68.8235 (2)<br>69 (41)<br>69.4118 (2)<br>69.8<br>Proportion:0.0071 (n=90)<br>Cumulative:0.896\",\"Bin: [70, 72)<br>Observed:<br>70 (40)<br>70.5882 (3)<br>71 (44)<br>71.1765 (2)<br>Proportion:0.007 (n=89)<br>Cumulative:0.903\",\"Bin: [72, 74)<br>Observed:<br>72 (43)<br>72.9412<br>73 (35)<br>73.2<br>73.5<br>Proportion:0.0064 (n=81)<br>Cumulative:0.9094\",\"Bin: [74, 76)<br>Observed:<br>74 (32)<br>74.7059 (3)<br>74.8<br>75 (29)<br>75.2941<br>75.3<br>75.8824<br>75.9<br>Proportion:0.0055 (n=69)<br>Cumulative:0.9149\",\"Bin: [76, 78)<br>Observed:<br>76 (29)<br>77 (32)<br>Proportion:0.0048 (n=61)<br>Cumulative:0.9197\",\"Bin: [78, 80)<br>Observed:<br>78 (47)<br>78.2353<br>79 (23)<br>Proportion:0.0056 (n=71)<br>Cumulative:0.9253\",\"Bin: [80, 82)<br>Observed:<br>80 (30)<br>80.5882<br>81 (23)<br>Proportion:0.0043 (n=54)<br>Cumulative:0.9296\",\"Bin: [82, 84)<br>Observed:<br>82 (26)<br>82.9412<br>83 (35)<br>Proportion:0.0049 (n=62)<br>Cumulative:0.9345\",\"Bin: [84, 86)<br>Observed:<br>84 (25)<br>84.1176 (2)<br>84.7059<br>84.9<br>85 (18)<br>85.2941 (2)<br>Proportion:0.0039 (n=49)<br>Cumulative:0.9383\",\"Bin: [86, 88)<br>Observed:<br>86 (20)<br>86.4706<br>86.7<br>87 (19)<br>87.0588<br>87.6471<br>87.7<br>Proportion:0.0035 (n=44)<br>Cumulative:0.9418\",\"Bin: [88, 90)<br>Observed:<br>88 (18)<br>88.8<br>88.9<br>89 (23)<br>Proportion:0.0034 (n=43)<br>Cumulative:0.9452\",\"Bin: [90, 92)<br>Observed:<br>90 (17)<br>90.8<br>91 (26)<br>91.7647 (2)<br>Proportion:0.0036 (n=46)<br>Cumulative:0.9489\",\"Bin: [92, 94)<br>Observed:<br>92 (13)<br>92.1<br>92.3529 (3)<br>93 (20)<br>Proportion:0.0029 (n=37)<br>Cumulative:0.9518\",\"Bin: [94, 96)<br>Observed:<br>94 (13)<br>94.3<br>94.7059 (2)<br>95 (20)<br>Proportion:0.0028 (n=36)<br>Cumulative:0.9546\",\"Bin: [96, 98)<br>Observed:<br>96 (15)<br>97 (21)<br>Proportion:0.0028 (n=36)<br>Cumulative:0.9575\",\"Bin: [98, 100)<br>Observed:<br>98 (12)<br>98.2353<br>99 (16)<br>Proportion:0.0023 (n=29)<br>Cumulative:0.9598\",\"Bin: [100, 102)<br>Observed:<br>100 (18)<br>100.7<br>101 (12)<br>101.176<br>Proportion:0.0025 (n=32)<br>Cumulative:0.9623\",\"Bin: [102, 104)<br>Observed:<br>102 (9)<br>103 (15)<br>Proportion:0.0019 (n=24)<br>Cumulative:0.9642\",\"Bin: [104, 106)<br>Observed:<br>104 (19)<br>104.706<br>105 (14)<br>105.294<br>Proportion:0.0028 (n=35)<br>Cumulative:0.967\",\"Bin: [106, 108)<br>Observed:<br>106 (11)<br>107 (12)<br>Proportion:0.0018 (n=23)<br>Cumulative:0.9688\",\"Bin: [108, 110)<br>Observed:<br>108 (9)<br>109 (8)<br>109.412<br>Proportion:0.0014 (n=18)<br>Cumulative:0.9702\",\"Bin: [110, 112)<br>Observed:<br>110 (4)<br>110.588<br>111 (6)<br>111.8<br>Proportion:9e-04 (n=12)<br>Cumulative:0.9712\",\"Bin: [112, 114)<br>Observed:<br>112 (10)<br>113 (7)<br>Proportion:0.0013 (n=17)<br>Cumulative:0.9725\",\"Bin: [114, 116)<br>Observed:<br>114 (8)<br>115 (9)<br>Proportion:0.0013 (n=17)<br>Cumulative:0.9738\",\"Bin: [116, 118)<br>Observed:<br>116 (6)<br>117 (9)<br>117.2<br>Proportion:0.0013 (n=16)<br>Cumulative:0.9751\",\"Bin: [118, 120)<br>Observed:<br>118 (3)<br>118.824<br>119 (9)<br>Proportion:0.001 (n=13)<br>Cumulative:0.9761\",\"Bin: [120, 122)<br>Observed:<br>120 (8)<br>121 (5)<br>Proportion:0.001 (n=13)<br>Cumulative:0.9772\",\"Bin: [122, 124)<br>Observed:<br>122 (4)<br>123 (5)<br>Proportion:7e-04 (n=9)<br>Cumulative:0.9779\",\"Bin: [124, 126)<br>Observed:<br>124 (9)<br>125 (8)<br>125.882<br>Proportion:0.0014 (n=18)<br>Cumulative:0.9793\",\"Bin: [126, 128)<br>Observed:<br>126 (5)<br>127 (7)<br>Proportion:9e-04 (n=12)<br>Cumulative:0.9802\",\"Bin: [128, 130)<br>Observed:<br>128 (6)<br>129 (4)<br>Proportion:8e-04 (n=10)<br>Cumulative:0.981\",\"Bin: [130, 132)<br>Observed:<br>130<br>131 (2)<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9813\",\"Bin: [132, 134)<br>Observed:<br>132 (3)<br>133 (3)<br>Proportion:5e-04 (n=6)<br>Cumulative:0.9817\",\"Bin: [134, 136)<br>Observed:<br>134 (6)<br>134.706<br>135 (6)<br>Proportion:0.001 (n=13)<br>Cumulative:0.9828\",\"Bin: [136, 138)<br>Observed:<br>136 (6)<br>137 (3)<br>Proportion:7e-04 (n=9)<br>Cumulative:0.9835\",\"Bin: [138, 140)<br>Observed:<br>138 (5)<br>139 (8)<br>Proportion:0.001 (n=13)<br>Cumulative:0.9845\",\"Bin: [140, 142)<br>Observed:<br>140 (4)<br>141 (2)<br>Proportion:5e-04 (n=6)<br>Cumulative:0.985\",\"Bin: [142, 144)<br>Observed:<br>142 (4)<br>143<br>Proportion:4e-04 (n=5)<br>Cumulative:0.9854\",\"Bin: [144, 146)<br>Observed:<br>144 (3)<br>144.706<br>145 (3)<br>Proportion:6e-04 (n=7)<br>Cumulative:0.9859\",\"Bin: [146, 148)<br>Observed:<br>146 (5)<br>147 (2)<br>Proportion:6e-04 (n=7)<br>Cumulative:0.9865\",\"Bin: [148, 150)<br>Observed:<br>148 (4)<br>149 (3)<br>149.7<br>Proportion:6e-04 (n=8)<br>Cumulative:0.9871\",\"Bin: [150, 152)<br>Observed:<br>150 (4)<br>151 (4)<br>Proportion:6e-04 (n=8)<br>Cumulative:0.9877\",\"Bin: [152, 154)<br>Observed:<br>152 (4)<br>153<br>Proportion:4e-04 (n=5)<br>Cumulative:0.9881\",\"Bin: [154, 156)<br>Observed:<br>154 (3)<br>155 (2)<br>155.882<br>Proportion:5e-04 (n=6)<br>Cumulative:0.9886\",\"Bin: [156, 158)<br>Observed:<br>156 (2)<br>157 (3)<br>Proportion:4e-04 (n=5)<br>Cumulative:0.989\",\"Bin: [158, 160]<br>Observed:<br>158 (6)<br>159 (2)<br>160 (4)<br>Proportion:9e-04 (n=12)<br>Cumulative:0.99\",\"Q<sub>0.99<\\/sub>:161<br>Bin: (160, 893]<br>Observed:<br>161 - 893 (89 distinct)<br>Proportion:0.01 (n=127)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 16.00
</font><font size="2"> 18.00 </font><font size="3"> 23.00
</font><font size="4"> <strong>31.00</strong> </font><font size="3">
46.00 </font><font size="2"> 71.00 </font><font size="1"> 92.21
</font></td>
</tr>
<tr class="odd">
<td class="gt_row gt_left" headers="Variable">alt</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">ALT</td>
<td class="gt_row gt_right" headers="n">14882</td>
<td class="gt_row gt_right" headers="Missing">1102</td>
<td class="gt_row gt_right" headers="Distinct">490</td>
<td class="gt_row gt_right" headers="Info">1.000</td>
<td class="gt_row gt_right" headers="Mean">49.79</td>
<td class="gt_row gt_right" headers="Gmd">39.89</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-8d2fe65c4a967f62da1a"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-8d2fe65c4a967f62da1a">{"x":{"values":[162,225,349,496,637,694,674,656,652,640,588,541,511,436,475,425,366,340,342,288,307,261,255,242,215,217,206,198,188,158,165,163,139,149,120,125,113,117,119,103,84,82,70,73,83,52,67,53,55,60,50,45,44,44,44,48,38,28,29,37,21,24,40,33,24,14,22,21,21,21,22,22,16,13,14,17,15,13,12,13,11,13,12,8,10,10,12,12,7,6,9,9,6,7,4,6,9,6,7,8,4,5,2,5,153],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:9<br>Bin: [2, 10)<br>Observed:<br>2<br>5 (8)<br>5.88235<br>6 (15)<br>7 (30)<br>8 (46)<br>9 (61)<br>Proportion:0.0109 (n=162)<br>Cumulative:0.0109\",\"Bin: [10, 12)<br>Observed:<br>10 (92)<br>11 (133)<br>Proportion:0.0151 (n=225)<br>Cumulative:0.026\",\"Bin: [12, 14)<br>Observed:<br>12 (142)<br>13 (205)<br>13.5294 (2)<br>Proportion:0.0235 (n=349)<br>Cumulative:0.0495\",\"Bin: [14, 16)<br>Observed:<br>14 (219)<br>15 (274)<br>15.2941 (2)<br>15.8824<br>Proportion:0.0333 (n=496)<br>Cumulative:0.0828\",\"Bin: [16, 18)<br>Observed:<br>16 (321)<br>17 (313)<br>17.0588<br>17.4<br>17.6471<br>Proportion:0.0428 (n=637)<br>Cumulative:0.1256\",\"Bin: [18, 20)<br>Observed:<br>18 (369)<br>18.3<br>18.8235 (2)<br>19 (322)<br>Proportion:0.0466 (n=694)<br>Cumulative:0.1722\",\"Bin: [20, 22)<br>Observed:<br>20 (325)<br>20.3<br>20.5<br>20.5882<br>21 (345)<br>21.1765<br>Proportion:0.0453 (n=674)<br>Cumulative:0.2175\",\"Bin: [22, 24)<br>Observed:<br>22 (314)<br>22.3529<br>23 (340)<br>23.5294<br>Proportion:0.0441 (n=656)<br>Cumulative:0.2616\",\"Bin: [24, 26)<br>Observed:<br>24 (340)<br>24.1176<br>24.7059<br>25 (304)<br>25.2941 (3)<br>25.8824 (3)<br>Proportion:0.0438 (n=652)<br>Cumulative:0.3054\",\"Bin: [26, 28)<br>Observed:<br>26 (336)<br>26.4706<br>27 (302)<br>27.6471<br>Proportion:0.043 (n=640)<br>Cumulative:0.3484\",\"Bin: [28, 30)<br>Observed:<br>28 (289)<br>28.2353 (3)<br>28.8235 (2)<br>29 (293)<br>29.4118<br>Proportion:0.0395 (n=588)<br>Cumulative:0.3879\",\"Bin: [30, 32)<br>Observed:<br>30 (268)<br>30.5882 (3)<br>31 (267)<br>31.1765 (2)<br>31.7647<br>Proportion:0.0364 (n=541)<br>Cumulative:0.4243\",\"Bin: [32, 34)<br>Observed:<br>32 (258)<br>32.3529 (5)<br>32.9412 (3)<br>33 (241)<br>33.5294 (4)<br>Proportion:0.0343 (n=511)<br>Cumulative:0.4586\",\"Bin: [34, 36)<br>Observed:<br>34 (207)<br>34.1176 (4)<br>34.7059<br>35 (221)<br>35.2941 (2)<br>35.8824<br>Proportion:0.0293 (n=436)<br>Cumulative:0.4879\",\"Bin: [36, 38)<br>Observed:<br>36 (228)<br>36.4706 (2)<br>36.8<br>37 (237)<br>37.0588 (4)<br>37.6471 (3)<br>Proportion:0.0319 (n=475)<br>Cumulative:0.5198\",\"Bin: [38, 40)<br>Observed:<br>38 (219)<br>38.2353 (4)<br>38.8235<br>39 (199)<br>39.4118 (2)<br>Proportion:0.0286 (n=425)<br>Cumulative:0.5484\",\"Bin: [40, 42)<br>Observed:<br>40 (188)<br>40.5<br>41 (172)<br>41.1765 (2)<br>41.7647 (3)<br>Proportion:0.0246 (n=366)<br>Cumulative:0.573\",\"Bin: [42, 44)<br>Observed:<br>42 (176)<br>42.3529 (2)<br>43 (160)<br>43.5294 (2)<br>Proportion:0.0228 (n=340)<br>Cumulative:0.5958\",\"Bin: [44, 46)<br>Observed:<br>44 (173)<br>44.1176<br>44.7059 (2)<br>45 (160)<br>45.2941 (2)<br>45.8824 (4)<br>Proportion:0.023 (n=342)<br>Cumulative:0.6188\",\"Bin: [46, 48)<br>Observed:<br>46 (142)<br>46.4706 (4)<br>46.9<br>47 (135)<br>47.0588 (5)<br>47.6471<br>Proportion:0.0194 (n=288)<br>Cumulative:0.6382\",\"Bin: [48, 50)<br>Observed:<br>48 (145)<br>48.2<br>48.2353 (3)<br>48.8235 (2)<br>49 (154)<br>49.3<br>49.4118<br>Proportion:0.0206 (n=307)<br>Cumulative:0.6588\",\"Bin: [50, 52)<br>Observed:<br>50 (128)<br>50.5882 (2)<br>51 (128)<br>51.1765 (2)<br>51.7647<br>Proportion:0.0175 (n=261)<br>Cumulative:0.6763\",\"Bin: [52, 54)<br>Observed:<br>52 (133)<br>52.3529 (2)<br>52.9412<br>53 (118)<br>53.5294<br>Proportion:0.0171 (n=255)<br>Cumulative:0.6935\",\"Bin: [54, 56)<br>Observed:<br>54 (118)<br>54.1176 (2)<br>54.7059 (2)<br>55 (117)<br>55.2941 (3)<br>Proportion:0.0163 (n=242)<br>Cumulative:0.7097\",\"Bin: [56, 58)<br>Observed:<br>56 (101)<br>56.4706 (2)<br>57 (108)<br>57.0588 (2)<br>57.6471 (2)<br>Proportion:0.0144 (n=215)<br>Cumulative:0.7242\",\"Bin: [58, 60)<br>Observed:<br>58 (121)<br>58.2353 (5)<br>59 (89)<br>59.4118 (2)<br>Proportion:0.0146 (n=217)<br>Cumulative:0.7387\",\"Bin: [60, 62)<br>Observed:<br>60 (105)<br>60.5882 (2)<br>61 (96)<br>61.1765 (2)<br>61.7647<br>Proportion:0.0138 (n=206)<br>Cumulative:0.7526\",\"Bin: [62, 64)<br>Observed:<br>62 (97)<br>62.3529<br>62.9412<br>63 (96)<br>63.5294 (2)<br>63.8<br>Proportion:0.0133 (n=198)<br>Cumulative:0.7659\",\"Bin: [64, 66)<br>Observed:<br>64 (87)<br>64.1176<br>64.7059 (4)<br>65 (94)<br>65.8824 (2)<br>Proportion:0.0126 (n=188)<br>Cumulative:0.7785\",\"Bin: [66, 68)<br>Observed:<br>66 (78)<br>67 (75)<br>67.0588<br>67.6471 (4)<br>Proportion:0.0106 (n=158)<br>Cumulative:0.7891\",\"Bin: [68, 70)<br>Observed:<br>68 (82)<br>68.2353 (3)<br>69 (79)<br>69.4118<br>Proportion:0.0111 (n=165)<br>Cumulative:0.8002\",\"Bin: [70, 72)<br>Observed:<br>70 (88)<br>70.5882 (2)<br>71 (71)<br>71.7647 (2)<br>Proportion:0.011 (n=163)<br>Cumulative:0.8112\",\"Bin: [72, 74)<br>Observed:<br>72 (63)<br>72.3529<br>72.9412 (4)<br>73 (70)<br>73.5294<br>Proportion:0.0093 (n=139)<br>Cumulative:0.8205\",\"Bin: [74, 76)<br>Observed:<br>74 (65)<br>74.1176 (2)<br>75 (79)<br>75.2941 (2)<br>75.8824<br>Proportion:0.01 (n=149)<br>Cumulative:0.8305\",\"Bin: [76, 78)<br>Observed:<br>76 (66)<br>77 (53)<br>77.0588<br>Proportion:0.0081 (n=120)<br>Cumulative:0.8386\",\"Bin: [78, 80)<br>Observed:<br>78 (66)<br>78.2353<br>78.8235<br>79 (57)<br>Proportion:0.0084 (n=125)<br>Cumulative:0.847\",\"Bin: [80, 82)<br>Observed:<br>80 (58)<br>80.5882<br>81 (52)<br>81.7647 (2)<br>Proportion:0.0076 (n=113)<br>Cumulative:0.8546\",\"Bin: [82, 84)<br>Observed:<br>82 (57)<br>82.3529<br>82.9412 (2)<br>83 (56)<br>83.5294<br>Proportion:0.0079 (n=117)<br>Cumulative:0.8625\",\"Bin: [84, 86)<br>Observed:<br>84 (57)<br>84.7059<br>85 (59)<br>85.8824 (2)<br>Proportion:0.008 (n=119)<br>Cumulative:0.8704\",\"Bin: [86, 88)<br>Observed:<br>86 (40)<br>86.4706 (5)<br>87 (57)<br>87.6471<br>Proportion:0.0069 (n=103)<br>Cumulative:0.8774\",\"Bin: [88, 90)<br>Observed:<br>88 (39)<br>88.2353<br>88.8235<br>89 (43)<br>Proportion:0.0056 (n=84)<br>Cumulative:0.883\",\"Bin: [90, 92)<br>Observed:<br>90 (43)<br>90.5882<br>91 (35)<br>91.1765 (2)<br>91.7647<br>Proportion:0.0055 (n=82)<br>Cumulative:0.8885\",\"Bin: [92, 94)<br>Observed:<br>92 (35)<br>92.3529<br>92.9412 (2)<br>93 (31)<br>93.5294<br>Proportion:0.0047 (n=70)<br>Cumulative:0.8932\",\"Bin: [94, 96)<br>Observed:<br>94 (40)<br>94.1176<br>94.7059<br>95 (29)<br>95.2941<br>95.8824<br>Proportion:0.0049 (n=73)<br>Cumulative:0.8981\",\"Bin: [96, 98)<br>Observed:<br>96 (39)<br>97 (42)<br>97.0588<br>97.6471<br>Proportion:0.0056 (n=83)<br>Cumulative:0.9037\",\"Bin: [98, 100)<br>Observed:<br>98 (21)<br>98.2353 (2)<br>99 (29)<br>Proportion:0.0035 (n=52)<br>Cumulative:0.9072\",\"Bin: [100, 102)<br>Observed:<br>100 (26)<br>101 (41)<br>Proportion:0.0045 (n=67)<br>Cumulative:0.9117\",\"Bin: [102, 104)<br>Observed:<br>102 (25)<br>102.941 (2)<br>103 (24)<br>103.529 (2)<br>Proportion:0.0036 (n=53)<br>Cumulative:0.9153\",\"Bin: [104, 106)<br>Observed:<br>104 (24)<br>105 (27)<br>105.294 (3)<br>105.6<br>Proportion:0.0037 (n=55)<br>Cumulative:0.919\",\"Bin: [106, 108)<br>Observed:<br>106 (34)<br>106.471<br>107 (25)<br>Proportion:0.004 (n=60)<br>Cumulative:0.923\",\"Bin: [108, 110)<br>Observed:<br>108 (19)<br>108.235<br>108.824 (3)<br>109 (27)<br>Proportion:0.0034 (n=50)<br>Cumulative:0.9264\",\"Bin: [110, 112)<br>Observed:<br>110 (22)<br>111 (20)<br>111.1<br>111.176<br>111.765<br>Proportion:0.003 (n=45)<br>Cumulative:0.9294\",\"Bin: [112, 114)<br>Observed:<br>112 (19)<br>112.353 (2)<br>112.941 (2)<br>113 (21)<br>Proportion:0.003 (n=44)<br>Cumulative:0.9323\",\"Bin: [114, 116)<br>Observed:<br>114 (21)<br>114.706<br>115 (21)<br>115.882<br>Proportion:0.003 (n=44)<br>Cumulative:0.9353\",\"Bin: [116, 118)<br>Observed:<br>116 (20)<br>116.471 (2)<br>117 (21)<br>117.647<br>Proportion:0.003 (n=44)<br>Cumulative:0.9382\",\"Bin: [118, 120)<br>Observed:<br>118 (25)<br>118.235<br>119 (21)<br>119.412<br>Proportion:0.0032 (n=48)<br>Cumulative:0.9415\",\"Bin: [120, 122)<br>Observed:<br>120 (16)<br>121 (20)<br>121.176<br>121.765<br>Proportion:0.0026 (n=38)<br>Cumulative:0.944\",\"Bin: [122, 124)<br>Observed:<br>122 (15)<br>122.353<br>122.941<br>123 (9)<br>123.529 (2)<br>Proportion:0.0019 (n=28)<br>Cumulative:0.9459\",\"Bin: [124, 126)<br>Observed:<br>124 (12)<br>124.118 (2)<br>125 (15)<br>Proportion:0.0019 (n=29)<br>Cumulative:0.9479\",\"Bin: [126, 128)<br>Observed:<br>126 (20)<br>127 (16)<br>127.059<br>Proportion:0.0025 (n=37)<br>Cumulative:0.9503\",\"Bin: [128, 130)<br>Observed:<br>128 (11)<br>129 (10)<br>Proportion:0.0014 (n=21)<br>Cumulative:0.9518\",\"Bin: [130, 132)<br>Observed:<br>130 (15)<br>131 (9)<br>Proportion:0.0016 (n=24)<br>Cumulative:0.9534\",\"Bin: [132, 134)<br>Observed:<br>132 (24)<br>132.941<br>133 (15)<br>Proportion:0.0027 (n=40)<br>Cumulative:0.9561\",\"Bin: [134, 136)<br>Observed:<br>134 (8)<br>134.118<br>134.706<br>135 (22)<br>135.882<br>Proportion:0.0022 (n=33)<br>Cumulative:0.9583\",\"Bin: [136, 138)<br>Observed:<br>136 (10)<br>137 (12)<br>137.059 (2)<br>Proportion:0.0016 (n=24)<br>Cumulative:0.9599\",\"Bin: [138, 140)<br>Observed:<br>138 (4)<br>138.824<br>139 (9)<br>Proportion:9e-04 (n=14)<br>Cumulative:0.9608\",\"Bin: [140, 142)<br>Observed:<br>140 (9)<br>140.588<br>141 (11)<br>141.176<br>Proportion:0.0015 (n=22)<br>Cumulative:0.9623\",\"Bin: [142, 144)<br>Observed:<br>142 (11)<br>142.353<br>143 (9)<br>Proportion:0.0014 (n=21)<br>Cumulative:0.9637\",\"Bin: [144, 146)<br>Observed:<br>144 (7)<br>144.706 (2)<br>145 (11)<br>145.294<br>Proportion:0.0014 (n=21)<br>Cumulative:0.9651\",\"Bin: [146, 148)<br>Observed:<br>146 (13)<br>147 (7)<br>147.059<br>Proportion:0.0014 (n=21)<br>Cumulative:0.9665\",\"Bin: [148, 150)<br>Observed:<br>148 (6)<br>148.235<br>149 (15)<br>Proportion:0.0015 (n=22)<br>Cumulative:0.968\",\"Bin: [150, 152)<br>Observed:<br>150 (11)<br>151 (11)<br>Proportion:0.0015 (n=22)<br>Cumulative:0.9695\",\"Bin: [152, 154)<br>Observed:<br>152 (7)<br>153 (9)<br>Proportion:0.0011 (n=16)<br>Cumulative:0.9706\",\"Bin: [154, 156)<br>Observed:<br>154 (6)<br>155 (7)<br>Proportion:9e-04 (n=13)<br>Cumulative:0.9714\",\"Bin: [156, 158)<br>Observed:<br>156 (9)<br>157 (4)<br>157.647<br>Proportion:9e-04 (n=14)<br>Cumulative:0.9724\",\"Bin: [158, 160)<br>Observed:<br>158 (7)<br>158.824<br>159 (9)<br>Proportion:0.0011 (n=17)<br>Cumulative:0.9735\",\"Bin: [160, 162)<br>Observed:<br>160 (7)<br>160.588<br>161 (7)<br>Proportion:0.001 (n=15)<br>Cumulative:0.9745\",\"Bin: [162, 164)<br>Observed:<br>162 (4)<br>163 (9)<br>Proportion:9e-04 (n=13)<br>Cumulative:0.9754\",\"Bin: [164, 166)<br>Observed:<br>164 (4)<br>164.118<br>165 (7)<br>Proportion:8e-04 (n=12)<br>Cumulative:0.9762\",\"Bin: [166, 168)<br>Observed:<br>166 (7)<br>167 (6)<br>Proportion:9e-04 (n=13)<br>Cumulative:0.9771\",\"Bin: [168, 170)<br>Observed:<br>168 (7)<br>169 (4)<br>Proportion:7e-04 (n=11)<br>Cumulative:0.9778\",\"Bin: [170, 172)<br>Observed:<br>170 (7)<br>171 (5)<br>171.176<br>Proportion:9e-04 (n=13)<br>Cumulative:0.9787\",\"Bin: [172, 174)<br>Observed:<br>172 (5)<br>173 (7)<br>Proportion:8e-04 (n=12)<br>Cumulative:0.9795\",\"Bin: [174, 176)<br>Observed:<br>174 (3)<br>175 (4)<br>175.294<br>Proportion:5e-04 (n=8)<br>Cumulative:0.98\",\"Bin: [176, 178)<br>Observed:<br>176 (5)<br>177 (3)<br>177.647 (2)<br>Proportion:7e-04 (n=10)<br>Cumulative:0.9807\",\"Bin: [178, 180)<br>Observed:<br>178 (3)<br>179 (7)<br>Proportion:7e-04 (n=10)<br>Cumulative:0.9814\",\"Bin: [180, 182)<br>Observed:<br>180 (8)<br>180.588<br>181 (3)<br>Proportion:8e-04 (n=12)<br>Cumulative:0.9822\",\"Bin: [182, 184)<br>Observed:<br>182 (6)<br>183 (6)<br>Proportion:8e-04 (n=12)<br>Cumulative:0.983\",\"Bin: [184, 186)<br>Observed:<br>184 (3)<br>185 (4)<br>Proportion:5e-04 (n=7)<br>Cumulative:0.9835\",\"Bin: [186, 188)<br>Observed:<br>186 (4)<br>187 (2)<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9839\",\"Bin: [188, 190)<br>Observed:<br>188 (6)<br>188.824<br>189 (2)<br>Proportion:6e-04 (n=9)<br>Cumulative:0.9845\",\"Bin: [190, 192)<br>Observed:<br>190 (7)<br>191 (2)<br>Proportion:6e-04 (n=9)<br>Cumulative:0.9851\",\"Bin: [192, 194)<br>Observed:<br>192 (3)<br>193 (2)<br>193.529<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9855\",\"Bin: [194, 196)<br>Observed:<br>194 (4)<br>195 (3)<br>Proportion:5e-04 (n=7)<br>Cumulative:0.986\",\"Bin: [196, 198)<br>Observed:<br>197 (4)<br>Proportion:3e-04 (n=4)<br>Cumulative:0.9862\",\"Bin: [198, 200)<br>Observed:<br>198 (4)<br>199 (2)<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9866\",\"Bin: [200, 202)<br>Observed:<br>200 (4)<br>201 (5)<br>Proportion:6e-04 (n=9)<br>Cumulative:0.9872\",\"Bin: [202, 204)<br>Observed:<br>202 (3)<br>203 (2)<br>203.529<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9876\",\"Bin: [204, 206)<br>Observed:<br>204 (2)<br>205 (4)<br>205.294<br>Proportion:5e-04 (n=7)<br>Cumulative:0.9881\",\"Bin: [206, 208)<br>Observed:<br>206 (2)<br>207 (6)<br>Proportion:5e-04 (n=8)<br>Cumulative:0.9886\",\"Bin: [208, 210)<br>Observed:<br>208 (3)<br>209<br>Proportion:3e-04 (n=4)<br>Cumulative:0.9889\",\"Bin: [210, 212)<br>Observed:<br>210<br>211 (3)<br>211.765<br>Proportion:3e-04 (n=5)<br>Cumulative:0.9892\",\"Bin: [212, 214)<br>Observed:<br>212.941<br>213<br>Proportion:1e-04 (n=2)<br>Cumulative:0.9894\",\"Bin: [214, 216]<br>Observed:<br>214 (3)<br>216 (2)<br>Proportion:3e-04 (n=5)<br>Cumulative:0.9897\",\"Q<sub>0.99<\\/sub>:218<br>Bin: (216, 2252]<br>Observed:<br>217 - 2252 (106 distinct)<br>Proportion:0.0103 (n=153)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 14
</font><font size="2"> 16 </font><font size="3"> 23
</font><font size="4"> <strong>36</strong> </font><font size="3"> 61
</font><font size="2"> 96 </font><font size="1"> 127 </font></td>
</tr>
<tr class="even">
<td class="gt_row gt_left" headers="Variable">ggt</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">GGT</td>
<td class="gt_row gt_right" headers="n">12148</td>
<td class="gt_row gt_right" headers="Missing">3836</td>
<td class="gt_row gt_right" headers="Distinct">617</td>
<td class="gt_row gt_right" headers="Info">1.000</td>
<td class="gt_row gt_right" headers="Mean">71.24</td>
<td class="gt_row gt_right" headers="Gmd">69.73</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-5c1ee32172431f9fdc62"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-5c1ee32172431f9fdc62">{"x":{"values":[519,962,1105,1035,982,827,756,651,567,503,418,332,318,273,266,194,180,172,152,139,123,110,89,107,79,87,62,66,69,64,48,43,44,36,36,46,34,21,25,19,22,25,16,15,18,16,27,16,15,17,11,15,11,12,12,18,8,12,10,13,9,11,9,12,5,5,6,3,10,4,8,6,3,8,4,6,3,4,1,3,7,5,2,3,3,6,0,4,4,4,122],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:11<br>Bin: [5, 15)<br>Observed:<br>5 - 14.1176 (14 distinct)<br>Proportion:0.0427 (n=519)<br>Cumulative:0.0427\",\"Bin: [15, 20)<br>Observed:<br>15 - 19.4118 (11 distinct)<br>Proportion:0.0792 (n=962)<br>Cumulative:0.1219\",\"Bin: [20, 25)<br>Observed:<br>20 - 24.7059 (11 distinct)<br>Proportion:0.091 (n=1105)<br>Cumulative:0.2129\",\"Bin: [25, 30)<br>Observed:<br>25 - 29.4118 (13 distinct)<br>Proportion:0.0852 (n=1035)<br>Cumulative:0.2981\",\"Bin: [30, 35)<br>Observed:<br>30 - 34.1176 (11 distinct)<br>Proportion:0.0808 (n=982)<br>Cumulative:0.3789\",\"Bin: [35, 40)<br>Observed:<br>35 - 39.4118 (13 distinct)<br>Proportion:0.0681 (n=827)<br>Cumulative:0.447\",\"Bin: [40, 45)<br>Observed:<br>40 - 44.7059 (12 distinct)<br>Proportion:0.0622 (n=756)<br>Cumulative:0.5092\",\"Bin: [45, 50)<br>Observed:<br>45 - 49.4118 (13 distinct)<br>Proportion:0.0536 (n=651)<br>Cumulative:0.5628\",\"Bin: [50, 55)<br>Observed:<br>50 - 54.7059 (11 distinct)<br>Proportion:0.0467 (n=567)<br>Cumulative:0.6095\",\"Bin: [55, 60)<br>Observed:<br>55 - 59 (12 distinct)<br>Proportion:0.0414 (n=503)<br>Cumulative:0.6509\",\"Bin: [60, 65)<br>Observed:<br>60 (99)<br>61 (101)<br>62 (67)<br>63 (70)<br>64 (73)<br>64.1176<br>64.7059 (7)<br>Proportion:0.0344 (n=418)<br>Cumulative:0.6853\",\"Bin: [65, 70)<br>Observed:<br>65 (59)<br>65.8824 (2)<br>66 (68)<br>67 (70)<br>68 (66)<br>68.2353 (2)<br>69 (64)<br>69.4118<br>Proportion:0.0273 (n=332)<br>Cumulative:0.7126\",\"Bin: [70, 75)<br>Observed:<br>70 (61)<br>70.5882 (5)<br>71 (67)<br>71.1765<br>71.7647<br>72 (64)<br>73 (58)<br>74 (61)<br>Proportion:0.0262 (n=318)<br>Cumulative:0.7388\",\"Bin: [75, 80)<br>Observed:<br>75 (59)<br>76 (57)<br>76.4706 (9)<br>77 (44)<br>78 (56)<br>79 (48)<br>Proportion:0.0225 (n=273)<br>Cumulative:0.7613\",\"Bin: [80, 85)<br>Observed:<br>80 (62)<br>80.5882<br>81 (53)<br>81.7647<br>82 (44)<br>82.3529 (7)<br>83 (56)<br>84 (42)<br>Proportion:0.0219 (n=266)<br>Cumulative:0.7832\",\"Bin: [85, 90)<br>Observed:<br>85 (41)<br>86 (38)<br>87 (33)<br>88 (42)<br>88.2353 (4)<br>89 (36)<br>Proportion:0.016 (n=194)<br>Cumulative:0.7991\",\"Bin: [90, 95)<br>Observed:<br>90 (41)<br>91 (40)<br>92 (27)<br>93 (43)<br>94 (25)<br>94.1176 (4)<br>Proportion:0.0148 (n=180)<br>Cumulative:0.814\",\"Bin: [95, 100)<br>Observed:<br>95 (47)<br>96 (36)<br>97 (41)<br>97.0588<br>98 (17)<br>99 (30)<br>Proportion:0.0142 (n=172)<br>Cumulative:0.8281\",\"Bin: [100, 105)<br>Observed:<br>100 (34)<br>101 (23)<br>102 (29)<br>103 (29)<br>103.529<br>104 (35)<br>104.706<br>Proportion:0.0125 (n=152)<br>Cumulative:0.8406\",\"Bin: [105, 110)<br>Observed:<br>105 (36)<br>105.294 (2)<br>105.882 (5)<br>106 (24)<br>107 (24)<br>108 (23)<br>108.824<br>109 (24)<br>Proportion:0.0114 (n=139)<br>Cumulative:0.8521\",\"Bin: [110, 115)<br>Observed:<br>110 (20)<br>111 (28)<br>111.765<br>112 (24)<br>113 (28)<br>114 (22)<br>Proportion:0.0101 (n=123)<br>Cumulative:0.8622\",\"Bin: [115, 120)<br>Observed:<br>115 (23)<br>116 (23)<br>117 (22)<br>117.647 (3)<br>118 (22)<br>119 (17)<br>Proportion:0.0091 (n=110)<br>Cumulative:0.8713\",\"Bin: [120, 125)<br>Observed:<br>120 (22)<br>121 (20)<br>122 (9)<br>123 (19)<br>123.529<br>124 (18)<br>Proportion:0.0073 (n=89)<br>Cumulative:0.8786\",\"Bin: [125, 130)<br>Observed:<br>125 (13)<br>126 (28)<br>127 (15)<br>128 (26)<br>129 (21)<br>129.412 (4)<br>Proportion:0.0088 (n=107)<br>Cumulative:0.8874\",\"Bin: [130, 135)<br>Observed:<br>130 (14)<br>131 (21)<br>132 (11)<br>133 (18)<br>134 (15)<br>Proportion:0.0065 (n=79)<br>Cumulative:0.8939\",\"Bin: [135, 140)<br>Observed:<br>135 (15)<br>135.294 (7)<br>136 (11)<br>137 (16)<br>138 (15)<br>139 (23)<br>Proportion:0.0072 (n=87)<br>Cumulative:0.9011\",\"Bin: [140, 145)<br>Observed:<br>140 (8)<br>141 (14)<br>141.176 (2)<br>142 (12)<br>143 (16)<br>144 (10)<br>Proportion:0.0051 (n=62)<br>Cumulative:0.9062\",\"Bin: [145, 150)<br>Observed:<br>145 (17)<br>146 (6)<br>147 (17)<br>148 (16)<br>149 (10)<br>Proportion:0.0054 (n=66)<br>Cumulative:0.9116\",\"Bin: [150, 155)<br>Observed:<br>150 (14)<br>151 (15)<br>152 (7)<br>152.941 (2)<br>153 (18)<br>154 (13)<br>Proportion:0.0057 (n=69)<br>Cumulative:0.9173\",\"Bin: [155, 160)<br>Observed:<br>155 (16)<br>155.294<br>156 (8)<br>157 (12)<br>158 (13)<br>158.824 (2)<br>159 (12)<br>Proportion:0.0053 (n=64)<br>Cumulative:0.9225\",\"Bin: [160, 165)<br>Observed:<br>160 (13)<br>161 (9)<br>162 (11)<br>163 (7)<br>164 (6)<br>164.706 (2)<br>Proportion:0.004 (n=48)<br>Cumulative:0.9265\",\"Bin: [165, 170)<br>Observed:<br>165 (7)<br>166 (6)<br>167 (10)<br>168 (12)<br>169 (8)<br>Proportion:0.0035 (n=43)<br>Cumulative:0.93\",\"Bin: [170, 175)<br>Observed:<br>170 (5)<br>170.588<br>171 (9)<br>172 (11)<br>173 (8)<br>174 (10)<br>Proportion:0.0036 (n=44)<br>Cumulative:0.9337\",\"Bin: [175, 180)<br>Observed:<br>175 (5)<br>176 (5)<br>176.471<br>177 (8)<br>178 (10)<br>179 (7)<br>Proportion:0.003 (n=36)<br>Cumulative:0.9366\",\"Bin: [180, 185)<br>Observed:<br>180 (2)<br>181 (7)<br>182 (11)<br>182.353<br>183 (7)<br>184 (8)<br>Proportion:0.003 (n=36)<br>Cumulative:0.9396\",\"Bin: [185, 190)<br>Observed:<br>185 (11)<br>186 (6)<br>187 (6)<br>188 (7)<br>188.235 (3)<br>189 (13)<br>Proportion:0.0038 (n=46)<br>Cumulative:0.9434\",\"Bin: [190, 195)<br>Observed:<br>190 (5)<br>191 (6)<br>192 (5)<br>193 (7)<br>194 (8)<br>194.118 (3)<br>Proportion:0.0028 (n=34)<br>Cumulative:0.9462\",\"Bin: [195, 200)<br>Observed:<br>195 (3)<br>196 (7)<br>197 (5)<br>198 (3)<br>199 (3)<br>Proportion:0.0017 (n=21)<br>Cumulative:0.9479\",\"Bin: [200, 205)<br>Observed:<br>200 (10)<br>202<br>203 (8)<br>204 (6)<br>Proportion:0.0021 (n=25)<br>Cumulative:0.95\",\"Bin: [205, 210)<br>Observed:<br>205 (8)<br>206 (4)<br>208 (3)<br>209 (4)<br>Proportion:0.0016 (n=19)<br>Cumulative:0.9515\",\"Bin: [210, 215)<br>Observed:<br>210 (5)<br>211 (4)<br>211.765<br>212 (4)<br>213 (5)<br>214 (3)<br>Proportion:0.0018 (n=22)<br>Cumulative:0.9533\",\"Bin: [215, 220)<br>Observed:<br>215 (2)<br>216 (7)<br>217 (5)<br>217.647 (2)<br>218 (3)<br>219 (6)<br>Proportion:0.0021 (n=25)<br>Cumulative:0.9554\",\"Bin: [220, 225)<br>Observed:<br>220 (2)<br>221 (5)<br>222 (3)<br>223 (2)<br>223.529<br>224 (3)<br>Proportion:0.0013 (n=16)<br>Cumulative:0.9567\",\"Bin: [225, 230)<br>Observed:<br>225 (3)<br>226 (4)<br>227 (3)<br>228 (3)<br>229 (2)<br>Proportion:0.0012 (n=15)<br>Cumulative:0.9579\",\"Bin: [230, 235)<br>Observed:<br>230 (4)<br>231 (4)<br>232 (4)<br>233 (4)<br>234 (2)<br>Proportion:0.0015 (n=18)<br>Cumulative:0.9594\",\"Bin: [235, 240)<br>Observed:<br>235<br>236 (5)<br>237 (5)<br>238 (3)<br>239 (2)<br>Proportion:0.0013 (n=16)<br>Cumulative:0.9607\",\"Bin: [240, 245)<br>Observed:<br>240 (5)<br>241 (4)<br>241.176 (2)<br>242 (7)<br>243 (5)<br>244 (4)<br>Proportion:0.0022 (n=27)<br>Cumulative:0.963\",\"Bin: [245, 250)<br>Observed:<br>245 (2)<br>246 (4)<br>247 (2)<br>247.059<br>249 (7)<br>Proportion:0.0013 (n=16)<br>Cumulative:0.9643\",\"Bin: [250, 255)<br>Observed:<br>250 (6)<br>251 (3)<br>252.941<br>253 (3)<br>254 (2)<br>Proportion:0.0012 (n=15)<br>Cumulative:0.9655\",\"Bin: [255, 260)<br>Observed:<br>255 (2)<br>256 (5)<br>257 (3)<br>258 (4)<br>258.824<br>259 (2)<br>Proportion:0.0014 (n=17)<br>Cumulative:0.9669\",\"Bin: [260, 265)<br>Observed:<br>260 (4)<br>261<br>262<br>263 (3)<br>264 (2)<br>Proportion:9e-04 (n=11)<br>Cumulative:0.9678\",\"Bin: [265, 270)<br>Observed:<br>265 (5)<br>266 (2)<br>267 (2)<br>268<br>269 (5)<br>Proportion:0.0012 (n=15)<br>Cumulative:0.969\",\"Bin: [270, 275)<br>Observed:<br>271 (3)<br>272 (4)<br>273 (2)<br>274 (2)<br>Proportion:9e-04 (n=11)<br>Cumulative:0.97\",\"Bin: [275, 280)<br>Observed:<br>275 (5)<br>276 (2)<br>277<br>278 (2)<br>279 (2)<br>Proportion:0.001 (n=12)<br>Cumulative:0.9709\",\"Bin: [280, 285)<br>Observed:<br>280<br>281 (2)<br>282<br>283 (2)<br>284 (6)<br>Proportion:0.001 (n=12)<br>Cumulative:0.9719\",\"Bin: [285, 290)<br>Observed:<br>285 (7)<br>286<br>287 (4)<br>288 (5)<br>288.235<br>Proportion:0.0015 (n=18)<br>Cumulative:0.9734\",\"Bin: [290, 295)<br>Observed:<br>290 (2)<br>292<br>292.353<br>293<br>294 (2)<br>294.118<br>Proportion:7e-04 (n=8)<br>Cumulative:0.9741\",\"Bin: [295, 300)<br>Observed:<br>295 (2)<br>296 (3)<br>297<br>298 (4)<br>299 (2)<br>Proportion:0.001 (n=12)<br>Cumulative:0.9751\",\"Bin: [300, 305)<br>Observed:<br>300 (2)<br>301<br>302<br>303 (5)<br>304<br>Proportion:8e-04 (n=10)<br>Cumulative:0.9759\",\"Bin: [305, 310)<br>Observed:<br>305 (3)<br>307 (4)<br>308 (4)<br>309 (2)<br>Proportion:0.0011 (n=13)<br>Cumulative:0.977\",\"Bin: [310, 315)<br>Observed:<br>310 (2)<br>311<br>312 (2)<br>313 (3)<br>314<br>Proportion:7e-04 (n=9)<br>Cumulative:0.9777\",\"Bin: [315, 320)<br>Observed:<br>315<br>316<br>317 (3)<br>317.647<br>318 (3)<br>319 (2)<br>Proportion:9e-04 (n=11)<br>Cumulative:0.9786\",\"Bin: [320, 325)<br>Observed:<br>320 (2)<br>322 (2)<br>323<br>324 (4)<br>Proportion:7e-04 (n=9)<br>Cumulative:0.9793\",\"Bin: [325, 330)<br>Observed:<br>325<br>326 (4)<br>327 (2)<br>328 (4)<br>329.412<br>Proportion:0.001 (n=12)<br>Cumulative:0.9803\",\"Bin: [330, 335)<br>Observed:<br>331<br>334 (4)<br>Proportion:4e-04 (n=5)<br>Cumulative:0.9807\",\"Bin: [335, 340)<br>Observed:<br>336 (2)<br>338 (2)<br>339<br>Proportion:4e-04 (n=5)<br>Cumulative:0.9811\",\"Bin: [340, 345)<br>Observed:<br>340 (3)<br>343<br>344 (2)<br>Proportion:5e-04 (n=6)<br>Cumulative:0.9816\",\"Bin: [345, 350)<br>Observed:<br>346 (2)<br>349<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9819\",\"Bin: [350, 355)<br>Observed:<br>350 (3)<br>351 (2)<br>352<br>352.941<br>353 (2)<br>354<br>Proportion:8e-04 (n=10)<br>Cumulative:0.9827\",\"Bin: [355, 360)<br>Observed:<br>355<br>356 (2)<br>359<br>Proportion:3e-04 (n=4)<br>Cumulative:0.983\",\"Bin: [360, 365)<br>Observed:<br>360<br>361<br>362<br>363<br>364 (3)<br>364.706<br>Proportion:7e-04 (n=8)<br>Cumulative:0.9837\",\"Bin: [365, 370)<br>Observed:<br>366 (2)<br>367 (3)<br>369<br>Proportion:5e-04 (n=6)<br>Cumulative:0.9842\",\"Bin: [370, 375)<br>Observed:<br>370<br>372<br>374<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9844\",\"Bin: [375, 380)<br>Observed:<br>375 (2)<br>376<br>377<br>378 (2)<br>379 (2)<br>Proportion:7e-04 (n=8)<br>Cumulative:0.9851\",\"Bin: [380, 385)<br>Observed:<br>381<br>382<br>384 (2)<br>Proportion:3e-04 (n=4)<br>Cumulative:0.9854\",\"Bin: [385, 390)<br>Observed:<br>385 (3)<br>386<br>387<br>389<br>Proportion:5e-04 (n=6)<br>Cumulative:0.9859\",\"Bin: [390, 395)<br>Observed:<br>391<br>392<br>394<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9862\",\"Bin: [395, 400)<br>Observed:<br>395<br>396<br>398<br>399<br>Proportion:3e-04 (n=4)<br>Cumulative:0.9865\",\"Bin: [400, 405)<br>Observed:<br>404<br>Proportion:1e-04 (n=1)<br>Cumulative:0.9866\",\"Bin: [405, 410)<br>Observed:<br>405.882<br>409 (2)<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9868\",\"Bin: [410, 415)<br>Observed:<br>410<br>413 (6)<br>Proportion:6e-04 (n=7)<br>Cumulative:0.9874\",\"Bin: [415, 420)<br>Observed:<br>415 (2)<br>416<br>419 (2)<br>Proportion:4e-04 (n=5)<br>Cumulative:0.9878\",\"Bin: [420, 425)<br>Observed:<br>420<br>423.529<br>Proportion:2e-04 (n=2)<br>Cumulative:0.988\",\"Bin: [425, 430)<br>Observed:<br>426 (2)<br>428<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9882\",\"Bin: [430, 435)<br>Observed:<br>431<br>433<br>434<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9885\",\"Bin: [435, 440)<br>Observed:<br>435<br>435.294<br>436<br>438<br>439 (2)<br>Proportion:5e-04 (n=6)<br>Cumulative:0.989\",\"\",\"Bin: [445, 450)<br>Observed:<br>445 (2)<br>446<br>449<br>Proportion:3e-04 (n=4)<br>Cumulative:0.9893\",\"Bin: [450, 455)<br>Observed:<br>450<br>452<br>453<br>454<br>Proportion:3e-04 (n=4)<br>Cumulative:0.9896\",\"Bin: [455, 460]<br>Observed:<br>456 (2)<br>458<br>460<br>Proportion:3e-04 (n=4)<br>Cumulative:0.99\",\"Q<sub>0.99<\\/sub>:462<br>Bin: (460, 3236]<br>Observed:<br>462 - 3236 (109 distinct)<br>Proportion:0.01 (n=122)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 15.0
</font><font size="2"> 18.0 </font><font size="3"> 27.0
</font><font size="4"> <strong>44.0</strong> </font><font size="3"> 77.0
</font><font size="2"> 139.0 </font><font size="1"> 204.6 </font></td>
</tr>
<tr class="odd">
<td class="gt_row gt_left" headers="Variable">gluc</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">Glucose</td>
<td class="gt_row gt_right" headers="n">13766</td>
<td class="gt_row gt_right" headers="Missing">2218</td>
<td class="gt_row gt_right" headers="Distinct">1200</td>
<td class="gt_row gt_right" headers="Info">1.000</td>
<td class="gt_row gt_right" headers="Mean">6.388</td>
<td class="gt_row gt_right" headers="Gmd">1.872</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-5c1825c85b0301fccd27"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-5c1825c85b0301fccd27">{"x":{"values":[185,67,96,90,188,276,357,457,344,604,679,714,728,448,656,597,521,495,259,409,395,336,285,193,251,251,240,203,119,189,191,166,155,94,141,150,121,88,78,107,104,95,91,61,70,63,46,56,48,56,50,45,50,33,46,32,40,41,27,34,35,29,27,27,21,25,27,24,23,27,17,14,19,9,17,26,21,14,7,15,15,11,13,6,7,10,6,17,12,3,9,8,8,1,7,13,11,8,6,7,3,10,140],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:4.1<br>Bin: [0.611111111111111, 4.2)<br>Observed:<br>0.611111 - 4.19 (72 distinct)<br>Proportion:0.0134 (n=185)<br>Cumulative:0.0134\",\"Bin: [4.2, 4.3)<br>Observed:<br>4.2 (16)<br>4.22222 (24)<br>4.23<br>4.25 (2)<br>4.26 (2)<br>4.27778 (19)<br>4.28<br>4.29 (2)<br>Proportion:0.0049 (n=67)<br>Cumulative:0.0183\",\"Bin: [4.3, 4.4)<br>Observed:<br>4.3 (19)<br>4.31 (2)<br>4.33333 (35)<br>4.35<br>4.36<br>4.368<br>4.37<br>4.38<br>4.38889 (33)<br>4.39 (2)<br>Proportion:0.007 (n=96)<br>Cumulative:0.0253\",\"Bin: [4.4, 4.5)<br>Observed:<br>4.4 - 4.49631 (12 distinct)<br>Proportion:0.0065 (n=90)<br>Cumulative:0.0318\",\"Bin: [4.5, 4.6)<br>Observed:<br>4.5 - 4.592 (13 distinct)<br>Proportion:0.0137 (n=188)<br>Cumulative:0.0455\",\"Bin: [4.6, 4.7)<br>Observed:<br>4.6 - 4.69 (14 distinct)<br>Proportion:0.02 (n=276)<br>Cumulative:0.0655\",\"Bin: [4.7, 4.8)<br>Observed:<br>4.7 - 4.79 (15 distinct)<br>Proportion:0.0259 (n=357)<br>Cumulative:0.0915\",\"Bin: [4.8, 4.9)<br>Observed:<br>4.8 - 4.89 (16 distinct)<br>Proportion:0.0332 (n=457)<br>Cumulative:0.1247\",\"Bin: [4.9, 5)<br>Observed:<br>4.9 - 4.9959 (15 distinct)<br>Proportion:0.025 (n=344)<br>Cumulative:0.1496\",\"Bin: [5, 5.1)<br>Observed:<br>5 - 5.096 (13 distinct)<br>Proportion:0.0439 (n=604)<br>Cumulative:0.1935\",\"Bin: [5.1, 5.2)<br>Observed:<br>5.1 - 5.19 (15 distinct)<br>Proportion:0.0493 (n=679)<br>Cumulative:0.2428\",\"Bin: [5.2, 5.3)<br>Observed:<br>5.2 - 5.29 (16 distinct)<br>Proportion:0.0519 (n=714)<br>Cumulative:0.2947\",\"Bin: [5.3, 5.4)<br>Observed:<br>5.3 - 5.39 (15 distinct)<br>Proportion:0.0529 (n=728)<br>Cumulative:0.3476\",\"Bin: [5.4, 5.5)<br>Observed:<br>5.4 - 5.49549 (14 distinct)<br>Proportion:0.0325 (n=448)<br>Cumulative:0.3801\",\"Bin: [5.5, 5.6)<br>Observed:<br>5.5 - 5.59 (13 distinct)<br>Proportion:0.0477 (n=656)<br>Cumulative:0.4278\",\"Bin: [5.6, 5.7)<br>Observed:<br>5.6 - 5.69 (15 distinct)<br>Proportion:0.0434 (n=597)<br>Cumulative:0.4712\",\"Bin: [5.7, 5.8)<br>Observed:<br>5.7 - 5.79 (15 distinct)<br>Proportion:0.0378 (n=521)<br>Cumulative:0.509\",\"Bin: [5.8, 5.9)<br>Observed:<br>5.8 - 5.89 (15 distinct)<br>Proportion:0.036 (n=495)<br>Cumulative:0.545\",\"Bin: [5.9, 6)<br>Observed:<br>5.9 - 5.99508 (15 distinct)<br>Proportion:0.0188 (n=259)<br>Cumulative:0.5638\",\"Bin: [6, 6.1)<br>Observed:<br>6 - 6.09 (13 distinct)<br>Proportion:0.0297 (n=409)<br>Cumulative:0.5935\",\"Bin: [6.1, 6.2)<br>Observed:<br>6.1 - 6.19 (15 distinct)<br>Proportion:0.0287 (n=395)<br>Cumulative:0.6222\",\"Bin: [6.2, 6.3)<br>Observed:<br>6.2 - 6.29 (16 distinct)<br>Proportion:0.0244 (n=336)<br>Cumulative:0.6466\",\"Bin: [6.3, 6.4)<br>Observed:<br>6.3 - 6.39 (15 distinct)<br>Proportion:0.0207 (n=285)<br>Cumulative:0.6673\",\"Bin: [6.4, 6.5)<br>Observed:<br>6.4 - 6.496 (14 distinct)<br>Proportion:0.014 (n=193)<br>Cumulative:0.6813\",\"Bin: [6.5, 6.6)<br>Observed:<br>6.5 - 6.59 (12 distinct)<br>Proportion:0.0182 (n=251)<br>Cumulative:0.6995\",\"Bin: [6.6, 6.7)<br>Observed:<br>6.6 - 6.69 (15 distinct)<br>Proportion:0.0182 (n=251)<br>Cumulative:0.7178\",\"Bin: [6.7, 6.8)<br>Observed:<br>6.7 - 6.79 (15 distinct)<br>Proportion:0.0174 (n=240)<br>Cumulative:0.7352\",\"Bin: [6.8, 6.9)<br>Observed:<br>6.8 - 6.89 (13 distinct)<br>Proportion:0.0147 (n=203)<br>Cumulative:0.75\",\"Bin: [6.9, 7)<br>Observed:<br>6.9 - 6.99426 (14 distinct)<br>Proportion:0.0086 (n=119)<br>Cumulative:0.7586\",\"Bin: [7, 7.1)<br>Observed:<br>7 - 7.09 (13 distinct)<br>Proportion:0.0137 (n=189)<br>Cumulative:0.7723\",\"Bin: [7.1, 7.2)<br>Observed:<br>7.1 - 7.19 (16 distinct)<br>Proportion:0.0139 (n=191)<br>Cumulative:0.7862\",\"Bin: [7.2, 7.3)<br>Observed:<br>7.2 - 7.29 (14 distinct)<br>Proportion:0.0121 (n=166)<br>Cumulative:0.7983\",\"Bin: [7.3, 7.4)<br>Observed:<br>7.3 - 7.39 (13 distinct)<br>Proportion:0.0113 (n=155)<br>Cumulative:0.8095\",\"Bin: [7.4, 7.5)<br>Observed:<br>7.4 - 7.49385 (13 distinct)<br>Proportion:0.0068 (n=94)<br>Cumulative:0.8164\",\"Bin: [7.5, 7.6)<br>Observed:<br>7.5 - 7.59 (12 distinct)<br>Proportion:0.0102 (n=141)<br>Cumulative:0.8266\",\"Bin: [7.6, 7.7)<br>Observed:<br>7.6 - 7.69 (14 distinct)<br>Proportion:0.0109 (n=150)<br>Cumulative:0.8375\",\"Bin: [7.7, 7.8)<br>Observed:<br>7.7 - 7.79 (12 distinct)<br>Proportion:0.0088 (n=121)<br>Cumulative:0.8463\",\"Bin: [7.8, 7.9)<br>Observed:<br>7.8 - 7.896 (13 distinct)<br>Proportion:0.0064 (n=88)<br>Cumulative:0.8527\",\"Bin: [7.9, 8)<br>Observed:<br>7.9 - 7.99344 (13 distinct)<br>Proportion:0.0057 (n=78)<br>Cumulative:0.8583\",\"Bin: [8, 8.1)<br>Observed:<br>8 - 8.09 (11 distinct)<br>Proportion:0.0078 (n=107)<br>Cumulative:0.8661\",\"Bin: [8.1, 8.2)<br>Observed:<br>8.1 - 8.18 (12 distinct)<br>Proportion:0.0076 (n=104)<br>Cumulative:0.8737\",\"Bin: [8.2, 8.3)<br>Observed:<br>8.2 - 8.29 (12 distinct)<br>Proportion:0.0069 (n=95)<br>Cumulative:0.8806\",\"Bin: [8.3, 8.4)<br>Observed:<br>8.3 - 8.39 (13 distinct)<br>Proportion:0.0066 (n=91)<br>Cumulative:0.8872\",\"Bin: [8.4, 8.5)<br>Observed:<br>8.4 - 8.49303 (12 distinct)<br>Proportion:0.0044 (n=61)<br>Cumulative:0.8916\",\"Bin: [8.5, 8.6)<br>Observed:<br>8.5 (32)<br>8.51 (2)<br>8.52<br>8.53 (4)<br>8.55 (2)<br>8.55556 (24)<br>8.56 (2)<br>8.57 (2)<br>8.59<br>Proportion:0.0051 (n=70)<br>Cumulative:0.8967\",\"Bin: [8.6, 8.7)<br>Observed:<br>8.6 - 8.69 (13 distinct)<br>Proportion:0.0046 (n=63)<br>Cumulative:0.9013\",\"Bin: [8.7, 8.8)<br>Observed:<br>8.7 - 8.79 (12 distinct)<br>Proportion:0.0033 (n=46)<br>Cumulative:0.9046\",\"Bin: [8.8, 8.9)<br>Observed:<br>8.8 - 8.89 (12 distinct)<br>Proportion:0.0041 (n=56)<br>Cumulative:0.9087\",\"Bin: [8.9, 9)<br>Observed:<br>8.9 - 8.99262 (12 distinct)<br>Proportion:0.0035 (n=48)<br>Cumulative:0.9122\",\"Bin: [9, 9.1)<br>Observed:<br>9 (17)<br>9.01<br>9.02 (3)<br>9.04813 (2)<br>9.05 (3)<br>9.05556 (21)<br>9.06 (3)<br>9.08 (4)<br>9.09 (2)<br>Proportion:0.0041 (n=56)<br>Cumulative:0.9162\",\"Bin: [9.1, 9.2)<br>Observed:<br>9.1 - 9.19 (11 distinct)<br>Proportion:0.0036 (n=50)<br>Cumulative:0.9199\",\"Bin: [9.2, 9.3)<br>Observed:<br>9.2 - 9.29 (11 distinct)<br>Proportion:0.0033 (n=45)<br>Cumulative:0.9231\",\"Bin: [9.3, 9.4)<br>Observed:<br>9.3 - 9.39 (12 distinct)<br>Proportion:0.0036 (n=50)<br>Cumulative:0.9268\",\"Bin: [9.4, 9.5)<br>Observed:<br>9.4 (8)<br>9.41 (2)<br>9.43<br>9.44<br>9.44444 (13)<br>9.45 (3)<br>9.46 (3)<br>9.49<br>9.49221<br>Proportion:0.0024 (n=33)<br>Cumulative:0.9292\",\"Bin: [9.5, 9.6)<br>Observed:<br>9.5 - 9.59 (11 distinct)<br>Proportion:0.0033 (n=46)<br>Cumulative:0.9325\",\"Bin: [9.6, 9.7)<br>Observed:<br>9.6 (2)<br>9.61<br>9.61111 (10)<br>9.62 (2)<br>9.65874<br>9.66<br>9.66667 (12)<br>9.67<br>9.68<br>9.69<br>Proportion:0.0023 (n=32)<br>Cumulative:0.9348\",\"Bin: [9.7, 9.8)<br>Observed:<br>9.7 - 9.79 (12 distinct)<br>Proportion:0.0029 (n=40)<br>Cumulative:0.9377\",\"Bin: [9.8, 9.9)<br>Observed:<br>9.8 - 9.88889 (11 distinct)<br>Proportion:0.003 (n=41)<br>Cumulative:0.9407\",\"Bin: [9.9, 10)<br>Observed:<br>9.9 - 9.9918 (12 distinct)<br>Proportion:0.002 (n=27)<br>Cumulative:0.9427\",\"Bin: [10, 10.1)<br>Observed:<br>10 (17)<br>10.02 (2)<br>10.03 (2)<br>10.04<br>10.0473<br>10.05<br>10.0556 (6)<br>10.06<br>10.08 (2)<br>10.09<br>Proportion:0.0025 (n=34)<br>Cumulative:0.9452\",\"Bin: [10.1, 10.2)<br>Observed:<br>10.1 - 10.17 (11 distinct)<br>Proportion:0.0025 (n=35)<br>Cumulative:0.9477\",\"Bin: [10.2, 10.3)<br>Observed:<br>10.2 (2)<br>10.21<br>10.22 (3)<br>10.2222 (13)<br>10.23 (2)<br>10.25<br>10.2694<br>10.2778 (6)<br>Proportion:0.0021 (n=29)<br>Cumulative:0.9498\",\"Bin: [10.3, 10.4)<br>Observed:<br>10.3 (2)<br>10.32<br>10.3333 (8)<br>10.34 (2)<br>10.36 (2)<br>10.37 (2)<br>10.38<br>10.3889 (8)<br>10.39<br>Proportion:0.002 (n=27)<br>Cumulative:0.9518\",\"Bin: [10.4, 10.5)<br>Observed:<br>10.4 (5)<br>10.43 (2)<br>10.4359<br>10.44 (5)<br>10.4444 (8)<br>10.45 (2)<br>10.46<br>10.47<br>10.4914 (2)<br>Proportion:0.002 (n=27)<br>Cumulative:0.9537\",\"Bin: [10.5, 10.6)<br>Observed:<br>10.5 (5)<br>10.52<br>10.54<br>10.55<br>10.5556 (10)<br>10.56<br>10.58 (2)<br>Proportion:0.0015 (n=21)<br>Cumulative:0.9553\",\"Bin: [10.6, 10.7)<br>Observed:<br>10.6 - 10.69 (11 distinct)<br>Proportion:0.0018 (n=25)<br>Cumulative:0.9571\",\"Bin: [10.7, 10.8)<br>Observed:<br>10.7 (4)<br>10.7222 (9)<br>10.75<br>10.76<br>10.77<br>10.7778 (10)<br>10.78<br>Proportion:0.002 (n=27)<br>Cumulative:0.959\",\"Bin: [10.8, 10.9)<br>Observed:<br>10.8 (3)<br>10.82 (2)<br>10.8244 (2)<br>10.8333 (5)<br>10.84<br>10.85 (2)<br>10.87<br>10.8889 (8)<br>Proportion:0.0017 (n=24)<br>Cumulative:0.9608\",\"Bin: [10.9, 11)<br>Observed:<br>10.9 (5)<br>10.94 (2)<br>10.9444 (11)<br>10.95<br>10.98<br>10.99 (3)<br>Proportion:0.0017 (n=23)<br>Cumulative:0.9624\",\"Bin: [11, 11.1)<br>Observed:<br>11 (12)<br>11.01<br>11.0465<br>11.05<br>11.0556 (8)<br>11.07<br>11.09 (3)<br>Proportion:0.002 (n=27)<br>Cumulative:0.9644\",\"Bin: [11.1, 11.2)<br>Observed:<br>11.1 (3)<br>11.1111 (4)<br>11.16<br>11.1667 (5)<br>11.17<br>11.18<br>11.19 (2)<br>Proportion:0.0012 (n=17)<br>Cumulative:0.9656\",\"Bin: [11.2, 11.3)<br>Observed:<br>11.21<br>11.213<br>11.2222 (2)<br>11.24 (2)<br>11.25<br>11.27<br>11.2778 (5)<br>11.28<br>Proportion:0.001 (n=14)<br>Cumulative:0.9667\",\"Bin: [11.3, 11.4)<br>Observed:<br>11.3 (4)<br>11.31<br>11.32<br>11.3333 (4)<br>11.34 (3)<br>11.36<br>11.3889 (3)<br>11.39 (2)<br>Proportion:0.0014 (n=19)<br>Cumulative:0.968\",\"Bin: [11.4, 11.5)<br>Observed:<br>11.4 (3)<br>11.42<br>11.4444 (3)<br>11.47<br>11.49<br>Proportion:7e-04 (n=9)<br>Cumulative:0.9687\",\"Bin: [11.5, 11.6)<br>Observed:<br>11.5 (7)<br>11.536<br>11.54<br>11.55<br>11.5556 (5)<br>11.57 (2)<br>Proportion:0.0012 (n=17)<br>Cumulative:0.9699\",\"Bin: [11.6, 11.7)<br>Observed:<br>11.6 (3)<br>11.6016<br>11.6111 (9)<br>11.64 (2)<br>11.65<br>11.6571<br>11.6667 (5)<br>11.67 (2)<br>11.69 (2)<br>Proportion:0.0019 (n=26)<br>Cumulative:0.9718\",\"Bin: [11.7, 11.8)<br>Observed:<br>11.7 - 11.79 (11 distinct)<br>Proportion:0.0015 (n=21)<br>Cumulative:0.9733\",\"Bin: [11.8, 11.9)<br>Observed:<br>11.8 (2)<br>11.8333 (2)<br>11.84<br>11.85<br>11.88<br>11.8889 (7)<br>Proportion:0.001 (n=14)<br>Cumulative:0.9744\",\"Bin: [11.9, 12)<br>Observed:<br>11.9 (2)<br>11.91<br>11.93 (2)<br>11.9444 (2)<br>Proportion:5e-04 (n=7)<br>Cumulative:0.9749\",\"Bin: [12, 12.1)<br>Observed:<br>12 (6)<br>12.01 (2)<br>12.03<br>12.04<br>12.0556 (2)<br>12.06<br>12.08<br>12.09<br>Proportion:0.0011 (n=15)<br>Cumulative:0.976\",\"Bin: [12.1, 12.2)<br>Observed:<br>12.1 (3)<br>12.11<br>12.1111 (2)<br>12.12<br>12.14<br>12.15<br>12.1667 (6)<br>Proportion:0.0011 (n=15)<br>Cumulative:0.977\",\"Bin: [12.2, 12.3)<br>Observed:<br>12.2 (2)<br>12.22<br>12.2222 (3)<br>12.27<br>12.2778 (4)<br>Proportion:8e-04 (n=11)<br>Cumulative:0.9778\",\"Bin: [12.3, 12.4)<br>Observed:<br>12.3 (2)<br>12.32<br>12.3232 (2)<br>12.3333 (4)<br>12.38<br>12.3889 (3)<br>Proportion:9e-04 (n=13)<br>Cumulative:0.9788\",\"Bin: [12.4, 12.5)<br>Observed:<br>12.4<br>12.44<br>12.4444 (3)<br>12.49<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9792\",\"Bin: [12.5, 12.6)<br>Observed:<br>12.5 (4)<br>12.51<br>12.5556 (2)<br>Proportion:5e-04 (n=7)<br>Cumulative:0.9797\",\"Bin: [12.6, 12.7)<br>Observed:<br>12.6 (4)<br>12.6111 (3)<br>12.64<br>12.6667 (2)<br>Proportion:7e-04 (n=10)<br>Cumulative:0.9805\",\"Bin: [12.7, 12.8)<br>Observed:<br>12.7 (5)<br>12.7778<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9809\",\"Bin: [12.8, 12.9)<br>Observed:<br>12.81 (2)<br>12.8333 (10)<br>12.86<br>12.87<br>12.88<br>12.8889<br>12.89<br>Proportion:0.0012 (n=17)<br>Cumulative:0.9821\",\"Bin: [12.9, 13)<br>Observed:<br>12.9<br>12.91 (2)<br>12.93 (2)<br>12.9338<br>12.9444 (3)<br>12.95<br>12.98<br>12.99<br>Proportion:9e-04 (n=12)<br>Cumulative:0.983\",\"Bin: [13, 13.1)<br>Observed:<br>13 (2)<br>13.02<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9832\",\"Bin: [13.1, 13.2)<br>Observed:<br>13.1<br>13.11<br>13.1111 (2)<br>13.12<br>13.15<br>13.1667 (3)<br>Proportion:7e-04 (n=9)<br>Cumulative:0.9839\",\"Bin: [13.2, 13.3)<br>Observed:<br>13.2<br>13.2222 (3)<br>13.26<br>13.2778 (2)<br>13.29<br>Proportion:6e-04 (n=8)<br>Cumulative:0.9845\",\"Bin: [13.3, 13.4)<br>Observed:<br>13.3 (2)<br>13.32<br>13.3224<br>13.3333<br>13.36<br>13.3779<br>13.3889<br>Proportion:6e-04 (n=8)<br>Cumulative:0.985\",\"Bin: [13.4, 13.5)<br>Observed:<br>13.4444<br>Proportion:1e-04 (n=1)<br>Cumulative:0.9851\",\"Bin: [13.5, 13.6)<br>Observed:<br>13.5 (5)<br>13.5556<br>13.59<br>Proportion:5e-04 (n=7)<br>Cumulative:0.9856\",\"Bin: [13.6, 13.7)<br>Observed:<br>13.6 (3)<br>13.6111<br>13.63 (2)<br>13.6555<br>13.6667 (4)<br>13.67<br>13.68<br>Proportion:9e-04 (n=13)<br>Cumulative:0.9866\",\"Bin: [13.7, 13.8)<br>Observed:<br>13.71<br>13.72<br>13.7222 (2)<br>13.74<br>13.75<br>13.77<br>13.7778 (4)<br>Proportion:8e-04 (n=11)<br>Cumulative:0.9874\",\"Bin: [13.8, 13.9)<br>Observed:<br>13.8<br>13.81<br>13.8333 (2)<br>13.8889 (4)<br>Proportion:6e-04 (n=8)<br>Cumulative:0.9879\",\"Bin: [13.9, 14)<br>Observed:<br>13.9<br>13.94<br>13.9444 (2)<br>13.95<br>13.96<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9884\",\"Bin: [14, 14.1)<br>Observed:<br>14<br>14.01 (2)<br>14.0556 (3)<br>14.0995<br>Proportion:5e-04 (n=7)<br>Cumulative:0.9889\",\"Bin: [14.1, 14.2)<br>Observed:<br>14.1<br>14.1111<br>14.1667<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9891\",\"Bin: [14.2, 14.3]<br>Observed:<br>14.2<br>14.21 (2)<br>14.22 (2)<br>14.2778 (5)<br>Proportion:7e-04 (n=10)<br>Cumulative:0.9898\",\"Q<sub>0.99<\\/sub>:14.4<br>Bin: (14.3, 39]<br>Observed:<br>14.32 - 39 (109 distinct)<br>Proportion:0.0102 (n=140)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 4.611
</font><font size="2"> 4.833 </font><font size="3"> 5.210
</font><font size="4"> <strong>5.778</strong> </font><font size="3">
6.898 </font><font size="2"> 8.660 </font><font size="1"> 10.315
</font></td>
</tr>
<tr class="even">
<td class="gt_row gt_left" headers="Variable">chol</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">Cholesterol</td>
<td class="gt_row gt_right" headers="n">13979</td>
<td class="gt_row gt_right" headers="Missing">2005</td>
<td class="gt_row gt_right" headers="Distinct">1010</td>
<td class="gt_row gt_right" headers="Info">1.000</td>
<td class="gt_row gt_right" headers="Mean">4.842</td>
<td class="gt_row gt_right" headers="Gmd">1.241</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-e8be28ec44cd3ee47f07"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-e8be28ec44cd3ee47f07">{"x":{"values":[157,31,32,30,48,42,66,31,72,68,99,91,113,110,128,100,133,132,130,148,194,169,126,177,204,182,210,198,228,187,237,190,230,253,286,216,283,167,262,235,308,236,280,235,284,235,311,224,267,227,277,203,175,197,243,179,256,185,224,151,172,168,188,163,199,147,139,75,130,103,129,110,117,82,101,76,92,69,102,61,54,46,48,49,56,49,43,35,33,30,37,24,31,28,30,23,21,4,14,18,17,17,16,141],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:2.61<br>Bin: [1, 2.65)<br>Observed:<br>1 - 2.64 (63 distinct)<br>Proportion:0.0112 (n=157)<br>Cumulative:0.0112\",\"Bin: [2.65, 2.7)<br>Observed:<br>2.65 (2)<br>2.6574<br>2.66425 (11)<br>2.69<br>2.69012 (16)<br>Proportion:0.0022 (n=31)<br>Cumulative:0.0134\",\"Bin: [2.7, 2.75)<br>Observed:<br>2.7 (4)<br>2.7153<br>2.71599 (11)<br>2.73 (2)<br>2.74 (3)<br>2.74185 (11)<br>Proportion:0.0023 (n=32)<br>Cumulative:0.0157\",\"Bin: [2.75, 2.8)<br>Observed:<br>2.76772 (11)<br>2.77<br>2.79359 (18)<br>Proportion:0.0021 (n=30)<br>Cumulative:0.0179\",\"Bin: [2.8, 2.85)<br>Observed:<br>2.8 (3)<br>2.81<br>2.81945 (18)<br>2.83 (2)<br>2.84<br>2.84532 (23)<br>Proportion:0.0034 (n=48)<br>Cumulative:0.0213\",\"Bin: [2.85, 2.9)<br>Observed:<br>2.85 (2)<br>2.86 (3)<br>2.87 (3)<br>2.87046<br>2.87118 (14)<br>2.88<br>2.89 (2)<br>2.89632<br>2.89705 (15)<br>Proportion:0.003 (n=42)<br>Cumulative:0.0243\",\"Bin: [2.9, 2.95)<br>Observed:<br>2.9 (9)<br>2.91 (3)<br>2.9154<br>2.92 (4)<br>2.92292 (22)<br>2.93 (4)<br>2.94<br>2.94804<br>2.94878 (21)<br>Proportion:0.0047 (n=66)<br>Cumulative:0.029\",\"Bin: [2.95, 3)<br>Observed:<br>2.95 (2)<br>2.96 (3)<br>2.97<br>2.9739<br>2.97465 (19)<br>2.98 (3)<br>2.99 (2)<br>Proportion:0.0022 (n=31)<br>Cumulative:0.0313\",\"Bin: [3, 3.05)<br>Observed:<br>3 (8)<br>3.00052 (21)<br>3.01 (2)<br>3.02 (7)<br>3.02638 (25)<br>3.03 (4)<br>3.04 (4)<br>3.0444<br>Proportion:0.0052 (n=72)<br>Cumulative:0.0364\",\"Bin: [3.05, 3.1)<br>Observed:<br>3.05148<br>3.05225 (29)<br>3.06 (3)<br>3.07 (4)<br>3.07812 (26)<br>3.08 (2)<br>3.09 (3)<br>Proportion:0.0049 (n=68)<br>Cumulative:0.0413\",\"Bin: [3.1, 3.15)<br>Observed:<br>3.1 (21)<br>3.1032<br>3.10398 (21)<br>3.11 (3)<br>3.12 (7)<br>3.12985 (36)<br>3.13 (7)<br>3.14 (3)<br>Proportion:0.0071 (n=99)<br>Cumulative:0.0484\",\"Bin: [3.15, 3.2)<br>Observed:<br>3.15 (8)<br>3.15572 (33)<br>3.16 (3)<br>3.17 (5)<br>3.18 (5)<br>3.18158 (32)<br>3.19 (5)<br>Proportion:0.0065 (n=91)<br>Cumulative:0.0549\",\"Bin: [3.2, 3.25)<br>Observed:<br>3.2 (17)<br>3.20664 (2)<br>3.20745 (39)<br>3.21 (5)<br>3.22 (5)<br>3.23 (7)<br>3.2325<br>3.23332 (32)<br>3.24 (5)<br>Proportion:0.0081 (n=113)<br>Cumulative:0.063\",\"Bin: [3.25, 3.3)<br>Observed:<br>3.25 (3)<br>3.25918 (45)<br>3.26 (3)<br>3.27 (4)<br>3.28 (7)<br>3.28505 (43)<br>3.29 (5)<br>Proportion:0.0079 (n=110)<br>Cumulative:0.0708\",\"Bin: [3.3, 3.35)<br>Observed:<br>3.3 (21)<br>3.3024<br>3.31 (7)<br>3.31092 (45)<br>3.32 (7)<br>3.33<br>3.33678 (41)<br>3.34 (5)<br>Proportion:0.0092 (n=128)<br>Cumulative:0.08\",\"Bin: [3.35, 3.4)<br>Observed:<br>3.35 (4)<br>3.36 (6)<br>3.36265 (41)<br>3.37 (3)<br>3.3798<br>3.38 (6)<br>3.38852 (33)<br>3.39 (6)<br>Proportion:0.0072 (n=100)<br>Cumulative:0.0871\",\"Bin: [3.4, 3.45)<br>Observed:<br>3.4 (27)<br>3.41 (2)<br>3.41438 (45)<br>3.42 (11)<br>3.43 (5)<br>3.43938 (2)<br>3.44 (7)<br>3.44025 (34)<br>Proportion:0.0095 (n=133)<br>Cumulative:0.0966\",\"Bin: [3.45, 3.5)<br>Observed:<br>3.45 (7)<br>3.46 (10)<br>3.46524 (2)<br>3.46611 (33)<br>3.47 (15)<br>3.48 (8)<br>3.49 (8)<br>3.4911 (4)<br>3.49198 (45)<br>Proportion:0.0094 (n=132)<br>Cumulative:0.1061\",\"Bin: [3.5, 3.55)<br>Observed:<br>3.5 - 3.54371 (11 distinct)<br>Proportion:0.0093 (n=130)<br>Cumulative:0.1154\",\"Bin: [3.55, 3.6)<br>Observed:<br>3.55 (11)<br>3.56 (10)<br>3.56958 (47)<br>3.57 (10)<br>3.58 (7)<br>3.59 (5)<br>3.59454 (4)<br>3.59545 (54)<br>Proportion:0.0106 (n=148)<br>Cumulative:0.126\",\"Bin: [3.6, 3.65)<br>Observed:<br>3.6 (37)<br>3.61 (9)<br>3.62 (11)<br>3.6204 (2)<br>3.62131 (53)<br>3.63 (7)<br>3.64 (9)<br>3.64626<br>3.64718 (65)<br>Proportion:0.0139 (n=194)<br>Cumulative:0.1399\",\"Bin: [3.65, 3.7)<br>Observed:<br>3.65 (16)<br>3.66 (5)<br>3.6636<br>3.67 (5)<br>3.67305 (63)<br>3.68 (9)<br>3.69 (9)<br>3.69798 (2)<br>3.69891 (59)<br>Proportion:0.0121 (n=169)<br>Cumulative:0.1519\",\"Bin: [3.7, 3.75)<br>Observed:<br>3.7 (33)<br>3.71 (12)<br>3.7152<br>3.72 (6)<br>3.72478 (54)<br>3.73 (7)<br>3.74 (13)<br>Proportion:0.009 (n=126)<br>Cumulative:0.161\",\"Bin: [3.75, 3.8)<br>Observed:<br>3.75 (8)<br>3.75065 (65)<br>3.76 (13)<br>3.77 (8)<br>3.77651 (63)<br>3.78 (7)<br>3.79 (13)<br>Proportion:0.0127 (n=177)<br>Cumulative:0.1736\",\"Bin: [3.8, 3.85)<br>Observed:<br>3.8 (33)<br>3.80142 (2)<br>3.80238 (63)<br>3.81 (9)<br>3.8184<br>3.82 (10)<br>3.82728<br>3.82825 (62)<br>3.83 (14)<br>3.84 (9)<br>Proportion:0.0146 (n=204)<br>Cumulative:0.1882\",\"Bin: [3.85, 3.9)<br>Observed:<br>3.85 (4)<br>3.85314<br>3.85411 (69)<br>3.86 (14)<br>3.87 (8)<br>3.879<br>3.87998 (68)<br>3.88 (5)<br>3.89 (12)<br>Proportion:0.013 (n=182)<br>Cumulative:0.2012\",\"Bin: [3.9, 3.95)<br>Observed:<br>3.9 (34)<br>3.90486<br>3.90585 (54)<br>3.91 (12)<br>3.92 (15)<br>3.93 (11)<br>3.93072 (6)<br>3.93171 (68)<br>3.94 (9)<br>Proportion:0.015 (n=210)<br>Cumulative:0.2163\",\"Bin: [3.95, 4)<br>Observed:<br>3.95 - 3.999 (11 distinct)<br>Proportion:0.0142 (n=198)<br>Cumulative:0.2304\",\"Bin: [4, 4.05)<br>Observed:<br>4 (46)<br>4.0083 (2)<br>4.00931 (73)<br>4.01 (11)<br>4.02 (9)<br>4.03 (11)<br>4.03416 (7)<br>4.03518 (57)<br>4.04 (12)<br>Proportion:0.0163 (n=228)<br>Cumulative:0.2467\",\"Bin: [4.05, 4.1)<br>Observed:<br>4.05 (13)<br>4.0506 (2)<br>4.06 (12)<br>4.06002 (2)<br>4.06105 (63)<br>4.07 (6)<br>4.08 (9)<br>4.08588<br>4.08691 (66)<br>4.09 (13)<br>Proportion:0.0134 (n=187)<br>Cumulative:0.2601\",\"Bin: [4.1, 4.15)<br>Observed:<br>4.1 (54)<br>4.1022 (2)<br>4.11 (16)<br>4.11174 (2)<br>4.11278 (59)<br>4.12 (14)<br>4.13 (10)<br>4.1376 (4)<br>4.13864 (69)<br>4.14 (7)<br>Proportion:0.017 (n=237)<br>Cumulative:0.2771\",\"Bin: [4.15, 4.2)<br>Observed:<br>4.15 (10)<br>4.1538<br>4.16 (5)<br>4.16346 (3)<br>4.16451 (79)<br>4.17 (12)<br>4.18 (12)<br>4.18932 (4)<br>4.19 (5)<br>4.19038 (59)<br>Proportion:0.0136 (n=190)<br>Cumulative:0.2907\",\"Bin: [4.2, 4.25)<br>Observed:<br>4.2 (48)<br>4.2054 (2)<br>4.21 (15)<br>4.21518 (3)<br>4.21624 (62)<br>4.22 (10)<br>4.23 (5)<br>4.24 (16)<br>4.24104 (2)<br>4.24211 (67)<br>Proportion:0.0165 (n=230)<br>Cumulative:0.3071\",\"Bin: [4.25, 4.3)<br>Observed:<br>4.25 (12)<br>4.257<br>4.26 (16)<br>4.2669 (3)<br>4.26798 (103)<br>4.27 (9)<br>4.28 (5)<br>4.29 (15)<br>4.29276<br>4.29384 (88)<br>Proportion:0.0181 (n=253)<br>Cumulative:0.3252\",\"Bin: [4.3, 4.35)<br>Observed:<br>4.3 (59)<br>4.3086<br>4.31 (11)<br>4.31862 (8)<br>4.31971 (79)<br>4.32 (14)<br>4.33 (13)<br>4.34 (12)<br>4.34448 (5)<br>4.34558 (84)<br>Proportion:0.0205 (n=286)<br>Cumulative:0.3457\",\"Bin: [4.35, 4.4)<br>Observed:<br>4.35 (11)<br>4.36 (10)<br>4.37 (8)<br>4.37034 (3)<br>4.37144 (82)<br>4.38 (8)<br>4.386<br>4.39 (13)<br>4.3962 (3)<br>4.39731 (77)<br>Proportion:0.0155 (n=216)<br>Cumulative:0.3611\",\"Bin: [4.4, 4.45)<br>Observed:<br>4.4 (58)<br>4.41 (11)<br>4.42 (12)<br>4.42206 (4)<br>4.42318 (80)<br>4.43 (5)<br>4.44 (9)<br>4.44792 (5)<br>4.44904 (99)<br>Proportion:0.0202 (n=283)<br>Cumulative:0.3814\",\"Bin: [4.45, 4.5)<br>Observed:<br>4.45 (9)<br>4.46 (17)<br>4.47 (16)<br>4.47378 (6)<br>4.47491 (82)<br>4.48 (20)<br>4.49 (14)<br>4.49964 (3)<br>Proportion:0.0119 (n=167)<br>Cumulative:0.3933\",\"Bin: [4.5, 4.55)<br>Observed:<br>4.5 (50)<br>4.50078 (79)<br>4.51 (9)<br>4.515<br>4.52 (6)<br>4.5255 (2)<br>4.52664 (90)<br>4.53 (14)<br>4.54 (11)<br>Proportion:0.0187 (n=262)<br>Cumulative:0.412\",\"Bin: [4.55, 4.6)<br>Observed:<br>4.55 (12)<br>4.55136 (5)<br>4.55251 (89)<br>4.56 (10)<br>4.57 (7)<br>4.57722 (5)<br>4.57838 (78)<br>4.58 (19)<br>4.59 (9)<br>4.5924<br>Proportion:0.0168 (n=235)<br>Cumulative:0.4289\",\"Bin: [4.6, 4.65)<br>Observed:<br>4.6 (55)<br>4.60308 (7)<br>4.60424 (96)<br>4.61 (18)<br>4.6182 (2)<br>4.62 (12)<br>4.62894 (6)<br>4.63 (10)<br>4.63011 (92)<br>4.64 (10)<br>Proportion:0.022 (n=308)<br>Cumulative:0.4509\",\"Bin: [4.65, 4.7)<br>Observed:<br>4.65 (10)<br>4.6548 (5)<br>4.65598 (93)<br>4.66 (12)<br>4.67 (14)<br>4.68 (6)<br>4.68066 (3)<br>4.68184 (80)<br>4.69 (13)<br>Proportion:0.0169 (n=236)<br>Cumulative:0.4678\",\"Bin: [4.7, 4.75)<br>Observed:<br>4.7 (69)<br>4.70652 (4)<br>4.70771 (78)<br>4.71 (9)<br>4.72 (11)<br>4.7214<br>4.73 (12)<br>4.73238 (6)<br>4.73357 (85)<br>4.74 (5)<br>Proportion:0.02 (n=280)<br>Cumulative:0.4878\",\"Bin: [4.75, 4.8)<br>Observed:<br>4.75 (6)<br>4.75824 (3)<br>4.75944 (89)<br>4.76 (14)<br>4.77 (10)<br>4.78 (7)<br>4.7841 (4)<br>4.78531 (90)<br>4.79 (12)<br>Proportion:0.0168 (n=235)<br>Cumulative:0.5046\",\"Bin: [4.8, 4.85)<br>Observed:<br>4.8 (64)<br>4.80996 (4)<br>4.81 (6)<br>4.81117 (86)<br>4.82 (7)<br>4.8246<br>4.83 (7)<br>4.83582 (2)<br>4.83704 (94)<br>4.84 (13)<br>Proportion:0.0203 (n=284)<br>Cumulative:0.5249\",\"Bin: [4.85, 4.9)<br>Observed:<br>4.85 (9)<br>4.86 (4)<br>4.86168 (4)<br>4.86291 (78)<br>4.87 (14)<br>4.88 (11)<br>4.88754 (4)<br>4.88877 (103)<br>4.89 (8)<br>Proportion:0.0168 (n=235)<br>Cumulative:0.5417\",\"Bin: [4.9, 4.95)<br>Observed:<br>4.9 - 4.94051 (11 distinct)<br>Proportion:0.0222 (n=311)<br>Cumulative:0.564\",\"Bin: [4.95, 5)<br>Observed:<br>4.95 - 4.99224 (11 distinct)<br>Proportion:0.016 (n=224)<br>Cumulative:0.58\",\"Bin: [5, 5.05)<br>Observed:<br>5 (53)<br>5.0052<br>5.01 (7)<br>5.01684 (7)<br>5.01811 (88)<br>5.02 (10)<br>5.03 (10)<br>5.04 (6)<br>5.0427 (2)<br>5.04397 (83)<br>Proportion:0.0191 (n=267)<br>Cumulative:0.5991\",\"Bin: [5.05, 5.1)<br>Observed:<br>5.05 - 5.09571 (11 distinct)<br>Proportion:0.0162 (n=227)<br>Cumulative:0.6154\",\"Bin: [5.1, 5.15)<br>Observed:<br>5.1 (56)<br>5.11 (14)<br>5.12 (12)<br>5.12028 (3)<br>5.12157 (75)<br>5.13 (8)<br>5.1342 (2)<br>5.14 (10)<br>5.14614 (5)<br>5.14744 (92)<br>Proportion:0.0198 (n=277)<br>Cumulative:0.6352\",\"Bin: [5.15, 5.2)<br>Observed:<br>5.15 (11)<br>5.16 (7)<br>5.17 (9)<br>5.172 (3)<br>5.17331 (67)<br>5.18 (4)<br>5.1858<br>5.19 (7)<br>5.19786 (6)<br>5.19917 (88)<br>Proportion:0.0145 (n=203)<br>Cumulative:0.6497\",\"Bin: [5.2, 5.25)<br>Observed:<br>5.2 (53)<br>5.21 (9)<br>5.2116<br>5.22 (7)<br>5.22372 (8)<br>5.22504 (73)<br>5.23 (11)<br>5.2374<br>5.24 (10)<br>5.24958 (2)<br>Proportion:0.0125 (n=175)<br>Cumulative:0.6622\",\"Bin: [5.25, 5.3)<br>Observed:<br>5.25 (6)<br>5.25091 (80)<br>5.26 (8)<br>5.2632 (2)<br>5.27 (6)<br>5.27544 (2)<br>5.27677 (76)<br>5.28 (6)<br>5.29 (11)<br>Proportion:0.0141 (n=197)<br>Cumulative:0.6763\",\"Bin: [5.3, 5.35)<br>Observed:<br>5.3 (45)<br>5.3013 (7)<br>5.30264 (80)<br>5.31 (8)<br>5.3148 (2)<br>5.32 (16)<br>5.32716 (4)<br>5.3285 (58)<br>5.33 (9)<br>5.34 (14)<br>Proportion:0.0174 (n=243)<br>Cumulative:0.6937\",\"Bin: [5.35, 5.4)<br>Observed:<br>5.35 (6)<br>5.35302 (2)<br>5.35437 (82)<br>5.36 (6)<br>5.37 (5)<br>5.37888<br>5.38 (4)<br>5.38024 (69)<br>5.39 (4)<br>Proportion:0.0128 (n=179)<br>Cumulative:0.7065\",\"Bin: [5.4, 5.45)<br>Observed:<br>5.4 (53)<br>5.40474 (4)<br>5.4061 (81)<br>5.41 (6)<br>5.418<br>5.42 (11)<br>5.43 (7)<br>5.4306<br>5.43197 (86)<br>5.44 (6)<br>Proportion:0.0183 (n=256)<br>Cumulative:0.7248\",\"Bin: [5.45, 5.5)<br>Observed:<br>5.45 (6)<br>5.45646 (6)<br>5.45784 (77)<br>5.46 (6)<br>5.47 (7)<br>5.48 (10)<br>5.48232 (4)<br>5.4837 (63)<br>5.49 (5)<br>5.4954<br>Proportion:0.0132 (n=185)<br>Cumulative:0.738\",\"Bin: [5.5, 5.55)<br>Observed:<br>5.5 (49)<br>5.50818 (3)<br>5.50957 (72)<br>5.51 (4)<br>5.52 (9)<br>5.53 (5)<br>5.53404 (2)<br>5.53544 (74)<br>5.54 (6)<br>Proportion:0.016 (n=224)<br>Cumulative:0.7541\",\"Bin: [5.55, 5.6)<br>Observed:<br>5.55 (5)<br>5.5599 (2)<br>5.56 (5)<br>5.5613 (55)<br>5.57 (6)<br>5.58 (7)<br>5.58576<br>5.58717 (61)<br>5.59 (7)<br>5.5986 (2)<br>Proportion:0.0108 (n=151)<br>Cumulative:0.7649\",\"Bin: [5.6, 5.65)<br>Observed:<br>5.6 (46)<br>5.61 (2)<br>5.61162 (2)<br>5.61304 (60)<br>5.62 (7)<br>5.63 (7)<br>5.63748 (3)<br>5.6389 (42)<br>5.64 (3)<br>Proportion:0.0123 (n=172)<br>Cumulative:0.7772\",\"Bin: [5.65, 5.7)<br>Observed:<br>5.65 - 5.69064 (11 distinct)<br>Proportion:0.012 (n=168)<br>Cumulative:0.7892\",\"Bin: [5.7, 5.75)<br>Observed:<br>5.7 (44)<br>5.7018<br>5.71 (5)<br>5.71506<br>5.7165 (61)<br>5.72 (4)<br>5.73 (7)<br>5.74 (5)<br>5.74092 (3)<br>5.74237 (57)<br>Proportion:0.0134 (n=188)<br>Cumulative:0.8026\",\"Bin: [5.75, 5.8)<br>Observed:<br>5.75 - 5.7941 (11 distinct)<br>Proportion:0.0117 (n=163)<br>Cumulative:0.8143\",\"Bin: [5.8, 5.85)<br>Observed:<br>5.8 - 5.84584 (11 distinct)<br>Proportion:0.0142 (n=199)<br>Cumulative:0.8285\",\"Bin: [5.85, 5.9)<br>Observed:<br>5.85 (2)<br>5.8566<br>5.86 (5)<br>5.87 (9)<br>5.87022 (3)<br>5.8717 (61)<br>5.88 (7)<br>5.89 (3)<br>5.89608 (4)<br>5.89757 (52)<br>Proportion:0.0105 (n=147)<br>Cumulative:0.839\",\"Bin: [5.9, 5.95)<br>Observed:<br>5.9 (30)<br>5.91 (5)<br>5.92 (3)<br>5.92194 (5)<br>5.92344 (37)<br>5.93 (5)<br>5.934<br>5.94 (3)<br>5.9478 (3)<br>5.9493 (47)<br>Proportion:0.0099 (n=139)<br>Cumulative:0.849\",\"Bin: [5.95, 6)<br>Observed:<br>5.95 (3)<br>5.96 (5)<br>5.97 (5)<br>5.97366<br>5.97517 (48)<br>5.98 (7)<br>5.99 (5)<br>5.99952<br>Proportion:0.0054 (n=75)<br>Cumulative:0.8544\",\"Bin: [6, 6.05)<br>Observed:<br>6 (25)<br>6.00103 (30)<br>6.01 (6)<br>6.0114<br>6.02 (8)<br>6.0269 (54)<br>6.03 (2)<br>6.04 (4)<br>Proportion:0.0093 (n=130)<br>Cumulative:0.8637\",\"Bin: [6.05, 6.1)<br>Observed:<br>6.05 (3)<br>6.05124 (7)<br>6.05277 (38)<br>6.06 (5)<br>6.063 (2)<br>6.07 (3)<br>6.0771 (2)<br>6.07863 (37)<br>6.08 (2)<br>6.09 (4)<br>Proportion:0.0074 (n=103)<br>Cumulative:0.871\",\"Bin: [6.1, 6.15)<br>Observed:<br>6.1 (24)<br>6.10296 (2)<br>6.1045 (51)<br>6.11<br>6.12 (3)<br>6.12882 (4)<br>6.13 (3)<br>6.13037 (40)<br>6.1404<br>Proportion:0.0092 (n=129)<br>Cumulative:0.8802\",\"Bin: [6.15, 6.2)<br>Observed:<br>6.15 (3)<br>6.15623 (44)<br>6.16 (8)<br>6.1662<br>6.17<br>6.18 (5)<br>6.18054<br>6.1821 (46)<br>6.192<br>Proportion:0.0079 (n=110)<br>Cumulative:0.8881\",\"Bin: [6.2, 6.25)<br>Observed:<br>6.2 (24)<br>6.20797 (38)<br>6.21 (2)<br>6.22 (2)<br>6.23 (3)<br>6.23226 (2)<br>6.23383 (39)<br>6.24 (6)<br>6.2436<br>Proportion:0.0084 (n=117)<br>Cumulative:0.8965\",\"Bin: [6.25, 6.3)<br>Observed:<br>6.25 (4)<br>6.25812<br>6.2597 (34)<br>6.26 (5)<br>6.27 (5)<br>6.28398 (2)<br>6.28557 (28)<br>6.29 (3)<br>Proportion:0.0059 (n=82)<br>Cumulative:0.9024\",\"Bin: [6.3, 6.35)<br>Observed:<br>6.3 (19)<br>6.31 (3)<br>6.31143 (26)<br>6.32 (5)<br>6.33 (2)<br>6.3357 (4)<br>6.3373 (40)<br>6.34 (2)<br>Proportion:0.0072 (n=101)<br>Cumulative:0.9096\",\"Bin: [6.35, 6.4)<br>Observed:<br>6.35<br>6.36 (3)<br>6.36317 (35)<br>6.37 (2)<br>6.38 (3)<br>6.38742 (2)<br>6.38903 (28)<br>6.39<br>6.3984<br>Proportion:0.0054 (n=76)<br>Cumulative:0.915\",\"Bin: [6.4, 6.45)<br>Observed:<br>6.4 (27)<br>6.41 (3)<br>6.41328 (3)<br>6.4149 (25)<br>6.42 (2)<br>6.43 (2)<br>6.44 (4)<br>6.44077 (26)<br>Proportion:0.0066 (n=92)<br>Cumulative:0.9216\",\"Bin: [6.45, 6.5)<br>Observed:<br>6.45 (4)<br>6.46 (2)<br>6.46663 (28)<br>6.47 (2)<br>6.4758<br>6.48 (3)<br>6.49<br>6.4925 (28)<br>Proportion:0.0049 (n=69)<br>Cumulative:0.9265\",\"Bin: [6.5, 6.55)<br>Observed:<br>6.5 (18)<br>6.51 (2)<br>6.51837 (35)<br>6.52<br>6.5274<br>6.53 (2)<br>6.54 (4)<br>6.54258 (3)<br>6.54423 (36)<br>Proportion:0.0073 (n=102)<br>Cumulative:0.9338\",\"Bin: [6.55, 6.6)<br>Observed:<br>6.55<br>6.5532<br>6.56 (2)<br>6.56844 (2)<br>6.57 (2)<br>6.5701 (26)<br>6.58 (5)<br>6.59 (4)<br>6.5943<br>6.59596 (17)<br>Proportion:0.0044 (n=61)<br>Cumulative:0.9382\",\"Bin: [6.6, 6.65)<br>Observed:<br>6.6 (16)<br>6.62 (2)<br>6.62016 (2)<br>6.62183 (13)<br>6.64<br>6.64602 (2)<br>6.6477 (18)<br>Proportion:0.0039 (n=54)<br>Cumulative:0.9421\",\"Bin: [6.65, 6.7)<br>Observed:<br>6.65 (2)<br>6.67 (2)<br>6.67188<br>6.67356 (17)<br>6.68 (3)<br>6.69 (2)<br>6.69943 (19)<br>Proportion:0.0033 (n=46)<br>Cumulative:0.9453\",\"Bin: [6.7, 6.75)<br>Observed:<br>6.7 (18)<br>6.71 (2)<br>6.72<br>6.7253 (20)<br>6.73 (3)<br>6.74 (4)<br>Proportion:0.0034 (n=48)<br>Cumulative:0.9488\",\"Bin: [6.75, 6.8)<br>Observed:<br>6.75 (2)<br>6.75116 (18)<br>6.7596<br>6.76 (2)<br>6.77<br>6.77532<br>6.77703 (20)<br>6.78<br>6.79 (3)<br>Proportion:0.0035 (n=49)<br>Cumulative:0.9523\",\"Bin: [6.8, 6.85)<br>Observed:<br>6.8 (10)<br>6.80118 (3)<br>6.8029 (22)<br>6.81 (3)<br>6.82704 (2)<br>6.82876 (13)<br>6.83<br>6.84 (2)<br>Proportion:0.004 (n=56)<br>Cumulative:0.9563\",\"Bin: [6.85, 6.9)<br>Observed:<br>6.85<br>6.85463 (14)<br>6.86 (2)<br>6.87<br>6.88<br>6.8805 (28)<br>6.89 (2)<br>Proportion:0.0035 (n=49)<br>Cumulative:0.9598\",\"Bin: [6.9, 6.95)<br>Observed:<br>6.9 (7)<br>6.90636 (13)<br>6.91 (3)<br>6.92 (4)<br>6.93 (2)<br>6.93223 (12)<br>6.94 (2)<br>Proportion:0.0031 (n=43)<br>Cumulative:0.9629\",\"Bin: [6.95, 7)<br>Observed:<br>6.95<br>6.9581 (19)<br>6.96<br>6.97<br>6.98 (2)<br>6.9822<br>6.98396 (8)<br>6.99 (2)<br>Proportion:0.0025 (n=35)<br>Cumulative:0.9654\",\"Bin: [7, 7.05)<br>Observed:<br>7 (12)<br>7.00983 (9)<br>7.03 (2)<br>7.0357 (8)<br>7.04 (2)<br>Proportion:0.0024 (n=33)<br>Cumulative:0.9677\",\"Bin: [7.05, 7.1)<br>Observed:<br>7.05<br>7.06<br>7.06156 (7)<br>7.0692<br>7.07 (3)<br>7.08<br>7.08743 (15)<br>7.09<br>Proportion:0.0021 (n=30)<br>Cumulative:0.9699\",\"Bin: [7.1, 7.15)<br>Observed:<br>7.1 (7)<br>7.1133 (13)<br>7.1208 (2)<br>7.13916 (14)<br>7.14<br>Proportion:0.0026 (n=37)<br>Cumulative:0.9725\",\"Bin: [7.15, 7.2)<br>Observed:<br>7.15 (2)<br>7.16 (2)<br>7.16503 (14)<br>7.18908<br>7.19089 (5)<br>Proportion:0.0017 (n=24)<br>Cumulative:0.9742\",\"Bin: [7.2, 7.25)<br>Observed:<br>7.2 (5)<br>7.21 (2)<br>7.21494<br>7.21676 (6)<br>7.22<br>7.23 (3)<br>7.2408<br>7.24263 (12)<br>Proportion:0.0022 (n=31)<br>Cumulative:0.9765\",\"Bin: [7.25, 7.3)<br>Observed:<br>7.26849 (12)<br>7.29252<br>7.29436 (15)<br>Proportion:0.002 (n=28)<br>Cumulative:0.9785\",\"Bin: [7.3, 7.35)<br>Observed:<br>7.3 (7)<br>7.31<br>7.32<br>7.32023 (11)<br>7.34<br>7.34609 (9)<br>Proportion:0.0021 (n=30)<br>Cumulative:0.9806\",\"Bin: [7.35, 7.4)<br>Observed:<br>7.37<br>7.37196 (13)<br>7.38<br>7.39596<br>7.39783 (7)<br>Proportion:0.0016 (n=23)<br>Cumulative:0.9823\",\"Bin: [7.4, 7.45)<br>Observed:<br>7.4 (5)<br>7.41 (3)<br>7.42369 (5)<br>7.43<br>7.44956 (7)<br>Proportion:0.0015 (n=21)<br>Cumulative:0.9838\",\"Bin: [7.45, 7.5)<br>Observed:<br>7.47<br>7.47543 (2)<br>7.4994<br>Proportion:3e-04 (n=4)<br>Cumulative:0.984\",\"Bin: [7.5, 7.55)<br>Observed:<br>7.5 (5)<br>7.50129 (4)<br>7.51<br>7.52716 (4)<br>Proportion:0.001 (n=14)<br>Cumulative:0.985\",\"Bin: [7.55, 7.6)<br>Observed:<br>7.55<br>7.55303 (5)<br>7.5594 (2)<br>7.57<br>7.57889 (7)<br>7.5852<br>7.59<br>Proportion:0.0013 (n=18)<br>Cumulative:0.9863\",\"Bin: [7.6, 7.65)<br>Observed:<br>7.6 (5)<br>7.60476 (7)<br>7.62<br>7.63063 (4)<br>Proportion:0.0012 (n=17)<br>Cumulative:0.9876\",\"Bin: [7.65, 7.7)<br>Observed:<br>7.65649 (6)<br>7.66 (2)<br>7.67<br>7.68<br>7.68042<br>7.68236 (5)<br>7.69<br>Proportion:0.0012 (n=17)<br>Cumulative:0.9888\",\"Bin: [7.7, 7.75]<br>Observed:<br>7.7 (4)<br>7.70823<br>7.71 (2)<br>7.7142<br>7.73<br>7.73409 (5)<br>7.74 (2)<br>Proportion:0.0011 (n=16)<br>Cumulative:0.9899\",\"Q<sub>0.99<\\/sub>:7.75996<br>Bin: (7.75, 10.8639420589757]<br>Observed:<br>7.758 - 10.8639 (95 distinct)<br>Proportion:0.0101 (n=141)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 3.156
</font><font size="2"> 3.466 </font><font size="3"> 4.061
</font><font size="4"> <strong>4.785</strong> </font><font size="3">
5.535 </font><font size="2"> 6.273 </font><font size="1"> 6.751
</font></td>
</tr>
<tr class="odd">
<td class="gt_row gt_left" headers="Variable">plat</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">Platelets</td>
<td class="gt_row gt_right" headers="n">14423</td>
<td class="gt_row gt_right" headers="Missing">1561</td>
<td class="gt_row gt_right" headers="Distinct">479</td>
<td class="gt_row gt_right" headers="Info">1.000</td>
<td class="gt_row gt_right" headers="Mean">241.4</td>
<td class="gt_row gt_right" headers="Gmd">73.61</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-eaf5f892cfdb0496833a"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-eaf5f892cfdb0496833a">{"x":{"values":[166,40,46,38,54,52,62,66,89,100,111,158,180,181,183,220,227,276,329,356,356,367,450,443,462,468,453,462,496,503,480,503,441,419,412,392,373,333,331,318,316,283,222,221,216,186,150,166,142,120,96,90,101,69,76,73,72,43,45,41,36,32,23,17,12,19,14,146],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:91<br>Bin: [20, 95)<br>Observed:<br>20 - 94 (59 distinct)<br>Proportion:0.0115 (n=166)<br>Cumulative:0.0115\",\"Bin: [95, 100)<br>Observed:<br>95 (9)<br>96 (11)<br>97 (7)<br>98 (7)<br>99 (6)<br>Proportion:0.0028 (n=40)<br>Cumulative:0.0143\",\"Bin: [100, 105)<br>Observed:<br>100 (15)<br>101 (10)<br>102 (7)<br>103 (6)<br>104 (8)<br>Proportion:0.0032 (n=46)<br>Cumulative:0.0175\",\"Bin: [105, 110)<br>Observed:<br>105 (4)<br>106 (9)<br>107 (6)<br>108 (14)<br>109 (5)<br>Proportion:0.0026 (n=38)<br>Cumulative:0.0201\",\"Bin: [110, 115)<br>Observed:<br>110 (14)<br>111 (9)<br>112 (10)<br>113 (6)<br>114 (15)<br>Proportion:0.0037 (n=54)<br>Cumulative:0.0239\",\"Bin: [115, 120)<br>Observed:<br>115 (9)<br>116 (13)<br>117 (11)<br>118 (9)<br>119 (10)<br>Proportion:0.0036 (n=52)<br>Cumulative:0.0275\",\"Bin: [120, 125)<br>Observed:<br>120 (7)<br>121 (15)<br>122 (15)<br>123 (13)<br>124 (12)<br>Proportion:0.0043 (n=62)<br>Cumulative:0.0318\",\"Bin: [125, 130)<br>Observed:<br>125 (7)<br>126 (11)<br>127 (18)<br>128 (12)<br>129 (18)<br>Proportion:0.0046 (n=66)<br>Cumulative:0.0363\",\"Bin: [130, 135)<br>Observed:<br>130 (18)<br>131 (14)<br>132 (21)<br>133 (21)<br>134 (15)<br>Proportion:0.0062 (n=89)<br>Cumulative:0.0425\",\"Bin: [135, 140)<br>Observed:<br>135 (17)<br>136 (21)<br>137 (25)<br>138 (15)<br>139 (22)<br>Proportion:0.0069 (n=100)<br>Cumulative:0.0494\",\"Bin: [140, 145)<br>Observed:<br>140 (22)<br>141 (14)<br>142 (31)<br>143 (24)<br>144 (20)<br>Proportion:0.0077 (n=111)<br>Cumulative:0.0571\",\"Bin: [145, 150)<br>Observed:<br>145 (33)<br>146 (25)<br>147 (36)<br>148 (31)<br>149 (33)<br>Proportion:0.011 (n=158)<br>Cumulative:0.0681\",\"Bin: [150, 155)<br>Observed:<br>150 (29)<br>151 (28)<br>152 (37)<br>153 (51)<br>154 (35)<br>Proportion:0.0125 (n=180)<br>Cumulative:0.0806\",\"Bin: [155, 160)<br>Observed:<br>155 (41)<br>156 (27)<br>157 (32)<br>158 (38)<br>159 (43)<br>Proportion:0.0125 (n=181)<br>Cumulative:0.0931\",\"Bin: [160, 165)<br>Observed:<br>160 (38)<br>161 (36)<br>162 (27)<br>163 (36)<br>164 (46)<br>Proportion:0.0127 (n=183)<br>Cumulative:0.1058\",\"Bin: [165, 170)<br>Observed:<br>165 (44)<br>166 (54)<br>167 (57)<br>168 (39)<br>169 (26)<br>Proportion:0.0153 (n=220)<br>Cumulative:0.1211\",\"Bin: [170, 175)<br>Observed:<br>170 (32)<br>171 (55)<br>172 (42)<br>173 (54)<br>174 (44)<br>Proportion:0.0157 (n=227)<br>Cumulative:0.1368\",\"Bin: [175, 180)<br>Observed:<br>175 (54)<br>176 (56)<br>177 (40)<br>178 (55)<br>179 (71)<br>Proportion:0.0191 (n=276)<br>Cumulative:0.1559\",\"Bin: [180, 185)<br>Observed:<br>180 (59)<br>181 (68)<br>182 (64)<br>183 (77)<br>184 (61)<br>Proportion:0.0228 (n=329)<br>Cumulative:0.1787\",\"Bin: [185, 190)<br>Observed:<br>185 (81)<br>186 (89)<br>187 (55)<br>188 (62)<br>189 (69)<br>Proportion:0.0247 (n=356)<br>Cumulative:0.2034\",\"Bin: [190, 195)<br>Observed:<br>190 (74)<br>191 (63)<br>192 (70)<br>193 (70)<br>194 (79)<br>Proportion:0.0247 (n=356)<br>Cumulative:0.2281\",\"Bin: [195, 200)<br>Observed:<br>195 (61)<br>196 (68)<br>197 (83)<br>198 (81)<br>199 (74)<br>Proportion:0.0254 (n=367)<br>Cumulative:0.2536\",\"Bin: [200, 205)<br>Observed:<br>200 (86)<br>201 (86)<br>202 (92)<br>203 (91)<br>204 (95)<br>Proportion:0.0312 (n=450)<br>Cumulative:0.2848\",\"Bin: [205, 210)<br>Observed:<br>205 (88)<br>206 (95)<br>207 (77)<br>208 (87)<br>209 (96)<br>Proportion:0.0307 (n=443)<br>Cumulative:0.3155\",\"Bin: [210, 215)<br>Observed:<br>210 (86)<br>211 (83)<br>212 (98)<br>213 (97)<br>214 (98)<br>Proportion:0.032 (n=462)<br>Cumulative:0.3475\",\"Bin: [215, 220)<br>Observed:<br>215 (103)<br>216 (96)<br>217 (94)<br>218 (79)<br>219 (96)<br>Proportion:0.0324 (n=468)<br>Cumulative:0.3799\",\"Bin: [220, 225)<br>Observed:<br>220 (89)<br>221 (88)<br>222 (94)<br>223 (91)<br>224 (91)<br>Proportion:0.0314 (n=453)<br>Cumulative:0.4114\",\"Bin: [225, 230)<br>Observed:<br>225 (101)<br>226 (94)<br>227 (74)<br>228 (99)<br>229 (94)<br>Proportion:0.032 (n=462)<br>Cumulative:0.4434\",\"Bin: [230, 235)<br>Observed:<br>230 (100)<br>231 (108)<br>232 (112)<br>233 (106)<br>234 (70)<br>Proportion:0.0344 (n=496)<br>Cumulative:0.4778\",\"Bin: [235, 240)<br>Observed:<br>235 (97)<br>236 (100)<br>237 (116)<br>238 (94)<br>239 (96)<br>Proportion:0.0349 (n=503)<br>Cumulative:0.5127\",\"Bin: [240, 245)<br>Observed:<br>240 (104)<br>241 (100)<br>242 (95)<br>243 (96)<br>244 (85)<br>Proportion:0.0333 (n=480)<br>Cumulative:0.5459\",\"Bin: [245, 250)<br>Observed:<br>245 (112)<br>246 (103)<br>247 (84)<br>248 (107)<br>249 (97)<br>Proportion:0.0349 (n=503)<br>Cumulative:0.5808\",\"Bin: [250, 255)<br>Observed:<br>250 (92)<br>251 (98)<br>252 (79)<br>253 (86)<br>254 (86)<br>Proportion:0.0306 (n=441)<br>Cumulative:0.6114\",\"Bin: [255, 260)<br>Observed:<br>255 (83)<br>256 (96)<br>257 (82)<br>258 (85)<br>259 (73)<br>Proportion:0.0291 (n=419)<br>Cumulative:0.6404\",\"Bin: [260, 265)<br>Observed:<br>260 (79)<br>261 (63)<br>262 (95)<br>263 (101)<br>264 (74)<br>Proportion:0.0286 (n=412)<br>Cumulative:0.669\",\"Bin: [265, 270)<br>Observed:<br>265 (72)<br>266 (77)<br>267 (85)<br>268 (82)<br>269 (76)<br>Proportion:0.0272 (n=392)<br>Cumulative:0.6962\",\"Bin: [270, 275)<br>Observed:<br>270 (80)<br>271 (73)<br>272 (77)<br>273 (74)<br>274 (69)<br>Proportion:0.0259 (n=373)<br>Cumulative:0.722\",\"Bin: [275, 280)<br>Observed:<br>275 (53)<br>276 (71)<br>277 (73)<br>278 (62)<br>279 (74)<br>Proportion:0.0231 (n=333)<br>Cumulative:0.7451\",\"Bin: [280, 285)<br>Observed:<br>280 (63)<br>281 (80)<br>282 (69)<br>283 (60)<br>284 (59)<br>Proportion:0.0229 (n=331)<br>Cumulative:0.7681\",\"Bin: [285, 290)<br>Observed:<br>285 (53)<br>286 (65)<br>287 (66)<br>288 (68)<br>289 (66)<br>Proportion:0.022 (n=318)<br>Cumulative:0.7901\",\"Bin: [290, 295)<br>Observed:<br>290 (64)<br>291 (56)<br>292 (74)<br>293 (59)<br>294 (63)<br>Proportion:0.0219 (n=316)<br>Cumulative:0.812\",\"Bin: [295, 300)<br>Observed:<br>295 (60)<br>296 (53)<br>297 (53)<br>298 (65)<br>299 (52)<br>Proportion:0.0196 (n=283)<br>Cumulative:0.8317\",\"Bin: [300, 305)<br>Observed:<br>300 (52)<br>301 (42)<br>302 (35)<br>303 (49)<br>304 (44)<br>Proportion:0.0154 (n=222)<br>Cumulative:0.847\",\"Bin: [305, 310)<br>Observed:<br>305 (52)<br>306 (35)<br>307 (47)<br>308 (49)<br>309 (38)<br>Proportion:0.0153 (n=221)<br>Cumulative:0.8624\",\"Bin: [310, 315)<br>Observed:<br>310 (49)<br>311 (44)<br>312 (40)<br>313 (41)<br>314 (42)<br>Proportion:0.015 (n=216)<br>Cumulative:0.8773\",\"Bin: [315, 320)<br>Observed:<br>315 (42)<br>316 (35)<br>317 (40)<br>318 (42)<br>319 (27)<br>Proportion:0.0129 (n=186)<br>Cumulative:0.8902\",\"Bin: [320, 325)<br>Observed:<br>320 (34)<br>321 (25)<br>322 (34)<br>323 (30)<br>324 (27)<br>Proportion:0.0104 (n=150)<br>Cumulative:0.9006\",\"Bin: [325, 330)<br>Observed:<br>325 (41)<br>326 (29)<br>327 (27)<br>328 (33)<br>329 (36)<br>Proportion:0.0115 (n=166)<br>Cumulative:0.9122\",\"Bin: [330, 335)<br>Observed:<br>330 (26)<br>331 (27)<br>332 (36)<br>333 (27)<br>334 (26)<br>Proportion:0.0098 (n=142)<br>Cumulative:0.922\",\"Bin: [335, 340)<br>Observed:<br>335 (24)<br>336 (26)<br>337 (22)<br>338 (33)<br>339 (15)<br>Proportion:0.0083 (n=120)<br>Cumulative:0.9303\",\"Bin: [340, 345)<br>Observed:<br>340 (19)<br>341 (18)<br>342 (16)<br>343 (18)<br>344 (25)<br>Proportion:0.0067 (n=96)<br>Cumulative:0.937\",\"Bin: [345, 350)<br>Observed:<br>345 (21)<br>346 (22)<br>347 (13)<br>348 (16)<br>349 (18)<br>Proportion:0.0062 (n=90)<br>Cumulative:0.9432\",\"Bin: [350, 355)<br>Observed:<br>350 (21)<br>351 (24)<br>352 (18)<br>353 (20)<br>354 (18)<br>Proportion:0.007 (n=101)<br>Cumulative:0.9502\",\"Bin: [355, 360)<br>Observed:<br>355 (12)<br>356 (11)<br>357 (20)<br>358 (10)<br>359 (16)<br>Proportion:0.0048 (n=69)<br>Cumulative:0.955\",\"Bin: [360, 365)<br>Observed:<br>360 (15)<br>361 (19)<br>362 (18)<br>363 (12)<br>364 (12)<br>Proportion:0.0053 (n=76)<br>Cumulative:0.9603\",\"Bin: [365, 370)<br>Observed:<br>365 (13)<br>366 (19)<br>367 (14)<br>368 (15)<br>369 (12)<br>Proportion:0.0051 (n=73)<br>Cumulative:0.9653\",\"Bin: [370, 375)<br>Observed:<br>370 (18)<br>371 (11)<br>372 (20)<br>373 (7)<br>374 (16)<br>Proportion:0.005 (n=72)<br>Cumulative:0.9703\",\"Bin: [375, 380)<br>Observed:<br>375 (8)<br>376 (12)<br>377 (7)<br>378 (4)<br>379 (12)<br>Proportion:0.003 (n=43)<br>Cumulative:0.9733\",\"Bin: [380, 385)<br>Observed:<br>380 (7)<br>381 (10)<br>382 (6)<br>383 (15)<br>384 (7)<br>Proportion:0.0031 (n=45)<br>Cumulative:0.9764\",\"Bin: [385, 390)<br>Observed:<br>385 (6)<br>386 (11)<br>387 (7)<br>388 (11)<br>389 (6)<br>Proportion:0.0028 (n=41)<br>Cumulative:0.9793\",\"Bin: [390, 395)<br>Observed:<br>390 (6)<br>391 (9)<br>392 (9)<br>393 (4)<br>394 (8)<br>Proportion:0.0025 (n=36)<br>Cumulative:0.9818\",\"Bin: [395, 400)<br>Observed:<br>395 (3)<br>396 (7)<br>397 (5)<br>398 (10)<br>399 (7)<br>Proportion:0.0022 (n=32)<br>Cumulative:0.984\",\"Bin: [400, 405)<br>Observed:<br>400 (5)<br>401 (5)<br>402 (7)<br>403 (2)<br>404 (4)<br>Proportion:0.0016 (n=23)<br>Cumulative:0.9856\",\"Bin: [405, 410)<br>Observed:<br>405<br>406 (5)<br>407 (4)<br>408 (6)<br>409<br>Proportion:0.0012 (n=17)<br>Cumulative:0.9868\",\"Bin: [410, 415)<br>Observed:<br>410 (2)<br>411<br>412 (8)<br>414<br>Proportion:8e-04 (n=12)<br>Cumulative:0.9876\",\"Bin: [415, 420)<br>Observed:<br>415 (2)<br>416 (2)<br>417 (5)<br>418 (6)<br>419 (4)<br>Proportion:0.0013 (n=19)<br>Cumulative:0.9889\",\"Bin: [420, 425]<br>Observed:<br>420 (5)<br>421 (3)<br>422 (2)<br>425 (4)<br>Proportion:0.001 (n=14)<br>Cumulative:0.9899\",\"Q<sub>0.99<\\/sub>:426<br>Bin: (425, 1118]<br>Observed:<br>426 - 1118 (92 distinct)<br>Proportion:0.0101 (n=146)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 140
</font><font size="2"> 162 </font><font size="3"> 199
</font><font size="4"> <strong>238</strong> </font><font size="3"> 281
</font><font size="2"> 324 </font><font size="1"> 354 </font></td>
</tr>
<tr class="even">
<td class="gt_row gt_left" headers="Variable">lsm</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">Liver Stiffness Measure (kPa)</td>
<td class="gt_row gt_right" headers="n">15984</td>
<td class="gt_row gt_right" headers="Missing">0</td>
<td class="gt_row gt_right" headers="Distinct">384</td>
<td class="gt_row gt_right" headers="Info">1.000</td>
<td class="gt_row gt_right" headers="Mean">7.743</td>
<td class="gt_row gt_right" headers="Gmd">4.991</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-4fcde13d436fa0df8db6"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-4fcde13d436fa0df8db6">{"x":{"values":[208,523,1035,1703,1795,1620,1219,1305,1064,594,687,464,547,297,222,306,163,162,252,171,88,83,110,148,74,65,43,59,78,58,52,40,29,20,23,21,34,49,38,30,18,20,10,12,10,15,16,46,14,27,10,14,6,8,6,3,8,9,6,11,4,11,5,11,13,18,6,168],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:2.8<br>Bin: [0.9, 3)<br>Observed:<br>0.9 - 2.9 (14 distinct)<br>Proportion:0.013 (n=208)<br>Cumulative:0.013\",\"Bin: [3, 3.5)<br>Observed:<br>3 (79)<br>3.1 (59)<br>3.2 (91)<br>3.3 (173)<br>3.4 (121)<br>Proportion:0.0327 (n=523)<br>Cumulative:0.0457\",\"Bin: [3.5, 4)<br>Observed:<br>3.5 (219)<br>3.6 (207)<br>3.7 (166)<br>3.8 (250)<br>3.9 (193)<br>Proportion:0.0648 (n=1035)<br>Cumulative:0.1105\",\"Bin: [4, 4.5)<br>Observed:<br>4 (321)<br>4.1 (287)<br>4.2 (209)<br>4.3 (488)<br>4.4 (398)<br>Proportion:0.1065 (n=1703)<br>Cumulative:0.217\",\"Bin: [4.5, 5)<br>Observed:<br>4.5 (276)<br>4.6 (397)<br>4.7 (254)<br>4.8 (442)<br>4.9 (426)<br>Proportion:0.1123 (n=1795)<br>Cumulative:0.3293\",\"Bin: [5, 5.5)<br>Observed:<br>5 (228)<br>5.1 (327)<br>5.2 (207)<br>5.3 (524)<br>5.4 (334)<br>Proportion:0.1014 (n=1620)<br>Cumulative:0.4307\",\"Bin: [5.5, 6)<br>Observed:<br>5.5 (232)<br>5.6 (292)<br>5.7 (165)<br>5.8 (248)<br>5.9 (282)<br>Proportion:0.0763 (n=1219)<br>Cumulative:0.5069\",\"Bin: [6, 6.5)<br>Observed:<br>6 (229)<br>6.1 (435)<br>6.2 (170)<br>6.3 (247)<br>6.4 (224)<br>Proportion:0.0816 (n=1305)<br>Cumulative:0.5886\",\"Bin: [6.5, 7)<br>Observed:<br>6.5 (165)<br>6.6 (185)<br>6.7 (264)<br>6.8 (287)<br>6.9 (163)<br>Proportion:0.0666 (n=1064)<br>Cumulative:0.6552\",\"Bin: [7, 7.5)<br>Observed:<br>7 (107)<br>7.1 (141)<br>7.2 (106)<br>7.3 (119)<br>7.4 (121)<br>Proportion:0.0372 (n=594)<br>Cumulative:0.6923\",\"Bin: [7.5, 8)<br>Observed:<br>7.5 (109)<br>7.6 (154)<br>7.7 (154)<br>7.8 (152)<br>7.9 (118)<br>Proportion:0.043 (n=687)<br>Cumulative:0.7353\",\"Bin: [8, 8.5)<br>Observed:<br>8 (117)<br>8.1 (111)<br>8.2 (78)<br>8.3 (88)<br>8.4 (70)<br>Proportion:0.029 (n=464)<br>Cumulative:0.7643\",\"Bin: [8.5, 9)<br>Observed:<br>8.5 (56)<br>8.6 (104)<br>8.7 (122)<br>8.8 (174)<br>8.9 (91)<br>Proportion:0.0342 (n=547)<br>Cumulative:0.7985\",\"Bin: [9, 9.5)<br>Observed:<br>9 (62)<br>9.1 (67)<br>9.2 (48)<br>9.23<br>9.3 (61)<br>9.4 (58)<br>Proportion:0.0186 (n=297)<br>Cumulative:0.8171\",\"Bin: [9.5, 10)<br>Observed:<br>9.5 (51)<br>9.6 (47)<br>9.7 (38)<br>9.8 (36)<br>9.9 (50)<br>Proportion:0.0139 (n=222)<br>Cumulative:0.831\",\"Bin: [10, 10.5)<br>Observed:<br>10 (64)<br>10.1 (60)<br>10.2 (81)<br>10.3 (44)<br>10.4 (57)<br>Proportion:0.0191 (n=306)<br>Cumulative:0.8502\",\"Bin: [10.5, 11)<br>Observed:<br>10.5 (49)<br>10.6 (26)<br>10.7 (29)<br>10.8 (38)<br>10.9 (21)<br>Proportion:0.0102 (n=163)<br>Cumulative:0.8604\",\"Bin: [11, 11.5)<br>Observed:<br>11 (34)<br>11.1 (25)<br>11.2 (36)<br>11.3 (33)<br>11.4 (34)<br>Proportion:0.0101 (n=162)<br>Cumulative:0.8705\",\"Bin: [11.5, 12)<br>Observed:<br>11.5 (39)<br>11.6 (45)<br>11.7 (25)<br>11.8 (97)<br>11.9 (46)<br>Proportion:0.0158 (n=252)<br>Cumulative:0.8863\",\"Bin: [12, 12.5)<br>Observed:<br>12 (71)<br>12.1 (24)<br>12.2 (25)<br>12.3 (30)<br>12.4 (21)<br>Proportion:0.0107 (n=171)<br>Cumulative:0.897\",\"Bin: [12.5, 13)<br>Observed:<br>12.5 (20)<br>12.6 (29)<br>12.7 (9)<br>12.8 (15)<br>12.9 (15)<br>Proportion:0.0055 (n=88)<br>Cumulative:0.9025\",\"Bin: [13, 13.5)<br>Observed:<br>13 (16)<br>13.1 (17)<br>13.2 (12)<br>13.3 (22)<br>13.4 (16)<br>Proportion:0.0052 (n=83)<br>Cumulative:0.9077\",\"Bin: [13.5, 14)<br>Observed:<br>13.5 (18)<br>13.6 (19)<br>13.7 (14)<br>13.8 (36)<br>13.9 (23)<br>Proportion:0.0069 (n=110)<br>Cumulative:0.9145\",\"Bin: [14, 14.5)<br>Observed:<br>14 (33)<br>14.1 (23)<br>14.2 (15)<br>14.3 (51)<br>14.4 (26)<br>Proportion:0.0093 (n=148)<br>Cumulative:0.9238\",\"Bin: [14.5, 15)<br>Observed:<br>14.5 (14)<br>14.6 (15)<br>14.7 (16)<br>14.8 (19)<br>14.9 (10)<br>Proportion:0.0046 (n=74)<br>Cumulative:0.9284\",\"Bin: [15, 15.5)<br>Observed:<br>15 (16)<br>15.1 (16)<br>15.2 (5)<br>15.3 (12)<br>15.4 (16)<br>Proportion:0.0041 (n=65)<br>Cumulative:0.9325\",\"Bin: [15.5, 16)<br>Observed:<br>15.5 (7)<br>15.6 (11)<br>15.7 (12)<br>15.8 (3)<br>15.9 (10)<br>Proportion:0.0027 (n=43)<br>Cumulative:0.9352\",\"Bin: [16, 16.5)<br>Observed:<br>16 (16)<br>16.1 (10)<br>16.2 (8)<br>16.3 (19)<br>16.4 (6)<br>Proportion:0.0037 (n=59)<br>Cumulative:0.9389\",\"Bin: [16.5, 17)<br>Observed:<br>16.5 (10)<br>16.6 (18)<br>16.7 (4)<br>16.8 (18)<br>16.9 (28)<br>Proportion:0.0049 (n=78)<br>Cumulative:0.9438\",\"Bin: [17, 17.5)<br>Observed:<br>17 (4)<br>17.1 (24)<br>17.2 (2)<br>17.3 (27)<br>17.4<br>Proportion:0.0036 (n=58)<br>Cumulative:0.9474\",\"Bin: [17.5, 18)<br>Observed:<br>17.5 (12)<br>17.6 (17)<br>17.7 (6)<br>17.8 (12)<br>17.9 (5)<br>Proportion:0.0033 (n=52)<br>Cumulative:0.9506\",\"Bin: [18, 18.5)<br>Observed:<br>18 (9)<br>18.1 (2)<br>18.2 (9)<br>18.3 (5)<br>18.4 (15)<br>Proportion:0.0025 (n=40)<br>Cumulative:0.9531\",\"Bin: [18.5, 19)<br>Observed:<br>18.5 (4)<br>18.6 (8)<br>18.7<br>18.8 (12)<br>18.9 (4)<br>Proportion:0.0018 (n=29)<br>Cumulative:0.955\",\"Bin: [19, 19.5)<br>Observed:<br>19 (4)<br>19.1 (7)<br>19.2 (4)<br>19.4 (5)<br>Proportion:0.0013 (n=20)<br>Cumulative:0.9562\",\"Bin: [19.5, 20)<br>Observed:<br>19.5 (5)<br>19.6 (10)<br>19.8 (7)<br>19.9<br>Proportion:0.0014 (n=23)<br>Cumulative:0.9576\",\"Bin: [20, 20.5)<br>Observed:<br>20 (7)<br>20.1 (2)<br>20.2 (3)<br>20.3 (3)<br>20.4 (6)<br>Proportion:0.0013 (n=21)<br>Cumulative:0.959\",\"Bin: [20.5, 21)<br>Observed:<br>20.5 (4)<br>20.6 (4)<br>20.7 (8)<br>20.8 (5)<br>20.9 (13)<br>Proportion:0.0021 (n=34)<br>Cumulative:0.9611\",\"Bin: [21, 21.5)<br>Observed:<br>21<br>21.1 (12)<br>21.2<br>21.3 (34)<br>21.4<br>Proportion:0.0031 (n=49)<br>Cumulative:0.9642\",\"Bin: [21.5, 22)<br>Observed:<br>21.5 (8)<br>21.6 (14)<br>21.7 (2)<br>21.8 (13)<br>21.9<br>Proportion:0.0024 (n=38)<br>Cumulative:0.9665\",\"Bin: [22, 22.5)<br>Observed:<br>22 (10)<br>22.1 (5)<br>22.2<br>22.3 (11)<br>22.4 (3)<br>Proportion:0.0019 (n=30)<br>Cumulative:0.9684\",\"Bin: [22.5, 23)<br>Observed:<br>22.5 (2)<br>22.6 (9)<br>22.8 (6)<br>22.9<br>Proportion:0.0011 (n=18)<br>Cumulative:0.9695\",\"Bin: [23, 23.5)<br>Observed:<br>23.1 (6)<br>23.2 (6)<br>23.3 (3)<br>23.4 (5)<br>Proportion:0.0013 (n=20)<br>Cumulative:0.9708\",\"Bin: [23.5, 24)<br>Observed:<br>23.5 (3)<br>23.6<br>23.7<br>23.8<br>23.9 (4)<br>Proportion:6e-04 (n=10)<br>Cumulative:0.9714\",\"Bin: [24, 24.5)<br>Observed:<br>24 (3)<br>24.2 (2)<br>24.3 (3)<br>24.4 (4)<br>Proportion:8e-04 (n=12)<br>Cumulative:0.9722\",\"Bin: [24.5, 25)<br>Observed:<br>24.5 (2)<br>24.6<br>24.7<br>24.8 (5)<br>24.9<br>Proportion:6e-04 (n=10)<br>Cumulative:0.9728\",\"Bin: [25, 25.5)<br>Observed:<br>25 (3)<br>25.1 (5)<br>25.2 (2)<br>25.4 (5)<br>Proportion:9e-04 (n=15)<br>Cumulative:0.9737\",\"Bin: [25.5, 26)<br>Observed:<br>25.5<br>25.6 (5)<br>25.7 (9)<br>25.8<br>Proportion:0.001 (n=16)<br>Cumulative:0.9747\",\"Bin: [26, 26.5)<br>Observed:<br>26 (11)<br>26.1 (2)<br>26.2 (3)<br>26.3 (29)<br>26.4<br>Proportion:0.0029 (n=46)<br>Cumulative:0.9776\",\"Bin: [26.5, 27)<br>Observed:<br>26.6 (6)<br>26.7 (7)<br>26.9<br>Proportion:9e-04 (n=14)<br>Cumulative:0.9785\",\"Bin: [27, 27.5)<br>Observed:<br>27 (23)<br>27.1 (3)<br>27.4<br>Proportion:0.0017 (n=27)<br>Cumulative:0.9802\",\"Bin: [27.5, 28)<br>Observed:<br>27.6<br>27.7 (9)<br>Proportion:6e-04 (n=10)<br>Cumulative:0.9808\",\"Bin: [28, 28.5)<br>Observed:<br>28 (6)<br>28.1<br>28.4 (7)<br>Proportion:9e-04 (n=14)<br>Cumulative:0.9817\",\"Bin: [28.5, 29)<br>Observed:<br>28.5 (2)<br>28.8 (4)<br>Proportion:4e-04 (n=6)<br>Cumulative:0.982\",\"Bin: [29, 29.5)<br>Observed:<br>29<br>29.1<br>29.2 (3)<br>29.3 (2)<br>29.4<br>Proportion:5e-04 (n=8)<br>Cumulative:0.9825\",\"Bin: [29.5, 30)<br>Observed:<br>29.5<br>29.7 (2)<br>29.8 (2)<br>29.9<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9829\",\"Bin: [30, 30.5)<br>Observed:<br>30.3 (2)<br>30.4<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9831\",\"Bin: [30.5, 31)<br>Observed:<br>30.5<br>30.6 (2)<br>30.7 (3)<br>30.8 (2)<br>Proportion:5e-04 (n=8)<br>Cumulative:0.9836\",\"Bin: [31, 31.5)<br>Observed:<br>31<br>31.2 (6)<br>31.3<br>31.4<br>Proportion:6e-04 (n=9)<br>Cumulative:0.9842\",\"Bin: [31.5, 32)<br>Observed:<br>31.6 (4)<br>31.8 (2)<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9845\",\"Bin: [32, 32.5)<br>Observed:<br>32 (5)<br>32.14<br>32.2 (2)<br>32.3<br>32.4 (2)<br>Proportion:7e-04 (n=11)<br>Cumulative:0.9852\",\"Bin: [32.5, 33)<br>Observed:<br>32.5 (2)<br>32.9 (2)<br>Proportion:3e-04 (n=4)<br>Cumulative:0.9855\",\"Bin: [33, 33.5)<br>Observed:<br>33 (6)<br>33.3 (2)<br>33.4 (3)<br>Proportion:7e-04 (n=11)<br>Cumulative:0.9862\",\"Bin: [33.5, 34)<br>Observed:<br>33.5<br>33.8 (4)<br>Proportion:3e-04 (n=5)<br>Cumulative:0.9865\",\"Bin: [34, 34.5)<br>Observed:<br>34<br>34.1<br>34.3 (9)<br>Proportion:7e-04 (n=11)<br>Cumulative:0.9872\",\"Bin: [34.5, 35)<br>Observed:<br>34.8 (13)<br>Proportion:8e-04 (n=13)<br>Cumulative:0.988\",\"Bin: [35, 35.5)<br>Observed:<br>35<br>35.3 (17)<br>Proportion:0.0011 (n=18)<br>Cumulative:0.9891\",\"Bin: [35.5, 36]<br>Observed:<br>35.5<br>35.6 (2)<br>35.8 (3)<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9895\",\"Q<sub>0.99<\\/sub>:36.5<br>Bin: (36, 75]<br>Observed:<br>36.1 - 75 (86 distinct)<br>Proportion:0.0105 (n=168)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 3.5
</font><font size="2"> 3.9 </font><font size="3"> 4.6
</font><font size="4"> <strong>5.9</strong> </font><font size="3"> 8.2
</font><font size="2"> 12.6 </font><font size="1"> 17.8 </font></td>
</tr>
<tr class="odd">
<td class="gt_row gt_left" headers="Variable">iqr</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">Inter-quartile range of LSM</td>
<td class="gt_row gt_right" headers="n">15126</td>
<td class="gt_row gt_right" headers="Missing">858</td>
<td class="gt_row gt_right" headers="Distinct">177</td>
<td class="gt_row gt_right" headers="Info">0.995</td>
<td class="gt_row gt_right" headers="Mean">1.112</td>
<td class="gt_row gt_right" headers="Gmd">1.032</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-84dd044e91b855b747ff"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-84dd044e91b855b747ff">{"x":{"values":[257,0,633,1,1334,2,1446,0,1440,2,1322,0,1160,1,1162,1,907,1,738,1,658,0,494,0,461,2,348,2,265,0,259,0,213,1,188,0,142,0,134,0,123,0,111,0,100,0,106,0,66,0,68,0,61,0,60,0,58,0,43,0,46,0,32,0,37,0,34,0,22,0,26,0,19,0,31,0,20,0,23,0,16,0,18,0,21,0,16,0,26,0,12,0,14,0,9,0,16,0,19,1,9,0,7,0,10,0,6,0,8,0,6,0,10,0,8,0,9,0,5,0,3,0,8,0,6,0,3,0,8,0,3,0,7,0,3,0,13,10,155],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:0.1<br>Bin: [0.095, 0.15)<br>Observed:<br>0.095<br>0.1 (255)<br>0.13<br>Proportion:0.017 (n=257)<br>Cumulative:0.017\",\"\",\"Bin: [0.2, 0.25)<br>Observed:<br>0.2 (633)<br>Proportion:0.0418 (n=633)<br>Cumulative:0.0588\",\"Bin: [0.25, 0.3)<br>Observed:<br>0.27<br>Proportion:1e-04 (n=1)<br>Cumulative:0.0589\",\"Bin: [0.3, 0.35)<br>Observed:<br>0.3 (1332)<br>0.301<br>0.34<br>Proportion:0.0882 (n=1334)<br>Cumulative:0.1471\",\"Bin: [0.35, 0.4)<br>Observed:<br>0.38<br>0.396<br>Proportion:1e-04 (n=2)<br>Cumulative:0.1472\",\"Bin: [0.4, 0.45)<br>Observed:<br>0.4 (1445)<br>0.414<br>Proportion:0.0956 (n=1446)<br>Cumulative:0.2428\",\"\",\"Bin: [0.5, 0.55)<br>Observed:<br>0.5 (1436)<br>0.512<br>0.53<br>0.54 (2)<br>Proportion:0.0952 (n=1440)<br>Cumulative:0.338\",\"Bin: [0.55, 0.6)<br>Observed:<br>0.576<br>0.58<br>Proportion:1e-04 (n=2)<br>Cumulative:0.3382\",\"Bin: [0.6, 0.65)<br>Observed:<br>0.6 (1322)<br>Proportion:0.0874 (n=1322)<br>Cumulative:0.4256\",\"\",\"Bin: [0.7, 0.75)<br>Observed:<br>0.7 (1159)<br>0.708<br>Proportion:0.0767 (n=1160)<br>Cumulative:0.5022\",\"Bin: [0.75, 0.8)<br>Observed:<br>0.774<br>Proportion:1e-04 (n=1)<br>Cumulative:0.5023\",\"Bin: [0.8, 0.85)<br>Observed:<br>0.8 (1160)<br>0.81<br>0.816<br>Proportion:0.0768 (n=1162)<br>Cumulative:0.5791\",\"Bin: [0.85, 0.9)<br>Observed:<br>0.85<br>Proportion:1e-04 (n=1)<br>Cumulative:0.5792\",\"Bin: [0.9, 0.95)<br>Observed:<br>0.9 (907)<br>Proportion:0.06 (n=907)<br>Cumulative:0.6392\",\"Bin: [0.95, 1)<br>Observed:<br>0.95<br>Proportion:1e-04 (n=1)<br>Cumulative:0.6392\",\"Bin: [1, 1.05)<br>Observed:<br>1 (736)<br>1.001<br>1.02<br>Proportion:0.0488 (n=738)<br>Cumulative:0.688\",\"Bin: [1.05, 1.1)<br>Observed:<br>1.07<br>Proportion:1e-04 (n=1)<br>Cumulative:0.6881\",\"Bin: [1.1, 1.15)<br>Observed:<br>1.1 (657)<br>1.12<br>Proportion:0.0435 (n=658)<br>Cumulative:0.7316\",\"\",\"Bin: [1.2, 1.25)<br>Observed:<br>1.2 (494)<br>Proportion:0.0327 (n=494)<br>Cumulative:0.7642\",\"\",\"Bin: [1.3, 1.35)<br>Observed:<br>1.3 (461)<br>Proportion:0.0305 (n=461)<br>Cumulative:0.7947\",\"Bin: [1.35, 1.4)<br>Observed:<br>1.37<br>1.38<br>Proportion:1e-04 (n=2)<br>Cumulative:0.7949\",\"Bin: [1.4, 1.45)<br>Observed:<br>1.4 (348)<br>Proportion:0.023 (n=348)<br>Cumulative:0.8179\",\"Bin: [1.45, 1.5)<br>Observed:<br>1.456<br>1.47<br>Proportion:1e-04 (n=2)<br>Cumulative:0.818\",\"Bin: [1.5, 1.55)<br>Observed:<br>1.5 (265)<br>Proportion:0.0175 (n=265)<br>Cumulative:0.8355\",\"\",\"Bin: [1.6, 1.65)<br>Observed:<br>1.6 (259)<br>Proportion:0.0171 (n=259)<br>Cumulative:0.8526\",\"\",\"Bin: [1.7, 1.75)<br>Observed:<br>1.7 (212)<br>1.725<br>Proportion:0.0141 (n=213)<br>Cumulative:0.8667\",\"Bin: [1.75, 1.8)<br>Observed:<br>1.78<br>Proportion:1e-04 (n=1)<br>Cumulative:0.8668\",\"Bin: [1.8, 1.85)<br>Observed:<br>1.8 (187)<br>1.824<br>Proportion:0.0124 (n=188)<br>Cumulative:0.8792\",\"\",\"Bin: [1.9, 1.95)<br>Observed:<br>1.9 (142)<br>Proportion:0.0094 (n=142)<br>Cumulative:0.8886\",\"\",\"Bin: [2, 2.05)<br>Observed:<br>2 (133)<br>2.016<br>Proportion:0.0089 (n=134)<br>Cumulative:0.8975\",\"\",\"Bin: [2.1, 2.15)<br>Observed:<br>2.1 (123)<br>Proportion:0.0081 (n=123)<br>Cumulative:0.9056\",\"\",\"Bin: [2.2, 2.25)<br>Observed:<br>2.2 (111)<br>Proportion:0.0073 (n=111)<br>Cumulative:0.9129\",\"\",\"Bin: [2.3, 2.35)<br>Observed:<br>2.3 (100)<br>Proportion:0.0066 (n=100)<br>Cumulative:0.9195\",\"\",\"Bin: [2.4, 2.45)<br>Observed:<br>2.4 (106)<br>Proportion:0.007 (n=106)<br>Cumulative:0.9266\",\"\",\"Bin: [2.5, 2.55)<br>Observed:<br>2.5 (66)<br>Proportion:0.0044 (n=66)<br>Cumulative:0.9309\",\"\",\"Bin: [2.6, 2.65)<br>Observed:<br>2.6 (68)<br>Proportion:0.0045 (n=68)<br>Cumulative:0.9354\",\"\",\"Bin: [2.7, 2.75)<br>Observed:<br>2.7 (61)<br>Proportion:0.004 (n=61)<br>Cumulative:0.9394\",\"\",\"Bin: [2.8, 2.85)<br>Observed:<br>2.8 (60)<br>Proportion:0.004 (n=60)<br>Cumulative:0.9434\",\"\",\"Bin: [2.9, 2.95)<br>Observed:<br>2.9 (58)<br>Proportion:0.0038 (n=58)<br>Cumulative:0.9472\",\"\",\"Bin: [3, 3.05)<br>Observed:<br>3 (43)<br>Proportion:0.0028 (n=43)<br>Cumulative:0.9501\",\"\",\"Bin: [3.1, 3.15)<br>Observed:<br>3.1 (46)<br>Proportion:0.003 (n=46)<br>Cumulative:0.9531\",\"\",\"Bin: [3.2, 3.25)<br>Observed:<br>3.2 (32)<br>Proportion:0.0021 (n=32)<br>Cumulative:0.9552\",\"\",\"Bin: [3.3, 3.35)<br>Observed:<br>3.3 (37)<br>Proportion:0.0024 (n=37)<br>Cumulative:0.9577\",\"\",\"Bin: [3.4, 3.45)<br>Observed:<br>3.4 (34)<br>Proportion:0.0022 (n=34)<br>Cumulative:0.9599\",\"\",\"Bin: [3.5, 3.55)<br>Observed:<br>3.5 (22)<br>Proportion:0.0015 (n=22)<br>Cumulative:0.9614\",\"\",\"Bin: [3.6, 3.65)<br>Observed:<br>3.6 (26)<br>Proportion:0.0017 (n=26)<br>Cumulative:0.9631\",\"\",\"Bin: [3.7, 3.75)<br>Observed:<br>3.7 (19)<br>Proportion:0.0013 (n=19)<br>Cumulative:0.9644\",\"\",\"Bin: [3.8, 3.85)<br>Observed:<br>3.8 (31)<br>Proportion:0.002 (n=31)<br>Cumulative:0.9664\",\"\",\"Bin: [3.9, 3.95)<br>Observed:<br>3.9 (20)<br>Proportion:0.0013 (n=20)<br>Cumulative:0.9677\",\"\",\"Bin: [4, 4.05)<br>Observed:<br>4 (23)<br>Proportion:0.0015 (n=23)<br>Cumulative:0.9693\",\"\",\"Bin: [4.1, 4.15)<br>Observed:<br>4.1 (15)<br>4.147<br>Proportion:0.0011 (n=16)<br>Cumulative:0.9703\",\"\",\"Bin: [4.2, 4.25)<br>Observed:<br>4.2 (18)<br>Proportion:0.0012 (n=18)<br>Cumulative:0.9715\",\"\",\"Bin: [4.3, 4.35)<br>Observed:<br>4.3 (21)<br>Proportion:0.0014 (n=21)<br>Cumulative:0.9729\",\"\",\"Bin: [4.4, 4.45)<br>Observed:<br>4.4 (16)<br>Proportion:0.0011 (n=16)<br>Cumulative:0.974\",\"\",\"Bin: [4.5, 4.55)<br>Observed:<br>4.5 (26)<br>Proportion:0.0017 (n=26)<br>Cumulative:0.9757\",\"\",\"Bin: [4.6, 4.65)<br>Observed:<br>4.6 (12)<br>Proportion:8e-04 (n=12)<br>Cumulative:0.9765\",\"\",\"Bin: [4.7, 4.75)<br>Observed:<br>4.7 (14)<br>Proportion:9e-04 (n=14)<br>Cumulative:0.9774\",\"\",\"Bin: [4.8, 4.85)<br>Observed:<br>4.8 (9)<br>Proportion:6e-04 (n=9)<br>Cumulative:0.978\",\"\",\"Bin: [4.9, 4.95)<br>Observed:<br>4.9 (16)<br>Proportion:0.0011 (n=16)<br>Cumulative:0.979\",\"\",\"Bin: [5, 5.05)<br>Observed:<br>5 (19)<br>Proportion:0.0013 (n=19)<br>Cumulative:0.9803\",\"Bin: [5.05, 5.1)<br>Observed:<br>5.073<br>Proportion:1e-04 (n=1)<br>Cumulative:0.9804\",\"Bin: [5.1, 5.15)<br>Observed:<br>5.1 (9)<br>Proportion:6e-04 (n=9)<br>Cumulative:0.981\",\"\",\"Bin: [5.2, 5.25)<br>Observed:<br>5.2 (7)<br>Proportion:5e-04 (n=7)<br>Cumulative:0.9814\",\"\",\"Bin: [5.3, 5.35)<br>Observed:<br>5.3 (10)<br>Proportion:7e-04 (n=10)<br>Cumulative:0.9821\",\"\",\"Bin: [5.4, 5.45)<br>Observed:<br>5.4 (6)<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9825\",\"\",\"Bin: [5.5, 5.55)<br>Observed:<br>5.5 (8)<br>Proportion:5e-04 (n=8)<br>Cumulative:0.983\",\"\",\"Bin: [5.6, 5.65)<br>Observed:<br>5.6 (5)<br>5.648<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9834\",\"\",\"Bin: [5.7, 5.75)<br>Observed:<br>5.7 (10)<br>Proportion:7e-04 (n=10)<br>Cumulative:0.9841\",\"\",\"Bin: [5.8, 5.85)<br>Observed:<br>5.8 (8)<br>Proportion:5e-04 (n=8)<br>Cumulative:0.9846\",\"\",\"Bin: [5.9, 5.95)<br>Observed:<br>5.9 (9)<br>Proportion:6e-04 (n=9)<br>Cumulative:0.9852\",\"\",\"Bin: [6, 6.05)<br>Observed:<br>6 (5)<br>Proportion:3e-04 (n=5)<br>Cumulative:0.9855\",\"\",\"Bin: [6.1, 6.15)<br>Observed:<br>6.1 (3)<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9857\",\"\",\"Bin: [6.2, 6.25)<br>Observed:<br>6.2 (8)<br>Proportion:5e-04 (n=8)<br>Cumulative:0.9862\",\"\",\"Bin: [6.3, 6.35)<br>Observed:<br>6.3 (6)<br>Proportion:4e-04 (n=6)<br>Cumulative:0.9866\",\"\",\"Bin: [6.4, 6.45)<br>Observed:<br>6.4 (3)<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9868\",\"\",\"Bin: [6.5, 6.55)<br>Observed:<br>6.5 (8)<br>Proportion:5e-04 (n=8)<br>Cumulative:0.9874\",\"\",\"Bin: [6.6, 6.65)<br>Observed:<br>6.6 (3)<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9876\",\"\",\"Bin: [6.7, 6.75)<br>Observed:<br>6.7 (7)<br>Proportion:5e-04 (n=7)<br>Cumulative:0.988\",\"\",\"Bin: [6.8, 6.85)<br>Observed:<br>6.8 (3)<br>Proportion:2e-04 (n=3)<br>Cumulative:0.9882\",\"\",\"Bin: [6.9, 6.95)<br>Observed:<br>6.9 (13)<br>Proportion:9e-04 (n=13)<br>Cumulative:0.9891\",\"Bin: [6.95, 7]<br>Observed:<br>7 (10)<br>Proportion:7e-04 (n=10)<br>Cumulative:0.9898\",\"Q<sub>0.99<\\/sub>:7.1<br>Bin: (7, 50.3]<br>Observed:<br>7.1 - 50.3 (73 distinct)<br>Proportion:0.0102 (n=155)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 0.2
</font><font size="2"> 0.3 </font><font size="3"> 0.5
</font><font size="4"> <strong>0.7</strong> </font><font size="3"> 1.2
</font><font size="2"> 2.1 </font><font size="1"> 3.0 </font></td>
</tr>
<tr class="even">
<td class="gt_row gt_left" headers="Variable">cap</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">Controlled attenuation parameter (CAP)</td>
<td class="gt_row gt_right" headers="n">14405</td>
<td class="gt_row gt_right" headers="Missing">1579</td>
<td class="gt_row gt_right" headers="Distinct">255</td>
<td class="gt_row gt_right" headers="Info">1.000</td>
<td class="gt_row gt_right" headers="Mean">303.4</td>
<td class="gt_row gt_right" headers="Gmd">47.72</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-38c7c847f5c1e5024674"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-38c7c847f5c1e5024674">{"x":{"values":[152,3,6,7,16,14,8,13,14,16,18,17,20,13,20,19,20,17,26,18,21,25,34,26,24,37,35,34,223,225,232,231,232,246,238,233,244,262,252,238,217,239,224,256,271,242,254,247,261,265,231,244,222,245,247,237,278,228,257,200,230,242,210,238,216,224,222,228,230,216,216,197,214,170,165,165,171,160,165,146,142,148,120,135,116,102,120,96,79,72,67,62,62,62,57,66,47,47,34,42,31,29,46,204],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:192<br>Bin: [150, 194)<br>Observed:<br>150 - 193 (38 distinct)<br>Proportion:0.0106 (n=152)<br>Cumulative:0.0106\",\"Bin: [194, 196)<br>Observed:<br>194 (2)<br>195<br>Proportion:2e-04 (n=3)<br>Cumulative:0.0108\",\"Bin: [196, 198)<br>Observed:<br>196 (4)<br>197 (2)<br>Proportion:4e-04 (n=6)<br>Cumulative:0.0112\",\"Bin: [198, 200)<br>Observed:<br>198 (3)<br>199 (4)<br>Proportion:5e-04 (n=7)<br>Cumulative:0.0117\",\"Bin: [200, 202)<br>Observed:<br>200 (14)<br>201 (2)<br>Proportion:0.0011 (n=16)<br>Cumulative:0.0128\",\"Bin: [202, 204)<br>Observed:<br>202 (6)<br>203 (7)<br>203.5<br>Proportion:0.001 (n=14)<br>Cumulative:0.0137\",\"Bin: [204, 206)<br>Observed:<br>204 (5)<br>205 (3)<br>Proportion:6e-04 (n=8)<br>Cumulative:0.0143\",\"Bin: [206, 208)<br>Observed:<br>206 (7)<br>207 (6)<br>Proportion:9e-04 (n=13)<br>Cumulative:0.0152\",\"Bin: [208, 210)<br>Observed:<br>208 (8)<br>209 (6)<br>Proportion:0.001 (n=14)<br>Cumulative:0.0162\",\"Bin: [210, 212)<br>Observed:<br>210 (10)<br>211 (6)<br>Proportion:0.0011 (n=16)<br>Cumulative:0.0173\",\"Bin: [212, 214)<br>Observed:<br>212 (8)<br>213 (10)<br>Proportion:0.0012 (n=18)<br>Cumulative:0.0185\",\"Bin: [214, 216)<br>Observed:<br>214 (10)<br>215 (7)<br>Proportion:0.0012 (n=17)<br>Cumulative:0.0197\",\"Bin: [216, 218)<br>Observed:<br>216 (12)<br>217 (8)<br>Proportion:0.0014 (n=20)<br>Cumulative:0.0211\",\"Bin: [218, 220)<br>Observed:<br>218 (7)<br>219 (6)<br>Proportion:9e-04 (n=13)<br>Cumulative:0.022\",\"Bin: [220, 222)<br>Observed:<br>220 (9)<br>221 (11)<br>Proportion:0.0014 (n=20)<br>Cumulative:0.0234\",\"Bin: [222, 224)<br>Observed:<br>222 (8)<br>223 (11)<br>Proportion:0.0013 (n=19)<br>Cumulative:0.0247\",\"Bin: [224, 226)<br>Observed:<br>224 (12)<br>225 (8)<br>Proportion:0.0014 (n=20)<br>Cumulative:0.0261\",\"Bin: [226, 228)<br>Observed:<br>226 (7)<br>227 (10)<br>Proportion:0.0012 (n=17)<br>Cumulative:0.0273\",\"Bin: [228, 230)<br>Observed:<br>228 (13)<br>229 (13)<br>Proportion:0.0018 (n=26)<br>Cumulative:0.0291\",\"Bin: [230, 232)<br>Observed:<br>230 (13)<br>231 (5)<br>Proportion:0.0012 (n=18)<br>Cumulative:0.0303\",\"Bin: [232, 234)<br>Observed:<br>232 (10)<br>233 (11)<br>Proportion:0.0015 (n=21)<br>Cumulative:0.0318\",\"Bin: [234, 236)<br>Observed:<br>234 (12)<br>235 (12)<br>235.5<br>Proportion:0.0017 (n=25)<br>Cumulative:0.0335\",\"Bin: [236, 238)<br>Observed:<br>236 (20)<br>237 (14)<br>Proportion:0.0024 (n=34)<br>Cumulative:0.0359\",\"Bin: [238, 240)<br>Observed:<br>238 (17)<br>239 (9)<br>Proportion:0.0018 (n=26)<br>Cumulative:0.0377\",\"Bin: [240, 242)<br>Observed:<br>240 (13)<br>241 (11)<br>Proportion:0.0017 (n=24)<br>Cumulative:0.0394\",\"Bin: [242, 244)<br>Observed:<br>242 (17)<br>243 (20)<br>Proportion:0.0026 (n=37)<br>Cumulative:0.0419\",\"Bin: [244, 246)<br>Observed:<br>244 (16)<br>245 (19)<br>Proportion:0.0024 (n=35)<br>Cumulative:0.0444\",\"Bin: [246, 248)<br>Observed:<br>246 (20)<br>247 (14)<br>Proportion:0.0024 (n=34)<br>Cumulative:0.0467\",\"Bin: [248, 250)<br>Observed:<br>248 (108)<br>249 (115)<br>Proportion:0.0155 (n=223)<br>Cumulative:0.0622\",\"Bin: [250, 252)<br>Observed:<br>250 (108)<br>251 (117)<br>Proportion:0.0156 (n=225)<br>Cumulative:0.0778\",\"Bin: [252, 254)<br>Observed:<br>252 (122)<br>253 (110)<br>Proportion:0.0161 (n=232)<br>Cumulative:0.0939\",\"Bin: [254, 256)<br>Observed:<br>254 (109)<br>255 (122)<br>Proportion:0.016 (n=231)<br>Cumulative:0.11\",\"Bin: [256, 258)<br>Observed:<br>256 (122)<br>257 (110)<br>Proportion:0.0161 (n=232)<br>Cumulative:0.1261\",\"Bin: [258, 260)<br>Observed:<br>258 (118)<br>259 (128)<br>Proportion:0.0171 (n=246)<br>Cumulative:0.1431\",\"Bin: [260, 262)<br>Observed:<br>260 (116)<br>260.5<br>261 (121)<br>Proportion:0.0165 (n=238)<br>Cumulative:0.1597\",\"Bin: [262, 264)<br>Observed:<br>262 (130)<br>263 (103)<br>Proportion:0.0162 (n=233)<br>Cumulative:0.1758\",\"Bin: [264, 266)<br>Observed:<br>264 (111)<br>265 (133)<br>Proportion:0.0169 (n=244)<br>Cumulative:0.1928\",\"Bin: [266, 268)<br>Observed:<br>266 (126)<br>267 (136)<br>Proportion:0.0182 (n=262)<br>Cumulative:0.211\",\"Bin: [268, 270)<br>Observed:<br>268 (130)<br>269 (122)<br>Proportion:0.0175 (n=252)<br>Cumulative:0.2285\",\"Bin: [270, 272)<br>Observed:<br>270 (113)<br>271 (125)<br>Proportion:0.0165 (n=238)<br>Cumulative:0.245\",\"Bin: [272, 274)<br>Observed:<br>272 (117)<br>273 (100)<br>Proportion:0.0151 (n=217)<br>Cumulative:0.26\",\"Bin: [274, 276)<br>Observed:<br>274 (109)<br>275 (130)<br>Proportion:0.0166 (n=239)<br>Cumulative:0.2766\",\"Bin: [276, 278)<br>Observed:<br>276 (106)<br>277 (118)<br>Proportion:0.0156 (n=224)<br>Cumulative:0.2922\",\"Bin: [278, 280)<br>Observed:<br>278 (124)<br>279 (132)<br>Proportion:0.0178 (n=256)<br>Cumulative:0.31\",\"Bin: [280, 282)<br>Observed:<br>280 (136)<br>281 (135)<br>Proportion:0.0188 (n=271)<br>Cumulative:0.3288\",\"Bin: [282, 284)<br>Observed:<br>282 (128)<br>283 (114)<br>Proportion:0.0168 (n=242)<br>Cumulative:0.3456\",\"Bin: [284, 286)<br>Observed:<br>284 (127)<br>284.5<br>285 (126)<br>Proportion:0.0176 (n=254)<br>Cumulative:0.3632\",\"Bin: [286, 288)<br>Observed:<br>286 (124)<br>287 (123)<br>Proportion:0.0171 (n=247)<br>Cumulative:0.3804\",\"Bin: [288, 290)<br>Observed:<br>288 (143)<br>289 (117)<br>289.5<br>Proportion:0.0181 (n=261)<br>Cumulative:0.3985\",\"Bin: [290, 292)<br>Observed:<br>290 (125)<br>291 (140)<br>Proportion:0.0184 (n=265)<br>Cumulative:0.4169\",\"Bin: [292, 294)<br>Observed:<br>292 (98)<br>293 (133)<br>Proportion:0.016 (n=231)<br>Cumulative:0.4329\",\"Bin: [294, 296)<br>Observed:<br>294 (111)<br>295 (133)<br>Proportion:0.0169 (n=244)<br>Cumulative:0.4498\",\"Bin: [296, 298)<br>Observed:<br>296 (112)<br>297 (110)<br>Proportion:0.0154 (n=222)<br>Cumulative:0.4653\",\"Bin: [298, 300)<br>Observed:<br>298 (126)<br>299 (119)<br>Proportion:0.017 (n=245)<br>Cumulative:0.4823\",\"Bin: [300, 302)<br>Observed:<br>300 (135)<br>301 (112)<br>Proportion:0.0171 (n=247)<br>Cumulative:0.4994\",\"Bin: [302, 304)<br>Observed:<br>302 (123)<br>303 (114)<br>Proportion:0.0165 (n=237)<br>Cumulative:0.5159\",\"Bin: [304, 306)<br>Observed:<br>304 (133)<br>305 (145)<br>Proportion:0.0193 (n=278)<br>Cumulative:0.5352\",\"Bin: [306, 308)<br>Observed:<br>306 (114)<br>307 (113)<br>307.5<br>Proportion:0.0158 (n=228)<br>Cumulative:0.551\",\"Bin: [308, 310)<br>Observed:<br>308 (131)<br>309 (126)<br>Proportion:0.0178 (n=257)<br>Cumulative:0.5688\",\"Bin: [310, 312)<br>Observed:<br>310 (105)<br>311 (95)<br>Proportion:0.0139 (n=200)<br>Cumulative:0.5827\",\"Bin: [312, 314)<br>Observed:<br>312 (109)<br>313 (121)<br>Proportion:0.016 (n=230)<br>Cumulative:0.5987\",\"Bin: [314, 316)<br>Observed:<br>314 (126)<br>314.5<br>315 (115)<br>Proportion:0.0168 (n=242)<br>Cumulative:0.6155\",\"Bin: [316, 318)<br>Observed:<br>316 (114)<br>317 (96)<br>Proportion:0.0146 (n=210)<br>Cumulative:0.6301\",\"Bin: [318, 320)<br>Observed:<br>318 (121)<br>319 (117)<br>Proportion:0.0165 (n=238)<br>Cumulative:0.6466\",\"Bin: [320, 322)<br>Observed:<br>320 (107)<br>321 (109)<br>Proportion:0.015 (n=216)<br>Cumulative:0.6616\",\"Bin: [322, 324)<br>Observed:<br>322 (116)<br>322.5<br>323 (107)<br>Proportion:0.0156 (n=224)<br>Cumulative:0.6771\",\"Bin: [324, 326)<br>Observed:<br>324 (113)<br>325 (109)<br>Proportion:0.0154 (n=222)<br>Cumulative:0.6925\",\"Bin: [326, 328)<br>Observed:<br>326 (127)<br>327 (101)<br>Proportion:0.0158 (n=228)<br>Cumulative:0.7084\",\"Bin: [328, 330)<br>Observed:<br>328 (136)<br>329 (94)<br>Proportion:0.016 (n=230)<br>Cumulative:0.7243\",\"Bin: [330, 332)<br>Observed:<br>330 (114)<br>331 (101)<br>331.5<br>Proportion:0.015 (n=216)<br>Cumulative:0.7393\",\"Bin: [332, 334)<br>Observed:<br>332 (103)<br>333 (113)<br>Proportion:0.015 (n=216)<br>Cumulative:0.7543\",\"Bin: [334, 336)<br>Observed:<br>334 (93)<br>335 (104)<br>Proportion:0.0137 (n=197)<br>Cumulative:0.768\",\"Bin: [336, 338)<br>Observed:<br>336 (104)<br>336.5<br>337 (109)<br>Proportion:0.0149 (n=214)<br>Cumulative:0.7829\",\"Bin: [338, 340)<br>Observed:<br>338 (87)<br>339 (83)<br>Proportion:0.0118 (n=170)<br>Cumulative:0.7947\",\"Bin: [340, 342)<br>Observed:<br>340 (84)<br>341 (81)<br>Proportion:0.0115 (n=165)<br>Cumulative:0.8061\",\"Bin: [342, 344)<br>Observed:<br>342 (92)<br>343 (73)<br>Proportion:0.0115 (n=165)<br>Cumulative:0.8176\",\"Bin: [344, 346)<br>Observed:<br>344 (93)<br>345 (78)<br>Proportion:0.0119 (n=171)<br>Cumulative:0.8294\",\"Bin: [346, 348)<br>Observed:<br>346 (89)<br>347 (71)<br>Proportion:0.0111 (n=160)<br>Cumulative:0.8405\",\"Bin: [348, 350)<br>Observed:<br>348 (82)<br>349 (83)<br>Proportion:0.0115 (n=165)<br>Cumulative:0.852\",\"Bin: [350, 352)<br>Observed:<br>350 (74)<br>351 (72)<br>Proportion:0.0101 (n=146)<br>Cumulative:0.8621\",\"Bin: [352, 354)<br>Observed:<br>352 (69)<br>353 (73)<br>Proportion:0.0099 (n=142)<br>Cumulative:0.872\",\"Bin: [354, 356)<br>Observed:<br>354 (83)<br>355 (65)<br>Proportion:0.0103 (n=148)<br>Cumulative:0.8823\",\"Bin: [356, 358)<br>Observed:<br>356 (59)<br>357 (61)<br>Proportion:0.0083 (n=120)<br>Cumulative:0.8906\",\"Bin: [358, 360)<br>Observed:<br>358 (67)<br>359 (68)<br>Proportion:0.0094 (n=135)<br>Cumulative:0.9\",\"Bin: [360, 362)<br>Observed:<br>360 (60)<br>361 (56)<br>Proportion:0.0081 (n=116)<br>Cumulative:0.908\",\"Bin: [362, 364)<br>Observed:<br>362 (57)<br>363 (45)<br>Proportion:0.0071 (n=102)<br>Cumulative:0.9151\",\"Bin: [364, 366)<br>Observed:<br>364 (68)<br>365 (52)<br>Proportion:0.0083 (n=120)<br>Cumulative:0.9234\",\"Bin: [366, 368)<br>Observed:<br>366 (55)<br>367 (41)<br>Proportion:0.0067 (n=96)<br>Cumulative:0.9301\",\"Bin: [368, 370)<br>Observed:<br>368 (41)<br>369 (38)<br>Proportion:0.0055 (n=79)<br>Cumulative:0.9356\",\"Bin: [370, 372)<br>Observed:<br>370 (36)<br>371 (36)<br>Proportion:0.005 (n=72)<br>Cumulative:0.9406\",\"Bin: [372, 374)<br>Observed:<br>372 (35)<br>373 (32)<br>Proportion:0.0047 (n=67)<br>Cumulative:0.9452\",\"Bin: [374, 376)<br>Observed:<br>374 (33)<br>375 (29)<br>Proportion:0.0043 (n=62)<br>Cumulative:0.9495\",\"Bin: [376, 378)<br>Observed:<br>376 (39)<br>377 (23)<br>Proportion:0.0043 (n=62)<br>Cumulative:0.9538\",\"Bin: [378, 380)<br>Observed:<br>378 (28)<br>379 (34)<br>Proportion:0.0043 (n=62)<br>Cumulative:0.9581\",\"Bin: [380, 382)<br>Observed:<br>380 (23)<br>381 (34)<br>Proportion:0.004 (n=57)<br>Cumulative:0.9621\",\"Bin: [382, 384)<br>Observed:<br>382 (33)<br>383 (33)<br>Proportion:0.0046 (n=66)<br>Cumulative:0.9667\",\"Bin: [384, 386)<br>Observed:<br>384 (18)<br>385 (29)<br>Proportion:0.0033 (n=47)<br>Cumulative:0.9699\",\"Bin: [386, 388)<br>Observed:<br>386 (24)<br>387 (23)<br>Proportion:0.0033 (n=47)<br>Cumulative:0.9732\",\"Bin: [388, 390)<br>Observed:<br>388 (12)<br>389 (22)<br>Proportion:0.0024 (n=34)<br>Cumulative:0.9756\",\"Bin: [390, 392)<br>Observed:<br>390 (25)<br>391 (17)<br>Proportion:0.0029 (n=42)<br>Cumulative:0.9785\",\"Bin: [392, 394)<br>Observed:<br>392 (15)<br>393 (16)<br>Proportion:0.0022 (n=31)<br>Cumulative:0.9806\",\"Bin: [394, 396)<br>Observed:<br>394 (14)<br>395 (15)<br>Proportion:0.002 (n=29)<br>Cumulative:0.9826\",\"Bin: [396, 398]<br>Observed:<br>396 (16)<br>397 (18)<br>398 (12)<br>Proportion:0.0032 (n=46)<br>Cumulative:0.9858\",\"Q<sub>0.99<\\/sub>:400<br>Bin: (398, 400]<br>Observed:<br>399 (14)<br>400 (190)<br>Proportion:0.0142 (n=204)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 248.0
</font><font size="2"> 254.0 </font><font size="3"> 272.0
</font><font size="4"> <strong>302.0</strong> </font><font size="3">
333.0 </font><font size="2"> 359.6 </font><font size="1"> 376.0
</font></td>
</tr>
<tr class="odd">
<td class="gt_row gt_left" headers="Variable">iqr_cap</td>
<td class="gt_row gt_left" headers="Label"
style="font-size: small">Inter-quartile range of CAP</td>
<td class="gt_row gt_right" headers="n">13190</td>
<td class="gt_row gt_right" headers="Missing">2794</td>
<td class="gt_row gt_right" headers="Distinct">139</td>
<td class="gt_row gt_right" headers="Info">0.999</td>
<td class="gt_row gt_right" headers="Mean">27.14</td>
<td class="gt_row gt_right" headers="Gmd">13.45</td>
<td class="gt_row gt_right" headers=" "><span
id="htmlwidget-4180f631735eb6bcbbdb"
class="sparkline html-widget"></span>
<script type="application/json" data-for="htmlwidget-4180f631735eb6bcbbdb">{"x":{"values":[134,60,0,110,0,115,1,164,0,191,1,225,0,289,0,377,0,369,0,403,0,432,0,448,1,500,0,448,0,485,0,500,0,494,2,488,1,477,0,473,0,471,0,450,0,435,0,397,0,365,0,370,2,299,0,314,0,288,0,239,0,248,0,205,0,200,0,159,0,155,0,137,0,105,0,126,1,94,0,79,0,91,1,80,0,74,0,42,0,56,0,47,0,48,0,35,1,33,0,27,0,22,0,25,0,26,0,11,0,17,0,14,0,10,1,11,0,15,0,7,0,11,0,8,0,8,10,132],"options":{"type":"bar","chartRangeMin":0,"zeroColor":"lightgray","barWidth":1,"barSpacing":1,"tooltipFormatter":"function(sparkline, options, field){\n       debugger;\n       return [\"Q<sub>0.01<\\/sub>:6<br>Bin: [1, 7)<br>Observed:<br>1 (3)<br>1.6<br>2 (3)<br>3 (8)<br>3.6<br>4 (22)<br>4.9<br>5 (47)<br>6 (48)<br>Proportion:0.0102 (n=134)<br>Cumulative:0.0102\",\"Bin: [7, 7.5)<br>Observed:<br>7 (60)<br>Proportion:0.0045 (n=60)<br>Cumulative:0.0147\",\"\",\"Bin: [8, 8.5)<br>Observed:<br>8 (110)<br>Proportion:0.0083 (n=110)<br>Cumulative:0.023\",\"\",\"Bin: [9, 9.5)<br>Observed:<br>9 (114)<br>9.3<br>Proportion:0.0087 (n=115)<br>Cumulative:0.0318\",\"Bin: [9.5, 10)<br>Observed:<br>9.5<br>Proportion:1e-04 (n=1)<br>Cumulative:0.0318\",\"Bin: [10, 10.5)<br>Observed:<br>10 (163)<br>10.25<br>Proportion:0.0124 (n=164)<br>Cumulative:0.0443\",\"\",\"Bin: [11, 11.5)<br>Observed:<br>11 (191)<br>Proportion:0.0145 (n=191)<br>Cumulative:0.0588\",\"Bin: [11.5, 12)<br>Observed:<br>11.5<br>Proportion:1e-04 (n=1)<br>Cumulative:0.0588\",\"Bin: [12, 12.5)<br>Observed:<br>12 (225)<br>Proportion:0.0171 (n=225)<br>Cumulative:0.0759\",\"\",\"Bin: [13, 13.5)<br>Observed:<br>13 (289)<br>Proportion:0.0219 (n=289)<br>Cumulative:0.0978\",\"\",\"Bin: [14, 14.5)<br>Observed:<br>14 (377)<br>Proportion:0.0286 (n=377)<br>Cumulative:0.1264\",\"\",\"Bin: [15, 15.5)<br>Observed:<br>15 (369)<br>Proportion:0.028 (n=369)<br>Cumulative:0.1544\",\"\",\"Bin: [16, 16.5)<br>Observed:<br>16 (403)<br>Proportion:0.0306 (n=403)<br>Cumulative:0.1849\",\"\",\"Bin: [17, 17.5)<br>Observed:<br>17 (432)<br>Proportion:0.0328 (n=432)<br>Cumulative:0.2177\",\"\",\"Bin: [18, 18.5)<br>Observed:<br>18 (448)<br>Proportion:0.034 (n=448)<br>Cumulative:0.2516\",\"Bin: [18.5, 19)<br>Observed:<br>18.5<br>Proportion:1e-04 (n=1)<br>Cumulative:0.2517\",\"Bin: [19, 19.5)<br>Observed:<br>19 (500)<br>Proportion:0.0379 (n=500)<br>Cumulative:0.2896\",\"\",\"Bin: [20, 20.5)<br>Observed:<br>20 (448)<br>Proportion:0.034 (n=448)<br>Cumulative:0.3236\",\"\",\"Bin: [21, 21.5)<br>Observed:<br>21 (485)<br>Proportion:0.0368 (n=485)<br>Cumulative:0.3603\",\"\",\"Bin: [22, 22.5)<br>Observed:<br>22 (500)<br>Proportion:0.0379 (n=500)<br>Cumulative:0.3983\",\"\",\"Bin: [23, 23.5)<br>Observed:<br>23 (494)<br>Proportion:0.0375 (n=494)<br>Cumulative:0.4357\",\"Bin: [23.5, 24)<br>Observed:<br>23.5<br>23.8<br>Proportion:2e-04 (n=2)<br>Cumulative:0.4359\",\"Bin: [24, 24.5)<br>Observed:<br>24 (488)<br>Proportion:0.037 (n=488)<br>Cumulative:0.4729\",\"Bin: [24.5, 25)<br>Observed:<br>24.8<br>Proportion:1e-04 (n=1)<br>Cumulative:0.4729\",\"Bin: [25, 25.5)<br>Observed:<br>25 (477)<br>Proportion:0.0362 (n=477)<br>Cumulative:0.5091\",\"\",\"Bin: [26, 26.5)<br>Observed:<br>26 (472)<br>26.3<br>Proportion:0.0359 (n=473)<br>Cumulative:0.545\",\"\",\"Bin: [27, 27.5)<br>Observed:<br>27 (471)<br>Proportion:0.0357 (n=471)<br>Cumulative:0.5807\",\"\",\"Bin: [28, 28.5)<br>Observed:<br>28 (450)<br>Proportion:0.0341 (n=450)<br>Cumulative:0.6148\",\"\",\"Bin: [29, 29.5)<br>Observed:<br>29 (435)<br>Proportion:0.033 (n=435)<br>Cumulative:0.6478\",\"\",\"Bin: [30, 30.5)<br>Observed:<br>30 (397)<br>Proportion:0.0301 (n=397)<br>Cumulative:0.6779\",\"\",\"Bin: [31, 31.5)<br>Observed:<br>31 (365)<br>Proportion:0.0277 (n=365)<br>Cumulative:0.7055\",\"\",\"Bin: [32, 32.5)<br>Observed:<br>32 (370)<br>Proportion:0.0281 (n=370)<br>Cumulative:0.7336\",\"Bin: [32.5, 33)<br>Observed:<br>32.5<br>32.75<br>Proportion:2e-04 (n=2)<br>Cumulative:0.7337\",\"Bin: [33, 33.5)<br>Observed:<br>33 (299)<br>Proportion:0.0227 (n=299)<br>Cumulative:0.7564\",\"\",\"Bin: [34, 34.5)<br>Observed:<br>34 (314)<br>Proportion:0.0238 (n=314)<br>Cumulative:0.7802\",\"\",\"Bin: [35, 35.5)<br>Observed:<br>35 (288)<br>Proportion:0.0218 (n=288)<br>Cumulative:0.802\",\"\",\"Bin: [36, 36.5)<br>Observed:<br>36 (239)<br>Proportion:0.0181 (n=239)<br>Cumulative:0.8202\",\"\",\"Bin: [37, 37.5)<br>Observed:<br>37 (248)<br>Proportion:0.0188 (n=248)<br>Cumulative:0.839\",\"\",\"Bin: [38, 38.5)<br>Observed:<br>38 (205)<br>Proportion:0.0155 (n=205)<br>Cumulative:0.8545\",\"\",\"Bin: [39, 39.5)<br>Observed:<br>39 (200)<br>Proportion:0.0152 (n=200)<br>Cumulative:0.8697\",\"\",\"Bin: [40, 40.5)<br>Observed:<br>40 (159)<br>Proportion:0.0121 (n=159)<br>Cumulative:0.8817\",\"\",\"Bin: [41, 41.5)<br>Observed:<br>41 (155)<br>Proportion:0.0118 (n=155)<br>Cumulative:0.8935\",\"\",\"Bin: [42, 42.5)<br>Observed:<br>42 (137)<br>Proportion:0.0104 (n=137)<br>Cumulative:0.9039\",\"\",\"Bin: [43, 43.5)<br>Observed:<br>43 (105)<br>Proportion:0.008 (n=105)<br>Cumulative:0.9118\",\"\",\"Bin: [44, 44.5)<br>Observed:<br>44 (126)<br>Proportion:0.0096 (n=126)<br>Cumulative:0.9214\",\"Bin: [44.5, 45)<br>Observed:<br>44.5<br>Proportion:1e-04 (n=1)<br>Cumulative:0.9215\",\"Bin: [45, 45.5)<br>Observed:<br>45 (94)<br>Proportion:0.0071 (n=94)<br>Cumulative:0.9286\",\"\",\"Bin: [46, 46.5)<br>Observed:<br>46 (79)<br>Proportion:0.006 (n=79)<br>Cumulative:0.9346\",\"\",\"Bin: [47, 47.5)<br>Observed:<br>47 (91)<br>Proportion:0.0069 (n=91)<br>Cumulative:0.9415\",\"Bin: [47.5, 48)<br>Observed:<br>47.5<br>Proportion:1e-04 (n=1)<br>Cumulative:0.9415\",\"Bin: [48, 48.5)<br>Observed:<br>48 (80)<br>Proportion:0.0061 (n=80)<br>Cumulative:0.9476\",\"\",\"Bin: [49, 49.5)<br>Observed:<br>49 (74)<br>Proportion:0.0056 (n=74)<br>Cumulative:0.9532\",\"\",\"Bin: [50, 50.5)<br>Observed:<br>50 (42)<br>Proportion:0.0032 (n=42)<br>Cumulative:0.9564\",\"\",\"Bin: [51, 51.5)<br>Observed:<br>51 (56)<br>Proportion:0.0042 (n=56)<br>Cumulative:0.9607\",\"\",\"Bin: [52, 52.5)<br>Observed:<br>52 (47)<br>Proportion:0.0036 (n=47)<br>Cumulative:0.9642\",\"\",\"Bin: [53, 53.5)<br>Observed:<br>53 (48)<br>Proportion:0.0036 (n=48)<br>Cumulative:0.9679\",\"\",\"Bin: [54, 54.5)<br>Observed:<br>54 (35)<br>Proportion:0.0027 (n=35)<br>Cumulative:0.9705\",\"Bin: [54.5, 55)<br>Observed:<br>54.8<br>Proportion:1e-04 (n=1)<br>Cumulative:0.9706\",\"Bin: [55, 55.5)<br>Observed:<br>55 (33)<br>Proportion:0.0025 (n=33)<br>Cumulative:0.9731\",\"\",\"Bin: [56, 56.5)<br>Observed:<br>56 (27)<br>Proportion:0.002 (n=27)<br>Cumulative:0.9751\",\"\",\"Bin: [57, 57.5)<br>Observed:<br>57 (22)<br>Proportion:0.0017 (n=22)<br>Cumulative:0.9768\",\"\",\"Bin: [58, 58.5)<br>Observed:<br>58 (25)<br>Proportion:0.0019 (n=25)<br>Cumulative:0.9787\",\"\",\"Bin: [59, 59.5)<br>Observed:<br>59 (26)<br>Proportion:0.002 (n=26)<br>Cumulative:0.9807\",\"\",\"Bin: [60, 60.5)<br>Observed:<br>60 (11)<br>Proportion:8e-04 (n=11)<br>Cumulative:0.9815\",\"\",\"Bin: [61, 61.5)<br>Observed:<br>61 (17)<br>Proportion:0.0013 (n=17)<br>Cumulative:0.9828\",\"\",\"Bin: [62, 62.5)<br>Observed:<br>62 (14)<br>Proportion:0.0011 (n=14)<br>Cumulative:0.9839\",\"\",\"Bin: [63, 63.5)<br>Observed:<br>63 (10)<br>Proportion:8e-04 (n=10)<br>Cumulative:0.9846\",\"Bin: [63.5, 64)<br>Observed:<br>63.7<br>Proportion:1e-04 (n=1)<br>Cumulative:0.9847\",\"Bin: [64, 64.5)<br>Observed:<br>64 (11)<br>Proportion:8e-04 (n=11)<br>Cumulative:0.9855\",\"\",\"Bin: [65, 65.5)<br>Observed:<br>65 (15)<br>Proportion:0.0011 (n=15)<br>Cumulative:0.9867\",\"\",\"Bin: [66, 66.5)<br>Observed:<br>66 (7)<br>Proportion:5e-04 (n=7)<br>Cumulative:0.9872\",\"\",\"Bin: [67, 67.5)<br>Observed:<br>67 (11)<br>Proportion:8e-04 (n=11)<br>Cumulative:0.988\",\"\",\"Bin: [68, 68.5)<br>Observed:<br>68 (8)<br>Proportion:6e-04 (n=8)<br>Cumulative:0.9886\",\"\",\"Bin: [69, 69.5)<br>Observed:<br>69 (8)<br>Proportion:6e-04 (n=8)<br>Cumulative:0.9892\",\"Bin: [69.5, 70]<br>Observed:<br>70 (10)<br>Proportion:8e-04 (n=10)<br>Cumulative:0.99\",\"Q<sub>0.99<\\/sub>:71<br>Bin: (70, 200]<br>Observed:<br>71 - 200 (51 distinct)<br>Proportion:0.01 (n=132)<br>Cumulative:1\"][field[0].offset];\n       }","height":20,"width":200},"width":200,"height":20},"evals":["options.tooltipFormatter"],"jsHooks":[]}</script></td>
<td class="gt_row gt_center" headers="Quantiles"><font size="1"> 11
</font><font size="2"> 14 </font><font size="3"> 18
</font><font size="4"> <strong>25</strong> </font><font size="3"> 33
</font><font size="2"> 42 </font><font size="1"> 49 </font></td>
</tr>
</tbody>
</table>

```{=html}

</div>
```

</div>

## Categorical

<div>

```{=html}
<div id="alugvptgbb" style="padding-left:0px;padding-right:0px;padding-top:10px;padding-bottom:10px;overflow-x:auto;overflow-y:auto;width:auto;height:auto;">
<style>#alugvptgbb table {
  font-family: system-ui, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

#alugvptgbb thead, #alugvptgbb tbody, #alugvptgbb tfoot, #alugvptgbb tr, #alugvptgbb td, #alugvptgbb th {
  border-style: none;
}

#alugvptgbb p {
  margin: 0;
  padding: 0;
}

#alugvptgbb .gt_table {
  display: table;
  border-collapse: collapse;
  line-height: normal;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  font-weight: normal;
  font-style: normal;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#alugvptgbb .gt_caption {
  padding-top: 4px;
  padding-bottom: 4px;
}

#alugvptgbb .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#alugvptgbb .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 3px;
  padding-bottom: 5px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#alugvptgbb .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#alugvptgbb .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#alugvptgbb .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#alugvptgbb .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#alugvptgbb .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#alugvptgbb .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#alugvptgbb .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#alugvptgbb .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 5px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#alugvptgbb .gt_spanner_row {
  border-bottom-style: hidden;
}

#alugvptgbb .gt_group_heading {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  text-align: left;
}

#alugvptgbb .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#alugvptgbb .gt_from_md > :first-child {
  margin-top: 0;
}

#alugvptgbb .gt_from_md > :last-child {
  margin-bottom: 0;
}

#alugvptgbb .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#alugvptgbb .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
}

#alugvptgbb .gt_stub_row_group {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 5px;
  padding-right: 5px;
  vertical-align: top;
}

#alugvptgbb .gt_row_group_first td {
  border-top-width: 2px;
}

#alugvptgbb .gt_row_group_first th {
  border-top-width: 2px;
}

#alugvptgbb .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#alugvptgbb .gt_first_summary_row {
  border-top-style: solid;
  border-top-color: #D3D3D3;
}

#alugvptgbb .gt_first_summary_row.thick {
  border-top-width: 2px;
}

#alugvptgbb .gt_last_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#alugvptgbb .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#alugvptgbb .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#alugvptgbb .gt_last_grand_summary_row_top {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-bottom-style: double;
  border-bottom-width: 6px;
  border-bottom-color: #D3D3D3;
}

#alugvptgbb .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#alugvptgbb .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#alugvptgbb .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#alugvptgbb .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#alugvptgbb .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#alugvptgbb .gt_sourcenote {
  font-size: 90%;
  padding-top: 4px;
  padding-bottom: 4px;
  padding-left: 5px;
  padding-right: 5px;
}

#alugvptgbb .gt_left {
  text-align: left;
}

#alugvptgbb .gt_center {
  text-align: center;
}

#alugvptgbb .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#alugvptgbb .gt_font_normal {
  font-weight: normal;
}

#alugvptgbb .gt_font_bold {
  font-weight: bold;
}

#alugvptgbb .gt_font_italic {
  font-style: italic;
}

#alugvptgbb .gt_super {
  font-size: 65%;
}

#alugvptgbb .gt_footnote_marks {
  font-size: 75%;
  vertical-align: 0.4em;
  position: initial;
}

#alugvptgbb .gt_asterisk {
  font-size: 100%;
  vertical-align: 0;
}

#alugvptgbb .gt_indent_1 {
  text-indent: 5px;
}

#alugvptgbb .gt_indent_2 {
  text-indent: 10px;
}

#alugvptgbb .gt_indent_3 {
  text-indent: 15px;
}

#alugvptgbb .gt_indent_4 {
  text-indent: 20px;
}

#alugvptgbb .gt_indent_5 {
  text-indent: 25px;
}
</style>
```
+-----------+-----------+-----------+-----------+-----------+-----------+
| **`vcte0` |           |           |           |           |           |
| Descr     |           |           |           |           |           |
| iptives** |           |           |           |           |           |
+===========+===========+===========+===========+===========+===========+
| 2         |           |           |           |           |           |
| Ca        |           |           |           |           |           |
| tegorical |           |           |           |           |           |
| Variables |           |           |           |           |           |
| of 13     |           |           |           |           |           |
| V         |           |           |           |           |           |
| ariables, |           |           |           |           |           |
| 15984     |           |           |           |           |           |
| Obs       |           |           |           |           |           |
| ervations |           |           |           |           |           |
+-----------+-----------+-----------+-----------+-----------+-----------+
| Variable  | Label     | n         | Missing   | Distinct  |           |
+-----------+-----------+-----------+-----------+-----------+-----------+
| id        | Study ID  | 15984     | 0         | 15984     | V1 -\     |
|           |           |           |           |           | V9999\    |
|           |           |           |           |           | Min       |
|           |           |           |           |           | /Max/Mean |
|           |           |           |           |           | Width: 2  |
|           |           |           |           |           | / 6 / 5.3 |
+-----------+-----------+-----------+-----------+-----------+-----------+
| sex       | Sex (male | 15984     | 0         | 2         | []{#html  |
|           | = 1)      |           |           |           | widget-07 |
|           |           |           |           |           | d48ab9098 |
|           |           |           |           |           | 012be3896 |
|           |           |           |           |           | .         |
|           |           |           |           |           | sparkline |
|           |           |           |           |           | .htm      |
|           |           |           |           |           | l-widget} |
|           |           |           |           |           | `         |
|           |           |           |           |           | ``{=html} |
|           |           |           |           |           | <script t |
|           |           |           |           |           | ype="appl |
|           |           |           |           |           | ication/j |
|           |           |           |           |           | son" data |
|           |           |           |           |           | -for="htm |
|           |           |           |           |           | lwidget-0 |
|           |           |           |           |           | 7d48ab909 |
|           |           |           |           |           | 8012be389 |
|           |           |           |           |           | 6">{"x":{ |
|           |           |           |           |           | "values": |
|           |           |           |           |           | [9365,661 |
|           |           |           |           |           | 9],"optio |
|           |           |           |           |           | ns":{"typ |
|           |           |           |           |           | e":"bar", |
|           |           |           |           |           | "chartRan |
|           |           |           |           |           | geMin":0, |
|           |           |           |           |           | "zeroColo |
|           |           |           |           |           | r":"light |
|           |           |           |           |           | gray","ba |
|           |           |           |           |           | rWidth":3 |
|           |           |           |           |           | ,"barSpac |
|           |           |           |           |           | ing":2,"t |
|           |           |           |           |           | ooltipFor |
|           |           |           |           |           | matter":" |
|           |           |           |           |           | function( |
|           |           |           |           |           | sparkline |
|           |           |           |           |           | , options |
|           |           |           |           |           | , field){ |
|           |           |           |           |           | \n        |
|           |           |           |           |           | debugger; |
|           |           |           |           |           | \n        |
|           |           |           |           |           | return [\ |
|           |           |           |           |           | "male<br> |
|           |           |           |           |           | Proportio |
|           |           |           |           |           | n:0.5859  |
|           |           |           |           |           | (n=9365)\ |
|           |           |           |           |           | ",\"femal |
|           |           |           |           |           | e<br>Prop |
|           |           |           |           |           | ortion:0. |
|           |           |           |           |           | 4141 (n=6 |
|           |           |           |           |           | 619)\"][f |
|           |           |           |           |           | ield[0].o |
|           |           |           |           |           | ffset];\n |
|           |           |           |           |           |        }" |
|           |           |           |           |           | ,"height" |
|           |           |           |           |           | :20,"widt |
|           |           |           |           |           | h":15},"w |
|           |           |           |           |           | idth":15, |
|           |           |           |           |           | "height": |
|           |           |           |           |           | 20},"eval |
|           |           |           |           |           | s":["opti |
|           |           |           |           |           | ons.toolt |
|           |           |           |           |           | ipFormatt |
|           |           |           |           |           | er"],"jsH |
|           |           |           |           |           | ooks":[]} |
|           |           |           |           |           | </script> |
|           |           |           |           |           | ```       |
+-----------+-----------+-----------+-----------+-----------+-----------+

```{=html}

</div>
```

</div>
:::

# Inspect missing data

`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
d <- select(vcte0, ast, alt, ggt, chol, plat, gluc)
missChk(d, prednmiss=TRUE, omitpred='id')
```

```{=html}
</details>
```
0 variables have no NAs and 6 variables have NAs

d has 15984 observations (10764 complete) and 6 variables (0 complete)

                      Minimum   Maximum     Mean
  ----------------- --------- --------- --------
  Per variable           1102      3836   2342.3
  Per observation           0         6      0.9

  : Number of NAs

    1102   1561   2005   2218   3332   3836
  ------ ------ ------ ------ ------ ------
       1      1      1      1      1      1

  : Frequency distribution of number of NAs per variable

        0      1      2     3     4     5      6
  ------- ------ ------ ----- ----- ----- ------
    10764   1594   1875   439   189   101   1022

  : Frequency distribution of number of incomplete variables per
  observation

    Loading required namespace: plotly

::: panel-tabset
## 

## NAs/var

![](LiverRisk_val_files/figure-markdown/fig-missingness1-1.png)

## NAs/obs

![](LiverRisk_val_files/figure-markdown/chnk4-1.png)

## Mean \# var NA when other var NA

![](LiverRisk_val_files/figure-markdown/chnk5-1.png)

## NAs/var vs mean \# other variables NA

![](LiverRisk_val_files/figure-markdown/chnk6-1.png)

## Clustering of missingness

![](LiverRisk_val_files/figure-markdown/chnk7-1.png)

## Sequential

     ggt   gluc   ast   chol   plat   alt
  ------ ------ ----- ------ ------ -----
    3836    705   331    253     81    14

  : Sequential frequency-ordered exclusions due to NAs

## NA combinations

    Warning: Ignoring 2 observations

```{=html}
<div id="htmlwidget-ad50b350417ab6357f19" style="width:607px;height:250px;" class="plotly html-widget"></div>
<script type="application/json" data-for="htmlwidget-ad50b350417ab6357f19">{"x":{"visdat":{"c608616286aa":["function () ","plotlyVisDat"]},"cur_data":"c608616286aa","attrs":{"c608616286aa":{"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"x":{},"y":{},"xend":{},"yend":{},"type":"scatter","mode":"lines","color":["gray80"],"line":{"width":0.75},"hoverinfo":"none","showlegend":false,"inherit":true},"c608616286aa.1":{"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"x":{},"y":{},"xend":{},"yend":{},"type":"scatter","mode":"lines","color":["gray80"],"line":{"width":0.75},"hoverinfo":"none","showlegend":false,"inherit":true},"c608616286aa.2":{"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"x":{},"y":{},"type":"scatter","mode":"markers","text":{},"hoverinfo":"text","color":["black"],"size":[35],"showlegend":false,"inherit":true},"c608616286aa.3":{"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"x":{},"y":{},"xend":{},"yend":{},"type":"scatter","mode":"lines","text":{},"hoverinfo":"text","color":["blue"],"name":"Marginal Counts","showlegend":true,"line":{"width":3},"inherit":true},"c608616286aa.4":{"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"x":{},"y":{},"xend":{},"yend":{},"type":"scatter","mode":"lines","text":{},"hoverinfo":"text","color":["black"],"name":"Combination Counts","showlegend":true,"line":{"width":3},"inherit":true},"c608616286aa.5":{"alpha_stroke":1,"sizes":[10,100],"spans":[1,20],"x":{},"y":[1,2,3,4,5,6],"text":{},"type":"scatter","mode":"text","textposition":"middle right","hoverinfo":"none","showlegend":false,"inherit":true}},"layout":{"width":607,"height":250,"margin":{"b":40,"l":60,"t":25,"r":10},"xaxis":{"domain":[0,1],"automargin":true,"title":"","tickvals":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25],"range":[-2,26.600000000000001],"showgrid":false,"showticklabels":false,"zeroline":false},"yaxis":{"domain":[0,1],"automargin":true,"title":"","tickvals":[1,2,3,4,5,6],"showgrid":false,"showticklabels":false},"legend":{"x":0.5,"y":0,"xanchor":"center","yanchor":"top","orientation":"h"},"hovermode":"closest","showlegend":true},"source":"A","config":{"modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"data":[{"x":[-2,25,null,-2,25,null,-2,25,null,-2,25,null,-2,25,null,-2,25],"y":[1,1,null,2,2,null,3,3,null,4,4,null,5,5,null,6,6],"type":"scatter","mode":"lines","line":{"color":"rgba(204,204,204,1)","width":0.75},"hoverinfo":["none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none"],"showlegend":false,"marker":{"color":"rgba(204,204,204,1)","line":{"color":"rgba(204,204,204,1)"}},"textfont":{"color":"rgba(204,204,204,1)"},"error_y":{"color":"rgba(204,204,204,1)"},"error_x":{"color":"rgba(204,204,204,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[1,1,null,2,2,null,3,3,null,4,4,null,5,5,null,6,6,null,7,7,null,8,8,null,9,9,null,10,10,null,11,11,null,12,12,null,13,13,null,14,14,null,15,15,null,16,16,null,17,17,null,18,18,null,19,19,null,20,20,null,21,21,null,22,22,null,23,23,null,24,24,null,25,25],"y":[1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5,null,1,7.5],"type":"scatter","mode":"lines","line":{"color":"rgba(204,204,204,1)","width":0.75},"hoverinfo":["none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none",null,"none","none"],"showlegend":false,"marker":{"color":"rgba(204,204,204,1)","line":{"color":"rgba(204,204,204,1)"}},"textfont":{"color":"rgba(204,204,204,1)"},"error_y":{"color":"rgba(204,204,204,1)"},"error_x":{"color":"rgba(204,204,204,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[1,1,2,2,2,2,2,2,3,4,5,6,7,7,8,8,8,9,9,9,9,10,10,10,11,12,12,13,13,14,14,14,14,14,15,15,15,16,16,16,17,17,17,18,18,18,19,19,19,19,20,20,20,20,21,21,21,21,21,22,22,23,24,24,25,25],"y":[5,6,1,2,3,4,5,6,6,4,3,5,3,4,3,4,6,3,4,5,6,2,5,6,2,2,5,4,6,2,3,4,5,6,2,3,4,3,4,5,3,5,6,4,5,6,2,4,5,6,2,3,4,5,1,3,4,5,6,4,5,1,2,4,2,6],"type":"scatter","mode":"markers","text":["<b>ast<br>ggt<\/b><br><br>Count: 1470<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>15984<\/sub><\/span> = 0.092<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>5220<\/sub><\/span> = 0.282<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>3332<\/sub><\/span> = 0.441","<b>ast<br>ggt<\/b><br><br>Count: 1470<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>15984<\/sub><\/span> = 0.092<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>5220<\/sub><\/span> = 0.282<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>3836<\/sub><\/span> = 0.383","<b>alt<br>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 1022<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>15984<\/sub><\/span> = 0.064<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>5220<\/sub><\/span> = 0.196<br>Fraction of subjects w/alt: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>1102<\/sub><\/span> = 0.927","<b>alt<br>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 1022<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>15984<\/sub><\/span> = 0.064<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>5220<\/sub><\/span> = 0.196<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>1561<\/sub><\/span> = 0.655","<b>alt<br>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 1022<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>15984<\/sub><\/span> = 0.064<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>5220<\/sub><\/span> = 0.196<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>2005<\/sub><\/span> = 0.51","<b>alt<br>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 1022<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>15984<\/sub><\/span> = 0.064<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>5220<\/sub><\/span> = 0.196<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>2218<\/sub><\/span> = 0.461","<b>alt<br>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 1022<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>15984<\/sub><\/span> = 0.064<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>5220<\/sub><\/span> = 0.196<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>3332<\/sub><\/span> = 0.307","<b>alt<br>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 1022<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>15984<\/sub><\/span> = 0.064<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>5220<\/sub><\/span> = 0.196<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>3836<\/sub><\/span> = 0.266","<b>ggt<\/b><br><br>Count: 658<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>658<\/sup>⁄<sub>15984<\/sub><\/span> = 0.041<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>658<\/sup>⁄<sub>5220<\/sub><\/span> = 0.126<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>658<\/sup>⁄<sub>3836<\/sub><\/span> = 0.172","<b>gluc<\/b><br><br>Count: 349<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>349<\/sup>⁄<sub>15984<\/sub><\/span> = 0.022<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>349<\/sup>⁄<sub>5220<\/sub><\/span> = 0.067<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>349<\/sup>⁄<sub>2218<\/sub><\/span> = 0.157","<b>chol<\/b><br><br>Count: 250<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>250<\/sup>⁄<sub>15984<\/sub><\/span> = 0.016<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>250<\/sup>⁄<sub>5220<\/sub><\/span> = 0.048<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>250<\/sup>⁄<sub>2005<\/sub><\/span> = 0.125","<b>ast<\/b><br><br>Count: 245<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>245<\/sup>⁄<sub>15984<\/sub><\/span> = 0.015<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>245<\/sup>⁄<sub>5220<\/sub><\/span> = 0.047<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>245<\/sup>⁄<sub>3332<\/sub><\/span> = 0.074","<b>chol<br>gluc<\/b><br><br>Count: 196<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>15984<\/sub><\/span> = 0.012<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>5220<\/sub><\/span> = 0.038<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>2005<\/sub><\/span> = 0.098","<b>chol<br>gluc<\/b><br><br>Count: 196<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>15984<\/sub><\/span> = 0.012<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>5220<\/sub><\/span> = 0.038<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>2218<\/sub><\/span> = 0.088","<b>chol<br>gluc<br>ggt<\/b><br><br>Count: 135<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>15984<\/sub><\/span> = 0.008<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>5220<\/sub><\/span> = 0.026<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>2005<\/sub><\/span> = 0.067","<b>chol<br>gluc<br>ggt<\/b><br><br>Count: 135<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>15984<\/sub><\/span> = 0.008<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>5220<\/sub><\/span> = 0.026<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>2218<\/sub><\/span> = 0.061","<b>chol<br>gluc<br>ggt<\/b><br><br>Count: 135<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>15984<\/sub><\/span> = 0.008<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>5220<\/sub><\/span> = 0.026<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>3836<\/sub><\/span> = 0.035","<b>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 114<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>5220<\/sub><\/span> = 0.022<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>2005<\/sub><\/span> = 0.057","<b>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 114<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>5220<\/sub><\/span> = 0.022<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>2218<\/sub><\/span> = 0.051","<b>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 114<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>5220<\/sub><\/span> = 0.022<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>3332<\/sub><\/span> = 0.034","<b>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 114<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>5220<\/sub><\/span> = 0.022<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>3836<\/sub><\/span> = 0.03","<b>plat<br>ast<br>ggt<\/b><br><br>Count: 112<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>5220<\/sub><\/span> = 0.021<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>1561<\/sub><\/span> = 0.072","<b>plat<br>ast<br>ggt<\/b><br><br>Count: 112<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>5220<\/sub><\/span> = 0.021<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>3332<\/sub><\/span> = 0.034","<b>plat<br>ast<br>ggt<\/b><br><br>Count: 112<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>5220<\/sub><\/span> = 0.021<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>3836<\/sub><\/span> = 0.029","<b>plat<\/b><br><br>Count: 78<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>78<\/sup>⁄<sub>15984<\/sub><\/span> = 0.005<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>78<\/sup>⁄<sub>5220<\/sub><\/span> = 0.015<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>78<\/sup>⁄<sub>1561<\/sub><\/span> = 0.05","<b>plat<br>ast<\/b><br><br>Count: 74<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>15984<\/sub><\/span> = 0.005<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>5220<\/sub><\/span> = 0.014<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>1561<\/sub><\/span> = 0.047","<b>plat<br>ast<\/b><br><br>Count: 74<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>15984<\/sub><\/span> = 0.005<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>5220<\/sub><\/span> = 0.014<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>3332<\/sub><\/span> = 0.022","<b>gluc<br>ggt<\/b><br><br>Count: 71<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>5220<\/sub><\/span> = 0.014<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>2218<\/sub><\/span> = 0.032","<b>gluc<br>ggt<\/b><br><br>Count: 71<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>5220<\/sub><\/span> = 0.014<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>3836<\/sub><\/span> = 0.019","<b>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 70<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>1561<\/sub><\/span> = 0.045","<b>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 70<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>2005<\/sub><\/span> = 0.035","<b>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 70<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>2218<\/sub><\/span> = 0.032","<b>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 70<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>3332<\/sub><\/span> = 0.021","<b>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 70<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>3836<\/sub><\/span> = 0.018","<b>plat<br>chol<br>gluc<\/b><br><br>Count: 68<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>1561<\/sub><\/span> = 0.044","<b>plat<br>chol<br>gluc<\/b><br><br>Count: 68<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>2005<\/sub><\/span> = 0.034","<b>plat<br>chol<br>gluc<\/b><br><br>Count: 68<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>2218<\/sub><\/span> = 0.031","<b>chol<br>gluc<br>ast<\/b><br><br>Count: 32<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>2005<\/sub><\/span> = 0.016","<b>chol<br>gluc<br>ast<\/b><br><br>Count: 32<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>2218<\/sub><\/span> = 0.014","<b>chol<br>gluc<br>ast<\/b><br><br>Count: 32<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>3332<\/sub><\/span> = 0.01","<b>chol<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>2005<\/sub><\/span> = 0.015","<b>chol<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>3332<\/sub><\/span> = 0.009","<b>chol<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>3836<\/sub><\/span> = 0.008","<b>gluc<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>2218<\/sub><\/span> = 0.014","<b>gluc<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>3332<\/sub><\/span> = 0.009","<b>gluc<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>3836<\/sub><\/span> = 0.008","<b>plat<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 24<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>5220<\/sub><\/span> = 0.005<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>1561<\/sub><\/span> = 0.015","<b>plat<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 24<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>5220<\/sub><\/span> = 0.005<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>2218<\/sub><\/span> = 0.011","<b>plat<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 24<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>5220<\/sub><\/span> = 0.005<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>3332<\/sub><\/span> = 0.007","<b>plat<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 24<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>5220<\/sub><\/span> = 0.005<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>3836<\/sub><\/span> = 0.006","<b>plat<br>chol<br>gluc<br>ast<\/b><br><br>Count: 20<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>5220<\/sub><\/span> = 0.004<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>1561<\/sub><\/span> = 0.013","<b>plat<br>chol<br>gluc<br>ast<\/b><br><br>Count: 20<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>5220<\/sub><\/span> = 0.004<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>2005<\/sub><\/span> = 0.01","<b>plat<br>chol<br>gluc<br>ast<\/b><br><br>Count: 20<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>5220<\/sub><\/span> = 0.004<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>2218<\/sub><\/span> = 0.009","<b>plat<br>chol<br>gluc<br>ast<\/b><br><br>Count: 20<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>5220<\/sub><\/span> = 0.004<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>3332<\/sub><\/span> = 0.006","<b>alt<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 16<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003<br>Fraction of subjects w/alt: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>1102<\/sub><\/span> = 0.015","<b>alt<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 16<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003<br>Fraction of subjects w/chol: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>2005<\/sub><\/span> = 0.008","<b>alt<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 16<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>2218<\/sub><\/span> = 0.007","<b>alt<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 16<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>3332<\/sub><\/span> = 0.005","<b>alt<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 16<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>3836<\/sub><\/span> = 0.004","<b>gluc<br>ast<\/b><br><br>Count: 15<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>2218<\/sub><\/span> = 0.007","<b>gluc<br>ast<\/b><br><br>Count: 15<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003<br>Fraction of subjects w/ast: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>3332<\/sub><\/span> = 0.005","<b>alt<\/b><br><br>Count: 14<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>14<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>14<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003<br>Fraction of subjects w/alt: <span style=\"font-size: 82%;\"><sup>14<\/sup>⁄<sub>1102<\/sub><\/span> = 0.013","<b>plat<br>gluc<\/b><br><br>Count: 13<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>5220<\/sub><\/span> = 0.002<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>1561<\/sub><\/span> = 0.008","<b>plat<br>gluc<\/b><br><br>Count: 13<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>5220<\/sub><\/span> = 0.002<br>Fraction of subjects w/gluc: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>2218<\/sub><\/span> = 0.006","<b>plat<br>ggt<\/b><br><br>Count: 12<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>5220<\/sub><\/span> = 0.002<br>Fraction of subjects w/plat: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>1561<\/sub><\/span> = 0.008","<b>plat<br>ggt<\/b><br><br>Count: 12<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>5220<\/sub><\/span> = 0.002<br>Fraction of subjects w/ggt: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>3836<\/sub><\/span> = 0.003"],"hoverinfo":["text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text","text"],"showlegend":false,"marker":{"color":"rgba(0,0,0,1)","size":[35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35,35],"sizemode":"area","line":{"color":"rgba(0,0,0,1)"}},"textfont":{"color":"rgba(0,0,0,1)","size":35},"error_y":{"color":"rgba(0,0,0,1)","width":35},"error_x":{"color":"rgba(0,0,0,1)","width":35},"line":{"color":"rgba(0,0,0,1)","width":35},"xaxis":"x","yaxis":"y","frame":null},{"x":[0,-0.57455683003128255,null,0,-0.81386861313868608,null,0,-1.0453597497393117,null,0,-1.1564129301355579,null,0,-1.7372262773722629,null,0,-2],"y":[1,1,null,2,2,null,3,3,null,4,4,null,5,5,null,6,6],"type":"scatter","mode":"lines","text":["<b>alt<\/b><br><br>Marginal count: 1102<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1102<\/sup>⁄<sub>15984<\/sub><\/span> = 0.069<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1102<\/sup>⁄<sub>5220<\/sub><\/span> = 0.211","<b>alt<\/b><br><br>Marginal count: 1102<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1102<\/sup>⁄<sub>15984<\/sub><\/span> = 0.069<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1102<\/sup>⁄<sub>5220<\/sub><\/span> = 0.211",null,"<b>plat<\/b><br><br>Marginal count: 1561<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1561<\/sup>⁄<sub>15984<\/sub><\/span> = 0.098<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1561<\/sup>⁄<sub>5220<\/sub><\/span> = 0.299","<b>plat<\/b><br><br>Marginal count: 1561<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1561<\/sup>⁄<sub>15984<\/sub><\/span> = 0.098<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1561<\/sup>⁄<sub>5220<\/sub><\/span> = 0.299",null,"<b>chol<\/b><br><br>Marginal count: 2005<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>2005<\/sup>⁄<sub>15984<\/sub><\/span> = 0.125<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>2005<\/sup>⁄<sub>5220<\/sub><\/span> = 0.384","<b>chol<\/b><br><br>Marginal count: 2005<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>2005<\/sup>⁄<sub>15984<\/sub><\/span> = 0.125<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>2005<\/sup>⁄<sub>5220<\/sub><\/span> = 0.384",null,"<b>gluc<\/b><br><br>Marginal count: 2218<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>2218<\/sup>⁄<sub>15984<\/sub><\/span> = 0.139<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>2218<\/sup>⁄<sub>5220<\/sub><\/span> = 0.425","<b>gluc<\/b><br><br>Marginal count: 2218<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>2218<\/sup>⁄<sub>15984<\/sub><\/span> = 0.139<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>2218<\/sup>⁄<sub>5220<\/sub><\/span> = 0.425",null,"<b>ast<\/b><br><br>Marginal count: 3332<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>3332<\/sup>⁄<sub>15984<\/sub><\/span> = 0.208<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>3332<\/sup>⁄<sub>5220<\/sub><\/span> = 0.638","<b>ast<\/b><br><br>Marginal count: 3332<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>3332<\/sup>⁄<sub>15984<\/sub><\/span> = 0.208<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>3332<\/sup>⁄<sub>5220<\/sub><\/span> = 0.638",null,"<b>ggt<\/b><br><br>Marginal count: 3836<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>3836<\/sup>⁄<sub>15984<\/sub><\/span> = 0.24<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>3836<\/sup>⁄<sub>5220<\/sub><\/span> = 0.735","<b>ggt<\/b><br><br>Marginal count: 3836<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>3836<\/sup>⁄<sub>15984<\/sub><\/span> = 0.24<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>3836<\/sup>⁄<sub>5220<\/sub><\/span> = 0.735"],"hoverinfo":["text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text"],"name":"Marginal Counts","showlegend":true,"line":{"color":"rgba(0,0,255,1)","width":3},"marker":{"color":"rgba(0,0,255,1)","line":{"color":"rgba(0,0,255,1)"}},"textfont":{"color":"rgba(0,0,255,1)"},"error_y":{"color":"rgba(0,0,255,1)"},"error_x":{"color":"rgba(0,0,255,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[1,1,null,2,2,null,3,3,null,4,4,null,5,5,null,6,6,null,7,7,null,8,8,null,9,9,null,10,10,null,11,11,null,12,12,null,13,13,null,14,14,null,15,15,null,16,16,null,17,17,null,18,18,null,19,19,null,20,20,null,21,21,null,22,22,null,23,23,null,24,24,null,25,25],"y":[6.5,7.5,null,6.5,7.1952380952380954,null,6.5,6.9476190476190478,null,6.5,6.7374149659863942,null,6.5,6.6700680272108848,null,6.5,6.666666666666667,null,6.5,6.6333333333333337,null,6.5,6.591836734693878,null,6.5,6.5775510204081629,null,6.5,6.5761904761904759,null,6.5,6.5530612244897961,null,6.5,6.5503401360544213,null,6.5,6.5482993197278914,null,6.5,6.5476190476190474,null,6.5,6.5462585034013605,null,6.5,6.5217687074829929,null,6.5,6.5210884353741498,null,6.5,6.5210884353741498,null,6.5,6.5163265306122451,null,6.5,6.5136054421768703,null,6.5,6.5108843537414964,null,6.5,6.5102040816326534,null,6.5,6.5095238095238095,null,6.5,6.5088435374149656,null,6.5,6.5081632653061225],"type":"scatter","mode":"lines","text":["<b>ast<br>ggt<\/b><br><br>Count: 1470<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>15984<\/sub><\/span> = 0.092<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>5220<\/sub><\/span> = 0.282","<b>ast<br>ggt<\/b><br><br>Count: 1470<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>15984<\/sub><\/span> = 0.092<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1470<\/sup>⁄<sub>5220<\/sub><\/span> = 0.282",null,"<b>alt<br>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 1022<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>15984<\/sub><\/span> = 0.064<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>5220<\/sub><\/span> = 0.196","<b>alt<br>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 1022<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>15984<\/sub><\/span> = 0.064<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>1022<\/sup>⁄<sub>5220<\/sub><\/span> = 0.196",null,"<b>ggt<\/b><br><br>Count: 658<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>658<\/sup>⁄<sub>15984<\/sub><\/span> = 0.041<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>658<\/sup>⁄<sub>5220<\/sub><\/span> = 0.126","<b>ggt<\/b><br><br>Count: 658<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>658<\/sup>⁄<sub>15984<\/sub><\/span> = 0.041<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>658<\/sup>⁄<sub>5220<\/sub><\/span> = 0.126",null,"<b>gluc<\/b><br><br>Count: 349<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>349<\/sup>⁄<sub>15984<\/sub><\/span> = 0.022<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>349<\/sup>⁄<sub>5220<\/sub><\/span> = 0.067","<b>gluc<\/b><br><br>Count: 349<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>349<\/sup>⁄<sub>15984<\/sub><\/span> = 0.022<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>349<\/sup>⁄<sub>5220<\/sub><\/span> = 0.067",null,"<b>chol<\/b><br><br>Count: 250<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>250<\/sup>⁄<sub>15984<\/sub><\/span> = 0.016<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>250<\/sup>⁄<sub>5220<\/sub><\/span> = 0.048","<b>chol<\/b><br><br>Count: 250<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>250<\/sup>⁄<sub>15984<\/sub><\/span> = 0.016<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>250<\/sup>⁄<sub>5220<\/sub><\/span> = 0.048",null,"<b>ast<\/b><br><br>Count: 245<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>245<\/sup>⁄<sub>15984<\/sub><\/span> = 0.015<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>245<\/sup>⁄<sub>5220<\/sub><\/span> = 0.047","<b>ast<\/b><br><br>Count: 245<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>245<\/sup>⁄<sub>15984<\/sub><\/span> = 0.015<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>245<\/sup>⁄<sub>5220<\/sub><\/span> = 0.047",null,"<b>chol<br>gluc<\/b><br><br>Count: 196<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>15984<\/sub><\/span> = 0.012<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>5220<\/sub><\/span> = 0.038","<b>chol<br>gluc<\/b><br><br>Count: 196<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>15984<\/sub><\/span> = 0.012<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>196<\/sup>⁄<sub>5220<\/sub><\/span> = 0.038",null,"<b>chol<br>gluc<br>ggt<\/b><br><br>Count: 135<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>15984<\/sub><\/span> = 0.008<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>5220<\/sub><\/span> = 0.026","<b>chol<br>gluc<br>ggt<\/b><br><br>Count: 135<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>15984<\/sub><\/span> = 0.008<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>135<\/sup>⁄<sub>5220<\/sub><\/span> = 0.026",null,"<b>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 114<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>5220<\/sub><\/span> = 0.022","<b>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 114<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>114<\/sup>⁄<sub>5220<\/sub><\/span> = 0.022",null,"<b>plat<br>ast<br>ggt<\/b><br><br>Count: 112<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>5220<\/sub><\/span> = 0.021","<b>plat<br>ast<br>ggt<\/b><br><br>Count: 112<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>15984<\/sub><\/span> = 0.007<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>112<\/sup>⁄<sub>5220<\/sub><\/span> = 0.021",null,"<b>plat<\/b><br><br>Count: 78<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>78<\/sup>⁄<sub>15984<\/sub><\/span> = 0.005<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>78<\/sup>⁄<sub>5220<\/sub><\/span> = 0.015","<b>plat<\/b><br><br>Count: 78<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>78<\/sup>⁄<sub>15984<\/sub><\/span> = 0.005<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>78<\/sup>⁄<sub>5220<\/sub><\/span> = 0.015",null,"<b>plat<br>ast<\/b><br><br>Count: 74<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>15984<\/sub><\/span> = 0.005<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>5220<\/sub><\/span> = 0.014","<b>plat<br>ast<\/b><br><br>Count: 74<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>15984<\/sub><\/span> = 0.005<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>74<\/sup>⁄<sub>5220<\/sub><\/span> = 0.014",null,"<b>gluc<br>ggt<\/b><br><br>Count: 71<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>5220<\/sub><\/span> = 0.014","<b>gluc<br>ggt<\/b><br><br>Count: 71<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>71<\/sup>⁄<sub>5220<\/sub><\/span> = 0.014",null,"<b>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 70<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013","<b>plat<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 70<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>70<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013",null,"<b>plat<br>chol<br>gluc<\/b><br><br>Count: 68<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013","<b>plat<br>chol<br>gluc<\/b><br><br>Count: 68<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>15984<\/sub><\/span> = 0.004<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>68<\/sup>⁄<sub>5220<\/sub><\/span> = 0.013",null,"<b>chol<br>gluc<br>ast<\/b><br><br>Count: 32<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006","<b>chol<br>gluc<br>ast<\/b><br><br>Count: 32<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>32<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006",null,"<b>chol<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006","<b>chol<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006",null,"<b>gluc<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006","<b>gluc<br>ast<br>ggt<\/b><br><br>Count: 31<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>31<\/sup>⁄<sub>5220<\/sub><\/span> = 0.006",null,"<b>plat<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 24<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>5220<\/sub><\/span> = 0.005","<b>plat<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 24<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>15984<\/sub><\/span> = 0.002<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>24<\/sup>⁄<sub>5220<\/sub><\/span> = 0.005",null,"<b>plat<br>chol<br>gluc<br>ast<\/b><br><br>Count: 20<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>5220<\/sub><\/span> = 0.004","<b>plat<br>chol<br>gluc<br>ast<\/b><br><br>Count: 20<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>20<\/sup>⁄<sub>5220<\/sub><\/span> = 0.004",null,"<b>alt<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 16<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003","<b>alt<br>chol<br>gluc<br>ast<br>ggt<\/b><br><br>Count: 16<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>16<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003",null,"<b>gluc<br>ast<\/b><br><br>Count: 15<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003","<b>gluc<br>ast<\/b><br><br>Count: 15<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>15<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003",null,"<b>alt<\/b><br><br>Count: 14<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>14<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>14<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003","<b>alt<\/b><br><br>Count: 14<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>14<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>14<\/sup>⁄<sub>5220<\/sub><\/span> = 0.003",null,"<b>plat<br>gluc<\/b><br><br>Count: 13<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>5220<\/sub><\/span> = 0.002","<b>plat<br>gluc<\/b><br><br>Count: 13<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>13<\/sup>⁄<sub>5220<\/sub><\/span> = 0.002",null,"<b>plat<br>ggt<\/b><br><br>Count: 12<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>5220<\/sub><\/span> = 0.002","<b>plat<br>ggt<\/b><br><br>Count: 12<br>Fraction of subjects: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>15984<\/sub><\/span> = 0.001<br>Fraction of subjects w/any cond: <span style=\"font-size: 82%;\"><sup>12<\/sup>⁄<sub>5220<\/sub><\/span> = 0.002"],"hoverinfo":["text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text",null,"text","text"],"name":"Combination Counts","showlegend":true,"line":{"color":"rgba(0,0,0,1)","width":3},"marker":{"color":"rgba(0,0,0,1)","line":{"color":"rgba(0,0,0,1)"}},"textfont":{"color":"rgba(0,0,0,1)"},"error_y":{"color":"rgba(0,0,0,1)"},"error_x":{"color":"rgba(0,0,0,1)"},"xaxis":"x","yaxis":"y","frame":null},{"x":[25.699999999999999,25.699999999999999,25.699999999999999,25.699999999999999,25.699999999999999,25.699999999999999],"y":[1,2,3,4,5,6],"text":["alt","plat","chol","gluc","ast","ggt"],"type":"scatter","mode":"text","textposition":["middle right","middle right","middle right","middle right","middle right","middle right"],"hoverinfo":["none","none","none","none","none","none"],"showlegend":false,"marker":{"color":"rgba(140,86,75,1)","line":{"color":"rgba(140,86,75,1)"}},"error_y":{"color":"rgba(140,86,75,1)"},"error_x":{"color":"rgba(140,86,75,1)"},"line":{"color":"rgba(140,86,75,1)"},"xaxis":"x","yaxis":"y","frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.20000000000000001,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
```
:::

## Impute missing predictor values using `aregImpute` and calculate the average LiverRisk per patient over imputations. For later comparison, also calculate FIB-4.

::: cell
`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
if(!file.exists('LR_imp.rds')) {

  set.seed(8675309)

  vcte_imp <- aregImpute(lsm ~ ast + alt + ggt + chol + plat + gluc, vcte0, n.impute = 10)

  saveRDS(vcte_imp, 'LR_imp.rds')
  } else vcte_imp <- readRDS('LR_imp.rds')

vcte_stack <- lapply(1:vcte_imp$n.impute, function(j) mutate(completer(vcte_imp, j, T), imp = j)) |> do.call(what = rbind)

lrisk_df <- select(vcte_stack, sex, age, ast, alt, ggt, gluc, chol, plat)
vcte_stack$liverrisk <- liverrisk(lrisk_df) 
lr_imp <- vcte_stack |> group_by(id) |> 
    summarise(liverrisk = mean(liverrisk),
              fib4 = mean(ast * age / plat / sqrt(alt)))
vcte0$liverrisk <- lr_imp$liverrisk
vcte0$fib4 <- lr_imp$fib4
```

```{=html}
</details>
```
:::

# Results

::: cell
`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
describe(select(vcte0, lsm, liverrisk))
```

```{=html}
</details>
```
::: cell-output-display
```{=html}
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

<style>
.earrows {color:silver;font-size:11px;}

fcap {
 font-family: Verdana;
 font-size: 12px;
 color: MidnightBlue
 }

smg {
 font-family: Verdana;
 font-size: 10px;
 color: &#808080;
}

hr.thinhr { margin-top: 0.15em; margin-bottom: 0.15em; }

span.xscript {
position: relative;
}
span.xscript sub {
position: absolute;
left: 0.1em;
bottom: -1ex;
}
</style>

<title>select(vcte0, lsm, liverrisk) Descriptives</title>
<font color="MidnightBlue"><div align=center><span style="font-weight:bold">select(vcte0, lsm, liverrisk) <br><br> 2  Variables   15984  Observations</span></div></font>
<hr class="thinhr">
<span style="font-weight:bold">lsm</span>: Liver Stiffness Measure (kPa)<div style='float: right; text-align: right;'><img src="data:image/png;base64,
iVBORw0KGgoAAAANSUhEUgAAAJcAAAANCAYAAACkYvxcAAAEDmlDQ1BrQ0dDb2xvclNwYWNlR2VuZXJpY1JHQgAAOI2NVV1oHFUUPpu5syskzoPUpqaSDv41lLRsUtGE2uj+ZbNt3CyTbLRBkMns3Z1pJjPj/KRpKT4UQRDBqOCT4P9bwSchaqvtiy2itFCiBIMo+ND6R6HSFwnruTOzu5O4a73L3PnmnO9+595z7t4LkLgsW5beJQIsGq4t5dPis8fmxMQ6dMF90A190C0rjpUqlSYBG+PCv9rt7yDG3tf2t/f/Z+uuUEcBiN2F2Kw4yiLiZQD+FcWyXYAEQfvICddi+AnEO2ycIOISw7UAVxieD/Cyz5mRMohfRSwoqoz+xNuIB+cj9loEB3Pw2448NaitKSLLRck2q5pOI9O9g/t/tkXda8Tbg0+PszB9FN8DuPaXKnKW4YcQn1Xk3HSIry5ps8UQ/2W5aQnxIwBdu7yFcgrxPsRjVXu8HOh0qao30cArp9SZZxDfg3h1wTzKxu5E/LUxX5wKdX5SnAzmDx4A4OIqLbB69yMesE1pKojLjVdoNsfyiPi45hZmAn3uLWdpOtfQOaVmikEs7ovj8hFWpz7EV6mel0L9Xy23FMYlPYZenAx0yDB1/PX6dledmQjikjkXCxqMJS9WtfFCyH9XtSekEF+2dH+P4tzITduTygGfv58a5VCTH5PtXD7EFZiNyUDBhHnsFTBgE0SQIA9pfFtgo6cKGuhooeilaKH41eDs38Ip+f4At1Rq/sjr6NEwQqb/I/DQqsLvaFUjvAx+eWirddAJZnAj1DFJL0mSg/gcIpPkMBkhoyCSJ8lTZIxk0TpKDjXHliJzZPO50dR5ASNSnzeLvIvod0HG/mdkmOC0z8VKnzcQ2M/Yz2vKldduXjp9bleLu0ZWn7vWc+l0JGcaai10yNrUnXLP/8Jf59ewX+c3Wgz+B34Df+vbVrc16zTMVgp9um9bxEfzPU5kPqUtVWxhs6OiWTVW+gIfywB9uXi7CGcGW/zk98k/kmvJ95IfJn/j3uQ+4c5zn3Kfcd+AyF3gLnJfcl9xH3OfR2rUee80a+6vo7EK5mmXUdyfQlrYLTwoZIU9wsPCZEtP6BWGhAlhL3p2N6sTjRdduwbHsG9kq32sgBepc+xurLPW4T9URpYGJ3ym4+8zA05u44QjST8ZIoVtu3qE7fWmdn5LPdqvgcZz8Ww8BWJ8X3w0PhQ/wnCDGd+LvlHs8dRy6bLLDuKMaZ20tZrqisPJ5ONiCq8yKhYM5cCgKOu66Lsc0aYOtZdo5QCwezI4wm9J/v0X23mlZXOfBjj8Jzv3WrY5D+CsA9D7aMs2gGfjve8ArD6mePZSeCfEYt8CONWDw8FXTxrPqx/r9Vt4biXeANh8vV7/+/16ffMD1N8AuKD/A/8leAvFY9bLAAAAOGVYSWZNTQAqAAAACAABh2kABAAAAAEAAAAaAAAAAAACoAIABAAAAAEAAACXoAMABAAAAAEAAAANAAAAABXzt1gAAAMDSURBVFgJ7VlNSGpBFP4s+9O0n03bQhBLsKUQuqhFLSJX9uM2iDBylwYtXLYWgyBXuamIaBdtWkWbdrlKjYoiUClblFSU1XvfgKLQG+Ut3uLdOTDcmTk/M/Pdc889M6PLZDLfqEHpdBrDw8P4+vqC3+/HysoK9Hp9DS3F1joCuu/fVAuEk5MTuN3uslgikYDD4Si3VUUh8BMCDT911ur7/PzEzc1NLTHF1zgCf+VcoVAIfX19eHh40Dh8avkyBOpyrqurqyobR0dH4N90a2urql81FAKVCNTMuc7PzzEwMFCpU1V/fHxEV1dXVZ9qKASIgJ7JuYzoXDKivnIuGULa5ekHBwelq2fyLqOnpyfs7u5ifX1dJqZ4GkRAelhVLBYRj8elsExPT+Pt7Q1jY2M4PT3F6uqqVF4xtYOANOdKJpPo7++vCw2j0YiWlhbk8/m65JXQ/4+ANHLVcb5aRojRi/Lj4+Po6enBy8sLdnZ2sLe3B7PZjNHR0bKsqmgDgT9GLuZSdrsdd3d3dSHR2NgoroT4ZGltbRVOdn9/L/T52/R6vbi4uMDh4SE8Hg9GRkZgsVjAHefl5SV6e3thMpmEMzY0VJ+S0HHpsIyQMnp/f0dTUxN0Op1MTPH+AQK6yrvFpaUl3N7e4vX1FalUCs/Pz3VPgc7A+0Y+WfhyWSqjH+8mmcd9fHyUXz512K4kOkd7e7twJOrTDp2dcnQu8lioy18x+dx40HYulxMyrFOGYzY3N4t5UI6OTwdkP/VZ54fAJ21xDPJIHR0dwnZpXaX1MApfX1+DaQP75ubmsL29LcYuraM0J+rSHgvn1NbWJnbXk5OTiEajIu0YGhrC/v6+wI3rdblcKBQKODs7E31TU1M4Pj4WtyK0QxlivLy8LDZSi4uLWFtbQyAQEDkv1z0/P4/NzU34fD5sbGyIaVmtVthsNhwcHMDpdAosDAaDCCA8FZiZmcHExISwv7CwgGw2i3A4LAIA25x/MBgUY5Tea2dnJ2ZnZxGJRNDd3Y1YLFaCAL8AkgJJhD2dMfkAAAAASUVORK5CYII=" alt="image" /></div>
<style>
 .hmisctable542165 {
 border: none;
 font-size: 85%;
 }
 .hmisctable542165 td {
 text-align: center;
 padding: 0 1ex 0 1ex;
 }
 .hmisctable542165 th {
 color: MidnightBlue;
 text-align: center;
 padding: 0 1ex 0 1ex;
 font-weight: normal;
 }
 </style>
 <table data-quarto-disable-processing="true" class="hmisctable542165">
 <tr><th>n</th><th>missing</th><th>distinct</th><th>Info</th><th>Mean</th><th>Gmd</th><th>.05</th><th>.10</th><th>.25</th><th>.50</th><th>.75</th><th>.90</th><th>.95</th></tr>
 <tr><td>15984</td><td>0</td><td>384</td><td>1</td><td>7.743</td><td>4.991</td><td> 3.5</td><td> 3.9</td><td> 4.6</td><td> 5.9</td><td> 8.2</td><td>12.6</td><td>17.8</td></tr>
 </table>

<span style="font-size: 85%;"><font color="MidnightBlue">lowest</font> : 0.9  1.3  1.5  1.6  1.8  ,  <font color="MidnightBlue">highest</font>: 72.6 73.2 73.5 74.6 75  </span>
<hr class="thinhr">
<span style="font-weight:bold">liverrisk</span><div style='float: right; text-align: right;'><img src="data:image/png;base64,
iVBORw0KGgoAAAANSUhEUgAAAJcAAAANCAYAAACkYvxcAAAEDmlDQ1BrQ0dDb2xvclNwYWNlR2VuZXJpY1JHQgAAOI2NVV1oHFUUPpu5syskzoPUpqaSDv41lLRsUtGE2uj+ZbNt3CyTbLRBkMns3Z1pJjPj/KRpKT4UQRDBqOCT4P9bwSchaqvtiy2itFCiBIMo+ND6R6HSFwnruTOzu5O4a73L3PnmnO9+595z7t4LkLgsW5beJQIsGq4t5dPis8fmxMQ6dMF90A190C0rjpUqlSYBG+PCv9rt7yDG3tf2t/f/Z+uuUEcBiN2F2Kw4yiLiZQD+FcWyXYAEQfvICddi+AnEO2ycIOISw7UAVxieD/Cyz5mRMohfRSwoqoz+xNuIB+cj9loEB3Pw2448NaitKSLLRck2q5pOI9O9g/t/tkXda8Tbg0+PszB9FN8DuPaXKnKW4YcQn1Xk3HSIry5ps8UQ/2W5aQnxIwBdu7yFcgrxPsRjVXu8HOh0qao30cArp9SZZxDfg3h1wTzKxu5E/LUxX5wKdX5SnAzmDx4A4OIqLbB69yMesE1pKojLjVdoNsfyiPi45hZmAn3uLWdpOtfQOaVmikEs7ovj8hFWpz7EV6mel0L9Xy23FMYlPYZenAx0yDB1/PX6dledmQjikjkXCxqMJS9WtfFCyH9XtSekEF+2dH+P4tzITduTygGfv58a5VCTH5PtXD7EFZiNyUDBhHnsFTBgE0SQIA9pfFtgo6cKGuhooeilaKH41eDs38Ip+f4At1Rq/sjr6NEwQqb/I/DQqsLvaFUjvAx+eWirddAJZnAj1DFJL0mSg/gcIpPkMBkhoyCSJ8lTZIxk0TpKDjXHliJzZPO50dR5ASNSnzeLvIvod0HG/mdkmOC0z8VKnzcQ2M/Yz2vKldduXjp9bleLu0ZWn7vWc+l0JGcaai10yNrUnXLP/8Jf59ewX+c3Wgz+B34Df+vbVrc16zTMVgp9um9bxEfzPU5kPqUtVWxhs6OiWTVW+gIfywB9uXi7CGcGW/zk98k/kmvJ95IfJn/j3uQ+4c5zn3Kfcd+AyF3gLnJfcl9xH3OfR2rUee80a+6vo7EK5mmXUdyfQlrYLTwoZIU9wsPCZEtP6BWGhAlhL3p2N6sTjRdduwbHsG9kq32sgBepc+xurLPW4T9URpYGJ3ym4+8zA05u44QjST8ZIoVtu3qE7fWmdn5LPdqvgcZz8Ww8BWJ8X3w0PhQ/wnCDGd+LvlHs8dRy6bLLDuKMaZ20tZrqisPJ5ONiCq8yKhYM5cCgKOu66Lsc0aYOtZdo5QCwezI4wm9J/v0X23mlZXOfBjj8Jzv3WrY5D+CsA9D7aMs2gGfjve8ArD6mePZSeCfEYt8CONWDw8FXTxrPqx/r9Vt4biXeANh8vV7/+/16ffMD1N8AuKD/A/8leAvFY9bLAAAAOGVYSWZNTQAqAAAACAABh2kABAAAAAEAAAAaAAAAAAACoAIABAAAAAEAAACXoAMABAAAAAEAAAANAAAAABXzt1gAAAKYSURBVFgJ7VlNS3JBFH7UNyMDNy2S0HARQUTQIkFw48dWAhGh3OUm0EWE6wpcBoKbaiO60SQQwY2bCv+AHwgR+LHRAj9AkMoCK3ubAS/6+nq90SZyZuHMOfOc69yHc+fMmSOqVqsfENBarRY2NzdRLBZht9vh9XoFWDHIJDMg+vhsQghYW1vDzc0NB72+vobRaORkNmAM/MuAIOeq1+tQKBQDtouLiyiXywM6JjAG+hkQ9wujxvl8fmiqUqkgHA4P6ZmCMdBjQNDONT8/j0aj0bPh+tnZWTw+PkIkEnE6NmAM9BgQtHM1m80efqBvt9vw+/3odrsDeiYwBggDf3K5HC8Tn9kk3t/fR2J2d3fp2ctms43EsInJZGBsWLRarYjFYrzsLC0t4fb2FlNTU7w4NjlZDPCGxVKpNNaxCF0ER3YwFh4ny3nGvS2vc11eXo6z5+aDwSCUSiVzMI4RNuANi3K5nGaDX6FJJpNhf38fBwcHmJ6e/oopw/4yBkY618bGBtLp9Lded2ZmBgsLCzAYDNDr9dBoNFCpVHh+fsbc3Ny3ns2Mfz4Dov7a4tbWFrLZLB4eHn7EyiUSCV0HyVbFYjFNGEjS8Pr6CqlUire3N6ysrIDslqTm2el0oNPpkMlksLy8jLu7O1gsFphMJmxvb9Ny1cnJCY6OjlAoFJBKpbC6ukplciFM7vNqtRqtmwYCATw9PUGtViORSND/Oj09hdPphMvlwsvLC0KhEIiONLfbTde1vr4Oh8NBdb0fcl1DnndxcUE/rvPzc9zf34Ngr66ucHx83IMO9OQcu7OzA61WO6D/n7C3twez2YxIJEIjBynXCWnRaJQmY4eHh0Nwcpb2eDw4OzsDudPka8lkEvF4HD6fj4P9Bcjv9azYDbFgAAAAAElFTkSuQmCC" alt="image" /></div>
<style>
 .hmisctable144132 {
 border: none;
 font-size: 74%;
 }
 .hmisctable144132 td {
 text-align: center;
 padding: 0 1ex 0 1ex;
 }
 .hmisctable144132 th {
 color: MidnightBlue;
 text-align: center;
 padding: 0 1ex 0 1ex;
 font-weight: normal;
 }
 </style>
 <table data-quarto-disable-processing="true" class="hmisctable144132">
 <tr><th>n</th><th>missing</th><th>distinct</th><th>Info</th><th>Mean</th><th>Gmd</th><th>.05</th><th>.10</th><th>.25</th><th>.50</th><th>.75</th><th>.90</th><th>.95</th></tr>
 <tr><td>15984</td><td>0</td><td>15984</td><td>1</td><td>6.827</td><td>2.072</td><td> 4.497</td><td> 4.875</td><td> 5.522</td><td> 6.394</td><td> 7.556</td><td> 9.046</td><td>10.322</td></tr>
 </table>

<span style="font-size: 85%;"><font color="MidnightBlue">lowest</font> : 2.3025  2.57277 2.82747 2.83219 2.84301 ,  <font color="MidnightBlue">highest</font>: 49.5469 50.328  55.6447 58.7999 63.3619</span>
<hr class="thinhr">
```
:::
:::

::: panel-tabset
## 

## Calibration curve

::: cell
`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
out <- vcte0$lsm>8
pos <- vcte0$liverrisk>8

sens <- sum(pos & out) / sum(out)
spec <- sum(!pos & !out) / sum(!out)
ppv <- sum(pos & out) / sum(pos)
npv <- sum(!pos & !out) / sum(!pos)

xt <- xtabs(~out+pos) |> prop.table()
lr_confusion <- c(sens = sens, spec = spec, ppv = ppv, npv = npv)

rect_df <- data.frame(xmin = c(0, 8), xmax = c(8, lsm_max), ymin = c(0, 8), ymax = c(8, lsm_max))
rect_df2 <- data.frame(xmin = c(0, 8), xmax = c(8, lsm_max), ymin = c(8, 0), ymax = c(lsm_max, 8))

lr_cal <- ggplot(vcte0, aes(liverrisk, lsm)) + 
  geom_rect(aes(x = NULL, y = NULL, xmin = xmin, xmax = xmax, ymin = ymin, ymax=ymax), data = rect_df, fill = "green", alpha = 0.2) +
  geom_rect(aes(x = NULL, y = NULL, xmin = xmin, xmax = xmax, ymin = ymin, ymax=ymax), data = rect_df2, fill = "red", alpha = 0.2) +
  geom_point(alpha = 0.4) + 
  geom_abline(intercept = 0, slope = 1, color = 2) + geom_smooth() + 
  geom_abline(slope = c(1.3, 0.7), intercept = c(0, 0), color = "red", linetype = 3) + 
  annotate("label", x = c(4, 4, 50, 50), y = c(4, 12, 4, 12), 
           label = paste0(c("TN", "FN", "FP", "TP"), ": ", round(100*c(xt)), "%")) + 
  annotate("text", x = c(48, 58), y = c(58, 44), 
           label = c("+30%", "-30%"), color = "red") + 
  scale_x_continuous(name = "Predicted LSM (LiverRisk)", limits = c(0, lsm_max), breaks = seq(0, lsm_max, by = 4)) + 
  scale_y_continuous(name = "Observed LSM", limits = c(0, lsm_max), breaks = seq(0, lsm_max, by = 4))
lr_cal
```

```{=html}
</details>
```
::: cell-output-display
![](LiverRisk_val_files/figure-markdown/liverrisk_cal-1.png)
:::
:::

## ROC-curve for predicting LSM \>8kPa

::: cell
`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
ROC <- function(test, outcome, score = NULL) {
  df <- data.frame(test = test, outcome = outcome) |> arrange(-test)
  k <- sum(df$outcome)
  l <- sum(!df$outcome)
  roc <- lapply(unique(round(df$test, 2)), function(x) {
    pos <- df$test > x
    sens <- sum(pos & df$outcome) / k
    spec <- sum(!pos & !df$outcome) / l
    sens_lower <- sens - qnorm(0.975) * sqrt(sens * (1 - sens) / k)
    sens_upper <- sens + qnorm(0.975) * sqrt(sens * (1 - sens) / k)
    c(sens = sens, spec = spec, sens_lower = sens_lower, sens_upper = sens_upper)
  }) |> do.call(what = rbind) |> as.data.frame()
  attr(roc, "AUC") <- survival::concordance(outcome~test, data = df)$concordance
  roc$score <- score
  roc
}

lr_roc <- ROC(vcte0$liverrisk, vcte0$lsm > 8, "LR")
lr_auc <- survival::concordance((lsm>8) ~ liverrisk, data = vcte0) 
f4_roc <- ROC(vcte0$fib4, vcte0$lsm > 8, "zFIB-4")
f4_auc <- survival::concordance((lsm>8) ~ fib4, data = vcte0) 

roc_legend <- paste0(c("LiverRisk", "FIB-4"), " (AUC = ", c(round(100*lr_auc$concordance, 1),
                       round(100*f4_auc$concordance, 1)), "%)")

ggplot(rbind(f4_roc, lr_roc), aes(1-spec, sens, color = score, fill = score)) + 
  geom_ribbon(aes(ymin = sens_lower, ymax = sens_upper), linewidth = 0, alpha = 0.4) + 
  geom_step() + 
  geom_abline(slope = 1, linetype = 3) + 
  scale_color_discrete("Model predicting LSM > 8kPa", labels = roc_legend) + 
  scale_fill_discrete("Model predicting LSM > 8kPa", labels = roc_legend) + 
  scale_x_continuous("1 - Specificity") + scale_y_continuous("Sensitivity") +
  theme(legend.text.align = 1, legend.position = c(0.6, 0.25)) 
```

```{=html}
</details>
```
::: cell-output-display
![](LiverRisk_val_files/figure-markdown/roc_revealed-1.png)
:::
:::

## ROC-curve for predicting LSM \>12kPa

::: cell
`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
lr_roc12 <- ROC(vcte0$liverrisk, vcte0$lsm > 12, "LR")
lr_auc12 <- survival::concordance((lsm>12) ~ liverrisk, data = vcte0) 
f4_roc12 <- ROC(vcte0$fib4, vcte0$lsm > 12, "zFIB-4")
f4_auc12 <- survival::concordance((lsm>12) ~ fib4, data = vcte0) 

roc_legend <- paste0(c("LiverRisk", "FIB-4"), " (AUC = ", c(round(100*lr_auc12$concordance, 1),
                       round(100*f4_auc12$concordance, 1)), "%)")

ggplot(rbind(f4_roc, lr_roc), aes(1-spec, sens, color = score, fill = score)) + 
  geom_ribbon(aes(ymin = sens_lower, ymax = sens_upper), linewidth = 0, alpha = 0.4) + 
  geom_step() + 
  geom_abline(slope = 1, linetype = 3) + 
  scale_color_discrete("Model predicting LSM > 12kPa", labels = roc_legend) + 
  scale_fill_discrete("Model predicting LSM > 12kPa", labels = roc_legend) + 
  scale_x_continuous("1 - Specificity") + scale_y_continuous("Sensitivity") +
  theme(legend.text.align = 1, legend.position = c(0.6, 0.25)) 
```

```{=html}
</details>
```
::: cell-output-display
![](LiverRisk_val_files/figure-markdown/roc_12-1.png)
:::
:::

## Goodness-of-fit statistics

::: cell
`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
lr_cal_fit <- ols(lsm ~ liverrisk, vcte0)
names(lr_cal_fit$coefficients) <- NULL

lr_auc_cont <- survival::concordance(as.numeric(lsm) ~ liverrisk, data = vcte0) 


lr_confusion
```

```{=html}
</details>
```
::: {.cell-output .cell-output-stdout}
         sens      spec       ppv       npv 
    0.4032572 0.8835720 0.5455442 0.8103222 
:::

`<details class="code-fold">
<summary>`{=html}Code`</summary>`{=html}

``` {.r .cell-code}
c(R2 = 1 - sum((vcte0$lsm - vcte0$liverrisk)^2)/sum((vcte0$lsm - mean(vcte0$lsm))^2), MAPE = mean(abs(vcte0$liverrisk - vcte0$lsm)), cal_intercept = coef(lr_cal_fit)[1], 
cal_slope = coef(lr_cal_fit)[2], 
"AUC (continuous)" = lr_auc_cont$concordance,
"AUC (>8kPa)" = lr_auc$concordance, 
"AUC (>12kPa)" = lr_auc12$concordance) 
```

```{=html}
</details>
```
::: {.cell-output .cell-output-stdout}
                  R2             MAPE    cal_intercept        cal_slope 
           0.1004391        3.0150794        1.1803612        0.9611926 
    AUC (continuous)      AUC (>8kPa)     AUC (>12kPa) 
           0.6553304        0.7527007        0.7996785 
:::
:::
:::

