# User Login Analysis System
![](https://github.com/Ajay96700/SQL---User-Login-Analysis-System/blob/main/Images%20-%201.png)

# Overview
This project involves creating a relational database system to track user information and their login activity, followed by executing analytical queries to derive meaningful business insights.

```SQL
CREATE TABLE users (
    USER_ID INT PRIMARY KEY,
    USER_NAME VARCHAR(20) NOT NULL,
    USER_STATUS VARCHAR(20) NOT NULL
);
```

```SQL
CREATE TABLE logins (
    USER_ID INT,
    LOGIN_TIMESTAMP DATETIME NOT NULL,
    SESSION_ID INT PRIMARY KEY,
    SESSION_SCORE INT,
    FOREIGN KEY (USER_ID) REFERENCES USERS(USER_ID)
);
```

Inserted data in users table
```SQL
INSERT INTO USERS VALUES (1, 'Alice', 'Active');
INSERT INTO USERS VALUES (2, 'Bob', 'Inactive');
INSERT INTO USERS VALUES (3, 'Charlie', 'Active');
INSERT INTO USERS  VALUES (4, 'David', 'Active');
INSERT INTO USERS  VALUES (5, 'Eve', 'Inactive');
INSERT INTO USERS  VALUES (6, 'Frank', 'Active');
INSERT INTO USERS  VALUES (7, 'Grace', 'Inactive');
INSERT INTO USERS  VALUES (8, 'Heidi', 'Active');
INSERT INTO USERS VALUES (9, 'Ivan', 'Inactive');
INSERT INTO USERS VALUES (10, 'Judy', 'Active');
```

Inserted data in logins table
```SQL
INSERT INTO LOGINS  VALUES (1, '2023-07-15 09:30:00', 1001, 85);
INSERT INTO LOGINS VALUES (2, '2023-07-22 10:00:00', 1002, 90);
INSERT INTO LOGINS VALUES (3, '2023-08-10 11:15:00', 1003, 75);
INSERT INTO LOGINS VALUES (4, '2023-08-20 14:00:00', 1004, 88);
INSERT INTO LOGINS  VALUES (5, '2023-09-05 16:45:00', 1005, 82);
INSERT INTO LOGINS  VALUES (6, '2023-10-12 08:30:00', 1006, 77);
INSERT INTO LOGINS  VALUES (7, '2023-11-18 09:00:00', 1007, 81);
INSERT INTO LOGINS VALUES (8, '2023-12-01 10:30:00', 1008, 84);
INSERT INTO LOGINS  VALUES (9, '2023-12-15 13:15:00', 1009, 79);
INSERT INTO LOGINS VALUES (1, '2024-01-10 07:45:00', 1011, 86);
INSERT INTO LOGINS VALUES (2, '2024-01-25 09:30:00', 1012, 89);
INSERT INTO LOGINS VALUES (3, '2024-02-05 11:00:00', 1013, 78);
INSERT INTO LOGINS VALUES (4, '2024-03-01 14:30:00', 1014, 91);
INSERT INTO LOGINS VALUES (5, '2024-03-15 16:00:00', 1015, 83);
INSERT INTO LOGINS VALUES (6, '2024-04-12 08:00:00', 1016, 80);
INSERT INTO LOGINS VALUES (7, '2024-05-18 09:15:00', 1017, 82);
INSERT INTO LOGINS VALUES (8, '2024-05-28 10:45:00', 1018, 87);
INSERT INTO LOGINS VALUES (9, '2024-06-15 13:30:00', 1019, 76);
INSERT INTO LOGINS VALUES (10, '2024-06-25 15:00:00', 1010, 92);
INSERT INTO LOGINS VALUES (10, '2024-06-26 15:45:00', 1020, 93);
INSERT INTO LOGINS VALUES (10, '2024-06-27 15:00:00', 1021, 92);
INSERT INTO LOGINS VALUES (10, '2024-06-28 15:45:00', 1022, 93);
INSERT INTO LOGINS VALUES (1, '2024-01-10 07:45:00', 1101, 86);
INSERT INTO LOGINS VALUES (3, '2024-01-25 09:30:00', 1102, 89);
INSERT INTO LOGINS VALUES (5, '2024-01-15 11:00:00', 1103, 78);
INSERT INTO LOGINS VALUES (2, '2023-11-10 07:45:00', 1201, 82);
INSERT INTO LOGINS VALUES (4, '2023-11-25 09:30:00', 1202, 84);
INSERT INTO LOGINS VALUES (6, '2023-11-15 11:00:00', 1203, 80);
```


# Questions

## Management wants to see all the users that did not login in the past 5 month Return Username.

```SQL
select USER_NAME from users where USER_NAME not in (
select U.USER_NAME
from logins L
left join users U on L.USER_ID = u.USER_ID
where LOGIN_TIMESTAMP > DATEADD(Month, -5, '2024-06-28')
)
```

## For business units, quarterly analysis Calculate How many users and how many sessions were at each quarter. Order by quarter by newest to oldest. Return first day of the quarter, session count and user count.

Select DATETRUNC(quarter, MIN(login_timestamp)) as First_quarter_date, COUNT(distinct user_id) as No_of_users, COUNT(session_id) as No_of_session
from logins
group by DATEPART(quarter, LOGIN_TIMESTAMP)


## Display user id that log in in january 2024 and did not log in on nov 2023. Return User_ID

```SQL
select distinct USER_ID
from Logins
where LOGIN_TIMESTAMP between '2024-01-01' and '2024-01-31' and USER_ID not in (select USER_ID
from Logins
where LOGIN_TIMESTAMP between '2023-11-01' and '2023-11-30')
```

## Add to the query from 2 the percentage change in sessions from last quarter. Return First day of quarter, session count, session_prev_count, Session_percentage_change

```SQL
with Prev_count_and_percentage as (
Select DATETRUNC(quarter, MIN(login_timestamp)) as First_quarter_date, COUNT(distinct user_id) as No_of_users, COUNT(session_id) as No_of_session
from logins
group by DATEPART(quarter, LOGIN_TIMESTAMP)
)

select *, Lag(No_of_session, 1) over(order by First_quarter_date) as Pre_session_count,
(No_of_session - (Lag(No_of_session, 1) over(order by First_quarter_date))) * 100.0 / (Lag(No_of_session, 1) over(order by First_quarter_date)) as Session_percentage_change
from Prev_count_and_percentage
```

## Display the user that had the highest session score (Max) for each day. Return - Date , username and score

```SQL
with CTE as (
Select l.LOGIN_TIMESTAMP, u.USER_NAME, max(l.SESSION_SCORE) as Highest_score 
from logins L
inner join users U on L.USER_ID = u.USER_ID
group by l.LOGIN_TIMESTAMP, u.USER_NAME
)
select * from (
select *, row_number() over(partition by login_timestamp order by highest_score desc) as RN
from CTE 
) a
where RN = 1
```

## To identify our best users - Return the users that had a session on every single day since their first login (Make assumption if needed). Return UserID

```SQL
with Everysingleday as (
Select User_id, min(login_timestamp) as First_date, Max(login_timestamp) as last_date,
DATEDIFF(Day, min(login_timestamp), Max(login_timestamp)+1 ) as No_days_must_logins, count(distinct Login_timestamp) as Logins
from Logins
group by User_id
having DATEDIFF(Day, min(login_timestamp), Max(login_timestamp)+1 ) = count(distinct Login_timestamp)
)

select USER_ID
from Everysingleday
where No_days_must_logins = Logins
```
