Check-In 2:
The initial schema has changed slightly as I flesh out the details of how I would like my health tracker to perform; its main changes being how fitness is logged and categorized. I have also renamed some tables and attributes to more accurately describe what they are representing. I would also omit the customizable feature for the pet, as well as the Health Logs table, for the sake of simplicity. 

This is the schema satisfying 1NF:
## PROGRAMS

| program_id (PK) | program_name | day (PK) | split_type | exercise (PK) | muscle_group | equipment |
|---|---|---|---|---|---|---|
| 1 | My Workout Program | monday | push | bench press | chest | barbell |
| 1 | My Workout Program | monday | push | shoulder press | shoulders | machine |
| 1 | My Workout Program | tuesday | pull | bicep curl | biceps | machine |

## PROGRESSION

| log_id (PK) | program_id | date | day | exercise | sets | reps | weight_lbs |
|---|---|---|---|---|---|---|---|
| 1 | 1 | 2026-10-01 | monday | bench press | 3 | 8 | 45 |
| 2 | 1 | 2026-10-01 | monday | shoulder press | 3 | 8 | 45 |
| 3 | 1 | 2026-10-01 | tuesday | bicep curl | 3 | 8 | 45 |

## JOURNAL

| entry_id (PK) | date | text |
|---|---|---|
| 1 | 2026-10-01 | Today was so great I did blah blah blah. My coworker was SO rude to me, I had to remind myself I am NOT being paid enough to be angry. I decided to treat myself to two coffees from Buckets- just because I wanted to try out the different flavors, but I forgot I don’t exactly do well with caffeine, so then I spent 5 hours deep cleaning my bedroom till 3 am. It be spotless though. |

## QUOTES

| quote_id (PK) | date | text | author | origin |
|---|---|---|---|---|
| 1 | 2026-10-01 | Look again at that dot. That's here. That's home. That's us. On it everyone you love, everyone you know, everyone you ever heard of, every human being who ever was, lived out their lives. The aggregate of our joy and suffering, thousands of confident religions, ideologies, and economic doctrines, every hunter and forager, every hero and coward, every creator and destroyer of civilization, every king and peasant, every young couple in love, every mother and father, hopeful child, inventor and explorer, every teacher of morals, every corrupt politician, every "superstar," every "supreme leader," every saint and sinner in the history of our species lived there--on a mote of dust suspended in a sunbeam. | Carl Sagan | Pale Blue Dot: A Vision of the Human Future in Space |


I have decided to remove the user_id attribute from Journal and Quotes, since the app is intended for one user and is NOT a social platform. Since Journal and Quotes store independent text entries, and each attribute depends on nothing but the key, these satisfy 3NF. PROGRAM and PROGRESSION tables need to be adjusted, since there are recurring attributes between the tables.

If a user wanted to add a new exercise, like a Leg Press, to an exercise library without assigning it to a workout day, you could not insert it without also providing a program ID, day, and split type. This creates an insertion anomaly because exercise information was dependent on the existence of a program entry. Creating a separate Exercises table allows new exercises to be added independently of workout schedules. 
If a user removed the only scheduled instance of Bicep Curl from the weekly program, and if the exercise information existed only in the Programs table, then deleting that row would also remove the only record containing its muscle group and other exercise information. This could result in accidentally losing useful exercise performance data.
By separating Exercises from Program preserves the exercise information even when it is removed from a workout schedule.
In the original Program table, exercise information was repeated whenever an exercise appeared in a program.
For example, if Bench Press appeared on multiple days, its muscle group would be recorded multiple times. If I wanted to change its primary muscle group, I would have to update every occurrence.
If one occurrence was missed, the database could contain inconsistent information. Storing exercise information once in Exercises ensures that changes only need to be made in one location.
The functional dependencies: 
(program_id, day) determines split_type. A program's workout day determines its split type, and not the exercise. This is a partial dependency. 
exercise determines muscle_group. Only exercise determines its primary muscle group, so this is a partial dependency. 
(program_id, day, exercise) determines equipment	The scheduled exercise determines the selected equipment within that program day. The equipment cannot soley depend on the exercise because exercises may have alternatives using different equipment,

Finally, program_name is only dependant on program_id. 

PROGRAMS decomposed into 2NF, which also satisfies 3NF:
### PROGRAMS

| program_id (PK) | program_name |
|---|---|
| 1 | My Workout Program |

### SPLITS

| program_id (PK, FK) | day (PK) | split_type |
|---|---|---|
| 1 | monday | push |
| 1 | tuesday | pull |

### EXERCISES

| exercise_id (PK) | exercise_name | muscle_group |
|---|---|---|
| 1 | bench press | chest |
| 2 | shoulder press | shoulders |
| 3 | bicep curl | biceps |

### WORKOUTS

| program_id (PK, FK) | day (PK, FK) | exercise_id (PK, FK) | equipment |
|---|---|---|---|
| 1 | monday | 1 | barbell |
| 1 | monday | 2 | machine |
| 1 | tuesday | 3 | machine |

Progression already satisfies 2NF. Although there could be a potential transitive dependency where the date determines the day. Progression_day is redundant since the date of the log is already recorded. Day could be removed entirely to satisfy 3NF. The most difficult part of the resolution process was determining which attributes constituted a primary/composite key and which attributes depended on which. I thought about having the PROGRESSION table reference the composite key (program_id, day, exercise_id) in WORKOUTS as a foreign key to ensure every progression log corresponds to an exercise actually assigned to that workout day. But realistically, people deviate from their splits all the time for whatever reason, and those workouts should still be able to be logged independently of whatever workout was supposed to be performed.

## Final 3NF Schema

### PROGRAM

| program_id (PK) | program_name |
|---|---|
| 1 | My Workout Program |

### SPLITS

| program_id (PK, FK) | day (PK) | split_type |
|---|---|---|
| 1 | monday | push |
| 1 | tuesday | pull |

### EXERCISES

| exercise_id (PK) | exercise_name | muscle_group |
|---|---|---|
| 1 | bench press | chest |
| 2 | shoulder press | shoulders |
| 3 | bicep curl | biceps |

### WORKOUTS

| program_id (PK, FK) | day (PK, FK) | exercise_id (PK, FK) | equipment |
|---|---|---|---|
| 1 | monday | 1 | barbell |
| 1 | monday | 2 | machine |
| 1 | tuesday | 3 | machine |

### PROGRESSION

| log_id (PK) | program_id | day | exercise_id (FK) | date | sets | reps | weight_lbs |
|---|---|---|---|---|---|---|---|
| 1 | 1 | monday | 1 | 2026-10-01 | 3 | 8 | 45 |
| 2 | 1 | monday | 2 | 2026-10-01 | 3 | 8 | 45 |
| 3 | 1 | tuesday | 3 | 2026-10-01 | 3 | 8 | 45 |

### JOURNAL

| entry_id (PK) | date | text |
|---|---|---|
| 1 | 2026-10-01 | Today was so great I did blah blah blah. My coworker was SO rude to me, I had to remind myself I am NOT being paid enough to be angry. I decided to treat myself to two coffees from Buckets- just because I wanted to try out the different flavors, but I forgot I don’t exactly do well with caffeine, so then I spent 5 hours deep cleaning my bedroom till 3 am. It be spotless though. |

### QUOTES

| quote_id (PK) | date | text | author | origin |
|---|---|---|---|---|
| 1 | 2026-10-01 | Look again at that dot. That's here. That's home. That's us. On it everyone you love, everyone you know, everyone you ever heard of, every human being who ever was, lived out their lives. The aggregate of our joy and suffering, thousands of confident religions, ideologies, and economic doctrines, every hunter and forager, every hero and coward, every creator and destroyer of civilization, every king and peasant, every young couple in love, every mother and father, hopeful child, inventor and explorer, every teacher of morals, every corrupt politician, every "superstar," every "supreme leader," every saint and sinner in the history of our species lived there--on a mote of dust suspended in a sunbeam. | Carl Sagan | Pale Blue Dot: A Vision of the Human Future in Space |
