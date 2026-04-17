---
title: "SQL Noir – Case #003 (Intermediate)"
date: 2026-04-13
categories: [Writeups, Detective, Database, Forensics]
tags: [sql, forensics, investigation, database]
description: Investigation of a murder case at Coral Bay Marina using SQL queries to identify the suspect through multiple data sources.
image: /assets/img/sqlnoir-cover.png
---
# Case #003: The Miami Marina Murder

A body was found floating near the docks of Coral Bay Marina in the early hours of August 14, 1986. Your job, detective, is to find the murderer and bring them to justice. This case might require the use of JOINs, wildcard searches, and logical deduction. Get to work, detective.

# Objectives

1. Find the murderer.

# Investigation Steps
## 1) Retrieve crime scene details

Since the case description already provides us with the **date** and **location**, we can use this information to query the `crime_scene` table and retrieve details about the incident.

```sql
SELECT * FROM crime_scene WHERE date = "19860814" AND location = "Coral Bay Marina";
```

| id | date     | location         | description                                                                                                                                          |
| -- | -------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 43 | 19860814 | Coral Bay Marina | The body of an unidentified man was found near the docks. Two people were seen nearby: one who lives on 300ish "Ocean Drive" and another whose first name ends with "ul" and his last name ends with "ez".


From the result, we learn that a body was discovered at **Coral Bay Marina** on the specified date. More importantly, the description mentions that there were **two people** nearby, including one who lives around **"300ish Ocean Drive"**.

## 2) Identify suspects based on address and name pattern

From the previous step, we found that one of the individuals lives around **"300ish Ocean Drive"**.  
So, we query the `person` table to find anyone living on **Ocean Drive**.  

At the same time, we also try filtering based on a possible name pattern ending with **"ul"** and **"ez"** to narrow down suspects further.

```sql
SELECT * FROM person WHERE address LIKE "%Ocean Drive%" OR name LIKE "%ul %ez";
```

| id  | name            | alias       | occupation      | address         |
| --- | --------------- | ----------- | --------------- | --------------- |
| 1   | Marco Romano    | The Shadow  | Fisherman       | 22 Ocean Drive  |
| 5   | Michael Santos  | Silent Mike | Bartender       | 33 Ocean Drive  |
| 62  | Jesse Brooks    | The Judge   | Court Clerk     | 234 Ocean Drive |
| 101 | Carlos Mendez   | Los Ojos    | Fisherman       | 369 Ocean Drive |
| 102 | Raul Gutierrez  | The Cobra   | Nightclub Owner | 45 Sunset Ave   |
| 105 | Victor Martinez | Slick Vic   | Bartender       | 33 Ocean Drive  |

From the results, we get multiple individuals living on Ocean Drive.

However, based on the clue mentioning **"300ish Ocean Drive"**, we can narrow it down to addresses that are closer to that range:

- **234 Ocean Drive**
- **369 Ocean Drive**

This significantly reduces our suspect list and gives us a smaller group to investigate further in the next step.

## 3) Verify suspects using interview transcripts

From the previous step, we narrowed down our suspects to those living around **300ish Ocean Drive**, specifically:

- **Carlos Mendez (ID: 101)**
- **Raul Gutierrez (ID: 102)**

To gather more information, we check their interview transcripts to see if there are any useful clues.

```sql
SELECT * FROM interviews WHERE person_id IN (101, 102);
```

| id  | person_id | transcript                                                             |
| --- | --------- | ---------------------------------------------------------------------- |
| 101 | 101       | I saw someone check into a hotel on August 13. The guy looked nervous. |
| 103 | 102       | I heard someone checked into a hotel with "Sunset" in the name.        |

From the results, we get two important clues:

One suspect mentioned someone checking into a hotel on August 13
Another mentioned a hotel with "Sunset" in its name

This suggests that the suspect may have stayed at a hotel before or after the incident.
These clues will help us identify the exact location and narrow down the suspect further in the next step.

## 4) Identify hotel check-in records based on clues

From the previous step, we obtained two important clues:

- The suspect checked into a hotel on **August 13**
- The hotel name contains the word **"Sunset"**

Based on this information, we query the `hotel_checkins` table to find all matching records.

```sql
SELECT * FROM hotel_checkins 
WHERE check_in_date LIKE "%0813" 
AND hotel_name LIKE "%Sunset%";
```

| id  | person_id | hotel_name          | check_in_date |
| --- | --------- | ------------------- | ------------- |
| 2   | 27        | Sunset Bay Hotel    | 19860813      |
| 7   | 12        | Sunset Harbor Hotel | 19860813      |
| 10  | 15        | Sunset Palms Resort | 19860813      |
| 12  | 17        | Sunset Shore Hotel  | 19860813      |
| 14  | 19        | Sunset Marina Inn   | 19860813      |
| ... | ...       | ...                 | ...           |
| 99  | 10        | Sunset Cove Inn     | 19860813      |
| 100 | 54        | Sunset Crest Resort | 19860813      |

From the results, we can see that there are many individuals who checked into hotels with **"Sunset"** in the name on **August 13**.

However, not all of these are relevant to our investigation.
We need to correlate these records with our previous suspects:

- **Carlos Mendez (ID: 101)**
- **Raul Gutierrez (ID: 102)**

Since none of the person_id values in the results match **101** or **102**, this suggests that:

- Either the suspect used a different identity
- Or the suspect is not one of our initial suspects

This gives us a new direction for the investigation and indicates that we need to expand our search in the next step.

## 5) Filter suspects based on suspicious activity

From the previous step, we obtained a large list of individuals who checked into hotels with **"Sunset"** in the name on **August 13**.

To narrow down the suspects further, we look into the `surveillance_records` table to identify any **suspicious activities** associated with these individuals.

```sql
SELECT * FROM surveillance_records WHERE person_id BETWEEN 6 AND 55;
```

| id  | person_id | hotel_checkin_id | suspicious_activity                 |
| --- | --------- | ---------------- | ----------------------------------- |
| 6   | 6         | 34               | Spotted entering late at night      |
| 7   | 7         | 89               | Seen arguing with an unknown person |
| 8   | 8         | 2                | Left suddenly at 3 AM               |
| ... | ...       | ...              | ...                                 |

From the results, most individuals do not show any clearly suspicious behavior (many entries are NULL or normal activities).

However, a few stand out:

- Person ID 6 → Spotted entering late at night
- Person ID 7 → Seen arguing with an unknown person
- Person ID 8 → Left suddenly at 3 AM

These behaviors appear more suspicious compared to others.

So at this point, instead of analyzing everyone, we narrow down our focus to these three individuals based on instinct and relevance of their activities.

In the next step, we will check their confession records to identify the actual suspect.

## 6) Identify the culprit through confession

From the previous step, we narrowed down our suspects to three individuals:

- Person ID 6  
- Person ID 7  
- Person ID 8  

To determine who is responsible, we check their confession records one by one.

After testing each suspect individually, we find that only one of them has a clear confession.

```sql
SELECT * FROM confessions WHERE person_id = 8;
```

| id | person_id | confession                                                                 |
| -- | --------- | -------------------------------------------------------------------------- |
| 73 | 8         | Alright! I did it. I was paid to make sure he never left the marina alive. |

From the result, **Person ID 8** admits to the crime.

This confirms that he is the individual responsible for the murder.

# Answer
**Thomas Brown**