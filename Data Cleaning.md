# MySQL notes
## Data Cleaning via MySQL
*Reference: @AlexTheAnalyst vid 19 from [Data Analyst Bootcamp](*https://www.youtube.com/watch?v=rGx1QNdYzvs&list=PLUaB-1hjhk8FE_XZ87vPPSfHqb6OcM0cF&pp=iAQ)*

### Summary of Major Steps:
1. import data set
2. create working duplicate from raw data
3. clean data 
  * 3a. Remove Duplicates
  * 3b. Standardize the Data
  * 3c. Null Values or blank values
  * 3d. Remove Any Columns

#### Step 1: import data set
* download from @AlexTheAnalyst MySQL YouTube Series: [Layoffs Dataset](https://www.kaggle.com/datasets/swaptr/layoffs-2022)

```
SELECT *
FROM layoffs;
```
* this query selects ALL DATA (*) from *layoffs* table under world_layoffs database
* data set imported

#### Step 2: create working duplicate/s from raw data
```
CREATE TABLE layoffs_staging3
LIKE layoffs;
```
* this query creates duplicate columns of the raw table; shall be the working table
> best practice **NOT TO WORK ON RAW DATA**
* nothing will appear after running it
* but will show action output success

```
SELECT *
FROM layoffs_staging3;
```
* this query selects ALL DATA (*) from *layoffs_staging3* table under world_layoffs database
* shows columns duplicated from layoffs table **WITHOUT THE DATA**

```
INSERT layoffs_staging3
SELECT *
FROM layoffs;
```
* this query populates the duplicated columns on *layoffs_staging3* table with raw data from *layoffs* table
* nothing will appear after running it
* but will show action output success

```
SELECT *
FROM layoffs_staging3;
```
* this query selects ALL DATA (*) from *layoffs_staging3* table under world_layoffs database
* shall show inserted data now

#### Step 3: clean data
##### 3a. Remove Duplicates
* identify duplicates

````
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, 
percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging3;
````
* this query selects ALL DATA (*) from *layoffs_staging3* table under world_layoffs database
* this query creates row_num column based on partition by all column available

```
WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, 
percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging3
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1;
```
* this query creates CTE named duplicate_cte based on previous row number & partition generated
* then this query selects ALL DATA (*) from the duplicate_cte and filters those with row_num > 1

```
SELECT *
FROM layoffs_staging3
WHERE company = 'Casper';
```
* this query selects ALL DATA (*) from *layoffs_staging3* table under world_layoffs database
* where company name = 'Casper'
* this query will return 3 results, most likely duplicates
* you will need to delete 1, not all, duplicates

###### Creating another table
```
CREATE TABLE `layoffs_staging4` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
> right-click *layoffs_staging3* table >  Send to SQL editor > create statement = automatically generates the table statement; update table name
* this query creates another table named *layoffs_staging4* with the listed columns 
* nothing will appear after running it
* but will show action output success

```
SELECT *
FROM layoffs_staging4;
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database
* this shows columns duplicated from layoffs table **WITHOUT THE DATA**

```
INSERT INTO layoffs_staging4
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, 
percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging3;
```
* this query populates the duplicated columns on *layoffs_staging4* table with raw data from *layoffs_staging3* table
* this generates row_number column based on listed partitions
* nothing will appear after running it
* but will show action output success

###### Showing duplicates
```
SELECT *
FROM layoffs_staging4
WHERE row_num > 1;
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database
* where row_num > 1
* shows the duplicates

###### Deleting duplicates
```
DELETE
FROM layoffs_staging4
WHERE row_num > 1;
```
* this query deletes the duplicates from *layoffs_staging4* table under world_layoffs database

```
SELECT *
FROM layoffs_staging4
WHERE row_num > 1;
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database
* where row_num > 1
* shows blank rows since row_num > 1 or duplicates are already deleted


#### Step 3: clean data
##### 3b) Standardize the Data
* finding issues in your data and fixing it

###### Remove extra spaces via TRIM
```
SELECT company, TRIM(company)
FROM layoffs_staging4;
```
* this query selects company data from *layoffs_staging4* table under world_layoffs database
* TRIM via company

```
UPDATE layoffs_staging4
SET company = TRIM(company);
```
* this query removes extra spaces
* nothing will appear after running it
* but will show action output success

###### Standardize naming 
```
SELECT DISTINCT industry
FROM layoffs_staging4
ORDER BY 1;
```
* this query selects industry data from *layoffs_staging4* table under world_layoffs database
* ordered by 1 = ascending order

```
SELECT *
FROM layoffs_staging4
WHERE industry LIKE 'Crypto%';
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database
* where industry is like 'Crypto%'

```
UPDATE layoffs_staging4
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```
* this query updates *layoffs_staging4* table under world_layoffs database
* where industry is like 'Crypto%', convert it to 'Crypto'
* nothing will appear after running it
* but will show action output success

```
SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging4
ORDER BY 1;
```
* this query selects country data from *layoffs_staging4* table under world_layoffs database
* trims out '.' from country 

```
UPDATE layoffs_staging4
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```
* this query updates *layoffs_staging4* table under world_layoffs database
* where country is like 'United States%', remove '.' trailing it
* nothing will appear after running it
* but will show action output success

```
SELECT *
FROM layoffs_staging4;
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database

###### Updating column formats
```
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging4;
```
* this query selects date data from *layoffs_staging4* table under world_layoffs database
* converts string to date format

```
UPDATE layoffs_staging4
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
```
* this query updates the date format from *layoffs_staging4* table under world_layoffs database
* nothing will appear after running it
* but will show action output success

```
SELECT `date`
FROM layoffs_staging4;
```
* this query shows updated the date format from *layoffs_staging4* table under world_layoffs database

```
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```
* this query updates date column from layoffs_staging2 table under world_layoffs database

```
SELECT `date`
FROM layoffs_staging2;
```
* this query checks altered table via schemas

```
SELECT *
FROM layoffs_staging4;
```
* this query shows latest table

#### Step 3: clean data
##### 3c. Null Values or blank values
* checking null values for 2 columns

```
SELECT *
FROM layoffs_staging4
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database
* WHERE total_laid_off IS NULL, column data is NULL
* AND percentage_laid_off IS NULL, column data is NULL

```
UPDATE layoffs_staging4
SET industry = NULL
WHERE industry = '';
```
* this query updates blank data of under industry column to NULL first
* nothing will appear after running it
* but will show action output success

```
SELECT *
FROM layoffs_staging4
WHERE industry IS NULL
OR industry = '';
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database
* shows where industry is NULL or BLANK = ''

```
SELECT *
FROM layoffs_staging4
WHERE company = 'Airbnb';
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database
* WHERE company = 'Airbnb'
* shows 2 rows

```
 SELECT t1.industry, t2.industry
 FROM layoffs_staging4 t1
 JOIN layoffs_staging4 t2
	ON t1.company = t2.company
    AND t1.location = t2.location
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;
```
* this query shows within *layoffs_staging4* table all NULL industries

```
UPDATE layoffs_staging4 t1
JOIN layoffs_staging4 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```
* this query shows within *layoffs_staging4* table

```
SELECT *
FROM layoffs_staging4
WHERE company LIKE 'Bally%';
```
* this query updates *layoffs_staging4* table under world_layoffs database
* where company is like 'Bally%'

```
SELECT *
FROM layoffs_staging4;
```
* this query shows latest table

#### Step 3: clean data
##### 3d. Remove Any Columns
> NOTE: you have to be confident before deleting data
* removing unused columns 

```
SELECT *
FROM layoffs_staging4
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database
* WHERE total_laid_off IS NULL, column data is NULL
* AND percentage_laid_off IS NULL, column data is NULL

```
SELECT *
FROM layoffs_staging4;
```
* this query selects ALL DATA (*) from *layoffs_staging4* table under world_layoffs database

```
ALTER TABLE layoffs_staging4
DROP COLUMN row_num;
```
* this query drops row_num column
* nothing will appear after running it
* but will show action output success

```
DELETE
FROM layoffs_staging4
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```
* this query deletes row when:
  * WHERE total_laid_off IS NULL
  * AND percentage_laid_off IS NULL
* nothing will appear after running it
* but will show action output success
