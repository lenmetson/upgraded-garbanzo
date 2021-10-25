# Using Dictionary Matching in Parliamentary Texts

## Why use this method

## Tools - setting up R

In this tutorial, I won't go into downloading and setting up R Studio, these are better explained in other places. I have listed some useful tutorials for getting started. If you follow these, you will be ready to follow along with this mini-project.

* [Downloading R](https://cran.r-project.org/bin/windows/base/)
* [Downloading R Studio](https://www.rstudio.com/products/rstudio/download/)
* [Setting up R Studio](https://rstudio-education.github.io/hopr/starting.html)

## Getting the Data
First we need to source some data. Pre-scraped dataset of various Hansards are available online. We will focus on examples from the UK. Specifically, we will use the parlScot dataset (Braby and Stewart, 2021) available by following the following link: https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/EQ9WBE. This data set contains 1.8 million speeches from the Scottish Parliament made between 1999 and 2021.

For this mini-project, we will restrict our analysis to variables already contained in the data. We will analyse the corpus using string matching to measure when a Scottish MP (MSP) is referencing their own region or constituency (`constituency` or `region`) in their speech (`speech`). We will then compare whether this matches their "type" (`msp_type`)- i.e. whether they were elected at the constituency or regional level.

The corpus is stored as an `.rds` file. This stores a single R object which means that the data is already prepared for use in R.

## Preparing the data in R
First we need to install our packages. 
```
if(!require("tidyverse")) {install.packages("tidyverse"); library("tidyverse")}

```

* Installing packages
* Reading in data
* Add in other variables
* Filter down the data to select what we want

## Analysis
* Set dictionary
* Carry out string match
* Summarise data  

## Vizualisation
* What viz, ggplot
