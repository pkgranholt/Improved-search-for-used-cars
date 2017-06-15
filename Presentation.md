# Exploratory data analysis
## auto data set
Let's start with looking at the auto data set first, and view the summary:

```
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

The price seems to be centered around the lower prices, with the 3rd quartile
being listed at 11 000 EUR.

Let's plot price vs. kilometers to see if this holds true.
```
plot(auto$kilometer, auto$price)
```

The plot confirms that the majority of cars are below 10 000 EUR. We can see
that prices seem to be on a continuous scale, while kilometers is in bins. This
plot does a poor job in explaining exactly how many cars there are in each bin
for kilometers, so let's make a plot to visualize this as well.
```
ggplot(auto, aes(kilometer)) +
  geom_density(fill = "lightcyan")
```

This reveals two things - one - there is definitively binning, which means
kilometers is rounded to some numbers. Two - there is an incredibly high number
of cars that have registered with 150 000 kilometers compared to the other
bins. This is likely because the data is truncated, that is, values over
150 000 km is combined with those in the 150 000 km group.

To be absolutely certain there is binning in the kilometer data, let's find
unique kilometer values.
```
table(auto%kilometer)

5000  10000  20000  30000  40000  50000  60000  70000  80000  90000 100000 125000 150000
1591   1433   4273   4930   5524   6398   7185   7698   8368   9078  10580  24020 101943
```

Binning confirmed. I suspect that the high number of 150 000 km cars reflect
that these are older cars. This makes sense since the car prices that are
usually below 10 000 EUR. Let's confirm this now.
```
long <- auto[kilometer == 150000]
hist(long$yearOfRegistration)
```

We can see that among the cars that have registered 150 000 km, most of them
are older, with fewer and fewer cars that have driven that far for each year
we get closer to 2017.


Let's now look at the different car types - how are they different?
```
ggplot(fifty, aes(price, yearOfRegistration, col = vehicleType)) +
  geom_point(alpha = 0.01) +
  geom_smooth(se = TRUE) +
  facet_wrap(~vehicleType)
```

We can see that there are some interesting patterns here. First off, the low
alpha-value allow us to see the spread in prices, and where there are a lot
of similarly-priced cars. We can see that hatchbacks typically cost less, and
that the spread is quite small. The sedan-class has a wider spread than the
other car types, which is hardly surprising, as there's a wide range of sedans.

One particular issue we can see from this plot is the "upward-tail" at the
lower price ranges that are particularly noticeable for hatchbacks and SUVs.
This comes primarily from ads that have cars for lease.

A final point I found interesting is that there seems to be a sharper
downward shift in interest for convertibles after the financial crisis. If one
compares it to sedans, you can se how the curve has a much sharper bend around
2010.


## reliability data set
Let's now move on to the reliability data set. We'll start off with a summary:
```
summary(rel)
```

We see that the report years are between 2013 and 2017, with car production
years between 2002 and 2015. Each report has 2 to 11 years old cars, so
these numbers match up. The fault rate is between 2.1% and a crazy 45.1%.

Let's start by looking at how many observations we have for each car brand:
```
table(rel$brand)

alfa romeo       audi        bmw  chevrolet   chrysler    citroen      dacia   daihatsu       fiat       ford      honda
        81        295        304        112         15        342         69         15        200        412        194
   hyundai        kia      mazda   mercedes       mini mitsubishi     nissan       opel    peugeot    porsche    renault
       213        191        255        391         59         83        148        342        186         72        350
      seat      skoda      smart     subaru     suzuki     toyota volkswagen      volvo
       167        183         51         32        158        330        624        143
```

How does the fault rate vary across the different brands?
```
ggplot(rel, aes(car_age, fault_rate, col = nationality), legend = FALSE) +
  geom_point(alpha = 0.2) +
  geom_smooth(se = TRUE) +
  facet_wrap(~nationality) +
  theme(legend.position = 'none')
```

When we compare the different brand's fault rates across years, we see that
there are some big differences. Look at how Mini and Porsche differs. Porsche
has lower than half of the expected fault rate when the cars are 11 years old.

Let's break it down by mileage as well.
```
ggplot(rel, aes(mileage/1000, fault_rate, col = brand), legend = FALSE) +
  geom_point(alpha = 0.2) +
  geom_smooth(se = TRUE) +
  facet_wrap(~brand) +
  theme(legend.position = 'none')
```

Now we see part of the reason why Porsche does so well - it appears that they
aren't driven as far as a lot of  the other brands. Porsches are luxury cars
after all, it makes sense that they are driven less. But Mini still compares
poorly against the Japanese brands, for instance. Could it be that the
nationality of the cars are an important factor?
```
ggplot(rel, aes(car_age, fault_rate, col = nationality), legend = FALSE) +
  geom_smooth(se = FALSE)

ggplot(rel, aes(mileage/1000, fault_rate, col = nationality), legend = FALSE) +
  geom_smooth(se = FALSE)
```

When we focus on age, it seems the Japanese and German cars have a lower fault
rate, but after about five years, the Japanese cars have even lower fault rates
than the German cars. French cars have a reputation for not being the most
reliable - it seems this data supports that notion.

When we focus on mileage, the German cars have the lowest fault rates for the
most part. We can also see here that many of the German cars are driven further
than the Japanese cars. This might explain in part why the German fault rates
are higher when just age is accounted for.


## crash rating data set
Let's start off with a summary of the data set.
```
summary(crash)

brand            model         stars          model_y_start   model_y_end  
renault   : 33   3      :  6   Min.   :1.000   Min.   :1995   Min.   :2002  
volkswagen: 32   megane :  5   1st Qu.:4.000   1st Qu.:2004   1st Qu.:2010  
ford      : 28   5      :  4   Median :5.000   Median :2009   Median :2017  
citroen   : 25   c      :  4   Mean   :4.372   Mean   :2008   Mean   :2013  
mercedes  : 24   passat :  4   3rd Qu.:5.000   3rd Qu.:2013   3rd Qu.:2017  
kia       : 23   6      :  3   Max.   :5.000   Max.   :2016   Max.   :2017  
(Other)   :306   (Other):445   
```

After the cleaning, we see that the model years overlap nicely with the other
data sets.

The crash test scores show that most cars get a very good safety rating. More
than half of the cars got five stars, according to the median of five stars.

Let's look closer at the crash ratings.

```
ggplot(crash, aes(as.integer(stars), y_mean, col = brand), legend = FALSE) +
  geom_point(alpha = 0.5) +
  facet_wrap(~brand) +
  theme(legend.position = 'none')
```

Here is the stars-rating plotted against the middle of the production run of
each car, shown by brand. Most seem to have a pattern that goes up to the right
in the graphs, which indicates that as newer cars come to market, they also
achieve a better crash test score.

We can also see that those few brands that have only five star ratings, tend to
only have recent car models.
