# E-commerce Exam Database Project

## Table of Contents

1. Project Overview
2. Database Setup
3. Tables and Structure
4. Sample Data
5. Queries and Analysis
6. Advanced Queries
7. Notes

---

## Project Overview

This project is an **E-commerce database system** designed to manage products, customers, orders, payments, and shipping information. It allows retrieval of various reports such as top-selling products, customer spending, monthly revenue, and shipping performance.

---

## Database Setup

```sql
CREATE DATABASE E_commerce_Exam;
USE E_commerce_Exam;
```

---

## Tables and Structure

### 1. Categories

```sql
CREATE TABLE Categories (
    CategoryID INT PRIMARY KEY,
    CategoryName VARCHAR(50)
);
```

### 2. Product

```sql
CREATE TABLE Product (
    ProductID INT PRIMARY KEY,
    Name VARCHAR(50),
    CategoryID INT,
    Price DECIMAL(10,2),
    AddedDate DATE,
    Stock INT,
    FOREIGN KEY (CategoryID) REFERENCES Categories(CategoryID)
);
```

### 3. Customer

```sql
CREATE TABLE Customer (
    CustomerID INT PRIMARY KEY,
    Name VARCHAR(50),
    Email VARCHAR(50),
    Phone VARCHAR(15),
    Address VARCHAR(100),
    RegistrationDate DATE
);
```

### 4. Orders

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID)
);
```

### 5. OrderItem

```sql
CREATE TABLE OrderItem (
    OrderItemID INT PRIMARY KEY,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    SubTotal DECIMAL(10,2),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Product(ProductID)
);
```

### 6. Payment

```sql
CREATE TABLE Payment (
    PaymentID INT PRIMARY KEY,
    OrderID INT,
    PaymentDate DATE,
    PaymentMethod VARCHAR(20),
    PaymentStatus VARCHAR(20),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);
```

### 7. Shipping

```sql
CREATE TABLE Shipping (
    ShippingID INT PRIMARY KEY,
    OrderID INT,
    ShippingDate DATE,
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);
```

---

## Sample Data

* **Categories:** Electronics, Clothing, Footwear, Furniture, Grocery, Toys, Books, Sports, Beauty, Accessories.
* **Products:** Laptop, Shirt, Shoes, Chair, Rice, Toy Car, Story Book, Football, Face Wash, Wallet.
* **Customers:** 10 initial customers with sample emails, phone, and registration dates.
* **Orders & OrderItems:** 10 orders with multiple items.
* **Payments:** Paid, Pending, and Failed statuses.
* **Shipping:** Shipping dates for all orders.

---

## Queries and Analysis

### 1. Product & Category Analysis

* List all products with category:

```sql
SELECT p.ProductID, p.Name, p.Price, c.CategoryName
FROM Product p
INNER JOIN Categories c ON p.CategoryID = c.CategoryID
ORDER BY p.Price DESC;
```

* Total revenue per category:

```sql
SELECT c.CategoryName, SUM(oi.SubTotal) AS TotalRevenue
FROM Categories c
JOIN Product p ON c.CategoryID = p.CategoryID
JOIN OrderItem oi ON p.ProductID = oi.ProductID
GROUP BY c.CategoryID, c.CategoryName
ORDER BY TotalRevenue DESC;
```

### 2. Customer Analysis

* Customers registered after 2022 or spent above 10000:

```sql
SELECT DISTINCT c.CustomerID, c.Name, c.Email, c.RegistrationDate
FROM Customer c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
LEFT JOIN OrderItem oi ON o.OrderID = oi.OrderID
GROUP BY c.CustomerID, c.Name, c.Email, c.RegistrationDate
HAVING c.RegistrationDate > '2022-12-31' OR SUM(oi.SubTotal) > 10000;
```

* Top spending customer:

```sql
SELECT c.CustomerID, c.Name, c.Email, SUM(oi.SubTotal) AS TotalSpent
FROM Customer c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN OrderItem oi ON o.OrderID = oi.OrderID
JOIN Payment p ON o.OrderID = p.OrderID
WHERE p.PaymentStatus = 'Paid'
GROUP BY c.CustomerID, c.Name, c.Email
ORDER BY TotalSpent DESC
LIMIT 1;
```

### 3. Order & Payment Analysis

* Orders in the last 6 months:

```sql
SELECT p.ProductID, p.Name, p.Price, o.OrderDate
FROM Orders o
JOIN OrderItem oi ON o.OrderID = oi.OrderID
JOIN Product p ON oi.ProductID = p.ProductID
WHERE o.OrderDate >= CURDATE() - INTERVAL 6 MONTH;
```

* Orders not shipped yet:

```sql
SELECT o.OrderID, o.OrderDate, s.ShippingDate
FROM Orders o
LEFT JOIN Shipping s ON o.OrderID = s.OrderID
WHERE s.ShippingDate IS NULL;
```

### 4. Product Analysis

* Products not ordered yet:

```sql
SELECT p.ProductID, p.Name, p.Price
FROM Product p
WHERE p.ProductID NOT IN (SELECT DISTINCT ProductID FROM OrderItem);
```

* Top 5 expensive products:

```sql
SELECT * FROM Product
ORDER BY Price DESC
LIMIT 5;
```

* Products in stock:

```sql
SELECT * FROM Product
WHERE Stock > 0;
```

---

## Advanced Queries

* Monthly revenue (paid orders):

```sql
SELECT YEAR(o.OrderDate) AS year, MONTH(o.OrderDate) AS month, ANY_VALUE(MONTHNAME(o.OrderDate)) AS month_name, SUM(oi.SubTotal) AS monthly_revenue
FROM Orders o
JOIN OrderItem oi ON o.OrderID = oi.OrderID
JOIN Payment p ON o.OrderID = p.OrderID
WHERE p.PaymentStatus = 'Paid'
GROUP BY YEAR(o.OrderDate), MONTH(o.OrderDate)
ORDER BY year, month;
```

* Cumulative revenue:

```sql
SELECT year, month, month_name, monthly_revenue,
       SUM(monthly_revenue) OVER (ORDER BY year, month ROWS UNBOUNDED PRECEDING) AS cumulative_revenue
FROM (
    SELECT YEAR(o.OrderDate) AS year, MONTH(o.OrderDate) AS month, ANY_VALUE(MONTHNAME(o.OrderDate)) AS month_name, SUM(oi.SubTotal) AS monthly_revenue
    FROM Orders o
    JOIN OrderItem oi ON o.OrderID = oi.OrderID
    JOIN Payment p ON o.OrderID = p.OrderID
    WHERE p.PaymentStatus = 'Paid'
    GROUP BY YEAR(o.OrderDate), MONTH(o.OrderDate)
) AS monthly_data
ORDER BY year, month;
```

* Running total of orders:

```sql
SELECT o.OrderID, o.OrderDate, SUM(oi.SubTotal) OVER (ORDER BY o.OrderDate, o.OrderID ROWS UNBOUNDED PRECEDING) AS running_total
FROM Orders o
JOIN OrderItem oi ON o.OrderID = oi.OrderID
JOIN Payment p ON o.OrderID = p.OrderID AND p.PaymentStatus = 'Paid'
ORDER BY o.OrderDate, o.OrderID;
```

---

## Notes

* All monetary calculations consider only **Paid** payments.
* `Stock` field in Product table is used to track inventory updates.
* The database supports **archiving old delivered orders** and maintaining historical data.
* Queries utilize window functions, aggregate functions, joins, and subqueries for advanced analysis.
