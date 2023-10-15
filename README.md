# Practical_machine_learning_week4_project
options(tidyverse.quiet = TRUE)


suppressPackageStartupMessages({
  suppressMessages({
    library(tidyverse)
    library(lubridate)
    library(caret)
    library(doParallel)
    library(lime)
  })
})  

theme_set(theme_light())

# Load Data
suppressWarnings({
pml_training <- readr::read_csv("pml-training.csv", 
                                na=c("", "#DIV/0!", "NA"), 
                                progress = F,
                                col_types = cols())

pml_testing  <- readr::read_csv("pml-testing.csv",
                                na=c("", "#DIV/0!", "NA"), 
                                progress = F,
                                col_types = cols())})

# Collect Informations
missingValues <- sapply(pml_training, function(x) sum(is.na(x)))
missingValues <- missingValues[missingValues>0]
userNames <- unique(pml_training$user_name)

# Prepare Data
prepare <- function(x,.train=T) {
  x$cvtd_timestamp <- x$raw_timestamp_part_1 +  x$raw_timestamp_part_2/1000000
  x$raw_timestamp_part_1 <- NULL
  x$raw_timestamp_part_2 <- NULL
  x$X1 <- NULL
  if(.train)
    x$classe <- as.factor(x$classe)
  x$user_name <- factor(x$user_name,userNames)
  x$new_window <- NULL
  x %>% select(-matches(names(missingValues)))
} 

pml_training <- prepare(pml_training)
pml_testing <- prepare(pml_testing,F)
