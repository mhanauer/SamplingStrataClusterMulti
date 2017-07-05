---
title: "Survey Weighting for Stratas, Clusters, and Multistage"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(
	echo = TRUE,
	message = FALSE,
	warning = FALSE
)
```
Here are three examples of selecting and two examples of analyzing data from clusters, stratas, and multistage (i.e. take a cluster sample then sample the strata with the selected clusters).  The first example uses two packages survey and sampling, which will each need to be installed and libraried.  In this first example, we look at how to select a stratified random sample and then calculate the mean using the correct survey design.  The first step is to create an artificial data set.  In this data set, there are three continuous variables (y, x1, and x2) and one strata variable indicating which of the five stratas a data point lies within.  We can then use the strata function in the sampling package to select a subset of data based upon stratas.  In the example below, we specified the strata variable as x3 and are telling R to select 100 units from each of the five stratas using a simple random sample without replacement (srswor) method.  Then we need to combine the original data with the other variables (i.e. x1 and x2 are dropped when we selected the stratified sample) to obtain a full data set.  Then we can calculate the survey mean using the svydesign function in the survey package.  For this example, the id =~ 1, because we used a simple random sample from the stratas, the strata argument calls for the variable containing the stratum, which is Stratum in this example, and then we need to specify the data, which is the full data set.  Finally, we can calculate the mean by using the surveymean function.  In this example, we are calculating the mean of y, by specifying the y variable in the full data set and also specifying the survey design. 
```{r, message=FALSE, warning=FALSE}
library(survey)
library(sampling)

# Where 1,2,3,4,5 indicate different types of schools pre-school, primary, elementary, middle, and high school.  So want to sample a set number from
data = data.frame(y = rnorm(1000), x1 = rnorm(1000), x2 = rnorm(1000), x3 = rep(c(1,2,3,4,5), 200))
data = as.data.frame(data)

dataStrat = strata(data = data, stratanames = c("x3"), size = c(rep(100,5)), method = "srswor")

dataStratFull = getdata(data, dataStrat)
dataStratFull = as.data.frame(dataStratFull)
head(dataStratFull)

dataStratDesign = svydesign(id = ~1, strata = ~ Stratum, data = dataStratFull)

svymean(~ dataStratFull$y , dataStratDesign)

```
Now we are going to select a random sample from clusters.  Y, x1, and x4 (previously x3) are the same as before, and now we have a variable titled x3 that identifies the cluster.  In this example, there are 100 clusters with 100 data points in each cluster.  To select the data based upon cluster we use the cluster function.  The cluster function is the same as the strata, but instead of strataname, we input the clustername (x3 in this example) and then select the number of data points we want from each cluster indicated by the size argument, which is 50 in this example.  Then for the survey design instead of identifying the strata variable, the user will identify the cluster variable.     
```{r, message=FALSE, warning=FALSE}
data = data.frame(y = rnorm(10000), x1 = rnorm(10000), x2 = rnorm(10000), x3 = rep(c(1:100), 100), x4 = rep(c(1,2,3,4,5), 2000))
data = as.data.frame(data)

library(sampling)

dataCluster = cluster(data = data, clustername = c("x3"), size = 50, method = "srswor")

dataClusterFull = getdata(data, dataCluster)
dataClusterFull = as.data.frame(dataClusterFull)
head(dataClusterFull)

dataClusterDesign = svydesign(id = ~1, cluster = ~ x3, data = dataClusterFull) 
svymean(~ dataStratFull$y , dataStratDesign)
```
Finally, here is an example of using multistage sampling to first select from clusters and then within those clusters select from stratas.  This data set has variables for the clusters (x3) and the stratas (x4).  Now we can use the mstage function where we select the stages of sampling, which in this example is select from clusters and then from stratas, the method of selection for sampling strategy, which is srswor for the cluster and strata samples.  Then we select the varnames, which are the cluster and stratasnames, and select the size.  This size is a list with the first argument stating that we want five clusters selected at random and from those five clusters select 50 from each strata at random totaling 250 data points.  
```{r, message=FALSE, warning=FALSE}
data = data.frame(y = rnorm(10000), x1 = rnorm(10000), x2 = rnorm(10000), x3 = rep(c(1:100), 100), x4 = rep(c(1,2,3,4,5), 2000))
data = as.data.frame(data)

dataMulti = mstage(data, stage = list("cluster", "stratified"), method = list("srswor", "srswor"), varnames = c("x3","x4"), size = list(5,c(50,50,50,50,50)))
dataMultiTotal = getdata(data, dataMulti)
dataMultiTotal = as.data.frame(dataMultiTotal)
head(dataMultiTotal)
dim(dataMultiTotal)


```



