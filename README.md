# Ecommerce-Store
Design and Analysis Ecommerce Store to help customer search and purchase products.

# 🛒 E-Commerce Database Schema

This project contains a complete PostgreSQL database schema for a basic E-Commerce application.  
It includes the core **Entites** needed for managing:

- Categories  
- Products  
- Customers  
- Orders
- Order details

## Relationship between Entites
![Relationship](ER-Diagrams/relationships.svg)

## Database script 


 #### Categories Table
```sql

CREATE TABLE Categories (
    category_id INTEGER  PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL
);
```

 #### PRODUCTS Table
```sql
CREATE TABLE PRODUCTS (
    product_id INTEGER PRIMARY KEY,
    category_id INTEGER NOT NULL,
    name VARCHAR(50) NOT NULL,
    description Text NOT NULL,
    price decimal(7,3) NOT NULL Check(price >= 0),
    stock_quantity INTEGER  NOT NULL Check( stock_quantity >= 0),

    FOREIGN KEY (category_id)  REFERENCES Categories(category_id) ON DELETE CASCADE
);
```
## Customers Table
```sql
CREATE TABLE CUSTOMERS (
    customer_id INTEGER PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(50) NOT NULL,
    password VARCHAR(50) NOT NULL
);
```
## ORDERS Table
```sql
CREATE TABLE ORDERS (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date timestamp,
    total_amount decimal(7,3) NOT NULL Check(total_amount >= 0),

    FOREIGN KEY (customer_id)  REFERENCES CUSTOMERS(cutomer_id) ON DELETE CASCADE
);
```
## ORDER_DETAILS Table
```sql
CREATE TABLE ORDER_DETAILS (
    order_details_id INTEGER PRIMARY KEY,
    order_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL,
    unit_price decimal(7,3) NOT NULL Check(unit_price >= 0),

    FOREIGN KEY (order_id)  REFERENCES  ORDERS(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id)  REFERENCES  PRODUCTS(product_id) ON DELETE CASCADE
);
```

## ERD Diagram
![ERD diagram](ER-Diagrams/ecommerce-ERD.svg)

## 📊 Reports Query
**1- SQL query to generate a daily report of the total revenue for a specific date.**
```sql
select (od.quantity * od.unit_price) as Revenu ,ord.order_date
from orders ord join order_details od
ON or.order_id = od.order_id
where ord.order_date = $1
GROUP BY ord.order_date;

```

**2- SQL query to generate a monthly report of the top-selling products in a given month.**
```sql
select prd.name , SUM(prd.unit_price * od.quantity) as times_sold
from order_details od
JOIN products prd ON prd.product_id = od.product_id
JOIN orders ord ON ord.order_id = od.order_id 
where DTAE_TRUNC('month',order_date) = $1
GROUP BY prd.product_name
ORDER BY times_sold DESC;

```

**3- Write a SQL query to retrieve a list of customers who have placed orders totaling more than $500 in the past month.
Include customer names and their total order amounts.**
```sql
select c.first_name || ' ' || c.last_name as customer_name , SUM(ord.total_amount) as totoal_order_amount 
from customers c JOIN orders ord
where DTAE_TRUNC('month',order_date) = DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month' )
GROUP BY  c.first_name, c.last_name
HAVING SUM(ord.total_amount) > 500
ORDER BY total_order_amount DESC;

```

## Applying Denormalization mechanism on customer and order entities.

**create new table customer_order_preview with the following attributes(columns)**

| customer_id | first_name | last_name | email    | order_id | order_date | total_amount |
|-------------|-----------|-----------|-----------|----------|------------|--------------|

**SQL Query**
```sql
CREATE TABLE customer_order_preview AS
SELECT
    c.customer_id,
    c.first_name,
    c.last_name,
    c.email,
    ord.order_id,
    ord.order_date,
    ord.total_amount
FROM customers c
JOIN orders ord ON c.customer_id = ord.customer_id;
```
## Apply Queries for searching
**SQL query to search for all products with the word "camera" in either the product name or description.**
```sql
SELECT * FROM PRODUCTS
WHERE PRODUCT_NAME LIKE '%camera%' OR PRODUCT_DESCIPTION LIKE '%camera%'
```
**Problem**
- Full scan of the table.
- prevent the use of Index.
   
**Optimize search solution by using FULL Text Search**
```sql
CREATE FULLTEXT INDEX idx_text_serach
ON PRODUCT(prodName,prodDescr,prodLongDescr);

SELECT * FROM PRODUCT
WHERE MATCH(prodName,prodDescr,prodLongDescr)
AGAINST ('camera');
```

**Write a trigger to Create a sale history [Above customer , product], when a new order is made in the "Orders" table, automatically generates a sale history record for that order,capturing details such as the order date, customer, product, total amount, and quantity. The trigger should be triggered on Order insertion**

1-Create SaleHistory Table
```sql 

CREATE TABLE  SalesHistory(
historyID  IDENTITY PRIMARY KEY,
order_id INT NOT NULL,
product_id INT NOT NULL,
Order_Date TIMESTAMP NOT NULL,
Customer_ID INT NOT NULL,
total_amount decimal(7,3) NOT NULL CHECK(total_amount > 0),
quantity INT NOT NULL CHECK(quantity > 0),
    
FOREIGN KEY (order_id) REFERENCES orders(order_id),
FOREIGN KEY (product_id) REFERENCES product(product_id),
FOREIGN KEY (customer_id) REFERENCES customer(customer_id),
);

```
2- Create sale history Trigger
```sql
CREATE TRIGGER SALE_HISTORY_TRIGGER
    AFTER INSERT ON Order
   AS
   BEGIN
    SET NOCOUNT ON;

 -- INSERT
    INSERT INTO SaleHistory
    SELECT
        i.order_id,
        i.order_date,
        i.customer_id,
          (
            SELECT od.product_id,od.quantity , (od.unit_price * od.quantity ) As total_amount
            FROM ORDER_DETAILS od
            WHERE od.order_id = i.order_id
        ),
    FROM inserted i;
    END;
```
## Transaction queries - In the product table

**Write a transaction query to lock the field quantity with product id = 211 from being updated**
```sql
BEGIN;

SELECT quantity
FROM product
WHERE product_id = 211
FOR UPDATE;

COMMIT;
```
**Write a transaction query to lock row with product id = 211 from being updated**

```sql
BEGIN;

SELECT *
FROM product
WHERE product_id = 211
FOR UPDATE;

COMMIT;
```

## Query Performance Benchmark

|             | No Index | Idx(name) & Idx(state_id) | Idx(name,state_id) |
|-------------|----------|---------------------------|--------------------|
|  QPS        |          |                           |                    |
| Total Times |          |                           |                    |


|    QPS      | No Index | Idx(state_id) | Idx(state_id,city,address) | Both Idx |
|-------------|----------|---------------|----------------------------|----------|
|     Q1      |          |               |                            |          |
|     Q2      |          |               |                            |          |




