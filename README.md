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

```
 ## Categories Table
```sql
CREATE TABLE Categories (
    category_id INTEGER  PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL
);

## Products Table
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

## Customers Table
```sql
CREATE TABLE CUSTOMERS (
    customer_id INTEGER PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(50) NOT NULL,
    password VARCHAR(50) NOT NULL
);

## ORDERS Table
```sql
CREATE TABLE ORDERS (
    order_id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    order_date timestamp,
    total_amount decimal(7,3) NOT NULL Check(total_amount >= 0),

    FOREIGN KEY (customer_id)  REFERENCES CUSTOMERS(cutomer_id) ON DELETE CASCADE
);

## ORDER_DETAILS Table
```sql
CREATE TABLE ORDER_DETAILS (
    order_details_id INTEGER PRIMARY KEY,
    order_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    qyantity INTEGER NOT NULL,
    unit_price decimal(7,3) NOT NULL Check(unit_price >= 0),

    FOREIGN KEY (order_id)  REFERENCES  ORDERS(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id)  REFERENCES  PRODUCTS(product_id) ON DELETE CASCADE
);
```

## ERD Diagram
![ERD diagram](ER-Diagrams/ecommerce-ERD.svg)






