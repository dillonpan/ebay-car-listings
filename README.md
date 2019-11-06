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

Note: If you run into an error named UnicodeDecodeError, try the next two most popular encoding types (Latin-1 and Windows-1252) in the open() function. Example: autos = pandas.read_csv('autos.csv','r', encoding='Latin-1')

# Opening and Exploring the Data:
```python
import pandas
import numpy

autos = pandas.read_csv('autos.csv', 'r', encoding='utf8')
```
Note: You can use 'print(head(x))' with 'x' being an integer of how many rows from the top you want to print (Ex.'print(head(3))' to print the first 3 rows). Unlike reading the CSV using the function open(), pandas has already seperated the header from the body.

# Clean Columns
```python
print(autos.columns)

Index(['dateCrawled', 'name', 'seller', 'offerType', 'price', 'abtest',
       'vehicleType', 'yearOfRegistration', 'gearbox', 'powerPS', 'model',
       'odometer', 'monthOfRegistration', 'fuelType', 'brand',
       'notRepairedDamage', 'dateCreated', 'nrOfPictures', 'postalCode',
       'lastSeen'],
      dtype='object')
```
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

