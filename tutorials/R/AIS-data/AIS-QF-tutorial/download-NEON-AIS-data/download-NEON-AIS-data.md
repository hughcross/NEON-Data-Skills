---
syncID: 317ecab8e00b4a959a76dba181bb33b8
title: "Download and work with NEON Aquatic Instrument Data"
description: "Tutorial for downloading NEON AIS data using the neonUtilities package and then exploring and understanding the downloaded data"
dateCreated: 2020-05-19
authors: Bobby Hensley, Guy Litt, Megan Jones
contributors: Felipe Sanchez
estimatedTime: 3 hours
packagesLibraries: neonUtilities, ggplot2
topics: data-management, rep-sci, data-viz
languageTool: R, API
code1: https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/download-NEON-AIS-data.R
tutorialSeries:
urlTitle: explore-neon-ais-data
---

This tutorial covers downloading NEON Aquatic Instrument System (AIS) data, 
using the neonUtilities R package, as well as basic instruction in beginning to 
explore and work with the downloaded data, including guidance in navigating 
data documentation, separating data using the horizontal location
(HOR) variable, interpreting quality flags, and resampling time intervals. 

The following material steps through the multiple considerations in 
interpreting NEON data, and ultimately achieves a data comparison between 
two different sensors at nearby locations that are published at different
time intervals. This sort of data wrangling is useful for comparing different 
data streams, and/or preparing data into a consistent format for modeling.

<div id="ds-objectives" markdown="1">

## Objectives

After completing this activity, you will be able to:

* Download NEON AIS data using the `neonUtilities` package.
* Understand downloaded data sets and load them into R for analyses.
* Separate data collected at different sensor locations using the HOR variable.
* Understand and interpret quality flags, including how to discover what non-
* standard quality flags mean.
* Aggregate time series to higher intervals and impute (fill-in) observations 
* where absent.

## Things You'll Need To Complete This Tutorial
To complete this tutorial you will need R (version >3.4) and, 
preferably, RStudio loaded on your computer.

### Install R Packages

* **neonUtilities**: Basic functions for accessing NEON data
* **ggplot2**: Plotting functions
* **dplyr**: Data manipulation functions
* **padr**: Time-series data preparation functions

These packages are on CRAN and can be installed by 
`install.packages()`.

### Additional Resources

* <a href="https://github.com/NEONScience/NEON-Utilities/neonUtilities" target="_blank">GitHub repository for neonUtilities</a>

</div>

## Download Files and Load Directly to R: loadByProduct()

The most popular function in `neonUtilities` is `loadByProduct()`. 
This function downloads data from the NEON API, merges the site-by-month 
files, and loads the resulting data tables into the R environment, 
assigning each data type to the appropriate R class. This is a popular 
choice because it ensures you're always working with the most up-to-date data, 
and it ends with ready-to-use tables in R. However, if you use it in
a workflow you run repeatedly, keep in mind it will re-download the 
data every time.

Before we get the NEON data, we need to install (if not already done) and load 
the neonUtilities R package, as well as other packages we will use in the 
analysis. 


    # Install neonUtilities package if you have not yet.
    install.packages("neonUtilities")
    install.packages("ggplot2")
    install.packages("dplyr")
    install.packages("padr")


    # Set global option to NOT convert all character variables to factors
    options(stringsAsFactors=F)
    
    # Load required packages
    library(neonUtilities)
    library(ggplot2)
    library(dplyr)
    library(padr)

The inputs to `loadByProduct()` control which data to download and how 
to manage the processing. The following are frequently used inputs: 

* `dpID`: the data product ID, e.g. DP1.20288.001
* `site`: defaults to "all", meaning all sites with available data; 
can be a vector of 4-letter NEON site codes, e.g. 
`c("MART","ARIK","BARC")`.
* `startdate` and `enddate`: defaults to NA, meaning all dates 
with available data; or a date in the form YYYY-MM, e.g. 
2017-06. Since NEON data are provided in month packages, finer 
scale querying is not available. Both start and end date are 
inclusive.
* `package`: either basic or expanded data package. Expanded data 
packages generally include additional information about data 
quality, such as individual quality flag test results. Not every 
NEON data product has an expanded package; if the expanded package 
is requested but there isn't one, the basic package will be 
downloaded.
* `avg`: defaults to "all", to download all data; or the 
number of minutes in the averaging interval. See example below; 
only applicable to IS data.
* `savepath`: the file path you want to download to; defaults to the 
working directory.
* `check.size`: T or F; should the function pause before downloading 
data and warn you about the size of your download? Defaults to T; if 
you are using this function within a script or batch process you 
will want to set this to F.
* `token`: this allows you to input your NEON API token to obtain faster 
downloads. 
Learn more about NEON API tokens in the <a href="https://www.neonscience.org/neon-api-tokens-tutorial" target="_blank">**Using an API Token when Accessing NEON Data with neonUtilities** tutorial</a>. 

There are additional inputs you can learn about in the 
<a href="https://www.neonscience.org/neonDataStackR" target="_blank">**Use the neonUtilities R Package to Access NEON Data** tutorial</a>. 

The `dpID` is the data product identifier of the data you want to 
download. The DPID can be found on the 
<a href="http://data.neonscience.org/data-products/explore" target="_blank">
Explore Data Products page</a>.

It will be in the form DP#.#####.###. For this tutorial, we'll use some data
products collected in NEON's Aquatic Instrument System: 

* DP1.20288.001: Water quality
* DP1.20033.001: Nitrate in surface water
* DP1.20016.001: Elevation of surface water

Now it's time to consider the NEON field site of interest. If not specified, 
the default will download a data product from all sites. The following are 
4-letter site codes for NEON's 34 aquatics sites as of 2020:

ARIK = Arikaree River CO        
BARC = Barco Lake FL          
BIGC = Upper Big Creek CA       
BLDE = Black Deer Creek WY      
BLUE = Blue River OK            
BLWA = Black Warrior River AL        
CARI = Caribou Creek AK         
COMO = Como Creek CO          
CRAM = Crampton Lake WI         
CUPE = Rio Cupeyes PR           
FLNT = Flint River GA           
GUIL = Rio Guilarte PR          
HOPB = Lower Hop Brook MA       
KING = Kings Creek KS         
LECO = LeConte Creek TN         
LEWI = Lewis Run VA             
LIRO = Little Rock Lake WI      
MART = Martha Creek WA            
MAYF = Mayfield Creek AL        
MCDI = McDiffett Creek KS    
MCRA = McRae Creek OR           
OKSR = Oksrukuyik Creek AK      
POSE = Posey Creek VA           
PRIN = Pringle Creek TX       
PRLA = Prairie Lake ND          
PRPO = Prairie Pothole ND     
REDB = Red Butte Creek UT       
SUGG = Suggs Lake FL            
SYCA = Sycamore Creek AZ        
TECR = Teakettle Creek CA        
TOMB = Lower Tombigbee River AL       
TOOK = Toolik Lake AK         
WALK = Walker Branch TN         
WLOU = West St Louis Creek CO       

In this exercise, we will want data from only one NEON field site, Pringle 
Creek, TX (PRIN) from February, 2020. 

Now let us download our data. If you are not using a NEON token to download 
your data, neonUtilities will ignore the `token` input. We set `check.size = F`  
so that the script runs well but remember you always want to check your 
download size first.


    # download data of interest - Water Quality
    waq <- loadByProduct(dpID="DP1.20288.001", site="PRIN", 
                         startdate="2020-02", enddate="2020-02", 
                         package="expanded", 
                         token = Sys.getenv("NEON_TOKEN"),
                         check.size = F)

<div id="ds-challenge" markdown="1">
### Challenge: Download Other Related Data products
  
Using what you've learned above, can you modify the code to download data for 
the following parameters?

* Data Product DP1.20033.001: nitrate in surface water
* Data Product DP1.20016.001: elevation of surface water
* The expanded data tables
* Dates matching the other data products you've downloaded

1. What is the size of the downloaded data? 
2. Without downloading all the data, how can you tell the difference in 
size between the "expanded" and "basic" packages? 
</div>


    # download data of interest - Nitrate in Suface Water
    nsw <-  loadByProduct(dpID="DP1.20033.001", site="PRIN", 
                          startdate="2020-02", enddate="2020-02", 
                          package="expanded", 
                          token = Sys.getenv("NEON_TOKEN"),
                          check.size = F)
    
    # #1. 2.0 MiB
    # #2. You can change check.size to True (T), and compare "basic" vs "expaneded"
    # package types. The basic package is 37.0 KiB, and the expanded is 42.4 KiB. 


    # download data of interest - Elevation of surface water
    eos <- loadByProduct(dpID="DP1.20016.001", site="PRIN",
                         startdate="2020-02", enddate="2020-02",
                         package="expanded",
                         token = Sys.getenv("NEON_TOKEN"),
                         check.size = F)

## Files Associated with Downloads

The data we've downloaded comes as an object that is a named list of objects. 
To work with each of them, select them from the list using the `$` operator. 


    # view all components of the list
    names(waq)

    ## [1] "readme_20288"           "sensor_positions_20288" "variables_20288"       
    ## [4] "waq_instantaneous"

    # View the dataFrame
    View(waq$waq_instantaneous)

We can see that there are four objects in the downloaded water quality data. One 
dataframe of data (`waq_instantaneous`) and three metadata files. 

If you'd like you can use the `$` operator to assign an object from an item in 
the list. If you prefer to extract each table from the list and work with it as 
independent objects, which we will do, you can use the `list2env()` function. 


    # unlist the variables and add to the global environment
    list2env(waq, .GlobalEnv)

    ## <environment: R_GlobalEnv>

So what exactly are these four files and why would you want to use them? 

* **data file(s)**: There will always be one or more dataframes that include the 
primary data of the data product you downloaded. Multiple dataframes are 
available when there are related datatables for a single data product.
* **readme_xxxxx**: The readme file, with the corresponding 5 digits from the 
data product number, provides you with important information relevant to the 
data product and the specific instance of downloading the data. Here you can find manual flagging notes for all sites, locations, and time periods.
* **sensor_postions_xxxxx**: this file contains information about the coordinates
of each sensor, relative to a reference location. 
* **variables_xxxxx**: this file contains all the variables found in the 
associated data table(s). This includes full definitions, units, and other 
important information. 

Let's perform the same thing for the surface water nitrate and elevation of 
surface water data products too:

    list2env(nsw, .GlobalEnv)

    ## <environment: R_GlobalEnv>

    list2env(eos, .GlobalEnv)

    ## <environment: R_GlobalEnv>

Note that a few more objects were added to the Global Environment, including:

* `NSW_15_minute`
* `EOS_5_min`
* `EOS_30_min`

The `15_minute` name indicates the time-averaging intervals in a dataset. Other 
examples may include `5_min` and `30_min` in the same data product, such as 
elevation of surface water (DP1.20016.001). If only one time average interests 
you, you may specify the time interval of interest when downloading the data when
calling `neonUtilities::loadByProduct()`.

## Data from Different Sensor Locations (HOR)

NEON often collects the same type of data from sensors in different locations. These 
data are delivered together but you will frequently want to plot the data 
separately or only include data from one sensor in your analysis. NEON uses the 
`horizontalPosition` variable in the data tables to describe which sensor 
data is collected from. The `horizontalPosition` is always a three digit number 
for AIS data. Non-shoreline HOR examples as of 2020 at AIS sites include:

* 101: stream sensors located at the **upstream** station on a **monopod mount**, 
* 111: stream sensors located at the **upstream** station on an **overhead cable mount**, 
* 131: stream sensors located at the **upstream** station on a **stand alone pressure transducer mount**, 
* 102: stream sensors located at the **downstream** station on a monopod mount, 
* 112: stream sensors located at the **downstream** station on an **overhead cable mount** 
* 132: stream sensors located at the **downstream** station on a **stand alone pressure transducer mount**, 
* 110: **pressure transducers** mounted to a **staff gauge**. 
* 103: sensors mounted on **buoys in lakes or rivers**
* 130 and 140: sensors mounted in the **littoral zone** of lakes

You'll frequently want to know which sensor locations are represented in your 
data. We can do this by looking for the `unique()` position designations in 
`horizontalPostions`. 


    # which sensor locations exist for water quality, DP1.20288.001?
    print("Water quality horizontal positions:")

    ## [1] "Water quality horizontal positions:"

    unique(waq_instantaneous$horizontalPosition)

    ## [1] "101" "102"

We can see that there are two water quality sensor positions at PRIN in February 
2020. As the locations of sensors can change at sites over time (especially with 
aquatic sensors as AIS sites undergo redesigns) it is a good idea to check 
horizontal positions when you're adding in new locations or a new date range to 
your analyses. 

Let's check the HOR locations for surface water nitrate and elevation too:

    # which sensor locations exist for other data products?
    print("Nitrate in Surface Water horizontal positions: ")

    ## [1] "Nitrate in Surface Water horizontal positions: "

    unique(NSW_15_minute$horizontalPosition)

    ## [1] "102"

    print("Elevation of Surface Water horizontal positions: ")

    ## [1] "Elevation of Surface Water horizontal positions: "

    unique(EOS_30_min$horizontalPosition)

    ## [1] "110" "132"

Now we can use this information to split water quality data into the two
different sensor set locations: upstream and the downstream. 


    # Split data into separate dataframes by upstream/downstream locations.
    
    waq_up <- 
      waq_instantaneous[(waq_instantaneous$horizontalPosition=="101"),]
    waq_down <- 
      waq_instantaneous[(waq_instantaneous$horizontalPosition=="102"),]
    
    # Note: The surface water nitrate sensor is only stationed at one location.
    
    eos_up <- EOS_30_min[(EOS_30_min$horizontalPosition=="110"),]
    eos_down <- EOS_30_min[(EOS_30_min$horizontalPosition=="132"),]

## Plot Data

Now that we have our data separated into the upstream and downstream data, let's
plot both of the data sets together. We want to create a plot of the measures of
Dissolved Oxygen from the two different sensors. 

First, let's identify the column names important for plotting - time and 
dissolved oxygen data:

    # One option is to view column names in the data frame
    colnames(waq_instantaneous)

    ##   [1] "domainID"                        "siteID"                         
    ##   [3] "horizontalPosition"              "verticalPosition"               
    ##   [5] "startDateTime"                   "endDateTime"                    
    ##   [7] "sensorDepth"                     "sensorDepthExpUncert"           
    ##   [9] "sensorDepthRangeQF"              "sensorDepthNullQF"              
    ##  [11] "sensorDepthGapQF"                "sensorDepthValidCalQF"          
    ##  [13] "sensorDepthSuspectCalQF"         "sensorDepthPersistQF"           
    ##  [15] "sensorDepthAlphaQF"              "sensorDepthBetaQF"              
    ##  [17] "sensorDepthFinalQF"              "sensorDepthFinalQFSciRvw"       
    ##  [19] "specificConductance"             "specificConductanceExpUncert"   
    ##  [21] "specificConductanceRangeQF"      "specificConductanceStepQF"      
    ##  [23] "specificConductanceNullQF"       "specificConductanceGapQF"       
    ##  [25] "specificConductanceSpikeQF"      "specificConductanceValidCalQF"  
    ##  [27] "specificCondSuspectCalQF"        "specificConductancePersistQF"   
    ##  [29] "specificConductanceAlphaQF"      "specificConductanceBetaQF"      
    ##  [31] "specificCondFinalQF"             "specificCondFinalQFSciRvw"      
    ##  [33] "dissolvedOxygen"                 "dissolvedOxygenExpUncert"       
    ##  [35] "dissolvedOxygenRangeQF"          "dissolvedOxygenStepQF"          
    ##  [37] "dissolvedOxygenNullQF"           "dissolvedOxygenGapQF"           
    ##  [39] "dissolvedOxygenSpikeQF"          "dissolvedOxygenValidCalQF"      
    ##  [41] "dissolvedOxygenSuspectCalQF"     "dissolvedOxygenPersistenceQF"   
    ##  [43] "dissolvedOxygenAlphaQF"          "dissolvedOxygenBetaQF"          
    ##  [45] "dissolvedOxygenFinalQF"          "dissolvedOxygenFinalQFSciRvw"   
    ##  [47] "dissolvedOxygenSaturation"       "dissolvedOxygenSatExpUncert"    
    ##  [49] "dissolvedOxygenSatRangeQF"       "dissolvedOxygenSatStepQF"       
    ##  [51] "dissolvedOxygenSatNullQF"        "dissolvedOxygenSatGapQF"        
    ##  [53] "dissolvedOxygenSatSpikeQF"       "dissolvedOxygenSatValidCalQF"   
    ##  [55] "dissOxygenSatSuspectCalQF"       "dissolvedOxygenSatPersistQF"    
    ##  [57] "dissolvedOxygenSatAlphaQF"       "dissolvedOxygenSatBetaQF"       
    ##  [59] "dissolvedOxygenSatFinalQF"       "dissolvedOxygenSatFinalQFSciRvw"
    ##  [61] "pH"                              "pHExpUncert"                    
    ##  [63] "pHRangeQF"                       "pHStepQF"                       
    ##  [65] "pHNullQF"                        "pHGapQF"                        
    ##  [67] "pHSpikeQF"                       "pHValidCalQF"                   
    ##  [69] "pHSuspectCalQF"                  "pHPersistenceQF"                
    ##  [71] "pHAlphaQF"                       "pHBetaQF"                       
    ##  [73] "pHFinalQF"                       "pHFinalQFSciRvw"                
    ##  [75] "chlorophyll"                     "chlorophyllExpUncert"           
    ##  [77] "chlorophyllRangeQF"              "chlorophyllStepQF"              
    ##  [79] "chlorophyllNullQF"               "chlorophyllGapQF"               
    ##  [81] "chlorophyllSpikeQF"              "chlorophyllValidCalQF"          
    ##  [83] "chlorophyllSuspectCalQF"         "chlorophyllPersistenceQF"       
    ##  [85] "chlorophyllAlphaQF"              "chlorophyllBetaQF"              
    ##  [87] "chlorophyllFinalQF"              "chlorophyllFinalQFSciRvw"       
    ##  [89] "turbidity"                       "turbidityExpUncert"             
    ##  [91] "turbidityRangeQF"                "turbidityStepQF"                
    ##  [93] "turbidityNullQF"                 "turbidityGapQF"                 
    ##  [95] "turbiditySpikeQF"                "turbidityValidCalQF"            
    ##  [97] "turbiditySuspectCalQF"           "turbidityPersistenceQF"         
    ##  [99] "turbidityAlphaQF"                "turbidityBetaQF"                
    ## [101] "turbidityFinalQF"                "turbidityFinalQFSciRvw"         
    ## [103] "fDOM"                            "rawCalibratedfDOM"              
    ## [105] "fDOMExpUncert"                   "fDOMRangeQF"                    
    ## [107] "fDOMStepQF"                      "fDOMNullQF"                     
    ## [109] "fDOMGapQF"                       "fDOMSpikeQF"                    
    ## [111] "fDOMValidCalQF"                  "fDOMSuspectCalQF"               
    ## [113] "fDOMPersistenceQF"               "fDOMAlphaQF"                    
    ## [115] "fDOMBetaQF"                      "fDOMTempQF"                     
    ## [117] "fDOMAbsQF"                       "fDOMFinalQF"                    
    ## [119] "fDOMFinalQFSciRvw"               "buoyNAFlag"                     
    ## [121] "spectrumCount"                   "publicationDate"                
    ## [123] "release"

    # Alternatively, view the variables object corresponding to the data product for more information
    View(variables_20288)
Quite a few columns in the water quality data product!

The time column we'll consider for instrumented systems is `endDateTime` 
because it approximately represents data within the interval on or before the 
`endDateTime` time stamp. Timestamp column choice matters for time-aggregated 
datasets, but should not matter for instantaneous data such as water quality.

When interpreting data, keep in mind NEON timestamps are always in UTC.

The data column we would like to plot is labeled `dissolvedOxygen`.


    # plot
    wqual <- ggplot() +
    	geom_line(data = waq_up, 
    	          aes(endDateTime, dissolvedOxygen,color="a"), 
    	          na.rm=TRUE ) +
    	geom_line(data = waq_down, 
    	          aes(endDateTime, dissolvedOxygen, color="b"), 
    	          na.rm=TRUE) +
    	geom_line(na.rm = TRUE) +
    	ylim(0, 20) + ylab("Dissolved Oxygen (mg/L)") +
    	xlab(" ") +
      scale_color_manual(values = c("blue","red"),
                         labels = c("upstream","downstream")) +
      labs(colour = "") + # Remove legend title
      theme(legend.position = "top") +
      ggtitle("PRIN Upstream and Downstream DO")
      
      
    
    wqual

![Line plot of dissolved oxygen in mg/L measured at the upstream(blue) and downstream(red) stations of the Pringle Creek site.](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/rfigs/plot-wqual-1.png)


Now let's try plotting fDOM. fDOM is only measured at the downstream location.
NEON also provides uncertainty values for each measurement. Let's also consider 
measurement uncertainty in the plot.

The data columns we would like to plot are labeled `fDOM` and `fDOMExpUncert`.


    # plot
    fdomUcert <- ggplot() +
    	geom_line(data = waq_down, 
    	          aes(endDateTime, fDOM), 
    	          na.rm=TRUE, color="orange") +
      geom_ribbon(data=waq_down, 
                  aes(x=endDateTime, 
                      ymin = (fDOM - fDOMExpUncert), 
                      ymax = (fDOM + fDOMExpUncert)), 
                  alpha = 0.4, fill = "grey75") +
    	geom_line( na.rm = TRUE) +
    	ylim(0, 200) + ylab("fDOM (QSU)") +
    	xlab(" ") +
      ggtitle("PRIN Downstream fDOM with Expected Uncertainty Bounds") 
    
    fdomUcert

![Line plot of fDOM(QSU) with expected uncertainty from the downstream station of Pringle Creek.](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/rfigs/plot-fdom-ucert-1.png)

<div id="ds-challenge" markdown="1">
### Challenge: Plot Nitrate in Surface Water Data

Using what you've learned above, identify horizontal postions and column names
for nitrate in surface water.

</div>


    # recall dataframes created in list2env() command, including NSW_15_minute
    
    # which sensor locations?
    unique(NSW_15_minute$horizontalPosition)
    
    # what is the column name of the data stream of interest?
    names(NSW_15_minute)

Using what you've learned above, plot nitrate in surface water.


    # plot
    plot_NSW <- ggplot(data = NSW_15_minute,
                       aes(endDateTime, surfWaterNitrateMean)) +
                       geom_line(na.rm=TRUE, color="blue") + 
                       ylab("NO3-N (uM)") + xlab(" ") +
                       ggtitle("PRIN Downstream Nitrate in Surface Water")
    
    plot_NSW

![Nitrate in surface water(uM) from the downstream station of Pringle Creek. Note that there is missing data from February 18th through February 24th.](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/rfigs/challenge-plot-nsw-1.png)
## Examine Quality Flagged Data

Data product quality flags fall under two distinct types:

* Automated quality flags, e.g. range, spike, step, null
* Manual science review quality flag

In instantaneous data such as water quality DP1.20288.001,
the quality flag columns are denoted with "QF".

In time-averaged data, most quality flags have been aggregated into
quality metrics, with column names denoted with "QM" representing
the fraction of flagged points within the time averaging window.


    waq_qf_names <- names(waq_down)[grep("QF", names(waq_down))]
    
    print(paste0("Total columns in DP1.20288.001 expanded package = ", 
                 as.character(length(waq_qf_names))))

    ## [1] "Total columns in DP1.20288.001 expanded package = 96"

    # water quality has 96 data columns with QF in the name, 
    # so let's just look at those corresponding to fDOM
    print("fDOM columns in DP1.20288.001 expanded package:")

    ## [1] "fDOM columns in DP1.20288.001 expanded package:"

    print(waq_qf_names[grep("fDOM", waq_qf_names)])

    ##  [1] "fDOMRangeQF"       "fDOMStepQF"        "fDOMNullQF"        "fDOMGapQF"        
    ##  [5] "fDOMSpikeQF"       "fDOMValidCalQF"    "fDOMSuspectCalQF"  "fDOMPersistenceQF"
    ##  [9] "fDOMAlphaQF"       "fDOMBetaQF"        "fDOMTempQF"        "fDOMAbsQF"        
    ## [13] "fDOMFinalQF"       "fDOMFinalQFSciRvw"

A quality flag (QF) of 0 indicates a pass, 1 indicates a fail, and -1 indicates
a test that could not be performed. For example, a range test cannot be 
performed on missing measurements.

Detailed quality flags test results are all available in the 
`package = 'expanded'` setting we specified when calling 
`neonUtilities::loadByProduct()`. If we had specified `package = 'basic'`,
we wouldn't be able to investigate the detail in the type of data flag thrown. 
We would only see the FinalQF columns.

The `AlphaQF` and `BetaQF` represent aggregated results of various QF tests, 
and vary by a data  product's algorithm. In most cases, an observation's 
`AlphaQF = 1` indicates whether or not at least one QF was set to a value 
of 1, and an observation's `BetaQF = 1` indicates whether or not at least one 
QF was set to value of -1.

Note that fDOM has a couple other data-stream specific QFs beyond the standard 
quality flags. These are specific to the algorithms used to correct raw fDOM 
readings using temperature and absorbance per Watras et al. (2011) and 
Downing et al. (2012).

Let's consider what types of fDOM quality flags were thrown.


    waq_qf_names <- names(waq_down)[grep("QF", names(waq_down))]
    
    print(paste0("Total QF columns: ",length(waq_qf_names)))

    ## [1] "Total QF columns: 96"

    # water quality has 96 data columns with QF in the name, 
    # so let us just look at those corresponding to fDOM
    fdom_qf_names <- waq_qf_names[grep("fDOM",waq_qf_names)]
    
    for(col_nam in fdom_qf_names){
      print(paste0(col_nam, " unique values: ", 
                   paste0(unique(waq_down[,col_nam]), 
                          collapse = ", ")))
    }

    ## [1] "fDOMRangeQF unique values: 0, -1"
    ## [1] "fDOMStepQF unique values: 0, 1, -1"
    ## [1] "fDOMNullQF unique values: 0, 1"
    ## [1] "fDOMGapQF unique values: 0, 1"
    ## [1] "fDOMSpikeQF unique values: 0, -1, 1"
    ## [1] "fDOMValidCalQF unique values: 0"
    ## [1] "fDOMSuspectCalQF unique values: 0"
    ## [1] "fDOMPersistenceQF unique values: 0"
    ## [1] "fDOMAlphaQF unique values: 0, 1"
    ## [1] "fDOMBetaQF unique values: 0, 1"
    ## [1] "fDOMTempQF unique values: 0, 1, -1"
    ## [1] "fDOMAbsQF unique values: 0, -1, 1, 2"
    ## [1] "fDOMFinalQF unique values: 0, 1"
    ## [1] "fDOMFinalQFSciRvw unique values: NA"

QF values generally mean the following:

* 0: Quality test passed
* 1: Quality test failed
* -1: Quality test could not be run
* 2: A special case for `fDOMAbsQF`

So what does `fDOMAbsQF = 2` mean? The data product's variable descriptions may 
provide us some clues.

Recall we previously viewed the water quality variables object that comes 
with every NEON data download. Now let's print the description corresponding 
to the `fDOMAbsQF` field name.

    print(variables_20288$description[which(variables_20288$fieldName == "fDOMAbsQF")])

    ## [1] "Quality flag indicating that fDOM absorbance corrections were applied = 0; unable to be applied = 1; absorbance values were high = 2; calculated correction factor was 1 (i.e. no absorbance correction was made) = 3"

So whenever `fDOMAbsQF = 2`, the absorbance values coming from the 
SUNA (surface water nitrate sensor) were high.

Now let's consider the total number of flags generated for each quality test:


    # Loop across the fDOM QF column names. 
    #  Within each column, count the number of rows that equal '1'.
    print("FLAG TEST - COUNT")

    ## [1] "FLAG TEST - COUNT"

    for (col_nam in fdom_qf_names){
      totl_qf_in_col <- length(which(waq_down[,col_nam] == 1))
      print(paste0(col_nam,": ",totl_qf_in_col))
    }

    ## [1] "fDOMRangeQF: 0"
    ## [1] "fDOMStepQF: 770"
    ## [1] "fDOMNullQF: 233"
    ## [1] "fDOMGapQF: 218"
    ## [1] "fDOMSpikeQF: 71"
    ## [1] "fDOMValidCalQF: 0"
    ## [1] "fDOMSuspectCalQF: 0"
    ## [1] "fDOMPersistenceQF: 0"
    ## [1] "fDOMAlphaQF: 9997"
    ## [1] "fDOMBetaQF: 238"
    ## [1] "fDOMTempQF: 9"
    ## [1] "fDOMAbsQF: 9016"
    ## [1] "fDOMFinalQF: 9997"
    ## [1] "fDOMFinalQFSciRvw: 0"

    # Let's also check out how many fDOMAbsQF = 2 exist
    print(paste0("fDOMAbsQF = 2: ",
                 length(which(waq_down[,"fDOMAbsQF"] == 2))))

    ## [1] "fDOMAbsQF = 2: 210"

    print(paste0("Total fDOM observations: ", nrow(waq_down) ))

    ## [1] "Total fDOM observations: 41769"

Above lists the total fDOM QFs from a month of data at PRIN, as well as the
total number of observation data points in the data file.

We see a notably higher quantity of fDOMAbsQF relative to other quality flags.
Why is that? How do we know where to look?

The `variables_20228` included in the download would be a good place to start. 
Let's check the description for `fDOMAbsQF` again.



    print(variables_20288[which(variables_20288$fieldName == "fDOMAbsQF"),])

    ##                table fieldName
    ## 1: waq_instantaneous fDOMAbsQF
    ##                                                                                                                                                                                                              description
    ## 1: Quality flag indicating that fDOM absorbance corrections were applied = 0; unable to be applied = 1; absorbance values were high = 2; calculated correction factor was 1 (i.e. no absorbance correction was made) = 3
    ##          dataType units downloadPkg pubFormat primaryKey categoricalCodeName
    ## 1: signed integer  <NA>    expanded   integer       <NA>                  NA

So `fDOMAbsQF = 1` means fDOM absorbance corrections were unable to be applied.

For specific details on the algorithms used to create a data product and it's 
corresponding quality tests, it's best to first check the data product's 
Algorithm Theoretical Basis Document (ATBD). For water quality, that is 
NEON.DOC.004931 listed as Documentation references in the README file and the 
data product's web page.

Are there any manual science review quality flags? If so, the explanation for 
flagging may also be viewed in the data product's README file or in the data 
product's web page on NEON's data portal.

## Filtering (Some) Quality Flagged Observations

A simple approach to removing quality flagged observations is to remove data 
when the finalQF is raised. Let's view a plotting example using fDOM:

    # Map QF label names forthe plot for the fDOMFinalQF grouping
    group_labels <- c("fDOMFinalQF = 0", "fDOMFinalQF = 1")
    names(group_labels) <- c("0","1")
    
    # Plot fDOM data, grouping by the fDOMFinalQF value
    ggplot2::ggplot(data = waq_down, 
                    aes(x = endDateTime, y = fDOM, group = fDOMFinalQF)) +
      ggplot2::geom_step() +
      facet_grid(fDOMFinalQF ~ ., 
                 labeller = labeller(fDOMFinalQF = group_labels)) +
      ggplot2::ggtitle("PRIN Sensor Set 102 fDOM final QF comparison")

![Line plots of fDOM data that received a quality flag of zero(top), and data that received a data quality flag of one(bottom). Note how the bottom plot has many spikes in the data and were appropriately given a flag value of one.](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/rfigs/omit-finalqf-1.png)

The top panel corresponding to `fDOMFinalQF = 0` represents all fDOM data 
that were not flagged.  Conversely, the `fDOMFinalQF = 1` represents all
flagged fDOM data. Clearly, many spikes look like they were appropriately 
flagged. However, some flagged data look like they could be useful, such as 
the 2020 February 18-February 24 time range.

Let's inspect the quality flags during that time.

    # Find row indices around February 22:
    idxs_Feb22 <- base::which(waq_down$endDateTime > as.POSIXct("2020-02-22"))[1:1440]
    
    print("FLAG TEST - COUNT")

    ## [1] "FLAG TEST - COUNT"

    for (col_nam in fdom_qf_names){
      totl_qf_in_col <- length(which(waq_down[idxs_Feb22,col_nam] == 1))
      print(paste0(col_nam,": ",totl_qf_in_col))
    }

    ## [1] "fDOMRangeQF: 0"
    ## [1] "fDOMStepQF: 8"
    ## [1] "fDOMNullQF: 0"
    ## [1] "fDOMGapQF: 0"
    ## [1] "fDOMSpikeQF: 0"
    ## [1] "fDOMValidCalQF: 0"
    ## [1] "fDOMSuspectCalQF: 0"
    ## [1] "fDOMPersistenceQF: 0"
    ## [1] "fDOMAlphaQF: 1440"
    ## [1] "fDOMBetaQF: 0"
    ## [1] "fDOMTempQF: 0"
    ## [1] "fDOMAbsQF: 1440"
    ## [1] "fDOMFinalQF: 1440"
    ## [1] "fDOMFinalQFSciRvw: 0"

Looks like all Feb 22, 2020 data were flagged with `fDOMAbsQF`, with a few 
step test quality flags as well.

Let's take a closer look at each `fDOMAbsQF` flag value by grouping data based 
on each `fDOMAbsQF` value:


    ggplot2::ggplot(data = waq_down, 
                    aes(x = endDateTime, y = fDOM, group = fDOMAbsQF)) +
      ggplot2::geom_step() +
      facet_grid(fDOMAbsQF ~ .) +
      ggplot2::ggtitle("PRIN Sensor Set 102 fDOMAbsQF comparison")

    ## Warning: Removed 233 row(s) containing missing values (geom_path).

![Line plots of fDOM absorbance quality flag (fDOMAbsQF) values that received a quality flag of -1,0,1, and 2.  Note how the plot corresponding to values flagged as 1 correspond to the same time frame of data missing in the surface water nitrate generated earlier.](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/rfigs/view-absQF-1.png)

The `fDOMAbsQF = 1` is the most common quality flag from any single test. 
This means the absorbance correction could not be applied to fDOM data. This 
absorbance test also causes the final quality flag test fail, but some users 
may wish to ignore the absorbance quality test entirely.

Note the `fDOMAbsQF = 1` time frame corresponds to the missing surface water
nitrate data, as shown in the surface water nitrate plot we generated earlier.
Here is a reminder of our nitrate data:

    plot_NSW

![Nitrate in surface water(uM) from the downstream station of Pringle Creek. Note that there is missing data from February 18th through February 24th.](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/rfigs/look-at-n03-again-1.png)

Some types of automated quality flags may be worth ignoring. Rather than use 
the `FinalQF` column to omit any quality flags, let's create a custom final 
quality flag by ignoring the`fDOMAbsQF` column, allowing us omit quality 
flagged fDOM data regardless of absorbance correction status.


    # Remove the absorbance and aggregated quality flag tests from list of fDOM QF tests:
    fdom_qf_non_abs_names <- fdom_qf_names[which(!fdom_qf_names %in% c("fDOMAlphaQF","fDOMBetaQF","fDOMAbsQF","fDOMFinalQF"))]
    
    # Create a custom quality flag column as the maximum QF value within each row
    waq_down$aggr_non_abs_QF <- apply( waq_down[,fdom_qf_non_abs_names],1,max, na.rm = TRUE)
    # The 'apply' function above allows us avoid a for-loop and more efficiently 
    #  iterate over each row.
    
    # Plot fDOM data, grouping by the custom quality flag column's value
    ggplot2::ggplot(data = waq_down, 
                    aes(x = endDateTime, y = fDOM, 
                        group = aggr_non_abs_QF)) +
      ggplot2::geom_step() +
      facet_grid(aggr_non_abs_QF ~ .) +
      ggplot2::ggtitle("PRIN Sensor Set 102 fDOM custom QF aggregation")

![Line plots of fDOM data where a custom quality flag has been generated by omitting the fDOMAbsQF. Note the increase in available data using the custom quality flag aggregation that ignored fDOMAbsQF.](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/rfigs/ignore-absQF-1.png)

Using the custom quality flag aggregation that ignored `fDOMAbsQF`, the 
aggregated `aggr_non_abs_QF` column we created increases the quantity of 
data that could be used for further analyses.

Note that the automated quality flag algorithms are not perfect, and a few 
suspect data points may occasionally pass the quality tests.

## Data Aggregation

Sensor data users commonly wish to aggregate data such that time stamps match 
across two different datasets. In the following example, we will show how to
combine elevation of surface water (DP1.20016.001) and water quality (DP1.20288.001)
data products into a single dataframe.

Water quality is published as an instantaneous record, which should be every
minute at non-buoy sites such as PRIN. We know a data product does not come 
from the buoy if the HOR location is different from `"103"`. Because elevation 
of surface water is already aggregated to 30-minute intervals, we want to 
aggregate the water quality data product to 30-minute intervals as well.

At PRIN in February 2020, the elevation of surface water sensor is co-located 
with the water quality sonde at `horizontalPosition = "102"`, meaning the 
downstream sensor set. In this lesson, let's ignore the upstream data at 
HOR 101 and just aggregate water quality's downstream data from HOR 102.

Data can easily be aggregated in different forms, such as the mean, min, max,
and sum. In the following code chunk, we'll aggregate the data values to 15 
minutes as a mean, and the finalQF values as a max between 0 and 1. More complex
functions may be needed for aggregating other types of data, such as the 
measurement uncertainty or special, non-binary quality flags like `fDOMAbsQF`.


    # Recall we already created the downstream object for water quality, waq_down
    
    # We first need to name each data stream within water quality. 
    # One trick is to find all the variable names by searching for "BetaQF"
    waq_strm_betaqf_cols <- names(waq_down)[grep("BetaQF",names(waq_down))]
    print(paste0("BetaQF column names: ",
                 paste0(waq_strm_betaqf_cols, collapse = ", ")))

    ## [1] "BetaQF column names: sensorDepthBetaQF, specificConductanceBetaQF, dissolvedOxygenBetaQF, dissolvedOxygenSatBetaQF, pHBetaQF, chlorophyllBetaQF, turbidityBetaQF, fDOMBetaQF"

    # Now let's remove the BetaQF from the column name:
    waq_strm_cols <- base::gsub("BetaQF","",waq_strm_betaqf_cols)
    # To keep column names short, some variable names had to be shortened
    # when appending "BetaQF", so let's add "uration" to "dissolvedOxygenSat"
    waq_strm_cols <- base::gsub("dissolvedOxygenSat",
                                "dissolvedOxygenSaturation",waq_strm_cols)
    print(paste0("Water quality sensor data stream names: ", 
                 paste0(waq_strm_cols, collapse = ", ")))

    ## [1] "Water quality sensor data stream names: sensorDepth, specificConductance, dissolvedOxygen, dissolvedOxygenSaturation, pH, chlorophyll, turbidity, fDOM"

    # We will also aggregate the final quality flags:
    waq_final_qf_cols <- names(waq_down)[grep("FinalQF",names(waq_down))]
    
    # Let's check to make sure our time column is in POSIXct format, which is 
    # needed if you download and read-in NEON data files without using the 
    # neonUtilities package.
    if("POSIXct" %in% class(waq_down$endDateTime)){
      print("Time column in waq_down is appropriately in POSIXct format")
    } else {
      print("Converting waq_down endDateTime column to POSIXct")
      waq_down$endDateTime <- as.POSIXct(waq_down$endDateTime, tz = "UTC")
    }

    ## [1] "Time column in waq_down is appropriately in POSIXct format"
Now that we have the column names of data and quality flags which we wish to
aggregate, we can now move to the aggregation steps! We're going to use some 
more advanced features using the `dplyr` and `padr` packages. Instead of looping 
by each column, let's employ the `dplyr` pipe operator, `%>%`, and call a 
function that acts on each data column of interest, which we've determined above.


    # Aggregate water quality data columns to 30 minute intervals, 
    # taking the mean of non-NA values within each 30-minute period. 
    # We explain each step in the dplyr piping operation in code 
    # comments:
    
    waq_30min_down <- waq_down %>% 
                  # pass the downstream data frame to the next function
                  # padr's thicken function adds a new column, roundedTime, 
                  # that shows the closest 30 min timestamp to
                  # to a given observation in time
      
                  padr::thicken(interval = "30 min",
                                by = "endDateTime",
                                colname = "roundedTime",
                                rounding = "down") %>%
                  # In 1-min data, there should now be sets of 30 
                  # corresponding to each 30-minute roundedTime
                  # We use dplyr to group data by unique roundedTime 
                  # values, and summarise each 30-min group
                  # by the the mean, for all data columns provided 
                  # in waq_strm_cols and waq_final_qf_cols
      
                  dplyr::group_by(roundedTime) %>% 
                    dplyr::summarise_at(vars(dplyr::all_of(c(waq_strm_cols, 
                                                      waq_final_qf_cols))), 
                                        mean, na.rm = TRUE)
    
    # Rather than binary values, quality flags are more like "quality 
    # metrics", defining the fraction of data flagged within an 
    # aggregation interval.
Now, we have a new dataframe for water quality data and associated final quality
flags aggregated to 30 minute time intervals. Now the downstream water quality 
data may be easily combined with the nearby, albeit non-co-located downstream 
30-minute averaged elevation of surface water data.

The following code chunk merges the data:


    # We have to specify the matching column from each dataframe
    all_30min_data_down <- base::merge(x = waq_30min_down, 
                                       y = eos_down, 
                                       by.x = "roundedTime", 
                                       by.y = "endDateTime")
    
    # Let's take a peek at the combined data frame's column names:
    colnames(all_30min_data_down)

    ##  [1] "roundedTime"                     "sensorDepth"                    
    ##  [3] "specificConductance"             "dissolvedOxygen"                
    ##  [5] "dissolvedOxygenSaturation"       "pH"                             
    ##  [7] "chlorophyll"                     "turbidity"                      
    ##  [9] "fDOM"                            "sensorDepthFinalQF"             
    ## [11] "sensorDepthFinalQFSciRvw"        "specificCondFinalQF"            
    ## [13] "specificCondFinalQFSciRvw"       "dissolvedOxygenFinalQF"         
    ## [15] "dissolvedOxygenFinalQFSciRvw"    "dissolvedOxygenSatFinalQF"      
    ## [17] "dissolvedOxygenSatFinalQFSciRvw" "pHFinalQF"                      
    ## [19] "pHFinalQFSciRvw"                 "chlorophyllFinalQF"             
    ## [21] "chlorophyllFinalQFSciRvw"        "turbidityFinalQF"               
    ## [23] "turbidityFinalQFSciRvw"          "fDOMFinalQF"                    
    ## [25] "fDOMFinalQFSciRvw"               "domainID"                       
    ## [27] "siteID"                          "horizontalPosition"             
    ## [29] "verticalPosition"                "startDateTime"                  
    ## [31] "surfacewaterElevMean"            "surfacewaterElevMinimum"        
    ## [33] "surfacewaterElevMaximum"         "surfacewaterElevVariance"       
    ## [35] "surfacewaterElevNumPts"          "surfacewaterElevExpUncert"      
    ## [37] "surfacewaterElevStdErMean"       "sWatElevRangeFailQM"            
    ## [39] "sWatElevRangePassQM"             "sWatElevRangeNAQM"              
    ## [41] "sWatElevPersistenceFailQM"       "sWatElevPersistencePassQM"      
    ## [43] "sWatElevPersistenceNAQM"         "sWatElevStepFailQM"             
    ## [45] "sWatElevStepPassQM"              "sWatElevStepNAQM"               
    ## [47] "sWatElevNullFailQM"              "sWatElevNullPassQM"             
    ## [49] "sWatElevNullNAQM"                "sWatElevGapFailQM"              
    ## [51] "sWatElevGapPassQM"               "sWatElevGapNAQM"                
    ## [53] "sWatElevSpikeFailQM"             "sWatElevSpikePassQM"            
    ## [55] "sWatElevSpikeNAQM"               "validCalFailQM"                 
    ## [57] "validCalPassQM"                  "validCalNAQM"                   
    ## [59] "sWatElevAlphaQM"                 "sWatElevBetaQM"                 
    ## [61] "sWatElevFinalQF"                 "sWatElevFinalQFSciRvw"          
    ## [63] "publicationDate"                 "release"
We now have matching time stamps for water quality and any other 30-minute  
averaged data product, such as elevation of surface water. The merged data 
frame facilitates direct comparison across different sensors.

Let's take a look with a plot of specific conductance versus water surface 
elevation:

    ggplot(data = all_30min_data_down, 
           aes(x = surfacewaterElevMean, y = specificConductance)) +
      geom_point() + 
      ggtitle("PRIN specific conductance vs. surface water elevation") + 
      xlab("Elevation [m ASL]") + 
      ylab("Specific conductance [uS/cm]")

    ## Warning: Removed 5 rows containing missing values (geom_point).

![Scatter plot of specific conductance (uS/cm) and elevation (m) from Pringle Creek. Specific conductance (uS/cm) is on the Y-axis and elevation (m) on the X-axis. A new data set of 30 minute aggregated water quality data was generated to match the measurement interval of surface water elevation.](https://raw.githubusercontent.com/NEONScience/NEON-Data-Skills/main/tutorials/R/AIS-data/AIS-QF-tutorial/download-NEON-AIS-data/rfigs/plot-eos-waq-1.png)

Aggregating high frequency time series data is a useful tool for understanding 
relationships between variables collected at different time intervals, and may
also be a required format for model inputs. 

Now that you have the basic tools and knowledge on how to read and wrangle NEON
AIS data, go have fun working on your scientific questions!

## Citations

Watras, C. J., Hanson, P. C., Stacy, T. L., Morrison, K. M., Mather, J., Hu, Y. 
H., & Milewski, P. (2011). A temperature compensation method for CDOM 
fluorescence sensors in freshwater. Limnology and Oceanography: Methods, 9(7), 
296-301.

Downing, B. D., Pellerin, B. A., Bergamaschi, B. A., Saraceno, J. F., & Kraus, T.
E. (2012). Seeing the light: The effects of particles, dissolved materials, and 
temperature on in situ measurements of DOM fluorescence in rivers and streams. 
Limnology and Oceanography: Methods, 10(10), 767-775.
