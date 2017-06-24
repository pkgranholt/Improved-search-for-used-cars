# Introduction

The idea for this capstone project came from a situation I had a few years ago when I was buying a used car. I spent a lot of time researching, finding the best car according to several criteria, among them safety and reliability. This information was available online, but it annoyed me that the web site I was using to search for cars didn’t have this information readily available. It was therefore a time-consuming process of finding certain brands and models that was suitable, and looking these up. This project aims to solve that problem, by attaching crash test data and reliability data to a large data set with used cars ads.

In order to do this, three data sets need to be stitched together. The data sets contain used car listings, crash test data and reliability data. Each present its own challenge in the wrangling process, and they need to be made uniform for the data sets to compare car models to car models.

A big thanks to my advisor for this project, Branko Kovac!

## The data sets and wrangling

There are three sources for this project:  

1. [Ebay-Klenanzeigen](https://www.kaggle.com/orgesleka/used-cars-database) is a scraping of the German version of Ebay with about 371 000 cars for sale. This will be used as the basis for the search recommendations.

The biggest challenge with this data is the size. There are many listings, but not all are relevant. One of the challenges is filtering out the listings that have information that is not meant to be searchable. For instance, quite a few cars have exorbitant prices, that when you read the text in the listing, are there to symbolize that the owner wants to trade the car for another car. This type of information needs to be cleaned or removed for the search parameters to be meaningful.

This data set has the most detail on the cars, down to mileage, production year, the specific model and features for each car. Not all of this information is possible to match to the more aggregated data of the other data sets, and therefore needs to be aggregated to some degree. Ideally, I still want to keep as much information as possible for all the entries, as this will return more meaningful search results.

This data set is simply called "auto" (the German word for car) in the R code.

2. [EURONCAP](https://www.adac.de/infotestrat/tests/crash-test/alletests.aspx) is a European car safety performance assessment programme that tests cars in crash tests. This link is from the German automobile club ADAC, which is the largest automobile club in Europe. They present the EURONCAP data in a more readily available format than EURONCAP themselves.

The biggest challenge here was that the data was stored in pictures. The pictures showed how many stars each car had attained. To create a viable data set from this information, I took the html-code for the website, ran it through some code to extract the table data, including the part of the picture names that indicated how many stars the picture showed (think "3_stars.img", where just the number 3 is extracted).

After that, it was just a question of dividing up the name-column into car brands and car models successfully.

This data set is called "crash" in the R code.

3. [TÜV auto reports](http://www.anusedcar.com) is a source that not only provides data on reliability across different brands, but also different models within each brand. The report is published each year, with different age brackets for the cars.

The biggest challenge in this data set is how the data is aggregated; the ages for the cars are grouped together two-and-two years. So for the 2017 report, the newest cars are in the 2-3 year old bracket. The next group is 4-5 years, and so on. This may have been done to create a simpler data set (I have tried to get in touch with them), but the grouping artifact is an issue when it comes to actually evaluating the different years independently.

To fix this, I've averaged the years that fall in two reports (for instance a car from 2014 is in the "2-3 year old car"-group in both 2016 and 2017). By doing this, the adjusted 2017 report is able to give different reliability data for each year, as a proxy for the actual data that is not available.

Data wrangling-wise, the best way of aggregating this data was to create columns with the dplyr package in R that are easily able to both identify where there are enough data to create the means, and to actually create them as a separate variable. Then the variable was inserted into the "pure" reliability data as a proxy to break up the two consecutive years with identical data

This data set is called "rel" (short for reliability) in the R code.

## About the data fields

The data sets all have information on car names. Only the biggest one have separate fields for car brands and models. Therefore this needs to be created for the other two data sets. There are a few naming conventions that need to be made similar, but apart from that, it's a simple enough process.

In addition, the difference in degree of aggregation means that it is vital to create a common level of detail for the data sets. For instance, what is described as a "Mercedes Benz E-Class Coupe" may merely be called "Mercedes E-Class" in another data set. The "coupe"-detail is lost when we reduce the data sets to the smallest common denominators.

The biggest data set has about 371 000 rows of used cars listings, but only a smaller subset will be used for this project. This is done for two reasons; to use just the data that has complete information for the variables that needs to be searchable, and because the larger data set have older and newer cars than the other two data sets, which is not helpful in this project. The cars whose age are only found in the larger data set is therefore removed. For instance, I don't have crash test data from 1991, therefore I've chosen not to include the cars that are that old.

A lot of the data fields in the listing-data set were not useful for this project, and were rejected. These include the number of pictures for each listing (which erroneously was set to 0 for all listings in the data set) and several meta-data fields that aren't relevant for this project.

All the data sets are in German, and a few issues pertaining to translations and measurements have come up. One is the issue with PS/Pferdstärke vs. the more common BHP/break horse power. After checking with several car enthusiasts, I've concluded that although these two are not technically equal, they are often used interchangeably. I've therefore decided to do so also. This has the benefit of making all of the PS-fields integers and "sensible", whereas a transformation of this variable would make it a little strange when searching (200 PS sounds better than 197.2 BHP).

I think it's also worth noting that the crash test criteria has changed a lot over the years. Anecdotally, I would compare a five-star rating in 2002 to a three-star rating in 2017. This is due to the ever-evolving new requirements for getting a top score, which includes new technology like brake assistance or lane-change warnings. I've intentionally kept the original ratings, as I believe that is more intuitive, but it could be interesting to include devaluations of the older ratings in a future project.

## Limitations

Having to rely on the smallest common denominator from three different data sets means there have to be some concessions. For one, the brands and models are possible to identify for the most part, but features are not. As one can imagine, the same car with different features may behave differently in both crash tests and reliability. A top-specced model typically has extra features, which may include some safety features. Therefore, the ratings should be thought of as a sort of "mean" for each of the car models within years they are tested for.

Secondly, the listing-data set is cut in half after cleaning, most of which could have been usable if the other data sets where more complete. The oldest reliability data is from 2002, where the crash test data is from about 2000 for most models (some have data before this, depending to the launch date of the model).

I think it's also important to recognize that this project will only help you identify which cars fall within which safety/reliability-rating, it will not help you decide on which car is right for you. Purchasing a car is, I believe at least partially an emotional action. If you are like me, and crave more information pertaining to the cars listed for sale, then this project may be of help. If you simply want to find a car you will like, you may be better helped by actually test driving the different cars.

# Data cleaning and wrangling

## Auto data set

By far the biggest of the three data sets, the auto data set is also the most structured one. information like price, brand and model are already created, and need only be cleaned.

We start off by loading the necessary libraries for the cleaning and wrangling.

```r
library(data.table)
library(dplyr)
library(tidyr)
```

Then we import and start cleaning the data set.

```r
auto <- fread(input = 'autos.csv')
```

What type of sellers are there, and how many of each are there?

```r
table(auto$seller)

gewerblich     privat
         3     371821
```

"Gewerblich" means commercial. For this project, I only want private sellers, so the commercial ads are removed.

```r
auto <- auto[auto$seller == 'privat']
auto$seller <- NULL
```

Let's take a closer look at the number of pictures for each ad.

```r
table(auto$nrOfPictures)

0
371821
```

There are no data in this column, so it is deleted.

```r
auto$nrOfPictures <- NULL
```

People looking for cars (eg. not cars that are for sale) needs to be taken out of the data set.

```r
auto <- auto[auto$offerType == 'Angebot']
auto$offerType <- NULL
```

Cars that are advertised as either very expensive or very inexpensive seem to be priced incorrectly most times, but offers for car-for-car trades or other advertisements where the price is incorrect. These are therefore removed. This removes many ads with good information, but they are for cars that are quite expensive, considering these are used cars. In the end, I feel this is a worthwhile trade-off. On the lower end of the spectrum, there are cars for sale that are only to be used for parts, and some cars for lease. Setting the floor at 101 euro means a lot of these are filtered out. There are still some cars for lease, which is fine.

```r
auto <- auto[price < 50000 & price > 100]
```

Note that the data set has no information on whether the prices are stated as "VB" which stands for Verhandlungsbasis (negotiable) or not, which could have been interesting as not all advertisements include a price with "VB".

The variable 'abtest' seems to be linked to some sort av A/B-testing that may or may not be linked to the website rather than the advertisements. This won't be used, so it is deleted. Several of the variables with metadata will not be used and they are removed.

```r
auto$abtest              <- NULL
auto$dateCrawled         <- NULL
auto$lastSeen            <- NULL
auto$dateCreated         <- NULL
auto$monthOfRegistration <- NULL
auto$postalCode          <- NULL
```

Cars advertised with an extremely powerful motor may not be accurately described in the ads. To control for this, cars that supposedly have more than 900 PS (about the same as 900 BHP) are removed. The same seems to be true for cars with 25 PS or less.

```r
auto <- auto[powerPS <= 900 & powerPS > 25]
```

The ads need to be connected by production year and brand/model. If this information is not available, we'll drop the row.

```r
table(is.na(auto$brand))

FALSE
319899

table(is.na(auto$model))

FALSE
319899
```

No empty values for either brand or model.

The production year for the reliability data is between 2002 and 2015 therefore cars that are newer or older than this are removed we'll assume that registration year from 'auto' and production year from 'rel' are the same, though this may not necessarily be true.

```r
auto <- auto[yearOfRegistration >= 2002 & yearOfRegistration <= 2015]
```

Now we come to some cleaning in the brand and model names. Some of these are pure substitutions, some are substitutions depending on what information is stored in the variables. There are also several examples of German wording that is removed, and not given English wording, to make it easier to read and match with the other data sets. An example would be "BMW 5er", which in English would be a BMW 5-series. This will now be referred to as simply "BMW 5". I chose this naming convention to make it easy to remember and match across the three different data sets. Often the headline of the ad has more information on the specific car model ("BMW 520d Touring edition", for example).

```r
auto$brand <- ifelse(auto$brand == 'mercedes_benz', 'mercedes', auto$brand)
auto$brand <- ifelse(auto$brand == 'alfa_romeo', 'alfa romeo',  auto$brand)
auto$brand <- ifelse(auto$brand == 'land_rover', 'land rover',  auto$brand)
auto$model <- ifelse(auto$model == 'rangerover', 'range rover', auto$model)
auto$model <- gsub('_', ' ',      auto$model)
auto$model <- sub(' reihe', '',   auto$model)
auto$model <- sub(' klasse', '',  auto$model)
auto$model <- sub('1er', '1',     auto$model)
auto$model <- sub('3er', '3',     auto$model)
auto$model <- sub('5er', '5',     auto$model)
auto$model <- sub('6er', '6',     auto$model)
auto$model <- sub('7er', '7',     auto$model)
auto$model <- sub('mx', 'mx-5',   auto$model)
auto$model <- sub(' max', '-max', auto$model)
```

Then there are some pure translations. Some of the wording here is chosen to make it as ubiquitous as possible. Note in particular the "mini-van" category, that includes mini-busses, a naming convention that may not be used in some regions. Also note that "limousine" does not translate to limousine. :)

```r
auto$notRepairedDamage <- sub('ja', 'yes',  auto$notRepairedDamage)
auto$notRepairedDamage <- sub('nein', 'no', auto$notRepairedDamage)

auto$gearbox <- sub('automatik', 'automatic', auto$gearbox)
auto$gearbox <- sub('manuell', 'manual',      auto$gearbox)

auto$fuelType <- sub('benzin', 'petrol',              auto$fuelType)
auto$fuelType <- sub('elektro', 'electric',           auto$fuelType)
auto$fuelType <- sub('lpg', 'liquefied petroleum',    auto$fuelType)
auto$fuelType <- sub('cng', 'compressed natural gas', auto$fuelType)

auto$vehicleType <- sub('cabrio', 'convertible',   auto$vehicleType)
auto$vehicleType <- sub('kombi', 'station wagon',  auto$vehicleType)
auto$vehicleType <- sub('kleinwagen', 'hatchback', auto$vehicleType)
auto$vehicleType <- sub('bus', 'mini-van',         auto$vehicleType)
auto$vehicleType <- sub('limousine', 'sedan',      auto$vehicleType)
```

Finally, fields with no data or "other"-categories are removed, and the variables are converted to factors.

```r
auto <- auto[brand != 'sonstige_autos']
auto <- auto[fuelType != 'andere']
auto <- auto[vehicleType != 'andere']
auto <- auto[vehicleType != '']
auto <- auto[gearbox != '']
auto <- auto[fuelType != '']
auto <- auto[notRepairedDamage != '']

auto$vehicleType       <- as.factor(auto$vehicleType)
auto$gearbox           <- as.factor(auto$gearbox)
auto$fuelType          <- as.factor(auto$fuelType)
auto$notRepairedDamage <- as.factor(auto$notRepairedDamage)
auto$brand             <- as.factor(auto$brand)
auto$model             <- as.factor(auto$model)
```

## Reliability data set

This data set was created by stitching together several lists. The data is already quite clean, but it will need some cleaning to make processing possible. For this data set, we are still using the libraries from the auto data set. We start off by importing the data.

```r
rel <- fread('reliability.csv')
```

Then we remove non-number characters from the strings that contain the number values, and transform these to numbers.

```r
rel$mileage <- sub(' km', '', rel$mileage)
rel$mileage <- sub(' '  , '', rel$mileage)
rel$mileage <-     as.integer(rel$mileage)

rel$fault_rate <- sub('%', '', rel$fault_rate)
rel$fault_rate <-   as.numeric(rel$fault_rate)

rel$car_prod_y <- as.numeric(rel$car_prod_y)
```

Both car brand and car model are in the same column in the data set. This needs to be divided into separate columns. This split is not perfect, as it takes the first space, and puts everything before the space into the brand column, while everything after the first space is put into the model column. For an Alfa Romeo 147, this will be wrong, but I'll get to this a little later.

```r
rel <- rel %>%
  separate(car_make_model, c('brand', 'model'), extra = 'merge')
```

To make it easier to get an exact match, brands and models with special letters are converted to normal letters. All names are also put in lower case.

```r
rel$brand <- sub('Š', 's', rel$brand)
rel$brand <- sub('ë', 'e', rel$brand)
rel$model <- sub('é', 'e', rel$model)
rel$model <- sub('ó', 'o', rel$model)
rel$model <- sub('´', '',  rel$model)

rel$brand <- tolower(rel$brand)
rel$model <- tolower(rel$model)
```

Next up is cleaning the brand and model names. Alfa Romeo, Mercedes Benz and the different Mini versions needs to be corrected. Mini is in this data set referred to as BMW Mini. While that is technically correct, I will be considering Mini as its own brand.

```r
rel$brand <- ifelse(rel$brand == 'mercedes', 'mercedes benz', rel$brand)
rel$model <- ifelse(rel$brand == 'mercedes benz', sub('benz ', '', rel$model), rel$model)
rel$brand <- ifelse(rel$brand == 'mercedes benz', 'mercedes', rel$brand)

rel$brand <- ifelse(rel$brand == 'alfa', 'alfa romeo', rel$brand)
rel$model <- ifelse(rel$brand == 'alfa romeo', sub('romeo ', '', rel$model), rel$model)

rel$model <- ifelse(rel$brand == 'bmw' & rel$model == 'mini', 'cooper', rel$model)
rel$brand <- ifelse(rel$brand == 'bmw' & rel$model == 'cooper', 'mini', rel$brand)
rel$model <- ifelse(rel$brand == 'bmw' & rel$model == 'mini countryman', 'countryman', rel$model)
rel$brand <- ifelse(rel$brand == 'bmw' & rel$model == 'countryman', 'mini', rel$brand)
```

The next part was a little tricky. In the original data set, fault rates only changed every two years. But I want to differentiate between the fault rates every year. The way I did this, was to create means between the reports that overlapped. See the picture below for a visual representation of the data.

![Explanation transformation of means in the reliability data](https://user-images.githubusercontent.com/26480394/27224571-1f806a74-5296-11e7-8838-06324a3be37b.png)

Note the diamond-pattern in the "cleaned data with means"-portion. We will create this pattern in a bit.

A car produced in 2014, will be in the "2-3 year old cars"-category in both 2016 and 2017. I use this fact to create a mean between the reported fault rates of the two reports, but for the same production year of the car. This way, if one were to look up a specific brand and model of car that is either two or three year old in 2017, you would expect some differences between them in fault rates and mileages. I think this is important, because the grouping where the fault rate changes only every two years, despite the report being made every year, is likely an artifact of the way the data is gathered or grouped after collection. As such, it's not accurately describing reality, where one would expect fault rates to gradually increase, as opposed to suddenly making an upward jump every two years.

The downside to creating these means is twofold. Firstly, the original data is changed. Ideally I would like to have avoided this. But in this case I believe the benefit outweighs the downside of doing it. Secondly, if you look at how fault rates are represented across the different report years, you'll see that the same problem is still present, only this time it is across report years instead of vehicle production years. The fault rate for a 2014 car in the 2016 and 2017 reports are identical after the transformation. This is not an issue for my particular project, as I'm not concerned with comparing reports from different years. If you want to search for the fault rate for a car, you'll want the newest report, not how different reports historically have evaluated it differently.

We'll start with creating the means for the fault rates. Here the data is transformed so that each report year is a separate column. This is done twice to remove rows that only contains NA-values in the year-columns for the fault rate. This in effect compresses the data set by quite a bit.

```r
rel_fr <- rel %>%
  spread(report_year, fault_rate) %>%
  select(2:3,5:10) %>%
  gather(report_year, fault_rate, 4:8, na.rm = TRUE) %>%
  distinct(brand, model, car_prod_y, report_year, .keep_all = TRUE) %>%
  spread(report_year, fault_rate)
```

Then a variable is created for even production years. This will be used to create means in the diamond pattern from the illustration.

```r
even_fr <- (rel_fr$car_prod_y == '2002' | rel_fr$car_prod_y == '2004' | rel_fr$car_prod_y == '2006' |
            rel_fr$car_prod_y == '2008' | rel_fr$car_prod_y == '2010' | rel_fr$car_prod_y == '2012' |
            rel_fr$car_prod_y == '2014' )
```

Now we use the even_fr-vector to create the means and put this value into the correct columns. Note that this process is only done four times, as there are only four two-year combinations of sequential report years in the data set.

The code below does the following: First the mean between the two report years is created. Then, every time the lowest of the two sequential years is an even number, the mean is put into the fault rate column for both report years (feel free to double check with the illustration again to see that this makes sense). A similar thing is done when the lowest of the two sequential years is an odd number, but the insertion of the mean is "lagged" by one report year, so that we get the diamond pattern from the illustration.

```r
rel_fr$mean1617 <- rowMeans(rel_fr[,c("2016", "2017")], na.rm = TRUE)
rel_fr$'2016'[even_fr] <- rel_fr$mean1617[even_fr]
rel_fr$'2017'[even_fr] <- rel_fr$mean1617[even_fr]

rel_fr$mean1516 <- rowMeans(rel_fr[,c("2015", "2016")], na.rm = TRUE)
rel_fr$'2015'[!even_fr] <- rel_fr$mean1516[!even_fr]
rel_fr$'2016'[!even_fr] <- rel_fr$mean1516[!even_fr]

rel_fr$mean1415 <- rowMeans(rel_fr[,c("2014", "2015")], na.rm = TRUE)
rel_fr$'2014'[even_fr] <- rel_fr$mean1415[even_fr]
rel_fr$'2015'[even_fr] <- rel_fr$mean1415[even_fr]

rel_fr$mean1314 <- rowMeans(rel_fr[,c("2013", "2014")], na.rm = TRUE)
rel_fr$'2013'[!even_fr] <- rel_fr$mean1314[!even_fr]
rel_fr$'2014'[!even_fr] <- rel_fr$mean1314[!even_fr]
```

Finally we transform the data frame back to the initial format with report year and fault rate in their own columns.

```r
rel_fr <- rel_fr %>%
  select(1:8) %>%
  gather(report_year, fault_rate, 4:8, na.rm = TRUE)
```

The same is now done for the mileage-variable.

```r
rel_mil <- rel %>%
  spread(report_year, mileage) %>%
  select(2:3,5:10) %>%
  gather(report_year, mileage, 4:8, na.rm = TRUE) %>%
  distinct(brand, model, car_prod_y, report_year, .keep_all = TRUE) %>%
  spread(report_year, mileage)

even_mil <- (rel_mil$car_prod_y == '2002' | rel_mil$car_prod_y == '2004' | rel_mil$car_prod_y == '2006' |
             rel_mil$car_prod_y == '2008' | rel_mil$car_prod_y == '2010' | rel_mil$car_prod_y == '2012' |
             rel_mil$car_prod_y == '2014')

rel_mil$mean1617 <- rowMeans(rel_mil[,c("2016", "2017")], na.rm = TRUE)
rel_mil$'2016'[even_mil] <- rel_mil$mean1617[even_mil]
rel_mil$'2017'[even_mil] <- rel_mil$mean1617[even_mil]

rel_mil$mean1516 <- rowMeans(rel_mil[,c("2015", "2016")], na.rm = TRUE)
rel_mil$'2015'[!even_mil] <- rel_mil$mean1516[!even_mil]
rel_mil$'2016'[!even_mil] <- rel_mil$mean1516[!even_mil]

rel_mil$mean1415 <- rowMeans(rel_mil[,c("2014", "2015")], na.rm = TRUE)
rel_mil$'2014'[even_mil] <- rel_mil$mean1415[even_mil]
rel_mil$'2015'[even_mil] <- rel_mil$mean1415[even_mil]

rel_mil$mean1314 <- rowMeans(rel_mil[,c("2013", "2014")], na.rm = TRUE)
rel_mil$'2013'[!even_mil] <- rel_mil$mean1314[!even_mil]
rel_mil$'2014'[!even_mil] <- rel_mil$mean1314[!even_mil]

rel_mil <- rel_mil %>%
  select(1:8) %>%
  gather(report_year, mileage, 4:8, na.rm = TRUE)
```

Finally the processed fault rate and mileage are combined and saved in the reliability data set. The temporary data frames are deleted, car age is created as a variable and nationality markers are created for use in the exploratory analysis later.

```r
rel <- cbind(rel_fr, mileage = rel_mil$mileage)
rm(rel_fr, rel_mil, even_fr, even_mil)

rel$report_year <- as.numeric(rel$report_year)
rel$car_age <- rel$report_year - rel$car_prod_y

rel$nationality <- ifelse(rel$brand == 'audi'    | rel$brand == 'bmw'     | rel$brand == 'mercedes' |
                          rel$brand == 'opel'    | rel$brand == 'porsche' | rel$brand == 'volkswagen',
                          'german', 'others')

rel$nationality <- ifelse(rel$brand == 'citroen' | rel$brand == 'peugeot' | rel$brand == 'renault',
                          'french', rel$nationality)

rel$nationality <- ifelse(rel$brand == 'honda'   | rel$brand == 'mazda'   | rel$brand == 'mitsubishi' |
                          rel$brand == 'nissan'  | rel$brand == 'subaru'  | rel$brand == 'suzuki' |
                          rel$brand == 'toyota',
                          'japanese', rel$nationality)

rel$brand       <- as.factor(rel$brand)
rel$model       <- as.factor(rel$model)
rel$nationality <- as.factor(rel$nationality)
```

## Crash data set

This scraped data set has a variable that sometimes include several types of information, but after this is tidied up, the data set is fairly simple. First the data set is loaded and only the rows with useful information is read. The reason the file only has useful information on every 4th row has to do with some pre-processing that was done in Excel. In this case I used Excel to extract the useful information from the html-code of a website. Because of limited functionality in Excel (or my lack of deep Excel knowledge, whichever applies), the data could not easily be saved in a traditional format with useful information on every row.

```r
crash <- fread('crash.csv')

crash <- crash[seq(1, nrow(crash), 4),]
```

Next is splitting brand and model names into two separate columns, which will be cleaned shortly. Test date is removed as it will not be used.

```r
crash <- crash %>%
  separate(brand_model, c('brand', 'model'), extra = 'merge')

crash$test_date <- NULL
```

In the data set, there is one 0-star-rating. This is not a true rating, there seems to be a mistake in the source. As it's only for this one rating, all 0-values are turned into 4-star-ratings (the true rating of the mistaken 0-rating). The stars-rating is converted to integers for the summary-portion of the exploratory analysis.

```r
crash$stars <- sub('0', '4', crash$stars)

crash$stars = as.integer(crash$stars)
```

We do the same cleaning with brand names, special letters and unconventional naming conventions, that we did in the other data sets.

```r
crash$brand <- sub('ò', 'o', crash$brand)
crash$model <- sub('é', 'e', crash$model)
crash$model <- sub('`', '', crash$model)

crash$brand <- tolower(crash$brand)
crash$model <- tolower(crash$model)

crash$model <- sub('benz citan kombi', 'citan kombi', crash$model)
crash$brand <- sub('vw', 'volkswagen', crash$brand)
crash$brand <- sub('alfa', 'alfa romeo', crash$brand)
crash$model <- sub('1er', '1', crash$model)
crash$model <- sub('3er', '3', crash$model)
crash$model <- sub('5er', '5', crash$model)

crash$model <- ifelse(crash$brand == 'alfa romeo', sub('romeo ', '', crash$model), crash$model)
crash$model <- ifelse(crash$brand == 'mercedes', sub('-klasse', '', crash$model), crash$model)

crash$brand <- ifelse(crash$brand == 'land', 'land rover', crash$brand)
crash$model <- ifelse(crash$brand == 'land rover', sub('rover ', '', crash$model), crash$model)

crash$brand <- ifelse(crash$brand == 'range', 'land rover', crash$brand)
crash$model <- ifelse(crash$brand == 'land rover', sub('rover', 'range rover', crash$model), crash$model)

crash$model <- ifelse(crash$brand == 'bmw' & crash$model == 'mini', 'mini', crash$model)
crash$brand <- ifelse(crash$model == 'mini', 'mini', crash$brand)
crash$model <- ifelse(crash$brand == 'mini' & crash$model == 'mini', '', crash$model)

crash$model <- ifelse(crash$brand == 'bmw' & crash$model == 'mini cooper', 'cooper', crash$model)
crash$brand <- ifelse(crash$model == 'cooper', 'mini', crash$brand)

crash$brand <- as.factor(crash$brand)
crash$model <- as.factor(crash$model)
```

This data set has some information in the years-column that is meant to give extra information to the reader. This information is removed.

```r
crash$model_y <- sub('/ A5 ', '', crash$model_y)
crash$model_y <- sub('\\(Peugeot Partner ', '', crash$model_y)
crash$model_y <- sub('Kleinbus ', '', crash$model_y)
crash$model_y <- sub('ab Ende 2013 ', '', crash$model_y)
crash$model_y <- sub('bgl. Citroen Spacetourer, Toyota Proace ', '', crash$model_y)
crash$model_y <- sub('bgl. Opel Vivaro; ', '', crash$model_y)
crash$model_y <- sub('bgl. Opel Vivaro, Nissan Primastar; ', '', crash$model_y)
crash$model_y <- sub('/ XLV ', '', crash$model_y)
crash$model_y <- sub('\\(Citroen C1,Peugeot 108 ', '', crash$model_y)
crash$model_y <- sub('Doppelkabine ', '', crash$model_y)
```

All dates for the model years are now cleaned. However, they are still in two different formats; "YYYY - YYYY" and "from YYYY" (presumably all the way until today). To make this a little easier to read for R, all "from year"-values are converted to a "YYYY - YYYY"-format. The end-year is set to be 2017 for all entries.

```r
crash$model_y <- ifelse(crash$model_y == 'ab 2017', '2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2016', '2016 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2015', '2015 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2014', '2014 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2013', '2013 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2012', '2012 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2011', '2011 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2010', '2010 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2009', '2009 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2008', '2008 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2007', '2007 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2006', '2006 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2005', '2005 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2004', '2004 - 2017', crash$model_y)
crash$model_y <- ifelse(crash$model_y == 'ab 2003', '2003 - 2017', crash$model_y)
```

Now all dates are in a YYYY - YYYY format. From there it's easy to divide the years into two separate columns.

```r
crash$model_y_start <- as.integer(substr(crash$model_y, 1, 4))
crash$model_y_end <- as.integer(substr(crash$model_y, 8, 11))
crash$model_y <- NULL
```

Cars that are too old to be matched with the reliability data set are removed.

```r
crash <- crash[model_y_end >= 2002]
```

Finally, nationality markers are created for this data set as well.

```r
crash$nationality <- ifelse(crash$brand == 'audi'    | crash$brand == 'bmw'     | crash$brand == 'mercedes' |
                            crash$brand == 'opel'    | crash$brand == 'porsche' | crash$brand == 'volkswagen',
                            'german', 'others')

crash$nationality <- ifelse(crash$brand == 'citroen' | crash$brand == 'peugeot' | crash$brand == 'renault',
                            'french', crash$nationality)

crash$nationality <- ifelse(crash$brand == 'honda'   | crash$brand == 'mazda'   | crash$brand == 'mitsubishi' |
                            crash$brand == 'nissan'  | crash$brand == 'subaru'  | crash$brand == 'suzuki' |
                            crash$brand == 'toyota',
                            'japanese', crash$nationality)
```

This concludes the wrangling part of the project.

# Exploratory data analysis

In this part I will show some of the analyses that were done to the data sets after cleaning and wrangling.

## Auto data set

Let's start with looking at the auto data set first, and view the summary:

```r
summary(auto)


     name               price              vehicleType    yearOfRegistration      gearbox          powerPS         model             kilometer
 Length:158995      Min.   :  101   convertible  :11379   Min.   :2002       automatic: 42794   Min.   : 26.0   Length:158995      Min.   :  5000
 Class :character   1st Qu.: 3500   coupe        : 7549   1st Qu.:2004       manual   :116201   1st Qu.: 97.0   Class :character   1st Qu.: 80000
 Mode  :character   Median : 6550   hatchback    :33363   Median :2007                          Median :131.0   Mode  :character   Median :150000
                    Mean   : 8871   mini-van     :18549   Mean   :2007                          Mean   :138.2                      Mean   :115551
                    3rd Qu.:11900   sedan        :42814   3rd Qu.:2010                          3rd Qu.:170.0                      3rd Qu.:150000
                    Max.   :49999   station wagon:35791   Max.   :2015                          Max.   :900.0                      Max.   :150000
                                    suv          : 9550
                   fuelType        brand           notRepairedDamage
 compressed natural gas:  391   Length:158995      no :148193
 diesel                :72054   Class :character   yes: 10802
 electric              :   25   Mode  :character
 hybrid                :  186
 liquefied petroleum   : 1983
 petrol                :84356

```

The price seems to be centered around the lower prices, with the 3rd quartile being listed at about 12 000 EUR. Let's take look at how the prices are distributed.

```r
ggplot(auto, aes(price)) +
  geom_histogram(fill = "blue")
```

![Histogram of price in the auto data set](https://user-images.githubusercontent.com/26480394/27425383-86c3290a-5738-11e7-8144-9044db387693.png)

The plot confirms that the majority of cars are on the lower end of the scale, and that there are gradually fewer and fewer cars with higher prices. Let's now look at the mileage of the cars.

```r
ggplot(auto, aes(kilometer)) +
  geom_histogram(fill = "blue")
```

![Histogram of kilometers in the auto data set](https://user-images.githubusercontent.com/26480394/27425427-b068a24e-5738-11e7-8be6-3e994debf02b.png)

It seems like the kilometer variable is in bins, which means kilometers is rounded to some numbers. There is also an incredibly high number of cars that have registered with 150 000 kilometers compared to the other bins. This is likely because the data is truncated, that is, values over 150 000 km is combined with those in the 150 000 km group.

To be absolutely certain there is binning in the kilometer data, let's find the unique kilometer values.

```r
table(auto$kilometer)

5000  10000  20000  30000  40000  50000  60000  70000  80000  90000 100000 125000 150000
1591   1433   4273   4930   5524   6398   7185   7698   8368   9078  10580  24020 101943
```

This confirms that the kilometer variable is saved in bins. I suspect that the high number of 150 000 km cars reflect that these are older cars. This makes sense since the car prices that are usually below 10 000 EUR. Let's confirm this now.

```r
driven_150000km  <- auto$yearOfRegistration[auto$kilometer == 150000]
driven_auto <- subset(auto, auto$kilometer == 150000)

ggplot(driven_auto, aes(driven_150000km)) +
  geom_histogram(fill = "blue", bins = 10)
```

![Histogram of registration year for cars that have driven 150 000 km](https://user-images.githubusercontent.com/26480394/27426021-90512b82-573a-11e7-87f7-c0ef19689f23.png)

We can see that among the cars that have registered 150 000 km, most of them are older, with fewer and fewer cars that have driven that far for each year we get closer to 2017.

Let's now look at the different car types - how are they different?

```r
ggplot(auto, aes(price, yearOfRegistration, col = vehicleType)) +
  geom_point(alpha = 0.01) +
  geom_smooth(se = TRUE) +
  facet_wrap(~vehicleType)
```

![plot of car types according to price and registration year](https://user-images.githubusercontent.com/26480394/27181190-c0cb6b92-51d6-11e7-9a6f-fec9bb35ccb3.png)

We can see that there are some interesting patterns here. First off, the semi-transparent dots allow us to see the spread in prices, and where there are a lot of similarly-priced cars. We can see that hatchbacks typically cost less, and that the spread is quite small. The sedan-class has a wider spread than the other car types, which is hardly surprising, as there's a wider range of sedans.

One particular issue we can see from this plot is the "upward-tail" at the lower price ranges that are particularly noticeable for hatchbacks and SUVs. This seems to come from a high number of cars for lease, and they are typically newer.

A final point I found interesting is that there seems to be a sharper downward shift in interest for convertibles after the financial crisis. If one compares it to hatchbacks, you can se how the curve has a much sharper bend around 2010.


## Reliability data set

Let's now move on to the reliability data set. We'll start off with a summary:

```r
summary(rel)

brand          model        car_prod_y    report_year     fault_rate       mileage      
volkswagen: 616   3      :  94   Min.   :2002   Min.   :2013   Min.   : 2.10   Min.   : 22500  
ford      : 403   5      :  87   1st Qu.:2007   1st Qu.:2014   1st Qu.: 9.50   1st Qu.: 55000  
mercedes  : 386   911    :  50   Median :2009   Median :2015   Median :16.00   Median : 79500  
renault   : 343   a      :  50   Mean   :2009   Mean   :2015   Mean   :17.51   Mean   : 82846  
opel      : 336   a3     :  50   3rd Qu.:2011   3rd Qu.:2016   3rd Qu.:24.45   3rd Qu.:106500  
citroen   : 335   a4     :  50   Max.   :2015   Max.   :2017   Max.   :45.10   Max.   :197500  
(Other)   :3502   (Other):5540                                                                 
car_age         nationality  
Min.   : 2.000   french  : 861  
1st Qu.: 4.000   german  :1998  
Median : 6.000   japanese:1178  
Mean   : 6.035   others  :1884  
3rd Qu.: 8.000                  
Max.   :11.000    
```

We see that the report years are between 2013 and 2017, with car production years between 2002 and 2015. Each report has 2 to 11 years old cars, so these numbers match up. The fault rate is between 2.1% and a crazy 45.1%.

Let's start by looking at how many observations we have for each car brand:

```r
table(rel$brand)

alfa romeo       audi        bmw  chevrolet   chrysler    citroen      dacia   daihatsu
        80        291        298        113         15        335         68         15
      fiat       ford      honda    hyundai        kia      mazda   mercedes       mini
       197        403        191        210        190        251        386         59
mitsubishi     nissan       opel    peugeot    porsche    renault       seat      skoda
        84        146        336        183         71        343        165        178
     smart     subaru     suzuki     toyota volkswagen      volvo
        50         32        152        322        616        141
```

How does the fault rate vary across the different brands?

```r
ggplot(rel, aes(car_age, fault_rate, col = brand), legend = FALSE) +
  geom_point(alpha = 0.2) +
  geom_smooth(se = TRUE) +
  facet_wrap(~brand) +
  theme(legend.position = 'none')
```

![plot car age fault rate brand](https://user-images.githubusercontent.com/26480394/27182099-7a30c05c-51da-11e7-907f-b9738b668b81.png)

When we compare the different brand's fault rates across years, we see that there are some big differences. Look at how Mini and Porsche differs. Porsche has lower than half of the expected fault rate when the cars are 11 years old.

Let's break it down by mileage as well.

```r
ggplot(rel, aes(mileage/1000, fault_rate, col = brand), legend = FALSE) +
  geom_point(alpha = 0.2) +
  geom_smooth(se = TRUE) +
  facet_wrap(~brand) +
  theme(legend.position = 'none')
```

![plot mileage fault rate brand](https://user-images.githubusercontent.com/26480394/27182033-2c15e4e2-51da-11e7-956a-65a6a9d4f18f.png)

Now we see part of the reason why Porsche does so well - it appears that they aren't driven as far as a lot of  the other brands. Porsches is a luxury brand after all, it makes sense that they are driven less. But Mini still compares poorly against the Japanese brands, for instance. Could it be that the nationality of the cars are an important factor?

```r
ggplot(rel, aes(car_age, fault_rate, col = nationality), legend = FALSE) +
  geom_point(alpha = 0.2) +
  geom_smooth(se = TRUE) +
  facet_wrap(~nationality) +
  theme(legend.position = 'none')

ggplot(rel, aes(car_age, fault_rate, col = nationality), legend = FALSE) +
  geom_smooth(method = 'loess', se = FALSE)
```

![plot car age fault rate nationality1](https://user-images.githubusercontent.com/26480394/27382370-f402f50c-5686-11e7-9847-f3cc1eb45fe7.png)

![plot car age fault rate nationality2](https://user-images.githubusercontent.com/26480394/27382415-2ef14330-5687-11e7-98ee-500b053f43a5.png)

When we focus on age, it seems the Japanese and German cars have a lower fault rate, but after about five years, the Japanese cars have even lower fault rates than the German cars. French cars have a reputation for not being the most reliable - it seems this data supports that notion.

```r
ggplot(rel, aes(mileage/1000, fault_rate, col = nationality), legend = FALSE) +
  geom_point(alpha = 0.2) +
  geom_smooth(se = TRUE) +
  facet_wrap(~nationality) +
  theme(legend.position = 'none')

ggplot(rel, aes(mileage/1000, fault_rate, col = nationality), legend = FALSE) +
  geom_smooth(se = FALSE)
```

![plot mileage fault rate nationality1](https://user-images.githubusercontent.com/26480394/27382489-8c0390dc-5687-11e7-801d-62df44ee4a6b.png)

![plot mileage fault rate nationality2](https://user-images.githubusercontent.com/26480394/27382451-6c43f8a4-5687-11e7-9b2a-b01c9d7b2634.png)

When we focus on mileage, the German cars have the lowest fault rates for the most part. We can also see here that many of the German cars are driven further than the Japanese cars. This might explain in part why the German fault rates are higher when just age is accounted for.

How does fault rate correlate with mileage and car age?

```r
cor(rel[,5:7], method='pearson')

           fault_rate   mileage   car_age
fault_rate  1.0000000 0.7402866 0.8379794
mileage     0.7402866 1.0000000 0.8023974
car_age     0.8379794 0.8023974 1.0000000
```

This looks like expected, as fault rate is positively correlated with both mileage and fault rate. We will look further into this under the machine learning part.


## Crash rating data set

Let's start off with a summary of the data set.

```r
summary(crash)

        brand         model         stars       model_y_start   model_y_end     nationality      y_mean
 renault   : 33   3      :  6   Min.   :1.000   Min.   :1995   Min.   :2002   french  : 78   Min.   :1998
 volkswagen: 32   megane :  5   1st Qu.:4.000   1st Qu.:2004   1st Qu.:2010   german  :112   1st Qu.:2007
 ford      : 28   5      :  4   Median :5.000   Median :2009   Median :2017   japanese: 90   Median :2012
 citroen   : 25   c      :  4   Mean   :4.372   Mean   :2008   Mean   :2013   others  :191   Mean   :2011
 mercedes  : 24   passat :  4   3rd Qu.:5.000   3rd Qu.:2013   3rd Qu.:2017                  3rd Qu.:2015
 kia       : 23   6      :  3   Max.   :5.000   Max.   :2016   Max.   :2017                  Max.   :2016
 (Other)   :306   (Other):445
```

After the cleaning, we see that the model years overlap nicely with the other data sets.

The crash test scores show that most cars get a very good safety rating. More than half of the cars got five stars, according to the median of five stars.

Let's look closer at the crash ratings.

```r
ggplot(crash, aes(stars, y_mean, col = brand), legend = FALSE) +
  geom_point(alpha = 0.5) +
  facet_wrap(~brand) +
  theme(legend.position = 'none')
```

![plot stars y_mean brand](https://user-images.githubusercontent.com/26480394/27182808-d1dd870c-51dc-11e7-9dfb-a3dd3a31c7ed.png)

Here is the stars-rating plotted against the middle of the production run of each car, shown by brand. Most seem to have a pattern that goes up to the right in the graphs, which indicates that as newer cars come to market, they also achieve a better crash test score. We can also see that those few brands that have only five star ratings, tend to only have recent car models. I think it's also interesting to see that a brand reputed for being safe, like Volvo, seems to do no better than for instance Subaru, which at least to me doesn't have a reputation for producing safe cars.

# Machine learning
## Regressions

After looking at the data in the previous section, it would be interesting to find some rule-of-thumb numbers from all this data, so that we have an idea of how the different variables effect the fault rate. These are not meant to be perfectly accurate, nor representative of the whole population of cars. They are merely a way of answering a question we haven't been able to answer so far - how is the fault rate dependent on both the mileage and the age of the car? We've seen how it is changing according to age and mileage separately, but that is of course not the whole picture. All cars age, and almost all cars are also driven some distance each year. If we assume that only these two variables can explain the fault rate, how much do each of them change the fault rate?

```r
reg_fault_rate <- lm(fault_rate ~ mileage + car_age, data = rel)
summary(reg_fault_rate)

Call:
lm(formula = fault_rate ~ mileage + car_age, data = rel)

Residuals:
     Min       1Q   Median       3Q      Max
-18.2588  -3.2994  -0.4254   2.9425  20.2590

Coefficients:
              Estimate Std. Error t value Pr(>|t|)    
(Intercept) -8.935e-01  1.746e-01  -5.119 3.17e-07 ***
mileage      5.311e-05  3.239e-06  16.398  < 2e-16 ***
car_age      2.321e+00  3.939e-02  58.926  < 2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 5.062 on 5918 degrees of freedom
Multiple R-squared:  0.7152,	Adjusted R-squared:  0.7151
F-statistic:  7429 on 2 and 5918 DF,  p-value: < 2.2e-16
```

Both independent variables get significant p-values, and the R-squared value is quite high, meaning the model looks like a good fit. According to this regression, the fault rate can be described in the following way:

```
Fault rate = -0.8935 + (0.00005311 * km) + (2.231 * car age)
```

If a car is five years old, and it has driven 70 000 km, we would expect the fault rate to be..

```
-0.8935 + (0.00005311 * 70000) + (2.231 * 5) =
-0.8935 + 3.7177 + 11.605 =
14.4292
```

..about 14%.

The rule-of-thumb we can read from this regression is that when both mileage and age is accounted for, the fault rate is expected to go up by just over 5% for every 100 000 km the car drives, and almost 12% every five years (and remember to subtract one percentage point if you want to calculate a fault rate for a specific combination of mileage and age).

What happens if we try to include nationality as dummy variables? I'll focus on French, German and Japanses cars.

```r
reg_fault_rate_nat <- lm(fault_rate ~ mileage + car_age + fre + ger + jap, data = rel)
summary(reg_fault_rate_nat)

Call:
lm(formula = fault_rate ~ mileage + car_age + fre + ger + jap,
    data = rel)

Residuals:
     Min       1Q   Median       3Q      Max
-16.1877  -2.9888  -0.2161   2.6953  19.3319

Coefficients:
              Estimate Std. Error t value Pr(>|t|)    
(Intercept)  9.410e-01  1.721e-01   5.466 4.79e-08 ***
mileage      5.816e-05  2.927e-06  19.868  < 2e-16 ***
car_age      2.281e+00  3.522e-02  64.778  < 2e-16 ***
fre          1.683e+00  1.830e-01   9.195  < 2e-16 ***
ger         -3.907e+00  1.439e-01 -27.145  < 2e-16 ***
jap         -4.719e+00  1.661e-01 -28.414  < 2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 4.438 on 5915 degrees of freedom
Multiple R-squared:  0.7811,	Adjusted R-squared:  0.781
F-statistic:  4222 on 5 and 5915 DF,  p-value: < 2.2e-16
```

We see that not only are the three dummy variables all significant, they also show the same pattern that we observed in the exploratory analysis. The French cars in general have a higher fault rate, while both German and Japanese cars have a lower fault rate. The Japanese cars have the lowest fault rate according to this regression. The adjusted R-squared increased, which means this regression model explains more of the variation in the data set than the first regression.

What about price? Can we create some estimation on how the different variables in our auto data set effects price? First the car age is created as a variable in the auto data set, then a linear regression is run with price as the dependent variable, and car age, gearbox, PS and mileage as the independent variables.

```r
auto$car_age <- 2017 - auto$yearOfRegistration

reg_price <- lm(price ~ car_age + gearbox + powerPS + kilometer, data = auto)
summary(reg_price)

Call:
lm(formula = price ~ car_age + gearbox + powerPS + kilometer,
    data = auto)

Residuals:
   Min     1Q Median     3Q    Max
-52632  -2234   -225   1812  36207

Coefficients:
                Estimate Std. Error t value Pr(>|t|)    
(Intercept)    1.478e+04  5.007e+01  295.28   <2e-16 ***
car_age       -1.009e+03  3.815e+00 -264.34   <2e-16 ***
gearboxmanual -1.306e+03  2.581e+01  -50.62   <2e-16 ***
powerPS        6.375e+01  1.863e-01  342.16   <2e-16 ***
kilometer     -3.267e-02  3.076e-04 -106.20   <2e-16 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 4006 on 158990 degrees of freedom
Multiple R-squared:  0.7178,	Adjusted R-squared:  0.7178
F-statistic: 1.011e+05 on 4 and 158990 DF,  p-value: < 2.2e-16
```

Again, the p-values are sufficiently low to assume that we did not get these results by pure chance, and the R-squared value is high.

If we wrote the equation in full, it would look like this:

```
Price = 14780 - (1009 * car age) - (1306 * manual transmission) + (63.75 * PS) - (0.03267 * km)
```

Here manual transmission means 1 and automatic transmission means 0.

If a car is five years old, has driven 70 000 km, has automatic transmission and 125 PS, what do we expect the price to be?

```
14780 - (1009 * 5) - (1306 * 0) + (63.75 * 125) - (0.03267 * 70 000) =
14780 - 5045 - 0 + 7968,75 - 2286.9 =
15 416.86
```

According to this regression, we would expect the price to be about 15 400 EUR.

Again we can create some rule-of-thumb numbers when thinking about the prices of the cars in this data set. For instance, each year a car get older, you'll lose about 1000 EUR in selling price. Every 10 000 km the car drives is expected to devalue the car by about 330 EUR. These numbers must be used with caution, or else you may end up thinking that a brand new car, with automatic transmission and 0 PS is worth 14 780 EUR - which doesn't make any sense.

What happens if we also include nationality of the cars as a variable?

```r

reg_price_nat <- lm(price ~ car_age + gearbox + powerPS + kilometer + fre + ger + jap, data = auto)
summary(reg_price_nat)

Call:
lm(formula = price ~ car_age + gearbox + powerPS + kilometer +
    fre + ger + jap, data = auto)

Residuals:
   Min     1Q Median     3Q    Max
-48928  -2143   -244   1725  36067

Coefficients:
                Estimate Std. Error  t value Pr(>|t|)    
(Intercept)    1.428e+04  5.013e+01  284.834  < 2e-16 ***
car_age       -9.922e+02  3.731e+00 -265.907  < 2e-16 ***
gearboxmanual -1.129e+03  2.528e+01  -44.658  < 2e-16 ***
powerPS        5.944e+01  1.881e-01  316.007  < 2e-16 ***
kilometer     -3.539e-02  3.017e-04 -117.306  < 2e-16 ***
fre           -2.772e+02  3.825e+01   -7.247 4.28e-13 ***
ger            1.887e+03  2.488e+01   75.846  < 2e-16 ***
jap            1.387e+02  4.317e+01    3.213  0.00131 **
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Residual standard error: 3908 on 158987 degrees of freedom
Multiple R-squared:  0.7315,	Adjusted R-squared:  0.7315
F-statistic: 6.189e+04 on 7 and 158987 DF,  p-value: < 2.2e-16
```

Yet again all the variables are significant, although the dummy variable for Japanese cars is less significant than the rest. The coefficient is also smaller, which indicates that Japanese cars don't have a high premium in Germany. We can also see that there is a rather large difference between the estimated price of a French and a German car (around 2000 EUR), even if all the other variables are identical. This is a large difference, but I guess it goes to show that Germans prefer German cars, and are willing to pay more for them.

We can see that the adjusted R-squared barely increased. This tells us that while these variables do explain a little more than before, the added explanatory power was small.

I think it's worth pointing out that these regressions only explain what we allow them to explain. If we ask the model to explain fault rate by mileage and age, it won't be able to explain the fault rate in any other way, even if there was a great variable that was excluded from the regression. For instance, we don't know if the cars have unrepaired damage or upgraded leather seats or what the service history is like. These are just estimations based on a large set of aggregated data.

# Combining the three data sets

First we join the auto and crash data sets. This is done with a left_join, which will make any matches on the auto data set with the potential many matches on the crash data set a separate line. This in effect doubles the data set. A very common example is where there are several crash tests for different production years of the same car. Therefore, a line is added so that only the matches where the registration year of the car is within the production run of that car is kept.

```r
auto <- left_join(auto, crash)
setDT(auto)
auto <- auto[model_y_start < yearOfRegistration & yearOfRegistration <= model_y_end]
```

Then the reliability data set is joined as well. To cut out the duplicate lines, car production year and registration year has to be the same. In addition, I've set the report year to 2017 to avoid duplicates from previous reports. This is a cheeky assumption, because it cuts out some useful information where the newest report is from 2016 and older. I'll explain more about this in the "discussion" portion further down.

```r
auto <- left_join(auto, rel)
setDT(auto)
auto <- auto[car_prod_y == yearOfRegistration & report_year == 2017]
```

At the end, we do a little bit of cleaning up, and rename the average mileage column to avoid confusion with the kilometer column.

```r
auto$brand       <- as.factor(auto$brand)
auto$model       <- as.factor(auto$model)
auto$nationality <- as.factor(auto$nationality)

auto$car_age     <- NULL
auto$report_year <- NULL
auto$car_prod_y  <- NULL
auto$nationality <- NULL
auto$model_y_end <- NULL

colnames(auto)[16] <- "avg_mileage"
```

A final variable is created from the difference between mileages in the ads and what was the average reported mileages for the cars were. This could help us see if a car has driven much more or much less than the average car of that production year, brand and model.

```r
auto$actual_mean_km <- auto$kilometer - auto$avg_mileage
```

# Creating the search function

At last, it's time for the search function. The function is created with default arguments so that if no arguments are specified, all cars are returned. If only a few arguments are specified, the rest of the arguments will include all cars (just like a regular filter search). The arguments can be in any order.

```r
search <- function(lower_price = 0, upper_price = 50000, vehicle_type = NULL,
                   brand_name = NULL, model_name = NULL, lower_power = 0,
                   upper_power = 900, lower_km = 0, upper_km = 150000,
                   lower_safety = 1, upper_safety = 5, upper_reliability = 50,
                   damaged = NULL){
  if (is.null(vehicle_type)) {
    vehicle_type <- levels(auto$vehicleType)}
  if (is.null(brand_name)) {
    brand_name <- levels(auto$brand)}
  if (is.null(model_name)) {
    model_name <- levels(auto$model)}
  if (is.null(damaged)) {
    damaged <- levels(auto$notRepairedDamage)}
  subset_data =  auto[price               >= lower_price       &
                      price               <= upper_price       &
                      vehicleType       %in% vehicle_type      &
                      brand             %in% brand_name        &
                      model             %in% model_name        &
                      notRepairedDamage %in% damaged           &
                      powerPS             >= lower_power       &
                      powerPS             <= upper_power       &
                      kilometer           >= lower_km          &
                      kilometer           <= upper_km          &
                      stars               >= lower_safety      &
                      stars               <= upper_safety      &
                      fault_rate          <= upper_reliability ]
  return(subset_data)}
```

Here's a search example:
* Only cars with a 5-star crash safety rating.
* Fault rate below 10%.
* Price below 10 000 EUR.
* Price above 500 EUR (to avoid cars for lease).
* A station wagon.
* A car with no unrepaired damage.
* In addition, I've sorted the cars so that those that have been driven less than the average is at the top.

```r
search_results <- search(lower_safety = 5, upper_reliability = 10, upper_price = 10000,
                         lower_price = 500, vehicle_type = 'station wagon', damaged = 'no')
search_results <- search_results[order(actual_mean_km)]
```

```
|name                                                           | price|vehicleType   | yearOfRegistration|gearbox   | powerPS|model    | kilometer|fuelType            |brand      |notRepairedDamage | stars| model_y_start| y_mean| fault_rate| avg_mileage| actual_mean_km|
|:--------------------------------------------------------------|-----:|:-------------|------------------:|:---------|-------:|:--------|---------:|:-------------------|:----------|:-----------------|-----:|-------------:|------:|----------:|-----------:|--------------:|
|BMW_520d_Touring_Aut._Navi_Prof.__Panorama__Head_Up            |  8000|station wagon |               2012|automatic |     184|5        |     50000|diesel              |bmw        |no                |     5|          2010| 2013.5|       9.20|       91500|         -41500|
|Skoda_Roomster_Active_Plus_1.2                                 |  8500|station wagon |               2013|manual    |      70|roomster |     30000|petrol              |skoda      |no                |     5|          2006| 2011.5|       9.20|       65000|         -35000|
|Opel_Astra_1.6_Sports_Tourer_150_Jahre_Opel                    |  9690|station wagon |               2012|manual    |     116|astra    |     60000|petrol              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          -3000|
|Renault_Clio_Grandtour_1.2_16V_75_Dynamique                    |  9790|station wagon |               2014|manual    |      73|clio     |     30000|petrol              |renault    |no                |     5|          2012| 2014.5|       8.05|       32500|          -2500|
|Renault_Clio_Grandtour_1.2_16V_75_Dynamique                    |  9790|station wagon |               2014|manual    |      73|clio     |     30000|petrol              |renault    |no                |     5|          2012| 2014.5|       8.05|       32500|          -2500|
|Renault_Clio_Grandtour_1.2_16V_75_Dynamique_TOP!               |  9790|station wagon |               2014|manual    |      73|clio     |     30000|petrol              |renault    |no                |     5|          2012| 2014.5|       8.05|       32500|          -2500|
|Seat_Ibiza_ST_1.4_16V_Reference_Salsa                          |  8790|station wagon |               2014|manual    |      86|ibiza    |     40000|petrol              |seat       |no                |     5|          2008| 2012.5|       6.60|       39000|           1000|
|Seat_Ibiza_ST_1.2_12V_Reference_4you                           |  9000|station wagon |               2014|manual    |      75|ibiza    |     40000|petrol              |seat       |no                |     5|          2008| 2012.5|       6.60|       39000|           1000|
|Opel_Astra_J_Sports_Tourer_1.7_CDTI_EcoFlex                    |  9200|station wagon |               2012|manual    |     136|astra    |     80000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          17000|
|Opel_Astra_1.3_CDTI_DPF_ecoFLEX_Sports_TourerStar...           |  8300|station wagon |               2012|manual    |      95|astra    |     90000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          27000|
|Renault_Clio_Grandtour_Energy_dCi_90_Start_&_Stop_Dy...        |  9900|station wagon |               2014|manual    |      90|clio     |     60000|diesel              |renault    |no                |     5|          2012| 2014.5|       8.05|       32500|          27500|
|Mercedes_Benz_Citan_aus_1._Hand__Scheckheft_bei_Mercedes_Benz  |  8990|station wagon |               2013|manual    |      90|c        |    125000|diesel              |mercedes   |no                |     5|          2007| 2012.0|       7.20|       82000|          43000|
|Mercedes_Benz_Citan_aus_1._Hand__Scheckheft_bei_Mercedes_Benz  |  8990|station wagon |               2013|manual    |      90|c        |    125000|diesel              |mercedes   |no                |     5|          2011| 2014.0|       7.20|       82000|          43000|
|Audi_A6_Avant_3.0_TDI_DPF_XENON_KAMERAInzahlungnahme           |  7999|station wagon |               2012|manual    |     204|a6       |    150000|diesel              |audi       |no                |     5|          2011| 2014.0|       7.00|      100500|          49500|
|Audi_A4_Avant_/_Seat_Exeo_2.0_TDI_DPF                          |  9500|station wagon |               2012|manual    |     120|a4       |    150000|diesel              |audi       |no                |     5|          2007| 2012.0|       7.40|       93500|          56500|
|BMW_530d_Touring_Aut.                                          |  9500|station wagon |               2012|automatic |     258|5        |    150000|diesel              |bmw        |no                |     5|          2010| 2013.5|       9.20|       91500|          58500|
|Opel_Astra_1.7_CDTI_DPF_Sports_Tourer                          |  9250|station wagon |               2012|manual    |     110|astra    |    125000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          62000|
|Opel_Astra_1.7_CDTI_DPF_Sports_Tourer_150_Jahre_Opel           |  8500|station wagon |               2012|manual    |     110|astra    |    125000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          62000|
|Audi_A6_Avant_2.0_TDI_DPF_multitronic_NAVI_FV23                |  8050|station wagon |               2013|automatic |     177|a6       |    150000|diesel              |audi       |no                |     5|          2011| 2014.0|       4.20|       88000|          62000|
|Opel_Astra_LPG_Turbo_Sports_Tourer_150_Jahre_Edition           |  9990|station wagon |               2012|manual    |     140|astra    |    125000|liquefied petroleum |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          62000|
|Opel_Astra_LPG_Turbo_Sports_Tourer_150_Jahre_Edition           |  9990|station wagon |               2012|manual    |     140|astra    |    125000|liquefied petroleum |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          62000|
|Opel_Astra_2.0_CDTI_DPF_Sports_Tourer_Innovation               |  5300|station wagon |               2012|manual    |     121|astra    |    125000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          62000|
|BMW_316d_DPF_Touring                                           |  9800|station wagon |               2012|manual    |     116|3        |    150000|diesel              |bmw        |no                |     5|          2005| 2008.5|       9.65|       86000|          64000|
|Volkswagen_3BG                                                 |  2500|station wagon |               2014|manual    |     101|passat   |    150000|diesel              |volkswagen |no                |     5|          2010| 2013.5|       7.90|       84000|          66000|
|FORD_GRAND_C_MAX_TITANIUM_2.0TDCI_AHK_TOP                      |  9800|station wagon |               2012|automatic |     140|c-max    |    150000|diesel              |ford       |no                |     5|          2010| 2013.5|       8.75|       67500|          82500|
|Opel_Astra_Sports_Tourer_1_7_D_mit_Navigation                  |  7800|station wagon |               2012|manual    |     110|astra    |    150000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          87000|
|Opel_Astra_1.7_CDTI_DPF_Sports_Tourer_Navi_Leder_Alu           |  6800|station wagon |               2012|manual    |     110|astra    |    150000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          87000|
|Opel_Astra_1.7_CDTI_DPF_Sports_Tourer_"1._HAND"                |  6800|station wagon |               2012|manual    |     131|astra    |    150000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          87000|
|Verkaufe__Opel_Astra_Diesel_Sportstouer___Top_Zustand          |  9900|station wagon |               2012|manual    |     165|astra    |    150000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          87000|
|Opel_Astra_1.7_CDTI_DPF_Sports_Tourer_Edition_gute_Ausstattung |  7250|station wagon |               2012|manual    |     125|astra    |    150000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          87000|
|Opel_Astra_1.7_CDTI_DPF_Sports_Tourer_Design_Edition           |  8499|station wagon |               2012|manual    |     110|astra    |    150000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          87000|
|Opel_Astra_1.7_CDTI_DPF_Sports_Tourer                          |  9650|station wagon |               2012|manual    |     110|astra    |    150000|diesel              |opel       |no                |     5|          2009| 2012.0|       8.90|       63000|          87000|
```

This gives us much more information than if we had just searched for station wagons with no unrepaired damages between 500 EUR and 10 000 EUR. We can see that there are no large differences in fault rates, but there are rather large differences between the average mileage and the mileage from the ads. The car at the top of the search results seem like a very interesting car, as it has driven only about half as much as the average car of that brand, model and production year.

Towards the bottom of the list we see all the cars that have 150 000 km listed as mileage. The sheer number of these, and the fact that the mileage variable is truncated, makes me think the highest mileage is much higher than 150 000 km.

# Discussion

At the end of this project, I'd like to point out some areas for further exploration.

There are several places where I've made choices that cuts down on the amount of data, particularly in the auto data set. As I started with more than 371 000 ads, this was not a problem, but it would be interesting to see how many could be used in the final search. A lot of the car models could with meticulous work be matched in the three data sets. However, this would require a lot of time. There are so many different model names, some similar and some not. As my time on this project was limited, I have not spent a lot of time matching model names, but this could be done in a future project. Another area that could increase the data available for the final search is a refinement that finds the latest available report year, even if that is from an older report.

There are also a lot of possibilities in expanding on the regressions. One might wonder if it could be possible to say something about the expected changes in price in the future for a car you are looking to buy today, for instance. Let's say you want to buy a family car today, but you only need it for four years. Wouldn't it be interesting to know what you could be expected to get for it after those four years? Such a calculator could then even advice you whether or not you should lease a car instead of buying one.

If you were looking at a an older car (12 years old +), the newest reliability report doesn't have data for that car. But what if one could look into predictions of what the current fault rate is, even though this is not available. What if the last fault rate of a car you were interested in was recorded in 2010, and you wanted some estimation on what the fault rate might be today? That could be interesting to look into.

In the car data, I have primarily looked at car brands and models. But there are sometimes a plethora of different cars within the same model. Perhaps one could use the headlines from the ads to mine out some more information about the cars, which could then be used to classify them at a more granular level. This could potentially save some of the data that was shaved off when the lowest common denominator for the three data sets were determined.
