# Problem 5

Consider the following database for a banking enterprise:

* Branch (***branch-name***: string, branch-city: string, assets: real)
* Account (***accno***: int, branch-name: string, balance: real)
* Depositor (***customer-name***: string, ***accno***: int)
* Customer (***customer-name***: string, customer-street: string, customer-city: string)
* Loan (***loan-number***: int, branch-name: string, amount: real)
* Borrower (***customer-name***: string, ***loan-number***: int)

![p5_schema](./p5_schema.jpeg)

## 1. Create the above tables by properly specifying the primary and foreign keys

```sql
CREATE TABLE Branch (
    br_name  varchar2(20)   PRIMARY KEY,
    br_city  varchar2(20),
    assets   number(10, 2)
);
```

```sql
CREATE TABLE Account (
    acc_no    number(10)     PRIMARY KEY,
    br_name   varchar2(20)   REFERENCES Branch,
    bal       number(10, 2)
);
```

```sql
CREATE TABLE Customer (
    cust_name    varchar2(20)   PRIMARY KEY,
    cust_street  varchar2(20),
    cust_city    varchar2(20)
);
```

```sql
CREATE TABLE Depositor (
    cust_name  varchar2(20)   REFERENCES Customer ON DELETE CASCADE,
    acc_no     number(10)     REFERENCES Account ON DELETE CASCADE,

    PRIMARY KEY(cust_name, acc_no)
);
```

*Note:* If `ON DELETE CASCADE` option is set, a delete operation in the master table will trigger a delete operation for corresponding records in all foreign tables.

```sql
CREATE TABLE Loan (
    loan_no   number(10)     PRIMARY KEY,
    br_name   varchar2(20)   REFERENCES Branch ON DELETE CASCADE
    amt       number(10, 2)
);
```

```sql
CREATE TABLE Borrower (
    cust_name   varchar2(20)   REFERENCES Customer ON DELETE CASCADE,
    loan_no     number(10)     REFERENCES Loan ON DELETE CASCADE,

    PRIMARY KEY(cust_name, loan_no)
);
```

## 2. Find all the customers who have at least two accounts at the main branch.

```sql
SELECT DISTINCT Depositor.cust_name

FROM   Depositor,
       Account

WHERE  Depositor.acc_no = Account.acc_no AND
       Account.br_name = 'Main'

GROUP BY Depositor.cust_name

HAVING COUNT(Depositor.acc_no) >= 2;  
```

## 3. Find all the customers who have an account at *all* the branches located in a specific city.

```sql
SELECT Customer.cust_name

FROM   Customer

WHERE  NOT EXISTS (

    (SELECT br_name
     FROM   Branch
     WHERE  br_city = 'Bangalore')

    MINUS

    (SELECT Account.br_name
     FROM   Depositor, Account
     WHERE  Depositor.acc_no = Account.acc_no AND
            Depositor.cust_name = Customer.cust_name)

);  
```

## 4. Demonstate how to delete all account tuples at every branch in a specific city.

```sql
DELETE FROM Account

WHERE  br_name IN (
    SELECT br_name
    FROM   branch
    WHERE  br_city = 'Bangalore'
);  
```

# Problem 4

The following tables are maintained by a book dealer:

* Author (***author-id***: int, name: string, city: string, country: string)
* Publisher (***publisher-id***: int, name: string, city: string)
* Category (***category-id***: int, description: string)
* Catalog (***book-id***: int, title: string, author-id: int, publisher-id: int, category-id: int, year: int, price: int)
* Order\_Details (***order-no***: int, book-id: int, quantity: int)

![p4_schema](./p4_schema.jpeg)

## 1. Create the above tables by properly specifying the primary and foreign keys

```sql
CREATE TABLE Author (
    author_id   number(6)        PRIMARY KEY,
    name        varchar2(20),
    city        varchar2(20),
    country     varchar2(20)
);
```

```sql
CREATE TABLE Publisher (
    publisher_id   number(6)     PRIMARY KEY,
    name           varchar2(30),
    city           varchar2(20),
    country        varchar2(20)
);
```

```sql
CREATE TABLE Category (
    category_id   number(6)      PRIMARY KEY,
    description   varchar2(20)
);
```

```sql
CREATE TABLE Catalog (
    book_id        number(6)     PRIMARY KEY,
    title          varchar2(20),
    author_id      number(6)     REFERENCES Author
    publisher_id   number(6)     REFERENCES Publisher
    category_id    number(6)     REFERENCES Category
    year           number(4),
    prince         number(7, 2)
);
```

```sql
CREATE TABLE Order_Details (
    order_no   number(6)         PRIMARY KEY,
    book_id    number(6)         REFERENCES Catalog,
    qty        number(6)
);
```

## 2. Find the authors who have 2 or more books in the catalog, where the price of the books is greater than the average price of the books in the catalog, and the year of publication is after 2000

```sql
SELECT *

FROM   Author

WHERE  author_id IN (

    SELECT author_id
    FROM   Catalog
    WHERE  price > (

        SELECT AVG(price)
        FROM   Catalog

    )

    AND year > 2000

    GROUP BY author_id

    HAVING Count(*) >= 2
);  
```

## 3. Find the author of the book with the most sales

```sql
SELECT author_id,
       name

FROM   Author,
       Catalog

WHERE  Author.author_id = Catalog.author_id AND
       Catalog.book_id IN (

           SELECT book_id
           FROM   Order_Details
           GROUP BY book_id
           HAVING SUM(qty) = (

               SELECT MAX(SUM(quantity))
               FROM   Order_Details
               GROUP BY book_id

           )

);  
```

## 4. Demonstrate how to increase the price of a specific publisher's books by 10%

```sql
UPDATE Catalog

SET    price = ( price * 1.1 )

WHERE  publisher_id IN (

    SELECT publisher_id
    FROM   Publisher
    WHERE  name = 'eee'
    
);  
```