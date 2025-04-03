# Netflix Movies and TV Shows Data Analysis using SQL
![Netflix Logo](![netflix logo](https://github.com/user-attachments/assets/58781d36-f1bc-419c-8bf7-ee7e913e0d9a)
)

# objective
The objective of this project is to analyze Netflix's content library using SQL by extracting insights on content distribution, ratings, country-wise content, popular actors, and genre trends. It helps in understanding content patterns, audience preferences, and business decisions for OTT platforms.

# Schema
``` sql
CREATE TABLE netflix
(
	show_id	VARCHAR(5),
	type    VARCHAR(10),
	title	VARCHAR(250),
	director VARCHAR(550),
	casts	VARCHAR(1050),
	country	VARCHAR(550),
	date_added	VARCHAR(55),
	release_year	INT,
	rating	VARCHAR(15),
	duration	VARCHAR(15),
	listed_in	VARCHAR(250),
	description VARCHAR(550)
);
```
# Business Problems and Solutions
# 1. Count the number of Movies vs TV Shows
``` sql
select type, 
count(*) as total_content
from netflix group by type;
```

# 2. Find the most common rating for movies and TV shows
```sql
select 
    type,
	rating
from(
    select
         type,
         rating,
	     count (*),
	     rank() over(partition by type order by count(*) desc) as ranking
       from netflix 
       group by 1,2
) as t1
where 
ranking = 1
```

# 3. List all movies released in a specific year (e.g., 2020)
```sql
select * from netflix where release_year = 2020 and type = 'Movie';
```

# 4. Find the top 5 countries with the most content on Netflix
``` sql
select 
     unnest(string_to_array(country,',')) as new_country,
	 count(show_id) as total_content
from netflix
group by 1
order by 2 desc
limit 5
```
# 5. Identify the longest movie
``` sql
select  *  from netflix
         where
		 type = 'Movie'
		 and 
		 duration = (select max(duration) from netflix)
   ```

# 6. Find content added in the last 5 years
```sql
select * from netflix 
where To_DATE(date_added,'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'
SELECT * FROM netflix  
WHERE CAST(date_added AS DATE) >= CURRENT_DATE - INTERVAL '5 years';
```
# 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!
``` sql
select * from netflix where director Ilike '%Rajiv Chilaka%'
```
# 8. List all TV shows with more than 5 seasons
``` sql
SELECT * FROM netflix  
WHERE type = 'TV Show'  
AND CAST(SPLIT_PART(duration, ' ', 1) AS INTEGER) > 5;
```
# 9. Count the number of content items in each genre
``` sql
select unnest(string_to_array(listed_in,',')) as genre,
  count(show_id) as total_content
  from netflix 
  group by genre
  order by total_content desc
```
# 10.Find each year and the average numbers of content release in India on netflix. return top 5 year with highest avg content release!
``` sql
select 
      EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD,YYYY')) as year,
	  count (*) as yearly_content,
	  Round(
      count(*)::numeric/(select count(*) from netflix where country = 'India')::numeric * 100
	  ,2) as avg_content_per_year
from netflix
where country = 'India'
group by 1
```
# 11. List all movies that are documentaries
``` sql
select * from netflix 
where listed_in ILIKE '%documentaries%'
```
# 12. Find all content without a director
``` sql
select * from netflix 
where director is null
```
# 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!
``` sql
select * from netflix 
where casts ILIKE '%Salman Khan%'
AND release_year > EXTRACT(YEAR FROM CURRENT_DATE)-10
```
# 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.
``` sql
Select
unnest(string_to_array(casts,',')) as actors,
count(*) as total_content
from netflix
where country ilike '%india%'
group by 1
order by 2 desc
limit 10
```
# 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.
``` sql
with new_table
as(
select
*,
   case
   when
       description ilike '%kill%' or
	   description ilike '%violence%'  then 'Bad_Content'
	   else 'Good_Content'
	   end category
from netflix
)
select category,
count(*) as total_content
from new_table
group by 1
```
