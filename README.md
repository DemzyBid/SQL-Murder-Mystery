SQL Murder Mystery — Solved!

A real SQL investigation using the SQL Murder Mystery database created by Knight Lab at Northwestern University.
---

About This Project

A murder took place in SQL City on January 15, 2018. Using nothing but SQL queries, I tracked down the murderer — and the mastermind who hired them — by following clues across multiple database tables.
This was my first real SQL project, completed as part of my data analytics training journey documented on LinkedIn under #DataAnalystDiaries.
---

The Challenge
> *"A crime has taken place and the detective needs your help. The crime was a murder that occurred sometime on Jan. 15, 2018 and took place in SQL City. Start by retrieving the crime scene report from the police department's database."*
The goal: use SQL queries to identify both the murderer AND the mastermind behind the crime.
---

Tools Used

SQL — querying the Murder Mystery database
Platform — mystery.knightlab.com
Concepts applied:
SELECT, FROM, WHERE
AND, OR, LIKE
ORDER BY, LIMIT
JOIN (connecting multiple tables)
IN, BETWEEN
COUNT, GROUP BY, HAVING
---

Database Schema

The database contains 8 tables:
Table	Contents
`crime_scene_report`	Crime records — date, type, description, city
`person`	Personal details — name, address, license ID, SSN
`interview`	Witness and suspect transcripts
`drivers_license`	License details — height, hair colour, car, plate number
`get_fit_now_member`	Gym membership records
`get_fit_now_check_in`	Gym attendance records
`facebook_event_checkin`	Event attendance records
`income`	Annual income records
---

The Investigation

Step 1 — Find the Crime Scene Report
```sql
SELECT * 
FROM crime_scene_report
WHERE type = 'murder'
AND date = 20180115
AND city = 'SQL City'
```
Finding: Two witnesses. Witness 1 lives at the last house on Northwestern Dr. Witness 2 named Annabel lives on Franklin Ave.
---

Step 2 — Identify the Witnesses
```sql
-- Witness 1: Last house on Northwestern Dr
SELECT * 
FROM person
WHERE address_street_name = 'Northwestern Dr'
ORDER BY address_number DESC
LIMIT 1

-- Witness 2: Annabel on Franklin Ave
SELECT * 
FROM person
WHERE name LIKE '%Annabel%'
AND address_street_name = 'Franklin Ave'
```
Finding:
Witness 1: Morty Schapiro — Person ID 14887
Witness 2: Annabel Miller — Person ID 16371
---

Step 3 — Pull the Witness Interviews
```sql
SELECT * 
FROM interview
WHERE person_id = 14887
OR person_id = 16371
```

Clues gathered:

Suspect is male
Had a Get Fit Now Gym bag
Membership ID starts with 48Z
Is a gold member
Car plate contains H42W
Was at the gym on January 9, 2018
---

Step 4 — Find the Suspect Through Gym Records (JOIN)
```sql
SELECT person.id, person.name, 
       get_fit_now_member.id, 
       get_fit_now_member.membership_status
FROM get_fit_now_member
JOIN person ON get_fit_now_member.person_id = person.id
JOIN get_fit_now_check_in ON get_fit_now_member.id = get_fit_now_check_in.membership_id
WHERE get_fit_now_member.id LIKE '48Z%'
AND get_fit_now_member.membership_status = 'gold'
AND get_fit_now_check_in.check_in_date = 20180109
```
Finding: Two suspects — Joe Germuska (48Z7A) and Jeremy Bowers (48Z55)
---

Step 5 — Confirm the Murderer Using Car Plate
```sql
SELECT person.id, person.name, drivers_license.plate_number
FROM person
JOIN drivers_license ON person.license_id = drivers_license.id
WHERE drivers_license.plate_number LIKE '%H42W%'
AND person.id IN (28819, 67318)
```

MURDERER IDENTIFIED: Jeremy Bowers

Plate number: 0H42W2 ✓
Gold gym member: 48Z55 ✓
At gym on January 9th ✓
---

Step 6 — Uncover the Mastermind (The Twist)

Jeremy Bowers' interview revealed he was hired by someone. Clues about the mastermind:
Female
Height: 5'5" (65") or 5'7" (67")
Red hair
Drives a Tesla Model S
Attended SQL Symphony Concert 3 times in December 2017
```sql
SELECT person.id, person.name, 
       drivers_license.height,
       drivers_license.hair_color,
       drivers_license.car_make,
       drivers_license.car_model,
       COUNT(facebook_event_checkin.event_id) AS concert_attendance
FROM person
JOIN drivers_license ON person.license_id = drivers_license.id
JOIN facebook_event_checkin ON person.id = facebook_event_checkin.person_id
WHERE drivers_license.gender = 'female'
AND drivers_license.height BETWEEN 65 AND 67
AND drivers_license.hair_color = 'red'
AND drivers_license.car_make = 'Tesla'
AND drivers_license.car_model = 'Model S'
AND facebook_event_checkin.event_name LIKE '%SQL Symphony%'
AND facebook_event_checkin.date LIKE '201712%'
GROUP BY person.id
HAVING COUNT(facebook_event_checkin.event_id) = 3
```

MASTERMIND IDENTIFIED: Miranda Priestly

Female ✓
Height: 66 inches ✓
Red hair ✓
Tesla Model S ✓
Attended SQL Symphony Concert exactly 3 times in December 2017 ✓
---

Key SQL Concepts Learned

Concept	Where it was used
`AND` / `OR`	Combining multiple filter conditions
`LIKE '%text%'`	Searching partial values in membership IDs and plate numbers
`ORDER BY DESC` + `LIMIT 1`	Finding the last house on Northwestern Dr
`JOIN`	Connecting person, gym, and license tables
`IN (val1, val2)`	Narrowing search to two specific suspects
`BETWEEN`	Filtering height range 65-67 inches
`COUNT` + `GROUP BY` + `HAVING`	Counting concert attendance per person
---

Personal Insight

One thing I discovered during this project: there are often multiple valid ways to write a SQL query that returns the same result. For example, when finding gym members who checked in on January 9th, the name can be retrieved from either the `person` table (via JOIN) or directly from the `get_fit_now_member` table. The better query depends on what information you actually need in the output.
This taught me that being a good analyst isn't just about getting the right answer — it's about understanding WHY your query returns what it does.
---

Files in This Repository

File	Description
`README.md`	This file — full walkthrough of the investigation
`queries.sql`	All SQL queries used in the investigation
---

About the Author

Bidemi Isewon — The CopyAnalyst
Data Analyst in Training | Copywriter | Building African data insights in public
🔗 LinkedIn
📊 Follow my journey: #DataAnalystDiaries
---
"The course gives you the map. Real data gives you the education."
