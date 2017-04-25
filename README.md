# Capstone project proposal

## What is the problem you want to solve?
When I last bought a (used) car, there were many choices that had to be made - what brand, how expensive, what type of car, and so on. What was much more difficult to find however, was how these cars compared in safety rating and reliability rating. I managed to find some German reports for reliability, and of course EURONCAP is a great source for safety ratings, but it took a lot of comparing and ranking the different options. All in all it was quite time consuming.  

To solve this tedious problem, I'd like to make a system that evaluates what you are searching for (eg. type of car, brand, price), and based on your search criteria, is able to either recommend the "best" (safest, most reliable or both) car. The second part of the recommendation comes into play if the search is too narrow to be able to recommend a car that fits your exact criteria. If this is the case, the code will try to find a car similar to what you were searching for, but that differs in one or two aspects. For instance, if you are looking for a car that has more than 400 BHP and is below a certain price point, which might return no hits, the code could suggest cars that have fewer than 400 BHP, but that is below the price point you want.  

## Who is your client and why do they care about this problem?
The client is anyone who wants to buy a used car, and that values safety and reliability. To my knowledge, no one has a search function where these two components are part of the search engine, and likewise, I don't know of anywhere that you can get recommendations that are _close_ to you search, and that tries to find similar cars if your search is too specific.  

## What data are you going to use for this?
The data is from three sources:  

1. [Ebay-Klenanzeigen](https://www.kaggle.com/orgesleka/used-cars-database) is a scraping of the German version of Ebay with about 370,000 cars for sale. This will be used as the basis for the search recommendations.
2. [EURONCAP](https://www.adac.de/infotestrat/tests/crash-test/alletests.aspx) is a European car safety performance assessment programme that tests cars in crash tests. This link is from the German automobile club ADAC, which is the largest automobile club in Europe. They present the EURONCAP data in a more readily available format than EURONCAP themselves.
3. [TÜV auto reports](http://www.anusedcar.com) is a source that not only provides data on reliability across different brands, but also different models within each brand. This makes it one of the best available reliability ratings that I know of.  

## Approach to solving the problem
First step is to gather all the data in a suitable format. Against the advise found on Springboard, I want to use data that is not only in CSV-format, but rather in three different formats; pictures, text lists and CSV. The latter one is simple enough. For the text lists, I need to collect them and make them into a neat dataset. This could easily be done in Excel for instance. The most difficult part is to convert the pictures into a dataset. After looking briefly at the web-page, I believe it is a relatively simple process of connecting data in the source code for the web-page to data in the table on the web-page.  

Next is connecting the car data with the other datasets. This could be quite challenging, as the brands and models are described quite differently in the datasets. I'm not yet sure if the best approach is to use regular expressions or find a method that finds the closest match on its own.

Then comes the search and recommendation parts of the code, assuming there are search results for the search. After that, comes the search results if there are no search results within the parameters set in the search. I propose to use clustering to group similar cars together, and pick top performers from the nearest cluster, according to safety rating, reliability rating and the search parameters.  

## What are the deliverables?
The goal is to gather the data to neat datasets, and create the code that is able to recommend a safe and reliable car according to the search parameters. If there are no cars that matches the search parameters, the code will provide similar matches. I also aim to provide a slide deck with examples of searches and recommendations, and some of the challenges that I came across during the project.
