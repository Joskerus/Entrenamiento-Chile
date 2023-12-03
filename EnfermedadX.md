---
title: "Taller Día 4 - Grupo 1 Estimación de las distribuciones de rezagos epidemiológicos: Enfermedad X"
author: "Kelly Charniga, PhD MPH & Zulma Cuucnubá MD, PhD"
date: "2023-11-23"
output: html_document
teaching: 90
exercises: 8
---



:::::::::::::::::::::::::::::::::::::: questions 
 
- ¿Cómo responder ante un brote de una enfermedad desconocida?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

Al final de este taller usted podrá: 
 
- Comprender los conceptos clave de las distribuciones de rezagos epidemiológicos para la Enfermedad X. 
 
- Comprender las estructuras de datos y las herramientas para el análisis de datos de rastreo de contactos. 
 
- Aprender a ajustar las estimaciones del intervalo serial y el período de incubación de la Enfermedad X teniendo en cuenta la censura por intervalo utilizando un marco de trabajo Bayesiano.
 
- Aprender a utilizar estos parámetros para informar estrategias de control en un brote de un patógeno desconocido. 
::::::::::::::::::::::::::::::::::::::::::::::::



## 1. Introducción

La Enfermedad X representa un hipotético, pero plausible, brote de una enfermedad infecciosa en el futuro. Este término fue acuñado por la Organización Mundial de la Salud (OMS) y sirve como un término general para un patógeno desconocido que podría causar una epidemia grave a nivel internacional. Este concepto representa la naturaleza impredecible de la aparición de enfermedades infecciosas y resalta la necesidad de una preparación global y mecanismos de respuesta rápida. La Enfermedad X simboliza el potencial de una enfermedad inesperada y de rápida propagación, y resalta la necesidad de sistemas de salud flexibles y adaptables, así como capacidades de investigación para identificar, comprender y combatir patógenos desconocidos.

En esta práctica, va a aprender a estimar los rezagos epidemiológicos, el tiempo entre dos eventos epidemiológicos, utilizando un conjunto de datos simulado de la Enfermedad X.

La Enfermedad X es causada por un patógeno desconocido y se transmite directamente de persona a persona. Específicamente, la practica se centrará en estimar el período de incubación y el intervalo serial.




## 2. Agenda

Parte 1. Individual o en grupo.

Parte 2. En 5 grupos de 8 personas. Construir estrategia de rastreo de contactos  y aislamiento y preparar presentación de máximo 10 mins.



## 3. Conceptos claves

#### **3.1. Rezagos epidemiológicos: Período de incubación e intervalo serial**

En epidemiología, las distribuciones de rezagos se refieren a los *retrasos temporales* entre dos eventos clave durante un brote. Por ejemplo: el tiempo entre el inicio de los síntomas y el diagnóstico, el tiempo entre la aparición de síntomas y la muerte, entre muchos otros.

Este taller se enfocará en dos rezagos clave conocidos como el período de incubación y el intervalo serial. Ambos son cruciales para informar la respuesta de salud pública.

El [**período de incubación**]{.underline} es el tiempo entre la infección y la aparición de síntomas.

El [**intervalo serial**]{.underline} es el tiempo entre la aparición de síntomas entre el caso primario y secundario.

La relación entre estos parámetros tiene un impacto en si la enfermedad se transmite antes [(**transmisión pre-sintomática**)]{.underline} o después de que los síntomas [(**transmisión sintomática**)]{.underline} se hayan desarrollado en el caso primario (Figura 1). 

![](practicalfig.jpg)

Figura 1. Relación entre el período de incubación y el intervalo serial en el momento de la transmisión (*Adaptado de Nishiura et al. 2020)*

#### 3.2. Distribuciones comunes de rezagos y posibles sesgos

##### **3.2.1 Sesgos potenciales**

Cuando se estiman rezagos epidemiológicos, es importante considerar posibles sesgos:


[**Censura**]{.underline} significa que sabemos que un evento ocurrió, pero no sabemos exactamente cuándo sucedió. La mayoría de los datos epidemiológicos están "doblemente censurados" debido a la incertidumbre que rodea tanto los tiempos de eventos primarios como secundarios. No tener en cuenta la censura puede llevar a estimaciones sesgadas de la desviación estándar del resago (Park et al. en progreso).


[**Truncamiento a la derecha**]{.underline} es un tipo de sesgo de muestreo relacionado con el proceso de recolección de datos. Surge porque solo se pueden observar los casos que han sido reportados. No tener en cuenta el truncamiento a la derecha durante la fase de crecimiento de una epidemia puede llevar a una subestimación del rezago medio (Park et al. en progreso).


El sesgo [**dinámico (o de fase epidémica**)]{.underline} es otro tipo de sesgo de muestreo. Afecta a los datos retrospectivos y está relacionado con la fase de la epidemia: durante la fase de crecimiento exponencial, los casos que desarrollaron síntomas recientemente están sobrerrepresentados en los datos observados, mientras que durante la fase de declive, estos casos están subrepresentados, lo que lleva a la estimación de intervalos de retraso más cortos y más largos, respectivamente (Park et al. en progreso).

##### **3.2.2 Distribuciones de rezagos**

Tres distribuciones de probabilidad comunes utilizadas para caracterizar rezagos en epidemiología de enfermedades infecciosas (Tabla 1):

+----------------+-----------------------------------------+
| Distribución   | Parámetros                              |
+================+:=======================================:+
| **Weibull**    | `shape` y `scale`                       |
+----------------+-----------------------------------------+
| **gamma**      | `shape` y `scale`                       |
+----------------+-----------------------------------------+
| **log normal** | `log mean` y `log standard deviation`   |
+----------------+-----------------------------------------+

: Tabla 1. Tres de las distribuciones de probabilidad más comunes para rezagos epidemiológicos.


## 4. Paquetes de *R* para la practica

En esta practica se usarán los siguientes paquetes de `R`:

-   `dplyr` para manejo de datos 

-   `epicontacts` para visualizar los datos de rastreo de contactos 

-   `ggplot2` y `patchwork` para gráficar

-   `incidence` para visualizar curvas epidemicas

-   `rstan` para estimar el período de incubación

-   `coarseDataTools` via `EpiEstim` para estimar el intervalo serial

Instrucciones de instalación para los paquetes: 



Para cargar los paquetes, escriba:


```r
library(dplyr)
library(epicontacts)
library(incidence)
library(coarseDataTools)
library(EpiEstim)
library(ggplot2)
library(loo)
library(patchwork)
library(rstan)
```

## 5. Datos

Esta práctica está partida en dos grupos para abordar dos enfermedades desconocidas con diferentes modos de transmisión.

Cargue los datos simulados, que están guardados como un archivo .RDS.

Hay dos elementos de interés:

-   `linelist`, un archivo que contiene una lista de casos de la Enfermedad X, un caso por fila. 

-   `contacts`,  un archivo con datos de rastreo de contactos que contiene información sobre pares de casos primarios y secundarios.



## 6. Exploración de los datos

#### **6.1. Exploración de los datos en `linelist`**

Empiece con `linelist`. Estos datos fueron recolectado como parte de la vigilancia epidemiológica rutinaria. Cada fila representa un caso de la Enfermedad X, y hay 7 variables:

-   `id`: número único de identificación del caso

-   `date_onset`: fecha de inicio de los síntomas del paciente

-   `sex`: : M = masculino; F = femenino

-   `age`: edad del paciente en años

-   `exposure`: información sobre cómo el paciente podría haber estado expuesto

-   `exposure_start`: primera fecha en que el paciente estuvo expuesto

-   `exposure_end`: última fecha en que el paciente estuvo expuesto

::: {.alert .alert-secondary}
                                
 *💡 **Preguntas (1)***                                          
                                                                 
 -   *¿Cuántos casos hay en los datos de `linelist`?*             
                                                                 
 -   *¿Qué proporción de los casos son femeninos?*                  
                                                                 
 -   *¿Cuál es la distribución de edades de los casos?*                    
                                                                 
 -   *¿Qué tipo de información sobre la exposición está disponible?* 

:::


```r
# Inspecionar los datos
head(linelist)
```

```{.output}
  id date_onset sex age                    exposure exposure_start exposure_end
1  1 2023-10-01   M  34 Close, skin-to-skin contact           <NA>         <NA>
2  2 2023-10-03   F  38 Close, skin-to-skin contact     2023-09-29   2023-09-29
3  3 2023-10-06   F  38 Close, skin-to-skin contact     2023-09-28   2023-09-28
4  4 2023-10-10   F  37                        <NA>     2023-09-25   2023-09-27
5  5 2023-10-11   F  33                        <NA>     2023-10-05   2023-10-05
6  6 2023-10-12   F  34 Close, skin-to-skin contact     2023-10-10   2023-10-10
```

```r
# P1
nrow(linelist)
```

```{.output}
[1] 166
```

```r
# P2
table(linelist$sex)[2]/nrow(linelist)
```

```{.output}
        M 
0.6144578 
```

```r
# P3
summary(linelist$age)
```

```{.output}
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  22.00   33.00   36.00   35.51   38.00   47.00 
```

```r
# P4
table(linelist$exposure, exclude = F)[1]/nrow(linelist)
```

```{.output}
Close, skin-to-skin contact 
                  0.6626506 
```

::: {.alert .alert-secondary}
   
 *💡 **Discusión***    
                     
- ¿Por qué cree que falta la información de exposición en algunos casos?       
                     
- Ahora, grafique la curva epidémica. ¿En qué parte del brote cree que está (principio, medio, final)? 

:::


```r
i <- incidence(linelist$date_onset)
plot(i) + 
  theme_classic() + 
  scale_fill_manual(values = "purple") +
  theme(legend.position = "none")
```

<img src="fig/EnfermedadX-rendered-epi curve-1.png" style="display: block; margin: auto;" />

Parece que la epidemía todavía podría esta creciendo.

#### **6.2.  Exploración de los datos de `rastreo de contactos`**

Ahora vea los datos de rastreo de contactos, que se obtuvieron a través de entrevistas a los pacientes. A los pacientes se les preguntó sobre actividades e interacciones recientes para identificar posibles fuentes de infección. Se emparejaron pares de casos primarios y secundarios si el caso secundario nombraba al caso primario como un contacto. Solo hay información de un subconjunto de los casos porque no todos los pacientes pudieron ser contactados para una entrevista.

Note que para este ejercicio, se asumirá que los casos secundarios solo tenían un posible infectante. En realidad, la posibilidad de que un caso tenga múltiples posibles infectantes necesita ser evaluada.

Los datos de rastreo de contactos tienen 4 variables:

-   `primary_case_id`: número de identificación único para el caso primario (infectante)

-   `secondary_case_id`: número de identificación único para el caso secundario (infectado)

-   `primary_onset_date`: fecha de inicio de síntomas del caso primario

-   `secondary_onset_date`: fecha de inicio de síntomas del caso secundario


```r
x <- make_epicontacts(linelist = linelist,
                       contacts = contacts,
                       from = "primary_case_id",
                       to = "secondary_case_id",
                       directed = TRUE) # Esto indica que los contactos son directos (i.e., este gráfico traza una flecha desde los casos primarios a los secundarios)

plot(x)
```

<!--html_preserve--><div id="htmlwidget-be318a6d67ce3cae5f0a" style="width:90%;height:700px;" class="visNetwork html-widget"></div>
<script type="application/json" data-for="htmlwidget-be318a6d67ce3cae5f0a">{"x":{"nodes":{"id":[1,2,4,5,6,8,9,10,11,12,15,16,19,21,22,23,25,28,29,30,31,32,33,34,36,37,38,41,42,43,44,45,46,47,51,52,53,54,55,57,59,60,61,63,64,65,66,68,69,70,71,73,74,78,79,80,81,83,86,88,89,90,92,96,98,99,100,102,103,105,106,107,108,111,113,114,115,117,119,120,121,122,125,127,128,129,133,136,138,141,142,144,149,152,153,154,156,158,159,160,161,162,164,165,166],"date_onset":["2023-10-01","2023-10-03","2023-10-10","2023-10-11","2023-10-12","2023-10-13","2023-10-15","2023-10-16","2023-10-16","2023-10-18","2023-10-20","2023-10-21","2023-10-24","2023-10-24","2023-10-25","2023-10-26","2023-10-28","2023-10-30","2023-10-30","2023-10-30","2023-10-31","2023-11-03","2023-11-03","2023-11-03","2023-11-04","2023-11-04","2023-11-05","2023-11-06","2023-11-06","2023-11-06","2023-11-06","2023-11-07","2023-11-07","2023-11-07","2023-11-08","2023-11-09","2023-11-10","2023-11-10","2023-11-10","2023-11-11","2023-11-12","2023-11-12","2023-11-12","2023-11-12","2023-11-13","2023-11-13","2023-11-13","2023-11-14","2023-11-14","2023-11-14","2023-11-14","2023-11-15","2023-11-15","2023-11-16","2023-11-16","2023-11-16","2023-11-16","2023-11-17","2023-11-17","2023-11-18","2023-11-18","2023-11-18","2023-11-18","2023-11-20","2023-11-20","2023-11-20","2023-11-20","2023-11-20","2023-11-21","2023-11-21","2023-11-21","2023-11-21","2023-11-22","2023-11-22","2023-11-23","2023-11-23","2023-11-23","2023-11-24","2023-11-25","2023-11-25","2023-11-25","2023-11-25","2023-11-26","2023-11-26","2023-11-26","2023-11-26","2023-11-26","2023-11-27","2023-11-27","2023-11-27","2023-11-27","2023-11-28","2023-11-29","2023-11-29","2023-11-29","2023-11-29","2023-11-29","2023-11-30","2023-11-30","2023-11-30","2023-11-30","2023-11-30","2023-11-30","2023-11-30","2023-11-30"],"sex":["M","F","F","F","F","M","F","M","F","M","M","F","M","M","M","M","M","F","F","M","M","M","M","M","F","M","M","F","F","M","F","M","M","F","M","M","F","M","M","M","M","F","F","M","M","M","F","M","M","F","M","M","F","M","M","F","F","M","F","M","M","F","F","M","M","M","F","F","M","M","M","M","M","F","F","M","M","F","M","F","F","F","M","M","M","M","M","M","F","M","F","M","F","M","M","F","F","F","M","F","M","F","M","M","F"],"age":[34,38,37,33,34,35,36,42,39,33,39,39,33,33,32,36,36,34,32,31,38,35,38,30,38,35,34,38,42,35,32,29,32,36,38,28,34,34,27,33,41,38,35,37,42,36,37,33,35,42,31,40,34,32,34,33,34,37,34,42,35,34,41,36,24,35,39,39,43,33,38,34,33,37,37,37,31,27,38,36,38,36,39,31,37,33,37,34,39,40,36,47,34,36,31,22,35,35,37,31,38,38,40,37,36],"exposure":["Close, skin-to-skin contact","Close, skin-to-skin contact",null,null,"Close, skin-to-skin contact",null,null,null,"Close, skin-to-skin contact",null,null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact",null,null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact",null,null,null,"Close, skin-to-skin contact","Close, skin-to-skin contact",null,null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact",null,null,null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact",null,"Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact","Close, skin-to-skin contact",null,"Close, skin-to-skin contact","Close, skin-to-skin contact",null,null,null,"Close, skin-to-skin contact"],"exposure_start":[null,"2023-09-29","2023-09-25","2023-10-05","2023-10-10","2023-10-08","2023-10-13","2023-10-04","2023-10-03","2023-10-11","2023-10-13","2023-10-19","2023-10-20","2023-10-05","2023-10-20","2023-10-14","2023-10-26","2023-10-27","2023-10-24","2023-10-17","2023-10-30","2023-11-02","2023-10-27","2023-10-30","2023-10-29","2023-10-25","2023-11-01","2023-11-03","2023-11-03","2023-11-01","2023-11-03","2023-11-05","2023-10-18","2023-10-30","2023-11-02",null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null],"exposure_end":[null,"2023-09-29","2023-09-27","2023-10-05","2023-10-10","2023-10-08","2023-10-13","2023-10-04","2023-10-03","2023-10-11","2023-10-14","2023-10-20","2023-10-20","2023-10-05","2023-10-20","2023-10-15","2023-10-26","2023-10-27","2023-10-25","2023-10-17","2023-10-30","2023-11-02","2023-10-27","2023-10-30","2023-10-30","2023-10-25","2023-11-01","2023-11-03","2023-11-03","2023-11-03","2023-11-04","2023-11-05","2023-10-19","2023-10-30","2023-11-04",null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null,null],"label":["1","2","4","5","6","8","9","10","11","12","15","16","19","21","22","23","25","28","29","30","31","32","33","34","36","37","38","41","42","43","44","45","46","47","51","52","53","54","55","57","59","60","61","63","64","65","66","68","69","70","71","73","74","78","79","80","81","83","86","88","89","90","92","96","98","99","100","102","103","105","106","107","108","111","113","114","115","117","119","120","121","122","125","127","128","129","133","136","138","141","142","144","149","152","153","154","156","158","159","160","161","162","164","165","166"],"title":["<p> id: 1<br>date_onset: 2023-10-01<br>sex: M<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 2<br>date_onset: 2023-10-03<br>sex: F<br>age: 38<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-09-29<br>exposure_end: 2023-09-29 <\/p>","<p> id: 4<br>date_onset: 2023-10-10<br>sex: F<br>age: 37<br>exposure: NA<br>exposure_start: 2023-09-25<br>exposure_end: 2023-09-27 <\/p>","<p> id: 5<br>date_onset: 2023-10-11<br>sex: F<br>age: 33<br>exposure: NA<br>exposure_start: 2023-10-05<br>exposure_end: 2023-10-05 <\/p>","<p> id: 6<br>date_onset: 2023-10-12<br>sex: F<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-10<br>exposure_end: 2023-10-10 <\/p>","<p> id: 8<br>date_onset: 2023-10-13<br>sex: M<br>age: 35<br>exposure: NA<br>exposure_start: 2023-10-08<br>exposure_end: 2023-10-08 <\/p>","<p> id: 9<br>date_onset: 2023-10-15<br>sex: F<br>age: 36<br>exposure: NA<br>exposure_start: 2023-10-13<br>exposure_end: 2023-10-13 <\/p>","<p> id: 10<br>date_onset: 2023-10-16<br>sex: M<br>age: 42<br>exposure: NA<br>exposure_start: 2023-10-04<br>exposure_end: 2023-10-04 <\/p>","<p> id: 11<br>date_onset: 2023-10-16<br>sex: F<br>age: 39<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-03<br>exposure_end: 2023-10-03 <\/p>","<p> id: 12<br>date_onset: 2023-10-18<br>sex: M<br>age: 33<br>exposure: NA<br>exposure_start: 2023-10-11<br>exposure_end: 2023-10-11 <\/p>","<p> id: 15<br>date_onset: 2023-10-20<br>sex: M<br>age: 39<br>exposure: NA<br>exposure_start: 2023-10-13<br>exposure_end: 2023-10-14 <\/p>","<p> id: 16<br>date_onset: 2023-10-21<br>sex: F<br>age: 39<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-19<br>exposure_end: 2023-10-20 <\/p>","<p> id: 19<br>date_onset: 2023-10-24<br>sex: M<br>age: 33<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-20<br>exposure_end: 2023-10-20 <\/p>","<p> id: 21<br>date_onset: 2023-10-24<br>sex: M<br>age: 33<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-05<br>exposure_end: 2023-10-05 <\/p>","<p> id: 22<br>date_onset: 2023-10-25<br>sex: M<br>age: 32<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-20<br>exposure_end: 2023-10-20 <\/p>","<p> id: 23<br>date_onset: 2023-10-26<br>sex: M<br>age: 36<br>exposure: NA<br>exposure_start: 2023-10-14<br>exposure_end: 2023-10-15 <\/p>","<p> id: 25<br>date_onset: 2023-10-28<br>sex: M<br>age: 36<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-26<br>exposure_end: 2023-10-26 <\/p>","<p> id: 28<br>date_onset: 2023-10-30<br>sex: F<br>age: 34<br>exposure: NA<br>exposure_start: 2023-10-27<br>exposure_end: 2023-10-27 <\/p>","<p> id: 29<br>date_onset: 2023-10-30<br>sex: F<br>age: 32<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-24<br>exposure_end: 2023-10-25 <\/p>","<p> id: 30<br>date_onset: 2023-10-30<br>sex: M<br>age: 31<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-17<br>exposure_end: 2023-10-17 <\/p>","<p> id: 31<br>date_onset: 2023-10-31<br>sex: M<br>age: 38<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-30<br>exposure_end: 2023-10-30 <\/p>","<p> id: 32<br>date_onset: 2023-11-03<br>sex: M<br>age: 35<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-11-02<br>exposure_end: 2023-11-02 <\/p>","<p> id: 33<br>date_onset: 2023-11-03<br>sex: M<br>age: 38<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-27<br>exposure_end: 2023-10-27 <\/p>","<p> id: 34<br>date_onset: 2023-11-03<br>sex: M<br>age: 30<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-30<br>exposure_end: 2023-10-30 <\/p>","<p> id: 36<br>date_onset: 2023-11-04<br>sex: F<br>age: 38<br>exposure: NA<br>exposure_start: 2023-10-29<br>exposure_end: 2023-10-30 <\/p>","<p> id: 37<br>date_onset: 2023-11-04<br>sex: M<br>age: 35<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-25<br>exposure_end: 2023-10-25 <\/p>","<p> id: 38<br>date_onset: 2023-11-05<br>sex: M<br>age: 34<br>exposure: NA<br>exposure_start: 2023-11-01<br>exposure_end: 2023-11-01 <\/p>","<p> id: 41<br>date_onset: 2023-11-06<br>sex: F<br>age: 38<br>exposure: NA<br>exposure_start: 2023-11-03<br>exposure_end: 2023-11-03 <\/p>","<p> id: 42<br>date_onset: 2023-11-06<br>sex: F<br>age: 42<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-11-03<br>exposure_end: 2023-11-03 <\/p>","<p> id: 43<br>date_onset: 2023-11-06<br>sex: M<br>age: 35<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-11-01<br>exposure_end: 2023-11-03 <\/p>","<p> id: 44<br>date_onset: 2023-11-06<br>sex: F<br>age: 32<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-11-03<br>exposure_end: 2023-11-04 <\/p>","<p> id: 45<br>date_onset: 2023-11-07<br>sex: M<br>age: 29<br>exposure: NA<br>exposure_start: 2023-11-05<br>exposure_end: 2023-11-05 <\/p>","<p> id: 46<br>date_onset: 2023-11-07<br>sex: M<br>age: 32<br>exposure: Close, skin-to-skin contact<br>exposure_start: 2023-10-18<br>exposure_end: 2023-10-19 <\/p>","<p> id: 47<br>date_onset: 2023-11-07<br>sex: F<br>age: 36<br>exposure: NA<br>exposure_start: 2023-10-30<br>exposure_end: 2023-10-30 <\/p>","<p> id: 51<br>date_onset: 2023-11-08<br>sex: M<br>age: 38<br>exposure: NA<br>exposure_start: 2023-11-02<br>exposure_end: 2023-11-04 <\/p>","<p> id: 52<br>date_onset: 2023-11-09<br>sex: M<br>age: 28<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 53<br>date_onset: 2023-11-10<br>sex: F<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 54<br>date_onset: 2023-11-10<br>sex: M<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 55<br>date_onset: 2023-11-10<br>sex: M<br>age: 27<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 57<br>date_onset: 2023-11-11<br>sex: M<br>age: 33<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 59<br>date_onset: 2023-11-12<br>sex: M<br>age: 41<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 60<br>date_onset: 2023-11-12<br>sex: F<br>age: 38<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 61<br>date_onset: 2023-11-12<br>sex: F<br>age: 35<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 63<br>date_onset: 2023-11-12<br>sex: M<br>age: 37<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 64<br>date_onset: 2023-11-13<br>sex: M<br>age: 42<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 65<br>date_onset: 2023-11-13<br>sex: M<br>age: 36<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 66<br>date_onset: 2023-11-13<br>sex: F<br>age: 37<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 68<br>date_onset: 2023-11-14<br>sex: M<br>age: 33<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 69<br>date_onset: 2023-11-14<br>sex: M<br>age: 35<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 70<br>date_onset: 2023-11-14<br>sex: F<br>age: 42<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 71<br>date_onset: 2023-11-14<br>sex: M<br>age: 31<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 73<br>date_onset: 2023-11-15<br>sex: M<br>age: 40<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 74<br>date_onset: 2023-11-15<br>sex: F<br>age: 34<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 78<br>date_onset: 2023-11-16<br>sex: M<br>age: 32<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 79<br>date_onset: 2023-11-16<br>sex: M<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 80<br>date_onset: 2023-11-16<br>sex: F<br>age: 33<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 81<br>date_onset: 2023-11-16<br>sex: F<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 83<br>date_onset: 2023-11-17<br>sex: M<br>age: 37<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 86<br>date_onset: 2023-11-17<br>sex: F<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 88<br>date_onset: 2023-11-18<br>sex: M<br>age: 42<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 89<br>date_onset: 2023-11-18<br>sex: M<br>age: 35<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 90<br>date_onset: 2023-11-18<br>sex: F<br>age: 34<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 92<br>date_onset: 2023-11-18<br>sex: F<br>age: 41<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 96<br>date_onset: 2023-11-20<br>sex: M<br>age: 36<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 98<br>date_onset: 2023-11-20<br>sex: M<br>age: 24<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 99<br>date_onset: 2023-11-20<br>sex: M<br>age: 35<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 100<br>date_onset: 2023-11-20<br>sex: F<br>age: 39<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 102<br>date_onset: 2023-11-20<br>sex: F<br>age: 39<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 103<br>date_onset: 2023-11-21<br>sex: M<br>age: 43<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 105<br>date_onset: 2023-11-21<br>sex: M<br>age: 33<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 106<br>date_onset: 2023-11-21<br>sex: M<br>age: 38<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 107<br>date_onset: 2023-11-21<br>sex: M<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 108<br>date_onset: 2023-11-22<br>sex: M<br>age: 33<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 111<br>date_onset: 2023-11-22<br>sex: F<br>age: 37<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 113<br>date_onset: 2023-11-23<br>sex: F<br>age: 37<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 114<br>date_onset: 2023-11-23<br>sex: M<br>age: 37<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 115<br>date_onset: 2023-11-23<br>sex: M<br>age: 31<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 117<br>date_onset: 2023-11-24<br>sex: F<br>age: 27<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 119<br>date_onset: 2023-11-25<br>sex: M<br>age: 38<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 120<br>date_onset: 2023-11-25<br>sex: F<br>age: 36<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 121<br>date_onset: 2023-11-25<br>sex: F<br>age: 38<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 122<br>date_onset: 2023-11-25<br>sex: F<br>age: 36<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 125<br>date_onset: 2023-11-26<br>sex: M<br>age: 39<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 127<br>date_onset: 2023-11-26<br>sex: M<br>age: 31<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 128<br>date_onset: 2023-11-26<br>sex: M<br>age: 37<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 129<br>date_onset: 2023-11-26<br>sex: M<br>age: 33<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 133<br>date_onset: 2023-11-26<br>sex: M<br>age: 37<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 136<br>date_onset: 2023-11-27<br>sex: M<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 138<br>date_onset: 2023-11-27<br>sex: F<br>age: 39<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 141<br>date_onset: 2023-11-27<br>sex: M<br>age: 40<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 142<br>date_onset: 2023-11-27<br>sex: F<br>age: 36<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 144<br>date_onset: 2023-11-28<br>sex: M<br>age: 47<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 149<br>date_onset: 2023-11-29<br>sex: F<br>age: 34<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 152<br>date_onset: 2023-11-29<br>sex: M<br>age: 36<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 153<br>date_onset: 2023-11-29<br>sex: M<br>age: 31<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 154<br>date_onset: 2023-11-29<br>sex: F<br>age: 22<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 156<br>date_onset: 2023-11-29<br>sex: F<br>age: 35<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 158<br>date_onset: 2023-11-30<br>sex: F<br>age: 35<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 159<br>date_onset: 2023-11-30<br>sex: M<br>age: 37<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 160<br>date_onset: 2023-11-30<br>sex: F<br>age: 31<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 161<br>date_onset: 2023-11-30<br>sex: M<br>age: 38<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 162<br>date_onset: 2023-11-30<br>sex: F<br>age: 38<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 164<br>date_onset: 2023-11-30<br>sex: M<br>age: 40<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 165<br>date_onset: 2023-11-30<br>sex: M<br>age: 37<br>exposure: NA<br>exposure_start: NA<br>exposure_end: NA <\/p>","<p> id: 166<br>date_onset: 2023-11-30<br>sex: F<br>age: 36<br>exposure: Close, skin-to-skin contact<br>exposure_start: NA<br>exposure_end: NA <\/p>"],"color.highlight.background":["#CCDDFF","#C4DCF7","#BDDBEF","#B6DAE7","#AFD9E0","#A8D8D8","#A0D7D0","#99D6C9","#92D5C1","#8BD4B9","#84D3B1","#7CD2AA","#7ED0A6","#89CEA7","#95CBA8","#A0C8A9","#ACC6AB","#B8C3AC","#C3C0AD","#CFBEAE","#DABBAF","#E6B8B0","#F2B5B1","#FDB3B2","#F7B1B4","#F0B0B5","#E8AFB6","#E0ADB7","#D8ACB8","#D0ABBA","#C8AABB","#C0A8BC","#B8A7BD","#B1A6BE","#A9A4C0","#A6A5BB","#AEA8AA","#B6AC9A","#BEAF89","#C6B378","#CEB667","#D5B957","#DDBD46","#E5C035","#EDC425","#F5C714","#FDCB03","#FFC808","#FFC513","#FFC11E","#FFBD29","#FFB934","#FFB540","#FFB14B","#FFAD56","#FFA961","#FFA56C","#FFA277","#FEA080","#F9A982","#F5B184","#F0B986","#ECC289","#E7CA8B","#E3D28D","#DFDB8F","#DAE391","#D6EB93","#D1F396","#CDFC98","#CDF99B","#CEF19E","#D0E8A1","#D2E0A5","#D3D8A8","#D5CFAB","#D6C7AE","#D8BFB2","#DAB6B5","#DBAEB8","#DDA6BC","#DF9FBE","#E2A3BB","#E4A7B8","#E7ABB4","#EAAFB1","#EDB2AE","#EFB6AA","#F2BAA7","#F5BEA4","#F8C2A1","#FBC69D","#FDCA9A","#FCCC9B","#F8CCA0","#F3CCA4","#EFCCA9","#EBCCAD","#E6CCB2","#E2CCB6","#DECCBB","#D9CCBF","#D5CCC4","#D1CCC8","#CDCDCD"],"color.background":["#CCDDFF","#C4DCF7","#BDDBEF","#B6DAE7","#AFD9E0","#A8D8D8","#A0D7D0","#99D6C9","#92D5C1","#8BD4B9","#84D3B1","#7CD2AA","#7ED0A6","#89CEA7","#95CBA8","#A0C8A9","#ACC6AB","#B8C3AC","#C3C0AD","#CFBEAE","#DABBAF","#E6B8B0","#F2B5B1","#FDB3B2","#F7B1B4","#F0B0B5","#E8AFB6","#E0ADB7","#D8ACB8","#D0ABBA","#C8AABB","#C0A8BC","#B8A7BD","#B1A6BE","#A9A4C0","#A6A5BB","#AEA8AA","#B6AC9A","#BEAF89","#C6B378","#CEB667","#D5B957","#DDBD46","#E5C035","#EDC425","#F5C714","#FDCB03","#FFC808","#FFC513","#FFC11E","#FFBD29","#FFB934","#FFB540","#FFB14B","#FFAD56","#FFA961","#FFA56C","#FFA277","#FEA080","#F9A982","#F5B184","#F0B986","#ECC289","#E7CA8B","#E3D28D","#DFDB8F","#DAE391","#D6EB93","#D1F396","#CDFC98","#CDF99B","#CEF19E","#D0E8A1","#D2E0A5","#D3D8A8","#D5CFAB","#D6C7AE","#D8BFB2","#DAB6B5","#DBAEB8","#DDA6BC","#DF9FBE","#E2A3BB","#E4A7B8","#E7ABB4","#EAAFB1","#EDB2AE","#EFB6AA","#F2BAA7","#F5BEA4","#F8C2A1","#FBC69D","#FDCA9A","#FCCC9B","#F8CCA0","#F3CCA4","#EFCCA9","#EBCCAD","#E6CCB2","#E2CCB6","#DECCBB","#D9CCBF","#D5CCC4","#D1CCC8","#CDCDCD"],"color.highlight.border":["black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black"],"color.border":["black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black"],"size":[20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20,20],"borderWidth":[2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2,2]},"edges":{"from":[1,1,5,4,6,12,9,15,2,22,22,25,25,29,16,31,34,25,36,30,38,41,28,44,41,23,45,45,37,31,42,54,47,61,54,46,79,68,80,60,74,71,70,88,83,64,63,34,113,69,90,108,113,80,73,107,70,78,64,105,55,108,113,103,128,156,99,128,89,92,106,103],"to":[2,5,8,10,11,16,19,21,23,25,30,32,33,34,36,38,42,43,46,51,52,53,55,57,59,61,65,66,69,70,78,79,80,81,83,86,90,92,96,98,100,102,107,111,114,115,117,119,120,121,122,125,127,129,133,136,138,141,142,144,149,152,153,154,158,159,160,161,162,164,165,166],"primary_onset_date":["2023-10-01","2023-10-01","2023-10-11","2023-10-10","2023-10-12","2023-10-18","2023-10-15","2023-10-20","2023-10-03","2023-10-25","2023-10-25","2023-10-28","2023-10-28","2023-10-30","2023-10-21","2023-10-31","2023-11-03","2023-10-28","2023-11-04","2023-10-30","2023-11-05","2023-11-06","2023-10-30","2023-11-06","2023-11-06","2023-10-26","2023-11-07","2023-11-07","2023-11-04","2023-10-31","2023-11-06","2023-11-10","2023-11-07","2023-11-12","2023-11-10","2023-11-07","2023-11-16","2023-11-14","2023-11-16","2023-11-12","2023-11-15","2023-11-14","2023-11-14","2023-11-18","2023-11-17","2023-11-13","2023-11-12","2023-11-03","2023-11-23","2023-11-14","2023-11-18","2023-11-22","2023-11-23","2023-11-16","2023-11-15","2023-11-21","2023-11-14","2023-11-16","2023-11-13","2023-11-21","2023-11-10","2023-11-22","2023-11-23","2023-11-21","2023-11-26","2023-11-29","2023-11-20","2023-11-26","2023-11-18","2023-11-18","2023-11-21","2023-11-21"],"secondary_onset_date":["2023-10-03","2023-10-11","2023-10-13","2023-10-16","2023-10-16","2023-10-21","2023-10-24","2023-10-24","2023-10-26","2023-10-28","2023-10-30","2023-11-03","2023-11-03","2023-11-03","2023-11-04","2023-11-05","2023-11-06","2023-11-06","2023-11-07","2023-11-08","2023-11-09","2023-11-10","2023-11-10","2023-11-11","2023-11-12","2023-11-12","2023-11-13","2023-11-13","2023-11-14","2023-11-14","2023-11-16","2023-11-16","2023-11-16","2023-11-16","2023-11-17","2023-11-17","2023-11-18","2023-11-18","2023-11-20","2023-11-20","2023-11-20","2023-11-20","2023-11-21","2023-11-22","2023-11-23","2023-11-23","2023-11-24","2023-11-25","2023-11-25","2023-11-25","2023-11-25","2023-11-26","2023-11-26","2023-11-26","2023-11-26","2023-11-27","2023-11-27","2023-11-27","2023-11-27","2023-11-28","2023-11-29","2023-11-29","2023-11-29","2023-11-29","2023-11-30","2023-11-30","2023-11-30","2023-11-30","2023-11-30","2023-11-30","2023-11-30","2023-11-30"],"width":[3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3],"color":["black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black","black"],"arrows.to":[true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true,true]},"nodesToDataframe":true,"edgesToDataframe":true,"options":{"width":"100%","height":"100%","nodes":{"shape":"dot"},"manipulation":{"enabled":false},"interaction":{"zoomSpeed":1},"edges":{"arrows":{"to":{"scaleFactor":2}}},"physics":{"stabilization":false}},"groups":null,"width":"90%","height":"700px","idselection":{"enabled":false,"style":"width: 150px; height: 26px","useLabels":true,"main":"Select by id"},"byselection":{"enabled":true,"style":"width: 150px; height: 26px","multiple":false,"hideColor":"rgba(200,200,200,0.5)","highlight":false,"variable":"id","main":"Select by id","values":[1,2,4,5,6,8,9,10,11,12,15,16,19,21,22,23,25,28,29,30,31,32,33,34,36,37,38,41,42,43,44,45,46,47,51,52,53,54,55,57,59,60,61,63,64,65,66,68,69,70,71,73,74,78,79,80,81,83,86,88,89,90,92,96,98,99,100,102,103,105,106,107,108,111,113,114,115,117,119,120,121,122,125,127,128,129,133,136,138,141,142,144,149,152,153,154,156,158,159,160,161,162,164,165,166]},"main":null,"submain":null,"footer":null,"background":"rgba(0, 0, 0, 0)","tooltipStay":300,"tooltipStyle":"position: fixed;visibility:hidden;padding: 5px;white-space: nowrap;font-family: verdana;font-size:14px;font-color:#000000;background-color: #EEEEEE;-moz-border-radius: 3px;-webkit-border-radius: 3px;border-radius: 3px;border: 1px solid #000000;","legend":{"width":0.1,"useGroups":false,"position":"left","ncol":1,"stepX":100,"stepY":100,"zoom":false},"opts_manipulation":{"datacss":"table.legend_table {\n  font-size: 11px;\n  border-width:1px;\n  border-color:#d3d3d3;\n  border-style:solid;\n}\ntable.legend_table td {\n  border-width:1px;\n  border-color:#d3d3d3;\n  border-style:solid;\n  padding: 2px;\n}\ndiv.table_content {\n  width:80px;\n  text-align:center;\n}\ndiv.table_description {\n  width:100px;\n}\n\n.operation {\n  font-size:20px;\n}\n\n.network-popUp {\n  display:none;\n  z-index:299;\n  width:250px;\n  /*height:150px;*/\n  background-color: #f9f9f9;\n  border-style:solid;\n  border-width:1px;\n  border-color: #0d0d0d;\n  padding:10px;\n  text-align: center;\n  position:fixed;\n  top:50%;  \n  left:50%;  \n  margin:-100px 0 0 -100px;  \n\n}","addNodeCols":["id","label"],"editNodeCols":["id","label"],"tab_add_node":"<span id=\"addnode-operation\" class = \"operation\">node<\/span> <br><table style=\"margin:auto;\"><tr><td>id<\/td><td><input id=\"addnode-id\"  type= \"text\" value=\"new value\"><\/td><\/tr><tr><td>label<\/td><td><input id=\"addnode-label\"  type= \"text\" value=\"new value\"><\/td><\/tr><\/table><input type=\"button\" value=\"save\" id=\"addnode-saveButton\"><\/button><input type=\"button\" value=\"cancel\" id=\"addnode-cancelButton\"><\/button>","tab_edit_node":"<span id=\"editnode-operation\" class = \"operation\">node<\/span> <br><table style=\"margin:auto;\"><tr><td>id<\/td><td><input id=\"editnode-id\"  type= \"text\" value=\"new value\"><\/td><\/tr><tr><td>label<\/td><td><input id=\"editnode-label\"  type= \"text\" value=\"new value\"><\/td><\/tr><\/table><input type=\"button\" value=\"save\" id=\"editnode-saveButton\"><\/button><input type=\"button\" value=\"cancel\" id=\"editnode-cancelButton\"><\/button>"},"highlight":{"enabled":true,"hoverNearest":false,"degree":1,"algorithm":"all","hideColor":"rgba(200,200,200,1)","labelOnly":true},"collapse":{"enabled":true,"fit":false,"resetHighlight":true,"clusterOptions":null,"keepCoord":true,"labelSuffix":"(cluster)"},"iconsRedraw":true},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

::: {.alert .alert-secondary}
                                                                                
 *💡 **Preguntas (2)***                                                                                         
        
-   Describa los grupos (clusters).                                                                                     
-   ¿Ve algún evento potencial de superpropagación (donde un caso propaga el patógeno a muchos otros casos)? 

:::

# *`_______Pausa 1 _________`*

------------------------------------------------------------------------

## 7. Estimación del período de incubación 

Ahora, enfoquese en el período de incubación. Se utilizará los datos del `linelist` para esta parte. Se necesitan ambos el tiempo de inicio de sintomas y el timpo de la posible exposición. Note que en los datos hay dos fechas de exposición, una de inicio y una de final. Algunas veces la fecha exacta de exposición es desconocida y en su lugar se obtiene la ventana de exposición durante la entrevista.

::: {.alert .alert-secondary}
*💡 **Preguntas (3)***

-   ¿Para cuántos casos tiene datos tanto de la fecha de inicio de síntomas como de exposición?

-   Calcule las ventanas de exposición. ¿Cuántos casos tienen una única fecha de exposición?
:::


```r
ip <- filter(linelist, !is.na(exposure_start) &
               !is.na(exposure_end))
nrow(ip)
```

```{.output}
[1] 50
```

```r
ip$exposure_window <- as.numeric(ip$exposure_end - ip$exposure_start)

table(ip$exposure_window)
```

```{.output}

 0  1  2 
34 11  5 
```

### 7.1. Estimación naive del período de incubación

Empiece calculando una estimación naive del período de incubación.


```r
# Máximo tiempo de período de incubación
ip$max_ip <- ip$date_onset - ip$exposure_start
summary(as.numeric(ip$max_ip))
```

```{.output}
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   1.00    3.00    4.50    6.38    7.75   20.00 
```

```r
# Mínimo tiempo de período de incubación
ip$min_ip <- ip$date_onset - ip$exposure_end
summary(as.numeric(ip$min_ip))
```

```{.output}
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   1.00    2.00    4.00    5.96    7.75   19.00 
```

### 7.2. Censura estimada ajustada del período de incubación

Ahora, ajuste  tres distribuciones de probabilidad a los datos del período de incubación teniendo en cuenta la censura doble. Adapte un código de `stan` que fue publicado por Miura et al. durante el brote global de mpox de 2022. Este método no tiene en cuenta el truncamiento a la derecha ni el sesgo dinámico. 

Recuerde que el interés principal es considerar tres distribuciones de probabilidad: *Weibull*, *gamma* y *log normal* (Ver Tabla 1).

`Stan` es un programa de software que implementa el algoritmo Monte Carlo Hamiltoniano (HMC por su siglas en inglés de Hamiltonian Monte Carlo). HMC es un método de Monte Carlo de cadena de Markov (MCMC) para ajustar modelos complejos a datos utilizando estadísticas bayesianas.


#### **7.1.1. Corra el modelo en Stan**

Ajuste las tres distribuciones en este bloque de código.


```r
# Prepare los datos
earliest_exposure <- as.Date(min(ip$exposure_start))

ip <- ip |>
  mutate(date_onset = as.numeric(date_onset - earliest_exposure),
         exposure_start = as.numeric(exposure_start - earliest_exposure),
         exposure_end = as.numeric(exposure_end - earliest_exposure)) |>
  select(id, date_onset, exposure_start, exposure_end)

# Configure algunas opciones para ejecutar las cadenas MCMC en paralelo
# Ejecución de las cadenas MCMC en paralelo significa que se ejecutaran varias cadenas al mismo tiempo usando varios núcleos de su computador
options(mc.cores=parallel::detectCores())

input_data <- list(N = length(ip$exposure_start), # NNúmero de observaciones
              tStartExposure = ip$exposure_start,
              tEndExposure = ip$exposure_end,
              tSymptomOnset = ip$date_onset)

# tres distribuciones de probabilidad
distributions <- c("weibull", "gamma", "lognormal") 

# Código de Stan 
code <- sprintf("
  data{
    int<lower=1> N;
    vector[N] tStartExposure;
    vector[N] tEndExposure;
    vector[N] tSymptomOnset;
  }
  parameters{
    real<lower=0> par[2];
    vector<lower=0, upper=1>[N] uE;	// Uniform value for sampling between start and end exposure
  }
  transformed parameters{
    vector[N] tE; 	// infection moment
    tE = tStartExposure + uE .* (tEndExposure - tStartExposure);
  }
model{
    // Contribution to likelihood of incubation period
    target += %s_lpdf(tSymptomOnset -  tE  | par[1], par[2]);
  }
  generated quantities {
    // likelihood for calculation of looIC
    vector[N] log_lik;
    for (i in 1:N) {
      log_lik[i] = %s_lpdf(tSymptomOnset[i] -  tE[i]  | par[1], par[2]);
    }
  }
", distributions, distributions)
names(code) <- distributions

# La siguiente línea puede tomar ~1.5 min
models <- mapply(stan_model, model_code = code)

# Toma ~40 sec.
fit <- mapply(sampling, models, list(input_data), 
              iter=3000, # Número de iteraciones (largo de la cadena MCMC)
              warmup=1000, # Número de muestras a descartar al inicio de MCMC
              chain=4) # Número de cadenas MCMC a ejecutar

pos <- mapply(function(z) rstan::extract(z)$par, fit, SIMPLIFY=FALSE) # muestreo posterior 
```

#### **7.1.2. Revisar si hay convergencia**

Ahora verifique la convergencia del modelo. Observe los valores de r-hat, los tamaños de muestra efectivos y las trazas MCMC. R-hat compara las estimaciones entre y dentro de cadenas para los parámetros del modelo; valores cercanos a 1 indican que las cadenas se han mezclado bien (Vehtari et al. 2021). El tamaño de muestra efectivo estima el número de muestras independientes después de tener en cuenta la dependencia en las cadenas MCMC (Lambert 2018). Para un modelo con 4 cadenas MCMC, se recomienda un tamaño total de muestra efectiva de al menos 400 (Vehtari et al. 2021).

Para cada modelo con distribución ajustada:

::: {.alert .alert-secondary}
*💡 **Questions (4)***

-   ¿Los valores de r-hat son cercanos a 1?

-   ¿Las 4 cadenas MCMC generalmente se superponen y permanecen alrededor de los mismos valores (se ven como orugas peludas)?
:::

#### **7.1.2.1. Convergencia para Gamma**


```r
print(fit$gamma, digits = 3, pars = c("par[1]","par[2]")) 
```

```{.output}
Inference for Stan model: anon_model.
4 chains, each with iter=3000; warmup=1000; thin=1; 
post-warmup draws per chain=2000, total post-warmup draws=8000.

        mean se_mean    sd  2.5%   25%   50%   75% 97.5% n_eff Rhat
par[1] 1.983   0.003 0.361 1.342 1.728 1.959 2.211 2.756 11718    1
par[2] 0.324   0.001 0.067 0.206 0.277 0.320 0.367 0.468 11541    1

Samples were drawn using NUTS(diag_e) at Sun Dec  3 22:39:33 2023.
For each parameter, n_eff is a crude measure of effective sample size,
and Rhat is the potential scale reduction factor on split chains (at 
convergence, Rhat=1).
```

```r
rstan::traceplot(fit$gamma, pars = c("par[1]","par[2]"))
```

<img src="fig/EnfermedadX-rendered-convergence gamma-1.png" style="display: block; margin: auto;" />

#### **7.1.2.2. Convergencia para log normal**


```r
print(fit$lognormal, digits = 3, pars = c("par[1]","par[2]")) 
```

```{.output}
Inference for Stan model: anon_model.
4 chains, each with iter=3000; warmup=1000; thin=1; 
post-warmup draws per chain=2000, total post-warmup draws=8000.

        mean se_mean    sd  2.5%   25%   50%   75% 97.5% n_eff Rhat
par[1] 1.529   0.001 0.114 1.305 1.453 1.530 1.606 1.752 10877    1
par[2] 0.801   0.001 0.085 0.657 0.741 0.794 0.854 0.985  9184    1

Samples were drawn using NUTS(diag_e) at Sun Dec  3 22:39:40 2023.
For each parameter, n_eff is a crude measure of effective sample size,
and Rhat is the potential scale reduction factor on split chains (at 
convergence, Rhat=1).
```

```r
rstan::traceplot(fit$lognormal, pars = c("par[1]","par[2]")) 
```

<img src="fig/EnfermedadX-rendered-convergence lognormal-1.png" style="display: block; margin: auto;" />

#### **7.1.2.3. Convergencia para Weibull**


```r
print(fit$weibull, digits = 3, pars = c("par[1]","par[2]")) 
```

```{.output}
Inference for Stan model: anon_model.
4 chains, each with iter=3000; warmup=1000; thin=1; 
post-warmup draws per chain=2000, total post-warmup draws=8000.

        mean se_mean    sd  2.5%   25%   50%   75% 97.5% n_eff Rhat
par[1] 1.372   0.002 0.149 1.093 1.268 1.370 1.470 1.671  8979    1
par[2] 6.974   0.008 0.778 5.564 6.439 6.937 7.474 8.611  8578    1

Samples were drawn using NUTS(diag_e) at Sun Dec  3 22:39:26 2023.
For each parameter, n_eff is a crude measure of effective sample size,
and Rhat is the potential scale reduction factor on split chains (at 
convergence, Rhat=1).
```

```r
rstan::traceplot(fit$weibull, pars = c("par[1]","par[2]")) 
```

<img src="fig/EnfermedadX-rendered-convergence weibull-1.png" style="display: block; margin: auto;" />

#### **7.1.3. Calcule los criterios de comparación de los modelos**

Calcule el criterio de información ampliamente aplicable (WAIC) y el criterio de información de dejar-uno-fuera (LOOIC) para comparar los ajustes de los modelos. El modelo con mejor ajuste es aquel con el WAIC o LOOIC más bajo. En esta sección también se resumirá las distribuciones y se hará algunos gráficos.

::: {.alert .alert-secondary}
*💡 **Questions (5)***

-   ¿Qué modelo tiene mejor ajuste?
:::

#### 


```r
# Calcule WAIC para los tres modelos
waic <- mapply(function(z) waic(extract_log_lik(z))$estimates[3,], fit)
waic
```

```{.output}
          weibull     gamma lognormal
Estimate 278.1658 276.22399 272.82976
SE        11.9069  13.39728  13.87513
```

```r
# Para looic, se necesita proveer los tamaños de muestra relativos
# al llamar a loo. Este paso lleva a mejores estimados de los tamaños de 
# muestra PSIS efectivos y del error de Monte Carlo 

# Extraer la verosimilitud puntual logarítmica para la distribución Weibull
loglik <- extract_log_lik(fit$weibull, merge_chains = FALSE)
# Obtener los tamaños de muestra relativos efectivos
r_eff <- relative_eff(exp(loglik), cores = 2)
# Calcula LOOIC
loo_w <- loo(loglik, r_eff = r_eff, cores = 2)$estimates[3,]
# Imprimir los resultados
loo_w[1]
```

```{.output}
Estimate 
278.1958 
```

```r
# Extraer la verosimilitud puntual logarítmica para la distribución gamma 
loglik <- extract_log_lik(fit$gamma, merge_chains = FALSE)
r_eff <- relative_eff(exp(loglik), cores = 2)
loo_g <- loo(loglik, r_eff = r_eff, cores = 2)$estimates[3,]
loo_g[1]
```

```{.output}
Estimate 
276.2397 
```

```r
# Extraer la verosimilitud puntual logarítmica para la distribución log normal 
loglik <- extract_log_lik(fit$lognormal, merge_chains = FALSE)
r_eff <- relative_eff(exp(loglik), cores = 2)
loo_l <- loo(loglik, r_eff = r_eff, cores = 2)$estimates[3,]
loo_l[1]
```

```{.output}
Estimate 
272.8405 
```

#### **7.1.4. Reporte los resultados**

La cola derecha de la distribución del período de incubación es importante para diseñar estrategias de control (por ejemplo, cuarentena), los percentiles del 25 al 75 informan sobre el momento más probable en que podría ocurrir la aparición de síntomas, y la distribución completa puede usarse como una entrada en modelos matemáticos o estadísticos, como para pronósticos (Lessler et al. 2009).

Obtenga las estadísticas resumidas.


```r
# We need to convert the parameters of the distributions to the mean and standard deviation delay

# In Stan, the parameters of the distributions are:
# Weibull: shape and scale
# Gamma: shape and inverse scale (aka rate)
# Log normal: mu and sigma
# Reference: https://mc-stan.org/docs/2_21/functions-reference/positive-continuous-distributions.html
means <- cbind(pos$weibull[,2]*gamma(1+1/pos$weibull[,1]), # mean of Weibull
               pos$gamma[,1] / pos$gamma[,2], # mean of gamma
               exp(pos$lognormal[,1]+pos$lognormal[,2]^2/2)) # mean of log normal

standard_deviations <- cbind(sqrt(pos$weibull[,2]^2*(gamma(1+2/pos$weibull[,1])-(gamma(1+1/pos$weibull[,1]))^2)),
                             sqrt(pos$gamma[,1]/(pos$gamma[,2]^2)),
                             sqrt((exp(pos$lognormal[,2]^2)-1) * (exp(2*pos$lognormal[,1] + pos$lognormal[,2]^2))))

# Print the mean delays and 95% credible intervals
probs <- c(0.025, 0.5, 0.975)

res_means <- apply(means, 2, quantile, probs)
colnames(res_means) <- colnames(waic)
res_means
```

```{.output}
       weibull    gamma lognormal
2.5%  5.176877 5.030843  5.016599
50%   6.357164 6.128345  6.339173
97.5% 7.905445 7.561368  8.450425
```

```r
res_sds <- apply(standard_deviations, 2, quantile, probs)
colnames(res_sds) <- colnames(waic)
res_sds
```

```{.output}
       weibull    gamma lognormal
2.5%  3.703054 3.417395  3.982472
50%   4.674519 4.368469  5.913868
97.5% 6.449455 5.873452 10.123743
```

```r
# Report the median and 95% credible intervals for the quantiles of each distribution
quantiles_to_report <- c(0.025, 0.05, 0.5, 0.95, 0.975, 0.99)

# Weibull
cens_w_percentiles <- sapply(quantiles_to_report, function(p) quantile(qweibull(p = p, shape = pos$weibull[,1], scale = pos$weibull[,2]), probs = probs))
colnames(cens_w_percentiles) <- quantiles_to_report
print(cens_w_percentiles)
```

```{.output}
          0.025      0.05      0.5     0.95    0.975     0.99
2.5%  0.2187522 0.4154240 4.102373 12.57757 14.48539 16.77056
50%   0.4738430 0.7919183 5.308907 15.43557 17.97290 21.12660
97.5% 0.8406648 1.2886249 6.655287 20.29349 24.21347 29.33028
```

```r
# Gamma
cens_g_percentiles <- sapply(quantiles_to_report, function(p) quantile(qgamma(p = p, shape = pos$gamma[,1], rate = pos$gamma[,2]), probs = probs))
colnames(cens_g_percentiles) <- quantiles_to_report
print(cens_g_percentiles)
```

```{.output}
          0.025      0.05      0.5     0.95    0.975     0.99
2.5%  0.3342093 0.5664656 4.106965 11.83008 13.78960 16.26057
50%   0.7196451 1.0626606 5.122019 14.61505 17.18492 20.49634
97.5% 1.1900510 1.6208902 6.309097 18.90259 22.53968 27.26054
```

```r
# Log normal
cens_ln_percentiles <- sapply(quantiles_to_report, function(p) quantile(qlnorm(p = p, meanlog = pos$lognormal[,1], sdlog= pos$lognormal[,2]), probs = probs))
colnames(cens_ln_percentiles) <- quantiles_to_report
print(cens_ln_percentiles)
```

```{.output}
          0.025      0.05      0.5     0.95    0.975     0.99
2.5%  0.6274741 0.8459504 3.688554 12.56880 15.60993 20.02950
50%   0.9751362 1.2521179 4.619095 17.01012 21.83923 29.18417
97.5% 1.3628378 1.6934989 5.765587 25.16803 33.98634 48.53571
```

Para cada modelo, encuentre estos elementos para el período de incubación estimado en la salida de arriba y escribalos abajo.

-   Media e intervalo de credibilidad del 95%

-   Desviación estándar e intervalo de credibilidad del 95%

-   Percentiles (e.g., 2.5, 5, 25, 50, 75, 95, 97.5, 99)

-   Los parámetros de las distribuciones ajustadas (e.g., shape y scale para distribución gamma)

#### **7.1.5. Grafique los resultados**


```r
# Prepare los resultados para graficarlos
df <- data.frame(
#Tome los valores de las medias para trazar la función de densidad acumulatica empirica
  inc_day = ((input_data$tSymptomOnset-input_data$tEndExposure)+(input_data$tSymptomOnset-input_data$tStartExposure))/2
)

x_plot <- seq(0, 30, by=0.1) # Esto configura el rango del eje x (número de días)

Gam_plot <- as.data.frame(list(dose= x_plot, 
                               pred= sapply(x_plot, function(q) quantile(pgamma(q = q, shape = pos$gamma[,1], rate = pos$gamma[,2]), probs = c(0.5))),
                               low = sapply(x_plot, function(q) quantile(pgamma(q = q, shape = pos$gamma[,1], rate = pos$gamma[,2]), probs = c(0.025))),
                               upp = sapply(x_plot, function(q) quantile(pgamma(q = q, shape = pos$gamma[,1], rate = pos$gamma[,2]), probs = c(0.975)))
))

Wei_plot <- as.data.frame(list(dose= x_plot, 
                               pred= sapply(x_plot, function(q) quantile(pweibull(q = q, shape = pos$weibull[,1], scale = pos$weibull[,2]), probs = c(0.5))),
                               low = sapply(x_plot, function(q) quantile(pweibull(q = q, shape = pos$weibull[,1], scale = pos$weibull[,2]), probs = c(0.025))),
                               upp = sapply(x_plot, function(q) quantile(pweibull(q = q, shape = pos$weibull[,1], scale = pos$weibull[,2]), probs = c(0.975)))
))

ln_plot <- as.data.frame(list(dose= x_plot, 
                              pred= sapply(x_plot, function(q) quantile(plnorm(q = q, meanlog = pos$lognormal[,1], sdlog= pos$lognormal[,2]), probs = c(0.5))),
                              low = sapply(x_plot, function(q) quantile(plnorm(q = q, meanlog = pos$lognormal[,1], sdlog= pos$lognormal[,2]), probs = c(0.025))),
                              upp = sapply(x_plot, function(q) quantile(plnorm(q = q, meanlog = pos$lognormal[,1], sdlog= pos$lognormal[,2]), probs = c(0.975)))
))

# Grafique las curvas de la distribución acumulada 
gamma_ggplot <- ggplot(df, aes(x=inc_day)) +
  stat_ecdf(geom = "step")+ 
  xlim(c(0, 30))+
  geom_line(data=Gam_plot, aes(x=x_plot, y=pred), color=RColorBrewer::brewer.pal(11, "RdBu")[11], linewidth=1) +
  geom_ribbon(data=Gam_plot, aes(x=x_plot,ymin=low,ymax=upp), fill = RColorBrewer::brewer.pal(11, "RdBu")[11], alpha=0.1) +
  theme_bw(base_size = 11)+
  labs(x="Incubation period (days)", y = "Proportion")+
  ggtitle("Gamma")

weibul_ggplot <- ggplot(df, aes(x=inc_day)) +
  stat_ecdf(geom = "step")+ 
  xlim(c(0, 30))+
  geom_line(data=Wei_plot, aes(x=x_plot, y=pred), color=RColorBrewer::brewer.pal(11, "RdBu")[11], linewidth=1) +
  geom_ribbon(data=Wei_plot, aes(x=x_plot,ymin=low,ymax=upp), fill = RColorBrewer::brewer.pal(11, "RdBu")[11], alpha=0.1) +
  theme_bw(base_size = 11)+
  labs(x="Incubation period (days)", y = "Proportion")+
  ggtitle("Weibull")

lognorm_ggplot <- ggplot(df, aes(x=inc_day)) +
  stat_ecdf(geom = "step")+ 
  xlim(c(0, 30))+
  geom_line(data=ln_plot, aes(x=x_plot, y=pred), color=RColorBrewer::brewer.pal(11, "RdBu")[11], linewidth=1) +
  geom_ribbon(data=ln_plot, aes(x=x_plot,ymin=low,ymax=upp), fill = RColorBrewer::brewer.pal(11, "RdBu")[11], alpha=0.1) +
  theme_bw(base_size = 11)+
  labs(x="Incubation period (days)", y = "Proportion")+
  ggtitle("Log normal")

(lognorm_ggplot|gamma_ggplot|weibul_ggplot) + plot_annotation(tag_levels = 'A') 
```

<img src="fig/EnfermedadX-rendered-plot ip-1.png" style="display: block; margin: auto;" />

En los gráficos anteriores, la línea negra es la distribución acumulativa empírica (los datos), mientras que la curva azul es la distribución de probabilidad ajustada con los intervalos de credibilidad del 95%. Asegúrese de que la curva azul esté sobre la línea negra.


::: {.alert .alert-secondary}
*💡 **Preguntas (6)***

-   ¿Son los ajustes de las distribuciones lo que espera?
:::

#### 

# *`_______Pausa 2 _________`*

## 8. Estimación del intervalo serial 

Ahora, estime el intervalo serial. Nuevamente, se realizará primero una estimación navie calculando la diferencia entre la fecha de inicio de síntomas entre el par de casos primario y secundario. 

1.  ¿Existen casos con intervalos seriales negativos en los datos (por ejemplo, el inicio de los síntomas en el caso secundario ocurrió antes del inicio de los síntomas en el caso primario)?

2.  Informe la mediana del intervalo serial, así como el mínimo y el máximo.

3.  Grafique la distribución del intervalo serial.

### 8.1. Estimación naive 


```r
contacts$diff <- as.numeric(contacts$secondary_onset_date - contacts$primary_onset_date)
summary(contacts$diff)
```

```{.output}
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
  1.000   4.000   6.000   7.625  10.000  23.000 
```

```r
hist(contacts$diff, xlab = "Serial interval (days)", breaks = 25, main = "", col = "pink")
```

<img src="fig/EnfermedadX-rendered-si naive-1.png" style="display: block; margin: auto;" />

### 8.2. Estimación ajustada por censura

Ahora se estimará el intervalo serial utilizando una implementación del paquete `courseDataTools` dentro del paquete R `EpiEstim`. Este método tiene en cuenta la censura doble y permite comparar diferentes distribuciones de probabilidad, pero no se ajusta por truncamiento a la derecha o sesgo dinámico.

Se considerará tres distribuciones de probabilidad y deberá seleccionar la que mejor se ajuste a los datos utilizando WAIC o LOOIC. Recuerde que la distribución con mejor ajuste tendrá el WAIC o LOOIC más bajo.

Ten en cuenta que en `coarseDataTools`, los parámetros para las distribuciones son ligeramente diferentes que en rstan. Aquí, los parámetros para la distribución gamma son shape y scale (forma y escala) (<https://cran.r-project.org/web/packages/coarseDataTools/coarseDataTools.pdf>).

Solo se ejecutará una cadena MCMC para cada distribución en interés del tiempo, pero en la práctica debería ejecutar más de una cadena para asegurarse de que el MCMC converge en la distribución objetivo. Usará las distribuciones a priori predeterminadas, que se pueden encontrar en la documentación del paquete (ver 'detalles' para la función dic.fit.mcmc aquí:
 (<https://cran.r-project.org/web/packages/coarseDataTools/coarseDataTools.pdf>).

#### 8.2.1. Preparación de los datos


```r
# Format interval-censored serial interval data

# Each line represents a transmission event 
# EL/ER show the lower/upper bound of the symptoms onset date in the primary case (infector)
# SL/SR show the same for the secondary case (infectee)
# type has entries 0 corresponding to doubly interval-censored data
# (see Reich et al. Statist. Med. 2009)
si_data <- contacts |>
  select(-primary_case_id, -secondary_case_id, -primary_onset_date, -secondary_onset_date,) |>
  rename(SL = diff) |>
  mutate(type = 0, EL = 0, ER = 1, SR = SL + 1) |>
  select(EL, ER, SL, SR, type)
```

#### 8.2.2. Ajuste una distribución gamma para el SI

Primero, ajuste una distribución gamma al intervalo serial.


```r
overall_seed <- 3 # seed for the random number generator for MCMC
MCMC_seed <- 007

# We will run the model for 4000 iterations with the first 1000 samples discarded as burnin
n_mcmc_samples <- 3000 # number of samples to draw from the posterior (after the burnin)

params = list(
dist = "G", # Fitting a Gamma distribution for the SI
method = "MCMC", # MCMC using coarsedatatools
burnin = 1000, # number of burnin samples (samples discarded at the beginning of MCMC) 
n1 = 50, # n1 is the number of pairs of mean and sd of the SI that are drawn
n2 = 50) # n2 is the size of the posterior sample drawn for each pair of mean, sd of SI

mcmc_control <- make_mcmc_control(
seed = MCMC_seed,
burnin = params$burnin)

dist <- params$dist

config <- make_config(
list(
si_parametric_distr = dist,
mcmc_control = mcmc_control,
seed = overall_seed,
n1 = params$n1,
n2 = params$n2))

# Fitting the SI
si_fit_gamma <- coarseDataTools::dic.fit.mcmc(
dat = si_data,
dist = dist,
init.pars = init_mcmc_params(si_data, dist),
burnin = mcmc_control$burnin,
n.samples = n_mcmc_samples,
seed = mcmc_control$seed)
```

```{.output}
Running 4000 MCMC iterations 
MCMCmetrop1R iteration 1 of 4000 
function value = -201.95281
theta = 
   1.04012
   0.99131
Metropolis acceptance rate = 0.00000

MCMCmetrop1R iteration 1001 of 4000 
function value = -202.42902
theta = 
   0.95294
   1.05017
Metropolis acceptance rate = 0.55445

MCMCmetrop1R iteration 2001 of 4000 
function value = -201.88547
theta = 
   1.18355
   0.82124
Metropolis acceptance rate = 0.56522

MCMCmetrop1R iteration 3001 of 4000 
function value = -202.11109
theta = 
   1.26323
   0.77290
Metropolis acceptance rate = 0.55148



@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
The Metropolis acceptance rate was 0.55500
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

Ahora observe los resultados.


```r
# Check convergence of MCMC chains 
converg_diag_gamma <- check_cdt_samples_convergence(si_fit_gamma@samples)
converg_diag_gamma


# Save MCMC samples in dataframe
si_samples_gamma <- data.frame(
type = 'Symptom onset',
shape = si_fit_gamma@samples$var1,
scale = si_fit_gamma@samples$var2,
p50 = qgamma(
p = 0.5, 
shape = si_fit_gamma@samples$var1, 
scale = si_fit_gamma@samples$var2)) |>
mutate( # Equation for conversion is here https://en.wikipedia.org/wiki/Gamma_distribution
mean = shape*scale,
sd = sqrt(shape*scale^2)
) 

# Get the mean, SD, and 95% CrI


si_summary_gamma <- 
  si_samples_gamma %>%
summarise(
mean_mean = quantile(mean,probs=.5),
mean_l_ci = quantile(mean,probs=.025),
mean_u_ci = quantile(mean,probs=.975),
sd_mean = quantile(sd, probs=.5),
sd_l_ci = quantile(sd,probs=.025),
sd_u_ci = quantile(sd,probs=.975)
)
si_summary_gamma

# Get the same summary statistics for the parameters of the distribution
si_samples_gamma |>
summarise(
shape_mean = quantile(shape, probs=.5),
shape_l_ci = quantile(shape, probs=.025),
shape_u_ci = quantile(shape, probs=.975),
scale_mean = quantile(scale, probs=.5),
scale_l_ci = quantile(scale, probs=.025),
scale_u_ci = quantile(scale, probs=.975)
)

# We will need these for plotting later
gamma_shape <- si_fit_gamma@ests['shape',][1]
gamma_rate <- 1 / si_fit_gamma@ests['scale',][1]
```

#### 8.2.3. Ajuste de una distribución log normal para el intervalo serial

Ahora, ajuste una distribución log normal a los datos del intervalo serial.


```r
# We will run the model for 4000 iterations with the first 1000 samples discarded as burnin
n_mcmc_samples <- 3000 # number of samples to draw from the posterior (after the burnin)

params = list(
dist = "L", # Fitting a log normal distribution for the SI
method = "MCMC", # MCMC using coarsedatatools
burnin = 1000, # number of burnin samples (samples discarded at the beginning of MCMC) 
n1 = 50, # n1 is the number of pairs of mean and sd of the SI that are drawn
n2 = 50) # n2 is the size of the posterior sample drawn for each pair of mean, sd of SI

mcmc_control <- make_mcmc_control(
seed = MCMC_seed,
burnin = params$burnin)

dist <- params$dist

config <- make_config(
list(
si_parametric_distr = dist,
mcmc_control = mcmc_control,
seed = overall_seed,
n1 = params$n1,
n2 = params$n2))

# Fitting the serial interval
si_fit_lnorm <- coarseDataTools::dic.fit.mcmc(
dat = si_data,
dist = dist,
init.pars = init_mcmc_params(si_data, dist),
burnin = mcmc_control$burnin,
n.samples = n_mcmc_samples,
seed = mcmc_control$seed)
```

```{.output}
Running 4000 MCMC iterations 
MCMCmetrop1R iteration 1 of 4000 
function value = -216.54581
theta = 
   1.94255
  -0.47988
Metropolis acceptance rate = 1.00000

MCMCmetrop1R iteration 1001 of 4000 
function value = -216.21549
theta = 
   1.81272
  -0.47418
Metropolis acceptance rate = 0.56643

MCMCmetrop1R iteration 2001 of 4000 
function value = -215.86791
theta = 
   1.85433
  -0.52118
Metropolis acceptance rate = 0.57221

MCMCmetrop1R iteration 3001 of 4000 
function value = -216.32347
theta = 
   1.93196
  -0.52205
Metropolis acceptance rate = 0.55948



@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
The Metropolis acceptance rate was 0.56875
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

Revise los resultados.


```r
# Revise la convergencia de las cadenas MCMC 
converg_diag_lnorm <- check_cdt_samples_convergence(si_fit_lnorm@samples)
converg_diag_lnorm


# Guarde las muestras de MCMC en un dataframe
si_samples_lnorm <- data.frame(
type = 'Symptom onset',
meanlog = si_fit_lnorm@samples$var1,
sdlog = si_fit_lnorm@samples$var2,
p50 = qlnorm(
p = 0.5, 
meanlog = si_fit_lnorm@samples$var1, 
sdlog = si_fit_lnorm@samples$var2)) |>
mutate( # La ecuación para la conversión está aquí https://en.wikipedia.org/wiki/Log-normal_distribution
mean = exp(meanlog + (sdlog^2/2)), 
sd = sqrt((exp(sdlog^2)-1) * (exp(2*meanlog + sdlog^2)))
)

# Obtenga la media, desviación estándar e intervalo de credibilidad del 95% 
si_summary_lnorm <- 
  si_samples_lnorm %>%
summarise(
mean_mean = quantile(mean,probs=.5),
mean_l_ci = quantile(mean,probs=.025),
mean_u_ci = quantile(mean,probs=.975),
sd_mean = quantile(sd, probs=.5),
sd_l_ci = quantile(sd,probs=.025),
sd_u_ci = quantile(sd,probs=.975)
)
si_summary_lnorm

# Obtenga las estadísticas reumen para los parámetros de la distribución
si_samples_lnorm |>
summarise(
meanlog_mean = quantile(meanlog, probs=.5),
meanlog_l_ci = quantile(meanlog, probs=.025),
meanlog_u_ci = quantile(meanlog, probs=.975),
sdlog_mean = quantile(sdlog, probs=.5),
sdlog_l_ci = quantile(sdlog, probs=.025),
sdlog_u_ci = quantile(sdlog, probs=.975)
)

lognorm_meanlog <- si_fit_lnorm@ests['meanlog',][1]
lognorm_sdlog <- si_fit_lnorm@ests['sdlog',][1]
```

#### 8.2.4. Ajuste de una distribución Weibull para el intervalo serial

Finalmente, ajuste de una distribución Weibull para los datos del intervalo serial.


```r
# We will run the model for 4000 iterations with the first 1000 samples discarded as burnin
n_mcmc_samples <- 3000 # number of samples to draw from the posterior (after the burnin)

params = list(
dist = "W", # Fitting a weibull distribution for the SI
method = "MCMC", # MCMC using coarsedatatools
burnin = 1000, # number of burnin samples (samples discarded at the beginning of MCMC) 
n1 = 50, # n1 is the number of pairs of mean and sd of the SI that are drawn
n2 = 50) # n2 is the size of the posterior sample drawn for each pair of mean, sd of SI

mcmc_control <- make_mcmc_control(
seed = MCMC_seed,
burnin = params$burnin)

dist <- params$dist

config <- make_config(
list(
si_parametric_distr = dist,
mcmc_control = mcmc_control,
seed = overall_seed,
n1 = params$n1,
n2 = params$n2))

# Ajuste el intervalo serial 
si_fit_weibull <- coarseDataTools::dic.fit.mcmc(
dat = si_data,
dist = dist,
init.pars = init_mcmc_params(si_data, dist),
burnin = mcmc_control$burnin,
n.samples = n_mcmc_samples,
seed = mcmc_control$seed)
```

Revise los resultados.


```r
# Revise covengencia
converg_diag_weibull <- check_cdt_samples_convergence(si_fit_weibull@samples)
```

```{.output}

Gelman-Rubin MCMC convergence diagnostic was successful.
```

```r
converg_diag_weibull
```

```{.output}
[1] TRUE
```

```r
# Guarde las muestra MCMC en un dataframe
si_samples_weibull <- data.frame(
type = 'Symptom onset',
shape = si_fit_weibull@samples$var1,
scale = si_fit_weibull@samples$var2,
p50 = qweibull(
p = 0.5, 
shape = si_fit_weibull@samples$var1, 
scale = si_fit_weibull@samples$var2)) |>
mutate( # La cuación para conversión está aquí https://en.wikipedia.org/wiki/Weibull_distribution
mean = scale*gamma(1+1/shape),
sd = sqrt(scale^2*(gamma(1+2/shape)-(gamma(1+1/shape))^2))
) 

# Obtenga las estadísticas resumen
si_summary_weibull <- 
  si_samples_weibull %>%
summarise(
mean_mean = quantile(mean,probs=.5),
mean_l_ci = quantile(mean,probs=.025),
mean_u_ci = quantile(mean,probs=.975),
sd_mean = quantile(sd, probs=.5),
sd_l_ci = quantile(sd,probs=.025),
sd_u_ci = quantile(sd,probs=.975)
)
si_summary_weibull
```

```{.output}
  mean_mean mean_l_ci mean_u_ci  sd_mean  sd_l_ci  sd_u_ci
1  7.692701  6.677548  8.809715 4.418743 3.797158 5.456492
```

```r
# Obtenga las estadísticas resumen para los parámetros de la distribución.
si_samples_weibull |>
summarise(
shape_mean = quantile(shape, probs=.5),
shape_l_ci = quantile(shape, probs=.025),
shape_u_ci = quantile(shape, probs=.975),
scale_mean = quantile(scale, probs=.5),
scale_l_ci = quantile(scale, probs=.025),
scale_u_ci = quantile(scale, probs=.975)
)
```

```{.output}
  shape_mean shape_l_ci shape_u_ci scale_mean scale_l_ci scale_u_ci
1   1.791214    1.50812   2.090071   8.631295   7.481234   9.943215
```

```r
weibull_shape <- si_fit_weibull@ests['shape',][1]
weibull_scale <- si_fit_weibull@ests['scale',][1]
```

#### 8.2.5. Grafique los resultados para el intervalo serial

Ahora, grafique el ajuste de las tres distribuciones. Asegúrese que la distribución ajuste bien los datos del intervalo serial.


```r
ggplot()+
	theme_classic()+
	geom_bar(data = contacts, aes(x=diff), fill = "#FFAD05") +
	scale_y_continuous(limits = c(0,14), breaks = c(0,2,4,6,8,10,12,14), expand = c(0,0)) +
	stat_function(
		linewidth=.8,
		fun = function(z, shape, rate)(dgamma(z, shape, rate) * length(contacts$diff)),
		args = list(shape = gamma_shape, rate = gamma_rate),
		aes(linetype  = 'Gamma')
	) +
	stat_function(
		linewidth=.8,
		fun = function(z, meanlog, sdlog)(dlnorm(z, meanlog, sdlog) * length(contacts$diff)),
		args = list(meanlog = lognorm_meanlog, sdlog = lognorm_sdlog),
		aes(linetype ='Log normal')
	) +
	stat_function(
		linewidth=.8,
		fun = function(z, shape, scale)(dweibull(z, shape, scale) * length(contacts$diff)),
		args = list(shape = weibull_shape, scale = weibull_scale),
		aes(linetype ='Weibull')
	) +
	scale_linetype_manual(values = c('solid','twodash','dotted')) +
	labs(x = "Days", y = "Number of case pairs") + 
	theme(
		legend.position = c(0.75, 0.75),
		plot.margin = margin(.2,5.2,.2,.2, "cm"),
		legend.title = element_blank(),
	) 
```

<img src="fig/EnfermedadX-rendered-plot dist-1.png" style="display: block; margin: auto;" />

Ahora calcule el WAIC y LOOIC. `coarseDataTools` no tiene una forma integrada de hacer esto, por lo que se necesita calcular la verosimilitud a partir de las cadenas MCMC y utilizar el paquete `loo` en R.



```r
# Cargue las funciones de verosimilitud de coarseDataTools
source("Functions/likelihood_function.R")
```

```{.warning}
Warning in file(filename, "r", encoding = encoding): cannot open file
'Functions/likelihood_function.R': No such file or directory
```

```{.error}
Error in file(filename, "r", encoding = encoding): cannot open the connection
```

```r
compare_gamma <- calc_looic_waic(symp = si_data, symp_si = si_fit_gamma, dist = "G")
```

```{.warning}
Warning: Relative effective sample sizes ('r_eff' argument) not specified.
For models fit with MCMC, the reported PSIS effective sample sizes and 
MCSE estimates will be over-optimistic.
```

```r
compare_lnorm <- calc_looic_waic(symp = si_data, symp_si = si_fit_lnorm, dist = "L")
```

```{.warning}
Warning: Relative effective sample sizes ('r_eff' argument) not specified.
For models fit with MCMC, the reported PSIS effective sample sizes and 
MCSE estimates will be over-optimistic.
```

```{.warning}
Warning: Some Pareto k diagnostic values are slightly high. See help('pareto-k-diagnostic') for details.
```

```r
compare_weibull <- calc_looic_waic(symp = si_data, symp_si = si_fit_weibull, dist = "W")
```

```{.warning}
Warning: 
2 (2.8%) p_waic estimates greater than 0.4. We recommend trying loo instead.
```

```{.warning}
Warning: Relative effective sample sizes ('r_eff' argument) not specified.
For models fit with MCMC, the reported PSIS effective sample sizes and 
MCSE estimates will be over-optimistic.
```

```r
# Imprima resultados
compare_gamma[["waic"]]$estimates
```

```{.output}
             Estimate         SE
elpd_waic -202.009546  7.0118463
p_waic       1.936991  0.4409454
waic       404.019093 14.0236925
```

```r
compare_lnorm[["waic"]]$estimates
```

```{.output}
             Estimate         SE
elpd_waic -202.215546  7.0178113
p_waic       1.874728  0.4106866
waic       404.431092 14.0356226
```

```r
compare_weibull[["waic"]]$estimates
```

```{.output}
             Estimate         SE
elpd_waic -203.941362  6.9020056
p_waic       2.083623  0.6633186
waic       407.882723 13.8040112
```

```r
compare_gamma[["looic"]]$estimates
```

```{.output}
            Estimate         SE
elpd_loo -202.015856  7.0144149
p_loo       1.943301  0.4437589
looic     404.031713 14.0288297
```

```r
compare_lnorm[["looic"]]$estimates
```

```{.output}
            Estimate         SE
elpd_loo -202.218954  7.0182826
p_loo       1.878136  0.4114664
looic     404.437909 14.0365651
```

```r
compare_weibull[["looic"]]$estimates
```

```{.output}
            Estimate         SE
elpd_loo -203.956018  6.9097884
p_loo       2.098279  0.6725387
looic     407.912036 13.8195768
```

Incluya lo siguiente cuando reporte el intervalo serial:

-   Media e intervalo de credibilidad del 95% 

-   Desviación estándar y e intervalo de credibilidad del 95% 

-   Los parámetros de la distribución ajustada (e.g., shape y scale para distribución gamma)

::: {.alert .alert-secondary}
*💡 **Preguntas (7)***

-   ¿Qué distribución tiene el menor WAIC y LOOIC??
:::


```r
si_summary_gamma
```

```{.output}
  mean_mean mean_l_ci mean_u_ci  sd_mean  sd_l_ci  sd_u_ci
1  7.659641   6.65395  8.707372 4.342087 3.602425 5.481316
```

```r
si_summary_lnorm
```

```{.output}
  mean_mean mean_l_ci mean_u_ci  sd_mean  sd_l_ci  sd_u_ci
1  7.780719  6.692829  9.300588 5.189349 3.936678 7.289224
```

```r
si_summary_weibull
```

```{.output}
  mean_mean mean_l_ci mean_u_ci  sd_mean  sd_l_ci  sd_u_ci
1  7.692701  6.677548  8.809715 4.418743 3.797158 5.456492
```

# *`_______Pausa 3 _________`*

## 9. Medidas de control 

### 9.1 Analicemos el resultado juntos

::: {.alert .alert-secondary}
Ahora ha finalizado el análisis estadístico 🥳

-   Compare el período de incubación y el intervalo serial.

-   ¿Cuál es más largo?

-   ¿Ocurre la transmisión pre-sintomática con la Enfermedad X? 

<!-- -->

-   ¿Serán medidas efectivas aislar a los individuos sintomáticos y rastrear y poner en cuarentena a sus contactos?

-   Si no lo son, ¿qué otras medidas se pueden implementar para frenar el brote?

:::

### 9.2 Diseño de una estrategía de rastreo de contactos

Basado en los rezagos estimados así como la demografía de los casos, diseñe una estrategia de rastreo de contactos para el brote, incluyendo un plan de comunicaciones (Pista: piense sobre la logística).

## 10. Valores verdaderos

Los valores verdaderos usados para simular las epidemías fueron:

+---------------+:---------------:+:-------------:+:-------------------------:+
|               | Distribución    | Media (días)   | Desviación estándar (días) |
+---------------+-----------------+---------------+---------------------------+
| Grupo 1       | Log normal      | 5.7           | 4.6                       |
+---------------+-----------------+---------------+---------------------------+
| Grupo 2       | Weibull         | 7.1           | 3.7                       |
+---------------+-----------------+---------------+---------------------------+

: Tabla 2. Valores verdaderos del ***período de incubación.***

+--------------+:---------------:+:------------:+:-------------------------:+
|              | Distribución    | Media (días)  | Desviación estándar (días) |
+--------------+-----------------+--------------+---------------------------+
| Group 1      | Gamma           | 8.4          | 4.9                       |
+--------------+-----------------+--------------+---------------------------+
| Group 2      | Gamma           | 4.8          | 3.3                       |
+--------------+-----------------+--------------+---------------------------+

: Tabla 3. Valores verdaderos del ***intervalo serial***.

¿Cómo se comparan sus estimaciones con los valores verdaderos? Discuta las posibles razones para las diferencias.

::::::::::::::::::::::::::::::::::::: keypoints 

Revise si al final de esta lección adquirió estas competencias:


- Comprender los conceptos clave de las distribuciones de retrasos epidemiológicos para la Enfermedad X.

- Entender las estructuras de datos y las herramientas para el análisis de datos de rastreo de contactos.

- Aprender cómo ajustar las estimaciones del intervalo serial y del período de incubación de la Enfermedad X teniendo en cuenta la censura por intervalo usando un marco de trabajo Bayesiano.

- Aprender a utilizar estos parámetros para informar estrategias de control en un brote de un patógeno desconocido.

::::::::::::::::::::::::::::::::::::::::::::::::

## 11. Recursos adicionales
En este práctica, en gran medida se ignoraron los sesgos de truncamiento a la derecha y sesgo dinámico, en parte debido a la falta de herramientas fácilmente disponibles que implementen las mejores prácticas. Para aprender más sobre cómo estos sesgos podrían afectar la estimación de las distribuciones de retraso epidemiológico en tiempo real, recomendamos un tutorial sobre el paquete `dynamicaltruncation` en R de Sam Abbott y Sang Woo Park (<https://github.com/parksw3/epidist-paper>).

## 12. Contribuidores

Kelly Charniga, Zachary Madewell, Zulma M. Cucunubá


## 13. Referencias

1.  Reich NG et al. Estimating incubation period distributions with coarse data. Stat Med. 2009;28:2769--84. PubMed <https://doi.org/10.1002/sim.3659>
2.  Miura F et al. Estimated incubation period for monkeypox cases confirmed in the Netherlands, May 2022. Euro Surveill. 2022;27(24):pii=2200448. <https://doi.org/10.2807/1560-7917.ES.2022.27.24.2200448>
3.  Abbott S, Park Sang W. Adjusting for common biases in infectious disease data when estimating distributions. 2022 [cited 7 November 2023]. <https://github.com/parksw3/dynamicaltruncation>
4.  Lessler J et al. Incubation periods of acute respiratory viral infections: a systematic review, The Lancet Infectious Diseases. 2009;9(5):291-300. [https://doi.org/10.1016/S1473-3099(09)70069-6](https://doi.org/10.1016/S1473-3099(09)70069-6){.uri}.
5.  Cori A et al. Estimate Time Varying Reproduction Numbers from Epidemic Curves. 2022 [cited 7 November 2023]. <https://github.com/mrc-ide/EpiEstim>
6.  Lambert B. A Student's Guide to Bayesian Statistics. Los Angeles, London, New Delhi, Singapore, Washington DC, Melbourne: SAGE, 2018.
7.  Vehtari A et al. Rank-normalization, folding, and localization: An improved R-hat for assessing convergence of MCMC. Bayesian Analysis 2021: Advance publication 1-28. <https://doi.org/10.1214/20-BA1221>
8.  Nishiura H et al. Serial interval of novel coronavirus (COVID-19) infections. Int J Infect Dis. 2020;93:284-286. <https://doi.org/10.1016/j.ijid.2020.02.060>


