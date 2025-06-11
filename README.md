# Library-Management-System
# Library Management System using SQL Project

![library](https://github.com/user-attachments/assets/6394d021-cc08-49fa-8292-80fad7500bd0)

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

![ERD1](https://github.com/user-attachments/assets/1ac0cfce-50a2-4dee-b65c-f4dbf245e677)

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library_db;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);



-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);



-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

-- Project TASK


-- Task 1. Create a New Book Record
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
```sql
SELECT * FROM BOOKS;
INSERT INTO BOOKS
	(isbn, book_title, category, rental_price, 	status, author, publisher)
VALUES
	('978-1-60129-456-2', 'To Kill a Mockingbird', 	'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. 	Lippincott & Co.');
```

--Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
```

-- Task 3: Delete a Record from the Issued Status Table
-- Objective: Delete the record with issued_id = 'IS129' from the issued_status table.
```sql
SELECT * FROM ISSUED_STATUS where ISSUED_ID = 'IS132';

DELETE FROM ISSUED_STATUS
WHERE ISSUED_ID = 'IS132';
```
-- Task 4: Retrieve All Books Issued by a Specific Employee
-- Objective: Select all books issued by the employee with emp_id = 'E101'.

```sql
SELECT * FROM issued_status where issued_emp_id = 'E101';
```

-- Task 5: List Members Who Have Issued More Than One Book
-- Objective: Use GROUP BY to find members who have issued more than one book.
```sql
select * from issued_status as ist;
SELECT * FROM members as m;

SELECT member_id, count(issued_id) FROM ISSUED_STATUS 
join members 
on ISSUED_STATUS.issued_member_id = members.member_id
group by member_id
having count(issued_id)>1;


select * from issued_status;

```
### 3. CTAS (Create Table As Select)

-- Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt
```sql
create table books_cnt
as
select 
	b.isbn,
	b.book_title,
	count(ist.issued_id) as no_book
from issued_status as ist
join books as b
on ist.issued_book_isbn = b.isbn
group by 1,2;

select * from books_cnt;

```
### 4. Data Analysis & Findings

-- Task 7. **Retrieve All Books in a Specific Category:
```sql
select * from books;
select * from books  where category = 'Classic';
```
-- Task 8: Find Total Rental Income by Category:
```sql
select 
	b.category,
	sum(b.rental_price) as total_price,
	count(*) as total_books
from issued_status as ist
join books as b
on ist.issued_book_isbn = b.isbn
group by 1;

```
-- Task 9. **List Members Who Registered in the Last 180 Days**:
```sql
select * from members;

insert into members(member_id,member_name,member_address,reg_date)
values
('C120','Aniket','133 Apple St','2025-04-25'),
('C121','Sumit','124 Sunst St','2025-03-20');

select * from members where reg_date >= current_date - interval '180 days';
```
-- Task 10: List Employees with Their Branch Manager's Name and their branch details**:
```sql
select 
	e1.*,
	b.manager_id,
	e2.emp_name as Manager_name
from employees as e1
join branch as b 
on e1.branch_id = b.branch_id 
join employees as e2
on b.manager_id = e2.emp_id;

```
-- Task 11.** Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
create table books_threshold
as
select * from books where rental_price > 6.5;

```
-- Task 12:** Retrieve the List of Books Not Yet Returned**
```sql
select 
	distinct ist.issued_book_name 
from issued_status as ist
left join return_status as rts on
ist.issued_id = rts.issued_id
where return_date is null;

```
### Advanced SQL Operations

-- Task 13: Identify Members with Overdue Books
-- Write a query to identify members who have overdue books (assume a 30-day return period). 
-- Display the member's name, book title, issue date, and days overdue.

```sql
select
	m.member_name, 
	bk.book_title, 
	ist.issued_date, 
	(current_date-ist.issued_date ) as return_book
from issued_status as ist
join members as m 
on ist.issued_member_id = m.member_id 
join books as bk
on ist.issued_book_isbn = bk.isbn
left join return_status as rst
on ist.issued_id = rst.issued_id
where 
rst.return_date is null
and
(current_date-ist.issued_date) > 30
order by bk.book_title;
```

--Task 14:** Update Book Status on Return
--Write a query to update the status of books in the books table to "available" when they are returned (based on entries in the return_status table)**.
```sql
select 
	*
from return_status as rst
left join books as b
on rst.return_book_isbn = b.isbn;



update table books
 set status = 'available'
 where isbn in
 (select return_book_isbn from return_status
 where return_date is not null
 );

select * from books;
select * from return_status;

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-451-52994-2';


SELECT * FROM books
WHERE isbn = '978-0-451-52994-2';

UPDATE books
SET status = 'no'
WHERE isbn = '978-0-451-52994-2';


select * FROM return_status
WHERE issued_id = 'IS130';

INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
VALUES
('RS125', 'IS130', CURRENT_DATE, 'Good');


SELECT * FROM return_status
WHERE issued_id = 'IS130';


-- Store Procedures
CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10), p_book_quality VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
    
BEGIN
    -- all your logic and code
    -- inserting into returns based on users input
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES
    (p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

	SELECT 
        issued_book_isbn,
        issued_book_name
        INTO
        v_isbn,
        v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
    
END;
$$



SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

select * FROM return_status
WHERE issued_id = 'IS135';



-- calling function 
CALL add_return_records('RS138', 'IS135', 'Good');

```



--Task 15: Branch Performance Report
--Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
SELECT * FROM branch;

SELECT * FROM issued_status;

SELECT * FROM employees;

SELECT * FROM books;

SELECT * FROM return_status;

CREATE TABLE branch_reports
AS
SELECT 
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) as number_book_issued,
    COUNT(rs.return_id) as number_of_book_return,
    SUM(bk.rental_price) as total_revenue
FROM issued_status as ist
JOIN 
employees as e
ON e.emp_id = ist.issued_emp_id
JOIN
branch as b
ON e.branch_id = b.branch_id
LEFT JOIN
return_status as rs
ON rs.issued_id = ist.issued_id
JOIN 
books as bk
ON ist.issued_book_isbn = bk.isbn
GROUP BY 1, 2;

SELECT * FROM branch_reports;
```

-- Task 16: CTAS: Create a Table of Active Members
-- Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.
```sql
CREATE TABLE active_members
AS
SELECT * FROM members
WHERE member_id IN (SELECT 
                        DISTINCT issued_member_id   
                    FROM issued_status
                    WHERE 
                        issued_date >= CURRENT_DATE - INTERVAL '2 month'
                    )
;


SELECT * FROM active_members;
```
-- Task 17: Find Employees with the Most Book Issues Processed
-- Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.
```sql
SELECT 
    e.emp_name,
    b.*,
    COUNT(ist.issued_id) as no_book_issued
FROM issued_status as ist
JOIN
employees as e
ON e.emp_id = ist.issued_emp_id
JOIN
branch as b
ON e.branch_id = b.branch_id
GROUP BY 1, 2 limit 3;
```
--Task 18: Identify Members Issuing High-Risk Books
--Write a query to identify members who have issued books atleast once with the status "damaged" in the books table. 
--Display the member name, book title, and the number of times they've issued damaged books

```sql
select * from members;

select * from issued_status;

select * from return_status;

select * from books;


select
	m.member_name,
	b.book_title,
	count(*) as damaged_issue_count
from books as b
join issued_status as ist
on b.isbn = ist.issued_book_isbn
join members as m
on m.member_id = ist.issued_member_id
left join return_status as rst
on ist.issued_id = rst.issued_id
where rst.book_quality = 'Damaged'
group by 1,2
having count(*) = 1;

```

/*
Task 19: Stored Procedure Objective: 

Create a stored procedure to manage the status of books in a library system. 

Description: Write a stored procedure that updates the status of a book in the library based on its issuance. 

The procedure should function as follows: 

The stored procedure should take the book_id as an input parameter. 

The procedure should first check if the book is available (status = 'yes'). 

If the book is available, it should be issued, and the status in the books table should be updated to 'no'. 

If the book is not available (status = 'no'), the procedure should return an error message indicating 
that the book is currently not available.
*/

```sql
CREATE OR REPLACE PROCEDURE issue_book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(30), p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
-- all the variabable
    v_status VARCHAR(10);

BEGIN
-- all the code
    -- checking if book is available 'yes'
    SELECT 
        status 
        INTO
        v_status
    FROM books
    WHERE isbn = p_issued_book_isbn;

    IF v_status = 'yes' THEN

        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES
        (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        UPDATE books
            SET status = 'no'
        WHERE isbn = p_issued_book_isbn;

        RAISE NOTICE 'Book records added successfully for book isbn : %', p_issued_book_isbn;


    ELSE
        RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;
    END IF;
END;
$$

-- Testing The function
SELECT * FROM books;
-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no
SELECT * FROM issued_status;

CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'


/*
Task 20: Create Table As Select (CTAS)
Objective: Create a CTAS (Create Table As Select) query to identify overdue books and calculate fines.

Description: Write a CTAS query to create a new table that lists each member and the books they have 
issued but not returned within 30 days.
The table should include:
    The number of overdue books.
    The total fines, with each day's fine calculated at $0.50.
    The number of books issued by each member.
    The resulting table should show:
    Member ID
    Number of overdue books
    Total fines
*/
create table overdue_books_and_calculated_fine
as
select
	m.member_id,
	count(*) as Number_of_overdue_books,
	sum((current_date-ist.issued_date-30)*0.5) AS total_fines
from members as m
join issued_status as ist
on m.member_id = ist.issued_member_id
left join return_status as rst
on ist.issued_id = rst.issued_id
where rst.return_date is null -- Book not yet returned
and (current_date - ist.issued_date)>30 -- More than 30 days
group by 1;

select * from overdue_books_and_calculated_fine;

```

## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.
2. **Set Up the Database**: Execute the SQL scripts in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `analysis_queries.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

## Author - Nitesh  Kumar Sharma

This project showcases SQL skills essential for database management and analysis. 
Thank you for your interest in this project!
