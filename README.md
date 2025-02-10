# NETFLIX_SQL
# Analyzing the data on Netflix content
# Netflix Movies and TV Shows Data Analysis using SQL

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## NETFLIX Data Analysis

### Create the Database `Netflix_db`

-- Create Table to import the data
```sql
DROP table if exists Netflix;
Create table Netflix
(
	show_id		VARCHAR(10),
	type		VARCHAR(20),
	title		VARCHAR(120),
	director	VARCHAR(250),
	casts		VARCHAR(900),
	country		VARCHAR(150),
	date_added	VARCHAR(25),
	release_year	INT,
	rating		VARCHAR(10),
	duration	VARCHAR(15),
	listed_in	VARCHAR(100),
	description VARCHAR(450)

);
```
-- Improt the data(.CSV file) into the database `Netflix_db`

-- Exploring the data
```sql
Select * from netflix

Select COUNT(DISTINCT(director)) from netflix
WHERE 

select count(case when show_id is null then 1 end) cnt_null from netflix

select count(description) from netflix
where date_added is is null

Select *from netflix
```
### Business Problems and Solutions
--Task 1. Count the Number of Movies vs TV Shows
```sql
select 
	type, 
	count(show_id) 
from netflix
group by 1

--Objective: Determine the distribution of content types on Netflix.
```
--Task 2. Find the Most Common Rating for Movies and TV Shows
```sql
Select type, rating
From
(
Select 
	type, 
	rating, 
	count(rating) as no_ratings,
	rank() over (partition by type order by count(rating) DESC) as rating_rank
from netflix
group by 1,2
order by 1,3 desc
) 
AS Temp
where rating_rank = 1

--Objective: The SQL query identifies the most frequently occurring content rating for each type (Movie or TV Show) in the Netflix dataset. It does this by counting ratings, ranking them by frequency, and selecting the highest-ranked rating per type.
```
--Task 3. List All Movies Released in a Specific Year (e.g., 2020)
```sql
Select title 
from netflix
where release_year = 2020 and type ilike 'movie'

-- Objective: The query retrieves the titles of all movies from the netflix table that were released in the year 2020.
/*
Example of converting changing data type to date and finding the specific year

select title, ADDED_DATE
from (
Select title, cast(date_added as date) as added_date from netflix
)
as t2
where to_char(added_date, 'YYYY') = '2020'
*/
```
--Task 4. Find the Top 5 Countries with the Most Content on Netflix

select 
	Trim(unnest(string_to_array(country, ','))) as each_country,
	count(show_id) as total_content
from netflix
group by 1
order by 2 desc
limit 5

--Objective: The query extracts individual country names from the country column, removes any leading or trailing spaces, counts the number of shows for each country, and returns the top 5 countries with the highest content count in descending order.
/*
select 
	unnest(string_to_array(country, ',')) as each_country
from netflix
*/

/* The query extracts and counts the occurrences of individual countries from a comma-separated list in the country column of the netflix table, then returns the top 5 countries with the highest content count in descending order.
*/

--Task 5.  Identify the Longest Movie

select title,duration from netflix
where type ilike 'movie'
	and duration = (select max(duration) 
					from netflix)

-- Objective: The query retrieves the title and duration of the longest movie from the netflix table by filtering records where type is 'Movie' and matching the maximum duration found in the dataset.

--Task 6. Find Content Added in the Last 5 Years

select *
 from netflix
where cast (date_added as date) >= (current_date - interval '5 years')

-- Objective: The query retrieves all records from the netflix table where the date_added field, converted to a date format, falls within the last 5 years from the current date.

--Task 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

select 
	* 	
from (
	select *,
		unnest(string_to_array(director, ',')) as directors  
	from netflix as t1
	)
where director = 'Rajiv Chilaka'


-- Objective: The query gives individual director names from the director column, expands them into separate rows using UNNEST(), and then filters the records to return only those where the director is 'Rajiv Chilaka'.

--Task 8. List All TV Shows with More Than 5 Seasons

select * 
from netflix
where type ilike 'tv show'
	and --SPLIT_PART(Column, delimiter, position)
	cast(split_part(duration,' ', 1) as integer) > 5
	and split_part(duration,' ',2) ilike '%seasons%'

-- Objective: The query retrieves all TV shows from the netflix table where the number of seasons (extracted from the duration column) is greater than 5, ensuring that the duration value explicitly mentions "seasons"

--Task 9. Count the Number of Content Items in Each Genre.

select genre, count(*)
from (
select *,
	unnest(string_to_array(listed_in, ',')) as genre 
from netflix) as temp
group by 1

-- Objective: The query splits the listed_in column (which contains comma-separated genres) into individual genres, counts the number of shows for each genre, and groups the results accordingly.

--Task 10. Find each year and the average numbers of content release in India on netflix,return top 5 year with highest avg content release!

select 
	added_year,
	count(show_id) as cnt_content
from 
(
select *, 
	to_char(cast(date_added as date),'YYYY') AS Added_year,
	trim(unnest(string_to_array(country, ','))) as countrys 
from netflix) as temp
where countrys ilike 'india'
group by 1
order by 2 desc
limit 5;

-- Objective: The query extracts the year from the date_added column, filters shows available in India, counts the number of shows added per year, and returns the top 5 years with the highest additions in descending order.

--Task 11. List All Movies that are Documentaries

select show_id, title, type, gener, listed_in
from ( 
select *, unnest(string_to_array(listed_in, ',')) as gener from netflix) as temp
where 
	type ilike 'movie'
	and 
	gener ilike 'Documentaries'

-- Another way of doing the same is

Select * 
from netflix
where
	type ilike 'movie'
	and
	listed_in ilike '%Documentaries%'
	
-- Objective: Retrieve all movies classified as documentaries.

--Task 12. Find All Content Without a Director

Select * 
from netflix
where
	director is null

-- Objective: List content that does not have a director.

--Task 13. Find How Many Movies. Actor 'Salman Khan' Appeared in the Last 10 Years.

Select * 
from (
select *,unnest(string_to_array(casts, ',')) actor from netflix
) as temp
where 
	type ilike 'movie'
	and
	actor ilike '%Salman khan%'

-- Objective: Count the number of movies featuring 'Salman Khan' in the last 10 years.

--Task 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

Select actors,count(*) as no_movies
from (
select *,
	unnest(string_to_array(casts, ',')) as actors,
	unnest(string_to_array(country, ',')) as countris
	from netflix
) as temp
where 
	type ilike 'movie'
	and 
	countris ilike 'india'
	and 
	actors is not null
group by 1
order by 2 desc
limit 10;

--Objective: Identify the top 10 actors with the most appearances in Indian-produced movies.

--Task 15. Categorize Content Based on the Presence of 'Kill' and 'Violent' Keywords.

select category, count(*)
from
(select *,
	(case 
		when description ilike '%kill%' or description ilike '%Violent%'
		then 'Bad'
		else 'Good'
	end) as category
	
from netflix) as categorise_content
group by 1

--Objective: It categorizes Netflix content as "Bad" if the description contains words like "kill" or "violent", otherwise as "Good".


--Task 16. Find the percentage of TV Shows vs Movies on Netflix.

SELECT 
    type,
    COUNT(*) AS total_count,
    ROUND((COUNT(*) * 100.0) / (SELECT COUNT(*) FROM netflix), 2) AS percentage
FROM netflix
GROUP BY type;


--Objective: It calculates the total number of titles for each content type (Movie or TV Show) in the Netflix dataset.












