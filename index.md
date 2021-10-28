# Using Dictionary Matching in Scottish Committee Transcripts

In this mini-project, we go through a very quick example of using string matching to analyse texts. The analysis is not comprehensive and is only meant to give some idea of how these things work.

## Tools - setting up R

I have listed some useful tutorials for getting started. If you follow these, you will be ready to follow along with this mini-project.

* [Downloading R](https://cran.r-project.org/bin/windows/base/)
* [Downloading R Studio](https://www.rstudio.com/products/rstudio/download/)
* [Setting up R Studio](https://rstudio-education.github.io/hopr/starting.html)

## Getting the Data
First we need to source some data. Pre-scraped dataset of various Hansards are available online. We will focus on examples from the UK. Specifically, we will use the parlScot dataset (Braby and Stewart, 2021) available by following the following [link](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/EQ9WBE).

Download the file 'parlScot_coms_v1.1.rds' and save it to a folder on your computer.

For this mini-project, we will restrict our analysis to variables already contained in the data. We will analyse the corpus using string matching method to measure when a Scottish MP (MSP) is referencing their own region or constituency in their speech. We will then compare whether this matches their "type" - i.e. whether they were elected at the constituency or regional level.

The corpus is stored as an `.rds` file. This stores a single R object which means that the data is already prepared for use in R.

## Preparing the data in R
The first thing we need to do is install and load our packages. The following code will check if the package is already installed. If so it will load it, if not it will install it and then load it.
```
if(!require("tidyverse")) {install.packages("tidyverse"); library("tidyverse")}
if(!require("here")) {install.packages("here"); library("here")}
if(!require("rjson")) {install.packages("rjson"); library("rjson")}
if(!require("ggplot2")) {install.packages("ggplot2"); library("ggplot2")}
```

Next, we load in our data. We are using the `here` package to specify file paths. This is so that the script can be run on different operating systems.

```
corpus <- readRDS(here("data_raw", "parlScot_coms_v1.1.rds")) # specify whatever path of the folder you have downloaded the data in to
```

We only want to look at speeches so we will discard anything that isn't a speech using a filter command. String matching is case sensitive, so the last step of preparing our data is to convert all the characters in the speeches (as well as the text in constituency and region names) to lower case.

```
corpus <- corpus %>% filter(is_speech == 1)

corpus$speech <- tolower(corpus$speech)
corpus$constituency <- tolower(corpus$constituency)
corpus$region <- tolower(corpus$region)

```

Finally, we want to split the corpus into the two types of MP.
```
corpus_con <- corpus %>% subset(msp_type == "Constituency")

corpus_reg <- corpus %>% subset(msp_type == "Region")
```

## Analysis

We are using a string matching method. The essence of this is that we are asking the computer to count all the instances of a particular set of strings that occurs in the text. The set of strings we are interested in finding is called our *dictionary*.

In the case we will have two types of dictionary. The first will simply be the name of the MSP's constituency/region. The second type of dictionary will be general terms associated with geographical representation. We will have two dictionaries of each type - one for constituency and one for region.

Thus there are two matches we are looking for. Let's start with the direct mention of a constituency name. For each row, we want to print a binary variable for whether an MP references the *name* of their region or constituency. We can then see the proportion of speeches that contained the name of the MSP's geographical area by taking a mean of the column.

```
corpus_con$name_ref <- as.integer(str_detect(corpus_con$speech, corpus_con$constituency))
corpus_reg$name_ref <- as.integer(str_detect(corpus_reg$speech, corpus_reg$region))

mean(corpus_con$name_ref) * 100
mean(corpus_reg$name_ref) * 100
```
As we can see from this Constituency MP's mention the name of their constituency in **0.59%** of their speeches but region MPs mention the name of their region in **1.20%** of their speeches.

Of course, there is more to geographic representation than saying the name. And it might be the case that regions are mentioned more any way. We will look at a broader search for geographical representation when we use our second type of dictionary. First, however, let's check how often Constituency MSP's reference their own region. We will do the same for regional MPs, just to that both data frames have the same variables so we can merge them later.

```
corpus_con$regional_ref <- as.integer(str_detect(corpus_con$speech, corpus_con$region))
corpus_reg$regional_ref <- as.integer(str_detect(corpus_reg$speech, corpus_reg$region))

mean(corpus_con$regional_ref) * 100
mean(corpus_reg$regional_ref) * 100
```
As we can see, Regional MSP's still mention their Region far more often than Constituency MP's mention their Region.

Moving on to our second type of dictionary. The first step is to define the relevant dictionaries. And then we can run the string match for both types of reference. We will run both dictionaries on both types of MSP. Of course, if we were doing this project properly, we would have to come up with theoretical justifications of the words we use, but for the purposes of this exercise, let's keep it simple.
```
con_ref_dict <- "my constitu*|our constiu*"
reg_ref_dict <- "my region*|our region*"

corpus_con$dict_con_ref <- as.integer(str_detect(corpus_con$speech, con_ref_dict))
corpus_con$dict_reg_ref <- as.integer(str_detect(corpus_con$speech, reg_ref_dict))

corpus_reg$dict_con_ref <- as.integer(str_detect(corpus_reg$speech, con_ref_dict))
corpus_reg$dict_reg_ref <- as.integer(str_detect(corpus_reg$speech, reg_ref_dict))
```

The last step on the analysis side of things is to combine these different types of geographical reference into a single variable. For this we will write a function that will print a 1 if the sum of the two string matching variables is 1 or more.


```
corpus_con$any_con_ref <- NA
corpus_con$any_con_ref[corpus_con$name_ref + corpus_con$dict_con_ref >= 1] <- 1
corpus_con$any_con_ref[corpus_con$name_ref + corpus_con$dict_con_ref == 0] <- 0


corpus_reg$any_reg_ref <- NA
corpus_reg$any_reg_ref[corpus_reg$name_ref + corpus_reg$dict_reg_ref >= 1] <- 1
corpus_reg$any_reg_ref[corpus_reg$name_ref + corpus_reg$dict_reg_ref == 0] <- 0

corpus_con$any_reg_ref <- NA
corpus_reg$any_con_ref <- NA

# the last two lines make sure there are matching columns
```

That's all the string matching done!

Now lets create a summary of what we found

```
mean(corpus_con$any_con_ref)*100

mean(corpus_reg$any_reg_ref)*100
```

If all went well, the answers for the two lines of code above are:
* Constituency MSP's reference their constituency in **1.5%** of their speeches
* Regional MSP's reference their region in **1.2%** of their speeches
```
