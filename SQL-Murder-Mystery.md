```sql

/*

OVERVIEW -
This portfolio project showcases my skills in SQL and problem solving through the SQL Murder Mystery exercise by Knight Lab. 
In this interactive challenge, I dive into a fictional murder mystery, leveraging SQL queries to investigate the crime and identify the culprit.

PROJECT DESCRIPTION -
The SQL Murder Mystery exercise is a captivating puzzle where I utilize SQL to examine a database containing various tables, including crime reports, witness statements, and suspect profiles. 
By formulating SQL queries, I strive to unveil critical information, connect the dots, and solve the mysterious murder case.

RESOURCES -
- SQL Murder Mystery by Knight Lab: [GitHub Repository](https://github.com/NUKnightLab/sql-mysteries)
- Alternatively, you can do all of this in-browser at https://mystery.knightlab.com/

TECHNOLOGIES USED -
- DB Browser for SQLite: Used to interact with the provided SQLite database
- SQL: Employed to write queries and retrieve relevant information from the database

AUTHOR INFORMATION- 
**Author:** Ernest Devaney
**Contact:** hello@ernestdevaney.com
**LinkedIn:** [Ernest Devaney](https://www.linkedin.com/in/ernest-devaney/)

Feel free to reach out if you have any questions or would like to discuss my approach to the SQL Murder Mystery exercise!

*/


--To start, I want to look at all tables names in this database.

SELECT name
FROM sqlite_master
WHERE type='table';

-- Let's see what's inside the crime_scene_report table!

SELECT * 
FROM crime_scene_report LIMIT 10;

-- Now to find the report for January 15th, 2018

SELECT *
FROM crime_scene_report
WHERE date = 20180115 AND city = 'SQL City' AND type = 'murder';

/* 
Bingo! Found the crime scene report. It reads '"Security footage shows that there were 2 witnesses. 
The first witness lives at the last house on "Northwestern Dr". 
The second witness, named Annabel, lives somewhere on "Franklin Ave".'
*/

-- I need to find out more about these witnesses. Let's look into the 'person' table.

SELECT * 
FROM person
LIMIT 10;

--Excellent! Address info is there. Let's start with the first witness.

SELECT *
FROM person
WHERE address_street_name = 'Northwestern Dr'
ORDER BY address_number DESC
LIMIT 1

-- So now I have Morty Schapiro's id number, license id, address, and SSN. I don't know how that's helpful so let's look around more.

SELECT *
FROM interview
WHERE person_id = 14887

/* 
A treasure trove of info here! 
'I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. 
The membership number on the bag started with "48Z". Only gold members have those bags. 
The man got into a car with a plate that included "H42W".
*/

-- I'm going to look into that partial plate number.

SELECT *
FROM drivers_license
WHERE gender = 'male' AND plate_number LIKE '%H42W%';

-- Two results. No names. Okay! Time to investigate the gym angle. Let's look at a bit of the table first.

SELECT *
FROM get_fit_now_member
LIMIT 10;

-- So 'id' will be the membership number.

SELECT *
FROM get_fit_now_member
WHERE membership_status = 'gold' AND id LIKE '48Z%';

-- Two hits. 'Joe Germuska' and 'Jeremy Bowers'. Let's see if either checked in on the day of the crime.

SELECT *
FROM get_fit_now_member gfnm
JOIN get_fit_now_check_in gfnci ON gfnm.id = gfnci.membership_id
WHERE gfnci.check_in_date = 20180115 AND gfnm.membership_status = 'gold' AND gfnci.membership_id LIKE '48Z%';

-- No hits. Okay, on to witness two. 'The second witness, named Annabel, lives somewhere on "Franklin Ave".'

SELECT *
FROM person
WHERE name LIKE '%Annabel%' AND address_street_name = 'Franklin Ave';

-- Annabel Miller. Let's see her statement. Nothing but the facts, ma'am.

SELECT *
FROM interview
WHERE person_id = 16371;

-- Annabel saw the murderer at her gym on Jan 9th! Let's go back and check that date.

-- First I just want to make sure the witness it telling the truth.

SELECT *
FROM get_fit_now_member gfnm
JOIN get_fit_now_check_in gfnci ON gfnm.id = gfnci.membership_id
WHERE gfnci.check_in_date = 20180109 AND gfnm.name = 'Annabel Miller';

-- Her story checks out. Let's see if we can find the killer.

SELECT *
FROM get_fit_now_member gfnm
JOIN get_fit_now_check_in gfnci ON gfnm.id = gfnci.membership_id
WHERE gfnci.check_in_date = 20180109 AND gfnm.membership_status = 'gold' AND gfnci.membership_id LIKE '48Z%';

-- 'Joe Germuska' and 'Jeremy Bowers' again! So, no closer to solving this. Time to start rooting through the other tables!

-- I don't know for sure that 'person_id' on the facebook event table is the same as 'person_id' on the gym membership table, but let's assume for now. Let's see if either suspect was up to anything on the date of the murder.

SELECT * 
FROM facebook_event_checkin
WHERE date = 20180115 AND (person_id = 28819 OR person_id = 67318);

-- So person_id 67318, Jeremy Bowers, was at an event called The Funky Grooves Tour on Jan 15. Let's see if either witness was also there. An accomplice maybe?

SELECT * 
FROM facebook_event_checkin
WHERE date = 20180115 AND (person_id = 16371 OR person_id = 14887);

-- The plot thickens! Both witnesses were at attendance at the same event. I want to see if these three have been to other events together.

SELECT * 
FROM facebook_event_checkin
WHERE person_id IN (16371, 14887, 67318)

-- They only have the one event in common, but it was worth a shot!

/*
I'm going to see if I can get to the killer's driver's license info using what I've uncovered so far.
Of the two people with partial license plates from earlier, can I trace one back to the gym? 
*/

-- Let's get the names of the two people with the partial plate matches. Why didn't I do this earlier?

SELECT *
FROM person 
WHERE license_id = 423327 OR license_id = 664760;

-- Okay! So it's Jeremy Bowers and someone that isn't Jeremy Bowers, so I think it's safe to say the killer is Jeremy Bowers. Let's check.

INSERT INTO solution VALUES (1, 'Jeremy Bowers');
        
        SELECT value FROM solution;

/* 
Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime.
*/

-- Okay then! I did wonder if the killer had given a statement.

SELECT *
FROM interview
WHERE person_id = 67318;

/*
'I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.'
*/

-- Okay then! Let's find this mystery woman.

SELECT *
FROM drivers_license
WHERE 
	height BETWEEN 65 AND 67
	AND hair_color = 'red'
	AND car_make = 'Tesla'
	AND car_model = 'Model S';
	
-- Three hits. License id's are 202298, 291182, 918773.

SELECT *
FROM person
WHERE license_id IN (202298, 291182, 918773);

-- Okay! So their id's are 78881, 90700, 99716. Let's check the fb event table.

SELECT *
FROM facebook_event_checkin
WHERE
	event_name = 'SQL Symphony Concert'
	AND date BETWEEN 20171201 AND 20171231
	AND person_id IN (78881, 90700, 99716);
	
-- That was easy! Got our killer, person_id 99716. Let's take off this mask and see who they really are!

SELECT *
FROM person
WHERE id = 99716;
	
-- The killer is... Miranda Priestly! I knew it all along. Let's check to be sure.

INSERT INTO solution VALUES (1, 'Miranda Priestly');
        
        SELECT value FROM solution;

/*
'Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!'
*/

-- The End



-- ...?


```