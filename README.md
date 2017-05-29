# Capstone project milestone report

## Introduction

When I last bought a used car, there were many choices that had to be made - what brand, how expensive, what type of car, and so on. What was much more difficult to find however, was how safe and reliable these cars were. I managed to find some reports for reliability ratings, and crash test safety ratings, but it took a lot of time comparing and ranking the different options. All in all it was quite time consuming.

The ideal solution to this problem would be if the website where I searched for used cars had filters on reliability and crash safety. This is not available for my preferred website, nor any other that I've looked at. Therefore, I want to create this solution myself.

The primary goal will be to make both crash test ratings and reliability ratings searchable in addition to other fields, such as car brand, model, car type, price, and so on. In order to do this, three data sets need to be stitched together. The data sets contain used car listings, crash test data and reliability data. Each present its own challenge in the wrangling process, and they need to be made uniform for the data sets to compare apples to apples.

In addition, I would like to see if there are viable methods for finding cars that are close to your search, even if there are none that match your exact search.

## The data sets and wrangling
There are three sources for this project:  

1. [Ebay-Klenanzeigen](https://www.kaggle.com/orgesleka/used-cars-database) is a scraping of the German version of Ebay with about 371,000 cars for sale. This will be used as the basis for the search recommendations.

The biggest challenge with this data is the size. There are many listings, but not all are relevant. One of the challenges is filtering out the listings that have information that is not meant to be searchable.

For instance, quite a few cars have exorbitant prices, that when you read the text in the listing, are there to symbolize that the owner wants to trade the car for another car. This type of information needs to be cleaned or removed for the search parameters to be meaningful.

This data set has the most detail on the cars, down to mileage, production year, the specific model and features for each car. Not all of this information is possible to match to the more aggregated data of the other data sets, and therefore needs to be aggregated to some degree. Ideally, I still want to keep as much information as possible for all the entries, as this will return more meaningful search results.

2. [EURONCAP](https://www.adac.de/infotestrat/tests/crash-test/alletests.aspx) is a European car safety performance assessment programme that tests cars in crash tests. This link is from the German automobile club ADAC, which is the largest automobile club in Europe. They present the EURONCAP data in a more readily available format than EURONCAP themselves.

The biggest challenge here was that the data was stored in pictures. The pictures showed how many stars each car had attained. To create a viable data set from this information, I took the html-code for the website, ran it through some code to extract the table data, including the part of the picture names that indicated how many stars the picture showed (think "3_stars.img", where just the number 3 is extracted).

After that, it was just a question of dividing up the name-column into car brands and car models successfully.

3. [TÜV auto reports](http://www.anusedcar.com) is a source that not only provides data on reliability across different brands, but also different models within each brand. The report is published each year, with different age brackets for the cars.

The biggest challenge in this data set is how the data is aggregated; the ages for the cars are grouped together two and two years. So for the 2017 report, the newest cars are in the 2-3 year old bracket. The next group is 4-5 years, and so on. This may have been done to create a simpler data set (I have tried to get in touch with them), but the grouping artifact is an issue when it comes to actually evaluating the different years independently.

To fix this, I've averaged the years that fall in two reports (for instance a car from 2014 is in the "2-3 year old car"-group in both 2016 and 2017). By doing this, the adjusted 2017 report is able to give different reliability data for each year, as a proxy for the actual data that is not available.

Data wrangling-wise, the best way of aggregating this data was to create columns with dplyr that are easily able to both identify where there are enough data to create the means, and to actually create them as a separate variable. Then the variable was inserted into the "pure" reliability data as a proxy to break up the two consecutive years with identical data.

## About the data fields
The data sets all have information on car names. Only the biggest one have separate fields for car brands and models. Therefore this needs to be created for the other two data sets. There are a few naming conventions that need to be made similar, but apart from that, it's a simple enough process.

In addition, the difference in degree of aggregation means that it is vital to create a common level of detail for the data sets. For instance, what is described as a "Mercedes Benz E-Class Coupe" may merely be called "Mercedes E-Class" in another data set. The "coupe"-detail is lost when we reduce the data sets to the smallest common denominators.

The biggest data set has about 371,000 rows of used cars listings, but about 170,000 will not be used for this project, as they are either older or newer than the data from the other data sets. For instance, I don't have crash test data from 1991, therefore I've chosen not to include the cars that are that old

A lot of the data fields in the listing-data set were not useful for this project, and were rejected. These include the number of pictures for each listing (which erroneously was set to 0 for all listings in the data set) and several meta-data fields that aren't relevant for this project.

All the data sets are in German, and a few issues pertaining to translations and measurements have come up. One is the issue with PS/Pferdstärke vs. the more common BHP/break horse power. After checking with several car enthusiasts, I've concluded that although these two are not technically equal, they are often used interchangeably. I've therefore decided to do so also.

I think it's also worth noting that the crash test criteria has changed a lot over the years. Anecdotally, I would compare a five-star rating in 2002 to a three-star rating in 2017. This is due to the ever-evolving new requirements for getting a top score, which includes new technology like brake assistance or lane-change warnings. I've intentionally kept the original ratings, as I believe that is more intuitive, but it could be interesting to include devaluations of the older ratings in a future project.

## * Limitations

Having to rely on the smallest common denominator from three different data sets means there have to be some concessions. For one, the brands and models are possible to identify for the most part, but features are not. As one can imagine, the same car with different features may behave differently in both crash tests and reliability. A top-specced model typically has extra features, which may include some safety features. Therefore, the ratings should be thought of as a sort of "mean" for each of the car models within years they are tested for.

Secondly, the listing-data set is almost cut in half after cleaning, most of which could have been usable if the other data sets where more complete. The oldest reliability data is from 2002, where the crash test data is from about 2000 for most models (some have data before this, depending to the launch date of the model).

I think it's also important to recognize that this project will only help you identify which cars fall within which safety/reliability-rating, it will not help you decide on which car is right for you. Purchasing a car is, I believe, for most at least a partially emotional action. If you are like me, and crave more information pertaining to the cars listed for sale, then this project may be of help. If you simply want to find a car you will like, you may be better helped by looking elsewhere.

## Preliminary exploration and initial findings

## The way forward

In the coming weeks I aim to create unified search-parameters for the search functionality, and test it thoroughly. My biggest concern is how wide the actual search functionality will be - will it "feel" like an actual search engine, or is it too limited in scope?
