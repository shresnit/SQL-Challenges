# SQL Murder Mystery
<br>
Challenge Source: https://mystery.knightlab.com/#experienced
<br>
<br>

**Case Plot**
<br>
There's been a Murder in SQL City! The SQL Murder Mystery is designed to be both a self-directed lesson to learn SQL concepts and commands and a fun game for experienced SQL users to solve an intriguing crime.
<br>
<br>
A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​. Start by retrieving the corresponding crime scene report from the police department’s database.
<br>

**Police Department Database Tables**
|Table Name                  |Columns in Table|
|----------------------|----------------------|
|crime_scene_report    |date integer, type text, description text, city text|
|drivers_license       |id integer PRIMARY KEY, age integer, height integer, eye_color text, hair_color text, gender text, plate_number text, car_make text, car_model text|
|facebook_event_checkin|person_id integer, event_id integer, event_name text, date integer, FOREIGN KEY (person_id) REFERENCES person(id)|
|interview             |person_id integer, transcript text, FOREIGN KEY (person_id) REFERENCES person(id) |
|get_fit_now_member    |id text PRIMARY KEY, person_id integer, name text, membership_start_date integer, membership_status text, FOREIGN KEY (person_id) REFERENCES person(id)|
|get_fit_now_check_in  |membership_id text, check_in_date integer, check_in_time integer, check_out_time integer, FOREIGN KEY (membership_id) REFERENCES get_fit_now_member(id)|
|income                |ssn CHAR PRIMARY KEY, annual_income integer|
|person                |id integer PRIMARY KEY, name text, license_id integer, address_number integer, address_street_name text, ssn CHAR REFERENCES income (ssn), FOREIGN KEY (license_id) REFERENCES drivers_license (id)|

  <br>
  <br>

## Solution

<br>
1. Finding the related case in crime scene report
<br>
<br>

      SELECT * 
      FROM crime_scene_report
      WHERE type = 'murder'AND city = 'SQL City' AND date = 20180115
      ;
         
|date                  |type  |description                                                                                                                                                                              |city    |
|----------------------|------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------|
|20180115              |murder|Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".|SQL City|

<br>
<br>
2. Identifying and analyzing the witnesses interview transcript
<br>
<br>

      SELECT person_id
	           , license_id
	           , transcript
      FROM interview AS i
      INNER JOIN person AS p
        ON i.person_id = p.id
      WHERE person_id IN (
  		  (SELECT id FROM person
			  WHERE address_street_name ='Northwestern Dr' 
			  ORDER BY address_number DESC
			  LIMIT 1),
			  (SELECT id FROM person
			  WHERE address_street_name = 'Franklin Ave' AND name LIKE 'Annabel%')
		              )
      ;
         
|person_id             |license_id|transcript                                                                                                                                                                               |
|----------------------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|14887                 |118009    |I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".|
|16371                 |490173    |I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.                                                                    |

<br>

<br>
3. Identifying the murderer with witness interview information
<br>
<br>

      SELECT c.id 
        , c.name AS Murderer
      	, b.membership_id
      	, a.membership_status
      	, b.check_in_date
      	, d.plate_number
      FROM get_fit_now_member AS a
      INNER JOIN get_fit_now_check_in AS b
      		ON a.id = b.membership_id
      INNER JOIN person AS c
      		ON a.person_id = c.id
      INNER JOIN drivers_license AS d
      		ON c.license_id	 = d.id
      WHERE b.membership_id LIKE '48Z%'AND
      	  a.membership_status = 'gold' AND
      	  b.check_in_date = 20180109 AND
      	  d.plate_number LIKE '%H42W%'

|id                    |Murderer|membership_id                                                                                                                                                                            |membership_status|check_in_date|plate_number|
|----------------------|--------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|-------------|------------|
|67318                 |Jeremy Bowers|48Z55                                                                                                                                                                                    |gold             |20180109     |0H42W2      |

<br>

<br>
4. Identifying and Analyzing interview transcript of the murderer
<br>
<br>

      SELECT * FROM interview
      WHERE person_id = 67318


|person_id             |transcript|
|----------------------|----------|
|67318                 |I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.|

<br>

<br>
5. Lets find the mastermind
<br>
<br>

        SELECT  B.id
        	, B.name
        	, A.gender
        	, C.annual_income
        	, A.height
        	, A.car_make
        	, A.car_model
        	, D. event_name
        	, COUNT(D.event_name) AS event_count
        FROM drivers_license AS A
        INNER JOIN person AS B
        	ON A.id = B.license_id
        INNER JOIN income AS C
        	ON B.ssn = C.ssn
        INNER JOIN facebook_event_checkin AS D
        	ON B.id = D.person_id
        WHERE (height BETWEEN 65 AND 67) AND
        	  (--hair_color = 'red' AND
        	  gender = 'female' AND
        	  car_make = 'Tesla' AND
        	  car_model = 'Model S' AND
        	  date LIKE '%201712%'	
        	  )

|id                    |name  |gender|annual_income|height|car_make|car_model|event_name          |event_count|
|----------------------|------|------|-------------|------|--------|---------|--------------------|-----------|
|99716                 |Miranda Priestly|female|310000       |66    |Tesla   |Model S  |SQL Symphony Concert|3          |
