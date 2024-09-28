# SQL Subqueries - Lab

## Introduction

Now that you've seen how subqueries work, it's time to get some practice writing them! Not all of the queries will require subqueries, but all will be a bit more complex and require some thought and review about aggregates, grouping, ordering, filtering, joins and subqueries. Good luck!  

## Objectives

You will be able to:

* Write subqueries to decompose complex queries

## CRM Database ERD

Once again, here's the schema for the CRM database you'll continue to practice with.

<img src="https://curriculum-content.s3.amazonaws.com/data-science/images/Database-Schema.png" width="600">

## Connect to the Database

As usual, start by importing the necessary packages and connecting to the database `data.sqlite`.


```python
# Your code here; import the necessary packages
import sqlite3
import pandas as pd
```


```python
# Your code here; create the connection
conn = sqlite3.Connection('data.sqlite')
```

## Write an Equivalent Query using a Subquery

The following query works using a `JOIN`. Rewrite it so that it uses a subquery instead.

```
SELECT
    customerNumber,
    contactLastName,
    contactFirstName
FROM customers
JOIN orders 
    USING(customerNumber)
WHERE orderDate = '2003-01-31'
;
```


```python
# Your code here

Query_eqv_subquery=("""
SELECT
    customerNumber,
    contactLastName,
    contactFirstName
FROM customers 
       WHERE customerNumber IN
       (SELECT customerNumber
       FROM orders 
       WHERE orderDate = '2003-01-31')
;
""")
pd.read_sql(Query_eqv_subquery,conn)
```

## Select the Total Number of Orders for Each Product Name

Sort the results by the total number of items sold for that product.


```python
# Your code here

Query_total_orders_per_product_name=("""
                                     SELECT  (SELECT PR.productName 
                                     FROM products PR 
                                     WHERE PR.productCode = ORD.productCode) AS productName,SUM(quantityOrdered) Total_quantity_ordered
                                     FROM orderdetails AS ORD
                                     WHERE ORD.productCode IN (SELECT productCode
                                                              FROM products PR)
                                     GROUP BY ORD.productCode
                                     ORDER BY Total_quantity_ordered DESC
                                     ;
                                     """)
pd.read_sql(Query_total_orders_per_product_name,conn)
```

## Select the Product Name and the  Total Number of People Who Have Ordered Each Product

Sort the results in descending order.

### A quick note on the SQL  `SELECT DISTINCT` statement:

The `SELECT DISTINCT` statement is used to return only distinct values in the specified column. In other words, it removes the duplicate values in the column from the result set.

Inside a table, a column often contains many duplicate values; and sometimes you only want to list the unique values. If you apply the `DISTINCT` clause to a column that has `NULL`, the `DISTINCT` clause will keep only one NULL and eliminates the other. In other words, the DISTINCT clause treats all `NULL` “values” as the same value.


```python
# Your code here
# Hint: because one of the tables we'll be joining has duplicate customer numbers, you should use DISTINCT
Query_products_total_no_ppe_ordered_each_prd=("""SELECT PR.productName, (SELECT COUNT(DISTINCT C.customerNumber)
            FROM customers AS C
            JOIN orders AS ORS
            USING(customerNumber)
            JOIN orderdetails AS ORD
            USING (orderNumber)
            WHERE PR.productCode=ORD.productCode) AS Total_Number_of_People
            FROM products AS PR 
            JOIN orderdetails AS ORD
            USING (productCode)
            GROUP BY PR.productCode
            ORDER BY Total_Number_of_People DESC
;""")
pd.read_sql(Query_products_total_no_ppe_ordered_each_prd,conn)
```

## Select the Employee Number, First Name, Last Name, City (of the office), and Office Code of the Employees Who Sold Products That Have Been Ordered by Fewer Than 20 people.

This problem is a bit tougher. To start, think about how you might break the problem up. Be sure that your results only list each employee once.


```python
# Your code here
Query_employees_sold_prds_ordered_by_less_than_20_customers=("""
SELECT E.employeeNumber, E.firstName,E.lastName, O.city,E.officeCode
            FROM employees AS E
            JOIN offices O
            USING (officeCode)
            JOIN customers C
            ON C.salesRepEmployeeNumber=E.employeeNumber
            JOIN orders AS ORS
            USING (customerNumber)
            JOIN orderdetails ORD
            USING (orderNumber)
            JOIN products PR
            USING (productCode)
            WHERE PR.productCode IN 
            (SELECT ORD.productCode 
            FROM orderdetails AS ORD
            JOIN orders AS ORS
            USING (orderNumber)
            JOIN customers C
            USING (customerNumber)
            GROUP BY ORD.productCode
            HAVING COUNT(DISTINCT customerNumber)<20)
            GROUP BY E.employeeNumber, E.firstName,E.lastName, O.city,E.officeCode
            ORDER BY E.employeeNumber           
;""")
pd.read_sql(Query_employees_sold_prds_ordered_by_less_than_20_customers,conn)
```

## Select the Employee Number, First Name, Last Name, and Number of Customers for Employees Whose Customers Have an Average Credit Limit Over 15K


```python
# Your code here
Query_No_of_Customers_AVG_Creditlimi_over15K=("""
       SELECT E.employeeNumber,E.firstName,E.LastName,COUNT(C.customerNumber) AS Number_of_customers
       FROM employees AS E
       JOIN customers AS C
       ON C.salesRepEmployeeNumber=E.employeeNumber
       WHERE C.customerNumber IN (
       SELECT C.customerNumber
       FROM customers C
       WHERE C.creditLimit>15000)
       GROUP BY E.employeeNumber
       ORDER BY Number_of_customers DESC

;""")
pd.read_sql(Query_No_of_Customers_AVG_Creditlimi_over15K,conn)

```

## Summary

In this lesson, you got to practice some more complex SQL queries, some of which required subqueries. There's still plenty more SQL to be had though; hope you've been enjoying some of these puzzles!
# SQL_Subqueries_Lab
