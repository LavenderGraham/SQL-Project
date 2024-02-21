# Final-Project-Transforming-and-Analyzing-Data-with-SQL

Final-Project-Transforming-and-Analyzing-Data-with-SQL

## Project/Goals
This project involves analyzing sales, product, and inventory data from a database called e-commerce which sells mechandise common consumer merchandise such as backpags, t-shirts, hoodies, pens, electronics, and stickers online. Many of the products being sold are by the Google company or its subsidiary Nest. Based on this company's sales it appears to either be based in California or market to Californian residents.


## Process
The first step of this project involved making the database and uploading the csv flies into tables in PgAdmin to make the database.

The second step was to clean the data. The first substep of this was removing columns that contain only NULL values or contain only one value which is not needed. For example, a currency column that contained only "USD" and "productRefundAmount" in all_sessions because it contained only NULLs. I also cast data types to numeric when needed. I divided numbers by 1,000,000 in productPrice and totalTransactionRevenue in all_sessions as these had been multiplied by a million. We were instructed to do this, but looking over I found the data made more sense this way, as prices were adjusted from very uncommon prices like $22,990,000 to $22.99. There are many more products in the world that cost $22.99 than $22,990,000. I fixed columns that had excess decimal places by rounding them to two decimal places where appropriate. I also filled in missing information in the product categories in all_sessions by using information in the product names. I checked for mismatches between product catagories and product names. I added a column ("usa_iscal) to specify whether or not a transaction took place in California. I added a column ("BigBrands) to specify whether or not a product was sold by YouTube or Google. I made sure there were no mismatches between products and product categories. I also made sure there were no mismatches between countries and cities. I removed a row about a transaction from somebody in "New York, Canada" assuming that was a flawed piece of data.

Here are the five assigned questions and a brief explanation of how I approached each of them.

Question 1  Which cities and countries have the highest level of transaction revenues on the site?

Using SUM functions and GROUP BY clauses to add up the total transaction revenue by country and city in all_sessions, and ORDER BY DESC to see what is at the top.

Question 2 What is the average number of products ordered from visitors in each city and country?

Using AVG functions and GROUP BY clauses to find average products ordered by country and city in all_sessions, and ORDER BY DESC to see what is at the top.
It is here that I noticed by looking at my top cities for sales that many of them were in California. So I paid attention to California when writing the rest of the scripts for the program.

Question 3 Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

For this question I also used the all_sessions table. For this question I wrote over a dozen queries each using CTEs.

I wrote queries to calculate the percentage of each product that were by Google (or its subsidary Nest) or Youtube. I also wrote queries to check if product categories contained any of the following keywords:
-home
-electronic
-apparel
-bag

Question 4 What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?

For this question I also used the all_sessions table.

I wrote functions using CTEs, SUM() functions, ROW_NUMBER() window functions, while grouping by country, city, and usa_iscal and using an ORDER BY DESC clause to find the top sold product in each country, in each city, and in California.

Question 5 Can we summarize the impact of revenue generated from each city/country?

For this question I also used the all_sessions table.

I wrote queries that use CTEs, SUM() functions, division, and multiplication to find the percentage of revenue that comes from each country, each city, and California.


Self-Made Question 1 Percentage of people who look at the site who actually make a purchase:

I used the all_sessions table. I wrote a query that uses a CTE and adds up the number of sessions with transactions and sessions without transactions on the site and divided them by the total number of sessions and muliplied by 100 to make a percentage.

Self-Made Question 2 How does this vary by whether or not someone is a Californian, a non-Cal American, or from a different country?

I used the all_sessions table. I wrote a query that uses a CTE that categorizes people as Californians, non-Cal Americans, or people from a different country. It then adds up the number of people in each group who made a purchase compared to how many look at the site and expresses that as a percentage.

Self-Made Question 3 What colors of T-shirts are in stock? How many unique products are available in each color? And what is the stock level of each color?

I used the products table for this. I created a temporary table that only contains the t-shirts being sold by the company and a new column of the colors of t shirts sold. Some similar color names were amalgamated. For example in my new "color" column, the colors "purple" and "lavender" were amalgamated to just purple.

I then used a query using a CTE and COUNT() functions to calculate the number and the percentage of unique t-shirt products we sell of each colour.

Self-Made Question 4 How many products are marketed to men versus women?

I used the products table.

I created an query that uses a CTE to count how many unique products contain the keywords "men" or "women" or "boy" or "girl" or do not contain any of these. I adjusted my query to ensure that products containing the keyword "women" are not counted as products containing the keyword "men" because the characters of "men" are a subset of the characters of "women" and this must be adjusted for.

After counting them I expressed them as a percentage.

## Results
Results:

Question 1:

The country with the highest revenue is by far the United States. Cities not available in the demo dataset form a pluarilty of the sales. However the top identified city is San Francisco

| Country       | Total Revenue |
|---------------|---------------|
| United States | 13154.17      |
| Isreal        | 602           |
| Australia     | 358           |
| Canada        | 82.16         |
| Switzerland   | 16.99         |

| City                          | Total Revenue |
|-------------------------------|---------------|
| not available in data demoset | 6092.56       |
| San Francisco                 | 1564.32       |
| Sunnyvale                     | 992.23        |
| Atlanta                       | 854.44        |
| Palo Alto                     | 608           |
| Tel Aviv-Yafo                 | 602           |
| New York                      | 530.36        |
| Mountain View                 | 483.36        |
| Los Angeles                   | 479.48        |
| Chicago                       | 449.52        |
| Seattle                       | 358           |
| Sydney                        | 358           |
| San Jose                      | 262.38        |
| Austin                        | 157.78        |
| Nashville                     | 157           |
| San Bruno                     | 103.77        |
| Toronto                       | 82.16         |
| Houston                       | 38.16         |
| Columbus                      | 21.99         |
| Zurich                        | 16.99         |

Question 2:

The average number of products ordered from Spain is 10. However that is based on one outlying order from someone in Madrid who ordered ten sets of socks. When this order is excluded the country with the most products ordered on average is the United States with four. The rest of the countries have an average number of products of 1.

Madrid has an average number of products of 10. However when the Madrid sock buying outlier was exluded the city with the highest numbers of orders was Salem.

The data for question 2 can be summarized as follows:

| Country       | Average Products Ordered |                                    |
|---------------|--------------------------|------------------------------------|
| Spain         | 10                       | (caused by a single order outlier) |
| United States | 4                        |                                    |
| Columbia      | 1                        |                                    |
| Finland       | 1                        |                                    |
| France        | 1                        |                                    |
| Argentina     | 1                        |                                    |
| Ireland       | 1                        |                                    |
| Mexico        | 1                        |                                    |
| India         | 1                        |                                    |
| Canada        | 1                        |                                    |
| Seattle       | 358                      |                                    |
| Sydney        | 358                      |                                    |
| San Jose      | 262.38                   |                                    |
| Austin        | 157.78                   |                                    |
| Nashville     | 157                      |                                    |
| San Bruno     | 103.77                   |                                    |
| Toronto       | 82.16                    |                                    |
| Houston       | 38.16                    |                                    |
| Columbus      | 21.99                    |                                    |
| Zurich        | 16.99                    |                                    |

| City                          | Average Products Ordered |
|-------------------------------|--------------------------|
| Madrid                        | 10                       |
| Salem                         | 8                        |
| not available in demo dataset | 6.75                     |
| Atlanta                       | 4                        |
| Houston                       | 2                        |
| New York                      | 1.6667                   |
| Detroit                       | 1                        |
| Dublin                        | 1                        |
| Los Angeles                   | 1                        |
| Mountain View                 | 1                        |
| Palo Alto                     | 1                        |
| San Francisco                 | 1                        |
|  San Jose                     | 1                        |
| Seattle                       | 1                        |
| (not set)                     | 1                        |
| Sunnyvale                     | 1                        |
| Ann Arbor                     | 1                        |
| Bengaluru                     | 1                        |
| Chicago                       | 1                        |
| Columbus                      | 1                        |
| Dallas                        | 1                        |



Question 3 Data can be summarized as follows:

| Category    | Percent In USA Orders | Percent in Non-USA Orders |
|-------------|-----------------------|---------------------------|
| Home        | 67.1                  | 50                        |
| Google      | 67.1                  | 75                        |
| Youtube     | 0                     | 0                         |
| Electronics | 3.9                   | 0                         |
| Apparel     | 31.6                  | 50                        |
| Bags        | 5.3                   | 0                         |

| Category    | Percent In California Orders | Percent In Unknown USA Location | Percent In Non-Cal USA Orders |
|-------------|------------------------------|---------------------------------|-------------------------------|
| Home        | 68.8                         | 68                              | 63.2                          |
| Google      | 40.6                         | 36                              | 31.6                          |
| Youtube     | 0                            | 0                               | 0                             |
| Electronics | 3.1                          | 8                               | 0                             |
| Apparel     | 31.2                         | 28                              | 36.9                          |
| Bags        | 3.1                          | 8                               | 5.3                           |

| Category    | Percent In California Orders | Percent Rest Of World Orders |
|-------------|------------------------------|------------------------------|
| Home        | 68.8                         | 60.9                         |
| Google      | 40.6                         | 36.9                         |
| Youtube     | 0                            | 0                            |
| Electronics | 3.1                          | 0                            |
| Apparel     | 31.2                         | 39.1                         |
| Bags        | 3.1                          | 4.3                          |


Question 4 Data can be summarized as follows:

| country       | v2ProductName                             | total_quantity_sold |
|---------------|-------------------------------------------|---------------------|
| United States | Leatherette Journal                       | 65                  |
| Spain         | Waze Dress Socks                          | 10                  |
| Colombia      | YouTube Men's Short Sleeve Hero Tee Black | 1                   |
| Finland       | Google Laptop and Cell Phone Stickers     | 1                   |
| France        | Android Wool Heather Cap Heather/Black    | 1                   |
| Argentina     | Google Alpine Style Backpack              | 1                   |
| Ireland       | Google Laptop Backpack                    | 1                   |
| Mexico        | YouTube Men's Vintage Tee                 | 1                   |
| India         | Google Men's Vintage Badge Tee Black      | 1                   |
| Canada        | Google Youth Short Sleeve T-shirt Green   | 1                   |

| city          | v2ProductName                                           | total_quantity_sold |
|---------------|---------------------------------------------------------|---------------------|
| Other         | Leatherette Journal                                     | 65                  |
| Madrid        | Waze Dress Socks                                        | 10                  |
| Salem         | Red Spiral Google Notebook                              | 8                   |
| Atlanta       | Reusable Shopping Bag                                   | 4                   |
| Houston       | Google Sunglasses                                       | 2                   |
| New York      | Nest® Protect Smoke + CO White Wired Alarm-USA          | 2                   |
| Detroit       | Google 22 oz Water Bottle                               | 1                   |
| Dublin        | Google Laptop Backpack                                  | 1                   |
| Los Angeles   | Google Men's Pullover Hoodie Grey                       | 1                   |
| Mountain View | Google Men's Bike Short Sleeve Tee Charcoal             | 1                   |
| Palo Alto     | Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel | 1                   |
| San Francisco | Nest® Cam Outdoor Security Camera - USA                 | 1                   |
| San Jose      | Nest® Cam Indoor Security Camera - USA                  | 1                   |
| Seattle       | Nest® Cam Indoor Security Camera - USA                  | 1                   |
| (not set)     | Android Wool Heather Cap Heather/Black                  | 1                   |
| Sunnyvale     | Nest® Cam Indoor Security Camera - USA                  | 1                   |
| Ann Arbor     | Google Men's Vintage Badge Tee Black                    | 1                   |
| Bengaluru     | Google Men's Vintage Badge Tee Black                    | 1                   |
| Chicago       | Google Sunglasses                                       | 1                   |
| Columbus      | Google Men's Short Sleeve Badge Tee Charcoal            | 1                   |
| Dallas        | YouTube Leatherette Notebook Combo                      | 1                   |

| Is California? | v2ProductName                                           | total_quantity_sold |
|----------------|---------------------------------------------------------|---------------------|
| Yes            | Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel | 8                   |
| Unknown        | Leatherette Journal                                     | 65                  |
| No             | Red Spiral Google Notebook                              | 2                   |
| Not USA        | Waze Dress Socks                                        | 10                  |


Question 5 Data can be summarized as follows:

| Country       | Percentage Of Revenue |
|---------------|-----------------------|
| United States | 92.5                  |
| Isreal        | 4.3                   |
| Australia     | 2.5                   |
| Canada        | 0.6                   |
| Switzerland   | 0.1                   |

| City                  | Percentage Of Revenue |
|-----------------------|-----------------------|
| Other                 | 56.98267541           |
| San Francisco         | 11.00601408           |
| Sunnyvale             | 6.980986849           |
| Atlanta               | 6.011544101           |
| Palo Alto             | 4.277677559           |
| Tel Aviv-Yafo         | 4.235463636           |
| New York              | 3.731429392           |
| Mountain View         | 3.400753659           |
| Los Angeles           | 3.373455322           |
| Other                 | 14.11753201           |
| Chicago               | 3.162667132           |
| Seattle               | 2.518764089           |
| Sydney                | 2.518764089           |
| San Jose              | 1.846014865           |
| Austin                | 1.110085469           |
| Nashville             | 1.104597659           |
| San Bruno             | 0.730089803           |
| Toronto               | 0.578049323           |
| Houston               | 0.274249788           |
| Columbus              | 0.154714029           |
| Zurich                | 0.119535759           |
| Bhubaneswar           | 0                     |
| Antwerp               | 0                     |
| Wroclaw               | 0                     |
| Montreal              | 0                     |
| Quimper               | 0                     |
| Fremont               | 0                     |
| Vladivostok           | 0                     |
| Chiyoda               | 0                     |
| Longtan District      | 0                     |
| Brussels              | 0                     |
| East Lansing          | 0                     |
| Maracaibo             | 0                     |
| Cupertino             | 0                     |
| Bellingham            | 0                     |
| Kuala Lumpur          | 0                     |
| Santa Monica          | 0                     |
| Kiev                  | 0                     |
| Budapest              | 0                     |
| Zagreb                | 0                     |
| University Park       | 0                     |
| Mississauga           | 0                     |
| Villeneuve-d'Ascq     | 0                     |
| Colombo               | 0                     |
| Indore                | 0                     |
| Orlando               | 0                     |
| Stockholm             | 0                     |
| Pune                  | 0                     |
| Bangkok               | 0                     |
| Moscow                | 0                     |
| Neipu Township        | 0                     |
| Calgary               | 0                     |
| Wrexham               | 0                     |
| Cluj-Napoca           | 0                     |
| San Antonio           | 0                     |
| Nice                  | 0                     |
| Coventry              | 0                     |
| Cape Town             | 0                     |
| Courbevoie            | 0                     |
| Egham                 | 0                     |
| Zhongli District      | 0                     |
| Vancouver             | 0                     |
| Esbjerg               | 0                     |
| Santiago              | 0                     |
| Jersey City           | 0                     |
| Berkeley              | 0                     |
| Copenhagen            | 0                     |
| Oslo                  | 0                     |
| Frankfurt             | 0                     |
| Rio de Janeiro        | 0                     |
| Cambridge             | 0                     |
| Salem                 | 0                     |
| Perth                 | 0                     |
| Hayward               | 0                     |
| Jaipur                | 0                     |
| Charlottetown         | 0                     |
| Cork                  | 0                     |
| Columbia              | 0                     |
| Greer                 | 0                     |
| Makati                | 0                     |
| Berlin                | 0                     |
| Dallas                | 0                     |
| Jacksonville          | 0                     |
| Chuo                  | 0                     |
| Krakow                | 0                     |
| Barcelona             | 0                     |
| Warsaw                | 0                     |
| Edmonton              | 0                     |
| Rosario               | 0                     |
| Shibuya               | 0                     |
| Pittsburgh            | 0                     |
| Jakarta               | 0                     |
| Boston                | 0                     |
| Burnaby               | 0                     |
| Rexburg               | 0                     |
| Ashburn               | 0                     |
| Ipoh                  | 0                     |
| Patna                 | 0                     |
| London                | 0                     |
| Doha                  | 0                     |
| Rotterdam             | 0                     |
| Bellflower            | 0                     |
| Philadelphia          | 0                     |
| Hyderabad             | 0                     |
| Athens                | 0                     |
| Parsippany-Troy Hills | 0                     |
| Pozuelo de Alarcon    | 0                     |
| Medellin              | 0                     |
| Glasgow               | 0                     |
| Monterrey             | 0                     |
| San Diego             | 0                     |
| Madison               | 0                     |
| Council Bluffs        | 0                     |
| Amsterdam             | 0                     |
| Osaka                 | 0                     |
| Lucknow               | 0                     |
| Karachi               | 0                     |
| Kolkata               | 0                     |
| Bogota                | 0                     |
| Wellesley             | 0                     |
| Dublin                | 0                     |
| Iasi                  | 0                     |
| La Victoria           | 0                     |
| San Mateo             | 0                     |
| Buenos Aires          | 0                     |
| Sandton               | 0                     |
| Las Vegas             | 0                     |
| Belo Horizonte        | 0                     |
| Odessa                | 0                     |
| Pleasanton            | 0                     |
| Saint Petersburg      | 0                     |
| Kalamazoo             | 0                     |
| Vilnius               | 0                     |
| Thessaloniki          | 0                     |
| Vienna                | 0                     |
| Brisbane              | 0                     |
| Rome                  | 0                     |
| St. Louis             | 0                     |
| Salford               | 0                     |
| Oviedo                | 0                     |
| Santa Clara           | 0                     |
| Antalya               | 0                     |
| Brno                  | 0                     |
| Bucharest             | 0                     |
| Portland              | 0                     |
| Ghent                 | 0                     |
| Menlo Park            | 0                     |
| El Paso               | 0                     |
| Lisbon                | 0                     |
| Marlboro              | 0                     |
| Eau Claire            | 0                     |
| St. John's            | 0                     |
| Waterloo              | 0                     |
| Sapporo               | 0                     |
| Indianapolis          | 0                     |
| Helsinki              | 0                     |
| Sao Paulo             | 0                     |
| Hong Kong             | 0                     |
| Westville             | 0                     |
| Marseille             | 0                     |
| Montreuil             | 0                     |
| Melbourne             | 0                     |
| South El Monte        | 0                     |
| (not set)             | 0                     |
| Piscataway Township   | 0                     |
| Kitchener             | 0                     |
| Lahore                | 0                     |
| Johnson City          | 0                     |
| Milan                 | 0                     |
| Chico                 | 0                     |
| Poznan                | 0                     |
| Amã                   | 0                     |
| Fortaleza             | 0                     |
| Mumbai                | 0                     |
| Nairobi               | 0                     |
| Raleigh               | 0                     |
| Alexandria            | 0                     |
| Adelaide              | 0                     |
| Ann Arbor             | 0                     |
| Minneapolis           | 0                     |
| Istanbul              | 0                     |
| Culiacan              | 0                     |
| Nanded                | 0                     |
| Redmond               | 0                     |
| Westlake Village      | 0                     |
| Oxford                | 0                     |
| Redwood City          | 0                     |
| Appleton              | 0                     |
| The Dalles            | 0                     |
| Bratislava            | 0                     |
| Timisoara             | 0                     |
| Mexico City           | 0                     |
| Druid Hills           | 0                     |
| Paris                 | 0                     |
| Gothenburg            | 0                     |
| Kovrov                | 0                     |
| Skopje                | 0                     |
| Asuncion              | 0                     |
| Phoenix               | 0                     |
| Bilbao                | 0                     |
| Boulder               | 0                     |
| Ottawa                | 0                     |
| Stanford              | 0                     |
| San Salvador          | 0                     |
| Detroit               | 0                     |
| Minato                | 0                     |
| Manchester            | 0                     |
| Bengaluru             | 0                     |
| Montevideo            | 0                     |
| Richardson            | 0                     |
| Tampa                 | 0                     |
| Avon                  | 0                     |
| Sacramento            | 0                     |
| New Delhi             | 0                     |
| Kansas City           | 0                     |
| Watford               | 0                     |
| Chennai               | 0                     |
| Santa Fe              | 0                     |
| Kirkland              | 0                     |
| Goose Creek           | 0                     |
| Dubai                 | 0                     |
| Irvine                | 0                     |
| Bandung               | 0                     |
| Akron                 | 0                     |
| Beijing               | 0                     |
| Hamburg               | 0                     |
| Prague                | 0                     |
| Milpitas              | 0                     |
| Shinjuku              | 0                     |
| Oakland               | 0                     |
| Munich                | 0                     |
| Sakai                 | 0                     |
| Denver                | 0                     |
| Yokohama              | 0                     |
| Nagoya                | 0                     |
| Charlotte             | 0                     |
| Madrid                | 0                     |
| Ho Chi Minh City      | 0                     |
| Forest Park           | 0                     |
| Ahmedabad             | 0                     |
| Chandigarh            | 0                     |
| Kharkiv               | 0                     |
| Bordeaux              | 0                     |
| Quebec City           | 0                     |
| Seoul                 | 0                     |
| Tempe                 | 0                     |
| Norfolk               | 0                     |
| Lake Oswego           | 0                     |
| Taguig                | 0                     |
| Kharagpur             | 0                     |
| South San Francisco   | 0                     |
| Sherbrooke            | 0                     |
| Izmir                 | 0                     |
| Riyadh                | 0                     |
| Auckland              | 0                     |
| Panama City           | 0                     |
| Fort Worth            | 0                     |
| Stuttgart             | 0                     |
| Petaling Jaya         | 0                     |
| Singapore             | 0                     |
| Belgrade              | 0                     |
| Hanoi                 | 0                     |
| Manila                | 0                     |
| Quezon City           | 0                     |
| LaFayette             | 0                     |
| Gurgaon               | 0                     |
| Washington            | 0                     |

| Is California | Percentage Of Revenue |
|---------------|-----------------------|
| Yes           | 31.2                  |
| Unknown       | 42.9                  |
| No            | 25.5                  |

Self Made Question 1: Percentage of people who look at the site who actually make a purchase?

| Transaction | Percentage Who Made A Transaction |
|-------------|-----------------------------------|
| Yes         | 0.53                              |
| No          | 99.47                             |

Self Made Question 2: How does this vary by whether or not someone is a Californian, a non-Cal American, or from a different country?


| Visitor Category    | Percentage Who Made A Transaction |
|---------------------|-----------------------------------|
| Californian         | 1.22                              |
| Non-Californian USA | 0.99                              |
| Non-USA             | 0.27                              |

Self Made Question 3: What colors of T-shirts are in stock? How many unique products are available in each color? And what is the stock level of each color?

| color    | Number Of Products |
|----------|--------------------|
| Charcoal | 80                 |
| Other    | 79                 |
| Blue     | 66                 |
| Grey     | 58                 |
| White    | 57                 |
| Black    | 54                 |
| Red      | 42                 |
| Green    | 22                 |
| Purple   | 19                 |
| Pink     | 10                 |
| Yellow   | 4                  |

| color    | color_percentage |
|----------|------------------|
| Charcoal | 16.29            |
| Other    | 16.09            |
| Blue     | 13.44            |
| Grey     | 11.81            |
| White    | 11.61            |
| Black    | 11               |
| Red      | 8.55             |
| Green    | 4.48             |
| Purple   | 3.87             |
| Pink     | 2.04             |
| Yellow   | 0.81             |


Self Made Question 4: How many products are marketed to men versus women?


| Product Gendered   Marketing Category | Number Of Products |
|---------------------------------------|--------------------|
| Men                                   | 348                |
| Women                                 | 270                |
| Boy                                   | 0                  |
| Girl                                  | 10                 |
| Non-Gendered                          | 464                |

| Product Gendered   Marketing Category | Percentage Of Products |
|---------------------------------------|------------------------|
| Men                                   | 31.9                   |
| Women                                 | 24.7                   |
| Boy                                   | 0                      |
| Girl                                  | 0.9                    |
| Non-Gendered                          | 42.5                   |


## Challenges 
The challenges of this project was dealing with all the missing data and NULL values and figuring out what the data meant. When I first looked at the data I didn't even know what I was looking at or how to make sense of it. None of the tables were connected.

At first I wondered if I could fill in missing values by calculating them, multiplying columns together to calculate "Total Transaction Revenue" or something. But I could not think of a way to do that. I ended up just working with the data that I had.

I cleaned the data at first by removing columns that had only Nulls or only one value.

I then answered the assigned questions using the all_sessions table using the subset of rows that had recorded transactions or transaction revenue. After removing a bad row this was only 80 rows out of more than 1500. I wondered if what I was doing was crazy when I started working through the questions on Sunday. On Monday I booked an AR as soon as I could get one and was assured that what I was doing was sensible and that I should keep going. On Tuesday when I finished writing and answering my own questions I booked two more ARs where I asked again if it was sensible to do data analysis on 80 rows out of more than 1500. I was told that this is sometimes sensible if this is what the data indicates what we should do.

On all three ARs I talked through my thought process on how I was answering the assigned questions and or writing and answering my own. They all said that my thought process was sensible.

I had an error with git uploading my files to github. So on the day that it is due I will be making my final AR to ask for assistance with that. (I have not done it yet, I'm writing my readme before making that AR as the mentors are not on duty as of me writing this)

## Future Goals

It is easier to come up plans for what to do if data about more transactions came in.

If more transaction data came in for places outside of the United States, in which place more useful analytical conclusions could be made about how nationality impacts what people are buying.

It was noticed by this data analyst that the country that had the second highest amount of sales revenue was Isreal, and that a lot of the sales revenue in the United States was from the San Francisco Bay Area or New York City. The SF Bay Area and New York are both locations with large Jewish populations, as is Isreal, so it would make sense to investigate if there is a connection. Perhaps Jewish people in the SF Bay or New York are doing word-of-mouth advertising of our website to their Jewish family and friends in Isreal. Or perhaps Jewish people in these metro areas that move to Isreal are continuing to use our site.

I think it would be useful to examine how the channel grouping (the column for this is in all_sessions) is influencing what products people buy and how much. It would be useful information to target our advertising.