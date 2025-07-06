# SQL Relational Database Final Project

**Point 1:**
```sql
CREATE SCHEMA pandemic;
USE pandemic;

CREATE TABLE countries (
  id INT AUTO_INCREMENT PRIMARY KEY,
  entity VARCHAR(50) NOT NULL,
  code VARCHAR (30) NOT NULL,
  UNIQUE(entity, code)
);

INSERT INTO countries(entity, code)
SELECT DISTINCT entity, code
FROM infectious_cases;

ALTER TABLE infectious_cases
ADD COLUMN id INT NOT NULL PRIMARY KEY AUTO_INCREMENT FIRST,
ADD COLUMN country_id INT;

SET SQL_SAFE_UPDATES = 0;

UPDATE infectious_cases AS ic
JOIN countries AS c ON ic.entity = c.entity AND ic.code = c.code
SET ic.country_id = c.id;

ALTER TABLE infectious_cases
DROP COLUMN entity,
DROP COLUMN code;
```
![Image for the task rdb-fp-p.1](/rdb-fp-p-1.webp)

**Point 2:**
```sql
ALTER TABLE infectious_cases
ADD COLUMN fk_country INT,
ADD CONSTRAINT fk_country
FOREIGN KEY(fk_country) REFERENCES countries(id);

SELECT COUNT(*) FROM infectious_cases;
```
![Image for the task rdb-fp-p.2](/rdb-fp-p-2.webp)

**Point 3:**
```sql
SELECT 
  AVG(Number_rabies) AS average,
  MIN(Number_rabies) as min,
  MAX(Number_rabies) as max,
  SUM(Number_rabies) as sum,
  countries.entity as country_name
FROM infectious_cases
INNER JOIN countries ON country_id = countries.id
WHERE Number_rabies IS NOT NULL AND LENGTH(Number_rabies) != 0
GROUP BY countries.id
ORDER BY AVG(Number_rabies) DESC
LIMIT 10;
```
![Image for the task rdb-fp-p.3](/rdb-fp-p-3.webp)

**Point 4:**
```sql
WITH temp AS (
SELECT
  STR_TO_DATE(CONCAT(Year, "-01-01"), "%Y-%m-%d") AS history_date,
  CURDATE() AS today
FROM infectious_cases
)
SELECT
  history_date,
  today,
  TIMESTAMPDIFF(YEAR, history_date, today) AS year_diff
FROM temp;
```
![Image for the task rdb-fp-p.4](/rdb-fp-p-4.webp)

**Point 5:**
```sql
DROP FUNCTION IF EXISTS average_infections;

DELIMITER //

CREATE FUNCTION average_infections(count_val FLOAT, period INT)
RETURNS FLOAT
NO SQL
DETERMINISTIC
BEGIN
  RETURN count_val / period;
END //

DELIMITER ;
```
```sql
SELECT
  average_infections(CAST(Number_rabies AS FLOAT), 12) AS rabies_per_month,
  average_infections(CAST(Number_malaria AS FLOAT), 4) AS malaria_per_quartal,
  average_infections(CAST(Number_hiv AS FLOAT), 2) AS hiv_per_half_year
FROM infectious_cases
WHERE Number_rabies != ''
  AND Number_malaria != ''
  AND Number_hiv != '';
```
![Image for the task rdb-fp-p.5](/rdb-fp-p-5.webp)
