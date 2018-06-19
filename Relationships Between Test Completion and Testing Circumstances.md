
# 1. During which weekdays do Dognition users complete the most tests?

#Question 1. write a query that will output one column with the original created_at time stamp from each row in the completed_tests table, and another column with a number that represents the day of the week associated with each of those time stamps. Limit your output to 200 rows starting at row 50.


	SELECT created_at, DAYOFWEEK(created_at)
	FROM complete_tests
	LIMIT 49,200;
	
	
#Question 2: Include a CASE statement in the query you wrote in Question 1 to output a third column that provides the weekday name (or an appropriate abbreviation) associated with each created_at time stamp.

	SELECT created_at, DAYOFWEEK(created_at),
	(CASE
			WHEN DAYOFWEEK(created_at)=1 THEN "Su"
			WHEN DAYOFWEEK(created_at)=2 THEN "Mo"
			WHEN DAYOFWEEK(created_at)=3 THEN "Tu"
			WHEN DAYOFWEEK(created_at)=4 THEN "We"
			WHEN DAYOFWEEK(created_at)=5 THEN "Th"
			WHEN DAYOFWEEK(created_at)=6 THEN "Fr"
			WHEN DAYOFWEEK(created_at)=7 THEN "Sa"
			END) AS daylabel
	FROM complete_tests
	LIMIT 49,200;
	
	
#Question 3: report the total number of tests completed on each weekday. Sort the results by the total number of tests completed in descending order. 

	SELECT DAYOFWEEK(created_at),COUNT(created_at) AS numtests,
	(CASE
			WHEN DAYOFWEEK(created_at)=1 THEN "Su"
			WHEN DAYOFWEEK(created_at)=2 THEN "Mo"
			WHEN DAYOFWEEK(created_at)=3 THEN "Tu"
			WHEN DAYOFWEEK(created_at)=4 THEN "We"
			WHEN DAYOFWEEK(created_at)=5 THEN "Th"
			WHEN DAYOFWEEK(created_at)=6 THEN "Fr"
			WHEN DAYOFWEEK(created_at)=7 THEN "Sa"
			END) AS daylabel
	FROM complete_tests
	GROUP BY daylabel
	ORDER BY numtests DESC
	
	
	
#Question 4: exclude the dog_guids that have a value of "1" in the exclude column (Hint: this query will require a join.)

	SELECT DAYOFWEEK(c.created_at),COUNT(c.created_at) AS numtests,
	(CASE
			WHEN DAYOFWEEK(c.created_at)=1 THEN "Su"
			WHEN DAYOFWEEK(c.created_at)=2 THEN "Mo"
			WHEN DAYOFWEEK(c.created_at)=3 THEN "Tu"
			WHEN DAYOFWEEK(c.created_at)=4 THEN "We"
			WHEN DAYOFWEEK(c.created_at)=5 THEN "Th"
			WHEN DAYOFWEEK(c.created_at)=6 THEN "Fr"
			WHEN DAYOFWEEK(c.created_at)=7 THEN "Sa"
			END) AS daylabel
	FROM complete_tests c JOIN dogs d
	ON c.dog_guid=d.dog_guid
	WHERE d.exclude IS NULL OR d.exclude=0
	GROUP BY daylabel
	ORDER BY numtests DESC;
	
	
	
#Question 5: a count of the number of tests completed on each day of the week, excluding all of the dog_guids and user_guids that the Dognition team flagged in the exclude columns.

	SELECT DAYOFWEEK(c.created_at) AS dayasnum, YEAR(c.created_at) AS year,
	COUNT(c.created_at) AS numtests,
	(CASE
					WHEN DAYOFWEEK(c.created_at)=1 THEN "Su"
					WHEN DAYOFWEEK(c.created_at)=2 THEN "Mo"
					WHEN DAYOFWEEK(c.created_at)=3 THEN "Tu"
					WHEN DAYOFWEEK(c.created_at)=4 THEN "We"
					WHEN DAYOFWEEK(c.created_at)=5 THEN "Th"
					WHEN DAYOFWEEK(c.created_at)=6 THEN "Fr"
					WHEN DAYOFWEEK(c.created_at)=7 THEN "Sa"
					END) AS daylabel
			FROM complete_tests c JOIN
			(SELECT DISTINCT dog_guid
			FROM dogs d JOIN users u
			ON d.user_guid=u.user_guid
			WHERE ((u.exclude IS NULL OR u.exclude=0)
			AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
	ON c.dog_guid=dogs_cleaned.dog_guid
	GROUP BY daylabel
	ORDER BY numtests DESC;
	

#Question 6: Adapt your query from Question 8 to provide a count of the number of tests completed on each weekday of each year in the Dognition data set. Exclude all dog_guids and user_guids with a value of "1" in their exclude columns. Sort the output by year in ascending order, and then by the total number of tests completed in descending order. HINT: you will need a function described in one of these references to retrieve the year of each time stamp in the created_at field:


	SELECT DAYOFWEEK(c.created_at) AS dayasnum, YEAR(c.created_at) AS
	year, COUNT(c.created_at) AS numtests,
			(CASE
					WHEN DAYOFWEEK(c.created_at)=1 THEN "Su"
					WHEN DAYOFWEEK(c.created_at)=2 THEN "Mo"
					WHEN DAYOFWEEK(c.created_at)=3 THEN "Tu"
					WHEN DAYOFWEEK(c.created_at)=4 THEN "We"
					WHEN DAYOFWEEK(c.created_at)=5 THEN "Th"
					WHEN DAYOFWEEK(c.created_at)=6 THEN "Fr"
					WHEN DAYOFWEEK(c.created_at)=7 THEN "Sa"
					END) AS daylabel
			FROM complete_tests c JOIN
			(SELECT DISTINCT dog_guid
			FROM dogs d JOIN users u
			ON d.user_guid=u.user_guid
			WHERE ((u.exclude IS NULL OR u.exclude=0)
			AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
	ON c.dog_guid=dogs_cleaned.dog_guid
	GROUP BY year,daylabel
	ORDER BY year ASC, numtests DESC;
	
	
	
#Question 7: adapt your query from Question 9 so that you only examine customers located in the United States, with Hawaii and Alaska residents excluded. HINTS: In this data set, the abbreviation for the United States is "US", the abbreviation for Hawaii is "HI" and the abbreviation for Alaska is "AK".

	SELECT DAYOFWEEK(c.created_at) AS dayasnum, YEAR(c.created_at) AS year,
	COUNT(c.created_at) AS numtests,
			(CASE
					WHEN DAYOFWEEK(c.created_at)=1 THEN "Su"
					WHEN DAYOFWEEK(c.created_at)=2 THEN "Mo"
					WHEN DAYOFWEEK(c.created_at)=3 THEN "Tu"
					WHEN DAYOFWEEK(c.created_at)=4 THEN "We"
					WHEN DAYOFWEEK(c.created_at)=5 THEN "Th"
					WHEN DAYOFWEEK(c.created_at)=6 THEN "Fr"
					WHEN DAYOFWEEK(c.created_at)=7 THEN "Sa"
					END) AS daylabel
			FROM complete_tests c JOIN
			(SELECT DISTINCT dog_guid
			FROM dogs d JOIN users u
			ON d.user_guid=u.user_guid
			WHERE ((u.exclude IS NULL OR u.exclude=0)
			AND u.country="US"
			AND (u.state!="HI" AND u.state!="AK")
			AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
	ON c.dog_guid=dogs_cleaned.dog_guid
	GROUP BY year,daylabel
	ORDER BY year ASC, numtests DESC;
	
	
	
Question 8: provide a count of the number of tests completed on each day of the week, with approximate time zones taken into account, in each year in the Dognition data set. Exclude all dog_guids and user_guids with a value of "1" in their exclude columns. Sort the output by year in ascending order, and then by the total number of tests completed in descending order. HINT: Don't forget to adjust for the time zone in your DAYOFWEEK statement and your CASE statement.


	SELECT DAYOFWEEK(DATE_SUB(created_at, interval 6 hour)) AS dayasnum,
	YEAR(c.created_at) AS year, COUNT(c.created_at) AS numtests,
			(CASE
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=1 THEN "Su"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=2 THEN "Mo"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=3 THEN "Tu"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=4 THEN "We"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=5 THEN "Th"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=6 THEN "Fr"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=7 THEN "Sa"
					END) AS daylabel
			FROM complete_tests c JOIN
			(SELECT DISTINCT dog_guid
			FROM dogs d JOIN users u
			ON d.user_guid=u.user_guid
			WHERE ((u.exclude IS NULL OR u.exclude=0)
			AND u.country="US"
			AND (u.state!="HI" AND u.state!="AK")
			AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
	ON c.dog_guid=dogs_cleaned.dog_guid
	GROUP BY year,daylabel
	ORDER BY year ASC, numtests DESC;
	
Question 9: Adapt your query from Question 8 so that the results are sorted by year in ascending order, and then by the day of the week in the following order: Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday.

	SELECT DAYOFWEEK(DATE_SUB(created_at, interval 6 hour)) AS dayasnum,
	YEAR(c.created_at) AS year, COUNT(c.created_at) AS numtests,
			(CASE
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=1 THEN "Su"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=2 THEN "Mo"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=3 THEN "Tu"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=4 THEN "We"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=5 THEN "Th"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=6 THEN "Fr"
					WHEN DAYOFWEEK(DATE_SUB(created_at, interval 6 hour))=7 THEN "Sa"
					END) AS daylabel
			FROM complete_tests c JOIN
			(SELECT DISTINCT dog_guid
			FROM dogs d JOIN users u
			ON d.user_guid=u.user_guid
			WHERE ((u.exclude IS NULL OR u.exclude=0)
			AND u.country="US"
			AND (u.state!="HI" AND u.state!="AK")
			AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
	ON c.dog_guid=dogs_cleaned.dog_guid
	GROUP BY year,daylabel
	ORDER BY year ASC, FIELD(daylabel,'Mo','Tu','We','Th','Fr','Sa','Su');
	
	
# 2. Which states and countries have the most Dognition users?

Question 10: Which 5 states within the United States have the most Dognition customers, once all dog_guids and user_guids with a value of "1" in their exclude columns are removed? Try using the following general strategy: count how many unique user_guids are associated with dogs in the complete_tests table, break up the counts according to state, sort the results by counts of unique user_guids in descending order, and then limit your output to 5 rows. California ("CA") and New York ("NY") should be at the top of your list.

	SELECT dogs_cleaned.state AS state, COUNT(DISTINCT dogs_cleaned.user_guid) AS
	numusers
	FROM complete_tests c JOIN
			(SELECT DISTINCT dog_guid, u.user_guid, u.state
			FROM dogs d JOIN users u
			ON d.user_guid=u.user_guid
			WHERE ((u.exclude IS NULL OR u.exclude=0)
			AND u.country="US"
			AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
	ON c.dog_guid=dogs_cleaned.dog_guid
	GROUP BY state
	ORDER BY numusers DESC
	LIMIT 5;
	
	
Question 11: Which 10 countries have the most Dognition customers, once all dog_guids and user_guids with a value of "1" in their exclude columns are removed? HINT: don't forget to remove the u.country="US" statement from your WHERE clause.

	SELECT dogs_cleaned.country AS country, COUNT(DISTINCT
	dogs_cleaned.user_guid) AS numusers
			FROM complete_tests c JOIN
			(SELECT DISTINCT dog_guid, u.user_guid, u.country
			FROM dogs d JOIN users u
			ON d.user_guid=u.user_guid
			WHERE ((u.exclude IS NULL OR u.exclude=0)
			AND (d.exclude IS NULL OR d.exclude=0))) AS dogs_cleaned
	ON c.dog_guid=dogs_cleaned.dog_guid
	GROUP BY country
	ORDER BY numusers DESC
	LIMIT 10;
