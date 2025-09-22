[_metadata_:author]:- "Emil Niclas Meyer-Hansen"
[_metadata_:date]:- "22/9/2025"
[_metadata_:tags]:- "markdown metadata"
# Bayesian Kaiser-Meyer-Olkin index

## Description
Function for computing the *Bayesian Kaiser-Meyer-Olkin* (BKMO) index, which is *a measure of the posterior 'sampling adequacy' of the data matrix given the (modeled) data* (Meyer-Hansen, 2025).

## Usage
```r
BKMO(r = NULL)
```

## Arguments
`r` Matrix of inter-correlations, where columns are variables and rows are posterior draws

## Details

The BKMO index is the posterior 'sampling adequacy' of a data matrix given the (modeled) data, which refers to the probability that the data matrix is factorable given the (linear) relationships between variables and prior beliefs about these relationships. It was introduced by Emil Niclas Meyer-Hansen (2025) and is a Bayesian reconceptualization of the Frequentist 'Mark IV' KMO index by Henry F. Kaiser, Edward P. Meyer, Ingram Olkin, and John Rice (Kaiser & Rice, 1974).

## Value

Matrix of BKMO index values, where columns consist of one BKMO index for each input variable and one overall BKMO index, while rows are posterior draws.

## Function

```r
BKMO <- function(r = NULL){
  if(!is.matrix(r)) r <- as.matrix(r)
  
  # Compute p (i.e., number of theorized manifestations)
  disc <- sqrt(1 + 8 * ncol(r))
  p <- (1 + disc) / 2
  
  # Define utility function for vectorized operation
  utility_KMO <- function(r = r, p = p){
    # Construct correlation matrix 
    R <- diag(1, p)
    R[upper.tri(R)] <- r
    R[lower.tri(R)] <- t(R)[lower.tri(R)]
    row.names(R) <- paste0('m', 1:p)
    colnames(R) <- paste0('m', 1:p)
    
    # Compute inverse R
    R_inv <- solve(R)
    
    # Compute anti-image R
    R_q <- -R_inv / sqrt(outer(diag(R_inv), diag(R_inv)))
    
    # Compute KMO
    r2 <- R^2
    q2 <- R_q^2
    diag(r2) <- 0
    diag(q2) <- 0
    kmo <- c(
      rowSums(r2) / (rowSums(r2) + rowSums(q2)),
      overall = sum(r2) / (sum(r2) + sum(q2))
    )
    return(kmo)
  }
  
  # Compute KMO for each posterior draw using vectorization
  bkmo <- do.call(rbind, lapply(1:nrow(r), function(i) utility_KMO(
        r = r[i, , drop = FALSE], p = p
      )
    )
  )
  return(as.data.frame(bkmo))
}
```

## References

- Kaiser, H. F., and J. Rice (1974): 'Little Jiffy, Mark IV', *Educational and Psychological Measurement*, 34(1): 111–117. DOI: [10.1177/001316447403400115](https://doi.org/10.1177/001316447403400115)
- Meyer-Hansen, E. N. (2025): 'Revisiting 'Little Jiffy, Mark IV': Towards a Bayesian KMO index', *Open Science Framework*, Working paper (v2025-09-19-10-52). DOI: [10.17605/OSF.IO/T3UPD](https://doi.org/10.17605/OSF.IO/T3UPD)

## Author

(Re)conceptualization & Code: Emil Niclas Meyer-Hansen

## Note

- Created: 2025-05-20

## See also

[`psych::KMO`](https://cran.r-project.org/web/packages/psych/index.html) for computing the Frequentist KMO index.

## Example

```r
# Simulate data
set.seed(1)
library("tidyverse")
sample_size <- 1000
n_manifestations <- 10
sesoi <- .3
loadings <- runif(n_manifestations, sesoi, .7)
df <- data.frame(
  latent = rnorm(sample_size)
)
for(i in 1:n_manifestations){
  df[,paste0("m", i)] <- loadings[i]*df$latent + rnorm(sample_size)
}
df <- df %>% select(-latent)

# Define maximum-likelihood functions
sd2 <- function(x = NULL, na.rm = FALSE){
  stopifnot(is.numeric(x))
  stopifnot(is.logical(na.rm))

  x_mean <- mean(x, na.rm = na.rm)
  x_var <- mean((x - x_mean)^2, na.rm = na.rm)
  x_sd <- sqrt(x_var)
  return(x_sd)
}
standardize2 <- function(x = NULL, na.rm = FALSE){
  stopifnot(is.numeric(x))
  stopifnot(is.logical(na.rm))
  
  x_mean <- mean(x, na.rm = na.rm)
  x_var <- mean((x - x_mean)^2, na.rm = na.rm)
  x_sd <- sqrt(x_var)

  x_standardized <- (x - x_mean) / x_sd
  return(x_standardized)
}

# Standardize data matrix
df <- apply(df, 2, standardize2) %>% as.data.frame()

# Specify MCMC
library("parallel")
posterior_samples <- 4000
chains <- cores <- parallel::detectCores()-1
warmup <- 1000
iter <- ceiling((posterior_samples / chains) + warmup)
posterior_samples <- (iter - warmup)*chains

# Specify Bayesian Generalized Linear Model (BGLM)
library("brms")
bglm_model <- brms::bf(
  mvbind(m1, m2, m3, m4, m5, m6, m7, m8, m9, m10) ~ 0,
  sigma ~ 0,
  family = gaussian(
    link = "identity"
  )
) + set_rescor(TRUE)


# Specify 'weakly-informative' model priors
bglm_priors <- brms::get_prior(bglm_model, data = df)
bglm_priors[1,] <- brms::set_prior(
  "lkj(2)",
  class = "rescor"
)

# Fit Bayesian multivariate linear model with brms
bglm_fit <- brms::brm(
  formula = bglm_model,
  family = gaussian(
    link = "identity"
  ),
  prior = bglm_priors,
  data = df, 
  chains = chains,
  cores = cores,
  iter = iter,
  warmup = warmup
)

# Extract posterior draws of the (residual) correlations from the fitted Bayesian model
library("stringr")
bglm_samples <- brms::as_draws_df(bglm_fit)
bglm_correlations <- bglm_samples[, stringr::str_detect(colnames(bglm_samples), "rescor__")]

# Compute Bayesian KMO index
library("bayestestR")
kmo_bayes <- BKMO(r = bglm_correlations)
hist(kmo_bayes$overall)
print(
  paste0(
    "Mean = ", mean(kmo_bayes$overall) %>% round(digits = 3),
    " (SD = ", sd2(kmo_bayes$overall)  %>% round(digits = 3),
    "; 95% HDI[",
    bayestestR::hdi(kmo_bayes$overall)$CI_low  %>% round(digits = 3), "; ",
    bayestestR::hdi(kmo_bayes$overall)$CI_high  %>% round(digits = 3), "])"
  )
)
```

---
Revised 9-22-2025 - [Emil Niclas Meyer-Hansen](mailto:emil098meyerhansen@gmail.com)
