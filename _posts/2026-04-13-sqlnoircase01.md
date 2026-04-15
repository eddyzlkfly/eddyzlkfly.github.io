---
title: "SQL Noir – Case #001 (Easy)"
date: 2026-04-13
categories: [Writeups, SQLNoir]
tags: [sql, ctf, investigation, database]
description: Investigation of the Blue Note Lounge briefcase theft using SQL queries.
image: /assets/img/sqlnoir-cover.png
---

# Case #001: The Vanishing Briefcase

Set in the gritty 1980s, a valuable briefcase has disappeared from the Blue Note Lounge. A witness reported that a man in a trench coat was seen fleeing the scene. Investigate the crime scene, review the list of suspects, and examine interview transcripts to reveal the culprit.

# Objectives

1. Retrieve the correct crime scene details to gather the key clue.
2. Identify the suspect whose profile matches the witness description.
3. Verify the suspect using their interview transcript.

# Investigation Steps
## 1) Identify suspects matching the witness description

Since the witness mentioned seeing a man wearing a trench coat, the first thing to do is filter the suspects based on their attire.

To do this, we run the following SQL query to retrieve suspects who were wearing a trench coat.
```sql
SELECT * FROM suspects WHERE attire = "trench coat"
```
The query returns three suspects who match the description. These are the individuals who were wearing a trench coat, so they become our primary suspects for now.

| id | name | attire | scar |
|----------|----------|----------|----------|
| 3   | Frankie Lombardi   | trench coat   | left cheek   |
| 183   | Vincent Malone   | trench coat   | left cheek   |
| 237   | Christopher Black   | trench coat   | right cheek   |

From the results, the suspects are:

- **Frankie Lombardi**
- **Vincent Malone**
- **Christopher Black**

## 2) Retrieve crime scene details

Since the case description already tells us that the incident happened at Blue Note Lounge, we can query the `crime_scene` table using that `location` to see what information we can gather about the incident.
```sql
SELECT * FROM crime_scene WHERE location = "Blue Note Lounge"
```

| id | date     | type  | location          | description |
|----|----------|-------|------------------|-------------|
| 76 | 19851120 | theft | Blue Note Lounge | A briefcase containing sensitive documents vanished. A witness reported a man in a trench coat with a scar on his left cheek fleeing the scene. |

From the result, we learn that a briefcase containing sensitive documents was stolen. More importantly, the description mentions that the witness saw a man wearing a trench coat with a scar on his left cheek fleeing the scene. This clue helps us narrow down our suspects even further.

From Step 1, we had three suspects wearing trench coats:

- Frankie Lombardi – scar on left cheek
- Vincent Malone – scar on left cheek
- Christopher Black – scar on right cheek

Since the witness specifically mentioned a scar on the **left cheek**, we can eliminate **Christopher Black** from our suspect list.

Now we are left with two possible suspects:

- **Frankie Lombardi**
- **Vincent Malone**

In the next step, we will check their interview transcripts to determine which one is responsible for the theft.

## 3) Verify the suspect using interview transcripts

Since we have already narrowed down our suspects to **Frankie Lombardi (ID 3)** and **Vincent Malone (ID 183)**, the next step is to check their interview transcripts.

To do this, we query the interviews table using their `suspect_id` values.
```sql
SELECT * FROM interviews WHERE suspect_id IN (3, 183)
```

| suspect_id | transcript |
|----------|----------|
| 3   | NULL   |
| 183   | I wasn’t going to steal it, but I did.   |

From the results, we can see that `suspect_id = 3` has no transcript recorded `(NULL)`. However, `suspect_id = 183` has a statement that says:

*"I wasn't going to steal it, but I did."*

This clearly confirms that **Vincent Malone** is the person responsible for stealing the briefcase.

## Answer
**Vincent Malone**