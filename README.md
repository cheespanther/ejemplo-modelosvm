## SUPORT VECTOR MODEL EXERCISE

---
title: "Ejercicio modelado SVM con datos IRIS"
author: "Iskar Waluyo (Dr.  Bharatendra Rai)"
date: "21/3/2020"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Cargar datos y librerías

```{r}
data(iris) # cargar data "iris" de R
library(ggplot2) # librería para gráficar
library(e1071) # librería para el modelo SVM
```

## Plot de los datos precargados

Plot de los datos "iris" incluidos en la paquetería. 

```{r}
qplot(Petal.Length, Petal.Width, data = iris, color = Species)

```

## Modelo de predicción SVM

El modelo Suppot Vector Machine (SVM) particiona los datos utilzando los márgenes más amplios psibles. Dado que pocos fenómenos tienen un comportamiento lineal se aplicó el "truco de kernel" al utilizar un función Gaussian Radial Basis (RBF) que tiene dos hiperparametros que controlan la flexibilidad del clasificador. 

Explicación "de mortales"
El modelo trata de clasificar los datos con base a lineas que separan los datos. Cuando no es posible, los datos se re-proyectan en una dimensión mayor, hasta poder separarlos linealmente.

El modelo tiene algunos parametros. 

1. Tipo de método de muestreo "kernel" - linear, radial, sigmoid
2. costo
3. epsilon

En este caso, se crea un modelo con cada uno de los distintos tipos de de muestreo kernel. Antes, se hará una demostración con el modelo que elija la paquetería. El modelo usa los datos de "iris" para separar los eventos según la especie con una línea. Usa todos los datos, menos el de "Sepecies" para modelar y clasificar.

El modelo por default escoge ciertos parametros que se pueden ver en el summmary. 

```{r}

#El comando SVM es parte de la paquetería "e1071"
modelo_svm_species <- svm(Species~., data = iris)
summary(modelo_svm_species)
```

Al graficar el modelo se puede observar como se calcularon 2 "líneas" que seperan los datos por, en este caso, "Species". 

```{r}
#El resumen del modelo muestra cuales fueron los parametros que se utilizaron por default
plot(modelo_svm_species, data=iris, Petal.Width~Petal.Length,
     slice = list(Sepal.Width = 3, Sepal.Length = 4))
```

### Prueba de capacidad de predicción del modelo

El modelo se puede probar con los datos de "iris" y generando una matriz de confusión para evaluar la efectividad del modelo para predecir la especie en función de los otros parámetros de los datos. 

```{r}

# Aplicación del modelo a los datos de "iris"
prediccion_svm <- predict(modelo_svm_species, iris)

# Matriz de confusión de la predicción de especies del modelo
table(Valor_predicho = prediccion_svm, valor_real = iris$Species)

```

La matriz de confusión anterior muestra que el modelo tuvo dos errores al identificar 2 versicolor como virginica y 2 virginica como versicolor (error de "falso positivo").

## Comparación de los diferentes tipos de muestreo para el kernel

### Modelo con muestreo tipo lineal

```{r}
modelo_svm_species_lineal <- svm(Species~., data = iris,
                          kernel = "linear")
summary(modelo_svm_species_lineal)

plot(modelo_svm_species_lineal, data=iris, Petal.Width~Petal.Length,
     slice = list(Sepal.Width = 3, Sepal.Length = 4))

prediccion_lineal <- predict(modelo_svm_species_lineal, iris)
# Matriz de confusión
table(Prediccion = prediccion_lineal, Real = iris$Species)

```

### Modelo con muestreo tipo sigmoid
```{r}
modelo_svm_species_sigmoid <- svm(Species~., data = iris,
                          kernel = "sigmoid")
summary(modelo_svm_species_sigmoid)

plot(modelo_svm_species_sigmoid, data=iris, Petal.Width~Petal.Length,
     slice = list(Sepal.Width = 3, Sepal.Length = 4))

prediccion_sigmoid <- predict(modelo_svm_species_sigmoid, iris)
table(Prediccion = prediccion_sigmoid, Real = iris$Species)
```

### Modelo con muestreo tipo polinomial
```{r}
modelo_svm_species_polinomial <- svm(Species~., data = iris,
                          kernel = "polynomial")
summary(modelo_svm_species_polinomial)

plot(modelo_svm_species_polinomial, data=iris, Petal.Width~Petal.Length,
     slice = list(Sepal.Width = 3, Sepal.Length = 4))

prediccion_polinomial <- predict(modelo_svm_species_polinomial, iris)
table(Prediccion = prediccion_polinomial, Real = iris$Species)

```

## Optimización de parametros
Como se mencionó anteriormente, el modelo SVM ocupa tres parámetros 1) el tipo de muestreo de kernel 2) el costo y 3) epsilon. Éstos se pueden ajustar para obtener la menor cantida de errores posible. La optimización se puede realizar utilizando la herramienta de "tune" de la paquetería. 

### parametros epsilon y costo

Epsilon - 

Costo - 

### Optimización con "tune"

De las pruebas anteriores se encontró que el modelo que mejor resultados fue el "radial" que en este caso es el modelo default. Por tal motivo, la optimización se realiza con ese modelo utilizando la herramietna "tune" de la paquetería "e1071"

Se establece un punto de inicio de 123 para iniciar las iteraciones. Y se va a correr el modelo con números para epsilon de 0 a 1 con incrementos de 0.1 (epsilon = seq(0,1,0.1)) y se probará con costos de 4 a 512 (cost = 2^(2:9)). La optimización se realiza de la siguiente manera: 

```{r}
set.seed(123)
modelo_opt <- tune(svm, Species~., data = iris, 
     ranges = list(epsilon = seq(0,1,0.1),
                   cost = 2^(2:9)))
summary(modelo_opt)
```

Al graficar la optimización del modelo, las zonas más obscuras denotan valores para los parámetros que dan mejores resultados. En la siguiente gráfica se puede observar que los mejores resultados para el modelo se obtienen cuando "cost" es menor a 200. Por lo tanto, se volverá a ajustar el model utilizando cost = 2^(2:7)


```{r}
plot(modelo_opt) #mejores resultados en los más oscuro
```

Se corre el mísmo código que en el paso anterior pero con el rengo de costos: cost = 2^(2:7)

```{r}
set.seed(123)
modelo_opt <- tune(svm, Species~., data = iris,
     ranges = list(epsilon = seq(0,1,0.1),
                   cost = 2^(2:7)))
plot(modelo_opt) #mejores resultados en los más oscuro
```

## Correr el modelo con los valores de la optimización

Del summary de la optimización, se encontró que los mejores valores para los parámetros son: 

cost: 8 y epsilon: 0

Se corre el modelo con clasificación tipo C, muestreo tipo radial, costo 8 y epsilon 0.

Estos parametros se pueden extraer directo del modelo utilizando modelo_opt$best.model

```{r}
modelo_opt$best.model
```

```{r}
modelo_optimo <- modelo_opt$best.model

plot(modelo_optimo, data=iris, Petal.Width~Petal.Length,
     slice = list(Sepal.Width = 3, Sepal.Length = 4))

```

## Preguntas

¿Cuál es el error de clasificación de cada uno de los modelos? 

¿Por qué rindió mejores resultados el modelo "radial" que los demás? 
