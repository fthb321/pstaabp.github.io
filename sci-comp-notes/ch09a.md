Week 10 Notes
=============

These are the notes for week 10 of Math 3001. [Return to all notes](../index.html)

Data Analysis
---------

The next section of the course discusses Data Analysis and trying to solve problems that involve data. 

###Basic Data Analysis

Let's look at some data that we've generated before.  Consider rolling a die 10 times

```
rolls=rand(1:6,10);
```

and then finding the counts of each roll 
```
theRange,counts=hist(rolls,0.5:6.5)
```

and then plotting the histogram using Gadfly  (you need to load the package)
```
plot(x=[1:6],y=counts/10,Geom.bar)
```

###Some basic stats

Recall that the mean of a data set is the sum divided by the size of the data set. 
```
sum(rolls)/length(rolls)
```

or using the mean command
```
mean(rolls)
```

The median (or middle value) can be found:
```
median(rolls)
```

The standard deviation of a data set is 
\\[\sigma = \sqrt{\frac{\sum (x_i-\bar{x})^{2}} {N}}\\]

where \\(\bar{x}\\) is the mean of the data, and \\(N\\) is the size (length) of the dataset. 

This can be written in Julia as follows:
```
sqrt(sum((rolls-mean(rolls)).^2)/(length(rolls)-1))
```

Alternatively, the standard deviation can be written
\\[\sigma = \sqrt{\frac{\sum x_i^{2} - n (\bar{x})^{2}}{n-1}}\\]

and this can be done in Julia via:
```
sqrt((sum(rolls.^2)-length(rolls)*mean(rolls)^2)/(length(rolls)-1))
```

but of course, there is the built-in command
```
std(rolls)
```


###Exercises

1. Create of vector of the sums of the rolls of 3 dice (you did this in homework).  Use 1,000 rolls. 
2. Create a histogram
3. Find the mean, median and standard deviation (using all three formulations) of the roll sums.  (Does your answer make sense?)








###Getting data from a file

Download the file [Gaz_ua_national.txt](Gaz_ua_national.txt) and save it somewhere that you can access it from Julia. 

###Load in the data

The data is a CSV file with tab delimeters and information about the data is [on the census website](http://www.census.gov/geo/maps-data/data/gazetteer2010.html)  To load it in type:
```
data,header=readdlm("Gaz_ua_national.txt",'\t',String,header=true)
```

Here's a few questions we may want to know:

1. What are the top 10 areas in population
1. Give a histogram plot in terms of population?  (What are good bin sizes?)
2. What is the total population of all areas? 
2. What the top 10 area in housing units?
2. What is the total number of housing units?
2. What is the average number of people per housing units for all areas? 
2. For the top 10 area in population, find the average number of people per housing unit?
2. What are the top 10 areas in land size?
3. What are the top 10 areas in water size?
4. What are the Massachusetts areas in the data?
5. What is the average population, median and standard deviation  of the areas?



###Getting the data into a reasonable form:

Since all of the data came is as a String  (note: we said this in the readdlm function), we now want to use the data and will need to convert it to ints and floats. First, we'll make a new array to store the data.  The function
```
data2=Array(Any,3592,11)
```

Creates an array of type "Any" with the dimensions the same size. Note the Any type can store different types within the same array.  Examining the data, this is what we want each column type to be:

| Columns | data type|
|---------|----------|
| Area/Zip-like Code (col 1), Area Name (col 2), Urban or Not (col 3) | String |
| Population (col 4), Housing Units (col 5), Area of Land and water (cols 6-9) | Int |
| Longitude (col 10), Latitude (col 11) | Float | 

###Filling the strings

Fill the first 3 columns with just the columns from the data array:
```
data2[:,[1,2,3]]=data[:,[1,2,3]]
```

Next, fill the next 6 columns as ints
```
data2[:,[4:9]]=int(data[:,[4:9])
```

and then if we try the same with the final two columns:
```
data2[:,[10,11]]=float(data[:,[10,11]])
```
we'll get an error:

```
ArgumentError("float64(String): invalid number format")
```

because the last column is in a strange format. To remedy this we need to strip the last column of all white space (spaces and tabs and such). 

The 10th column can be filled as expected with
```
data[:,10]=float(data[:,10])
```

but the 11th will need
```
data2[:,11]=float(map(strip,data[:,11]))
```

where the strip function (which gets rid of the white space) is applied to each element in the 11th column.  Then each is converted to a float.  Looking now at the data2 array:

```
3592x11 Array{Any,2}:
 "00037"  …   11.283  0.116  29.9676   -92.0982
 "00064"       4.369  0.008  34.1792   -82.3797
 "00091"       2.071  0.005  44.9486   -90.3159
 "00118"       2.864  0.02   33.8247   -88.5546
 "00145"      12.742  0.096  45.4632   -98.471 
 "00172"  …   15.443  0.745  46.9764  -123.796 
 "00199"     131.131  3.794  39.509    -76.3034
 "00226"       1.178  0.005  33.8305  -101.846 
 "00253"       3.387  0.001  38.9211   -97.2208
 "00280"      54.732  0.382  32.4285   -99.7472
```

shows a bit of it. 

###Sorting an array

To answer some of the other questions above, we need to sort the data.  Let's look at a simple example first.  Here's a 2D array of size 7 by 7 with random numbers in it:
```
A=rand(1:10,7,7)
```

and when I run this I get:
```
2  10  5  10  8  3  4
8   6  5   1  1  3  8
5   4  9   7  7  6  6
3   7  3   8  1  4  9
10  10  2   4  2  9  2
7   3  1   6  6  3  4
9   7  4   4  1  5  3
```


We can use the sort function to sort this.  (Look at the documentation for this).  If we do
```
sort(A,1)
```

Then we get each column sorted.   If instead, we want to sort by rows, type
```
sort(A,2)
```

and then each row is sorted.  The trouble is that for our array, we don't want to sort each column or each row, but want to sort by a column.  To do this, we use the `sortrows` method. 

For the array above, if we type
```
sortrows(A)
```

then the result is 
```
  2  10  5  10  8  3  4
  3   7  3   8  1  4  9
  5   4  9   7  7  6  6
  7   3  1   6  6  3  4
  8   6  5   1  1  3  8
  9   7  4   4  1  5  3
 10  10  2   4  2  9  2
```

where the array A is sorted by rows by the first column. To sort by the 2nd column, we do
```
sortrows(A,by=x->x[2])
```

and the result is
```
7   3  1   6  6  3  4
5   4  9   7  7  6  6
8   6  5   1  1  3  8
3   7  3   8  1  4  9
9   7  4   4  1  5  3
2  10  5  10  8  3  4
10  10  2   4  2  9  2
```


###Sorting the data2 array

If we want to find the top 10 areas by population, then we will sort by column 4 and we'll want it sorted in reverse order so we'll use the option rev=true.  Also, we'll want to store it, so we'll set it to a variable name:
```
byPop=sortrows(A,by=x->x[4],rev=true)
```

which will now have the data sorted by the 4th column (the population).   If we then just print out the top 10 rows (1st, 2nd and 4th columns):
```
dataByPop[1:10,[1,2,4]]
```

then we get:
```
10x3 Array{Any,2}:
 "63217"  "New York--Newark, NY--NJ--CT Urbanized Area"          18351295
 "51445"  "Los Angeles--Long Beach--Anaheim, CA Urbanized Area"  12150996
 "16264"  "Chicago, IL--IN Urbanized Area"                        8608208
 "56602"  "Miami, FL Urbanized Area"                              5502379
 "69076"  "Philadelphia, PA--NJ--DE--MD Urbanized Area"           5441567
 "22042"  "Dallas--Fort Worth--Arlington, TX Urbanized Area"      5121892
 "40429"  "Houston, TX Urbanized Area"                            4944332
 "92242"  "Washington, DC--VA--MD Urbanized Area"                 4586770
 "03817"  "Atlanta, GA Urbanized Area"                            4515419
 "09271"  "Boston, MA--NH--RI Urbanized Area"                     4181019
```

showing the top 10 areas by population.  



###Olympic Athletes

Download the file [OlympicAthletes_0.csv](OlympicAthletes_0.csv) and save it somewhere that you can access it from Julia.  It would be a good idea to open it in a spreadsheet (like excel).

You'll first need to load it in and store each data type in the correct format. 

Here's some questions to answer:

1. What is the total number of medals given in all Olympics in the dataset. 
2. Collectively taking each olympics, give the top 10 athletes by number of medals. 
3. Who had the most olympic gold medals in the Summer 2000 games?  How many medals?
4. Who has the most Olympic Silver medals in the data set (Collectively over multiple olympics)
5. Find some other interesting questions and answer them. 
