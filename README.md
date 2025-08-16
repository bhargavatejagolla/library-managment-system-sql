# 📚 Library Management System (SQL Project)

## 📖 Project Overview
The **Library Management System (LMS)** is a database-driven project built using **SQL** to demonstrate essential database management concepts such as:

- Database design & normalization  
- CRUD operations (Create, Read, Update, Delete)  
- SQL Joins and Constraints  
- Grouping and Aggregate functions  
- Subqueries & Views  
- **CTAS** (Create Table As Select) and advanced queries  

This project helps manage **books, members, employees, library branches, issue & return records**, and generates useful analytical insights for real-world library operations.

---

## ⚙️ Tech Stack
- **Database:** MySQL  
- **Language:** SQL  
- **Concepts Used:**  
  - **DDL (Data Definition Language)** – CREATE, ALTER, DROP  
  - **DML (Data Manipulation Language)** – INSERT, UPDATE, DELETE  
  - **Joins (INNER, LEFT, RIGHT, FULL)**  
  - **Constraints** (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, DEFAULT, NOT NULL)  
  - **Grouping & Aggregation** (GROUP BY, HAVING)  
  - **Subqueries & Nested Queries**  
  - **Views** for simplifying complex queries  
  - **CTAS** for table creation from existing queries  

---
## 🏗️ Database Schema

The system consists of the following tables:

### 1️⃣ BRANCH
- Stores details of library branches  
- Example fields: `Branch_ID (PK)`, `Branch_Name`, `Location`

### 2️⃣ EMPLOYEE
- Stores details of employees and their positions  
- Example fields: `Emp_ID (PK)`, `Emp_Name`, `Position`, `Branch_ID (FK)`

### 3️⃣ BOOKS
- Maintains catalog of books, categories, prices, and availability status  
- Example fields: `Book_ID (PK)`, `Title`, `Author`, `Category`, `Price`, `Status`

### 4️⃣ MEMBERS
- Contains registered library members  
- Example fields: `Member_ID (PK)`, `Name`, `Address`, `Join_Date`, `Contact`

### 5️⃣ ISSUED_STATUS
- Tracks book issue records  
- Example fields: `Issue_ID (PK)`, `Book_ID (FK)`, `Member_ID (FK)`, `Issue_Date`, `Due_Date`

### 6️⃣ RETURN_STATUS
- Tracks returned book details  
- Example fields: `Return_ID (PK)`, `Issue_ID (FK)`, `Return_Date`, `Fine_Amount`
---
### 1. Creating Database & Tables
```sql
use library;

CREATE TABLE BRANCH(
    branch_id VARCHAR(10)  PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(55),
    contact_no VARCHAR(35)
);

CREATE TABLE EMPLOYEE(
    emp_id VARCHAR(10) PRIMARY key,
    emp_name VARCHAR(25),
    position VARCHAR(25),
    salary FLOAT,
    branch_id VARCHAR(25)
);

CREATE TABLE BOOKS(
    isbn VARCHAR(25) PRIMARY key,
    book_title varchar(80),
    category varchar(25),	
    rental_price FLOAT,
    status varchar(15),
    author varchar(35),
    publisher varchar(50)
);

CREATE TABLE MEMBERS(
    member_id VARCHAR(15) PRIMARY key,
    member_name VARCHAR(20),
    member_address VARCHAR(20),
    reg_date DATE
);

CREATE TABLE ISSUED_STATUS(
    issued_id VARCHAR(15) PRIMARY key,
    issued_member_id VARCHAR(20),
    issued_book_name VARCHAR(70),
    issued_date DATE,
    issued_book_isbn VARCHAR(25),
    issued_emp_id VARCHAR(15)
);

CREATE TABLE RETURN_STATUS(
    return_id VARCHAR(15) PRIMARY KEY,
    issued_id VARCHAR(35),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(20)
);
```
### Library Management System - Database Design

## ✅ Explanation

- All entities such as **Branches**, **Employees**, **Books**, **Members**, **Issues**, and **Returns** are created as **normalized tables**.  
- **Primary Keys** are used to uniquely identify each record in their respective tables.  
- **Foreign Keys** are used to ensure **referential integrity** between related tables.
----
### 2. Adding Relationships (Foreign Keys):
 ```sql
ALTER TABLE issued_status
ADD CONSTRAINT fk_members FOREIGN KEY(issued_member_id) REFERENCES MEMBERS(member_id);

ALTER TABLE issued_status
ADD CONSTRAINT fk_books FOREIGN KEY(issued_book_isbn) REFERENCES books(isbn);

ALTER TABLE issued_status
ADD CONSTRAINT fk_employees FOREIGN KEY(issued_emp_id) REFERENCES EMPLOYEE(emp_id);

ALTER TABLE EMPLOYEE
ADD CONSTRAINT fk_branch FOREIGN KEY(branch_id) REFERENCES branch(branch_id);

ALTER TABLE RETURN_STATUS
ADD CONSTRAINT fk_issued_status FOREIGN KEY(issued_id) REFERENCES issued_status(issued_id);
```
## ✅ Explanation

- Maintains **data integrity** between **Issued Books**, **Members**, **Employees**, and **Returns**.  
- Avoids **duplication** and ensures **consistent data** throughout the database.
----
### 3. Inserting Sample Records:
```sql
INSERT INTO members(member_id, member_name, member_address, reg_date) 
VALUES
('C101', 'Alice Johnson', '123 Main St', '2021-05-15'),
('C102', 'Bob Smith', '456 Elm St', '2021-06-20');
```
## ✅ Explanation
-----
- Sample records were added for **Members**, **Branches**, **Employees**, **Books**, **Issues**, and **Returns** to test queries.  
### 4. CRUD Operations:
```sql
-- Create
INSERT INTO books (isbn, book_title, category, rental_price, status, author, publisher)
VALUES ('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');

-- Update
UPDATE members SET member_address = '125 Oak St' WHERE member_id = 'C103';

-- Delete
DELETE FROM issued_status WHERE issued_id='IS121';

-- Read
SELECT * FROM issued_status WHERE issued_emp_id='E101';
```
## ✅ Explanation

- **Create →** Added new books.  
- **Read →** Queried issued books by employee.  
- **Update →** Updated member details.  
- **Delete →** Removed unwanted issue records.
----
### 5. Data Analysis Queries:

**a) Members who issued more than one book**  
- This query identifies members who have issued more than one book.  
- It helps in analyzing borrowing patterns and tracking frequent readers.
  ```sql
  SELECT issued_member_id, COUNT(*)
  FROM issued_status
  GROUP BY 1
  HAVING COUNT(*) > 1;
  ```
**b) Rental income by category**  
- This query calculates the **total rental income** generated from books, grouped by their **category**.  
- Useful for analyzing which categories contribute the most revenue.
  ```sql
    SELECT b.category, SUM(b.rental_price) AS total_rental_price
    FROM books AS b
    JOIN issued_status AS ist ON ist.issued_book_isbn = b.isbn
    GROUP BY 1;
  ```
**c) Members with overdue books (>30 days)**  
- This query identifies **members who have not returned books within 30 days** of issue.  
- Helps in tracking **defaulters** and sending reminders or penalties.  
  ```sql
    SELECT ist.issued_member_id, m.member_name, bk.book_title, ist.issued_date,
    CURRENT_DATE - ist.issued_date AS over_dues
    FROM issued_status AS ist
    JOIN members AS m ON m.member_id = ist.issued_member_id
    JOIN books AS bk ON bk.isbn = ist.issued_book_isbn
    LEFT JOIN return_status AS rs ON rs.issued_id = ist.issued_id
    WHERE rs.return_id IS NULL
    AND (CURRENT_DATE - ist.issued_date) > 30;
  ```
## 🚀 Applications

- Library book issue/return management.  
- Employee & branch tracking.  
- Fine calculation for overdue books.  
- Report generation for management (e.g., income by category).  
- Useful for data analysts in publishing, rental, and education.  

## 🔮 Future Enhancements

- Build a frontend application (Flask/Django/React).  
- Add stored procedures & triggers (auto-update book status on return).  
- Implement fine calculation logic.  
- Introduce user authentication & roles (Admin, Librarian, Member).  
- Use views and indexing for faster queries.  
- Extend to a cloud-based DB with dashboards (Tableau/Power BI).
---
## 👨‍💻 Author

*Golla Bhargava TEja*  

- 🌐 [GitHub](https://github.com/bhargavatejagolla)  
- 💼 [LinkedIn](https://www.linkedin.com/in/golla-bhargava-teja/)  






