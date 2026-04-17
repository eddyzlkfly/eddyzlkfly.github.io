---
title: "SQL Noir – Case #002 (Easy)"
date: 2026-04-13
categories: [Writeups, Detective, Database, Forensics]
tags: [sql, forensics, investigation, database]
description: Investigation of a stolen vinyl record case at West Hollywood Records using SQL queries.
image: /assets/img/sqlnoir-cover.png
---
# Case #002: The Stolen Sound

In the neon glow of 1980s Los Angeles, the West Hollywood Records store was rocked by a daring theft. A prized vinyl record, worth over $10,000, vanished during a busy evening, leaving the store owner desperate for answers. Vaguely recalling the details, you know the incident occurred on July 15, 1983, at this famous store. Your task is to track down the thief and bring them to justice.

# Objectives
1. Retrieve the crime scene report for the record theft using the known date and location.
2. Retrieve witness records linked to that crime scene to obtain their clues.
3. Use the clues from the witnesses to find the suspect in the suspects table.
4. Retrieve the suspect's interview transcript to confirm the confession.

## Investigation Steps
## 1) Retrieve crime scene details

Since the case description already tells us that the theft happened on `July 15, 1983`, we can query the `crime_scene` table using that date to retrieve the crime scene report.
```sql
SELECT * FROM crime_scene WHERE date = 19830715;
```

| id | date | type | location | description |
|----------|----------|----------|----------|----------|
| 65   | 19830715   | theft   | 	West Hollywood Records   | A prized vinyl record was stolen from the store during a busy evening  |

From the result, we confirm that a prized vinyl record was stolen from **West Hollywood Records** during a busy evening. Now that we have the `crime_scene_id (65)`, we can use it to retrieve witness statements related to this incident in the next step.

## 2) Check witness statements

Now that we know the `crime_scene_id` is **65**, we can query the witnesses table to see what clues the witnesses reported during the incident.
```sql
SELECT * FROM witnesses WHERE crime_scene_id = 65;
```

| id | crime_scene_id | clue |
|----------|----------|----------|
| 28   | 65   | I saw a man wearing a red bandana rushing out of the store   |
| 75   | 65   | The main thing I remember is that he had a distinctive gold watch on his wrist   |

From the witness statements, we get two important clues:

- The suspect was seen wearing a **red bandana**.
- The suspect had a distinctive **gold watch** on his wrist.

These clues give us specific characteristics that we can use to filter the suspects in the database.

## 3) Identify suspects matching the clues

Using the clues from the witnesses, we can now query the suspects table to find individuals who match the description — specifically suspects who wear a **red bandana** and have a **gold watch**.
```sql
SELECT * FROM suspects WHERE bandana_color = "red" AND accessory = "gold watch";
```

| id | name | bandana_color | accessory |
|----------|----------|----------|----------|
| 35   | Tony Ramirez  | red   | gold watch   |
| 44   | Mickey Rivera   | red   | gold watch   |
| 97   | Rico Delgado   | red   | gold watch   |

The query returns three possible suspects:

- **Tony Ramirez**
- **Mickey Rivera**
- **Rico Delgado**

Since all three match the witness description, we need additional information to determine who actually committed the theft. The next step is to check their interview transcripts.

## 4) Verify using interview transcripts

Since our suspect list has been narrowed down to three individuals, we can check their interview transcripts using their `suspect_id` values.
```sql
SELECT * FROM interviews WHERE suspect_id IN (35, 44, 97);
```

| suspect_id | transcript |
|----------|----------|
| 35   | I wasn't anywhere near West Hollywood Records that night   |
| 44   | I was busy with my music career; I have nothing to do with this theft   |
| 97   | I couldn't help it. I snapped and took the record   |

From the results:

- Tony Ramirez denies being near the store that night.
- Mickey Rivera claims he was busy with his music career.
- Rico Delgado admits:
*"I couldn't help it. I snapped and took the record."*

This confession clearly identifies **Rico Delgado** as the person responsible for stealing the vinyl record.

## Answer
**Rico Delgado**