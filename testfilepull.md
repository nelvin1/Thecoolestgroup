---
title: "PushPull testing"
author: "Nelvin Vincent"
date: "10/06/2022"
output:
  html_document:
    toc: true
    theme: readable
    highlight: tango
    code_folding: show
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Downloading and Prepping the Data

```{r}
#Downloading and Prepping the Data
tele <- read.csv("tele.csv", stringsAsFactors = TRUE)
summary(tele)

#We are deleting the "duration" variable because it is an after the fact measurement. We only should be using variables that we know before the call
tele$duration <- NULL

# Deleting the column X
tele$X <- NULL

# Changing pdays to a dummy and deleting pdays
tele$pdaysdummy <- ifelse(tele$pdays == 999, 0, 1)
tele$pdays <- NULL

str(tele)
```

## Getting Data Ready for Analysis

```{r}
# Using model.matrix to convert all the factors to dummy variables
# We are converting all of the factors into dummy variables as the input into knn has to be numeric

telemm <- as.data.frame(model.matrix(~.,tele))
str(telemm)

# Randomize the rows in the data (shuffling the rows)
set.seed(12345)
tele_random <- telemm[sample(nrow(telemm)),]

#Normalize the data
normalize <- function(x) {
  return ((x - min(x)) / (max(x) - min(x)))
}

# we are going to normalize everything 
tele_norm <- as.data.frame(lapply(tele_random, normalize))
summary(tele_norm)
```


> Now you are ready to build your ANN model. Feel free to modify the data load, cleaning and preparation code above as per your preference.


```{r,cache=TRUE}
library(neuralnet)

tele_norm$X.Intercept. <- NULL
tele_train <- tele_norm[1:8238,]
tele_test <- tele_norm[8239:41188,]
tele_model <- neuralnet(formula = yyes ~ .,
                              data = tele_train, hidden=c(5,3),linear.output = TRUE)
plot(tele_model)
model_results<-compute(tele_model,tele_test[1:52])
summary(model_results)
```

