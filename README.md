# Subqueries-and-derived-tables
Dognition Company Database

#Question 1: Start by loading the sql library and database, and making the Dognition database your default database:
	
	%load_ext sql
	%sql mysql://studentuser:studentpw@mysqlserver/dognitiondb
	%sql USE dognitiondb
	
#Question 2: use a subquery to extract all the data from exam_answers that had test durations that were greater than the average duration for the "Yawn Warm-Up" game? Start by writing the query that gives you the average duration for the "Yawn Warm-Up" game by itself (and don't forget to exclude negative values):


	SELECT *
	FROM exam_answers
	WHERE TIMESTAMPDIFF(minute,start_time,end_time) >
	(SELECT AVG(TIMESTAMPDIFF(minute,start_time,end_time)) AS AvgDuration
	FROM exam_answers
	WHERE TIMESTAMPDIFF(minute,start_time,end_time)>0 AND
	test_name="Yawn Warm-Up");
	

#Question 3: determine the number of unique users in the users table who were NOT in the dogs table using a NOT EXISTS clause?


	SELECT DISTINCT u.user_guid AS uUserID
	FROM users u
	WHERE NOT EXISTS (SELECT d.user_guid
	FROM dogs d
	WHERE u.user_guid =d.user_guid);
	
	
#Question 4: determine a list of each dog a user in the users table owns, with its accompanying # of rows would be outputted per user_id when we left joined the users table on the dogs table (To complete the join on ONLY distinct UserIDs from the users table):


	SELECT DistinctUUsersID.user_guid AS uUserID, d.user_guid AS dUserID, count(*) AS numrows
	FROM (SELECT DISTINCT u.user_guid 
				FROM users u) AS DistinctUUsersID 
	LEFT JOIN dogs d
		 ON DistinctUUsersID.user_guid=d.user_guid
	GROUP BY DistinctUUsersID.user_guid
	ORDER BY numrows DESC
	
#Question 5: Write a query using an IN clause and equijoin syntax that outputs the dog_guid, breed group, state of the owner, and zip of the owner for each distinct dog in the Working, Sporting, and Herding breed groups. (You should get 10,254 rows; the query will be a little slower than some of the others we have practiced)


	SELECT DISTINCT d.dog_guid, d.breed_group, u.state, u.zip
	FROM dogs d, users u
	WHERE breed_group IN ('Working','Sporting','Herding') AND d.user_guid=u.user_guid;
	
	
#(cont.)Write the same query as in Question 6 using traditional join syntax.

	%%sql
	SELECT DISTINCT d.dog_guid, d.breed_group, u.state, u.zip
	FROM dogs d JOIN users u
	ON d.user_guid=u.user_guid
	WHERE breed_group IN ('Working','Sporting','Herding');
	
	
	
#(cont.)We saw earlier that user_guid 'ce7b75bc-7144-11e5-ba71-058fbc01cf0b' still ends up with 1819 rows of output after a left outer join with the dogs table. If you investigate why, you'll find out that's because there are duplicate user_guids in the dogs table as well. How would you adapt the query we wrote earlier (copied below) to only join unique UserIDs from the users table with unique UserIDs from the dog table?

#Rewrite the query above to only LEFT JOIN distinct user(s) from the user table whose user_guid='ce7b75bc-7144-11e5-ba71-058fbc01cf0b'. 

	SELECT DistinctUUsersID.user_guid AS uUserID, d.user_guid AS dUserID, count(*) AS numrows
	FROM (SELECT DISTINCT u.user_guid
			FROM users u
			WHERE u.user_guid='ce7b75bc-7144-11e5-ba71-058fbc01cf0b') AS DistinctUUsersID
	LEFT JOIN dogs d
			ON DistinctUUsersID.user_guid=d.user_guid
	GROUP BY DistinctUUsersID.user_guid
	ORDER BY numrows DESC;


#Question 6: retrieve a full list of all the DogIDs a user in the users table owns, with its accompagnying breed information whenever possible. HOWEVER, BEFORE YOU RUN THE QUERY MAKE SURE TO LIMIT YOUR OUTPUT TO 100 ROWS WITHIN THE SUBQUERY TO THE LEFT OF YOUR JOIN. 

	
	SELECT DistinctUUsersID.user_guid AS uUserID, DistictDUsersID.user_guid AS dUserID, DistictDUsersID.dog_guid AS DogID, DistictDUsersID.breed AS breed
	FROM (SELECT DISTINCT u.user_guid
			FROM users u LIMIT 100) AS DistinctUUsersID
	LEFT JOIN (SELECT DISTINCT d.user_guid, d.dog_guid, d.breed
			FROM dogs d) AS DistictDUsersID
	ON DistinctUUsersID.user_guid=DistictDUsersID.user_guid
	GROUP BY DistinctUUsersID.user_guid;
	
	
#(cont.)Add dog breed and dog weight to the columns that will be included in the final output of your query. In addition, use a HAVING clause to include only UserIDs who would have more than 10 rows in the output of the left join (your output should contain 5 rows).


	SELECT DistictUUsersID.user_guid AS userid, d.breed, d.weight, count(*) AS numrows
	FROM (SELECT DISTINCT u.user_guid
			FROM users u) AS DistictUUsersID
	LEFT JOIN dogs d
			ON DistictUUsersID.user_guid=d.user_guid
	GROUP BY DistictUUsersID.user_guid
	HAVING numrows>10
	ORDER BY numrows DESC;
