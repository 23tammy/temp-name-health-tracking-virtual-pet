# Health Tracking Virtual Pet

## 1. Problem Definition and Mobile Scope

### Problem Definition

Many fitness and wellness applications focus heavily on numbers, statistics, and strict goal tracking. This can make maintaining healthy habits feel more like a chore than an engaging activity. Users may also keep different aspects of their well-being, such as fitness, journaling, motivation, and mood tracking, in separate applications.

This project addresses this problem by combining personal wellness tracking with a virtual-pet-style game experience. The user maintains a customizable virtual pet whose well-being reflects the user's wellness activities. Users can create fitness plans, track exercises using sets, reps, and weight, record daily mood and hobbies, write journal entries, and save inspirational quotes. Completing wellness activities provides positive feedback and affects the virtual character's happiness.

The application is not intended to diagnose, treat, or medically evaluate users. Instead, it is designed as a simple motivational self-tracking and habit-building tool. The application intentionally avoids unnecessary features that could encourage excessive use or turn wellness into a competitive experience.

There will be no social networking or multiplayer functionality. Users will improve their pet's well-being through their own activities rather than through purchasing items. The application will not require users to spend money to progress.

### Mobile Platform Scope

The project will be developed as a cross-platform mobile application for Android and iOS using React Native. The semester scope will focus on the core individual wellness tracking and virtual-pet experience.

### Features Within Scope

#### Virtual Character

* Display a customizable virtual character.
* Change the character's status based on the user's logged activities.
* Provide visual feedback when the user completes wellness activities.
* Allow the pet to become happier or more bored depending on the user's activity.

#### Fitness Tracking

* Create and view personal fitness plans.
* Add exercises to fitness plans.
* Log individual sets, repetitions, and weight.
* View previously recorded workout data.
* Mark fitness activities or goals as completed.

#### Wellness Tracking

* Record daily mood.
* Record hobby-related activities.
* Display an overall wellness status.
* Increase pet happiness when the user participates in hobbies and wellness activities.

#### Journal

* Create personal journal entries.
* View previously saved journal entries.

#### Inspirational Quotes

* Save motivational or inspirational quotes.
* View previously saved quotes as part of a personal collection.

### Features Outside the Semester Scope

To keep the project manageable, the initial version will not include:

* Social networking or following other users
* Multiplayer functionality
* Real-time messaging
* Medical diagnosis or medical recommendations
* Apple Health or Google Fit integration
* Wearable device integration
* Professional healthcare-provider functionality
* Complex AI-generated health advice
* Competitive leaderboards
* In-app purchases or paid progression

These features could be considered for future versions, but they are not necessary to demonstrate the application's core concept.

---

## 2. Initial Database Design and Mechanics

The database will use PostgreSQL and will store the information necessary for the application's core wellness and game features. The database will focus on fitness tracking, mood, hobbies, journaling, inspirational quotes, and the virtual character rather than attempting to track every possible health metric.

### Database Relationships

* **Users → FitnessPlans:** A user can have multiple fitness plans.
* **FitnessPlans → Workouts:** A fitness plan can contain multiple exercises.
* **Workouts → WorkoutSets:** An exercise can contain multiple sets, with each set storing its own repetitions and weight.
* **Users → HealthLogs:** A user can have multiple daily health logs.
* **Users → JournalEntries:** A user can have multiple journal entries.
* **Users → SavedQuotes:** A user can save multiple inspirational quotes.

The fitness relationship is intentionally separated into multiple tables. A fitness plan contains exercises, while each exercise can have multiple sets with different repetitions and weights.


### SQL Schema

```sql
CREATE TABLE Users (
    user_id UUID PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    pet_name VARCHAR(50),
    pet_happiness INT DEFAULT 50,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE FitnessPlans (
    plan_id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    plan_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id)
        REFERENCES Users(user_id)
);

CREATE TABLE Workouts (
    workout_id SERIAL PRIMARY KEY,
    plan_id INT NOT NULL,
    exercise_name VARCHAR(100) NOT NULL,

    FOREIGN KEY (plan_id)
        REFERENCES FitnessPlans(plan_id)
);

CREATE TABLE WorkoutSets (
    set_id SERIAL PRIMARY KEY,
    workout_id INT NOT NULL,
    set_number INT NOT NULL,
    reps INT NOT NULL,
    weight DECIMAL(6,2) NOT NULL,

    FOREIGN KEY (workout_id)
        REFERENCES Workouts(workout_id)
);

CREATE TABLE HealthLogs (
    log_id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    mood INT NOT NULL,
    hobby VARCHAR(100),
    hobby_completed BOOLEAN DEFAULT FALSE,
    log_date DATE NOT NULL,

    FOREIGN KEY (user_id)
        REFERENCES Users(user_id)
);

CREATE TABLE JournalEntries (
    entry_id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    entry_text TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (user_id)
        REFERENCES Users(user_id)
);

CREATE TABLE SavedQuotes (
    quote_id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    quote_text TEXT NOT NULL,
    author VARCHAR(100),

    FOREIGN KEY (user_id)
        REFERENCES Users(user_id)
);
```

---

## 3. SQL Queries

Here are five that are a little more substantial while still being appropriate for your database design:

### 3. Relational Algebra Queries

**Retrieve a user's fitness plans and their exercises**

```text
π plan_name, exercise_name(σ user_id = 'USER_UUID'(FitnessPlans ⨝ FitnessPlans.plan_id = Workouts.plan_id Workouts))
```

---

**Retrieve exercises with their sets, reps, and weight**

```text
π exercise_name, set_number, reps, weight(σ plan_id = 1(Workouts ⨝ Workouts.workout_id = WorkoutSets.workout_id WorkoutSets))
```

---

**Retrieve a user's completed hobbies**

```text
π log_date, mood, hobby(σ user_id = 'USER_UUID' ∧ hobby_completed = TRUE(HealthLogs))
```


---

**Retrieve a user's journal entries and saved quotes**

```text
π entry_text, created_at(σ user_id = 'USER_UUID'(JournalEntries))
```

and

```text
π quote_text, author(σ user_id = 'USER_UUID'(SavedQuotes))
```


---

**Retrieve a user's fitness data across all related tables**

```text
π plan_name, exercise_name, set_number, reps, weight(σ user_id = 'USER_UUID'(FitnessPlans ⨝ FitnessPlans.plan_id = Workouts.plan_id Workouts ⨝ Workouts.workout_id = WorkoutSets.workout_id WorkoutSets))
```



## 4. AI Utilization Plan

GitHub Copilot will primarily be used as a programming assistant and learning tool throughout development. I will maintain a log of the prompts used and document how Copilot helps with debugging, understanding existing code, explaining database concepts, and improving implementations.

Rather than relying on Copilot to build the application independently, I will use it to help me understand problems and make informed development decisions. Prompts will generally provide Copilot with existing code, database schemas, queries, or error messages and ask it to explain the underlying concepts or identify potential issues.

### Example Prompts

* "I am encountering an issue where my React Native application successfully submits a workout, but the newly saved workout does not immediately appear on the screen. Here is the relevant component and Supabase query: [Snippet]. Can you explain what could cause the UI and database to become out of sync?"

* "I am trying to understand how my React Native application communicates with Supabase. Given my current component and database structure: [Snippet], can you explain the flow of information from the user entering their workout, to PostgreSQL storing the data, and finally back to the application when the workout history is displayed?"

