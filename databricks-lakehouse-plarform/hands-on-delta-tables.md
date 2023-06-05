# Hands-On Delta Tables

## ![](<../.gitbook/assets/image (3).png>)Creating our first Delta Table

Delta is the default file and table format using Databricks.

```sql
-- Creating our first Delta table
CREATE TABLE IF NOT EXISTS employees
(id INT, name STRING, salary DOUBLE);
```

```sql
-- Let's load some data in this table
-- Inserting 10 sample records into the "employees" table
INSERT INTO employees 
VALUES
(1, 'John', 5000.00),
(2, 'Jane', 6000.00),
(3, 'Mike', 5500.00),
(4, 'Emily', 4500.00),
(5, 'Michael', 7000.00),
(6, 'Sarah', 5200.00),
(7, 'David', 4800.00),
(8, 'Jennifer', 5800.00),
(9, 'Robert', 5100.00),
(10, 'Jessica', 5300.00);


```

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-05 12-46-58.png" alt=""><figcaption></figcaption></figure>

```sql
SELECT * FROM employees;
```

![](<../.gitbook/assets/image (1).png>)

#### Let's list down files underline this table

We will use `fs` a magic command through which we can interact with Databricks files system, the `ls` command will list down all the files for the employees. We can see below in the screenshot the data is stored under the Parquet file and a \_delta\_log folder is created which contains the transaction log.

```
%fs ls dbfs:/user/hive/warehouse/employees
```

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-05 12-51-10.png" alt=""><figcaption></figcaption></figure>

#### Updating Table

```sql
UPDATE employees
SET salary = salary + 100
WHERE name LIKE "J%"
```

```sql
SELECT * FROM employees;
```

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-05 12-59-44.png" alt=""><figcaption></figcaption></figure>

In the previous chapter, we saw during the [update](what-is-delta-lake.md#update) the writer process creates a copy of the file and applies the necessary updates to the new file. If we now see a list of the files of the employees table we will have two files.

```
%fs ls dbfs:/user/hive/warehouse/employees
```

<figure><img src="../.gitbook/assets/Screenshot from 2023-06-05 13-04-05.png" alt=""><figcaption></figcaption></figure>

