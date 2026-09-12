# Actividad 06 - Algoritmos Genéticos en Aprendizaje de Máquina

**Universidad Nacional del Altiplano - Puno**

Ingeniería de Sistemas

Maestría en Ciencia de Datos

**Curso:** Machine Learning I - Grupo A

**Presentado por:** 
* Darwin Rigoberto Mamani Quispe
* Melbis Chino Huaycani

Notebook completo: [`Actividad-06.ipynb`](./Actividad-06.ipynb)

---

## Resumen Ejecutivo

Se implementó un **algoritmo genético (AG) genérico desde cero** (Python + NumPy), reutilizado en tres aplicaciones distintas de aprendizaje de máquina sobre el dataset `breast_cancer` de scikit-learn (569 muestras, 30 características numéricas, clasificación binaria maligno/benigno), usando la misma partición train/test (70/30, estratificada) de las Actividades 04 y 05 para asegurar comparabilidad.

En los tres casos el ciclo del AG sigue la misma estructura:

**Representación del cromosoma → Inicialización aleatoria de la población → Función de aptitud (accuracy por validación cruzada) → Selección por torneo con elitismo → Cruzamiento → Mutación → Terminación (n generaciones fijas)**

---

## 1. Feature Selection (Parte A)

| | |
|---|---|
| **Modelo base** | Regresión Logística |
| **Cromosoma** | Vector binario de 30 genes (1 = característica incluida) |
| **Aptitud** | Accuracy CV (5-fold) − pequeña penalización por N° de características usadas |
| **Cruzamiento** | Un punto de corte |
| **Mutación** | Bit-flip |
| **Configuración** | Población de 16 individuos, 12 generaciones (2.7 s) |

## 2. Hyperparameter Optimization (Parte B)

| | |
|---|---|
| **Modelo base** | Árbol de Decisión (CART) |
| **Cromosoma** | `[max_depth, min_samples_split, min_samples_leaf, criterio]` |
| **Aptitud** | Accuracy CV (5-fold) |
| **Cruzamiento** | Un punto de corte |
| **Mutación** | Resampleo de un gen dentro de su rango válido |
| **Configuración** | Población de 14 individuos, 10 generaciones (3.6 s) |
| **Mejor solución** | `max_depth=3, min_samples_split=4, min_samples_leaf=3, criterion='gini'` |

## 3. Neuroevolution (Parte C)

| | |
|---|---|
| **Modelo base** | Red Neuronal (MLPClassifier) |
| **Cromosoma** | `[n_capas (1-3), neuronas_capa1, neuronas_capa2, neuronas_capa3, activación]` |
| **Aptitud** | Accuracy CV (3-fold, para reducir el costo computacional) |
| **Cruzamiento** | Uniforme (50% por gen) |
| **Mutación** | Resampleo de cada gen dentro de su conjunto válido |
| **Configuración** | Población de 8 individuos, 6 generaciones (3.8 s) |
| **Mejor solución** | 3 capas ocultas `(32, 64, 32)`, activación `tanh` |

---

## Resultados comparativos

| Parte | Modelo base | Configuración encontrada por el AG | Accuracy baseline | Accuracy con AG |
|---|---|---|---|---|
| A - Feature Selection | Regresión Logística | 11 de 30 características | 0.988 | 0.959* |
| B - Hyperparameter Optimization | Árbol de Decisión | max_depth=3, min_samples_split=4, min_samples_leaf=3, gini | 0.918 | 0.924 |
| C - Neuroevolution | Red Neuronal (MLP) | 3 capas (32,64,32), activación tanh | 0.953 | 0.977 |

> \* El accuracy CV (5-fold) de la Parte A sí mejora con el AG (0.980 → 0.985); el accuracy de test puntual baja levemente — ver conclusiones.

---

## Conclusiones

- En feature selection, el AG redujo el número de variables en casi 65% (de 30 a 11) mejorando la validación cruzada, aunque el accuracy puntual de test bajó levemente — resultado esperado, ya que el AG optimiza directamente sobre CV del set de entrenamiento y una sola partición de test puede no reflejar exactamente esa mejora.
- En hyperparameter optimization, el AG mejoró tanto la validación cruzada como el accuracy de test del árbol frente a la configuración por defecto, encontrando un árbol menos profundo y menos propenso a sobreajustar.
- En neuroevolution, el AG encontró una arquitectura de red neuronal claramente superior a una arquitectura genérica por defecto (accuracy de test de 0.953 a 0.977).

En general, en los tres casos el AG permitió explorar espacios de búsqueda muy grandes (2³⁰ combinaciones de características, miles de combinaciones de hiperparámetros, y un espacio combinatorio de arquitecturas de red) de forma mucho más eficiente que una búsqueda manual o exhaustiva, encontrando en pocas generaciones soluciones competitivas o mejores que las configuraciones por defecto.
