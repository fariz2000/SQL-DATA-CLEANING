# DATA-ANALYST-PROJECT

-- 1. Import the necesary tables 

SELECT *
FROM layoffs;

-- create a duplicate table which can be edited so that original data is kept untouched.

CREATE TABLE layoff_1
LIKE layoffs;

SELECT *
FROM layoff_1;

-- copy the values from layoffs to layoff_1

INSERT INTO layoff_1
SELECT *
FROM layoffs;

SELECT *
FROM layoff_1;

-- now find for duplicate values and delete them

SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company,location,industry,total_laid_off,percentage_laid_off,'date',stage,country,funds_raised_millions) AS row_id
FROM layoff_1;

-- now create a cte so that a condition can be applied

WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company,location,industry,total_laid_off,percentage_laid_off,'date',stage,country,funds_raised_millions) AS row_id
FROM layoff_1
)
SELECT*
FROM duplicate_cte
WHERE row_id>1;

DELETE
FROM duplicate_cte
WHERE row_id>1;

-- so a cte is not updatable , therefore we have top copy this into another table and then delete from that table
-- so now copy create table into clipboard(from schemas bar) and paste it here. and create a new column as row_id with INT

CREATE TABLE `layoff_2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_id` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

SELECT *
FROM layoff_2;

INSERT INTO layoff_2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company,location,industry,total_laid_off,percentage_laid_off,'date',stage,country,funds_raised_millions) AS row_id
FROM layoff_1;

SELECT *
FROM layoff_2;

DELETE
FROM layoff_2
WHERE row_id>1;

SELECT *
FROM layoff_2
WHERE row_id=2;

-- now the duplicate values have been deleted , now have to standarize the whole table like trim and clear null or blank values.

SELECT *
FROM layoff_2;

SELECT TRIM(company)
FROM layoff_2;

UPDATE layoff_2
SET company=TRIM(company);

SELECT *
FROM layoff_2;

--  chech all locations

SELECT DISTINCT(location)
FROM layoff_2
ORDER BY 1;

-- CHECK ALL INDUSTRIES

SELECT DISTINCT(industry)
FROM layoff_2
ORDER BY 1;

-- there is null and a blank value in industry . fill the null or blank values if possible

SELECT *
FROM layoff_2
WHERE industry is NULL OR industry='';

-- just check if there is another row of airbnb where there is industry

SELECT *
FROM layoff_2
WHERE company='Airbnb';

-- so these null values can be updated . but before that convert these blank values to null values.

UPDATE layoff_2
SET industry=NULL
WHERE industry='';

SELECT *
FROM layoff_2
WHERE industry is NULL;

-- NOW USING JOIN WE WILL FILL THE NULL VALUES

SELECT *
FROM layoff_2 t1
JOIN layoff_2 t2
	ON t1.company=t2.company
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;

-- UPDATE THE VALUES

UPDATE layoff_2 t1
JOIN layoff_2 t2
	ON t1.company=t2.company
SET t1.industry=t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;

SELECT *
FROM layoff_2
WHERE company='Airbnb';

-- now change date to proper date

SELECT `date`,
STR_TO_DATE(`date`,'%m/%d/%Y')
FROM layoff_2;

UPDATE layoff_2
SET `date`=STR_TO_DATE(`date`,'%m/%d/%Y');

SELECT *
FROM layoff_2;

ALTER TABLE layoff_2
MODIFY COLUMN `date` DATE;

SELECT *
FROM layoff_2;

-- DROP ROW_ID

ALTER TABLE layoff_2
DROP column row_id;

SELECT *
FROM layoff_2;

-- this is the required table
