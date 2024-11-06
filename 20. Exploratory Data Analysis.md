# MySQL Notes
## Exploratory Data Analysis via MySQL
*Reference: @AlexTheAnalyst vid 20 from [Data Analyst Bootcamp](https://www.youtube.com/watch?v=rGx1QNdYzvs&list=PLUaB-1hjhk8FE_XZ87vPPSfHqb6OcM0cF&pp=iAQ)

* aka EDA
* using cleaned data to check insights
* data download from @AlexTheAnalyst MySQL YouTube Series: [Layoffs Dataset](https://www.kaggle.com/datasets/swaptr/layoffs-2022)

```
SELECT *
FROM layoffs_staging2;
```
* this query selects ALL DATA (*) from *layoffs_staging2* table under world_layoffs database

```
SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging2;
```
* this query selects maximum total_laid_off & percentage_laid_off from *layoffs_staging2* table under world_layoffs database

```
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY total_laid_off DESC;
```
* this query selects ALL DATA (*) from *layoffs_staging2* table under world_layoffs database
* WHERE percentage_laid_off = 1
* ORDER BY total_laid_off DESC
  
```
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
```
* this query selects ALL DATA (*) from *layoffs_staging2* table under world_layoffs database
* WHERE percentage_laid_off = 1
* ORDER BY funds_raised_millions DESC

```
SELECT company,  SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
```
* this query selects company & sum of total_laid_off from *layoffs_staging2* table under world_layoffs database
* GROUP BY company
* ORDER BY 2 DESC
* shows the company & total layoffs per company

```
SELECT MIN(`date`), MAX(`date`)
FROM layoffs_staging2;
```
* this query selects min & max date from *layoffs_staging2* table under world_layoffs database
* shows the date coverage of layoffs 

```
SELECT industry,  SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
```
* this query selects industry & sum of total_laid_off from *layoffs_staging2* table under world_layoffs database
* GROUP BY industry
* ORDER BY 2 DESC
* shows industry/ies most impacted by the layoffs

```
SELECT country,  SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
```
* this query selects country & sum of total_laid_off from *layoffs_staging2* table under world_layoffs database
* GROUP BY country
* ORDER BY 2 DESC
* shows country most impacted by the layoffs

```
SELECT `date`,  SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY `date`
ORDER BY 1 DESC;
```
* this query selects date & sum of total_laid_off from *layoffs_staging2* table under world_layoffs database
* GROUP BY date
* ORDER BY 1 DESC
* shows laid offs per dates most impacted

```
SELECT YEAR(`date`),  SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;
```
* this query selects per year & sum of total_laid_off from *layoffs_staging2* table under world_layoffs database
* GROUP BY year
* ORDER BY 1 DESC
* shows laid offs per year most impacted

```
SELECT stage,  SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY stage
ORDER BY 2 DESC;
```
* this query selects by stage of the company & sum of total_laid_off from *layoffs_staging2* table under world_layoffs database
* GROUP BY stage of the company
* ORDER BY 1 DESC
* shows laid offs per stage of the company is at

```
SELECT company,  SUM(percentage_laid_off), AVG(percentage_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
```
* this query selects company, sum & average of percentage_laid_off from layoffs_staging2 table under world_layoffs database
* GROUP BY company
* ORDER BY 2 DESC
* shows the company & percent layoffs per company
> not as useful


### Rolling Total Laid Offs
```
SELECT SUBSTRING(`date`,6,2) AS `MONTH`, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY `MONTH`;
```
* query selects month & sum of total_laid_off from *layoffs_staging2* table under world_layoffs database
> possible issue: it includes months for multiple years

```
SELECT SUBSTRING(`date`,1,7) AS `MONTH`, SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(`date`,1,7) IS NOT NULL
GROUP BY `MONTH`
ORDER BY 1 ASC;
```
* this query selects year & month of sum of total_laid_off from *layoffs_staging2* table under world_layoffs database
* WHERE SUBSTRING(`date`,1,7) IS NOT NULL
* GROUP BY `MONTH`
* ORDER BY 1 ASC

```
WITH Rolling_Total AS
(
SELECT SUBSTRING(`date`,1,7) AS `MONTH`, SUM(total_laid_off) AS total_off
FROM layoffs_staging2
WHERE SUBSTRING(`date`,1,7) IS NOT NULL
GROUP BY `MONTH`
ORDER BY 1 ASC
)
SELECT `MONTH`, total_off
, SUM(total_off) OVER(ORDER BY `MONTH`) AS rolling_total
FROM Rolling_Total;
```
* query creates a Rolling_Total CTE having the following conditions:
  * selects year & month of sum of total_laid_off from layoffs_staging2 table under world_layoffs database
  * WHERE SUBSTRING(`date`,1,7) IS NOT NULL
  * GROUP BY `MONTH`
  * ORDER BY 1 ASC
* from which month and total lay_off are selected to show SUM(total_off) OVER(ORDER BY `MONTH`) AS rolling_total
* shows month by month progression

```
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company, YEAR(`date`)
ORDER BY company ASC;
```
* this query selects company, year, and sum of total_laid_off from layoffs_staging2 table under world_layoffs database
* GROUP BY company & year
* ORDER BY company ASC
* shows the company & total layoffs per company

### Ranking which years companies laid off the most employees
```
WITH Company_Year (company, years, total_laid_off) AS
(
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company, YEAR(`date`)
), Company_Year_Rank AS
(SELECT *, 
DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS Ranking
FROM Company_Year
WHERE years IS NOT NULL
)
SELECT *
FROM Company_Year_Rank
WHERE Ranking <= 5
;
```
* this query creates CTE for company, and total laid offs per year
* this query creates CTE for company ranking per year, top 5 layoffs 
