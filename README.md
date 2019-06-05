
<!-- README.md is generated from README.Rmd. Please edit that file -->

# immunotherapy

<!-- badges: start -->

<!-- badges: end -->

The goal of `immunotherapy` is to demonstrate building a TensorFlow
model in immunotherapy, based on peptide data.

This is based on the the original work by Leon Eyrich Jessen, and his
blog post on RViews: [“Deep Learning for Cancer
Immunotherapy”](https://blogs.rstudio.com/tensorflow/posts/2018-01-29-dl-for-cancer-immunotherapy/)

## Requirements

Packages for sequence logos and peptides

``` r
devtools::install_github("omarwagih/ggseqlogo")
devtools::install_github("leonjessen/PepTools")
```

## About peptides

From [Wikipedia](https://en.wikipedia.org/wiki/Peptide)

> Peptides (from Greek language πεπτός, peptós “digested”; derived from
> πέσσειν, péssein “to digest”) are short chains of amino acid monomers
> linked by peptide (amide) bonds.

<img src="inst/images/tetrapeptide_formula_wikipedia.png" width="400px" />

The package `PepTools` is “An R-package for making immunoinformatics
accessible”. For example, it exports a function `pep_encode()` to
convert a peptide string, e.g. “LLTDAQRIV” into a numerical
representation:

``` r
suppressPackageStartupMessages({
  library(PepTools, quietly = TRUE)
  library(magrittr, quietly = TRUE)
})

#' Use palette from ColorBrewer to recolour the image
#'
#' @param x Matrix
#' @param palette ColorBrewer palette. This value is passed to [scales::col_numeric()]
#' @param zero_colour Colour to use for zero and NA values
#' @param invert If `TRUE`, inverts the colour scale
recolour <- function(x, palette = "Blues"){
  dims <- dim(x)
  col_range <- range(x, na.rm = TRUE)
  col_custom <- scales::col_numeric(
    palette = palette, domain = col_range
  )
  as.raster(matrix(col_custom(x), nrow = dims[1], ncol = dims[2]))
}

# Plot peptide representation
plot_peptide <- function(x, palette = "Blues"){
  
  plot.new()
  x %>% 
    pep_encode() %>%
    .[1, , ] %>% 
    recolour(palette = palette) %>% 
    rasterImage(0, 0, 1, 1, interpolate = FALSE)
  title(paste0("Peptide representation of ", x))
}

plot_peptide("LLTDAQRIV")
```

![](README_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## Workflow

The objective is to create a TensorFlow model that can be called by user
code. The user code sends a request as a peptide string and receives a
response in the form of a data frame of predicted classes.

However, to achieve this, we need an intermediate function to translate
the string into a peptide representation and a tensor of the shape that
the model expects.

![](inst/images/workflow_objective.png)

![](inst/images/workflow_deployment.png)

## Deploying the model yourself

Then follow these steps:

1.  Ensure all the pre-requisites are in place

<!-- end list -->

  - Ensure you have access to a RStudio Connect instance with
    permissions to publish APIs. The instance should be configured to
    run TensorFlow models.

  - Run the file `0_install_pre_reqs.R` - this will install all the
    required packages, including `keras`, if not already installed.

  - Make sure you have established a connection to Connect
    
      - For example, deploy a test Shiny app
      - Specifically, the result of `rsconnect::accounts()` should not
        be empty

<!-- end list -->

1.  Train the model
    
      - Run `1_train_model.R`
      - This trains the TensorFlow model

2.  Publish the model to Connect

<!-- end list -->

  - At the moment there is no push-button deployment of TensorFlow
    models to Connect
  - Run `2_publish_model.R`
  - This publishes the model to Connect
  - Change the permissions to be open the model to the world (no login
    required)
  - Make a note of the “Solo URL”

<!-- end list -->

1.  Edit `config.yml` with solo url

<!-- end list -->

  - Add your solo url to `solo_url_tensorflow`

<!-- end list -->

1.  Publish the plumber API to Connect

<!-- end list -->

  - Run `4_publish_api.R`
  - Change the permissions to be open the model to the world (no login
    required)
  - Make a note of the “Solo URL”
  - Try to execute the model from the Swagger page

<!-- end list -->

1.  Edit `config.yml` with solo url

<!-- end list -->

  - Add your solo url to `solo_url_plumber`

<!-- end list -->

1.  Consume the model

<!-- end list -->

  - Open the file `5_consume_api.R`
  - Run the code and watch the predictions flow in\!
