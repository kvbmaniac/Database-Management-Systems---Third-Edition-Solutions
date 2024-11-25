# Database Management Systems - Chapter 5 Solutions

Online material is available for all exercises in this chapter on the book's webpage at -  
http://www.cs.wisc.edu/-dbbook  
This includes scripts to create tables for each exercise for use with Oracle, IBM DB2, Microsoft SQL Server, and MySQL.
### Exercise 5.1
Consider the following relations:  
Student (snum: integer, sname: string, major: string, level: string, age: integer)  
Class( name: string, meets_at: time, room: string, fid: integer)  
Enrolled(snum: integer, cname: string)  
Faculty (fid: integer, fnarne: string, deptid: integer)  

The meaning of these relations is straightforward; for example, Enrolled has one record per student-class pair such that the student is enrolled in the class.

Write the following queries in SQL. No duplicates should be printed in any of the answers.
1. Find the nari1es of all Juniors (level = JR) who are enrolled in a class taught by 1. Teach.
2. Find the age of the oldest student who is either a History major or enrolled in a course
taught by I. Teach.
3. Find the names of all classes that either meet in room R128 or have five or more students
enrolled.
4. Find the names of all students who are enrolled in two classes that meet at the same time.
5. Find the names of faculty members who teach in every room in which some class is taught.
6. Find the names of faculty members for whom the combined enrollment of the courses that they teach is less than five.
7. Print the level and the average age of students for that level, for each level.
8. Print the level and the average age of students for that level, for all levels except JR.
9. For each faculty member that has taught classes only in room R128, print the faculty member's name and the total number of classes she or he has taught.
10. Find the names of students enrolled in the maximum number of classes.
11. Find the names of students not enrolled in any class.
12. For each age value that appears in Students, find the level value that appears most often. For example, if there are more FR level students aged 18 than SR, JR, or SO students aged 18, you should print the pair (18, FR).

#### `Answers`
### Exercise 5.1

1. **Find the names of all Juniors (level = JR) who are enrolled in a class taught by I. Teach.**
   ```sql
   SELECT DISTINCT S.sname
   FROM Student S, Enrolled E, Class C, Faculty F
   WHERE S.snum = E.snum AND E.cname = C.name AND C.fid = F.fid AND S.level = 'JR' AND
    F.fname = 'I. Teach';
   ```

2. **Find the age of the oldest student who is either a History major or enrolled in a course taught by I. Teach.**
   ```sql
   SELECT MAX(S.age)
   FROM Student S
   WHERE S.major = 'History' OR S.snum IN (
       SELECT E.snum
       FROM Enrolled E, Class C, Faculty F
       WHERE E.cname = C.name AND C.fid = F.fid AND F.fname = 'I. Teach'
   );
   ```

3. **Find the names of all classes that either meet in room R128 or have five or more students enrolled.**
   ```sql
   SELECT DISTINCT C.name
   FROM Class C
   WHERE C.room = 'R128'
   UNION
   SELECT C.name
   FROM Class C, Enrolled E
   WHERE C.name = E.cname
   GROUP BY C.name
   HAVING COUNT(E.snum) >= 5;
   ```

4. **Find the names of all students who are enrolled in two classes that meet at the same time.**
   ```sql
   SELECT DISTINCT S.sname
   FROM Student S, Enrolled E1, Enrolled E2, Class C1, Class C2
   WHERE S.snum = E1.snum AND S.snum = E2.snum AND E1.cname = C1.name AND E2.cname = C2.name AND
    C1.meets_at = C2.meets_at AND C1.name <> C2.name;
   ```

5. **Find the names of faculty members who teach in every room in which some class is taught.**
   ```sql
   SELECT F.fname
   FROM Faculty F
   WHERE NOT EXISTS (
       SELECT C.room
       FROM Class C
       WHERE NOT EXISTS (
           SELECT *
           FROM Class C2
           WHERE C2.room = C.room AND C2.fid = F.fid
       )
   );
   ```

6. **Find the names of faculty members for whom the combined enrollment of the courses that they teach is less than five.**
   ```sql
   SELECT F.fname
   FROM Faculty F, Class C
   WHERE F.fid = C.fid
   GROUP BY F.fname
   HAVING SUM((SELECT COUNT(*) FROM Enrolled E WHERE E.cname = C.name)) < 5;
   ```

7. **Print the level and the average age of students for that level, for each level.**
   ```sql
   SELECT S.level, AVG(S.age)
   FROM Student S
   GROUP BY S.level;
   ```

8. **Print the level and the average age of students for that level, for all levels except JR.**
   ```sql
   SELECT S.level, AVG(S.age)
   FROM Student S
   WHERE S.level <> 'JR'
   GROUP BY S.level;
   ```

9. **For each faculty member that has taught classes only in room R128, print the faculty member's name and the total number of classes she or he has taught.**
   ```sql
   SELECT F.fname, COUNT(*)
   FROM Faculty F, Class C
   WHERE F.fid = C.fid AND C.room = 'R128'
   GROUP BY F.fname
   HAVING COUNT(DISTINCT C.room) = 1;
   ```

10. **Find the names of students enrolled in the maximum number of classes.**
    ```sql
    SELECT S.sname
    FROM Student S
    WHERE S.snum IN (
        SELECT E.snum
        FROM Enrolled E
        GROUP BY E.snum
        HAVING COUNT(E.cname) = (
            SELECT MAX(Cnt)
            FROM (SELECT COUNT(E2.cname) AS Cnt FROM Enrolled E2 GROUP BY E2.snum) AS MaxCnt
        )
    );
    ```

11. **Find the names of students not enrolled in any class.**
    ```sql
    SELECT S.sname
    FROM Student S
    WHERE S.snum NOT IN (SELECT E.snum FROM Enrolled E);
    ```

12. **For each age value that appears in Students, find the level value that appears most often.**
    ```sql
    SELECT S.age, S.level
    FROM Student S
    GROUP BY S.age, S.level
    HAVING S.level = (
        SELECT TOP 1 S2.level
        FROM Student S2
        WHERE S2.age = S.age
        GROUP BY S2.level
        ORDER BY COUNT(S2.level) DESC
    );
    ```
***
### Exercise 5.2
Consider the following schema:  
Suppliers( sid: integer, sname: string, address: string)  
Parts(pid: integer, pname: string, color: string)  
Catalog( sid: integer, pid: integer, cost: real)  
The Catalog relation lists the prices charged for parts by Suppliers.

Write the following queries in SQL:
1. Find the pnames of parts for which there is some supplier.
2. Find the snames of suppliers who supply every part.
3. Find the snames of suppliers who supply every red part.
4. Find the pnames of parts supplied by Acme Widget Suppliers and no one else.
5. Find the sids of suppliers who charge more for some part than the average cost of that part (averaged over all the suppliers who supply that part).
6. For each part, find the sname of the supplier who charges the most for that part.
7. Find the sids of suppliers who supply only red parts.
8. Find the sids of suppliers who supply a red part and a green part.
9. Find the sids of suppliers who supply a red part or a green part.
10. For every supplier that only supplies green parts, print the name of the supplier and the total number of parts that she supplies.
11. For every supplier that supplies a green part and a reel part, print the name and price of the most expensive part that she supplies.

#### `Answer`
### Exercise 5.2

1. **Find the pnames of parts for which there is some supplier.**
   ```sql
   SELECT DISTINCT P.pname
   FROM Parts P, Catalog C
   WHERE P.pid = C.pid;
   ```

2. **Find the snames of suppliers who supply every part.**
   ```sql
   SELECT S.sname
   FROM Suppliers S
   WHERE NOT EXISTS (
       SELECT P.pid
       FROM Parts P
       WHERE NOT EXISTS (
           SELECT *
           FROM Catalog C
           WHERE C.sid = S.sid AND C.pid = P.pid
       )
   );
   ```

3. **Find the snames of suppliers who supply every red part.**
   ```sql
   SELECT S.sname
   FROM Suppliers S
   WHERE NOT EXISTS (
       SELECT P.pid
       FROM Parts P
       WHERE P.color = 'red' AND NOT EXISTS (
           SELECT *
           FROM Catalog C
           WHERE C.sid = S.sid AND C.pid = P.pid
       )
   );
   ```

4. **Find the pnames of parts supplied by Acme Widget Suppliers and no one else.**
   ```sql
   SELECT P.pname
   FROM Parts P, Catalog C, Suppliers S
   WHERE P.pid = C.pid AND C.sid = S.sid AND S.sname = 'Acme Widget Suppliers'
   AND NOT EXISTS (
       SELECT *
       FROM Catalog C2
       WHERE C2.pid = P.pid AND C2.sid <> S.sid
   );
   ```

5. **Find the sids of suppliers who charge more for some part than the average cost of that part.**
   ```sql
   SELECT DISTINCT C.sid
   FROM Catalog C
   WHERE C.cost > (SELECT AVG(C2.cost) FROM Catalog C2 WHERE C2.pid = C.pid);
   ```

6. **For each part, find the sname of the supplier who charges the most for that part.**
   ```sql
   SELECT P.pname, S.sname
   FROM Parts P, Suppliers S, Catalog C
   WHERE P.pid = C.pid AND C.sid = S.sid
   AND C.cost = (SELECT MAX(C2.cost) FROM Catalog C2 WHERE C2.pid = P.pid);
   ```

7. **Find the sids of suppliers who supply only red parts.**
   ```sql
   SELECT DISTINCT S.sid
   FROM Suppliers S
   WHERE NOT EXISTS (
       SELECT P.pid
       FROM Parts P, Catalog C
       WHERE P.pid = C.pid AND C.sid = S.sid AND P.color <> 'red'
   );
   ```

8. **Find the sids of suppliers who supply a red part and a green part.**
   ```sql
   SELECT DISTINCT C.sid
   FROM Catalog C, Parts P1, Parts P2
   WHERE C.pid = P1.pid AND P1.color = 'red'
   AND C.sid IN (
       SELECT C2.sid
       FROM Catalog C2, Parts P2
       WHERE C2.pid = P2.pid AND P2.color = 'green'
   );
   ```

9. **Find the sids of suppliers who supply a red part or a green part.**
   ```sql
   SELECT DISTINCT C.sid
   FROM Catalog C, Parts P
   WHERE P.pid = C.pid AND (P.color = 'red' OR P.color = 'green');
   ```

10. **For every supplier that only supplies green parts, print the name of the supplier and the total number of parts that she supplies.**
    ```sql
    SELECT S.sname, COUNT(*)
    FROM Suppliers S, Catalog C, Parts P
    WHERE S.sid = C.sid AND C.pid = P.pid AND P.color = 'green'
    GROUP BY S.sname
    HAVING COUNT(DISTINCT P.color) = 1;
    ```

11. **For every supplier that supplies a green part and a red part, print the name and price of the most expensive part that she supplies.**
    ```sql
    SELECT S.sname, MAX(C.cost)
    FROM Suppliers S, Catalog C, Parts P
    WHERE S.sid = C.sid AND C.pid = P.pid AND (P.color = 'red' OR P.color = 'green')
    GROUP BY S.sname
    HAVING COUNT(DISTINCT P.color) = 2;
    ```


***
### Exercise 5.3
The following relations keep track of airline flight information:  
Flights(.flno: integer, from: string, to: string, distance: integer,
departs: time, arrives: time, price: integer)  
Aircraft( aid: integer, aname: string, cruisingrange: integer)  
Certified( eid: integer, aid: integer)  
Employees( eid: integer I ename: string, salary: integer)  

Note that the Employees relation describes pilots and other kinds of employees as well; every pilot is certified for some aircraft, and only pilots are certified to fly. Write each of the following queries in SQL. (Additional queries using the same schema are listed in the exercises for Chapter 4)

1. Find the names of aircraft such that all pilots certified to operate them earn more than $80,000.
2. For each pilot who is certified for more than three aircraft, find the eid and the maximum cruisingrange of the aircraft for which she or he is certified.
3. Find the names of pilots whose salary is less than the price of the cheapest route from Los Angeles to Honolulu.
4. For all aircraft with cruisingrange over 1000 miles, find the name of the aircraft and the average salary of all pilots certified for this aircraft.
5. Find the names of pilots certified for some Boeing aircraft.
6. Find the aids of all aircraft that can be used on routes from Los Angeles to Chicago.
7. Identify the routes that can be piloted by every pilot who makes more than $100,000.
8. Print the enames of pilots who can operate planes with cruisingrange greater than 3000 miles but are not certified on any Boeing aircraft.
9. A customer wants to travel from Madison to New York with no more than two changes of flight. List the choice of departure times from Madison if the customer wants to arrive in New York by 6 p.m.
10. Compute the difference between the average salary of a pilot and the average salary of all employees (including pilots).
11. Print the name and salary of every nonpilot whose salary is more than the average salary for pilots.
12. Print the names of employees who are certified only on aircrafts with cruising range longer than 1000 miles.
13. Print the names of employees who are certified only on aircrafts with cruising range longer than 1000 miles, but on at least two such aircrafts.
14. Print the names of employees who are certified only on aircrafts with cruising range longer than 1000 miles and who are certified on some Boeing aircraft.

#### `Answer`
### Exercise 5.3

1. **Find the names of aircraft such that all pilots certified to operate them earn more than $80,000.**
   ```sql
   SELECT A.aname
   FROM Aircraft A
   WHERE NOT EXISTS (
       SELECT C.eid
       FROM Certified C, Employees E
       WHERE C.aid = A.aid AND C.eid = E.eid AND E.salary <= 80000
   );
   ```

2. **For each pilot who is certified for more than three aircraft, find the eid and the maximum cruisingrange of the aircraft for which she or he is certified.**
   ```sql
   SELECT C.eid, MAX(A.cruisingrange)
   FROM Certified C, Aircraft A
   WHERE C.aid = A.aid
   GROUP BY C.eid
   HAVING COUNT(C.aid) > 3;
   ```

3. **Find the names of pilots whose salary is less than the price of the cheapest route from Los Angeles to Honolulu.**
   ```sql
   SELECT E.ename
   FROM Employees E
   WHERE E.salary < (
       SELECT MIN(F.price)
       FROM Flights F
       WHERE F.from = 'Los Angeles' AND F.to = 'Honolulu'
   );
   ```

4. **For all aircraft with cruisingrange over 1000 miles, find the name of the aircraft and the average salary of all pilots certified for this aircraft.**
   ```sql
   SELECT A.aname, AVG(E.salary)
   FROM Aircraft A, Certified C, Employees E
   WHERE A.aid = C.aid AND C.eid = E.eid AND A.cruisingrange > 1000
   GROUP BY A.aname;
   ```

5. **Find the names of pilots certified for some Boeing aircraft.**
   ```sql
   SELECT DISTINCT E.ename
   FROM Employees E, Certified C, Aircraft A
   WHERE E.eid = C.eid AND C.aid = A.aid AND A.aname LIKE 'Boeing%';
   ```

6. **Find the aids of all aircraft that can be used on routes from Los Angeles to Chicago.**
   ```sql
   SELECT A.aid
   FROM Aircraft A, Flights F
   WHERE F.from = 'Los Angeles' AND F.to = 'Chicago' AND A.cruisingrange >= F.distance;
   ```

7. **Identify the routes that can be piloted by every pilot who makes more than $100,000.**
   ```sql
   SELECT F.flno
   FROM Flights F
   WHERE NOT EXISTS (
       SELECT E.eid
       FROM Employees E
       WHERE E.salary > 100000 AND NOT EXISTS (
           SELECT C.aid
           FROM Certified C, Aircraft A
           WHERE E.eid = C.eid AND C.aid = A.aid AND A.cruisingrange >= F.distance
       )
   );
   ```

8. **Print the enames of pilots who can operate planes with cruisingrange greater than 3000 miles but are not certified on any Boeing aircraft.**
   ```sql
   SELECT DISTINCT E.ename
   FROM Employees E, Certified C, Aircraft A
   WHERE E.eid = C.eid AND A.aid = C.aid AND A.cruisingrange > 3000 AND E.eid NOT IN (
       SELECT C2.eid
       FROM Certified C2, Aircraft A2
       WHERE C2.aid = A2.aid AND A2.aname LIKE 'Boeing%'
   );
   ```

9. **List the choice of departure times from Madison if the customer wants to arrive in New York by 6 p.m. with no more than two changes of flight.**
   ```sql
   SELECT F1.departs
   FROM Flights F1, Flights F2
   WHERE F1.from = 'Madison' AND F2.to = 'New York' AND F1.to = F2.from AND F2.arrives <= '18:00'
   UNION
   SELECT F.departs
   FROM Flights F
   WHERE F.from = 'Madison' AND F.to = 'New York' AND F.arrives <= '18:00';
   ```

10. **Compute the difference between the average salary of a pilot and the average salary of all employees (including pilots).**
    ```sql
    SELECT (SELECT AVG(E.salary) FROM Employees E, Certified C WHERE E.eid = C.eid) - (SELECT AVG(E.salary) FROM Employees E) AS SalaryDifference;
    ```

11. **Print the name and salary of every nonpilot whose salary is more than the average salary for pilots.**
    ```sql
    SELECT E.ename, E.salary
    FROM Employees E
    WHERE E.eid NOT IN (SELECT C.eid FROM Certified C)
    AND E.salary > (SELECT AVG(E2.salary) FROM Employees E2, Certified C2 WHERE E2.eid = C2.eid);
    ```

12. **Print the names of employees who are certified only on aircrafts with cruising range longer than 1000 miles.**
    ```sql
    SELECT E.ename
    FROM Employees E
    WHERE NOT EXISTS (
        SELECT C.aid
        FROM Certified C, Aircraft A
        WHERE C.eid = E.eid AND C.aid = A.aid AND A.cruisingrange <= 1000
    );
    ```

***
### Exercise 5.4
Consider the following relational schema. An employee can work in more than one department; the pct_time field of the Works relation shows the percentage of time that a given employee works in a given department.

Emp(eid: integer, ename: string, age: integer, salary: real)  
Works(eid: integer, did: integer, pct_time: integer)  
Dept(did.· integer, budget: real, managerid: integer)  

Write the following queries in SQL:
1. Print the names and ages of each employee who works in both the Hardware department
and the Software department.
2. For each department with more than 20 full-time-equivalent employees (i.e., where the part-time and full-time employees add up to at least that many full-time employees), print the did together with the number of employees that work in that department.
3. Print the name of each employee whose salary exceeds the budget of all of the departments that he or she works in.
4. Find the managerids of managers who manage only departments with budgets greater than $1 million.
5. Find the enames of managers who manage the departments with the largest budgets.
6. If a manager manages more than one department, he or she controls the sum of all the budgets for those departments. Find the managerids of managers who control more than $5 million.
7. Find the managerids of managers who control the largest amounts.
8. Find the enames of managers who manage only departments with budgets larger than $1 million, but at least one department with budget less than $5 million.

#### `Answer`
### Exercise 5.4

1. **Find the pids of parts that are supplied by at least two different suppliers.**
   ```sql
   SELECT C.pid
   FROM Catalog C
   GROUP BY C.pid
   HAVING COUNT(DISTINCT C.sid) >= 2;
   ```

2. **Find the pids of parts that are supplied by every supplier who supplies part P2.**
   ```sql
   SELECT C1.pid
   FROM Catalog C1
   WHERE NOT EXISTS (
       SELECT C2.sid
       FROM Catalog C2
       WHERE C2.pid = 'P2' AND NOT EXISTS (
           SELECT C3.pid
           FROM Catalog C3
           WHERE C3.pid = C1.pid AND C3.sid = C2.sid
       )
   );
   ```

3. **Find the pids of the most expensive parts supplied by suppliers named Yosemite Sham.**
   ```sql
   SELECT C.pid
   FROM Catalog C, Suppliers S
   WHERE C.sid = S.sid AND S.sname = 'Yosemite Sham'
   AND C.cost = (SELECT MAX(C2.cost) FROM Catalog C2 WHERE C2.sid = S.sid);
   ```

4. **Find the snames of suppliers who supply some red part or are at the same city as supplier S2.**
   ```sql
   SELECT DISTINCT S.sname
   FROM Suppliers S, Parts P, Catalog C
   WHERE (S.sid = C.sid AND C.pid = P.pid AND P.color = 'red')
   OR S.city = (SELECT S2.city FROM Suppliers S2 WHERE S2.sid = 'S2');
   ```

5. **Find the snames of suppliers who supply every part supplied by supplier S1.**
   ```sql
   SELECT S.sname
   FROM Suppliers S
   WHERE NOT EXISTS (
       SELECT C1.pid
       FROM Catalog C1
       WHERE C1.sid = 'S1' AND NOT EXISTS (
           SELECT C2.pid
           FROM Catalog C2
           WHERE C2.sid = S.sid AND C2.pid = C1.pid
       )
   );
   ```

6. **Find the snames of suppliers who do not supply part P1.**
   ```sql
   SELECT S.sname
   FROM Suppliers S
   WHERE NOT EXISTS (
       SELECT *
       FROM Catalog C
       WHERE C.sid = S.sid AND C.pid = 'P1'
   );
   ```


***
### Exercise 5.5
Consider the instance of the Sailors relation shown in Figure 5.22.
1. Write SQL queries to compute the average rating, using AVG;  the sum of the ratings, using SUM; and the number of ratings, using COUNT.
2. If you divide the sum just computed by the count, would the result be the same as the average? How would your answer change if these steps were carried out with respect to the age field instead of rating?
3. Consider the following query: Find the names of sailors with a higher rating than all sailors with age < 21. The following two SQL queries attempt to obtain the answer to this question. Do they both compute the result? If not, explain why. Under what conditions would they compute the same result?

SELECT S.sname  
FROM Sailors S  
WHERE NOT EXISTS ( SELECT *  
FROM Sailors S2  
WHERE S2.age < 21  
AND S.rating <= S2.rating )  

SELECT *  
FROM Sailors S  
WHERE S.rating > ANY  
SELECT S2.rating  
FROM Sailors S2  
WHERE S2.age < 21  


4. Consider the instance of Sailors shown in Figure 5.22. Let us define instance Sl of Sailors to consist of the first two tuples, instance S2 to be the last two tuples, and S to be the given instance.

(a) Show the left outer join of S with itself, with the join condition being sid=sid.  
(b) Show the right outer join of S ,vith itself, with the join condition being sid=sid.  
(c) Show the full outer join of S with itself, with the join condition being Sid=sid.  
(d) Show the left outer join of Sl with S2, with the join condition being sid=sid.  
(e) Show the right outer join of Sl with S2, with the join condition being sid=sid.  
(f) Show the full outer join of 81 with S2, with the join condition being sid=sid.  

#### `Answer`
### Exercise 5.5

1. **Find the names of sailors who’ve reserved a red boat.**
   ```sql
   SELECT S.sname
   FROM Sailors S, Boats B, Reserves R
   WHERE S.sid = R.sid AND R.bid = B.bid AND B.color = 'red';
   ```

2. **Find the sids of sailors who’ve reserved a red or a green boat.**
   ```sql
   SELECT DISTINCT R.sid
   FROM Reserves R, Boats B
   WHERE R.bid = B.bid AND (B.color = 'red' OR B.color = 'green');
   ```

3. **Find the names of sailors who’ve reserved both a red and a green boat.**
   ```sql
   SELECT S.sname
   FROM Sailors S
   WHERE S.sid IN (
       SELECT R1.sid
       FROM Reserves R1, Boats B1
       WHERE R1.bid = B1.bid AND B1.color = 'red'
   )
   AND S.sid IN (
       SELECT R2.sid
       FROM Reserves R2, Boats B2
       WHERE R2.bid = B2.bid AND B2.color = 'green'
   );
   ```

4. **Find the sids of sailors with age over 20 who’ve not reserved a red boat.**
   ```sql
   SELECT S.sid
   FROM Sailors S
   WHERE S.age > 20
   AND NOT EXISTS (
       SELECT R.sid
       FROM Reserves R, Boats B
       WHERE S.sid = R.sid AND R.bid = B.bid AND B.color = 'red'
   );
   ```

5. **Find the names of sailors who’ve reserved all boats.**
   ```sql
   SELECT S.sname
   FROM Sailors S
   WHERE NOT EXISTS (
       SELECT B.bid
       FROM Boats B
       WHERE NOT EXISTS (
           SELECT R.bid
           FROM Reserves R
           WHERE S.sid = R.sid AND R.bid = B.bid
       )
   );
   ```

6. **Find the sids of sailors who’ve reserved all red boats.**
   ```sql
   SELECT S.sid
   FROM Sailors S
   WHERE NOT EXISTS (
       SELECT B.bid
       FROM Boats B
       WHERE B.color = 'red' AND NOT EXISTS (
           SELECT R.sid
           FROM Reserves R
           WHERE S.sid = R.sid AND R.bid = B.bid
       )
   );
   ```

7. **Find the names of sailors who’ve reserved at least two boats.**
   ```sql
   SELECT S.sname
   FROM Sailors S, Reserves R
   WHERE S.sid = R.sid
   GROUP BY S.sname
   HAVING COUNT(R.bid) >= 2;
   ```

8. **Find the names of sailors who’ve reserved a red boat and a green boat.**
   ```sql
   SELECT S.sname
   FROM Sailors S
   WHERE S.sid IN (
       SELECT R1.sid
       FROM Reserves R1, Boats B1
       WHERE R1.bid = B1.bid AND B1.color = 'red'
   )
   AND S.sid IN (
       SELECT R2.sid
       FROM Reserves R2, Boats B2
       WHERE R2.bid = B2.bid AND B2.color = 'green'
   );
   ```

***
### Exercise 5.6
Answer the following questions:
1. Explain the term impedance mismatch in the context of embedding SQL commands in a host language such as C.
2. How can the value of a host language variable be passed to an embedded SQL command?
3. Explain the WHENEVER command's use in error and exception handling.
4. Explain the need for cursors.
5. Give an example of a situation that calls for the use of embedded SQL; that is, interactive use of SQL commands is not enough, and some host language capabilities are needed.
6. Write a C program with embedded SQL commands to address your example in the previous answer.
7. Write a C program with embedded SQL commands to find the standard deviation of sailors' ages.
8. Extend the previous program to find all sailors whose age is within one standard deviation of the average age of all sailors.
9. Explain how you would write a C program to compute the transitive closure of a graph, represented as an 8QL relation Edges(jrom, to), using embedded SQL commands. (You need not write the program, just explain the main points to be dealt with.)
10. Explain the following terms with respect to cursors: 'updatability, sensitivity, and scrollability.
11. Define a cursor on the Sailors relation that is updatable, scrollable, and returns answers sorted by age. Which fields of Sailors can such a cursor not update? Why?
12. Give an example of a situation that calls for dynamic 8QL; that is, even embedded SQL is not sufficient.

#### `Answer`
### Exercise 5.6

1. **Find the sids of sailors who’ve reserved all boats.**
   ```sql
   SELECT S.sid
   FROM Sailors S
   WHERE NOT EXISTS (
       SELECT B.bid
       FROM Boats B
       WHERE NOT EXISTS (
           SELECT R.bid
           FROM Reserves R
           WHERE S.sid = R.sid AND R.bid = B.bid
       )
   );
   ```

2. **Find the sids of sailors who’ve reserved all red boats.**
   ```sql
   SELECT S.sid
   FROM Sailors S
   WHERE NOT EXISTS (
       SELECT B.bid
       FROM Boats B
       WHERE B.color = 'red' AND NOT EXISTS (
           SELECT R.sid
           FROM Reserves R
           WHERE S.sid = R.sid AND R.bid = B.bid
       )
   );
   ```

3. **Find the sids of sailors with age over 20 who’ve not reserved a red boat.**
   ```sql
   SELECT S.sid
   FROM Sailors S
   WHERE S.age > 20
   AND NOT EXISTS (
       SELECT R.sid
       FROM Reserves R, Boats B
       WHERE S.sid = R.sid AND R.bid = B.bid AND B.color = 'red'
   );
   ```

4. **Find the sids of sailors who’ve reserved a red and a green boat.**
   ```sql
   SELECT S.sid
   FROM Sailors S
   WHERE S.sid IN (
       SELECT R1.sid
       FROM Reserves R1, Boats B1
       WHERE R1.bid = B1.bid AND B1.color = 'red'
   )
   AND S.sid IN (
       SELECT R2.sid
       FROM Reserves R2, Boats B2
       WHERE R2.bid = B2.bid AND B2.color = 'green'
   );
   ```

***
### Exercise 5.7
Consider the following relational schema and briefly answer the questions that follow:  
Emp (eid: integer, cname: string, age: integer, salary: real)  
Works (eid: integer, did: integer, pet-time: integer)  
Dept (did.' integer, budget: real, managerid: integer)  
1. Define a table constraint on Emp that will ensure that ever)' employee makes at least $10,000.
2. Define a table constraint on Dept that will ensure that all managers have age> 30.
3. Define an assertion on Dept that will ensure that all managers have age> 30. Compare this assertion with the equivalent table constraint. Explain which is better.
4. Write SQL statements to delete all information about employees whose salaries exceed that of the manager of one or more departments that they work in. Be sure to ensure that all the relevant integrity constraints are satisfied after your updates.

#### `Answer`

***
### Exercise 5.8
Consider the following relations:  

Student (snum: integer, sname: string, major: string, level: string, age: integer)  
Class(name: string, meets_at: time, room: string, fid: integer)
Enrolled(snurn: integer, cnarne: string)  
Faculty(fid: integer, fnarne: string, deptid: integer)  

The meaning of these relations is straightforward; for example, Enrolled has one record per student-class pair such that the student is enrolled in the class.
1. Write the SQL statements required to create these relations, including appropriate versions of all primary and foreign key integrity constraints.
2. Express each of the following integrity constraints in SQL unless it is implied by the primary and foreign key constraint; if so, explain how it is implied. If the constraint cannot be expressed in SQL, say so. For each constraint, state what operations (inserts,
deletes, and updates on specific relations) must be monitored to enforce the constraint.
 1. Every class has a minimum enrollment of 5 students and a maximum enrollment of 30 students.
 2. At least one dass meets in each room.
 3. Every faculty member must teach at least two courses.
 4. Only faculty in the department with deptid=33 teach more than three courses.
 5. Every student must be enrolled in the course called Mathl0l.
 6. The room in which the earliest scheduled class (i.e., the class with the smallest meets_at value) meets should not be the same as the room in which the latest scheduled class meets.
 7. Two classes cannot meet in the same room at the same time.
 8. The department with the most faculty members must have fewer than twice the number of faculty members in the department with the fewest faculty members.
 9. No department can have more than 10 faculty members.
 10. A student cannot add more than two courses at a time (i.e., in a single update).
 11. The number of CS majors must be more than the number of Math majors.
 12. The number of distinct courses in which CS majors are enrolled is greater than the number of distinct courses in which Math majors are enrolled.
 13. The total enrollment in courses taught by faculty in the department with deptid is greater than the number of Math majors.
 14. There mUst be at least one CS major if there are any students whatsoever.
 15. Faculty members from different departments cannot teach in the same room.


#### `Answer`

***
### Exercise 5.9
Discuss the strengths and weaknesses of the trigger mechanism. Contrast triggers with other integrity constraints supported by SQL.

#### `Answer`

***
### Exercise 5.10
Consider the following relational schema. An employee can work in more than one department; the pct_time field of the Works relation shows the percentage of time that a given employee works in a given department.  

Emp (eid: integer, ename: string, age: integer, salary: real)  
Works (eid: integer, did: integer, pct_time: integer)  
Dept (did: integer, budget: real, managerid: integer)  

Write SQL-92 integrity constraints (domain, key, foreign key, or CHECK constraints; or assertions) or SQL:1999 triggers to ensure each of the following requirements, considered independently.
1. Employees must make a minimum salary of $1000.
2. Every manager must be also be an employee.
3. The total percentage of all appointments for an employee must be under 100%.
4. A manager must always have a higher salary than any employee that he or she manages.
5. Whenever an employee is given a raise, the manager's salary must be increased to be at least as much.
6. Whenever an employee is given a raise, the manager's salary must be increased to be at least as much. Further, whenever an employee is given a raise, the department's budget must be increased.


#### `Answer`

***
