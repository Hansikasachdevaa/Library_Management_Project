# Library_Management_Project
Project Overview
Project Title: Library Management System
Level: Intermediate
Database: library_db

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.


![library](https://github.com/user-attachments/assets/3426bf87-7ac9-46f4-b12e-2f6427bfee76)

Objectives
Set up the Library Management System Database: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
CRUD Operations: Perform Create, Read, Update, and Delete operations on the data.
CTAS (Create Table As Select): Utilize CTAS to create new tables based on query results.
Advanced SQL Queries: Develop complex queries to analyze and retrieve specific data.

Project Structure
1. Database Setup
<img width="1378" height="915" alt="DB scheme Library Project" src="https://github.com/user-attachments/assets/b183143e-3c05-4f6b-a839-280a4a12f319" />

Database Creation: Created a database named library_db.
Table Creation: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
-- creating database
CREATE DATABASE Library_project_2;

-- setting default database
USE Library_project_2;

-- creating branch table
CREATE TABLE branch (
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(55),
    contact_no VARCHAR(15)
);

alter table branch
modify contact_no  varchar(15);

-- creating employees table

DROP TABLE IF EXISTS employees;

CREATE TABLE employees (
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(20),
    position VARCHAR(10),
    salary INT,
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id)
        REFERENCES branch (branch_id)
);

-- creating books table
drop table books;
CREATE TABLE books (
    isbn VARCHAR(20) PRIMARY KEY,
    book_title VARCHAR(75),
    category VARCHAR(10),
    rental_price FLOAT,
    status VARCHAR(15),
    author VARCHAR(35),
    publisher VARCHAR(55)
);
alter table books
modify category varchar(20);

-- creating members table
drop table members;
CREATE TABLE members (
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(25),
    member_address VARCHAR(75),
    reg_date DATE
);

-- creating issued_status table

CREATE TABLE issued_status (
    issued_id VARCHAR(10) PRIMARY KEY,
    issued_member VARCHAR(10),
    issued_book VARCHAR(75),
    issued_date DATE,
    issued_book_isbn VARCHAR(20),
    issued_emp_id VARCHAR(10),
    FOREIGN KEY (issued_member)
        REFERENCES members (member_id),
    FOREIGN KEY (issued_book_isbn)
        REFERENCES books (isbn),
    FOREIGN KEY (issued_emp_id)
        REFERENCES employees (emp_id)
);

alter table issued_status
rename  column issued_member to issued_member_id;

-- creating return_status table
CREATE TABLE return_status (
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(10),
    return_book VARCHAR(75),
    return_date DATE,
    return_book_isbn VARCHAR(20),
    FOREIGN KEY (issued_id)
        REFERENCES issued_status (issued_id)
);

SELECT 
    TABLE_NAME,
    COLUMN_NAME,
    REFERENCED_TABLE_NAME,
    REFERENCED_COLUMN_NAME
FROM
    INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE
    REFERENCED_TABLE_NAME IS NOT NULL
        AND TABLE_SCHEMA = 'Library_project_2';
```

2. CRUD Operations
Create: Inserted sample records into the books table.
Read: Retrieved and displayed data from various tables.
Update: Updated records in the employees table.
Delete: Removed records from the members table as needed.

Task 1. Create a New Book Record -- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"
```sql
INSERT INTO BOOKS(isbn,book_title,category,rental_price,status,author,publisher)
values
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```

-- Task 2: Update an Existing Member's Address where member_id= 'C103'
```sql
update members 
set member_address = '125 Oak St'
where member_id='C103';
```

-- Task 3: Delete a Record from the Issued Status Table 
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.
```sql
delete  from issued_status 
where issued_id = 'IS121';
```
-- Task 4: Retrieve All Books Issued by a Specific Employee --
--  Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
select * from issued_status
 where issued_emp_id = 'E104';
 ```
 -- Task 5: List Members Who Have Issued More Than One Book
 ```sql
 select issued_member_id, count(issued_book) as total_issued_books
 from issued_status
 group by issued_member_id
 having total_issued_books>1;
```
 
 
-- Task 6: Create Summary Tables:
--  Use CTAS to generate new tables based on query results - each book and total book_issued_cnt
drop table book_issue_count;
```sql
create table book_issue_count as
select b.isbn,b.book_title, count(i.issued_id) as book_issue_count from
books b
join issued_status i
on b.isbn= i.issued_book_isbn
group by b.isbn, b.book_title;
select * from book_issue_count;
```

-- Task 7. Retrieve All Books in a Specific Category:
```sql
select *
from books where category='Classic';
```

-- Task 8: Find Total Rental Income by Category:
```sql
select b.category, sum(b.rental_price),
count(*) 
from issued_status as ist
join books b
ON b.isbn = ist.issued_book_isbn
GROUP BY b.category;
```

 -- task 9:List Members Who Registered in the Last 180 Days:
 ```sql
 select * from members where 
 datediff(now(),reg_date)=180;
```

--  task 10:List Employees with Their Branch Manager's Name and their branch details:
```sql
select e.emp_id, e.emp_name,
b.branch_id,
m.emp_name
from employees e
join branch b 
on e.branch_id = b.branch_id
join employees m
on b.manager_id= m.emp_id;
```

-- Task 11. Create a Table of Books with Rental Price Above a 7:
```sql
select * from books
where rental_price> 7;
```


-- Task 12: Retrieve the List of Books Not Yet Returned
```sql
select i.issued_book, i.issued_book_isbn
from return_status r
left join issued_status i 
on i.issued_book= r.issued_id
where r.issued_id is null;
```

-- Task 13: Identify Members with Overdue Books
-- Write a query to identify members who have overdue books (assume a 30-day return period). 
-- Display the member's_id, member's name, book title, issue date, and days overdue.
 -- Write a query to identify members who have overdue books (assume a 30-day return period).
 ```sql
WITH dataset_date AS (
    SELECT MAX(issued_date) AS ref_date
    FROM issued_status
)

SELECT 
    m.member_id,
    m.member_name,
    i.issued_book AS book_title,
    i.issued_date,
    DATEDIFF(d.ref_date, i.issued_date + INTERVAL 30 DAY) AS days_overdue
FROM issued_status i
LEFT JOIN return_status r 
    ON i.issued_id = r.issued_id
JOIN members m 
    ON m.member_id = i.issued_member_id
CROSS JOIN dataset_date d
WHERE r.issued_id IS NULL
AND i.issued_date + INTERVAL 30 DAY < d.ref_date;
```



-- Task 14: Update Book Status on Return
-- Write a query to 
-- update the status of books in the books table to "Yes"
-- when they are returned (based on entries in the return_status table).
```sql
update books b
join issued_status i
on b.isbn= i.issued_book_isbn
join return_status r
on i.issued_id = r.issued_id
set b.status='yes';
```



-- Task 15: Branch Performance Report
-- Create a query that generates a performance report for each branch-
-- showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.
```sql
Create table Branch_Reports 
as
select br.branch_id,
 count(i.issued_id) as Books_issued,
 count(r.return_id) as returned_books,
 sum(b.rental_price) as Revenue
from branch br
join employees e 
on br.branch_id= e.branch_id
join issued_status i 
on e.emp_id= i.issued_emp_id
join books b 
on b.isbn= i.issued_book_isbn
left join return_status r 
on i.issued_id = r.issued_id
group by br.branch_id;


select * from branch_reports;
```

-- Task 16: CTAS: Create a Table of Active Members
-- Use the CREATE TABLE AS (CTAS) statement to create a new table active_members-
-- containing members who have issued at least one book in the last 2 months.
```sql
create table  dataset_date AS 
    SELECT MAX(issued_date) AS ref_date
    FROM issued_status;

create table Active_Members 
as
select 
m.member_id,
m.member_name,
count(i.issued_book) as issued_books
from members m
join issued_status i
on m.member_id = i.issued_member_id 
cross join dataset_date dd
where timestampdiff(month,i.issued_date,dd.ref_date)<2
group by m.member_id, m.member_name
having count(i.issued_book) >=1;
select * from active_members;
```

-- Task 17: Find Employees with the Most Book Issues Processed
-- Write a query to find the top 3 employees- who have processed the most book issues.
-- Display the employee name, number of books processed, and their branch.
```sql
select e.branch_id, br.branch_address,br.contact_no,
e.emp_name,
count(i.issued_book) as issued_books
from employees e
join branch br
on e.branch_id= br.branch_id
join issued_status i
on e.emp_id= i.issued_emp_id 
group by e.branch_id, e.emp_name
order by issued_books desc
limit 3;
```

-- Task 18: Identify Members with Strong Category Preferences
-- Write a query to identify members who have issued books more than twice from the same category.
-- Display the member name, category, and number of books issued.
```sql
select m.member_name,
b.category,
count(i.issued_id) as books_issued
from members m
join issued_status i
on m.member_id= i.issued_member_id
join books b
on 
b.isbn= i.issued_book_isbn
group by m.member_name, b.category
having count(i.issued_id)> 2;
```



-- Task 19: Create Table As Select (CTAS) Objective:
-- Create a CTAS (Create Table As Select) query to identify member_id, their overdue books and calculate fines.
-- Each day fine-$0.50.
```sql
	create table overdue_report as 
	with due_date as
	 (
	            select issued_member_id,
				issued_id,
				issued_book_isbn,
				issued_date,
	            date_add(issued_date,interval 30 day) as due_date
	from issued_status
	 ),

	 overdue_day_count as 
	 (
	        select issued_id,
	        issued_date,
	        due_date,
	        datediff(ds.ref_date,due_date) as overdue_day_count,
            ds.ref_date
	 from due_date cross join dataset_date ds
	 )

	select 
	     od.issued_member_id,
		 m.member_name,
	     od.issued_id,
	     od.issued_book_isbn,
	     od.issued_date,
	     od.due_date,
	     odc.overdue_day_count,
	     overdue_day_count*0.50 as Fine_per_book,
		sum(overdue_day_count*0.50) over(partition by od.issued_member_id) as Total_member_fine
	from members m
	join due_date od on
	m.member_id = od.issued_member_id
	left join return_status r 
	on od.issued_id= r.issued_id
	join overdue_day_count odc
	on od.issued_id= odc.issued_id
	where odc.ref_date > odc.due_date and r.issued_id is null;

	select * from overdue_report ;
```

