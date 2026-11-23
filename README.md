[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/esteban-nieto/heart-disease-crispdm/blob/main/notebooks/analisis_crispdm.ipynb)

# Enfermedad cardiaca: factores de riesgo y modelos predictivos (CRISP-DM)

Análisis de qué factores de salud y hábitos se asocian con infarto o enfermedad cardiaca, y de qué tan bien se puede ordenar el riesgo con modelos predictivos. Sigue la metodología CRISP-DM.

**Autor:** Alejandro Pedreros
**Fecha:** 22/11/2026

El entregable es un único cuadernillo: `notebooks/analisis_crispdm.ipynb`. Se ejecuta de principio a fin con **Ejecutar todo**, sin pasos manuales.

## Datos

- **Fuente:** [Heart Disease Health Indicators (alexteboul)](https://www.kaggle.com/datasets/alexteboul/heart-disease-health-indicators-dataset) en Kaggle, licencia CC0, basado en la encuesta BRFSS 2015 de los CDC.
- **Carga:** el cuadernillo descarga el CSV directamente desde la URL raw de GitHub. No hay que descargar ni subir archivos a mano.
- **Tamaño:** 253.680 registros y 22 columnas. Se eliminan 23.899 duplicados exactos y quedan 229.781 registros.
- **Objetivo:** `HeartDiseaseorAttack` (infarto o enfermedad cardiaca), con 10,3 % de casos positivos.

## Estructura del repositorio

```
heart-disease-crispdm/
├── notebooks/
│   └── analisis_crispdm.ipynb    Análisis completo, de principio a fin
├── data/
│   └── heart_disease_health_indicators_BRFSS2015.csv    Dato original (referencia)
├── requirements.txt
├── .gitignore
└── README.md
```

Al ejecutar el cuadernillo se crean, localmente en la sesión, las carpetas `data/` (base SQLite) y `reports/` (tablas, figuras y reporte). Son efímeras en Colab y no se suben solas a GitHub.

## Qué genera el cuadernillo en `reports/`

- **Tablas CSV:** estadísticas descriptivas, asociaciones con el evento, métricas de validación y de prueba, ganancia por deciles, importancias SHAP y por permutación, coeficientes, intervalos bootstrap, métricas por subgrupo y calibración.
- **Figuras** (`reports/figures/`): distribuciones, correlación, tasas por edad y salud, curvas ROC, matriz de confusión, ganancia por deciles, curvas de aprendizaje, SHAP y calibración.
- **Reporte ejecutivo** (`reports/reporte_ejecutivo.md`): se arma con código a partir de las cifras calculadas en el cuadernillo.

## Cómo ejecutarlo

### Opción 1: Google Colab (recomendada)

1. Haz clic en **Abrir en Colab** arriba.
2. Ejecuta **Entorno de ejecución → Ejecutar todo**.

La fase de modelamiento tarda unos 30 minutos en Colab, sobre todo por SHAP.

### Opción 2: local

Se recomienda Python 3.11 o 3.12. Desde la raíz del repositorio:

```bash
pip install -r requirements.txt
jupyter notebook notebooks/analisis_crispdm.ipynb
```

Las versiones de las librerías están fijadas en `requirements.txt` para que los resultados coincidan.

## Metodología (CRISP-DM)

1. **Comprensión del negocio:** pregunta, datos y límites del estudio.
2. **Comprensión de los datos:** datos crudos, nulos, duplicados, validación de rangos BRFSS, variables derivadas y carga en SQLite.
3. **Preparación de los datos:** eliminación de duplicados y variables derivadas.
4. **Modelamiento:** línea base, regresión logística, Random Forest y gradient boosting, con validación cruzada de 5 folds.
5. **Evaluación:** métricas en prueba, ganancia por deciles, curvas ROC, matriz de confusión, curvas de aprendizaje, SHAP e importancia por permutación.
6. **Complemento:** intervalos bootstrap, equidad por sexo y edad, y calibración.

Cada bloque del cuadernillo sigue el mismo patrón: un título que explica qué se hace y por qué, la celda de código, y una explicación de lo que significan las cifras para el negocio.

## Resultados principales

| Modelo | ROC-AUC (prueba) | PR-AUC (prueba) | Recall (umbral 0,5) | Precisión (umbral 0,5) |
|---|---|---|---|---|
| Línea base (mayoritaria) | 0,500 | 0,103 | 0,000 | 0,000 |
| Regresión logística | 0,836 | 0,373 | 0,778 | 0,252 |
| Random Forest | 0,837 | 0,371 | 0,794 | 0,249 |
| Gradient boosting | 0,838 | 0,377 | 0,801 | 0,246 |

- Los factores más asociados son la edad, la salud general, la presión alta, el colesterol alto y el sexo.
- La regresión logística y el Random Forest rinden casi igual. Sus intervalos de confianza se traslapan.
- Con el mismo umbral, el modelo detecta más casos en hombres que en mujeres, y en mayores de 65 que en menores de 65. En mujeres la precisión también es menor, y en mayores de 65 el ROC-AUC baja a cerca de 0,76.
- Las probabilidades **no están calibradas**. Sirven para ordenar el riesgo, no como probabilidad absoluta de infarto.

## Consideraciones éticas

- **Datos de salud:** la encuesta no incluye nombres ni identificadores directos. Los resultados se presentan en agregado y no deben usarse para identificar personas.
- **Uso del modelo:** las predicciones sirven para orientar la prevención y la conversación clínica. No reemplazan el diagnóstico médico ni deben usarse para decidir seguros, empleo o acceso a servicios.
- **Sesgo y equidad:** la encuesta es telefónica y autoreportada. El desempeño cambia entre hombres y mujeres, y entre mayores y menores de 65 años. Hay que revisar esas brechas antes de cualquier uso real.
- **Falsos negativos y positivos:** con umbral 0,5 hay muchas alertas falsas. Esa carga debe compararse con el costo de pasar por alto un caso.
- **Licencia de los datos:** CC0 (dominio público), con atribución de la fuente.

## Limitaciones

- Datos autoreportados y de corte transversal: muestran asociaciones, no causas.
- Sin búsqueda de hiperparámetros ni validación en otra población o en otro año de BRFSS.
- Las probabilidades no están calibradas.
- SHAP se calcula sobre una muestra de 300 registros de prueba, por costo computacional.
