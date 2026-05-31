# Library System Management SQL Project

## Project Overview

**Project Title**: Library System Management  
**Level**: Intermediate  
**Database**: `library_sys_management`.

Project Outcomes

Through the development of this system, I was able to build a fully functional relational database that supports the core operations of a library. The project covered database design, transaction management, reporting, and business analysis using SQL.

By working with multiple related tables and realistic business scenarios, I gained practical experience creating relationships between entities, writing complex queries, and transforming operational data into meaningful reports that support decision-making.

The analytical queries developed throughout the project provide visibility into book circulation, member activity, rental revenue, overdue books, and branch operations.

---

# Project Structure

## 1. Database Setup

### Database Creation

```sql
create database library_sys_management;
```

---

### Create Table `branch`

```sql
create table branch
(
 branch_id varchar(10) primary key,
 manager_id varchar(10),
 branch_address varchar(30),
 contact_no varchar(15)           
);
```

---

### Create Table `employees`

```sql
create table employees
(
 emp_id varchar(10) primary key,
 emp_name varchar(30),
 position varchar(30),
 salary decimal(10,2),
 branch_id varchar(10),
 foreign key (branch_id) references branch(branch_id)
);
```

---

### Create Table `members`

```sql
create table members
(
 member_id varchar(10) primary key,
 member_name varchar(30),
 member_address varchar(30),
 reg_date date
);
```

---

### Create Table `books`

```sql
create table books
(
 isbn varchar(50) primary key,
 book_title varchar(80),
 category varchar(30),
 rental_price decimal(10,2),
 status varchar(10),
 author varchar(30),
 publisher varchar(30)
);
```

---

### Create Table `issued_status`

```sql
create table issued_status
(
 issued_id varchar(10) primary key,
 issued_member_id varchar(30),
 issued_book_name varchar(80),
 issued_date date,
 issued_book_isbn varchar(50),
 issued_emp_id varchar(10),
 foreign key(issued_member_id) references members(member_id),
 foreign key (issued_emp_id) references employees(emp_id),
 foreign key (issued_book_isbn) references books(isbn) 
);
```

---

### Create Table `return_status`

```sql
create table return_status
(
 return_id varchar(10) PRIMARY KEY,
 issued_id varchar(30),
 return_book_name varchar(80),
 return_date date,
 return_book_isbn varchar(50),
 foreign key (return_book_isbn) references books(isbn)
);
```
## Entity Relationship Diagram (ERD)

![Library ERD](library_erd.png)

---

# 2. CRUD Operations

## Task 1: Create a New Book Record

```sql
insert into books(isbn, book_title, category, rental_price, status, author, publisher)
values
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```

---

## Task 2: Update an Existing Member's Address

```sql
update members
set member_address = '125 Main St'
where member_id = 'C101';
```

---

## Task 3: Delete Records from `issued_status`

```sql
delete from issued_status
where issued_id = 'IS107';
```

---

## Task 4: Retrieve All Books Issued by a Specific Employee

```sql
select * from issued_status
where issued_emp_id = 'E101';
```

---

## Task 5: List Members Who Have Issued More Than One Book

```sql
select 
     issued_emp_id, 
	 count(issued_id) as total_book_issued 
from issued_status 
group by issued_emp_id
having count (issued_id) > 1;
```

---

# 3. CTAS (Create Table As Select)

## Task 6: Create Summary Table for Issued Books

```sql
create table book_cnts
as
select 
  b.isbn,
  b.book_title,
  count (ist.issued_id) as no_issued
from books as b
join 
issued_status as ist 
on ist.issued_book_isbn = b.isbn
group by 1,2;
```

---

# 4. Data Analysis & Findings

## Task 7: Retrieve All Books in a Specific Category

```sql
select * from books 
where category = 'Classic';
```

---

## Task 8: Find Total Rental Income by Category

```sql
select
   b.category,
   sum(b.rental_price),
   count(*)
from books as b
join 
issued_status as ist 
on ist.issued_book_isbn = b.isbn
group by 1;
```

---

## Task 9: List Members Who Registered in the Last 180 Days

```sql
select * from members
where reg_date >= current_date - interval '180 days';
```

---

## Task 10: List Employees with Their Branch Manager and Branch Details

```sql
select 
     e1.*,
	 e2.emp_name as manager,
	 b.branch_id
 from employees as e1
 join 
 branch as b
 on e1.branch_id = b.branch_id
 join 
 employees as e2
 on b.manager_id = e2.emp_id;
```

---

## Task 11: Create a Table of Books with Rental Price Above 7

```sql
create table books_price_greater_than_7
as
select * from books
where rental_price > 7;
```

---

## Task 12: Retrieve Books Not Yet Returned

```sql
select 
  distinct ist.issued_book_name
from issued_status as ist 
left join 
return_status as rs
on 
ist.issued_id = rs.issued_id
where rs.return_id is null;
```

---

## Task 13: Identify Members with Overdue Books

### Objective

Identify members who have overdue books assuming a 30-day return period.

### Query

```sql
select
   ist.issued_member_id,
   m.member_name,
   bk.book_title,
   ist.issued_date,
   current_date - ist.issued_date as over_dues
from issued_status as ist
join 
members as m 
on m.member_id = ist.issued_member_id
join 
books as bk
on 
bk.isbn = ist.issued_book_isbn
left join 
return_status as rs 
on 
rs.issued_id = ist.issued_id
where 
    rs.return_date is null
	and
	(current_date - ist.issued_date) > 30
order by 1;
```

---

# Conclusion

This project provided hands-on experience in designing and managing a relational database from the ground up. Beyond creating tables and relationships, the focus was on using SQL to solve operational problems and answer business questions.

Working through real-world scenarios such as tracking issued books, identifying overdue returns, measuring rental performance, and generating summary reports strengthened my understanding of relational database design, data modeling, SQL querying, joins, aggregations, CTAS operations, and business reporting.

The project demonstrates how SQL can be used not only to store data, but also to generate actionable insights that support day-to-day operations and decision-making.
