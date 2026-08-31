# Exercise 01: World Database SQL Practice

- Name: Josiah Davis
- Course: Database for Analytics
- Module: 1
- Database Used: World Database

---

See:

[MySQL: Setting Up the World Database](https://dev.mysql.com/doc/world-setup/en/)

---

## Instructions

- Answer each question below.
- All SQL commands **must be executed** against the World database.
- For each SQL command:
  - Include the SQL in a fenced code block
  - Include a **screenshot** showing the command and results
- Store screenshots in the `screenshots/` folder and embed them below each answer.

---

## Question 1

**Compare and contrast the data types used for:**

- `country.Population`
- `country.LifeExpectancy`

Why were these data types selected?

### Answer

Population is INT and LifeExpectancy is DECIMAL(3,1).

Population is a count, so a whole number makes sense. There's no such thing as half a person. INT goes up to about 2.1 billion which covers every country here, though China at 1.28 billion is closer to that ceiling than I expected.

LifeExpectancy is an average so it needs decimals. 78.4 is a real answer and rounding to 78 throws away information. DECIMAL(3,1) is three digits with one after the point, so 0.0 to 99.9. That's about as precise as the number actually is.

They used DECIMAL instead of FLOAT, which I think matters. DECIMAL is exact, FLOAT is an approximation. For a number people rank and compare, exact seems like the safer choice.

One other difference, Population is NOT NULL with a default of 0 but LifeExpectancy allows NULL. Every country has a population. Not every country has a known life expectancy.

### Screenshot

```sql
DESCRIBE country;
```

![Q1 Screenshot](screenshots/q1_datatypes.png)

---

## Question 2

**What is the data type of `country.IndepYear`?**
Why do you think this data type was selected?

### Answer

IndepYear is SMALLINT.

SMALLINT covers -32768 to 32767, so every year in the table fits with room left over, and it only takes 2 bytes where INT would take 4. Since it's just a year and not a full date, keeping it as a number keeps it small and it still sorts and compares fine.

The other thing is that it allows NULL. A lot of these rows have no independence year at all, either because the place was never independent or it's a territory of somewhere else. Population and SurfaceArea are both NOT NULL with defaults, but IndepYear is nullable on purpose. That's the schema saying the value doesn't exist instead of sticking a zero in there.

### Screenshot

```sql
DESCRIBE country;
```

![Q2 Screenshot](screenshots/q2_indepyear.png)

---

## Question 3

**Make a case for a different data type for `country.IndepYear`.**
Explain why your proposed data type might be better in some situations.

### Answer

DATE would be better if you cared about the actual day. Right now the table can tell you a country became independent in 1776 but not that it was July 4th. If you wanted exact elapsed time, or to sort by anniversary, or to join to some other date table, SMALLINT can't do that and DATE can.

The problem is DATE forces you to supply a month and day even when you don't have one. You'd end up making values up or defaulting everything to 1776-01-01, which is worse than just admitting you only have the year.

CHAR(4) is another option if the year is really a label and not a number, but that gives up numeric sorting so I wouldn't.

For this table SMALLINT seems right. It matches how precise the data actually is. DATE would win in a system where full dates were known and being used in calculations.

---

## Question 4

Write a SQL command to **list the names of all cities in alphabetical order**.

### SQL

```sql
SELECT Name
FROM city
ORDER BY Name;
```

### Screenshot

![Q4 Screenshot](screenshots/q4_cities_sorted.png)

Worth noting the output says 1000 rows returned but the city table has 4079. That's Workbench applying a default 1000 row limit, not the query. There's a dropdown in the toolbar that controls it.

---

## Question 5

Write a SQL command to
**list all forms of government from the `country` table**,
showing **each only once**, sorted alphabetically.

### SQL

```sql
SELECT DISTINCT GovernmentForm
FROM country
ORDER BY GovernmentForm;
```

### Screenshot

![Q5 Screenshot](screenshots/q5_government_forms.png)

35 distinct government forms across 239 countries.

---

## Question 6

Write a SQL command to **list all countries in the `Oceania` continent**.

### SQL

```sql
SELECT Name
FROM country
WHERE Continent = 'Oceania';
```

### Screenshot

![Q6 Screenshot](screenshots/q6_oceania.png)

28 rows. Continent is an ENUM and not a regular char column, so the value has to match one of the defined options exactly.

---

## Question 7

Write a SQL command to **list the names and country code of all cities**.

### SQL

```sql
SELECT Name, CountryCode
FROM city;
```

### Screenshot

![Q7 Screenshot](screenshots/q7_city_countrycode.png)

---

## Question 8

Write a SQL command to **update the city named `"Nashville-Davidson"` to `"Nashville"`**.

### SQL

```sql
UPDATE city
SET Name = 'Nashville'
WHERE Name = 'Nashville-Davidson';
```

### Screenshot

![Q8 Screenshot](screenshots/q8_update_city.png)

This failed the first time with Error 1175. Safe update mode blocks any UPDATE where the WHERE clause doesn't use a key column. Mine filters on Name, which isn't a key on the city table. ID is.

Fix was turning safe update mode off for the session:

```sql
SET SQL_SAFE_UPDATES = 0;
```

After that it ran and reported 1 row affected, rows matched 1, changed 1. One thing I found out the hard way is that safe update mode resets every time you reconnect, so it came back in a later session and I had to turn it off again.

Safe update mode is doing something useful though. An UPDATE with a non-key WHERE clause is exactly the kind of thing that quietly changes 4000 rows instead of one. Turning it off is fine on a practice database. On anything real I'd rather write the statement against the key.

---

## Question 9

Write a SQL command to **insert a new country named `"Narnia"`**
with a country code of `"NAR"`.
Use reasonable values for the remaining columns.

### SQL

```sql
INSERT INTO country (Code, Name, Continent, Region, Population)
VALUES ('NAR', 'Narnia', 'Europe', 'Fantasy', 1000000);
```

### Screenshot

![Q9 Screenshot](screenshots/q9_insert_narnia.png)

I expected this to fail since the country table has several NOT NULL columns that aren't in the insert list. It worked because all of those columns have a DEFAULT defined. SurfaceArea defaults to 0.00 and LocalName, GovernmentForm and Code2 all default to an empty string. NOT NULL with a default behaves pretty differently from NOT NULL without one.

Continent had to be 'Europe' or another valid ENUM value. Region is just char(26) so 'Fantasy' was fine.

---

## Question 10

Write a SQL command to **delete the country with the country code `"NAR"`**.

### SQL

```sql
DELETE FROM country
WHERE Code = 'NAR';
```

### Screenshot

![Q10 Screenshot](screenshots/q10_delete_narnia.png)

1 row affected. This one didn't trip safe update mode because Code is the primary key on country, which is exactly the difference that caused Error 1175 back in Question 8.
