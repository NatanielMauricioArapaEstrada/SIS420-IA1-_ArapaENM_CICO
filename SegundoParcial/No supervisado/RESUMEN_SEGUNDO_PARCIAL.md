# Resumen del segundo parcial

**Objetivo:** clasificar recortes de suelo en dos clases: **0 = limpio** y **1 = con basura**.

## 1. `segundo parcial no supervisado.ipynb`: tratamiento del dataset

- **Revisión inicial:** se conservaron **177 fotografías** tras eliminar manualmente las selfies. Se comprobaron sus dimensiones, con referencia 4096 × 3072 píxeles, y se corrigieron **15 verticales** mediante giros manuales de 90°.
- **Conservar el suelo:** se eliminó el 40% superior y se conservó el **60% inferior con todo el ancho**, para reducir cielo y elementos ajenos al suelo. Se verificó mostrando una imagen original junto a su recorte.
- **Crear imágenes pequeñas:** sobre esa zona se distribuyeron **40 columnas × 25 filas**: **1.000 recortes de 128 × 128 píxeles por foto**. Se extraen directamente de la fotografía, con solapamiento parcial: algunos vecinos comparten píxeles.
- **Guardar el origen:** se generaron **177.000 PNG** en `dataset ya tratado`, con una subcarpeta por fotografía para identificar el origen de cada recorte.
- **Reutilizar el procesamiento:** si esa carpeta existe, se cargan sus archivos sin repetir los recortes. Se mantienen las comprobaciones con miniaturas y se liberan las fotos grandes para ahorrar memoria.

## 2. `eleccion de k.ipynb`: distribución de datos y agrupamiento

La separación se hizo **por fotografía original**: todos sus recortes permanecen en un solo conjunto.

- **Entrenamiento:** 157 fotos y 157.000 recortes disponibles. Se seleccionaron **45.000 sin repetición**, con semilla 42, para limitar RAM y tiempo. Sus rutas y orden quedaron guardados.
- **Testeo:** 20 fotos reservadas, tomando **5 recortes por foto: 100 imágenes** etiquetadas manualmente, excluidas de K-Means, propagación y entrenamiento.

Se conservaron los tres canales **RGB**. Cada recorte se aplanó en **128 × 128 × 3 = 49.152 valores**, usando `float32` y dividiendo entre **255** para llevarlos a **0–1**, conservando el color.

Se compararon K=1 a 10 y K=50 con codo, silueta e inspección visual. Se eligió **K=50 por la utilidad de sus grupos para etiquetar**, aunque no tuvo la mejor silueta. Las clases finales siguen siendo dos.

## 3. Regreso al cuadernillo principal

Se etiquetó una imagen de distancia intermedia por grupo. Se usó **regresión logística binaria con sigmoide en PyTorch/CUDA**: primero con 50 etiquetas manuales y luego con las etiquetas propagadas a las 45.000 imágenes.

Después se realizaron **dos rondas de aprendizaje activo de 100 correcciones**, acumulando las etiquetas humanas y guardando cada modelo. Se compararon exactitud, precisión, recall, F1 y matrices de confusión sobre el mismo testeo.

**Resultado guardado:** tras propagación y ambas rondas activas, la exactitud fue 64% y el recall de basura 5,88%; las rondas no mejoraron esas métricas de testeo.
