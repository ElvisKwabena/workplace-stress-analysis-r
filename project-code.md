
---
title: "Modelling Workplace Stress and Health Behaviors Using R"
---

## **1. Introduction**

This report analyzes how work-related and demographic factors influence stress levels, health behaviors, and healthcare utilization among adults.
The dataset (`Work.csv`) contains 1,200 observations with variables on age, income, hours worked, gender, stress levels, smoking status, education, and job sector.

---

## **2. Importing and Exploring the Data**

```{r load-data}
# Load necessary libraries
library(dplyr)
library(ggplot2)

# Import data
data <- read.csv("Work.csv")

# Examine structure
str(data)
```

**Interpretation:**
The dataset consists of 12 variables and 1,200 observations. Variable types include numeric (age, income, hours worked), categorical (gender, education), and binary (smoker, high_stress).

---

## **3. Descriptive Statistics**

### (a) Continuous Variable – Age

```{r descriptive-age}
summary(data$age)
```

**Comment:**
The ages range from 2.3 to 72.8 years, with a mean of 34.8. Most respondents fall between 27.7 and 41.5 years, reflecting a working-age population.

---

### (b) Continuous Variable – Hours Worked

```{r descriptive-hours}
summary(data$hours_worked)
```

**Comment:**
Hours worked per week range from 20.6 to 54.2, with a mean of 40.05 hours, indicating a largely full-time workforce.

---

### (c) Categorical Variable – Gender

```{r gender-summary}
table(data$gender)
```

**Comment:**
The dataset includes 610 females and 590 males, showing a balanced gender distribution suitable for comparative analysis.

---

## **4. Random Sampling**

```{r random-sample}
set.seed(123)
sample_data <- data[sample(nrow(data), 250), ]
head(sample_data, 10)
```

**Comment:**
A random subset of 250 observations was selected for reproducible exploratory analysis.

---

## **5. Gender and Income Comparison**

### (a) Visual Comparison

```{r boxplot-income-gender}
boxplot(income ~ gender, data = data,
        main = "Income Distribution by Gender",
        xlab = "Gender", ylab = "Income",
        col = c("pink", "lightblue"))
```

### (b) Hypothesis Testing – t-Test

**Assumptions:**

1. Gender groups are independent
2. Income is continuous and approximately normal
3. Unequal variances allowed (Welch’s t-test)

```{r t-test-income}
# Normality check using Shapiro-Wilk
shapiro.test(data$income[data$gender == "Male"])
shapiro.test(data$income[data$gender == "Female"])

# Variance test
var.test(income ~ gender, data = data)

# Welch Two Sample t-test
t.test(income ~ gender, data = data, var.equal = FALSE)
```

**Interpretation:**
Although variance between groups differs significantly (p = 0.0167), the t-test shows **no significant difference in mean income** between males and females (p = 0.2772).

---

## **6. Predicting High Stress (Logistic Regression)**

### (a) Model Fitting

```{r logistic-model}
data$high_stress_bin <- ifelse(data$high_stress == "Yes", 1, 0)
data$smoker_bin <- ifelse(data$smoker == "Yes", 1, 0)

model_logit <- glm(high_stress_bin ~ age + income + hours_worked + smoker_bin,
                   data = data, family = binomial)

summary(model_logit)
```

**Interpretation:**

* **Hours worked (p < 0.001)** and **smoking status (p = 0.003)** are significant predictors of high stress.
* Age and income are not statistically significant.
* The model’s AIC = 1574.7, suggesting good fit for a simple model.

---

### (b) Model Evaluation

```{r confusion-matrix}
pred_probs <- predict(model_logit, type = "response")
pred_class <- ifelse(pred_probs > 0.5, 1, 0)
table(Predicted = pred_class, Actual = data$high_stress_bin)
```

**Confusion Matrix Interpretation:**
The model correctly classifies **(530 + 205) / 1200 = 61.25%** of observations.
Most false negatives occur among moderately stressed individuals.

---

## **7. Predicting Sick Days (Linear Regression)**

```{r linear-model}
model_lm <- lm(sick_days ~ age + income + stress_level + education + smoker, data = data)
summary(model_lm)
```

**Key Predictors:**

| Predictor    | Coefficient | p-value | Interpretation                                              |
| ------------ | ----------- | ------- | ----------------------------------------------------------- |
| stress_level | 0.0118      | < 0.001 | Each 1-unit increase in stress increases sick days by 0.012 |
| smokerYes    | 0.2182      | < 0.001 | Smokers take ~0.22 more sick days per year than non-smokers |

---

## **8. Conclusions**

* Higher stress levels and smoking status significantly predict greater sick days.
* Hours worked and smoking behavior drive higher stress probability.
* Gender differences in income are statistically insignificant in this sample.
* Findings suggest that **behavioral health interventions** targeting overwork and smoking may improve workplace wellness.

---

## **9. Skills Demonstrated**

* R Programming
* Data Wrangling and Visualization
* Statistical Inference (t-tests, ANOVA, regression)
* Logistic and Linear Modeling
* Model Evaluation (AIC, Confusion Matrix)

---

```{r session-info, echo=FALSE}
sessionInfo()
```

---

```


