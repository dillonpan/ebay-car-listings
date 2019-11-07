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
Take a look at the printed table and you might notice the following issues.  
There are a number of text columns where all (or nearly all) of the values are the same:
  1. seller
  2. offer_type

count      50000  
unique         2  
top       privat  
freq       49999  
Name: seller, dtype: object  

count       50000  
unique          2  
top       Angebot  
freq        49999  
Name: offer_type, dtype: object  

count    50000.0  
mean         0.0  
std          0.0  
min          0.0  
25%          0.0  
50%          0.0  
75%          0.0  
max          0.0  
Name: num_photos, dtype: float64  


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
print(autos["odometer_km"].head())
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

Looks like the majority of sellers tend to round their prices but what sticks out is the 1421 listings where the price is put at 0. Given that it is only about 2.5% of the 50,000 listings, we might want to consider removing them from the data. Let's take a look and see if theres a large amount of listing not exactly, but close to a price of 0. We are allowed to sort a value count by the price and not the count using the "ascending" optional argument.

```python
print(autos["price"].value_counts().sort_index(ascending=True).head(20))
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

From the looks of it, theres a few $1 or close to $1 listings and there's definately a few listings with extravgent prices listed on them. Since this is Ebay data we're working with, it's not farfetched to have bids start at $1.. In terms of the large prices, there's a massive price jump after the $350000 listing. The listings above 350,000 don't seem very realistic and should be considered removable.

Now that we have better information on the dataset, we can start working on the removal of "bad" data. The process is that we filter the current database "autos" and then re-assignment the post filtered information back as "autos". Thus we "autos" overwrite itself in a way.

```python
autos = autos[autos["price"].between(1,350000)]
print(autos["price"].describe())
```
count     48565.000000  
mean       5888.935591  
std        9059.854754  
min           1.000000  
25%        1200.000000  
50%        3000.000000  
75%        7490.000000  
max      350000.000000  
Name: price, dtype: float64  

# Exploring Price by Brand
```python
# the normalize optional argument alters the count to it's respective percentage of the total count
print(autos["brand"].value_counts(normalize=True))
```
volkswagen        0.212828
opel              0.108658
bmw               0.108597
mercedes_benz     0.095789
audi              0.085823
ford              0.069639
renault           0.047874
peugeot           0.029445
fiat              0.025986
seat              0.018944
skoda             0.016061
nissan            0.015258
mazda             0.015217
smart             0.014290
citroen           0.014125
toyota            0.012581
hyundai           0.009945
sonstige_autos    0.009698
volvo             0.009039
mini              0.008607
mitsubishi        0.008216
honda             0.007989
kia               0.007104
alfa_romeo        0.006610
porsche           0.005910
suzuki            0.005889
chevrolet         0.005663
chrysler          0.003480
dacia             0.002656
daihatsu          0.002512
jeep              0.002224
subaru            0.002121
land_rover        0.002039
saab              0.001627
daewoo            0.001565
jaguar            0.001524
trabant           0.001400
rover             0.001338
lancia            0.001133
lada              0.000597
Name: brand, dtype: float64 

There are lots of brands with a low percentage of listings, so let's limit our analysis to those with more than 5% of the total count:
```python
brand_counts = autos["brand"].value_counts(normalize=True)
# The below is the process of filtering a dataset in the pandas package
# Alternative way to look at it: common_brands = brand_counts[autos["brand"].value_counts(normalize=True) > .05].index
common_brands = brand_counts[brand_counts > .05].index
print(common_brands)
```
Index(['volkswagen', 'opel', 'bmw', 'mercedes_benz', 'audi', 'ford'], dtype='object')

```python
brand_mean_prices = {}

for brand in common_brands:
    brand_only = autos[autos["brand"] == brand]
    mean_price = brand_only["price"].mean()
    brand_mean_prices[brand] = int(mean_price)

print(brand_mean_prices)
```
{'volkswagen': 5332,  
 'opel': 2944,  
 'bmw': 8261,  
 'mercedes_benz': 8536,  
 'audi': 9212,  
 'ford': 3728}  
 
From the top 5 brands, it looks like audi, bmw, and mercedes are generally more expensive while opel and ford are the more affordable options. Volkswagen seems to be somewhere in the middle between. Keep in mind this is just a general average and does not include specifics of model, year, odometer numbers, etc.

# Exploring Mileage
```python
bmp_series = pandas.Series(brand_mean_prices)
# we can turn the brand_mean_prices dictionary to a pandas dataset ,like below
print(pandas.DataFrame(bmp_series, columns=["mean_price"]))
```
             \*mean_price\*
volkswagen           5332
opel                 2944
bmw                  8261
mercedes_benz        8536
audi                 9212
ford                 3728

```python
brand_mean_mileage = {}

for brand in common_brands:
    brand_only = autos[autos["brand"] == brand]
    mean_mileage = brand_only["odometer_km"].mean()
    brand_mean_mileage[brand] = int(mean_mileage)

mean_mileage = pandas.Series(brand_mean_mileage).sort_values(ascending=False)
mean_prices = pandas.Series(brand_mean_prices).sort_values(ascending=False)

brand_info = pandas.DataFrame(mean_mileage,columns=['mean_mileage'])
print(brand_info)
```

mean_mileage  
bmw	       132572  
mercedes_benz	130788  
opel	       129310  
audi	       129157  
volkswagen	128707  
ford	       124266  

```python
brand_info["mean_price"] = mean_prices
print(brand_info)
```

mean_mileage  mean_price  
bmw	       132572 8332  
mercedes_benz	130788 8628  
opel	       129310 2975  
audi	       129157 9336  
volkswagen	128707 5402  
ford	       124266 3749  

The range of car mileages does not vary as much as the prices do by brand. However, it does seem like higher priced vehicles do seem to have a slightly higher mileage.
