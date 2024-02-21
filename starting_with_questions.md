```
-- Question 1  Which cities and countries have the highest level of transaction revenues on the site?:

SELECT country, SUM(CAST("totalTransactionRevenue" AS NUMERIC)) as total_revenue
FROM all_sessions
WHERE "totalTransactionRevenue" IS NOT NULL
GROUP BY country
ORDER BY total_revenue DESC;

SELECT city, SUM(CAST("totalTransactionRevenue" AS NUMERIC)) as total_revenue
FROM all_sessions
WHERE "totalTransactionRevenue" IS NOT NULL
GROUP BY city
ORDER BY total_revenue DESC;


-- Question 2 What is the average number of products ordered from visitors in each city and country?

SELECT country, AVG("productQuantity") AS avg_products_ordered
FROM all_sessions
WHERE "productQuantity" IS NOT NULL
GROUP BY country
ORDER BY avg_products_ordered DESC;

SELECT city, AVG("productQuantity") AS avg_products_ordered
FROM all_sessions
WHERE "productQuantity" IS NOT NULL
GROUP BY city
ORDER BY avg_products_ordered DESC;


	--HMM orders from Madrid seem to be an anomoly. Is there only one order from Madrid
	--an outlier in terms of number of items?
SELECT *
FROM all_sessions
WHERE city = 'Madrid' AND "productQuantity" IS NOT NULL

	--Ahhh look it's an outlier. There is a single Madrid order containing
	--a non Null product quantity. There's ten people. Let's generate a version of this table
	--without this entry
DROP TABLE IF EXISTS all_sessions_ex_SP;

CREATE TEMP TABLE all_sessions_ex_SP AS
SELECT *
FROM all_sessions
WHERE NOT (city = 'Madrid' AND country = 'Spain' AND "productQuantity" = 10 AND "v2ProductName" = 
		  'Waze Dress Socks');

	--Now let's run these queries again with our new table

SELECT country, AVG("productQuantity") AS avg_products_ordered
FROM all_sessions_ex_SP
WHERE "productQuantity" IS NOT NULL
GROUP BY country
ORDER BY avg_products_ordered DESC;

SELECT city, AVG("productQuantity") AS avg_products_ordered
FROM all_sessions_ex_SP
WHERE "productQuantity" IS NOT NULL
GROUP BY city
ORDER BY avg_products_ordered DESC;


-- Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

WITH ProductOrders AS (
  SELECT "v2ProductCategory", "city", "country", "totalTransactionRevenue", "transactions"
  FROM all_sessions
  WHERE "v2ProductCategory" IS NOT NULL AND "transactions" > 0
)
SELECT "country", "v2ProductCategory", COUNT(*) as order_count
FROM ProductOrders
GROUP BY "country", "v2ProductCategory"
ORDER BY order_count DESC;

--What percentage of US orders are in the Home Category?
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
HomeCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%home%' THEN 1 ELSE 0 END) as IsHomeCategory
    FROM USOrders
)
SELECT 
    SUM(IsHomeCategory) * 100.0 / COUNT(*) AS percentage_home_category
FROM HomeCategoryOrders;

--What percentage of US orders are in the Google Brand?
WITH USOrders AS (
    SELECT "Big Brands"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
GoogleCategoryOrders AS (
    SELECT "Big Brands",
           (CASE WHEN "Big Brands" ILIKE '%nest%' THEN 1 
			WHEN "Big Brands" ILIKE '%google%' THEN 1
			ELSE 0 END) as IsGoogleCategory
    FROM USOrders
)
SELECT 
    SUM(IsGoogleCategory) * 100.0 / COUNT(*) AS percentage_google_category
FROM GoogleCategoryOrders;

		--Wow 67? Two thirds? Let's sanity check this.
		SELECT * FROM all_sessions
		WHERE "transactions" > 0 -- yup it's actually sane
		
--What percentage of US orders are in the YouTube Brand?
WITH USOrders AS (
    SELECT "Big Brands"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
YouTubeCategoryOrders AS (
    SELECT "Big Brands",
           (CASE WHEN "Big Brands" ILIKE '%youtube%' THEN 1 
			ELSE 0 END) as IsYouTubeCategory
    FROM USOrders
)
SELECT 
    SUM(IsYouTubeCategory) * 100.0 / COUNT(*) AS percentage_youtube_category
FROM YouTubeCategoryOrders;

		--Wow 0? Two thirds? Let's sanity check this.
		SELECT * FROM all_sessions
		WHERE "transactions" > 0 -- yup no YouTube products


--What percentage of US orders are in the electronics Category?

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
electronics AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%electronic%' THEN 1 ELSE 0 END) as IsElectronicsCategory
    FROM USOrders
)
SELECT 
    SUM(IsElectronicsCategory) * 100.0 / COUNT(*) AS percentage_electronic_category
FROM electronics;

--What percentage of US orders are in the apperal Category?

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ApparelCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%apparel%' THEN 1 ELSE 0 END) as IsApparelCategory
    FROM USOrders
)
SELECT 
    SUM(IsApparelCategory) * 100.0 / COUNT(*) AS percentage_apperal_category
FROM ApparelCategoryOrders;

--What percentage of US orders are in the bags Category?

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
BagsCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%bag%' THEN 1 ELSE 0 END) as IsBagsCategory
    FROM USOrders
)
SELECT 
    SUM(IsBagsCategory) * 100.0 / COUNT(*) AS percentage_bags_category
FROM BagsCategoryOrders;

--What percentage of non-USA orders are in the Home Category?
WITH nonUSOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
HomeNonUSA AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%home%' THEN 1 ELSE 0 END) as IsHomeCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsHomeCategory) * 100.0 / COUNT(*) AS percentage_home_category
FROM HomeNonUSA;

--What percentage of non-USA orders are in the Google Brand?

WITH nonUSOrders AS (
    SELECT "Big Brands"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
GoogleCategoryOrders AS (
    SELECT "Big Brands",
           (CASE WHEN "Big Brands" ILIKE '%nest%' THEN 1 
			WHEN "Big Brands" ILIKE '%google%' THEN 1
			ELSE 0 END) as IsGoogleCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsGoogleCategory) * 100.0 / COUNT(*) AS percentage_google_category
FROM GoogleCategoryOrders;

--What percentage of non-US orders are in the YouTube Brand?

WITH nonUSOrders AS (
    SELECT "Big Brands"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
YouTubeCategoryOrders AS (
    SELECT "Big Brands",
           (CASE WHEN "Big Brands" ILIKE '%youtube%' THEN 1 
			ELSE 0 END) as IsYouTubeCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsYouTubeCategory) * 100.0 / COUNT(*) AS percentage_youtube_category
FROM YouTubeCategoryOrders;

--What percentage of non-USA orders are in the Electronics Category?

WITH nonUSOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ElectronicsNonUSA AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%electronic%' THEN 1 ELSE 0 END) as IsElectronicsCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsElectronicsCategory) * 100.0 / COUNT(*) AS percentage_electronic_category
FROM ElectronicsNonUSA;

--What percentage of non-USA orders are in the apperal Category?

WITH nonUSOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ApparelNonUSA AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%apparel%' THEN 1 ELSE 0 END) as IsApparelCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsApparelCategory) * 100.0 / COUNT(*) AS percentage_apparel_category
FROM ApparelNonUSA;

--What percentage of non-USA orders are in the bags Category?

WITH nonUSOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
BagsNonUSA AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%bags%' THEN 1 ELSE 0 END) as IsBagsCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsBagsCategory) * 100.0 / COUNT(*) AS percentage_Bags_category
FROM BagsNonUSA;

```
-- Question 1  Which cities and countries have the highest level of transaction revenues on the site?:

SELECT country, SUM(CAST("totalTransactionRevenue" AS NUMERIC)) as total_revenue
FROM all_sessions
WHERE "totalTransactionRevenue" IS NOT NULL
GROUP BY country
ORDER BY total_revenue DESC;

SELECT city, SUM(CAST("totalTransactionRevenue" AS NUMERIC)) as total_revenue
FROM all_sessions
WHERE "totalTransactionRevenue" IS NOT NULL
GROUP BY city
ORDER BY total_revenue DESC;


-- Question 2 What is the average number of products ordered from visitors in each city and country?

SELECT country, AVG("productQuantity") AS avg_products_ordered
FROM all_sessions
WHERE "productQuantity" IS NOT NULL
GROUP BY country
ORDER BY avg_products_ordered DESC;

SELECT city, AVG("productQuantity") AS avg_products_ordered
FROM all_sessions
WHERE "productQuantity" IS NOT NULL
GROUP BY city
ORDER BY avg_products_ordered DESC;


	--HMM orders from Madrid seem to be an anomoly. Is there only one order from Madrid
	--an outlier in terms of number of items?
SELECT *
FROM all_sessions
WHERE city = 'Madrid' AND "productQuantity" IS NOT NULL

	--Ahhh look it's an outlier. There is a single Madrid order containing
	--a non Null product quantity. There's ten people. Let's generate a version of this table
	--without this entry
DROP TABLE IF EXISTS all_sessions_ex_SP;

CREATE TEMP TABLE all_sessions_ex_SP AS
SELECT *
FROM all_sessions
WHERE NOT (city = 'Madrid' AND country = 'Spain' AND "productQuantity" = 10 AND "v2ProductName" = 
		  'Waze Dress Socks');

	--Now let's run these queries again with our new table

SELECT country, AVG("productQuantity") AS avg_products_ordered
FROM all_sessions_ex_SP
WHERE "productQuantity" IS NOT NULL
GROUP BY country
ORDER BY avg_products_ordered DESC;

SELECT city, AVG("productQuantity") AS avg_products_ordered
FROM all_sessions_ex_SP
WHERE "productQuantity" IS NOT NULL
GROUP BY city
ORDER BY avg_products_ordered DESC;


-- Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

WITH ProductOrders AS (
  SELECT "v2ProductCategory", "city", "country", "totalTransactionRevenue", "transactions"
  FROM all_sessions
  WHERE "v2ProductCategory" IS NOT NULL AND "transactions" > 0
)
SELECT "country", "v2ProductCategory", COUNT(*) as order_count
FROM ProductOrders
GROUP BY "country", "v2ProductCategory"
ORDER BY order_count DESC;

--What percentage of US orders are in the Home Category?
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
HomeCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%home%' THEN 1 ELSE 0 END) as IsHomeCategory
    FROM USOrders
)
SELECT 
    SUM(IsHomeCategory) * 100.0 / COUNT(*) AS percentage_home_category
FROM HomeCategoryOrders;

--What percentage of US orders are in the Google Brand?
WITH USOrders AS (
    SELECT "Big Brands"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
GoogleCategoryOrders AS (
    SELECT "Big Brands",
           (CASE WHEN "Big Brands" ILIKE '%nest%' THEN 1 
			WHEN "Big Brands" ILIKE '%google%' THEN 1
			ELSE 0 END) as IsGoogleCategory
    FROM USOrders
)
SELECT 
    SUM(IsGoogleCategory) * 100.0 / COUNT(*) AS percentage_google_category
FROM GoogleCategoryOrders;

		--Wow 67? Two thirds? Let's sanity check this.
		SELECT * FROM all_sessions
		WHERE "transactions" > 0 -- yup it's actually sane
		
--What percentage of US orders are in the YouTube Brand?
WITH USOrders AS (
    SELECT "Big Brands"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
YouTubeCategoryOrders AS (
    SELECT "Big Brands",
           (CASE WHEN "Big Brands" ILIKE '%youtube%' THEN 1 
			ELSE 0 END) as IsYouTubeCategory
    FROM USOrders
)
SELECT 
    SUM(IsYouTubeCategory) * 100.0 / COUNT(*) AS percentage_youtube_category
FROM YouTubeCategoryOrders;

		--Wow 0? Two thirds? Let's sanity check this.
		SELECT * FROM all_sessions
		WHERE "transactions" > 0 -- yup no YouTube products


--What percentage of US orders are in the electronics Category?

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
electronics AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%electronic%' THEN 1 ELSE 0 END) as IsElectronicsCategory
    FROM USOrders
)
SELECT 
    SUM(IsElectronicsCategory) * 100.0 / COUNT(*) AS percentage_electronic_category
FROM electronics;

--What percentage of US orders are in the apperal Category?

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ApparelCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%apparel%' THEN 1 ELSE 0 END) as IsApparelCategory
    FROM USOrders
)
SELECT 
    SUM(IsApparelCategory) * 100.0 / COUNT(*) AS percentage_apperal_category
FROM ApparelCategoryOrders;

--What percentage of US orders are in the bags Category?

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
BagsCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%bag%' THEN 1 ELSE 0 END) as IsBagsCategory
    FROM USOrders
)
SELECT 
    SUM(IsBagsCategory) * 100.0 / COUNT(*) AS percentage_bags_category
FROM BagsCategoryOrders;

--What percentage of non-USA orders are in the Home Category?
WITH nonUSOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
HomeNonUSA AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%home%' THEN 1 ELSE 0 END) as IsHomeCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsHomeCategory) * 100.0 / COUNT(*) AS percentage_home_category
FROM HomeNonUSA;

--What percentage of non-USA orders are in the Google Brand?

WITH nonUSOrders AS (
    SELECT "Big Brands"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
GoogleCategoryOrders AS (
    SELECT "Big Brands",
           (CASE WHEN "Big Brands" ILIKE '%nest%' THEN 1 
			WHEN "Big Brands" ILIKE '%google%' THEN 1
			ELSE 0 END) as IsGoogleCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsGoogleCategory) * 100.0 / COUNT(*) AS percentage_google_category
FROM GoogleCategoryOrders;

--What percentage of non-US orders are in the YouTube Brand?

WITH nonUSOrders AS (
    SELECT "Big Brands"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
YouTubeCategoryOrders AS (
    SELECT "Big Brands",
           (CASE WHEN "Big Brands" ILIKE '%youtube%' THEN 1 
			ELSE 0 END) as IsYouTubeCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsYouTubeCategory) * 100.0 / COUNT(*) AS percentage_youtube_category
FROM YouTubeCategoryOrders;

--What percentage of non-USA orders are in the Electronics Category?

WITH nonUSOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ElectronicsNonUSA AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%electronic%' THEN 1 ELSE 0 END) as IsElectronicsCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsElectronicsCategory) * 100.0 / COUNT(*) AS percentage_electronic_category
FROM ElectronicsNonUSA;

--What percentage of non-USA orders are in the apperal Category?

WITH nonUSOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ApparelNonUSA AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%apparel%' THEN 1 ELSE 0 END) as IsApparelCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsApparelCategory) * 100.0 / COUNT(*) AS percentage_apparel_category
FROM ApparelNonUSA;

--What percentage of non-USA orders are in the bags Category?

WITH nonUSOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" != 'United States'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
BagsNonUSA AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%bags%' THEN 1 ELSE 0 END) as IsBagsCategory
    FROM nonUSOrders
)
SELECT 
    SUM(IsBagsCategory) * 100.0 / COUNT(*) AS percentage_Bags_category
FROM BagsNonUSA;


-- Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?
--part b: Comparing California with non california USA


--Home

--Is In California
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Yes'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
HomeCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%home%' THEN 1 ELSE 0 END) as IsHomeCategory
    FROM USOrders
)
SELECT 
    SUM(IsHomeCategory) * 100.0 / COUNT(*) AS percentage_home_category
FROM HomeCategoryOrders;


--Is In Unknown USA Location
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Unknown'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
HomeCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%home%' THEN 1 ELSE 0 END) as IsHomeCategory
    FROM USOrders
)
SELECT 
    SUM(IsHomeCategory) * 100.0 / COUNT(*) AS percentage_home_category
FROM HomeCategoryOrders;

--Is Not In California
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'No'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
HomeCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%home%' THEN 1 ELSE 0 END) as IsHomeCategory
    FROM USOrders
)
SELECT 
    SUM(IsHomeCategory) * 100.0 / COUNT(*) AS percentage_home_category
FROM HomeCategoryOrders;


--Google

--Is In California
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Yes'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
GoogleCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%nest%' THEN 1 
			WHEN "v2ProductCategory" ILIKE '%google%' THEN 1
			ELSE 0 END) as IsGoogleCategory
    FROM USOrders
)
SELECT 
    SUM(IsGoogleCategory) * 100.0 / COUNT(*) AS percentage_google_category
FROM GoogleCategoryOrders;

--Is In Unknown USA Location
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Unknown'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
GoogleCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%nest%' THEN 1 
			WHEN "v2ProductCategory" ILIKE '%google%' THEN 1
			ELSE 0 END) as IsGoogleCategory
    FROM USOrders
)
SELECT 
    SUM(IsGoogleCategory) * 100.0 / COUNT(*) AS percentage_google_category
FROM GoogleCategoryOrders;

--Is Not In California
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'No'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
GoogleCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%nest%' THEN 1 
			WHEN "v2ProductCategory" ILIKE '%google%' THEN 1
			ELSE 0 END) as IsGoogleCategory
    FROM USOrders
)
SELECT 
    SUM(IsGoogleCategory) * 100.0 / COUNT(*) AS percentage_google_category
FROM GoogleCategoryOrders;

--Electronics

--Is In California

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Yes'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ElectronicCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%electronic%' THEN 1 ELSE 0 END) as IsElectronicCategory
    FROM USOrders
)
SELECT 
    SUM(IsElectronicCategory) * 100.0 / COUNT(*) AS percentage_elecrtronic_category
FROM ElectronicCategoryOrders;

--Is In Unknown USA Location

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Unknown'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ElectronicCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%electronic%' THEN 1 ELSE 0 END) as IsElectronicCategory
    FROM USOrders
)
SELECT 
    SUM(IsElectronicCategory) * 100.0 / COUNT(*) AS percentage_elecrtronic_category
FROM ElectronicCategoryOrders;

--Is Not In California

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'No'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ElectronicCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%electronic%' THEN 1 ELSE 0 END) as IsElectronicCategory
    FROM USOrders
)
SELECT 
    SUM(IsElectronicCategory) * 100.0 / COUNT(*) AS percentage_elecrtronic_category
FROM ElectronicCategoryOrders;


--Apparel

--Is In California
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Yes'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ApparelCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%apparel%' THEN 1 ELSE 0 END) as IsApparelCategory
    FROM USOrders
)
SELECT 
    SUM(IsApparelCategory) * 100.0 / COUNT(*) AS percentage_apparel_category
FROM ApparelCategoryOrders;

--Is In Unknown USA Location
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Unknown'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ApparelCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%apparel%' THEN 1 ELSE 0 END) as IsApparelCategory
    FROM USOrders
)
SELECT 
    SUM(IsApparelCategory) * 100.0 / COUNT(*) AS percentage_apparel_category
FROM ApparelCategoryOrders;

--Is Not In California
WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'No'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ApparelCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%apparel%' THEN 1 ELSE 0 END) as IsApparelCategory
    FROM USOrders
)
SELECT 
    SUM(IsApparelCategory) * 100.0 / COUNT(*) AS percentage_apparel_category
FROM ApparelCategoryOrders;

--Bags

--Is In California

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Yes'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
BagsCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%bags%' THEN 1 ELSE 0 END) as IsBagsCategory
    FROM USOrders
)
SELECT 
    SUM(IsBagsCategory) * 100.0 / COUNT(*) AS percentage_bags_category
FROM BagsCategoryOrders;

--Is In Unknown USA Location

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'Unknown'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
BagsCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%bags%' THEN 1 ELSE 0 END) as IsBagsCategory
    FROM USOrders
)
SELECT 
    SUM(IsBagsCategory) * 100.0 / COUNT(*) AS percentage_bags_category
FROM BagsCategoryOrders;

--Is Not In California

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE "country" = 'United States' AND "usa_iscal" = 'No'
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
BagsCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%bags%' THEN 1 ELSE 0 END) as IsBagsCategory
    FROM USOrders
)
SELECT 
    SUM(IsBagsCategory) * 100.0 / COUNT(*) AS percentage_bags_category
FROM BagsCategoryOrders;


-- Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?
--part c: coding for everywhere other than california USA

--Home

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE ("usa_iscal" = 'No' OR "usa_iscal" IS NULL)
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
HomeCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%home%' THEN 1 ELSE 0 END) as IsHomeCategory
    FROM USOrders
)
SELECT 
    SUM(IsHomeCategory) * 100.0 / COUNT(*) AS percentage_home_category
FROM HomeCategoryOrders;

--Google

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE ("country" = 'No' OR "usa_iscal" IS NOT NULL)
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
GoogleCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%nest%' THEN 1 
			WHEN "v2ProductCategory" ILIKE '%google%' THEN 1
			ELSE 0 END) as IsGoogleCategory
    FROM USOrders
)
SELECT 
    SUM(IsGoogleCategory) * 100.0 / COUNT(*) AS percentage_google_category
FROM GoogleCategoryOrders;

--Electronics

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE ("usa_iscal" = 'No' OR "usa_iscal" IS NULL)
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ElectronicsCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%electronic%' THEN 1 ELSE 0 END) as IsElectronicsCategory
    FROM USOrders
)
SELECT 
    SUM(IsElectronicsCategory) * 100.0 / COUNT(*) AS percentage_electronics_category
FROM ElectronicsCategoryOrders;

--Apparel

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE ("usa_iscal" = 'No' OR "usa_iscal" IS NULL)
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
ApparelCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%apparel%' THEN 1 ELSE 0 END) as IsApparelCategory
    FROM USOrders
)
SELECT 
    SUM(IsApparelCategory) * 100.0 / COUNT(*) AS percentage_apparel_category
FROM ApparelCategoryOrders;

--Bags

WITH USOrders AS (
    SELECT "v2ProductCategory"
    FROM all_sessions
    WHERE ("usa_iscal" = 'No' OR "usa_iscal" IS NULL)
      AND "v2ProductCategory" IS NOT NULL 
      AND "transactions" > 0
),
BagsCategoryOrders AS (
    SELECT "v2ProductCategory",
           (CASE WHEN "v2ProductCategory" ILIKE '%bags%' THEN 1 ELSE 0 END) as IsBagsCategory
    FROM USOrders
)
SELECT 
    SUM(IsBagsCategory) * 100.0 / COUNT(*) AS percentage_bags_category
FROM BagsCategoryOrders;


--Question 4: What is the top-selling product from each city/country? 
--Can we find any pattern worthy of noting in the products sold?

--Part A: Finding the top product by country

WITH RankedProducts AS (
  SELECT
    country,
    "v2ProductName",
    "v2ProductCategory",
    SUM("productQuantity") AS total_quantity_sold,
    ROW_NUMBER() OVER(PARTITION BY country ORDER BY SUM("productQuantity") DESC) AS rank
  FROM all_sessions
  WHERE "productQuantity" IS NOT NULL AND "productQuantity" > 0
  GROUP BY country, "v2ProductName", "v2ProductCategory"
)
SELECT country, "v2ProductName", "v2ProductCategory", "total_quantity_sold"
FROM RankedProducts
WHERE rank = 1;

		--Same except excluding that one Spanish order

WITH RankedProducts AS (
  SELECT
    country,
    "v2ProductName",
    "v2ProductCategory",
    SUM("productQuantity") AS total_quantity_sold,
    ROW_NUMBER() OVER(PARTITION BY country ORDER BY SUM("productQuantity") DESC) AS rank
  FROM all_sessions_ex_SP
  WHERE "productQuantity" IS NOT NULL AND "productQuantity" > 0
  GROUP BY country, "v2ProductName", "v2ProductCategory"
)
SELECT country, "v2ProductName", "v2ProductCategory", "total_quantity_sold"
FROM RankedProducts
WHERE rank = 1;

--Part B: Finding the top product by city

WITH RankedProducts AS (
  SELECT
    city,
    "v2ProductName",
    "v2ProductCategory",
    SUM("productQuantity") AS total_quantity_sold,
    ROW_NUMBER() OVER(PARTITION BY city ORDER BY SUM("productQuantity") DESC) AS rank
  FROM all_sessions
  WHERE "productQuantity" IS NOT NULL AND "productQuantity" > 0
  GROUP BY city, "v2ProductName", "v2ProductCategory"
)
SELECT city, "v2ProductName", "v2ProductCategory", "total_quantity_sold"
FROM RankedProducts
WHERE rank = 1;

		--Same except excluding that one Spanish order

WITH RankedProducts AS (
  SELECT
    city,
    "v2ProductName",
    "v2ProductCategory",
    SUM("productQuantity") AS total_quantity_sold,
    ROW_NUMBER() OVER(PARTITION BY city ORDER BY SUM("productQuantity") DESC) AS rank
  FROM all_sessions_ex_SP
  WHERE "productQuantity" IS NOT NULL AND "productQuantity" > 0
  GROUP BY city, "v2ProductName", "v2ProductCategory"
)
SELECT city, "v2ProductName", "v2ProductCategory", "total_quantity_sold"
FROM RankedProducts
WHERE rank = 1;

--Part C: Finding the top product in California

WITH RankedProducts AS (
  SELECT
    usa_iscal,
    "v2ProductName",
    "v2ProductCategory",
    SUM("productQuantity") AS total_quantity_sold,
    ROW_NUMBER() OVER(PARTITION BY usa_iscal ORDER BY SUM("productQuantity") DESC) AS rank
  FROM all_sessions
  WHERE "productQuantity" IS NOT NULL AND "productQuantity" > 0
  GROUP BY usa_iscal, "v2ProductName", "v2ProductCategory"
)
SELECT usa_iscal, "v2ProductName", "v2ProductCategory", "total_quantity_sold"
FROM RankedProducts
WHERE rank = 1;

		--Same except excluding that one Spanish order

WITH RankedProducts AS (
  SELECT
    usa_iscal,
    "v2ProductName",
    "v2ProductCategory",
    SUM("productQuantity") AS total_quantity_sold,
    ROW_NUMBER() OVER(PARTITION BY usa_iscal ORDER BY SUM("productQuantity") DESC) AS rank
  FROM all_sessions_ex_SP
  WHERE "productQuantity" IS NOT NULL AND "productQuantity" > 0
  GROUP BY usa_iscal, "v2ProductName", "v2ProductCategory"
)
SELECT usa_iscal, "v2ProductName", "v2ProductCategory", "total_quantity_sold"
FROM RankedProducts
WHERE rank = 1;


--Question 5: Can we summarize the impact of revenue generated from each city/country?

--Percentage Of Revenue From Countries
WITH TotalRevenue AS (
    SELECT SUM(CAST("totalTransactionRevenue" AS NUMERIC)) AS total_revenue_global
    FROM all_sessions
    WHERE "totalTransactionRevenue" IS NOT NULL
), CountryRevenue AS (
    SELECT country, SUM(CAST("totalTransactionRevenue" AS NUMERIC)) AS total_revenue
    FROM all_sessions
    WHERE "totalTransactionRevenue" IS NOT NULL
    GROUP BY country
)
SELECT cr.country, cr.total_revenue, (cr.total_revenue / tr.total_revenue_global) * 100 AS percentage_of_total_revenue
FROM CountryRevenue cr, TotalRevenue tr
ORDER BY cr.total_revenue DESC;

--Percentage Of Revenue From Cities
WITH TotalRevenue AS (
    SELECT SUM(CAST("totalTransactionRevenue" AS NUMERIC)) AS total_revenue_global
    FROM all_sessions
    WHERE "totalTransactionRevenue" IS NOT NULL
), CityRevenue AS (
    SELECT city, SUM(CAST("totalTransactionRevenue" AS NUMERIC)) AS total_revenue
    FROM all_sessions
    WHERE "totalTransactionRevenue" IS NOT NULL AND city IS NOT NULL
    GROUP BY city
)
SELECT cr.city, cr.total_revenue, (cr.total_revenue / tr.total_revenue_global) * 100 AS percentage_of_total_revenue
FROM CityRevenue cr, TotalRevenue tr
ORDER BY cr.total_revenue DESC;

--Percentage Of Revenue From California

WITH TotalRevenue AS (
    SELECT SUM(CAST("totalTransactionRevenue" AS NUMERIC)) AS total_revenue_global
    FROM all_sessions
    WHERE "totalTransactionRevenue" IS NOT NULL
),
CaliforniaRevenue AS (
    SELECT 
        SUM(CASE WHEN "usa_iscal" = 'Yes' THEN CAST("totalTransactionRevenue" AS NUMERIC) ELSE 0 END) AS total_revenue_california,
        SUM(CASE WHEN "usa_iscal" = 'Unknown' THEN CAST("totalTransactionRevenue" AS NUMERIC) ELSE 0 END) AS total_revenue_unknown,
        SUM(CASE WHEN "usa_iscal" IS NULL OR "usa_iscal" = 'No' THEN CAST("totalTransactionRevenue" AS NUMERIC) ELSE 0 END) AS total_revenue_not_california
    FROM all_sessions
    WHERE "totalTransactionRevenue" IS NOT NULL
)
SELECT 
    (total_revenue_california / total_revenue_global) * 100 AS percentage_of_revenue_california,
    (total_revenue_unknown / total_revenue_global) * 100 AS percentage_of_revenue_unknown,
    (total_revenue_not_california / total_revenue_global) * 100 AS percentage_of_revenue_not_california
FROM 
    CaliforniaRevenue, TotalRevenue;

--According to The Jewish Community Federation And Endowment Fund: At 350,000 people, the Bay Area 
--Jewish population is the fourth largest 
--in the country. Jews are 4% of the total Bay Area population, higher 
--than the share of Jews in the U.S. population as a whole.

--Also according to The American Jewish Population Project, Jewish people comprise 13% of the
--population of New York City. Making it the largest population of Jewish people outside of Isreal

--Therefore it makes sense that a company that has a lot of sales in the San Francisco Bay Area
--and in New York would have a lot of sales in Isreal because there might be Jewish people in the
--SF Bay and in NYC who are telling their Jewish family and friends about the company in Isreal,
--or Jewish customers might move to Isreal and continue to buy from the company.

```