MySQL short Memo
================

For a full tutorial look at `MySQL
Tutorial <http://dev.mysql.com/doc/refman/5.6/en/tutorial.html>`__

-  Connect to mysql interpretor, create a database, give rights to some
   user:

   ::

       mysql -h mysql.example.com -u user -p
       CREATE DATABASE mydb;
       grant all privileges on mydb.* to user@localhost identified by 'password';

-  `show <http://dev.mysql.com/doc/refman/5.6/en/show.html>`__
   databases, select a database, show tables, `show
   columns <http://dev.mysql.com/doc/refman/5.6/en/show-columns.html>`__
   from a table (or
   `describe <http://dev.mysql.com/doc/refman/5.6/en/describe.html>`__ :

   ::

       SHOW DATABASES;
       USE mydb;
       SHOW TABLES;
       SHOW COLUMNS FROM mytable;

-  `Create
   table <http://dev.mysql.com/doc/refman/5.6/en/create-table.html>`__,
   `insert a row <http://dev.mysql.com/doc/refman/5.6/en/insert.html>`__
   and `select <http://dev.mysql.com/doc/refman/5.6/en/select.html>`__ :

   ::

       CREATE TABLE shop (
       article INT(4) UNSIGNED ZEROFILL DEFAULT '0000' NOT NULL,
       dealer  CHAR(20)                 DEFAULT ''     NOT NULL,
       price   DOUBLE(16,2)             DEFAULT '0.00' NOT NULL,
       PRIMARY KEY(article, dealer));
       INSERT INTO shop VALUES
       (1,'A',3.45),(1,'B',3.99),(2,'A',10.99),(3,'B',1.45),
       (3,'C',1.69),(3,'D',1.25),(4,'D',19.95);
       SELECT * FROM shop;

       +---------+--------+-------+
       | article | dealer | price |
       +---------+--------+-------+
       |    0001 | A      |  3.45 |
       |    0001 | B      |  3.99 |
       |    0002 | A      | 10.99 |
       |    0003 | B      |  1.45 |
       |    0003 | C      |  1.69 |
       |    0003 | D      |  1.25 |
       |    0004 | D      | 19.95 |
       +---------+--------+-------+

-  Replicate a database by using
   `mysqldump <http://dev.mysql.com/doc/refman/5.6/en/mysqldump.html>`__
   for `making a
   backup <http://dev.mysql.com/doc/refman/5.6/en/backup-and-recovery.html>`__:

   ::

       mysqldump -uuser -ppassword --add-drop-table mydb_prod | \
       mysql -uuser -ppassword mydb_test
       rm /tmp/prod-db

-  `Show database
   variables <http://dev.mysql.com/doc/refman/5.6/en/show-variables.html>`__

   ::

       show variables;
       +-----------------------------------------+-------------+
       | Variable_name                           | Value       |
       +-----------------------------------------+-------------+
       | auto_increment_increment                | 1           |
       | auto_increment_offset                   | 1           |
       .......
       | character_set_client                    | latin1      |
       | character_set_connection                | latin1      |
       | character_set_database                  | latin1      |
       ......
       | warning_count                           | 0           |
       +-----------------------------------------+-------------+

       show variables like 'character_set_client';
       +----------------------+--------+
       | Variable_name        | Value  |
       +----------------------+--------+
       | character_set_client | latin1 |
       +----------------------+--------+

Quick access to references in the Mysql reference manual.
---------------------------------------------------------

-  For a query tutorial look at the `examples of common
   queries <http://dev.mysql.com/doc/refman/5.6/en/examples.html>`__
   from mysql tutorial.
-  The *ALTER, CREATE, DROP, RENAME* are described in: `*Data Definition
   Statements*
   section <http://dev.mysql.com/doc/refman/5.6/en/sql-syntax-data-definition.html>`__.
-  *DELETE, DO, HANDLER, INSERT, LOAD, REPLACE, SELECT, TRUNCATE,
   UPDATE* are described in: `*Data Manipulation Statements*
   section <http://dev.mysql.com/doc/refman/5.6/en/sql-syntax-data-manipulation.html>`__.
-  The most used
   `*SELECT* <http://dev.mysql.com/doc/refman/5.6/en/select.html>`__
   contains a subsection on `*JOIN*
   syntax <http://dev.mysql.com/doc/refman/5.6/en/join.html>`__.
-  For issuing
   `*SELECT* <http://dev.mysql.com/doc/refman/5.6/en/select.html>`__
   statement we need
   `Operators <http://dev.mysql.com/doc/refman/5.6/en/non-typed-operators.html>`__
   and
   `functions <http://dev.mysql.com/doc/refman/5.6/en/functions.html>`__,
   if you know the name of the wanted function/operator there is a
   direct `Function and operator
   summary <http://dev.mysql.com/doc/refman/5.6/en/func-op-summary-ref.html>`__
-  The less used `server administration
   statements <http://dev.mysql.com/doc/refman/5.6/en/sql-syntax-server-administration.html>`__
   that contains *SHOW* and *SET* statement, and `replication
   statements <http://dev.mysql.com/doc/refman/5.6/en/sql-syntax-replication.html>`__.
-  `mysqldump
   manual <http://dev.mysql.com/doc/refman/5.6/en/mysqldump.html>`__
