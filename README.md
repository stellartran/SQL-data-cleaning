# SQL-data-cleaning
This is an educational project on data cleaning and preparation using SQLite. The original database in CSV format is located in the file club_member_info.csv. Here, we will explore the steps that need to be applied to obtain a cleansed version of the dataset.

## Data Introduction
```sql
SELECT * FROM club_member_info cmi LIMIT 10;
```
The result:
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|

## Cleanning Documentation
### 1. Create new table for cleaning
```sql
-- club_member_info definition
CREATE TABLE club_member_info (
	full_name VARCHAR(50),
	age INTEGER,
	martial_status VARCHAR(50),
	email VARCHAR(50),
	phone VARCHAR(50),
	full_address VARCHAR(50),
	job_title VARCHAR(50),
	membership_date VARCHAR(50)
);
```

### 2. Copy all values from original table
```sql
INSERT INTO club_member_info_cleaned
SELECT * FROM club_member_info;
```

### 3. Format full name
```sql
UPDATE club_member_info_cleaned SET full_name = TRIM(full_name);
UPDATE club_member_info_cleaned SET full_name = UPPER(full_name);
```

### 4. Mark error age values with 'unknown' value
```sql
UPDATE club_member_info_cleaned SET age = 'unknown' WHERE age > 100 OR age = '';
```

### 5. Correct column name and values of marital status
```sql
ALTER TABLE club_member_info_cleaned RENAME COLUMN martial_status TO marital_status;

UPDATE club_member_info_cleaned SET marital_status = 'divorced' WHERE marital_status = 'divored';
UPDATE club_member_info_cleaned SET marital_status = 'unknown' WHERE marital_status = '';
```

### 6. Check valid email basically
```SQL
SELECT COUNT(email) FROM club_member_info_cleaned WHERE email = '';
SELECT email FROM club_member_info_cleaned WHERE NOT(email LIKE '%@%.%');
```
*Both queries returned no invalid email addresses. However, SQLite's limited built-in functions mean this method only checks for the presence of '@' and '.', essential for most emails. While it doesn't guarantee 100% validity, it offers partial validation.*

### 7. Mark error phone numbers with 'unknown' value
```sql
UPDATE club_member_info_cleaned SET phone = 'unknown' WHERE NOT(phone LIKE '___-___-____');
```

### 8. Format full address
```sql
UPDATE club_member_info_cleaned SET full_address = TRIM(REPLACE (full_address,',',', '));
UPDATE club_member_info_cleaned SET full_address = REPLACE (full_address,'  ',' ') WHERE full_address LIKE '%  %';
```

Returned result:
|full_address_original|full_name_cleaned|
|------------|-----------------|
|3226 Eastlawn Pass,Temple,Texas|3226 Eastlawn Pass, Temple, Texas|
|4 Harbort Avenue,Fayetteville,North Carolina|4 Harbort Avenue, Fayetteville, North Carolina|

### 9. Mark empty job title with 'unknown' value
```sql
UPDATE club_member_info_cleaned SET job_title = 'unknown' WHERE job_title ='';
```

### 10. Correct year of membership date
```sql
UPDATE club_member_info_cleaned SET membership_date =
SUBSTRING(membership_date, 1, LENGTH(membership_date)-4)||
REPLACE(CAST(SUBSTRING(membership_date, LENGTH(membership_date)-3, 4) AS INTEGER),'19','20')
WHERE CAST(SUBSTRING(membership_date, LENGTH(membership_date)-3, 4) AS INTEGER) < 2000;
```

Returned result:
|original_date|cleaned_date|
|-------------|------------|
|3/12/1921|3/12/2021|
|10/1/1912|10/1/2012|
|2/20/1916|2/20/2016|

### 11. Remove duplicates
*Retreive duplicates*
```sql
SELECT ROWID, * FROM club_member_info_cleaned WHERE email IN (
	SELECT email FROM club_member_info_cleaned GROUP BY email HAVING COUNT(*) > 1)
ORDER BY email;
```
*As you can see from the table below, there are 19 duplicated rows.*
|rowid|full_name|age|marital_status|email|phone|full_address|job_title|membership_date|
|-----|---------|---|--------------|-----|-----|------------|---------|---------------|
|1801|ERWIN HUXTER|25|single|ehuxterm0@marketwatch.com|704-295-3261|0 Homewood Road, Charlotte, North Carolina|Software Test Engineer III|9/29/2017|
|1841|ERWIN HUXTER|25|married|ehuxterm0@marketwatch.com|704-295-3261|0 Homewood Road, Charlotte, North Carolina|Software Test Engineer III|9/29/2017|
|1921|ERWIN HUXTER|25|married|ehuxterm0@marketwatch.com|704-295-3261|0 Homewood Road, Charlotte, North Carolina|Software Test Engineer III|9/29/2017|
|564|GEORGES PREWETT|47|married|gprewettfl@mac.com|571-157-7766|76 Graedel Road, Alexandria, Virginia|Paralegal|10/5/2015|
|804|GEORGES PREWETT|47|single|gprewettfl@mac.com|571-157-7766|76 Graedel Road, Alexandria, Virginia|Paralegal|10/5/2015|
|1176|GARRICK REGLAR|66|divorced|greglar4r@answers.com|214-314-5437|6 Dottie Drive, Dallas, Texas|Safety Technician I|11/22/2015|
|1255|GARRICK REGLAR|66|single|greglar4r@answers.com|214-314-5437|6 Dottie Drive, Dallas, Texas|Safety Technician I|11/22/2015|
|1435|HASKELL BRADEN|unknown|divorced|hbradenri@freewebs.com|510-963-9848|35005 Waubesa Crossing, Berkeley, California|Dental Hygienist|11/4/2015|
|2001|HASKELL BRADEN|32|divorced|hbradenri@freewebs.com|510-963-9848|35005 Waubesa Crossing, Berkeley, California|Dental Hygienist|11/4/2015|
|815|MADDIE MORRALLEE|39|married|mmorralleemj@wordpress.com|712-853-2968|9339 Straubel Center, Sioux City, Iowa|Software Engineer II|1/29/2020|
|1015|MADDIE MORRALLEE|39|divorced|mmorralleemj@wordpress.com|712-853-2968|9339 Straubel Center, Sioux City, Iowa|Software Engineer II|1/29/2020|
|1481|NICKI FILLISKIRK|66|married|nfilliskirkd5@newsvine.com|410-848-2272|7657 Alpine Plaza, Baltimore, Maryland|Geologist IV|6/18/2021|
|1601|NICKI FILLISKIRK|66|married|nfilliskirkd5@newsvine.com|410-848-2272|7657 Alpine Plaza, Baltimore, Maryland|Geologist IV|6/18/2021|
|61|OBED MACCAUGHEN|27|divorced|omaccaughen1o@naver.com|815-990-8611|7069 Valley Edge Alley, Joliet, Illinois|Junior Executive|1/27/2012|
|260|OBED MACCAUGHEN|27|married|omaccaughen1o@naver.com|815-990-8611|7069 Valley Edge Alley, Joliet, Illinois|Junior Executive|1/27/2012|
|291|SEYMOUR LAMBLE|27|married|slamble81@amazon.co.uk|979-346-7243|90691 Veith Place, Bryan, Texas|Budget/Accounting Analyst IV|12/26/2017|
|451|SEYMOUR LAMBLE|27|single|slamble81@amazon.co.uk|979-346-7243|90691 Veith Place, Bryan, Texas|Budget/Accounting Analyst IV|12/26/2017|
|1324|TAMQRAH DUNKERSLEY|36|single|tdunkersley8u@dedecms.com|651-939-2423|0 Colorado Terrace, Saint Paul, Minnesota|VP Sales|6/27/2016|
|1404|TAMQRAH DUNKERSLEY|36|single|tdunkersley8u@dedecms.com|651-939-2423|0 Colorado Terrace, Saint Paul, Minnesota|VP Sales|6/27/2016|

*Remove duplicates*
```sql
DELETE FROM club_member_info_cleaned WHERE ROWID NOT IN (
	SELECT MIN(ROWID) FROM club_member_info_cleaned GROUP BY email
);
```

## Final Result
|full_name|age|marital_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|ADDIE LUSH|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass, Temple, Texas|Assistant Professor|7/31/2013|
|ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue, Fayetteville, North Carolina|Programmer III|5/27/2018|
|SYDEL SHARVELL|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place, Las Vegas, Nevada|Budget/Accounting Analyst I|10/6/2017|
|CONSTANTIN DE LA CRUZ|35|unknown|co3@bloglines.com|402-688-7162|6 Monument Crossing, Omaha, Nebraska|Desktop Support Technician|10/20/2015|
|GAYLOR REDHOLE|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass, New York City, New York|Legal Assistant|5/29/2019|
|WANDA DEL MAR|44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza, Hamilton, Ohio|Human Resources Assistant IV|3/24/2015|
|JOANN KENEALY|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway, Cincinnati, Ohio|Accountant IV|4/17/2013|
|JOETE CUDIFF|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza, Grand Rapids, Michigan|Research Nurse|11/16/2014|
|MENDIE ALEXANDRESCU|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace, New Orleans, Louisiana|Systems Administrator III|3/12/2021|
|FEY KLOSS|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park, Honolulu, Hawaii|Chemical Engineer|11/5/2014|

**Thank you for watching!**
