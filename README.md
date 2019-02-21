# Database assignment 4

_Assignment:_ 
https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment4.md

## Get up and running

Following will download the data, and create a Docker container.

You will be put inside the docker container where you have to execute  all the following commands

**Start container and install dependencies to fetch the database**
```
sudo docker run --rm --name my_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=pass1234 -d mysql
sudo docker exec -it my_mysql bash 
echo "Following is inside the container"
apt-get update
apt-get install wget unzip -y
```
**Download the data**
```
wget http://www.mysqltutorial.org/wp-content/uploads/2018/03/mysqlsampledatabase.zip
unzip mysqlsampledatabase.zip
```
**Import data** __You might have to run this command a couple of times, the server is not always that fast to boot up__
```
mysql -u root -ppass1234 < mysqlsampledatabase.sql
```

## Enable logging

The data is now loaded, we need logging for later, let's start the recording

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

**Bookkeeping** are going to be able to read/write to customers and payments to fix bugs, and they have to be able to change it for customer support if the data doesn't match, they need to read access in all of the other tables to join in the customer data for getting an overview of the customer, EXCEPT the employee of offices table
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

**Human resources** are the business managers in regards to the data, read/write to employee of offices
```
mysql -u root -ppass1234 -e " \
CREATE USER 'humanresource'@'%' IDENTIFIED BY 'pass1234'; \
GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels.employees TO 'humanresource'@'%'; \
GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels.offices TO 'humanresource'@'%'; \
FLUSH PRIVILEGES;   \
"
```

**Sales** talks to the customers, the need to add sales and insert/modify customers for new clients or new info about a client, read/write customers and orders, delete to orders as well if they regards
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

__We can see the effect by running__ `mysql -u humanresource -ppass1234 -e "USE classicmodels; SHOW TABLES;"`

## Exercise 2 - logging

Logging is already enabled, time to fire off some commands

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
WHERE argument != 'SHOW GLOBAL STATUS' \
ORDER BY event_time DESC \
LIMIT 100; \
"
```
#### Result of log

```
+----------------------------+-------------------------------------+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| event_time                 | user_host                           | command_type | argument                                                                                                                                                           |
+----------------------------+-------------------------------------+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 2019-02-21 21:26:32.157184 | root[root] @ localhost []           | Query        | SELECT event_time, user_host, command_type, argument FROM mysql.general_log WHERE argument != 'SHOW GLOBAL STATUS' ORDER BY event_time DESC LIMIT 100              |
| 2019-02-21 21:26:32.156905 | root[root] @ localhost []           | Query        | select @@version_comment limit 1                                                                                                                                   |
| 2019-02-21 21:26:32.156460 | [root] @ localhost []               | Connect      | root@localhost on  using Socket                                                                                                                                    |
| 2019-02-21 21:26:18.913767 | Inventory[Inventory] @ localhost [] | Quit         |                                                                                                                                                                    |
| 2019-02-21 21:26:18.913593 | Inventory[Inventory] @ localhost [] | Query        | INSERT INTO classicmodels.employees(lastName,firstName,extension,email,officeCode,reportsTo,jobTitle)VALUES('larsen','bo','x101','hacker@live.dk',1,1002,'hacker') |
| 2019-02-21 21:26:18.913346 | Inventory[Inventory] @ localhost [] | Query        | select @@version_comment limit 1                                                                                                                                   |
| 2019-02-21 21:26:18.912705 | [Inventory] @ localhost []          | Connect      | Inventory@localhost on  using Socket                                                                                                                               |
| 2019-02-21 21:26:12.597862 | root[root] @ localhost []           | Quit         |                                                                                                                                                                    |
| 2019-02-21 21:26:12.597707 | root[root] @ localhost []           | Query        | select @@version_comment limit 1                                                                                                                                   |
| 2019-02-21 21:26:12.597132 | [root] @ localhost []               | Connect      | root@localhost on  using Socket                                                                                                                                    |
| 2019-02-21 21:26:06.371001 | root[root] @ localhost []           | Quit         |                                                                                                                                                                    |
| 2019-02-21 21:26:06.354194 | root[root] @ localhost []           | Query        | FLUSH PRIVILEGES                                                                                                                                                   |
| 2019-02-21 21:26:06.337550 | root[root] @ localhost []           | Query        | GRANT ALL PRIVILEGES ON classicmodels . * TO 'IT'@'localhost'                                                                                                      |
| 2019-02-21 21:26:06.317099 | root[root] @ localhost []           | Query        | CREATE USER 'IT'@'localhost' IDENTIFIED BY <secret>                                                                                                                |
| 2019-02-21 21:26:06.316715 | root[root] @ localhost []           | Query        | select @@version_comment limit 1                                                                                                                                   |
| 2019-02-21 21:26:06.316141 | [root] @ localhost []               | Connect      | root@localhost on  using Socket                                                                                                                                    |
| 2019-02-21 21:26:00.738818 | root[root] @ localhost []           | Quit         |                                                                                                                                                                    |
| 2019-02-21 21:26:00.722107 | root[root] @ localhost []           | Query        | FLUSH PRIVILEGES                                                                                                                                                   |
| 2019-02-21 21:26:00.703041 | root[root] @ localhost []           | Query        | GRANT SELECT, INSERT, UPDATE ON classicmodels . customers TO 'Sales'@'%'                                                                                           |
| 2019-02-21 21:26:00.685901 | root[root] @ localhost []           | Query        | GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels . orderdetails TO 'Sales'@'%'                                                                                |
| 2019-02-21 21:26:00.668873 | root[root] @ localhost []           | Query        | GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels . orders TO 'Sales'@'%'                                                                                      |
| 2019-02-21 21:26:00.648602 | root[root] @ localhost []           | Query        | CREATE USER 'Sales'@'%' IDENTIFIED BY <secret>                                                                                                                     |
| 2019-02-21 21:26:00.648328 | root[root] @ localhost []           | Query        | select @@version_comment limit 1                                                                                                                                   |
| 2019-02-21 21:26:00.647865 | [root] @ localhost []               | Connect      | root@localhost on  using Socket                                                                                                                                    |
| 2019-02-21 21:25:54.539724 | root[root] @ localhost []           | Quit         |                                                                                                                                                                    |
| 2019-02-21 21:25:54.522911 | root[root] @ localhost []           | Query        | FLUSH PRIVILEGES                                                                                                                                                   |
| 2019-02-21 21:25:54.504273 | root[root] @ localhost []           | Query        | GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels.offices TO 'humanresource'@'%'                                                                               |
| 2019-02-21 21:25:54.486001 | root[root] @ localhost []           | Query        | GRANT SELECT, INSERT, UPDATE, DELETE ON classicmodels.employees TO 'humanresource'@'%'                                                                             |
| 2019-02-21 21:25:54.469238 | root[root] @ localhost []           | Query        | CREATE USER 'humanresource'@'%' IDENTIFIED BY <secret>                                                                                                             |
| 2019-02-21 21:25:54.469085 | root[root] @ localhost []           | Query        | select @@version_comment limit 1                                                                                                                                   |
| 2019-02-21 21:25:54.468607 | [root] @ localhost []               | Connect      | root@localhost on  using Socket                                                                                                                                    |
| 2019-02-21 21:25:47.985760 | root[root] @ localhost []           | Quit         |                                                                                                                                                                    |
| 2019-02-21 21:25:47.969406 | root[root] @ localhost []           | Query        | FLUSH PRIVILEGES                                                                                                                                                   |
| 2019-02-21 21:25:47.952241 | root[root] @ localhost []           | Query        | GRANT SELECT ON classicmodels . products TO 'Bookkeeping'@'%'                                                                                                      |
| 2019-02-21 21:25:47.933286 | root[root] @ localhost []           | Query        | GRANT SELECT ON classicmodels . productlines TO 'Bookkeeping'@'%'                                                                                                  |
| 2019-02-21 21:25:47.916182 | root[root] @ localhost []           | Query        | GRANT SELECT ON classicmodels . orders TO 'Bookkeeping'@'%'                                                                                                        |
| 2019-02-21 21:25:47.897504 | root[root] @ localhost []           | Query        | GRANT SELECT ON classicmodels . orderdetails TO 'Bookkeeping'@'%'                                                                                                  |
| 2019-02-21 21:25:47.878644 | root[root] @ localhost []           | Query        | GRANT SELECT, INSERT, UPDATE ON classicmodels . payments TO 'Bookkeeping'@'%'                                                                                      |
| 2019-02-21 21:25:47.861539 | root[root] @ localhost []           | Query        | GRANT SELECT, INSERT, UPDATE ON classicmodels . customers TO 'Bookkeeping'@'%'                                                                                     |
| 2019-02-21 21:25:47.843869 | root[root] @ localhost []           | Query        | CREATE USER 'Bookkeeping'@'%' IDENTIFIED BY <secret>                                                                                                               |
| 2019-02-21 21:25:47.843606 | root[root] @ localhost []           | Query        | select @@version_comment limit 1                                                                                                                                   |
| 2019-02-21 21:25:47.843182 | [root] @ localhost []               | Connect      | root@localhost on  using Socket                                                                                                                                    |
| 2019-02-21 21:25:37.399111 | root[root] @ localhost []           | Quit         |                                                                                                                                                                    |
| 2019-02-21 21:25:37.382193 | root[root] @ localhost []           | Query        | FLUSH PRIVILEGES                                                                                                                                                   |
| 2019-02-21 21:25:37.363837 | root[root] @ localhost []           | Query        | GRANT SELECT,INSERT,UPDATE,DELETE  ON classicmodels.productlines TO 'Inventory'@'%'                                                                                |
| 2019-02-21 21:25:37.347168 | root[root] @ localhost []           | Query        | GRANT SELECT,INSERT,UPDATE,DELETE  ON classicmodels.products TO 'Inventory'@'%'                                                                                    |
| 2019-02-21 21:25:37.325982 | root[root] @ localhost []           | Query        | CREATE USER 'Inventory'@'%' IDENTIFIED BY <secret>                                                                                                                 |
| 2019-02-21 21:25:37.325695 | root[root] @ localhost []           | Query        | select @@version_comment limit 1                                                                                                                                   |
| 2019-02-21 21:25:37.325186 | [root] @ localhost []               | Connect      | root@localhost on  using Socket                                                                                                                                    |
| 2019-02-21 21:25:31.387802 | root[root] @ localhost []           | Quit         |                                                                                                                                                                    |
+----------------------------+-------------------------------------+--------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+

```

_The status of the failing line isn't logged as failed, and can't find any flag to turn this on, but we can see it has been ran by the wrong user_

## Exercise 3 - backup and recovery

### Backup data and structure
_Users are not in the backup in following commands, the can be backed up through the internal user's table, but it is better to use the create user scripts for this_

`mysqldump -u root -ppass1234 --databases classicmodels > database_blackup_after_execises.sql`

_File is included in the repo_

Tip: File can be moved out of the container with `sudo docker cp my_mysql:/database_blackup_after_execises.sql .` and in with `sudo docker cp database_blackup_after_execises.sql my_mysql:/`

### Restore

**Delete database**

```
mysql -u root -ppass1234 -e " \
DROP DATABASE classicmodels; \
"
```

**Restore database**
```
mysql -u root -ppass1234 < /database_blackup_after_execises.sql
```

## Cleanup

Exit the shell with `CTRL + d`

Remove the docker container with `sudo docker rm -f my_mysql`
