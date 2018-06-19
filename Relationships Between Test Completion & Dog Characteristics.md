# 1. Assess whether Dognition personality dimensions are related to the number of tests completed

#Question 1: write a query that will output the Dognition personality dimension and total number of tests completed by each unique DogID. This query will be used as an inner subquery in the next question. LIMIT your output to 100 rows for troubleshooting purposes.

	SELECT d.dog_guid AS dogID, d.dimension AS dimension, count(c.created_at) AS numtests
	FROM dogs d, complete_tests c
	WHERE d.dog_guid=c.dog_guid
	GROUP BY dogID
	LIMIT 100;


#(cont.) Re-write the query in Question 2 using traditional join syntax

	SELECT d.dog_guid AS dogID, d.dimension AS dimension, count(c.created_at) AS numtests
	FROM dogs d JOIN complete_tests c
	ON d.dog_guid=c.dog_guid
	GROUP BY dogID
	LIMIT 100;
	
	
#Question 2: write a query that will output the average number of tests completed by unique dogs in each Dognition personality dimension. Choose either the query in Question 2 or 3 to serve as an inner query in your main query.


	SELECT dimension, AVG(numtests_per_dog.numtests) AS avg_tests_completed
		FROM( SELECT d.dog_guid AS dogID, d.dimension AS dimension, count(c.created_at) AS numtests
		FROM dogs d, complete_tests c
		WHERE d.dog_guid=c.dog_guid
		GROUP BY dogID) AS numtests_per_dog
	GROUP BY numtests_per_dog.dimension;

#OR


	SELECT dimension, AVG(numtests_per_dog.numtests) AS avg_tests_completed
		FROM( SELECT d.dog_guid AS dogID, d.dimension AS dimension, count(c.created_at) AS numtests
		FROM dogs d JOIN complete_tests c
		ON d.dog_guid=c.dog_guid
		GROUP BY dogID) AS numtests_per_dog
	GROUP BY numtests_per_dog.dimension;
	
	
#Question 3: How many unique DogIDs are summarized in the Dognition dimensions labeled "None" or ""?

	SELECT dimension, COUNT(dogID) AS num_dogs
		FROM (SELECT d.dog_guid AS dogID, d.dimension AS dimension
		FROM dogs d JOIN complete_tests c
		ON d.dog_guid=c.dog_guid
		WHERE d.dimension IS NULL OR d.dimension=''
		GROUP BY dogID) AS dogs_in_complete_tests
	GROUP BY dimension;
	
	
#Question 4:To determine whether there are any features that are common to all dogs that have non-NULL empty strings in the dimension column, write a query that outputs the breed, weight, value in the "exclude" column, first or minimum time stamp in the complete_tests table, last or maximum time stamp in the complete_tests table, and total number of tests completed by each unique DogID that has a non-NULL empty string in the dimension column.


	SELECT d.breed, d.weight, d.exclude, MIN(c.created_at) AS first_test,
	MAX(c.created_at) AS last_test,count(c.created_at) AS numtests
	FROM dogs d JOIN complete_tests c
	ON d.dog_guid=c.dog_guid
	WHERE d.dimension IS NOT NULL
	GROUP BY d.dog_guid;
	
#Question 5: Rewrite the query in Question 4 to exclude DogIDs with (1) non-NULL empty strings in the dimension column, (2) NULL values in the dimension column, and (3) values of "1" in the exclude column. NOTES AND HINTS: You cannot use a clause that says d.exclude does not equal 1 to remove rows that have exclude flags, because Dognition clarified that both NULL values and 0 values in the "exclude" column are valid data. 

	SELECT dimension, AVG(numtests_per_dog.numtests) AS avg_tests_completed, COUNT(DISTINCT dogID)
	    FROM( SELECT d.dog_guid AS dogID, d.dimension AS dimension, count(c.created_at)
	    AS numtests
	    FROM dogs d JOIN complete_tests c
	    ON d.dog_guid=c.dog_guid
	    WHERE (dimension IS NOT NULL AND dimension!='') AND (d.exclude IS NULL OR d.exclude=0)
	    GROUP BY dogID) AS numtests_per_dog
	GROUP BY numtests_per_dog.dimension;
	

# 2. Assess whether dog breeds are related to the number of tests completed

#Question 6: Write a query that outputs the breed, weight, value in the "exclude" column, first or minimum time stamp in the complete_tests table, last or maximum time stamp in the complete_tests table, and total number of tests completed by each unique DogID that has a NULL value in the breed_group column.

	SELECT d.breed, d.weight, d.exclude, MIN(c.created_at) AS first_test,
	MAX(c.created_at) AS last_test,count(c.created_at) AS numtests
	FROM dogs d JOIN complete_tests c
	ON d.dog_guid=c.dog_guid
	WHERE breed_group IS NULL
	GROUP BY d.dog_guid;
	
	
#Question 7: Adapt the query in Question 7 to examine the relationship between breed_group and number of tests completed. Exclude DogIDs with values of "1" in the exclude column. 

	SELECT breed_group, AVG(numtests_per_dog.numtests) AS avg_tests_completed, COUNT(DISTINCT dogID)
	    FROM( SELECT d.dog_guid AS dogID, d.breed_group AS breed_group,
	    count(c.created_at) AS numtests
	    FROM dogs d JOIN complete_tests c
	    ON d.dog_guid=c.dog_guid
	    WHERE d.exclude IS NULL OR d.exclude=0
	    GROUP BY dogID) AS numtests_per_dog
	GROUP BY breed_group;
	

#Question 8: Adapt the query in Question 7 to only report results for Sporting, Hound, Herding, and Working breed_groups using an IN clause.

	SELECT breed_group, AVG(numtests_per_dog.numtests) AS avg_tests_completed, COUNT(DISTINCT dogID)
	    FROM( SELECT d.dog_guid AS dogID, d.breed_group AS breed_group,
	    count(c.created_at) AS numtests
	    FROM dogs d JOIN complete_tests c
	    ON d.dog_guid=c.dog_guid
	    WHERE d.exclude IS NULL OR d.exclude=0
	    GROUP BY dogID) AS numtests_per_dog
	GROUP BY breed_group
	HAVING breed_group IN ('Sporting','Hound','Herding','Working');
	
	
#Question 9: Adapt the query in Question 7 to examine the relationship between breed_type and number of tests completed. Exclude DogIDs with values of "1" in the exclude column.

	%%sql
	SELECT breed_type, AVG(numtests_per_dog.numtests) AS avg_tests_completed, COUNT(DISTINCT dogID)
	    FROM( SELECT d.dog_guid AS dogID, d.breed_type AS breed_type,
	    count(c.created_at) AS numtests
	    FROM dogs d JOIN complete_tests c
	    ON d.dog_guid=c.dog_guid
	    WHERE d.exclude IS NULL OR d.exclude=0
	    GROUP BY dogID) AS numtests_per_dog
	GROUP BY breed_type;
	
	
	
# 3. Assess whether dog breeds and neutering are related to the number of tests completed

#Question 10: For each unique DogID, output its dog_guid, breed_type, number of completed tests, and use a CASE statement to include an extra column with a string that reads "Pure_Breed" whenever breed_type equals 'Pure Breed" and "Not_Pure_Breed" whenever breed_type equals anything else. LIMIT your output to 50 rows for troubleshooting.


	SELECT d.dog_guid AS dogID, d.breed_type AS breed_type,
	    CASE WHEN d.breed_type='Pure Breed' THEN 'pure_breed'
	    ELSE 'not_pure_breed'
	    END AS pure_breed,
	count(c.created_at) AS numtests
	FROM dogs d, complete_tests c
	WHERE d.dog_guid=c.dog_guid
	GROUP BY dogID
	LIMIT 50
	
	
#Question 11:  Adapt your queries to examine the relationship between breed_type and number of tests completed by Pure_Breed dogs and non_Pure_Breed dogs. 


	SELECT numtests_per_dog.pure_breed AS pure_breed,
	AVG(numtests_per_dog.numtests) AS avg_tests_completed, COUNT(DISTINCT dogID)
	    FROM( SELECT d.dog_guid AS dogID, d.breed_group AS breed_type,
		CASE WHEN d.breed_type='Pure Breed' THEN 'pure_breed'
		ELSE 'not_pure_breed'
		END AS pure_breed,
	    count(c.created_at) AS numtests
	    FROM dogs d JOIN complete_tests c
	    ON d.dog_guid=c.dog_guid
	    WHERE d.exclude IS NULL OR d.exclude=0
	    GROUP BY dogID) AS numtests_per_dog
	GROUP BY pure_breed;
	
	
#Question 12: Adapt your query from Question 15 to examine the relationship between breed_type, whether or not a dog was neutered (indicated in the dog_fixed field), and number of tests completed by Pure_Breed dogs and non_Pure_Breed dogs. There are DogIDs with null values in the dog_fixed column,


	SELECT numtests_per_dog.pure_breed AS pure_breed, neutered,
	AVG(numtests_per_dog.numtests) AS avg_tests_completed, COUNT(DISTINCT dogID)
	    FROM( SELECT d.dog_guid AS dogID, d.breed_group AS breed_type, d.dog_fixed AS neutered,
		CASE WHEN d.breed_type='Pure Breed' THEN 'pure_breed'
		ELSE 'not_pure_breed'
		END AS pure_breed,
	    count(c.created_at) AS numtests
	    FROM dogs d JOIN complete_tests c
	    ON d.dog_guid=c.dog_guid
	    WHERE d.exclude IS NULL OR d.exclude=0
	    GROUP BY dogID) AS numtests_per_dog
	GROUP BY pure_breed, neutered;
	
	
# 4. Other dog features that might be related to the number of tests completed, and a note about using averages as summary metrics

#Question 13: Adapt your query from Question 5 to include a column with the standard deviation for the number of tests completed by each Dognition personality dimension.

	SELECT dimension, AVG(numtests) AS avg_tests_completed, COUNT(DISTINCT dogID), STDDEV(numtests)
	    FROM( SELECT d.dog_guid AS dogID, d.dimension AS dimension, count(c.created_at) AS numtests
	    FROM dogs d JOIN complete_tests c
	    ON d.dog_guid=c.dog_guid
	    WHERE (dimension IS NOT NULL AND dimension!='') AND (d.exclude IS NULL OR d.exclude=0)
	    GROUP BY dogID) AS numtests_per_dog
	GROUP BY numtests_per_dog.dimension;
	
	
#Question 14: Write a query that calculates the average amount of time it took each dog breed_type to complete all of the tests in the exam_answers table. Exclude negative durations from the calculation, and include a column that calculates the standard deviation of durations for each breed_type group:

	SELECT d.breed_type AS breed_type,
	    AVG(TIMESTAMPDIFF(minute,e.start_time,e.end_time)) AS AvgDuration,
	    STDDEV(TIMESTAMPDIFF(minute,e.start_time,e.end_time)) AS StdDevDuration
	FROM dogs d JOIN exam_answers e
	ON d.dog_guid=e.dog_guid
	WHERE TIMESTAMPDIFF(minute,e.start_time,e.end_time)>0
	GROUP BY breed_type;
