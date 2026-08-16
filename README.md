# Riesgo clínico cardiovascular — Fundamentos probabilísticos

Análisis probabilístico sobre 297 pacientes cardíacos para responder una pregunta operativa: **qué información justifica derivar a un paciente a cateterismo**.

Proyecto de Primer Corte · Aprendizaje de Máquina No Supervisado · Universidad de La Sabana

---

## El problema de negocio

Una red de clínicas cardiológicas necesita decidir a qué pacientes deriva a angiografía coronaria — un procedimiento invasivo y costoso, con capacidad limitada. Hoy la priorización se apoya en el síntoma reportado por el paciente.

Este análisis evalúa si ese criterio es el correcto, usando once conceptos probabilísticos aplicados a datos clínicos reales.

## El dataset

**Heart Disease Data Set** — base de Cleveland, UCI Machine Learning Repository (ID 45).
Janosi, Steinbrunn, Pfisterer & Detrano (1989). Licencia CC BY 4.0.
https://archive.ics.uci.edu/dataset/45/heart+disease

- 303 registros originales, 14 atributos clínicos
- 6 eliminados por datos ausentes en `ca` y `thal` → **muestra final: n = 297**
- Variable objetivo: `num` binarizada en ausencia (0) vs. presencia (≥1)

Se usó **exclusivamente la base de Cleveland**. Las bases de Hungría y Suiza del mismo repositorio codifican el colesterol no medido como `0`, lo que habría contaminado la estimación por máxima verosimilitud.

---

## Hallazgos principales

### 1. El triaje por intensidad del dolor está invertido

Los pacientes **asintomáticos** presentan enfermedad en el **72.5%** de los casos, frente al 18.4% de quienes reportan angina atípica — y muy por encima de la probabilidad basal de 46.1%. Son 142 de los 297 pacientes, casi la mitad de la muestra.

Clínicamente corresponde a isquemia silente: obstrucción coronaria avanzada sin dolor característico, frecuente en pacientes diabéticos con neuropatía. La misma condición que daña las arterias daña los nervios que deberían avisar.

### 2. El colesterol aislado no discrimina

| Variable | Divergencia KL (10 bins) |
|---|---|
| `oldpeak` — depresión ST | **0.6409** bits |
| `thalach` — FC máxima | 0.5555 bits |
| `age` — edad | 0.2716 bits |
| `trestbps` — presión reposo | 0.0699 bits |
| `chol` — colesterol | **0.0657** bits |

El colesterol separa enfermos de sanos **9.8 veces peor** que la depresión del segmento ST. Las medias por grupo difieren en apenas 8.4 mg/dl (243.5 vs. 251.9) con desviaciones cercanas a 50.

La conclusión es robusta a la discretización: con 5 bins KL da 0.0237 y con 20 bins 0.1855, pero incluso en el escenario más favorable sigue por debajo de `oldpeak` medido con 10 bins.

### 3. Tres pruebas de bajo costo elevan la sospecha de 46% a 98%

Combinando angina por ejercicio, vasos obstruidos y prueba de talio, la probabilidad pasa de **0.4613** a **0.9770** — y el resultado es idéntico en cualquier orden de evaluación, porque en *odds* la actualización bayesiana es un producto de razones de verosimilitud.

---

## Los 11 conceptos aplicados

| Concepto | Resultado |
|---|---|
| Probabilidad condicional | P(D \| cp=4) = 0.7254 · P(D \| cp=2) = 0.1837 |
| Teorema de Bayes | prior 0.4613 → posterior 0.7629 · LR⁺ = 3.758 |
| Verosimilitud | AIC Log-Normal 1702.0 vs. Normal 1732.1 (Δ = 30.1) |
| MLE | μ̂ sanos = 243.49 · μ̂ enfermos = 251.85 mg/dl |
| Distribuciones paramétricas | `sex`→Bernoulli(0.677) · `cp`→Categórica(4) · `chol`→Log-Normal · `thalach`→Gaussiana |
| Esperanza y varianza | E[thalach]: 158.58 vs. 139.11 lpm · Var: 362.65 vs. 515.77 (+42%) |
| Independencia | χ²(sex, target) = 21.85, p = 2.9×10⁻⁶ |
| Correlación | r(oldpeak, slope) = 0.579 — multicolinealidad |
| Prior y posterior | 0.4613 → 0.9770, idéntico en ambos órdenes |
| Entropía | H = 0.9957 bits · H(·\|cp) = 0.7985 · ganancia = 0.1972 bits (19.8%) |
| Entropía cruzada | test = 0.2944 nats vs. baseline 0.6910 (−57.4%) |
| Divergencia KL | `chol` = 0.0657 vs. `oldpeak` = 0.6409 bits |

---

## Recomendaciones

1. **Redefinir el criterio de derivación**, reemplazando la priorización por intensidad del dolor por un puntaje objetivo basado en prueba de esfuerzo. El asintomático debe marcarse como alto riesgo, no bajo.
2. **Retirar el colesterol sérico del algoritmo de triaje.** Su valor clínico permanece en el manejo del riesgo a largo plazo — son decisiones distintas.
3. **Calibrar umbrales por sexo.** La prevalencia difiere 29.7 puntos entre hombres y mujeres; un corte único subdiagnostica a las mujeres.
4. **Documentar la actualización secuencial**, permitiendo detener el proceso diagnóstico cuando la sospecha ya es concluyente.

---

## Limitaciones

- **La probabilidad basal de 46.1% no es prevalencia poblacional.** Cleveland es una muestra de pacientes *ya referidos a angiografía*. La prevalencia en población general ronda el 5–10%. Los posteriores de este análisis aplican únicamente a pacientes que pasaron ese filtro; con prior de 7%, la misma cadena da 0.7886 en vez de 0.9770.
- **El posterior de 97.7% es una cota superior.** La actualización secuencial asume independencia condicional de las tres pruebas dado el diagnóstico, supuesto cuestionable dado que miden facetas de la misma isquemia.
- **n = 297 limita la validación.** El conjunto de prueba tiene 60 pacientes; la entropía cruzada de test resultó menor que la de entrenamiento, lo que refleja ruido de muestreo, no mérito del modelo.
- **Contexto temporal.** Los datos son de 1988, previos a la generalización de estatinas y angioplastia primaria.

---

## Estructura del repositorio

```
├── data/
│   └── processed_cleveland.data      # dataset original (UCI, sin encabezados)
├── notebooks/
│   └── proyecto_heart_disease.ipynb  # análisis completo, 11 conceptos
├── informe/
│   └── informe_gerencial.pdf         # informe ejecutivo, 2 páginas
└── README.md
```

## Cómo reproducir

```bash
pip install pandas numpy scipy scikit-learn matplotlib
jupyter notebook notebooks/proyecto_heart_disease.ipynb
```

Ajustar la variable `RUTA` en la primera celda según la ubicación del archivo de datos. El notebook se ejecuta de principio a fin sin errores y genera todas las cifras citadas en el informe.

**Stack:** Python · pandas · NumPy · SciPy · scikit-learn · matplotlib

---

## Referencia

Janosi, A., Steinbrunn, W., Pfisterer, M., & Detrano, R. (1989). *Heart Disease* [Data set]. UCI Machine Learning Repository. https://doi.org/10.24432/C52P4X
