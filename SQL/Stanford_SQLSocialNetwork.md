Q1. Find the names of all students who are friends with someone named Gabriel. 

    SELECT HS.name
    FROM (SELECT ID1 FROM Friend WHERE ID2 IN (SELECT ID FROM Highschooler AS HS WHERE HS.name='Gabriel')) AS friendID
    JOIN Highschooler AS HS
    ON friendID.ID1 = HS.ID

Q2. For every student who likes someone 2 or more grades younger than themselves, return that student's name and grade, and the name and grade of the student they like. 

    SELECT HS1.name as n1, HS1.grade as g1,  HS2.name as n2, HS2.grade as g2
    FROM Likes L
    LEFT JOIN Highschooler AS HS1
    ON L.ID1=HS1.ID
    LEFT JOIN Highschooler AS HS2
    ON L.ID2=HS2.ID
    WHERE g1 >= g2+2

Q3. For every pair of students who both like each other, return the name and grade of both students. Include each pair only once, with the two names in alphabetical order. 

    SELECT n1, g1, n2, g2
    FROM (SELECT mutual.couple AS couple, HS1.name AS n1, HS1.grade AS g1, HS2.name AS n2, HS2.grade AS g2
    FROM (SELECT L1.ID1+L1.ID2 as couple, L1.ID1 as id1, L1.ID2 as id2, L2.ID1 as id3, L2.ID2 as id4
    FROM LIKES L1, LIKES L2
    WHERE L1.ID1=L2.ID2 AND L1.ID2=L2.ID1) AS mutual
    LEFT JOIN Highschooler AS HS1
    ON HS1.ID = mutual.id1
    LEFT JOIN Highschooler AS HS2
    ON HS2.ID = mutual.id2
    ORDER BY HS1.name<HS2.name)
    GROUP BY couple

Q4. Find all students who do not appear in the Likes table (as a student who likes or is liked) and return their names and grades. Sort by grade, then by name within each grade. 

NOTE: this first code is not consistent with SQLite

    SELECT L1.name L1.grade
    FROM (SELECT HS.name HS.grade
    FROM Highschoolers AS HS
    LEFT JOIN Likes AS L
    WHERE ISNULL(L.ID1) AS L1
    JOIN
    (SELECT HS.name HS.grade
    FROM Highschoolers AS HS
    LEFT JOIN Likes AS L
    WHERE ISNULL(L.ID2) AS L2
    ORDER BY L1.grade, L1.name

Better code consistent with SQLite

    SELECT name,grade 
    FROM Highschooler 
    WHERE ID NOT IN
    (SELECT ID1 
    FROM Likes 
    UNION 
    SELECT ID2 
    FROM Likes) 
    ORDER BY grade, name

Q5. For every situation where student A likes student B, but we have no information about whom B likes (that is, B does not appear as an ID1 in the Likes table), return A and B's names and grades. 

    SELECT HS1.name, HS1.grade, HS2.name, HS2.grade
    FROM (SELECT ID1, ID2 
    FROM Likes
    WHERE ID2 IN (SELECT ID2 FROM Likes
    WHERE ID2 NOT IN (SELECT ID1 FROM Likes) ) ) AS B
    LEFT JOIN Highschooler AS HS1
    ON HS1.ID = B.ID1
    LEFT JOIN Highschooler AS HS2
    ON HS2.ID = B.ID2 

Q6. Find names and grades of students who only have friends in the same grade. Return the result sorted by grade, then by name within each grade. 

    SELECT name, grade
    FROM Highschooler
    WHERE ID NOT IN
    (SELECT ID1
    FROM Friend 
    JOIN Highschooler AS HS1 ON Friend.ID1 = HS1.ID
    JOIN Highschooler AS HS2 on Friend.ID2 = HS2.ID
    WHERE HS1.grade != HS2.grade)
    ORDER BY grade, name

Q7. For each student A who likes a student B where the two are not friends, find if they have a friend C in common (who can introduce them!). For all such trios, return the name and grade of A, B, and C. 

> Written with [StackEdit](https://stackedit.io/).