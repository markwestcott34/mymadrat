MADRaT Exercise
================
David Chen
2019-09-10

### Learning Objectives

In this exercise, we will practice reading in and transforming input data through MADRaT. We will create our own MADRat-based package for data input processing, and create the standar set of functions that allow for reading in standardized input data. For this example, we will use World Bank World Development Indicator (WDI) data, with the WDI library for easy direct download.

### 1. New "mymadrat" Package

First, create a new package through the "New Project-&gt; New Directory-&gt; R Package" option in RStudio and call it "mymadrat". Add the following script saved as madrat.R to the newly created R folder.

``` r
### madrat.R
#' @importFrom madrat vcat
> .onLoad <- function(libname, pkgname){
> madrat::setConfig(packages=c(madrat::getConfig("packages"),pkgname),
          .cfgchecks=FALSE, .verbose=FALSE)
> }
#create an own warning function which redirects calls to vcat (package internal)
> warning <- function(...) vcat(0,...)
# create a own stop function which redirects calls to stop (package internal)
> stop <- function(...) vcat(-1,...)
# create an own cat function which redirects calls to cat (package internal)
> cat <- function(...) vcat(1,...)
```

### 2. MADRaT Functions - Exercise

We will create new download, read, and convert functions for WDI indicators Population, GDP, Employment rate, Employment share in Agriculture, and Agricultural GDP. Each indicator should be a separate parameter within each function. Note that each function needs to be named as a file and as a function with the wrapper, for example, readWDI.R

Remember that complete magclass objects are an array with the the regional (ISO3) dimension in the first dimension, the temporal dimension in the second, and data values in the third dimemsion(s) (3.1, 3.2...).

### 2.1 Download function

Note that if direct download not possible, data files can be manually created in the inputdata/sources folder. In that case, a download function is not necessary, but naming of the source folder must match the read functions.

``` r
install.packages("WDI")
library(WDI)

downloadWDI <- function(...){

# Use the WDI() function to easily direct download WDI data;
  #with 1960 as the start year and 2018 as end; see ?WDI() for more help
install.packages("WDI")
library(WDI)

# Download these indicators, by creating a vector of indicators to put into WDI():
# "NY.GDP.MKTP.CD" National GDP in current USD
# "SP.POP.TOTL" Total population
# "SL.AGR.EMPL.ZS" Employment in agriculture as % of total employment
# "NV.AGR.TOTL.CD" Agricultural GDP in current USD

#Save the downloaded data as a .Rda file as the last step of the function
save()
}
```

### 2.2 Read function

Read functions are the first step in transforming input data into magclass objects. They should be as simple as possible, with most steps of filling in, transforming, and data cleaning reserved for the convert function. The Read function should be able to specify between indicators (subtypes).

``` r

readWDI <- function(...){

#begin by loading the dataset saved by the previous download function.

load(...)
  
#Use as.magpie() to transform the dataset into a magclass object. 
#Remember that magclass objects always have the spatial indicator (iso code, not country name) in their 1st dimension & the temporal dimension in 2nd dimenson.
#It is ideal to reshape the data into 'tidy' format, with only the last column containing data as.magpie(...tidy=TRUE). 
#In this case, we will use the melt() function. 

library(reshape2)
wdi <- melt(wdi, id.vars=c("iso2c", "year"))

#Now create the magpie object, fill in the missing parameters for as.magpie:
#Note the replacement parameter, which maintains the naming of the indicators


wdi <- as.magpie(wdi,..., replacement=".")

#Once the magclass object is created, take a look at it. What does the function fulldim() tell you?
#we can specify the subtype/indicator desired,
#so that the read function only returns one indicator at a time.
#Use [] to subset the subtype:

x <- wdi[]

return(x)
}
```

### 2.3 Convert Function

The convert function will complete the magclass object: All 249 countries represented in MAGPie need to have a value and be in ISO3 country code.

``` r

convertWDI <- function(...){

  #Units need to be regularized in this function as well, for instance, MAgPIE uses population and GDP in millions:
  if (subtype %in% "NY.GDP.MKTP.KD", "SP.POP.TOTL","NV.AGR.TOTL.KD" ) {
    x <- x/1e6
}

# For these datasets, we need to convert the iso2c countrycodes into iso3c (this is also often done in the read function..)
# We can use the function countrycode(), with the magclass call getRegions. 

# Note:
# Other datasets may require other mapping conversions, there exist magclass specific tools: 
# toolCountry2isocode() and toolCell2isoCell() for instance
  
library(countrycode)
getRegions(x)<-countrycode(getRegions(x),"iso2c","iso3c")


# Note that some regions have now been turned into NA; you can check with getRegions()
# It would be important to be certain that no information is lost. In this case the NA's are mostly WB aggregate regions. 

x<-x[!is.na(getCells(x)),,] #remove NA regions


#clean_magpie() cleans MAgPIE objects so that they follow some extended magpie object rules (currently it makes sure that the dimnames have names and removes cell numbers if it is purely regional data)

x <- clean_magpie(x)


#Now fill the missing countries using toolCountryFill(), choose an arbitrary (Not NA!) fill value for now.

x <- toolCountryFill()

#Now we have a complete MAgPIE object! congratulations. 

return(x)}
  
```

### 3. calcOutput

Magclass objects lend themselves easily to calculations between them. The calcOutput wrapper function calls the functions used to transform input data, called as calcOutput("type", "subtype", ...)

Use the read functions we just wrote to calculate agricultural GDP as a percentage of total gdp, using a new calcOutput function.

``` r
calcAgGDP <- function(){

readSource("WDI", subtype="NY.GDP.MKTP.KD", aggregate=FALSE)

#note that by default, readSource also converts; otherwise set convert=FALSE

}
```

After the function is written and built, call calc functions through the calcOutput("type") wrapper. Try it now with the calcAgGDP function. What is Germany's share of Agricultural GDP in 2010?

(By default, calcOutput functions will aggregate to the regional level.)

All functions are saved in the inputdata/cache file after the first run, increasing efficiency. This means that for any updates to functions, the older cached function needs to be deleted. Cache is toggled with setConfig(forcecache=TRUE)
