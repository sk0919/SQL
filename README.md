# MYSQL

```sql
-- Creating the employee table
CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(255),
    salary INT,
    manager_id INT
);

-- Inserting data into the employee table
INSERT INTO employee (emp_id, emp_name, salary, manager_id) VALUES 
(1, 'Ankit', 10000, 4),
(2, 'Mohit', 15000, 5),
(3, 'Vikas', 10000, 4),
(4, 'Rohit', 5000, 2),
(5, 'Mudit', 12000, 6),
(6,'Agam',12000 ,2 ),
(7,'Sanjay',9000 ,2 ),
(8,'Ashish',5000 ,2 );

-- Q. Query to find employees with a salary higher than their manager's salary
SELECT e1.emp_name AS EmployeeName,
       e1.salary AS EmployeeSalary,
       e2.emp_name AS ManagerName,
       e2.salary AS ManagerSalary 
FROM employee e1 
JOIN employee e2 ON e1.manager_id = e2.emp_id 
WHERE e1.salary > e2.salary;
```



### Difference between the below two queries ###
The first query is likely to be more optimized and faster. Here's why:

1. **Join vs Subquery**: The first query uses a **join** operation, which is generally faster than the **subquery** used in the second query. The subquery in the second query is executed once for each row processed by the outer query. This can be slow if the table has many rows.

2. **Number of Scans**: The first query scans the `employee` table twice, once for `e1` and once for `e2`. However, the second query potentially scans the `employee` table multiple times due to the subqueries, which could slow down the performance.

3. **Data Retrieval**: The first query retrieves all the required data in a single pass, whereas the second query needs to retrieve data multiple times due to the subqueries.

However, the actual performance can depend on various factors such as the database management system, the number of records in the table, the distribution of data, the use of indexes, and the database's query optimizer. It's always a good idea to test the performance of different queries on your specific system to determine which is most efficient. 

Remember, the key to writing efficient SQL queries is understanding your data and how your database handles queries. Always try to minimize the amount of data that your query needs to process. Use joins where appropriate, limit the use of subqueries, and take advantage of indexes. And most importantly, measure and monitor the performance of your queries.

- MySQL:
```sql
-- Query to find employees with a salary higher than their manager's salary
SELECT e1.emp_name AS employee_name,  
       e2.emp_name AS manager 
FROM employee e1, employee e2 
WHERE e1.manager_id = e2.emp_id 
AND e1.salary > e2.salary;

-- Query to find employees with a salary higher than their manager's salary
SELECT e1.emp_name AS EmployeeName,
       e1.salary AS EmployeeSalary,
       (SELECT e2.emp_name FROM employee e2 WHERE e1.manager_id = e2.emp_id) AS ManagerName,
       (SELECT e2.salary FROM employee e2 WHERE e1.manager_id = e2.emp_id) AS ManagerSalary 
FROM employee e1 
WHERE e1.salary > (SELECT e2.salary FROM employee e2 WHERE e1.manager_id = e2.emp_id);
```



### Q. To select the employees with the second highest salary: ###

  - PostgreSQL (pgSQL):
  ```sql
  SELECT emp_name, salary
  FROM employee
  WHERE salary = (
      SELECT DISTINCT salary
      FROM employee
      ORDER BY salary DESC
      LIMIT 1 OFFSET 1
  );
  ```

  - MySQL:
  ```sql
  SELECT emp_name, salary
  FROM employee
  WHERE salary = (
      SELECT DISTINCT salary
      FROM employee
      ORDER BY salary DESC
      LIMIT 1, 1
  );

  -- or another appraoch 
  SELECT * FROM (SELECT DISTINCT salary FROM employee ORDER BY salary DESC LIMIT 2 ) AS result ORDER BY salary LIMIT 1;
  ```

### Q. To select the employees with the nth highest salary, you can replace `n` with the desired rank: ###

  - PostgreSQL (pgSQL):
  ```sql
  SELECT emp_name, salary
  FROM employee
  WHERE salary = (
      SELECT DISTINCT salary
      FROM employee
      ORDER BY salary DESC
      LIMIT 1 OFFSET n-1
  );
  ```

  - MySQL:
  ```sql
  SELECT emp_name, salary
  FROM employee
  WHERE salary = (
      SELECT DISTINCT salary
      FROM employee
      ORDER BY salary DESC
      LIMIT n-1, 1
  );
  ```

In these queries, replace `n` with the rank of the salary you're interested in. For example, for the third highest salary, replace `n` with `3`.


## Q. Delete the duplicate rows without using inbuilt functions ##
Yes, you can delete duplicate rows in PostgreSQL (pgSQL) and MySQL without using inbuilt functions. However, it's important to note that this process involves creating a new table, copying distinct rows into it, and then renaming it to the original table name. Here's how you can do it:

1. PostgreSQL (pgSQL):

```sql
-- Create a new table and copy distinct rows into it
CREATE TABLE employee_temp AS
SELECT DISTINCT * FROM employee;

-- Drop the original table
DROP TABLE employee;

-- Rename the new table to the original table name
ALTER TABLE employee_temp RENAME TO employee;
```

2. MySQL:

```sql
-- Create a new table and copy distinct rows into it
CREATE TABLE employee_temp AS
SELECT DISTINCT * FROM employee;

-- Drop the original table
DROP TABLE employee;

-- Rename the new table to the original table name
RENAME TABLE employee_temp TO employee;
```


In these queries, replace `employee` with your actual table name. Please note that this method will remove all duplicate rows, not just extra occurrences. Also, be aware that this will permanently delete the original table, so make sure to backup any important data before proceeding.

# #

## Q. Delete the duplicate rows without using inbuilt functions (ALTERNATIVES) ##
Yes, there is an alternative way to delete duplicate rows without using inbuilt functions or creating a new table. However, it requires adding an additional unique ID column to your table. Here's how you can do it:

1. PostgreSQL (pgSQL):

```sql
-- Add a unique ID column
ALTER TABLE employee ADD COLUMN id SERIAL PRIMARY KEY;

-- Delete duplicate rows
DELETE FROM employee
WHERE id NOT IN (
    SELECT MIN(id)
    FROM employee
    GROUP BY emp_id, emp_name, salary, manager_id
);

-- Remove the unique ID column
ALTER TABLE employee DROP COLUMN id;
```

2. MySQL:

```sql
-- Add a unique ID column
ALTER TABLE employee ADD COLUMN id INT AUTO_INCREMENT PRIMARY KEY;

-- Delete duplicate rows
DELETE e1 FROM employee e1
INNER JOIN employee e2 
WHERE e1.id > e2.id AND e1.emp_id = e2.emp_id AND e1.emp_name = e2.emp_name AND e1.salary = e2.salary AND e1.manager_id = e2.manager_id;

-- Remove the unique ID column
ALTER TABLE employee DROP COLUMN id;
```

In these queries, replace `employee` with your actual table name. Please note that this method will remove all duplicate rows, not just extra occurrences. Also, be aware that this will permanently delete the original table, so make sure to backup any important data before proceeding.



