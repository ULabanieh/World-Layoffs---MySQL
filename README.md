# 🏢 Project Overview
---
This is a Data Cleaning and Data Exploration project on a dataset about layoffs in the tech industry from the start of the coronavirus pandemic in 2020 until March 2023 using MySQL. The project is comprised of 2 main parts: Data Cleaning and Exploratory Data Analysis. 

I created a new database called world_layoffs in MySQL Workbench and carried out the following steps to import the dataset (in CSV format):

1. Expand the newly created world_layoffs database
2. Right click on "Tables" and select "Table Data Import Wizard"
3. Browse the csv file location and click Next
4. Select "Create new table" and click Next
5. Review if the data types assigned by MySQL are correct and click Next
6. Click Next and wait for the dataset to be imported

An overview of the dataset we have:

- company: name of the company
- location: city where company is located
- industry: industry the company is specialized in
- total_laid_off: total number of people that have been laid off
- percentage_laid_off: proportion of employees that have been laid off compared to the total number of employees in a company
- date: date of the record
- stage: stage of the company
- country: country where company is located
- funds_raised_millions: how much funds the company has in millions of dollars
# Data Cleaning
---
Here I will showcase the main queries I used to clean the dataset and have it ready for exploration. For full detail, feel free to download the project files and explore them at will.

I divided the data cleaning into 4 steps:
1. Remove Duplicates
2. Standardize Data
3. Null values or blank values
4. Remove unimportant rows/columns

I've created a copy table to carry out the cleaning processes just in case there is some mistake or unintended changes, so that they won't affect the original table
## Remove Duplicates
---
```SQL
SELECT *, 
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`) AS row_num
FROM layoffs_staging;
```

![1446-02-20 12_34_00-MySQL Workbench](https://github.com/user-attachments/assets/a9a92eda-0ab1-480c-9b75-88e88ec476da)


```SQL
WITH duplicate_cte AS
(
SELECT *, 
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1;
```

![1446-02-20 12_37_22-MySQL Workbench](https://github.com/user-attachments/assets/18a2afad-160e-4f9a-9988-303d6481cb05)


Since update statements such as`DELETE` cannot be applied to CTEs, I will create a new staging table and run the following code:

```SQL
DROP TABLE IF EXISTS layoffs_staging2;
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

```SQL
INSERT INTO layoffs_staging2
SELECT *, 
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;
```

```SQL
DELETE
FROM layoffs_staging2
WHERE row_num > 1;
```

```SQL
SELECT *
FROM layoffs_staging2
WHERE row_num > 1;
```

![1446-02-20 12_42_21-MySQL Workbench](https://github.com/user-attachments/assets/5a1679f1-5e01-4c5f-a8c9-a3d80c8c6221)


## Standardize Data
---
```SQL
-- Remove additional space from company column
SELECT company, TRIM(company)
FROM layoffs_staging2;
```

![1446-02-20 12_43_31-MySQL Workbench](https://github.com/user-attachments/assets/044c5a90-a772-4576-ba4a-53b16fafbac0)


```SQL
UPDATE layoffs_staging2
SET company = TRIM(company);
```

```sql
-- Merge similar records 'Crypto', 'Crypto Currency, 'CryptoCurrency' in industry column
SELECT DISTINCT(industry)
FROM layoffs_staging2
ORDER BY 1;
```

![1446-02-20 12_46_22-MySQL Workbench](https://github.com/user-attachments/assets/1663c2c5-1b8c-4c27-b872-891562174f49)


```sql
SELECT * 
FROM layoffs_staging2
WHERE industry LIKE 'Crypto%';
```

![1446-02-20 12_45_50-MySQL Workbench](https://github.com/user-attachments/assets/f2776d2a-d987-4c53-9d43-6930a8b0a2b7)



```SQL
UPDATE layoffs_staging2
SET industry = 'Crypto Currency'
WHERE industry LIKE 'Crypto%';
```


```SQL
-- Standardize Country column

SELECT DISTINCT country
FROM layoffs_staging2
WHERE country LIKE 'United States%';
```

![1446-02-20 12_48_28-MySQL Workbench](https://github.com/user-attachments/assets/8361b10f-33c0-4e76-ac1a-dcc28b05de34)


```SQL
SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging2
ORDER BY 1;
```


![1446-02-20 12_48_57-MySQL Workbench](https://github.com/user-attachments/assets/523b0cd0-0c5e-4d18-9961-0b3558ff30c2)



```SQL
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';
```


```SQL
-- Change data type for date column from text to datetime
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging2;
```

![1446-02-20 12_50_01-MySQL Workbench](https://github.com/user-attachments/assets/28bc7aa6-9ecf-4218-879f-dd5c4e68cf64)


```SQL
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
```

```SQL
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

## Null values and blank values
---
Converted blank values to Null and

```SQL
SELECT *
FROM layoffs_staging2
WHERE industry IS NULL
OR industry = '';
```

![1446-02-20 13_19_25-MySQL Workbench](https://github.com/user-attachments/assets/5a8ef399-5586-49e8-b268-70698ab91319)


```SQL
 UPDATE layoffs_staging2
 SET industry = NULL
 WHERE industry = '';
```


## Remove unimportant rows/columns
---
Removing rows with missing data in total_laid_off and percentage_laid_off columns

```SQL
SELECT * 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

![1446-02-20 13_21_27-MySQL Workbench](https://github.com/user-attachments/assets/d59b4128-994c-41a3-81a6-e718363b6c5b)


```SQL
DELETE
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

Removing row_num column

```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

# Exploratory Data Analysis
---
Here is an overview of the insights I have found while exploring this data with MySQL. For full details, feel free to download the project files and explore them at will.

## Maximum layoffs at a time
---
```SQL
SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging2;
```

![1446-02-20 13_25_13-MySQL Workbench](https://github.com/user-attachments/assets/357710d7-0456-47c8-ba92-7570fb3c8142)


## Companies that had 100% of layoffs
---
```SQL
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY total_laid_off DESC;
```


![1446-02-20 13_26_57-MySQL Workbench](https://github.com/user-attachments/assets/e985d51c-b8e6-4859-91af-fa65ab708eb6)


## Layoffs by company
---
```SQL
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
```


![1446-02-20 13_27_53-MySQL Workbench](https://github.com/user-attachments/assets/b0d90792-8754-4364-a238-be2beb1976b0)


## Layoffs by industry
---
```SQL
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
```


![1446-02-20 13_30_08-MySQL Workbench](https://github.com/user-attachments/assets/2797490e-5a25-45c0-a566-cb8079b6fd03)


## Layoffs by country
---
```sql
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
```

![1446-02-20 13_30_34-MySQL Workbench](https://github.com/user-attachments/assets/8644df43-b11d-44af-8d30-508413e7986f)


## Layoffs and rolling total by month
---
```sql
WITH Rolling_Total AS(
SELECT substring(`date`, 1, 7) AS `month`, SUM(total_laid_off) AS total_off
FROM layoffs_staging2
WHERE substring(`date`, 1, 7) IS NOT NULL
GROUP BY `month`
ORDER BY 1 ASC
)
SELECT `month`, total_off, SUM(total_off) OVER(ORDER BY `month`) AS rolling_total
FROM Rolling_Total;
```

![1446-02-20 13_31_17-MySQL Workbench](https://github.com/user-attachments/assets/df15ee3b-175b-43c7-bfb0-8c8988d3d2fd)


## Top companies by total layoffs per year
---
```SQL
WITH Company_Year (company, years, total_laid_off) AS(
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company, YEAR(`date`)
ORDER BY 3 DESC
), Company_Year_Rank AS(
SELECT *, DENSE_RANK() OVER(PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
FROM Company_Year
WHERE years IS NOT NULL
)
SELECT *
FROM Company_Year_Rank
WHERE ranking <= 5;
```

![1446-02-20 13_31_43-MySQL Workbench](https://github.com/user-attachments/assets/ff9d2898-1edf-436d-88c5-e0f8249f1fa6)
