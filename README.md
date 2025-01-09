# red_neuronal_dengue
Red neuronal entrenada con datos 2023-2024 de datos de laboratorio de dengue


# Preparación de Datos para el Entrenamiento del Modelo de Red Neuronal

El entrenamiento del modelo de red neuronal requirió una preparación minuciosa de los datos para garantizar que las entradas y salidas estuvieran adecuadamente estructuradas. Este proceso fue posible gracias al etiquetado previo, que permitió transformar datos textuales desorganizados en información procesable por el modelo.

## METODOLOGÍA

### Transformación Inicial de las Etiquetas

En los datos originales, las columnas `DES_ANALITO` y `VALOR_RESULTADO` contenían información textual que describía los resultados de laboratorio. Sin embargo, para el modelo era necesario contar con etiquetas numéricas que clasificaran los datos en tres categorías:

- **-1:** Negativo  
- **0:** Neutro o desconocido  
- **1:** Positivo  

A partir del etiquetado anterior, se crearon cuatro nuevas columnas:

- **MOLECULAR:** Etiqueta para resultados relacionados con pruebas moleculares.
- **NS1:** Etiqueta para resultados del antígeno NS1.
- **IGM:** Etiqueta para resultados de anticuerpos IgM.
- **IGG:** Etiqueta para resultados de anticuerpos IgG.

Estas columnas fueron completadas con los valores numéricos correspondientes a las etiquetas numéricas según la interpretación de los datos textuales originales.

### Selección de Columnas de Interés

Para estructurar el DataFrame que se utilizaría en el entrenamiento, se seleccionaron dos grupos de columnas:

1. **Columnas de Entrada (features):**
   - `COD_EXAMEN`: Código único asociado al examen.
   - `VALOR_RESULTADO`: Descripción textual del resultado.
   - `DESC_PLANTILLA`: Detalles sobre el tipo de examen.
   - `INFORME_RESULTADO`: Información adicional del informe.

2. **Columnas de Salida (etiquetas):**
   - `MOLECULAR`, `NS1`, `IGM`, `IGG`.

Con estas columnas, se creó un nuevo DataFrame llamado `df_model`, que contenía únicamente la información necesaria para el modelo.

### Unificación de Datos de Entrada

Para facilitar el procesamiento de las entradas por parte del modelo, las columnas de entrada se unificaron en una sola columna llamada `text_input`. Esta columna se generó concatenando el contenido de las cuatro columnas de entrada (`COD_EXAMEN`, `VALOR_RESULTADO`, `DESC_PLANTILLA` y `INFORME_RESULTADO`) en formato de texto. Este proceso permitió que toda la información relevante estuviera concentrada en una única representación textual, lista para ser vectorizada y utilizada como entrada en la red neuronal.

El etiquetado previo fue crucial para la creación de las columnas `MOLECULAR`, `NS1`, `IGM` e `IGG`, ya que:

1. Facilitó la interpretación de los datos originales al estandarizar los resultados en valores numéricos (-1, 0, 1).
2. Permitió estructurar las etiquetas en un formato que el modelo pudiera procesar eficientemente (creación de las cuatro columnas: `MOLECULAR`, `NS1`, `IGM`, `IGG`).

---

# Desarrollo de la Red Neuronal para Etiquetado Automático de Resultados de Laboratorio de Dengue

El diseño de la red neuronal para el etiquetado de resultados de laboratorio de dengue se basó en un enfoque que integra múltiples características textuales en una única representación vectorial para su procesamiento. A continuación, se detalla el desarrollo técnico del modelo.

## METODOLOGÍA

### Entradas del Modelo

El conjunto de datos inicial incluye varias columnas que aportan información relevante para el etiquetado, específicamente:

- `COD_EXAMEN`: Código único asociado a cada examen de laboratorio.
- `VALOR_RESULTADO`: Resultado textual del examen (positivo, negativo, indeterminado, entre otros).
- `DESC_PLANTILLA`: Descripción estructurada del tipo de examen.
- `INFORME_RESULTADO`: Texto detallado del informe de resultados.

Estas columnas se combinaron en una sola entrada textual mediante la concatenación de sus valores (tratando los datos faltantes como cadenas vacías). Esto permitió crear una columna unificada llamada `text_input`, que sirvió como base para el preprocesamiento y entrenamiento del modelo.

### Preprocesamiento y Vectorización

Para transformar la información textual en un formato que pueda ser procesado por la red neuronal, se utilizó la técnica de **TF-IDF Vectorization**:

1. **Unificación de Texto:** La columna `text_input` fue creada combinando las cuatro columnas mencionadas, asegurando que no hubiera pérdida de información clave.
2. **Vectorización:** Se aplicó el vectorizador TF-IDF (Term Frequency-Inverse Document Frequency), limitando el número de características a 1826 para mantener un balance entre complejidad computacional y capacidad representativa. Esto resultó en una matriz de características cuya dimensión se ajusta al tamaño del conjunto de datos y al número de palabras clave seleccionadas.

La salida del vectorizador (`X`) es una matriz que representa cada registro de laboratorio como un vector numérico, capturando la relevancia relativa de las palabras presentes en los textos.

### Arquitectura del Modelo

La red neuronal fue diseñada para procesar la matriz de características generada por el vectorizador:

1. **Entrada:** La matriz TF-IDF, con una dimensión basada en las características seleccionadas (1826).
2. **Capas Ocultas:** 
   - 128 neuronas y activación ReLU en la primera capa.
   - Capas adicionales con 64 y 32 neuronas, también con activación ReLU, para reducir progresivamente la dimensionalidad y extraer patrones relevantes.
3. **Capa de Salida:** Una capa con activación softmax, diseñada para clasificar los datos en las diferentes etiquetas posibles (analitos y resultados como NS1 positivo, IgM negativo, etc.).
4. **Optimización:** El modelo se entrenó utilizando el optimizador Adam y la función de pérdida `categorical_crossentropy`, adecuada para problemas de clasificación multiclase.
5. **Precaución:** Se utilizó un `Dropout` del 30%, que, de forma aleatoria, “apaga” el 30% de las neuronas entre capas para evitar overfitting.

---

## Entrenamiento y Resultados

El modelo fue entrenado con datos etiquetados manualmente de 2023 y 2024. Durante el entrenamiento:

- **Métricas de Validación:** Se monitoreó la precisión del modelo en un subconjunto de datos reservados, logrando un buen desempeño en los datos disponibles.
- **Costo Computacional:** La red neuronal resultante tiene un tamaño aproximado de 3 MB (3000 KB), lo que la hace eficiente en términos de almacenamiento y uso.

---

## Limitaciones y Áreas de Mejora

1. **Restricciones de Datos:** El modelo fue entrenado exclusivamente con datos recientes (2023-2024), lo que implica que puede ser menos preciso al procesar registros con patrones no contemplados en el entrenamiento.
2. **Acceso a Datos:** Actualmente, las columnas clave `INFORME_RESULTADO` y `DESC_PLANTILLA` no están disponibles en las vistas con las que trabaja la Dirección de Investigación en Salud del IETSI, dificultando la incorporación de nuevos datos.
3. **Adaptación a Nuevos Datos:** Existe el riesgo de que nuevas palabras o patrones textuales introducidos en los datos futuros no sean reconocidos correctamente.

---

## Propuesta de Uso y Mejoras

1. **Ejemplo de Uso:** Los datos textuales se procesan con el vectorizador TF-IDF y se ingresan al modelo para obtener predicciones automáticas. El sistema etiqueta los resultados de laboratorio y genera salidas estructuradas para análisis posteriores.
2. **Interfaz de Usuario:** Es necesario desarrollar una interfaz que facilite el uso del modelo por parte de usuarios no técnicos.
3. **Reentrenamiento Regular:** Incorporar datos nuevos y diversificados de forma periódica para mantener y mejorar el desempeño del modelo.

A pesar de las limitantes, el modelo representa un avance significativo en la automatización del análisis de datos de laboratorio, con un impacto positivo en la eficiencia y precisión del etiquetado.
