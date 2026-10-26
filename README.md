# 📈 Multiple Linear Regression: Study Notes

> **Course:** DATA 603, Statistical Modelling with Data
> **Topic:** Lecture 1, Multiple Linear Regression (Part 1)
> **Language:** R
> **Status:** Concepts complete (Steps 1–8). Practice problems in progress.

These are my study notes from working through the first lecture of DATA 603. They are written in my own words, with worked examples, R code, a formula sheet, common pitfalls, and self-check questions with answers.

---

## 📑 Table of Contents

1. [Review of Simple Linear Regression](#1-review-of-simple-linear-regression)
2. [Why We Need Multiple Predictors](#2-why-we-need-multiple-predictors)
3. [The Multiple Regression Model and Least Squares](#3-the-multiple-regression-model-and-least-squares)
4. [Interpreting Coefficients and Confidence Intervals](#4-interpreting-coefficients-and-confidence-intervals)
5. [The Global F-Test (ANOVA)](#5-the-global-f-test-anova)
6. [Which Predictors Matter? t-Tests and the Partial F-Test](#6-which-predictors-matter-t-tests-and-the-partial-f-test)
7. [Model Fit: R², Adjusted R², RSE](#7-model-fit-r-adjusted-r-rse)
8. [Prediction](#8-prediction)
9. [The Complete Modelling Workflow](#9-the-complete-modelling-workflow)
10. [R Cheat Sheet](#10-r-cheat-sheet)
11. [Formula Sheet](#11-formula-sheet)
12. [Common Pitfalls](#12-common-pitfalls)
13. [Corrections to the Lecture Notes](#13-corrections-to-the-lecture-notes)
14. [Self-Check Questions (with Answers)](#14-self-check-questions-with-answers)
15. [Datasets and References](#15-datasets-and-references)

---

## 1. Review of Simple Linear Regression

### The setup
We have pairs of data $(x_i, y_i)$ for $i = 1, \dots, n$ and want to describe how $Y$ changes with $X$ using a straight line.

| Term | Also called | Toddler example |
|---|---|---|
| $X$ | predictor, independent, explanatory variable | Age (months) |
| $Y$ | response, dependent variable | Height (cm) |

### Three versions of "the model": keep them separate

| Expression | What it is |
|---|---|
| $Y = \beta_0 + \beta_1 X + \epsilon$ | **Population model.** Describes each individual observation, including random scatter. The β's are **true but unknown**. |
| $E(Y) = \beta_0 + \beta_1 X$ | **Regression equation.** The **average** Y at a given X. ε disappears because $E(\epsilon) = 0$. |
| $\hat{Y} = \hat\beta_0 + \hat\beta_1 X$ | **Estimated model.** The $\hat\beta$'s are **calculated from our sample** by least squares. |

- $\beta_0$ is the intercept: the value of Y when X = 0.
- $\beta_1$ is the slope: the change in Y per 1-unit increase in X.
- $\epsilon$ is the error: everything affecting Y that isn't X.

### Worked example: toddler height

Fitting height on age for 12 toddlers gives approximately:

$$\widehat{Height} \approx 60.25 + 0.85 \times Age$$

- **Interpretation:** for each additional month of age, predicted (average) height increases by about **0.85 cm**.
- **Warning:** the ages in the data run from **18 to 29 months**. Predicting for a 6-month-old would be **extrapolation**, and the 60.25 intercept (age 0) has no practical meaning.

### The three Sums of Squares

| Name | Formula | Measures |
|---|---|---|
| **SST** (Total) | $\sum (y_i - \bar{y})^2$ | Total variation of the data around the **mean** |
| **SSR** (Regression) | $\sum (\hat{y}_i - \bar{y})^2$ | Variation **explained** by the line |
| **SSE** (Error) | $\sum (y_i - \hat{y}_i)^2$ | Variation **left unexplained** (residuals) |

$$SST = SSR + SSE$$

> 💡 **BI analogy:** if your only "model" were `AVG(Height)`, SST is how wrong that flat prediction would be. The regression line removes part of that error (SSR), and what's left is SSE.

### R², the coefficient of determination

$$R^2 = \frac{SSR}{SST} = 1 - \frac{SSE}{SST}$$

**R² is the proportion of the variation in Y that is explained by the model.**

- Correct: *"75% of the variation in height is explained by age."*
- Incorrect: *"75% of the model is explained by the variation in age."* The thing being explained is always **Y**.

> ⚠️ R² is **not** the correlation coefficient. The correlation is **r** (between −1 and 1). In simple regression only, $R^2 = r^2$.

### MSE, the estimate of the error variance

$$MSE = \frac{SSE}{n - 2}$$

We divide by $n - 2$ because **two parameters** ($\beta_0, \beta_1$) were estimated, and each uses one degree of freedom. This makes MSE an **unbiased** estimator of $\text{Var}(\epsilon)$. With p predictors, the general rule is $n - p - 1$.

---

## 2. Why We Need Multiple Predictors

### The Advertising dataset
- **200 markets**
- **Response:** `sale`, sales of a product (thousands of units)
- **Predictors:** `tv`, `radio`, `newspaper`, advertising budgets (thousands of dollars)

### First attempt: three separate simple regressions

| Model | Estimated equation | r with sales | F (df = 1, 198) | p-value | 95% CI for slope |
|---|---|---|---|---|---|
| TV | $7.03 + 0.0475X_{TV}$ | 0.78 | 312.1 | ≈ 0 | (0.0422, 0.0528) |
| Radio | $9.31 + 0.2025X_{radio}$ | 0.58 | 98.42 | ≈ 0 | (0.1622, 0.2427) |
| Newspaper | $12.35 + 0.0547X_{news}$ | 0.23 | 10.89 | 0.001 | (0.0220, 0.0874) |

The hypotheses for each model:

$$H_0: \beta_1 = 0 \text{ (no linear relationship)} \qquad H_a: \beta_1 \neq 0$$

Worked F for newspaper:

$$F = \frac{MSR}{MSE} = \frac{282.34 / 1}{5134.805 / 198} = 10.89$$

- **df = (1, 198):** 1 predictor in the numerator; $n - 2 = 198$ in the denominator.
- On their own, **all three media look significant.**

### Interpreting with units
TV slope = 0.0475 means: *every additional **$1,000** on TV is **associated with** about **47.5 more units sold**, on average.*

> ⚠️ Say **"associated with,"** not **"causes."** This is observational data, so regression shows association, not causation.

### The problem
Each simple model **ignores the other media**. Markets that spend more on radio also tend to spend more on newspaper; the two predictors are **correlated**. So the newspaper-only model gives newspaper **credit for radio's effect**: newspaper acts as a **stand-in** for radio. Multiple regression fixes this by estimating each effect **while holding the others constant**.

---

## 3. The Multiple Regression Model and Least Squares

### The model

$$Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \dots + \beta_p X_p + \epsilon = \mu + \epsilon$$

- $p$ = the number of predictors
- $\mu = \beta_0 + \beta_1 X_1 + \dots + \beta_p X_p$ is the **deterministic part**: the predictable average.

**First-order model:** each quantitative predictor appears **on its own, to the power 1**. There are no $X^2$ terms and no $X_1 X_2$ products.
- $Y = \beta_0 + \beta_1 TV + \beta_2 TV^2$ is **not** first-order, because $TV^2$ is a second-order term. The number of predictors has nothing to do with it.

### Why it's harder than simple regression
1. **Model selection:** many possible subsets of predictors.
2. **Visualization:** you can't plot beyond 3D.
3. **Overfitting:** more predictors ≠ better; complex models fit noise.
4. **Computation:** needs matrix algebra (R does it).

### Geometry
- 1 predictor: least squares fits a **line**
- 2 predictors: it fits a **plane**
- 3 or more: it fits a **hyperplane**

In every case, least squares minimizes the squared **vertical** distances from the points to the fitted surface.

### Least squares estimation

Choose the $\hat\beta$'s that **minimize SSE**:

$$SSE = \sum_{i=1}^{n}\left(y_i - (\hat\beta_0 + \hat\beta_1 x_{1i} + \dots + \hat\beta_p x_{pi})\right)^2$$

The solution in matrix form:

$$\hat{\beta} = (X'X)^{-1}X'y$$

- **y:** a column of all **n** response values $(y_1, \dots, y_n)$
- **X:** the **design matrix**, with dimensions **n × (p + 1)**: one row per observation and one column per parameter

$$X = \begin{pmatrix}1 & x_{11} & \dots & x_{1p}\\ 1 & x_{21} & \dots & x_{2p}\\ \vdots & \vdots & & \vdots \\ 1 & x_{n1} & \dots & x_{np}\end{pmatrix}$$

| Object | Advertising dimensions |
|---|---|
| X | **200 × 4** (columns: 1, TV, radio, newspaper) |
| X′X | **4 × 4** |
| $\hat\beta$ | **4 × 1** |

- **Rows = observations (n). Columns = parameters (p + 1).**
- **The column of 1s produces the intercept.** Without it, the plane is forced through the origin (Y = 0 when all X = 0), which also distorts the slopes.

Verifying the formula in R:

```r
X <- model.matrix(sale ~ tv + radio + newspaper, data = Advertising)
y <- Advertising$sale
solve(t(X) %*% X) %*% t(X) %*% y   # same as coef(lm(...))
```

### The Advertising result

$$\widehat{Sale} = 2.939 + 0.0458X_{TV} + 0.1885X_{radio} - 0.0010X_{newspaper}$$

| Predictor | Slope alone | Slope in the multiple model |
|---|---|---|
| TV | 0.0475 | 0.0458 |
| Radio | 0.2025 | 0.1885 |
| Newspaper | 0.0547 | **−0.0010** |

TV barely changes because it is nearly **uncorrelated** with the other media. Newspaper **collapses** because it is correlated with radio; once radio is held constant, newspaper has nothing left to explain.

---

## 4. Interpreting Coefficients and Confidence Intervals

### Interpretation rules
- **Intercept $\hat\beta_0$:** the predicted Y when **all** predictors equal 0. It is only meaningful if 0 is realistic and inside the data range.
- **Slope $\hat\beta_i$:** the change in predicted Y for a 1-unit increase in $X_i$, **holding all other predictors constant**.

> 🔑 **Always include "holding the other predictors constant"** in a multiple regression interpretation.

**Advertising interpretations:**
- **TV (0.0458):** for a given radio and newspaper budget, an extra $1,000 on TV is associated with about **46 more units sold**, on average.
- **Radio (0.1885):** for a given TV and newspaper budget, an extra $1,000 on radio is associated with about **189 more units sold**, on average.
- **Newspaper (−0.0010):** about **1 fewer unit sold** per extra $1,000, which is effectively zero.

### Two goals of regression
1. **Association:** which predictors relate to Y, and how strongly?
2. **Prediction:** estimating Y for new data.

### Confidence intervals for coefficients

$$\hat\beta_i \pm t_{\alpha/2,\; n-p-1} \times S_{\hat\beta_i}$$

- $S_{\hat\beta_i}$ is the standard error, from the "Std. Error" column of `summary()`.
- We use a **t-distribution** because the error variance is **estimated**, not known.
- $t_{\alpha/2}$ is the **upper** critical value (positive).

**Worked example, TV (95%):**

- df = 200 − 3 − 1 = 196, so $t_{0.025,196} \approx 1.972$
- $0.045765 \pm 1.972 \times 0.001395 = 0.045765 \pm 0.00275$
- **CI: (0.0430, 0.0485)**

*"We are 95% confident that each extra $1,000 on TV is associated with an increase of 43 to 48.5 units sold, holding radio and newspaper constant."*

**What "95% confident" means:** if we repeated the sampling many times, about 95% of the intervals built this way would contain the true β. It does **not** mean there is a 95% chance that β is in this particular interval.

### Higher confidence gives wider intervals

| Level | t (df = 196) | TV interval |
|---|---|---|
| 95% | 1.97 | (0.0430, 0.0485) |
| 99% | 2.60 | (0.0421, 0.0494) |

A larger critical value multiplies the same standard error, so the interval widens. **More confidence costs precision.**

### CIs and hypothesis tests always agree
If **0 is not in the 95% CI**, then we **reject $H_0: \beta_i = 0$ at α = 0.05**. Newspaper's CI is **(−0.0126, 0.0105)**, which contains 0, so its effect is not distinguishable from zero.

> ⚠️ Don't drop a variable on one piece of evidence. Confirm with a t-test or partial F-test, compare adjusted R² and RSE, and consider whether there's a domain reason to keep it.

---

## 5. The Global F-Test (ANOVA)

### Vocabulary
| Term | Meaning |
|---|---|
| **p-value** | Probability of results at least this extreme **if H₀ were true** |
| **α** (significance level) | P(Type I error): rejecting H₀ when it's actually true (false alarm). Usually 0.05. |
| **Type II error** | Failing to reject H₀ when it's actually false (missed effect) |
| **Decision rule** | Reject H₀ if **p-value < α** |

> ⚠️ The notes use **β** for the Type II error probability. That's a different β from the regression coefficients.

### Hypotheses

$$H_0: \beta_1 = \beta_2 = \dots = \beta_p = 0 \qquad H_a: \text{at least one } \beta_i \neq 0$$

H₀ says the model is **no better than predicting the mean**: the **null model**, `lm(y ~ 1)`, for which SSR = 0 and SSE = SST.

### Why test the whole model first?
Running many separate t-tests **inflates the false-positive rate**: about 5 "significant" results per 100 useless predictors at α = 0.05. The global F-test is **one** test, so the overall Type I error rate stays at α.

### The ANOVA table

| Source | SS | df | Mean Square | F |
|---|---|---|---|---|
| Regression | SSR | $p$ | $MSR = SSR/p$ | $MSR/MSE$ |
| Error | SSE | $n - p - 1$ | $MSE = SSE/(n-p-1)$ | |
| Total | SST | $n - 1$ | | |

$$F_{obs} = \frac{SSR/p}{SSE/(n-p-1)} \sim F_{p,\; n-p-1}$$

Both the SS column and the df column add up.

**Intuition:** if the predictors are useless, $MSR \approx MSE$ and $F \approx 1$. A large F means the model explains far more than chance would.

### Advertising, worked from `aov()`

| Source | SS |
|---|---|
| TV | 3314.618 |
| Radio | 1545.617 |
| Newspaper | 0.089 |
| Residuals | 556.824 |

- SSR = 3314.618 + 1545.617 + 0.089 = **4860.324**
- MSR = 4860.324 / 3 = **1620.11**
- MSE = 556.824 / 196 = **2.841**
- **F = 570.3**, df = (3, 196), p ≈ 0, so **reject H₀**
- SST = 5417.149, and R² = 4860.324 / 5417.149 = **0.897**

### Sequential SS in `aov()`
`aov()` assigns SS **sequentially**. The first predictor gets credit for everything it explains alone, and each later predictor gets only the **extra** it adds.
- Newspaper listed **last** gets 0.089.
- Newspaper listed **first** would get **282.34**, its simple-regression SSR.
- **Only the total SSR stays the same** when the order changes.

### If the global F-test fails
**Stop.** Don't search the individual t-tests for something "significant." Rethink the predictors or the form of the model.

### If it succeeds
At least one predictor matters, but the test doesn't say which. Go on to Section 6.

---

## 6. Which Predictors Matter? t-Tests and the Partial F-Test

### A. The individual t-test

$$H_0: \beta_i = 0 \text{ (given the other predictors are in the model)} \qquad H_a: \beta_i \neq 0$$

$$T_{obs} = \frac{\hat\beta_i - 0}{S_{\hat\beta_i}} \sim t_{n-p-1}$$

The p-value is two-sided: $2 \times P(T \geq |t_{obs}|)$. R reports it as `Pr(>|t|)`.

**Advertising (df = 196):**

| Predictor | Estimate | Std. Error | t | p | Decision |
|---|---|---|---|---|---|
| TV | 0.045765 | 0.001395 | 32.81 | ≈ 0 | Keep |
| Radio | 0.188530 | 0.008611 | 21.89 | ≈ 0 | Keep |
| Newspaper | −0.001037 | 0.005871 | −0.18 | 0.86 | Candidate to drop |

> 🔑 t-tests in `summary()` are **not order-dependent**: each predictor is tested **given all the others**. Sequential SS in `aov()` **are** order-dependent.

### B. The partial F-test

Tests whether a **subset of q predictors** adds anything **given the others are already in the model**.

- **Full model:** all predictors
- **Reduced model:** the full model **minus** the q predictors being tested

> 🔑 **The predictors you're testing are the ones *missing* from the reduced model.**

$$H_0: \text{the } q \text{ tested } \beta\text{'s} = 0 \qquad H_a: \text{at least one of them} \neq 0$$

$$F_{obs} = \frac{(SSE_{reduced} - SSE_{full}) \,/\, (df_{reduced} - df_{full})}{SSE_{full} \,/\, df_{full}} \sim F_{q,\; df_{full}}$$

Here df means the **error** df: n − (number of predictors) − 1.

**Intuition:** dropping predictors **always** increases SSE. The question is whether it increases **more than chance would explain**.

**Advertising: test newspaper**

```r
reduced <- lm(sale ~ tv + radio, data = Advertising)
full    <- lm(sale ~ tv + radio + newspaper, data = Advertising)
anova(reduced, full)   # reduced model first
```

$$F = \frac{(556.9140 - 556.8253)/(197 - 196)}{556.8253/196} = \frac{0.0887}{2.841} = 0.031, \quad df = (1, 196), \; p = 0.86$$

We fail to reject H₀, so we **drop newspaper**.

**Connections:**
1. The SSE increase (0.0887) equals newspaper's **sequential SS** (0.089) from `aov()`, because newspaper was entered last.
2. For a **single** predictor, **F = t²**: $(-0.177)^2 \approx 0.031$. Same test, same p-value.
3. The partial F-test is especially useful for **q > 1**. For example, to test radio and newspaper **together** beyond TV:

```r
anova(lm(sale ~ tv, data = Advertising),
      lm(sale ~ tv + radio + newspaper, data = Advertising))   # q = 2
```

**Worked example:** n = 40. Full model has 5 predictors with SSE = 120; reduced model has 3 predictors with SSE = 180.
- $df_{full} = 40 - 5 - 1 = 34$ and $df_{reduced} = 40 - 3 - 1 = 36$, so q = 2
- $F = \frac{(180 - 120)/2}{120/34} = \frac{30}{3.53} = 8.5$, with df = **(2, 34)**
- The 5% cutoff is about 4.1, and 8.5 > 4.1, so **reject H₀**: the two dropped predictors matter.

### The final Advertising model

$$\widehat{Sale} = 2.921 + 0.04575X_{TV} + 0.18799X_{radio}$$

---

## 7. Model Fit: R², Adjusted R², RSE

### R²

$$R^2 = \frac{SSR}{SST} = 1 - \frac{SSE}{SST} = \text{Cor}(Y, \hat{Y})^2$$

> ⚠️ **R² can never decrease when a predictor is added**, even a useless one. It has **no penalty for complexity**, so it will always favour the bigger model. **Don't use plain R² to compare models with different numbers of predictors.**

### Adjusted R²

$$R^2_{adj} = 1 - \frac{SSE/(n-p-1)}{SST/(n-1)} = 1 - \frac{MSE}{SST/(n-1)}$$

- Penalizes each extra predictor through its degrees of freedom.
- **Can go down** when a useless predictor is added.
- **Always ≤ R².** If your adjusted R² comes out higher than R², recheck the arithmetic.

### RSE, the residual standard error

$$s = RSE = \sqrt{\frac{SSE}{n-p-1}} = \sqrt{MSE}$$

- The **typical size of a prediction error**, in the **same units as Y**.
- **Lower is better.**

### Advertising comparison
With $SST/(n-1) = 5417.149/199 = 27.222$:

| Model | SSE | df | MSE | R² | R²adj | RSE |
|---|---|---|---|---|---|---|
| Full (TV, radio, newspaper) | 556.825 | 196 | 2.841 | 0.89721 | 0.8956 | 1.686 |
| Reduced (TV, radio) | 556.914 | 197 | 2.827 | 0.89719 | **0.8962** | **1.681** |

- Plain R² favours the full model, but only by **0.00002**.
- Adjusted R² and RSE both favour the **reduced** model.
- **RSE interpretation:** *"Using TV and radio, our sales predictions are typically off by about 1,700 units."*

### Worked example
n = 30, p = 4, SSE = 250, SST = 850:
- MSE = 250 / 25 = **10**
- R² = 600 / 850 = **0.706**
- R²adj = 1 − 10 / (850/29) = 1 − 10 / 29.31 = **0.659**
- RSE = √10 = **3.16**

### Two useful facts
1. **Adjusted R² and RSE always agree on the same dataset.** Both are built from MSE, and SST is fixed.
2. **Dropping a predictor raises adjusted R² exactly when its |t| < 1** (partial F < 1). Plain R² goes **down or stays the same** whenever you drop a predictor. This is a looser bar than significance (|t| ≈ 1.97), so adjusted R² sometimes keeps variables that the t-test would drop.

---

## 8. Prediction

### The point prediction
Plug the values into the fitted equation. For TV = 200 and radio = 20 ($200k and $20k):

$$\widehat{Sale} = 2.9211 + 0.04575(200) + 0.18799(20) = 15.83 \text{ (about 15,830 units)}$$

### Confidence interval vs. prediction interval

| | Confidence interval | Prediction interval |
|---|---|---|
| Question | **Average** sales of *all* markets with these budgets | Sales for **one specific** new market |
| Uncertainty included | Estimated line only | Estimated line **plus** individual scatter (ε) |
| Width | Narrower | **Wider** |
| R | `interval = "confidence"` | `interval = "predict"` |

> 💡 You can predict the **average** height of 24-month-olds far more precisely than the height of **one** 24-month-old.

```r
newdata <- data.frame(tv = 200, radio = 20)   # names must match the model
predict(reduced, newdata, interval = "predict",    level = 0.95)
predict(reduced, newdata, interval = "confidence", level = 0.95)
```

**Prediction interval: fit = 15.83, lwr = 12.50, upr = 19.16.**

*"We are 95% confident that a market spending $200,000 on TV and $20,000 on radio will sell between about 12,504 and 19,160 units."*

**A second example:** TV = 100, radio = 30 gives fit = 13.14, PI = (9.81, 16.46), or **about 9,808 to 16,465 units**.

### What drives the width
$$\hat{y} \pm t \times \sqrt{s^2 + SE(\hat{y})^2}$$

With n = 200, $SE(\hat{y})$ is small, so **RSE dominates**. As a quick check, $15.83 \pm 1.97 \times 1.681 = (12.52, 19.14)$, which almost matches R's answer.

- **If RSE doubles, the prediction interval roughly doubles in width.**

### Don't extrapolate
- Only predict **within the range of the data** (in this dataset, TV runs from about 0.7 to 296 and radio from 0 to about 50).
- TV = 400 is **outside the data**, so predicting there is extrapolation. The linear relationship may not hold; ad spending often has diminishing returns.
- Watch for **hidden extrapolation**: each value can be in range individually while the **combination** never appears in the data. Check a scatterplot of the predictors against each other.

---

## 9. The Complete Modelling Workflow

```text
1. Fit the full model with all candidate predictors
        │
2. Global F-test ── fail to reject ──► STOP (no t-test hunting)
        │ reject
3. Individual t-tests: which predictors look weak (given the others)?
        │
4. Drop weak predictors ONE AT A TIME (largest p-value first)
   └─ refit after each drop (p-values change!)
   └─ confirm each drop with a partial F-test: anova(reduced, full)
        │
5. Compare candidate models: adjusted R² (higher is better), RSE (lower is better)
   └─ never use plain R² for comparison
        │
6. Predict with the final model: prediction interval, within the data range
```

---

## 10. R Cheat Sheet

```r
# --- Load data -------------------------------------------------------------
Advertising <- read.table(
  "https://raw.githubusercontent.com/DanikaLipman/DATA603/main/Advertising.txt",
  header = TRUE, sep = "\t")

# --- Fit models ------------------------------------------------------------
full    <- lm(sale ~ tv + radio + newspaper, data = Advertising)
reduced <- lm(sale ~ tv + radio, data = Advertising)
null    <- lm(sale ~ 1, data = Advertising)        # intercept only

# --- Coefficients and inference ----------------------------------------------
coef(full)                    # estimates
summary(full)                 # t-tests, R², adj R², RSE, global F
confint(full)                 # 95% CIs
confint(full, level = 0.99)   # 99% CIs

# --- ANOVA -----------------------------------------------------------------
aov(full)                     # sequential SS (order matters!)
anova(reduced, full)          # partial F-test (reduced first)

# --- Model fit -------------------------------------------------------------
summary(full)$r.squared
summary(full)$adj.r.squared
sigma(full)                   # RSE

# --- Prediction ------------------------------------------------------------
newdata <- data.frame(tv = 200, radio = 20)
predict(reduced, newdata, interval = "predict")      # one new observation
predict(reduced, newdata, interval = "confidence")   # mean response

# --- Visualise -------------------------------------------------------------
library(ggplot2)
ggplot(Advertising, aes(x = tv, y = sale)) +
  geom_point() +
  geom_smooth(method = "lm", se = FALSE)
```

---

## 11. Formula Sheet

| Quantity | Formula | df |
|---|---|---|
| SST | $\sum (y_i - \bar y)^2$ | $n - 1$ |
| SSR | $\sum (\hat y_i - \bar y)^2$ | $p$ |
| SSE | $\sum (y_i - \hat y_i)^2$ | $n - p - 1$ |
| Identity | $SST = SSR + SSE$ | |
| MSR | $SSR / p$ | |
| MSE | $SSE / (n - p - 1)$ | |
| Global F | $MSR / MSE$ | $(p,\; n-p-1)$ |
| t-statistic | $\hat\beta_i / S_{\hat\beta_i}$ | $n - p - 1$ |
| CI for $\beta_i$ | $\hat\beta_i \pm t_{\alpha/2} S_{\hat\beta_i}$ | $n - p - 1$ |
| Partial F | $\dfrac{(SSE_{red} - SSE_{full})/q}{SSE_{full}/df_{full}}$ | $(q,\; df_{full})$ |
| R² | $SSR/SST = 1 - SSE/SST$ | |
| Adjusted R² | $1 - \dfrac{SSE/(n-p-1)}{SST/(n-1)}$ | |
| RSE | $\sqrt{MSE}$ | |
| LSE | $\hat\beta = (X'X)^{-1}X'y$ | |

---

## 12. Common Pitfalls

| ❌ Pitfall | ✅ Fix |
|---|---|
| "X **causes** Y to increase" | "X is **associated with** an increase in Y" |
| Interpreting a slope without units | Always convert: $1,000 → 46 units, and so on |
| Forgetting "holding the others constant" | Required in every multiple regression interpretation |
| "R² of 0.75 means 75% of the model is explained by X" | "75% of the variation **in Y** is explained by the model" |
| Calling R² the correlation coefficient | r is correlation; R² is the coefficient of determination |
| Design matrix has p + 1 **rows** | Rows = **n** observations; columns = p + 1 |
| Dividing SSE by the wrong df | Write $n - p - 1$ **first**, then divide |
| Adjusted R² > R² | Impossible. Recheck the arithmetic. |
| Running t-tests after a non-significant global F | Stop; the model isn't useful |
| Thinking `aov()` SS don't depend on order | They're **sequential**, so order matters (t-tests don't) |
| "Dropped predictors matter because SSE increased" | SSE **always** increases; ask whether F exceeds the critical value |
| Reduced model = only the predictors being tested | Reduced model = full model **minus** the predictors being tested |
| Writing F's df as a fraction | Write as a pair: **(q, df_full)** |
| Comparing models with plain R² | Use **adjusted R²** and **RSE** |
| Using a confidence interval for one new observation | Use a **prediction interval** |
| "OK to predict if it's in range" without checking | Apply the rule to the numbers and commit to an answer |
| Contradictory numbers (high R², F ≈ 1) | Sanity-check that results tell a consistent story |

---

## 13. Corrections to the Lecture Notes

Errata I spotted while studying:

1. **R² is labelled "Sample Correlation Coefficient."** It is the **coefficient of determination**; the correlation coefficient is r.
2. **Simple regression hypotheses:** Hₐ says "cannot be expressed" in both H₀ and Hₐ. Hₐ should say **can**.
3. **Newspaper section** prints the **radio** confidence interval, and labels both newspaper intervals as β₀. The second should be **β₁**.
4. **The y vector** is written ending in $y_p$. It should end in **$y_n$** (one entry per observation).
5. **$t_{\alpha/2}$ is called the "α/2 quantile."** It's the **upper** α/2 critical value (positive).
6. **t-test p-value** is written as $P(T \geq t)$. For a two-sided test it is **$2 \times P(T \geq |t|)$**.
7. **Prediction example** is described as a "95% confidence interval," but `interval = "predict"` gives a **prediction interval**.
8. **Toddler plot subtitle** says "Height and Weight." The variables are **Height and Age**.

---

## 14. Self-Check Questions (with Answers)

<details>
<summary><b>Q1.</b> SST = 200 and SSE = 50. Find SSR and R², and interpret R².</summary>

SSR = 150 and R² = 0.75. **75% of the variation in Y is explained by the model**; 25% is unexplained.
</details>

<details>
<summary><b>Q2.</b> A model has 3 predictors and n = 50. What do you divide SSE by to get MSE?</summary>

$n - p - 1 = 50 - 3 - 1 = 46$.
</details>

<details>
<summary><b>Q3.</b> Why can't the toddler model predict a 6-month-old's height?</summary>

The data cover 18–29 months, so 6 months is **extrapolation**. Growth in infancy is much faster, so the line likely doesn't apply.
</details>

<details>
<summary><b>Q4.</b> Why is newspaper significant alone but not in the multiple model?</summary>

Newspaper is **correlated with radio**. Alone, it takes credit for radio's effect. Once radio is held constant, newspaper adds nothing.
</details>

<details>
<summary><b>Q5.</b> What are the dimensions of X for Advertising?</summary>

**200 × 4**: 200 markets; columns for 1 (intercept), TV, radio, newspaper. X′X is 4 × 4.
</details>

<details>
<summary><b>Q6.</b> Biomass = 67.76 × DrivingTime (mg, minutes). Which are true? (a) +0.0678 g per minute (b) time increases per 0.0676 g (c) +67.76 mg per minute (d) +67776 mg per minute</summary>

**(a) and (c).** 67.76 mg = 0.0678 g. (b) reverses the direction; (d) multiplies by 1000 instead of dividing.
</details>

<details>
<summary><b>Q7.</b> Fill in the ANOVA table: n = 30, p = 4, SSR = 600, SSE = 250.</summary>

df: regression 4, error 25, total 29. MSR = 150, MSE = 10, **F = 15**, SST = 850, R² = 0.71.
</details>

<details>
<summary><b>Q8.</b> The global F-test gives p = 0.40. What now?</summary>

Fail to reject H₀. **Stop**: don't go looking through t-tests (that risks false positives). Rethink the model.
</details>

<details>
<summary><b>Q9.</b> If newspaper is listed first in the formula, does its <code>aov()</code> SS change? Its t-test p-value?</summary>

The `aov()` SS **changes** (to 282.34) because it is sequential. The t-test p-value **doesn't change** (0.86) because each t-test is conditional on all other predictors.
</details>

<details>
<summary><b>Q10.</b> Compute newspaper's t-statistic (estimate −0.001037, SE 0.005871).</summary>

t = −0.177, df = 196, p = 0.86. Fail to reject: newspaper doesn't add significantly **given TV and radio**.
</details>

<details>
<summary><b>Q11.</b> To test whether radio and newspaper together add anything beyond TV, what are the models and q?</summary>

Partial F-test. Full: TV + radio + newspaper. **Reduced: TV only.** q = 2.
</details>

<details>
<summary><b>Q12.</b> A predictor has t = 0.8. If dropped, what happens to R² and adjusted R²?</summary>

Adjusted R² goes **up** (|t| < 1). Plain R² goes **down** (or stays the same).
</details>

<details>
<summary><b>Q13.</b> A manager is opening one new store. Confidence or prediction interval?</summary>

**Prediction interval**, because it is one individual observation, not an average.
</details>

<details>
<summary><b>Q14.</b> If RSE doubled, what happens to the prediction interval?</summary>

Its width roughly **doubles**, since RSE dominates the width when n is large.
</details>

---

## 15. Datasets and References

**Datasets** (from the course GitHub):
- [Advertising.txt](https://raw.githubusercontent.com/DanikaLipman/DATA603/main/Advertising.txt): sales and TV/radio/newspaper budgets for 200 markets
- [condominium.csv](https://raw.githubusercontent.com/DanikaLipman/DATA603/main/condominium.csv): list price, living area, floors, bedrooms, baths for 15 condos
- [canolayield_pg.csv](https://raw.githubusercontent.com/DanikaLipman/DATA603/main/canolayield_pg.csv): simulated canola yields with insecticide, fertilizer, heat, and rain

**References:**
- James, Witten, Hastie & Tibshirani, *An Introduction to Statistical Learning with Applications in R*. Springer.
- Wickham & Grolemund, *R for Data Science*. O'Reilly.
- DATA 603 lecture notes (Danika Lipman, based on content by Thuntida Ngamkham, with additions by Jim Stallard). These study notes are my own explanations and summaries.

---

### 🔜 Next up
- Condominium problem: full model, CIs, ANOVA, partial F-tests, model comparison, prediction
- Canola yield problem: build and defend a model
# DATA603
