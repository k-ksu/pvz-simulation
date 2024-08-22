**TASK**

It is necessary to implement a console utility for the pick-up point manager. This task is modified over 8 weeks. During this time, the readme is updated with details and wording of new requirements

WEEK_1

The program must have a help command, thanks to which you can get a list of available commands with a brief description.
List of commands to implement:

Accept an order from a courier

Return the order to the courier

Issue an order to the client

Get a list of orders

Accept return from customer

Get a list of returns
________________________________________

The program starts and waits for commands to arrive
The user types a command and presses Enter
After pressing the Enter key, the program executes the entered command and writes the changes to the file
The program waits for a new command to be entered
The program terminates execution after pressing the key combination ctrl+c or entering the exit command
When the command is executed, an additional "hash" field is written/updated to the output file

The additional "hash" field is calculated using the hash.GenerateHash() function
The GenerateHash() function is a third party function that needs to be imported and used, there is no need to implement it, it is provided by teachers or tutors.
The GenerateHash() function takes a "long time", for example several seconds. Using the function slows down the execution of the command and prevents you from entering the next command.
Come up with your own interfaces and use them in your program. Demonstrate mastery of training material about interfaces

WEEK_2

Modify the application written in Homework #1 so that the data processing process takes less time.

Exercise:

The application must run in independent mode. The user can issue the next command without waiting for the results of the previous one.
Speed up your application. Implement job processing in two threads using one of the studied competitive development patterns.
An application must use a locking mechanism (such as a mutex) to synchronize data access between processes.

_____________________________________

Add handling of system signals such as SIGINT (Ctrl+C) and SIGTERM to gracefully terminate the application. When the signal is received, the application must complete all tasks before exiting the console (graceful shutdown).
Add control of the number of goroutines through a separate command for execution. Those. The number of goroutines should change dynamically, without restarting the application.
Add notifications about the start and end of command processing for each goroutine. Implement in a separate goroutine.

WEEK_3

Modify the application written in Homework #2 to interact with data storage through Postgres rather than through a file.
Exercise:
Migrate your application from storing data in a file to Postgres.
Implement migration for DDL statements.
Use transactions.

**TECHNICAL DETAILS**

Languages: GO, PostgreSQL

….

**ARCHITECTURE**

In this assignment, I followed the principle of multi-layer architecture. I chose this option, because following the logic of the task, we can conditionally select the layers responsible for:
reading commands
business logic
working with data storage.
Moreover, many commands are “end-to-end”, that is, they follow through all layers of the architecture.
Layers:

CLI

MODULE

REPOSITORY

Additional parts for realising necessary logic:

VALIDATOR

DATABASE

There is also main with initialization of all necessary objects in cmd package to execute this code

I also supplement the description of the structure with a detailed diagram for a visual representation of how our code works:
scheme

**HOW TO START THIS PROGRAM**

this code supports 2 versions of interracting with user: via command line arguments and via console.

Initially, the usual code is entered to start the program: go run ./cmd/main.go

if you then enter an additional argument help, you will get a list of possible functions and continue working with the arguments
if you do not enter anything after the command, then you automatically go to the option of working with the command line, but I also recommend starting with the help command to get information about working with this version

**HOW FUNCTIONS RUN**

Accept an order from a courier: information about the order is read, the correctness of the entered data and the validity of the date are checked, then it is checked whether such an order was delivered to the delivery point earlier, and if everything is fine, it is added to the table with the status “accepted”

Return the order to the courier: we read the data about the order, which should be returned to the courier, check the correctness of the data, the presence of an order for delivery, and if everything is fine, we will return it - that is, we will remove it from our database of orders

Issue an order to the client: we get a list of orders, check which ones we have on the PVZ, check that the orders are in the accepted state, and also that they all belong to the same client, and if everything is fine, we issue them to the client, that is, we change their status to given

Get a list of orders: we get the client id, return an array of his orders and output everything to the console

Accept return from customer: we receive return data from the client, check whether there is such an order, whether it was issued, how many days have passed since issue - if less than 2, the order can be returned. If anything, we return it - in the database we change its state to accepted

Get a list of returns: we simply call the array with returns from the database and output it paginated to the console

**WHERE WE STORE INFO**

1. json - implemented. Minimal modifications required to switch to it
2. postgres - relevant in this version

**EXTRA TASK FOR WEEK_3**

Analyze queries in the database. Attach the analysis results to README.md.

I made an index for order_id and client_id since orders can be filtered by order_id and client_id. I don't make index on condition, because indexes on boolean columns are quite specific - they are usually not useful. I used hash index, since we use search only through comparison for equality, and in the case of a hash index it will be faster than with a b-tree index

Before indexing:

1. `explain INSERT INTO orders(order_id, client_id, condition, date_arrival, date_receiving) VALUES ('a', 'b', 'accepted', '10.10.1090', '10.10.1090')`

   Query plan:
   
   Insert on orders  (cost=0.00..0.01 rows=1 width=84)
   ->  Result  (cost=0.00..0.01 rows=1 width=84)


2. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders where order_id='24'`

   Query plan:

   Seq Scan on orders  (cost=0.00..19.00 rows=4 width=84)
   Filter: (order_id = '24'::text)


3. `explain UPDATE orders SET client_id='1', condition='given', date_arrival='10.10.1090', date_receiving='10.10.1090' WHERE order_id='3';`

   Query plan:
   
   Update on orders  (cost=0.00..19.00 rows=4 width=90)
   ->  Seq Scan on orders  (cost=0.00..19.00 rows=4 width=90)
   Filter: (order_id = '3'::text)


4. `explain DELETE FROM orders WHERE order_id = ('3')`

   Query plan:
   
   Delete on orders  (cost=0.00..19.00 rows=4 width=6)
   ->  Seq Scan on orders  (cost=0.00..19.00 rows=4 width=6)
   Filter: (order_id = '3'::text)


5. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE condition='accepted';`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.05 rows=1 width=84)
   Filter: (condition = 'accepted'::conditions)


6. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE order_id='1' AND client_id='2';`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.06 rows=1 width=84)
   Filter: ((order_id = '1'::text) AND (client_id = '2'::text))


7. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE client_id='2';`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.05 rows=1 width=84)
   Filter: (client_id = '2'::text)

   
8. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE client_id='2' AND condition='accepted';`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.06 rows=1 width=84)
   Filter: ((client_id = '2'::text) AND (condition = 'accepted'::conditions))


9. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE order_id = ANY ('{2, 5}');`

   Query plan

   Seq Scan on orders  (cost=0.00..1.05 rows=2 width=84)
   "  Filter: (order_id = ANY ('{2,5}'::text[]))"


After indexing with hash:

1. `explain INSERT INTO orders(order_id, client_id, condition, date_arrival, date_receiving) VALUES ('a', 'b', 'accepted', '10.10.1090', '10.10.1090')`

   Query plan:
   
   Insert on orders  (cost=0.00..0.01 rows=1 width=84)
   ->  Result  (cost=0.00..0.01 rows=1 width=84)


2. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders where order_id='24'`

   Query plan:

   Bitmap Heap Scan on orders  (cost=4.03..12.49 rows=4 width=84)
   Recheck Cond: (order_id = '24'::text)
   ->  Bitmap Index Scan on idx_order_id  (cost=0.00..4.03 rows=4 width=0)
   Index Cond: (order_id = '24'::text)


3. `explain UPDATE orders SET client_id='1', condition='given', date_arrival='10.10.1090', date_receiving='10.10.1090' WHERE order_id='3';`

   Query plan:
   
   Update on orders  (cost=4.03..12.49 rows=4 width=90)
   ->  Bitmap Heap Scan on orders  (cost=4.03..12.49 rows=4 width=90)
   Recheck Cond: (order_id = '3'::text)
   ->  Bitmap Index Scan on idx_order_id  (cost=0.00..4.03 rows=4 width=0)
   Index Cond: (order_id = '3'::text)


4. `explain DELETE FROM orders WHERE order_id = ('3')`

   Query plan:
   
   Delete on orders  (cost=4.03..12.49 rows=4 width=6)
   ->  Bitmap Heap Scan on orders  (cost=4.03..12.49 rows=4 width=6)
   Recheck Cond: (order_id = '3'::text)
   ->  Bitmap Index Scan on idx_order_id  (cost=0.00..4.03 rows=4 width=0)
   Index Cond: (order_id = '3'::text)


5. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE condition='accepted'`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.05 rows=1 width=84)
   Filter: (condition = 'accepted'::conditions)


6. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE order_id='1' AND client_id='2';`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.06 rows=1 width=84)
   Filter: ((order_id = '1'::text) AND (client_id = '2'::text))


7. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE client_id='2'`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.05 rows=1 width=84)
   Filter: (client_id = '2'::text)


8. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE client_id='2' AND condition='accepted';`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.06 rows=1 width=84)
   Filter: ((client_id = '2'::text) AND (condition = 'accepted'::conditions))


9. `explain SELECT order_id, client_id, condition, date_arrival, date_receiving FROM orders WHERE order_id = ANY ('{2, 5}');`

   Query plan:

   Seq Scan on orders  (cost=0.00..1.05 rows=2 width=84)
   "  Filter: (order_id = ANY ('{2,5}'::text[]))"
   Update on orders  (cost=4.03..12.49 rows=4 width=90)
   ->  Bitmap Heap Scan on orders  (cost=4.03..12.49 rows=4 width=90)
   Recheck Cond: (order_id = '3'::text)
   ->  Bitmap Index Scan on idx_order_id  (cost=0.00..4.03 rows=4 width=0)
   Index Cond: (order_id = '3'::text)

**EXTRA TASK FOR WEEK_4**

I add 2 diagrams

First one was made just for my understanding

Second one was made for the task. It is Sequence diagram. 
I took the article https://habr.com/ru/companies/epam_systems/articles/538018/ as an example