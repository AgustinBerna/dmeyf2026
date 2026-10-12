# DMEyF 2026 · competencia_01

## 1. Resumen

|  |  |
|---|---|
| Target de entrenamiento | BAJA+1/BAJA+2 contra CONTINUA |
| Features | Basales, flags, signed ranks por drift y lag/delta 1/2; 785 totales |
| Pesos | 50%/25%/25% de las ganancias; sino Junio contaba 2 veces|
| Búsqueda | 60 trials Optuna TPE, secuencial |
| Selección | 10 configuraciones distintas; un único TopN de su curva ponderada |
| Fit final | Marzo–junio; 20 semillas por configuración = 200 modelos |
| Scoring | Agosto; julio sólo aporta historia de features |
| Ensamble | Promedio uniforme de probabilidades de los 200 modelos |
| Corte | 10.500, congelado antes del entrenamiento final |

## 2. Decisiones

### Calidad, inflación y préstamos

Se corrigen problemas identificados por variable/mes: ccajas_depositos de mayo completo a NA (100% cero); ceros de Visa/Master_Finiciomora en mayo/agosto y de Visa/Master_fultimo_cierre en junio a NA. Internet fuera de 0/1 pasan a NA. 

Los montos nominales se ajustan por IPC. La lista y los valores IPC quedan explícitos en el script.

### Aguinaldo

La señal se calcula con valores crudos antes del IPC. Se marca cuando mpayroll de junio es ≥1,5 veces mayo, O el contador cpayroll_trx es ≥2 veces mayo. Si falta algún valor o mayo no es positivo, la señal individual queda NA; cualquiera positiva activa el flag combinado. Fuera de junio, los flags son cero.


### Features agregadas

Se agregaron de las 152 variables basales sus lags1 y 2, delta lag 1 y 2. Tres flags de aguinaldo y signed ranks de Master_mlimitecompra y Master_Finiciomora. Probé muchas combinaciones de features intra-mes sin mejoras en la ganancia. En total son 785 variables.


### Tres folds, selección y corte

Los folds 0 y 1 dejan un mes intermedio. El fold 2 incorpora mayo y evalúa junio sin gap intermedio. La primera prueba fue solo con folds 0 y 1, pero agregar mayo aumenta la ganancia en el público. Además de otra manera no tendria lags2 delta2 en entrenamiento

Se evalúan N=7.000…15.000 cada 500 clientes. Cada trial maximiza:

`0,50 × G_mayo(N) + 0,25 × G_junio_marabril(N) + 0,25 × G_junio_marabrilmay(N)`.

Estos pesos equilibran mayo/junio; no son pesos óptimos demostrados y los dos folds de junio no son poblaciones independientes. La búsqueda y el early stopping usan ganancia.

Se recuperan las predicciones de los top10, se promedian **dentro de cada fold** y se elige un único N de la curva ponderada del ensamble. 

### Semillas y ensamble

Los 200 modelos combinan diversidad de diez modelos y veinte semillas. Se promedian probabilidades; promediar rangos no superó el mejor máximo de probabilidades. 


## 3. Archivos compartidos

```text
README.md
correr.py
pipeline.py
basal_features.py
```


## 4. Paquetes de referencia

Python **3.13.12**, Windows x86_64, 8 threads de entrenamiento y límite DuckDB de 4 GB. La máquina dispone de 16 CPU lógicas y aproximadamente 32 GiB de RAM.

| Paquete | Versión |
|---|---:|
| duckdb | 1.5.5 |
| lightgbm | 4.7.0 |
| optuna | 3.6.1 |
| numpy | 2.4.4 |
| pandas | 3.0.1 |
| pyarrow | 24.0.0 |
| psutil | 7.2.2 |
| scikit-learn | 1.9.1 |


## 5. Reproducir 

Poné competencia_01_crudo.csv (se tiene que llamar asi) junto a los .py y ejecutá:
python -u correr.py
