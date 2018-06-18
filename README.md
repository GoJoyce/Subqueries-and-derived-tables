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


# Useful Logical Operators

# IF statement
#Question 1: combine a subquery with an IF statement to retrieve a list of unique user_guids with their classification as either an early or late user (based on when their first user entry was created)

	SELECT IF(cleaned_users.first_account<'2014-06-01','early_user','late_user') AS user_type,
	       COUNT(cleaned_users.first_account)
	FROM (SELECT user_guid, MIN(created_at) AS first_account 
	      FROM users
	      GROUP BY user_guid) AS cleaned_users
	GROUP BY user_type

#Question 2: Use an IF expression and the query you wrote in Question 1 as a subquery to determine the number of unique user_guids who reside in the United States (abbreviated "US") and outside of the US.


	SELECT IF(cleaned_users.country='US','In US','Outside US') AS user_location, count(cleaned_users.user_guid) AS num_guids
	FROM (SELECT DISTINCT user_guid, country
	    FROM users
	    WHERE user_guid IS NOT NULL AND country IS NOT NULL) AS cleaned_users
	GROUP BY user_location;

#(cont.)nested IF expression

	SELECT IF(cleaned_users.country='US','In US', 
		  IF(cleaned_users.country='N/A','Not Applicable','Outside US')) AS US_user, 
	      count(cleaned_users.user_guid)   
	FROM (SELECT DISTINCT user_guid, country 
	      FROM users
	      WHERE country IS NOT NULL) AS cleaned_users
	GROUP BY US_user
	
# CASE expressions
#same result as last question:

	SELECT CASE WHEN cleaned_users.country="US" THEN "In US"
		    WHEN cleaned_users.country="N/A" THEN "Not Applicable"
		    ELSE "Outside US"
		    END AS US_user, 
	      count(cleaned_users.user_guid)   
	FROM (SELECT DISTINCT user_guid, country 
	      FROM users
	      WHERE country IS NOT NULL) AS cleaned_users
	GROUP BY US_user
	
#(same result, more concise)

	SELECT CASE cleaned_users.country
		    WHEN "US" THEN "In US"
		    WHEN "N/A" THEN "Not Applicable"
		    ELSE "Outside US"
		    END AS US_user, 
	      count(cleaned_users.user_guid)   
	FROM (SELECT DISTINCT user_guid, country 
	      FROM users
	      WHERE country IS NOT NULL) AS cleaned_users
	GROUP BY US_user
	
#Question 3: Write a query using a CASE statement that outputs 3 columns: dog_guid, dog_fixed, and a third column that reads "neutered" every time there is a 1 in the "dog_fixed" column of dogs, "not neutered" every time there is a value of 0 in the "dog_fixed" column of dogs, and "NULL" every time there is a value of anything else in the "dog_fixed" column. 


	SELECT dog_guid, dog_fixed,
		CASE dog_fixed
			WHEN "1" THEN "neutered"
			WHEN "0" THEN "not neutered"
			END AS neutered
	FROM dogs
	LIMIT 200;

#Question 4: We learned that NULL values should be treated the same as "0" values in the exclude columns of the dogs and users tables. Write a query using a CASE statement that outputs 3 columns: dog_guid, exclude, and a third column that reads "exclude" every time there is a 1 in the "exclude" column of dogs and "keep" every time there is any other value in the exclude column. 


	SELECT dog_guid, exclude,
		CASE exclude
			WHEN "1" THEN "exclude"
			ELSE "keep"
			END AS exclude_cleaned
	FROM dogs
	LIMIT 200;
	
#rewrite:

	%%sql
	SELECT dog_guid, exclude, IF(exclude="1","exclude","keep") AS exclude_cleaned
	FROM dogs
	LIMIT 200;
	
	
#Question 5: Write a query that uses a CASE expression to output 3 columns: dog_guid, weight, and a third column that reads...
"very small" when a dog's weight is 1-10 pounds
"small" when a dog's weight is greater than 10 pounds to 30 pounds
"medium" when a dog's weight is greater than 30 pounds to 50 pounds
"large" when a dog's weight is greater than 50 pounds to 85 pounds
"very large" when a dog's weight is greater than 85 pounds

	
	dog_guid, weight,
		CASE
		WHEN weight<=0 THEN "very small"
		WHEN weight>10 AND weight<=30 THEN "small"
		WHEN weight>30 AND weight<=50 THEN "medium"
		WHEN weight>50 AND weight<=85 THEN "large"
		WHEN weight>85 THEN "very large"
		END AS weight_grouped
	FROM dogs
	LIMIT 200;


#Question 6: For each dog_guid, output its dog_guid, breed_type, number of completed tests, and use an IF statement to include an extra column that reads "Pure_Breed" whenever breed_type equals 'Pure Breed" and "Not_Pure_Breed" whenever breed_type equals anything else. LIMIT your output to 50 rows for troubleshooting. 

	SELECT d.dog_guid AS dogID, d.breed_type AS breed_type, count(c.created_at) AS numtests,
		IF(d.breed_type='Pure Breed','pure_breed', 'not_pure_breed') AS pure_breed
	FROM dogs d, complete_tests c
	WHERE d.dog_guid=c.dog_guid
	GROUP BY dogID, breed_type, pure_breed
	LIMIT 50;
	
	
#Question 7: Write a query that uses a CASE statement to report the number of unique user_guids associated with customers who live in the United States and who are in the following groups of states:

Group 1: New York (abbreviated "NY") or New Jersey (abbreviated "NJ")
Group 2: North Carolina (abbreviated "NC") or South Carolina (abbreviated "SC")
Group 3: California (abbreviated "CA")
Group 4: All other states with non-null values

	SELECT COUNT(DISTINCT user_guid),
		CASE
		WHEN (state="NY" OR state="NJ") THEN "Group 1-NY/NJ"
		WHEN (state="NC" OR state="SC") THEN "Group 2-NC/SC"
		WHEN state="CA" THEN "Group 3-CA"
		ELSE "Group 4-Other"
		END AS state_group
	FROM users
	WHERE country="US" AND state IS NOT NULL
	GROUP BY state_group;
	
	
#Question 8: Write a query that allows you to determine how many unique dog_guids are associated with dogs who are DNA tested and have either stargazer or socialite personality dimensions. 

	SELECT COUNT(DISTINCT dog_guid)
	FROM dogs
	WHERE dna_tested=1 AND (dimension='stargazer' OR dimension='socialite');
