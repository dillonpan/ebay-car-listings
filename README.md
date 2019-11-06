# ebay-car-listings
Project using the Python package "pandas" and various functions to clean and analyze data in CSV format

# Project Details:
The dataset we have is of used cars from eBay Kleinanzeigen, a section of the German eBay website

The columns (in order) within the CSV and their details are as follows:
1. dateCrawled - When this ad was first crawled. All field-values are taken from this date.  
2. name - Name of the car.  
3. seller - Whether the seller is private or a dealer.  
4. offerType - The type of listing  
5. price - The price on the ad to sell the car.  
6. abtest - Whether the listing is included in an A/B test.  
7. vehicleType - The vehicle Type.  
8. yearOfRegistration - The year in which which year the car was first registered.  
9. gearbox - The transmission type.  
10. powerPS - The power of the car in PS.  
11. model - The car model name.  
12. kilometer - How many kilometers the car has driven.  
13. monthOfRegistration - The month in which which year the car was first registered.  
14. fuelType - What type of fuel the car uses.  
15. brand - The brand of the car.  
16. notRepairedDamage - If the car has a damage which is not yet repaired.  
17. dateCreated - The date on which the eBay listing was created.  
18. nrOfPictures - The number of pictures in the ad.  
19. postalCode - The postal code for the location of the vehicle.  
20. lastSeenOnline - When the crawler saw this ad last online.  

Note: If you run into an error named UnicodeDecodeError, try the next two most popular encoding types (Latin-1 and Windows-1252) in the open() function. Example: autos = pandas.read_csv('autos.csv', encoding='Latin-1')

# Opening and Exploring the Data:
```python
import pandas
import numpy

autos = pandas.read_csv('autos.csv', encoding='utf8')
```
Note: You can use 'print(head(x))' with 'x' being an integer of how many rows from the top you want to print (Ex.'print(head(3))' to print the first 3 rows). The default optional argument of head() is 5.
Unlike reading the CSV using the function open(), pandas has already seperated the header from the body.

# Clean Columns
```python
print(autos.columns)
```
Index(['dateCrawled', 'name', 'seller', 'offerType', 'price', 'abtest',
       'vehicleType', 'yearOfRegistration', 'gearbox', 'powerPS', 'model',
       'odometer', 'monthOfRegistration', 'fuelType', 'brand',
       'notRepairedDamage', 'dateCreated', 'nrOfPictures', 'postalCode',
       'lastSeen'],
      dtype='object')
      
We'll make a few changes here:

1. Change the columns from camelcase to snakecase.
2. Change a few wordings to more accurately describe the columns.

```python
autos.columns = ['date_crawled', 'name', 'seller', 'offer_type', 'price', 'ab_test',
       'vehicle_type', 'registration_year', 'gearbox', 'power_ps', 'model',
       'odometer', 'registration_month', 'fuel_type', 'brand',
       'unrepaired_damage', 'ad_created', 'num_photos', 'postal_code',
       'last_seen']
 ```
 
# Initial Data Exploration and Cleaning
We'll start by exploring the data to find obvious areas where we can clean the data.  
```python
print(autos.describe(include='all'))
```

Take a look at the printed table and you might notice the following issues:

There are a number of text columns where all (or nearly all) of the values are the same:
  1. seller
  2. offer_type
The num_photos column looks odd, we'll need to investigate this further.

```python
autos["num_photos"].value_counts()
```
0    50000  
Name: num_photos, dtype: int64

It looks like the num_photos column has 0 for every column. We'll drop this column, plus the other two we noted as mostly one value.
```python
# the "axis=1" optional argument below means that we want to drop the column.
# The deault axis argument in .drop() is axis=0, aka dropping the row
autos = autos.drop(["num_photos", "seller", "offer_type"], axis=1)
```

Next, the data in columns "price" and "auto" both are in string form. This is because they include extra characters that we want to remove. We'll remove the extra characters in the column data and change the type from string to integer.

```python
autos["price"] = (autos["price"]
                          .str.replace("$","")
                          .str.replace(",","")
                          .astype(int)
                          )
print(autos["price"].head())
```

0    5000
1    8500
2    8990
3    4350
4    1350
Name: price, dtype: int64

```python
autos["odometer"] = (autos["odometer"]
                             .str.replace("km","")
                             .str.replace(",","")
                             .astype(int)
                             )
# also doing a quick change of the header to snakecase
autos.rename({"odometer": "odometer_km"}, axis=1, inplace=True)
autos["odometer_km"].head()
```

0    150000
1    150000
2     70000
3     70000
4    150000
Name: odometer_km, dtype: int64

# Exploring Odometer and Price
```python
print(autos["odometer_km"].value_counts())
```
150000    32424
125000     5170
100000     2169
90000      1757
80000      1436
70000      1230
60000      1164
50000      1027
5000        967
40000       819
30000       789
20000       784
10000       264
Name: odometer_km, dtype: int64

There are more high mileage cars listed, which makes sense but the odometer numbers do seem odd. All seem been rounded in some way, most likely due to the design of Ebay's website. It's unlikely all the posters for the car lisings manually rounded the odometer numbers themselves.

Now let's print the value counts of the prices, limiting to the top 20 as theres many more unique values in this column
```python
print(len(autos["price"]))
print(autos["price"].value_counts().head(20))
```
50000

0       1421
500      781
1500     734
2500     643
1000     639
1200     639
600      531
800      498
3500     498
2000     460
999      434
750      433
900      420
650      419
850      410
700      395
4500     394
300      384
2200     382
950      379
Name: price, dtype: int64

Looks like the majority of sellers tend to round their prices but what sticks out is the 1421 listings where the price is put at 0. Given that it is only about 2.5% of the 50,000 listings, we might want to consider removing them from the data. Before we do that, let's take a look and see if theres a large amount of listing not exactly, but close to a price of 0. We are allowed to sort a value count by the price and not the count.

```python
autos["price"].value_counts().sort_index(ascending=True).head(20)
```
0     1421
1      156
2        3
3        1
5        2
8        1
9        1
10       7
11       2
12       3
13       2
14       1
15       2
17       3
18       1
20       4
25       5
29       1
30       7
35       1

While we're at it, we should check if theres listings with absurdly high prices listed:
```python
print(autos["price"].value_counts().sort_index(ascending=False).head(20))
```
99999999    1
27322222    1
12345678    3
11111111    2
10000000    1
3890000     1
1300000     1
1234566     1
999999      2
999990      1
350000      1
345000      1
299000      1
295000      1
265000      1
259000      1
250000      1
220000      1
198000      1
197000      1
Name: price, dtype: int64

From the looks of it, theres a few $1 or close to $1 listings and there's definately a few listings with extravgent prices listed on them. Since this is Ebay data we're working with, it's not farfetched to have bids start at $1 so we should just remove the $0 listings. In terms of the large prices, there's a massive price jump after the $350000 listing. The listings above 350,000 don't seem very realistic so we should remove them as well.

```python
autos = autos[autos["price"].between(1,351000)]
autos["price"].describe()
```
