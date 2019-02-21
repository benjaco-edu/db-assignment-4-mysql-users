# Database assignment 4

_Assignment:_ 
https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment4.md

## Get up and running

Follwing will download the data, and create a docker container.

You will be put inside the docker container where you have to execute the all the following commands unless it is marjed otherwise

```
wget http://www.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip
unzip mysqlsampledatabase.zip
sudo docker run --rm --name my_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=pass1234 -d mysql
sudo docker cp mysqlsampledatabase.sql my_mysql:/tmp
sudo docker exec -it my_mysql bash 
# Following is inside the hash
sleep 5 # problems with not beeing ready, next line might have to be retried
mysql -u root -ppass1234 < /tmp/mysqlsampledatabase.sql
```

## Enable logging

The data is now loaded, we need logging for later, lets start the recording

```
mysql -u root -ppass1234 -e " \
SET global general_log = 1; SET global log_output = 'table' \
"
```

## Excercise 1 - user privileges

**Inventory** has to be albe to do all operations on products and productlines (contains the product description) to mange the products
```
mysql -u root -ppass1234 -e " \
CREATE USER 'Inventory'@'%' IDENTIFIED BY 'pass1234'; \
GRANT SELECT,INSERT,UPDATE,DELETE  ON classicmodels.products TO 'Inventory'@'%'; \
GRANT SELECT,INSERT,UPDATE,DELETE  ON classicmodels.productlines TO 'Inventory'@'%'; \
FLUSH PRIVILEGES; \
"
```

**Bookkeeping** are going to be able to read/write to customers and payments to fix bugs, and they have to be able to change it for customer support if the data dosnt match, they need read access in all of the other tables to join in the customer data for getting a overwiev of the customer, EXCEPT the employee og offices table
```
mysql -u root -ppass1234 -e " \
CREATE USER 'Bookkeeping'@'%' IDENTIFIED BY 'pass1234'; \
GRANT SELECT, INSERT, UPDATE ON classicmodels . customers TO 'Bookkeeping'@'%'; \
GRANT SELECT, INSERT, UPDATE ON classicmodels . payments TO 'Bookkeeping'@'%'; \
GRANT SELECT ON classicmodels . orderdetails TO 'Bookkeeping'@'%'; \
GRANT SELECT ON classicmodels . orders TO 'Bookkeeping'@'%'; \
GRANT SELECT ON classicmodels . productlines TO 'Bookkeeping'@'%'; \
GRANT SELECT ON classicmodels . products TO 'Bookkeeping'@'%'; \
FLUSH PRIVILEGES; \
"
```

**Human ressources** are the bussiness mangers in regards of the data, read/write to employee og offices
```
mysql -u root -ppass1234 -e " \
CREATE USER 'humanresource'@'%' IDENTIFIED BY 'pass1234'; \
GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels.employees TO 'humanresource'@'%'; \
GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels.offices TO 'humanresource'@'%'; \
FLUSH PRIVILEGES;   \
"
```

**Sales** talks to the customers, the need to add sales and insert/modify customers for new clients or new info about a client, read/write customers and orders, delete to orders as well if they regeds
```
mysql -u root -ppass1234 -e " \
CREATE USER 'Sales'@'%' IDENTIFIED BY 'pass1234'; \
GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels . orders TO 'Sales'@'%'; \
GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels . orderdetails TO 'Sales'@'%'; \
GRANT SELECT, INSERT, UPDATE ON classicmodels . customers TO 'Sales'@'%'; \
FLUSH PRIVILEGES; \
"
```

**IT** has to be able to do anything to keep the database up and running, more roles should be added when the company grows.
```
mysql -u root -ppass1234 -e " \
CREATE USER 'IT'@'localhost' IDENTIFIED BY 'pass1234'; \
GRANT ALL PRIVILEGES ON classicmodels . * TO 'IT'@'localhost'; \
FLUSH PRIVILEGES; \
"
```

## Exercise 2 - logging

Logging is allready enabled, time to fire of some commands

```
mysql -u root -ppass1234 -e " \
-- Employees \
INSERT INTO classicmodels.employees (employeeNumber,lastname, firstname, extension, email, officeCode, reportsTo, jobTitle) values (1703, 'Hanson','Cliver','x101','mymail@hotmail.com','1','1002','Junior Sales'); \
 \
INSERT INTO classicmodels.employees(lastName,firstName,extension,email,officeCode,reportsTo,jobTitle)VALUES('larsen','bo','x101','techsupport@live.dk',1,1002,'Tech support'); \
 \
-- Product \
INSERT INTO classicmodels.products( productName, productLine, productScale, productVendor, productDescription,quantityInStock, buyPrice, MRSP )  \
 VALUES ('Toyota Lexus', 'classic car' ,'1:10','Unimax Art Galleries', 'speedy','500', '50','90'); \
 \
-- Order \
INSERT INTO classicmodels.orders (orderNumber, orderDate, requiredDate, shippedDate, status, comments, customerNumber) VALUES (10500, '2013-01-09', '2013-01-18', null, 'On Hold', 'Check on availability.', 128); \
INSERT INTO classicmodels.orderdetails (orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber) VALUES (10500, 'S18_1749', 30, 136.00, 3); \
INSERT INTO classicmodels.orderdetails (orderNumber, productCode, quantityOrdered, priceEach, orderLineNumber) VALUES (10500, 'S18_2248', 50, 55.09, 2); \
"
```

### Illegal query

```
mysql -u Inventory -ppass1234 -e " \
INSERT INTO classicmodels.employees(lastName,firstName,extension,email,officeCode,reportsTo,jobTitle)VALUES('larsen','bo','x101','hacker@live.dk',1,1002,'hacker'); \
"
```

### Show the log

```
mysql -u root -ppass1234 -e " \
SELECT event_time, user_host, command_type, argument \
FROM mysql.general_log \
ORDER BY event_time DESC \
LIMIT 20; \
"
```
#### Result of log

```
todo
```

## Exercise 3 - backup and recovery





