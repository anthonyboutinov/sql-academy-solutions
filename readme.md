The following are my solutions to "Online Trainer" exercises at <a href="https://sql-academy.org/en/">SQL Academy</a>

The code below is only for reference for people who might be struggling with any of the exercises from that course. Additionally, be aware that the naming of fields and tables in the database of SQL-Academy can be inconsistent and/or have mistakes.

There is also a repository with solutions to a few other exercises from that course here: <a href="https://github.com/Lexir/sql-academy-solution">https://github.com/Lexir/sql-academy-solution</a>.

## Exercise 35

How many different classrooms of the school were used on 2.09.2019 for educational purposes?

```SQL
SELECT
    COUNT(classroom) AS count
FROM Schedule
WHERE
    DATE_FORMAT(DATE, "%d.%m.%Y") = "02.09.2019"
```

## Exercise 36

Print information about students living on Pushkin street (ul. Pushkina)?

```SQL
SELECT
    *
FROM
    Student
WHERE
    address LIKE "%ul. Pushkina%"
```

## Exercise 37

How old is the youngest student?
Fields in the resulting table: year
Use the "as year" construction to specify the student's age. This is necessary for correct verification.

```SQL
SELECT
    MIN(
        TIMESTAMPDIFF(YEAR, birthday, CURDATE())) AS YEAR
    FROM
        Student
```

## Exercise 38

How many Annas (Anna) go to school?
Fields in the resulting table: count

```SQL
SELECT
    COUNT(*) AS COUNT
FROM
    Student
WHERE
    first_name = "Anna"
```

## Exercise 39

How many students are in grade 10 B?
Fields in the resulting table: count

```SQL
SELECT
    COUNT(student) AS count
FROM
    Student_in_class
JOIN Class ON Class.id = Student_in_class.class
WHERE
    Class.name = "10 B"
```

## Exercise 42

How long will the student be in school, studying from the 2nd to the 4th academic subject ?
Fields in the resulting table:
time

```SQL
SELECT TIMEDIFF(
(SELECT end_pair FROM Timepair WHERE id = 4),
(SELECT start_pair FROM Timepair WHERE id = 2)
) AS time
```

## Exercise 43

Output the names of the teachers who engaged in physical culture. Sort the teachers by their last name.
Fields in the resulting table: last_name

```SQL
SELECT
    last_name
FROM
    Teacher
JOIN Schedule ON
    Schedule.teacher = Teacher.id
JOIN Subject ON
    Subject.id = SCHEDULE.subject
WHERE
    Subject.name = "Physical Culture"
ORDER BY
    last_name
```

## Exercise 44

Find the maximum age (number years) among students of 10 classes ?
Fields in the resulting table:
max_year

```SQL
SELECT
    MAX(
        TIMESTAMPDIFF(YEAR, birthday, CURDATE())) AS max_year
    FROM
        Student,
        Student_in_class,
        Class
    WHERE
        Class.id = Student_in_class.class
        AND Student_in_class.student = Student.id
        AND Class.name LIKE "%10%"
```

## Exercise 45

Which classroom(s) are in the highest demand?
Fields in the resulting table:
classroom

```SQL
SELECT
    classroom
FROM Schedule
GROUP BY
    classroom
HAVING
    COUNT(*) =(
        SELECT
            COUNT(*)
        FROM Schedule
        GROUP BY
            classroom
        ORDER BY
            COUNT(*)
        DESC
        LIMIT 1
    )
```

## Exercise 46

In which classes will the Krauze teacher introduce classes ?
Fields in the resulting table:
name

```SQL
SELECT DISTINCT
    Class.name
FROM
    Class
JOIN Schedule ON Schedule.class = Class.id
JOIN Teacher ON Teacher.id = Schedule.teacher
WHERE
    Teacher.last_name = "Krauze"
```

## Exercise 47

How many classes did Krauze hold on August 30, 2019?
Fields in the resulting table:
count

```SQL
SELECT
    COUNT(*) AS count
FROM Schedule
JOIN Teacher ON Teacher.id = teacher
WHERE
    last_name = "Krauze" AND DAY(date) = 30 AND MONTH(date) = 8 AND YEAR(date) = 2019
```

## Exercise 48

Print the number of students in classes in descending order
Fields in the resulting table:
name, count

```SQL
SELECT name,
    COUNT(Student_in_class.id) AS count
FROM
    Class
JOIN Student_in_class ON class = Class.id
GROUP BY
    class.id
ORDER BY
    count
DESC

```

## Exercise 50

What percentage of students were born in 2000? The result of rounding to the nearest whole in the smaller side.
Fields in the resulting table:
percent

```SQL
SELECT
    FLOOR(
        (
            SELECT
                COUNT(*)
            FROM
                Student
            WHERE
                YEAR(birthday) = 2000
        )
        /
        (
            SELECT
                COUNT(*)
            FROM
                Student
        ) * 100
    ) AS percent
```

## Exercise 55

Delete the companies that made the least number of flights.

```SQL
DELETE
FROM
    Company
WHERE
    id IN (
        SELECT
            company
        FROM
            Trip
        GROUP BY
            company
        HAVING
            COUNT(*) = (
                SELECT
                    COUNT(*) AS count
                FROM
                    Trip
                GROUP BY
                    company
                ORDER BY
                    count ASC
                LIMIT 1
            )
    )
```

## Exercise 58

Add a 5-star comment on housing located at 11218 Friel Place, New York on behalf of George Clooney
As the primary key (id), specify the number of entries in the table + 1.
The reservation of the room for which you need to leave a review has already been made, you just need to find it.

- Note: In this Exercise, primary key does not have Auto Increment, and the task clearly specifies to create a new value by counting the number of rows in the table. MySQL doesn't allow updating the table you are already using in an inner select as the update criteria. Many other database engines has support for this feature, but MySQL doesn't and you need to workaround the limitation. You can read about it here: https://www.inforbiro.com/blog/mysql-cant-specify-target-table-for-update-in-from-clause

```SQL
INSERT INTO Reviews(id, reservation_id, rating)
VALUES(
    (
    SELECT
        Rcount.count
    FROM
        (
        SELECT
            COUNT(*) + 1 AS COUNT
        FROM
            Reviews
    ) AS Rcount
),
(
    SELECT
        Reservations.id
    FROM
        Reservations
    JOIN Users ON Reservations.user_id = Users.id
    JOIN Rooms ON Reservations.room_id = Rooms.id
    WHERE
        Users.name = "George Clooney" AND Rooms.address = "11218, Friel Place, New York"
),
5
)
```

## Exercise 64

Display the number of reservations for each month of each year that had at least 1 reservation. Sort the result in ascending order of reservation date.
Fields in the resulting table:
year, month, amount

```SQL
SELECT
    YEAR(start_date) AS year,
    MONTH(start_date) AS month,
    COUNT(id) AS amount
FROM
    Reservations
GROUP BY
    year,
    month
ORDER BY
    year,
    month
```

## Exercise 66

Display a list of rooms with all amenities (TV, Internet, kitchen, and air conditioning), as well as the total number of days and the amount for all days of renting each of these rooms.
Fields in the resulting table:
home_type, address, days, total_fee

```SQL
SELECT
    home_type,
    address,
    IFNULL(
        SUM(DATEDIFF(end_date, start_date)),
        0
    ) AS days,
    IFNULL(SUM(total),
    0) AS total_fee
FROM
    Rooms
LEFT JOIN Reservations ON Rooms.id = Reservations.room_id
WHERE
    has_tv = TRUE AND has_internet = TRUE AND has_kitchen = TRUE AND has_air_con = TRUE
GROUP BY
    home_type,
    address
```

## Exercise 68

For each room selected at least 1 time, find the name of the person who last rented it and used it when they checked out
Fields in the resulting table:
room_id, name, end_date

```SQL
SELECT
    Reservations.room_id AS room_id,
    NAME,
    end_date
FROM
    Reservations
JOIN Users ON Reservations.user_id = Users.id
JOIN(
    SELECT room_id,
        MAX(end_date) AS max_end_date
    FROM
        Reservations
    GROUP BY
        room_id
) mx
ON
    mx.room_id = Reservations.room_id
WHERE
    end_date = mx.max_end_date
```

## Exercise 70

It is necessary to categorize rooms into economy, comfort, premium at a price <= 100, 100 < price < 200, >= 200, respectively. As a result, display a table with the name of the category and the number of housing falling into this category
Fields in the resulting table:
category, count

```SQL
(
    SELECT
        "economy" AS category,
        COUNT(*) AS count
    FROM
        Rooms
    WHERE
        price <= 100
)
UNION (
    SELECT
        "comfort" AS category,
        COUNT(*) AS count
    FROM
        Rooms
    WHERE
        price > 100 AND price < 200
)
UNION (
    SELECT
        "premium" AS category,
        COUNT(*) AS count
    FROM
        Rooms
    WHERE
        price >= 200
)
```
