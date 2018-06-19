#Question 1: write a query that will output the Dognition personality dimension and total number of tests completed by each unique DogID. This query will be used as an inner subquery in the next question. LIMIT your output to 100 rows for troubleshooting purposes.

	SELECT d.dog_guid AS dogID, d.dimension AS dimension, count(c.created_at) AS numtests
	FROM dogs d, complete_tests c
	WHERE d.dog_guid=c.dog_guid
	GROUP BY dogID
	LIMIT 100;
	
