
<!-- README.md is generated from README.Rmd. Please edit that file -->

# deploytidymodels

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![CRAN
status](https://www.r-pkg.org/badges/version/tidymodelsdeploy)](https://CRAN.R-project.org/package=tidymodelsdeploy)
<!-- badges: end -->

The goal of deploytidymodels is to provide fluent tooling to version,
share, and deploy a trained model
[workflow](https://workflows.tidymodels.org/). Functions handle both
recording and checking the model’s input data prototype, and loading the
packages needed for prediction.

## Installation

You can install the released version of deploytidymodels from
[CRAN](https://CRAN.R-project.org) with:

``` r
~~install.packages("deploytidymodels")~~
```

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("juliasilge/deploytidymodels")
```

## Example

You can use the [tidymodels](https://www.tidymodels.org/) ecosystem to
train a model, with a wide variety of preprocessing and model estimation
options.

``` r
library(parsnip)
library(workflows)
data(Sacramento, package = "modeldata")

rf_spec <-
    rand_forest() %>%
    set_mode("regression") %>%
    set_engine("ranger")

rf_fit <-
    workflow() %>%
    add_formula(price ~ type + sqft + beds + baths) %>%
    add_model(rf_spec) %>%
    fit(Sacramento)

rf_fit
#> ══ Workflow [trained] ══════════════════════════════════════════════════════════
#> Preprocessor: Formula
#> Model: rand_forest()
#> 
#> ── Preprocessor ────────────────────────────────────────────────────────────────
#> price ~ type + sqft + beds + baths
#> 
#> ── Model ───────────────────────────────────────────────────────────────────────
#> Ranger result
#> 
#> Call:
#>  ranger::ranger(x = maybe_data_frame(x), y = y, num.threads = 1,      verbose = FALSE, seed = sample.int(10^5, 1)) 
#> 
#> Type:                             Regression 
#> Number of trees:                  500 
#> Sample size:                      932 
#> Number of independent variables:  4 
#> Mtry:                             2 
#> Target node size:                 5 
#> Variable importance mode:         none 
#> Splitrule:                        variance 
#> OOB prediction error (MSE):       7091410811 
#> R squared (OOB):                  0.5875709
```

You can **version** and **share** your model by
[pinning](https://pins.rstudio.com/dev/) it, to a local folder, RStudio
Connect, Amazon S3, and more.

``` r
library(deploytidymodels)
library(pins)
library(plumber)

model_board <- board_temp()
model_board %>% pin_model(rf_fit, model_id = "sacramento_rf")
#> Creating new version '20210621T233246Z-d5e46'
```

You can **deploy** your pinned model via a [Plumber
API](https://www.rplumber.io/), which can be [hosted in a variety of
ways](https://www.rplumber.io/articles/hosting.html).

``` r
pr() %>%
    pr_model(model_board, "sacramento_rf") %>%
    pr_run(port = 8088)
```

Make predictions with your deployed model at its endpoint and new data.

``` r
library(tidyverse)
library(deploytidymodels)
data(Sacramento, package = "modeldata")

new_sac <- Sacramento %>% 
    slice_sample(n = 20) %>% 
    select(type, sqft, beds, baths)

predict_api("http://127.0.0.1:8088/predict", new_sac)
```

## Contributing

This project is released with a [Contributor Code of
Conduct](https://contributor-covenant.org/version/2/0/CODE_OF_CONDUCT.html).
By contributing to this project, you agree to abide by its terms.

-   For questions and discussions about tidymodels packages, modeling,
    and machine learning, please [post on RStudio
    Community](https://community.rstudio.com/new-topic?category_id=15&tags=tidymodels,question).

-   If you think you have encountered a bug, please [submit an
    issue](https://github.com/tidymodels/deploytidymodels/issues).

-   Either way, learn how to create and share a
    [reprex](https://reprex.tidyverse.org/articles/articles/learn-reprex.html)
    (a minimal, reproducible example), to clearly communicate about your
    code.

-   Check out further details on [contributing guidelines for tidymodels
    packages](https://www.tidymodels.org/contribute/) and [how to get
    help](https://www.tidymodels.org/help/).
