# 📚 Lecture Notes: Databases & SQL Fundamentals

---

## 1. Why SQL is Important

* **SQL (Structured Query Language)** is the language for interacting with databases.
* Almost every **data analyst, data scientist, or backend developer** needs SQL.
* Companies prioritize hiring people with strong SQL skills.

⚡ **Key point:** Even with tools like ChatGPT or GUI software, **you need SQL knowledge** to understand and optimize queries.

---

## 2. Data and Its Importance

* **“Data is the new oil.”**
* Wealth and power today come from **owning, analyzing, and leveraging data**.
* Historical revolutions in business:

  1. **Computers** – digitization of records.
  2. **Internet** – global connectivity, e-commerce boom (e.g., Flipkart vs. Big Bazaar).
  3. **AI Revolution** – relies heavily on massive datasets.

💡 **Example:** Google and Facebook don’t charge for services, because they monetize user data.

---

## 3. What is a Database?

* A **database is software** that stores, organizes, and manages data.
* It allows:

  * **Storage** → keep data in structured form.
  * **Retrieval** → fetch whenever required.
  * **Manipulation** → update, delete, or add records.

🛠️ **Key Uses:**

1. **Data Storage** – keep transaction or user data.
2. **Data Analysis** – generate insights for business.
3. **Record Keeping** – track customers, inventory, payments.
4. **Applications** – power login, registration, search, recommendations.

---

## 4. CRUD Operations

All applications boil down to four database operations (**CRUD**):

* **C**reate → Insert new data.
* **R**ead → Fetch existing data.
* **U**pdate → Modify existing records.
* **D**elete → Remove records.

💡 **Examples:**

* Register on Instagram → Create.
* Login verification → Read.
* Change address in Swiggy → Update.
* Delete an account → Delete.

---

## 5. Properties of an Ideal Database

1. **Integrity** → Data must be accurate & consistent.

   * e.g., Weight cannot be negative.
2. **Availability** → 24/7 uptime, zero downtime.
3. **Security** → Protect sensitive data (credit cards, health info).
4. **Application Independence** → Same database works for web, mobile, iOS apps.
5. **Concurrency** → Multiple users can access simultaneously.

---

## 6. Types of Databases

### (a) Relational Databases (RDBMS)

* Store data in **tables (rows & columns)**.
* Suitable for **structured data**.
* Examples: MySQL, PostgreSQL, Oracle, SQL Server.

🔹 **SQL Example:**

```sql
CREATE TABLE Students (
    student_id INT PRIMARY KEY,
    name VARCHAR(50),
    branch VARCHAR(50),
    cgpa DECIMAL(3,2)
);
```

---

### (b) NoSQL Databases

* For **unstructured or semi-structured data** (documents, images, JSON).
* Flexible schema.
* Example: MongoDB.

---

### (c) Columnar Databases

* Store data **by columns**, not rows.
* Great for **analytics & aggregations** (fast AVG, SUM).
* Examples: Amazon Redshift, Google BigQuery.

---

### (d) Graph Databases

* Store **networks of relationships**.
* Useful in social networks, recommendation systems.
* Examples: Neo4j, Amazon Neptune.

---

### (e) Key-Value Databases

* Store simple **key–value pairs**.
* Fast retrieval, used for caching.
* Example: Redis.

---

## 7. RDBMS Terminology

* **Relation** → Table.
* **Attribute** → Column.
* **Tuple** → Row.
* **Cardinality** → Number of rows.
* **Degree** → Number of columns.
* **Domain** → Allowed values in a column.
* **NULL** → Missing or unknown value.

---

## 8. DBMS (Database Management System)

* Software layer between **users/applications** and the **database**.
* Examples: MySQL Workbench, phpMyAdmin.

### Functions:

1. Data storage & retrieval.
2. Enforce integrity & transactions.
3. Handle concurrency (multiple users).
4. Security & user management.
5. Utilities like backup, restore, import/export.

---

## 9. Database Keys

Keys uniquely identify rows and maintain relationships.

1. **Super Key** → Any column(s) that uniquely identify a row.

   * Example: `{RollNo}`, `{Email}`, `{RollNo, Name}`.

2. **Candidate Key** → Minimal super key (no redundancy).

   * Example: `{RollNo}`, `{Email}`.

3. **Primary Key** → The chosen candidate key.

   * Must be **unique, non-null, stable**.

```sql
CREATE TABLE Students (
    roll_no INT PRIMARY KEY,
    name VARCHAR(50),
    email VARCHAR(50) UNIQUE
);
```

4. **Alternate Key** → Candidate keys not chosen as primary.

5. **Composite Key** → Primary key made of multiple columns.

   * Example: `{student_id, course_id}` in Enrollment table.

6. **Surrogate Key** → Artificial key (auto-increment ID).

7. **Foreign Key** → Refers to primary key of another table.

```sql
CREATE TABLE Enrollment (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES Students(roll_no),
    FOREIGN KEY (course_id) REFERENCES Courses(course_id)
);
```

---

## 10. Relationships & Cardinality

* Defines how tables/entities are related.

### (a) One-to-One (1:1)

Example: Person ↔ Driving License

### (b) One-to-Many (1\:N)

Example: Branch ↔ Students

* One branch has many students.

### (c) Many-to-Many (M\:N)

Example: Students ↔ Courses

* Students can enroll in multiple courses.
* Requires a **junction table** (Enrollment).

---

## 11. Drawbacks of Databases

1. **Complexity** – setup & management can be hard.
2. **Cost** – hardware, software, skilled admins are expensive.
3. **Performance Issues** – large datasets slow queries.
4. **Data Migration** – moving between systems is error-prone.
5. **Security Risks** – SQL injection, hacks.
6. **Rigidity** – schema is inflexible; changes are costly.

---

## 12. Tools for Practice

* **XAMPP** (phpMyAdmin) – beginner-friendly GUI.
* **MySQL Workbench** – professional tool.
* **PostgreSQL** – advanced RDBMS.

---

## 13. Practical SQL Examples

### Create a Table

```sql
CREATE TABLE Employees (
    emp_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2)
);
```

### Insert Data

```sql
INSERT INTO Employees (name, department, salary)
VALUES ('Rohit', 'Finance', 55000),
       ('Ayush', 'IT', 70000);
```

### Read Data

```sql
SELECT * FROM Employees WHERE department = 'IT';
```

### Update Data

```sql
UPDATE Employees
SET salary = 75000
WHERE name = 'Ayush';
```

### Delete Data

```sql
DELETE FROM Employees WHERE emp_id = 1;
```

---

# ✅ Summary

* Data is crucial → stored in **databases** → accessed via **SQL**.
* **CRUD** is the backbone of all applications.
* Types of databases: Relational, NoSQL, Columnar, Graph, Key-Value.
* RDBMS concepts: Relation, Attribute, Tuple, Keys, Relationships.
* DBMS ensures data management, integrity, concurrency, and security.
* SQL practice is essential for mastering data analysis.
