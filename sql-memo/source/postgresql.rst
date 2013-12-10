PostgreSQL Memo
===============

.. highlight:: plpgsql

References:
    :psql:`PostgreSQL documentation <index>`

psql administrative commands
----------------------------

The default database user and default database are called ``postgres``

To connect to default database:

..  code-block:: sh

    $ su - postgres
    $ psql

Add a new ordinary user and database **under a shell** with:

..  code-block:: sh

    $ createuser -DRS pguser
    $ createdb -O pguser pgdatabase

``-DRS`` tell this is an ordinary user, it is the default so can be
omitted.

You can also create a privileged user with:

..  code-block:: sh

    createuser --createdb --createrole pguser
    createuser --superuser pguser

A *superuser*  bypasses all permission checks This is a dangerous
privilege.

You can also create the user and database in *psql* when logged as the
initial superuser *postgres*::

  postgres=# CREATE USER pguser WITH PASSWORD 'pguserpass';
  postgres=# CREATE DATABASE pgdatabase OWNER pguser;

To connect to your database if your *postgresql* username match your
*unix* username:

..  code-block:: sh

    psql pgdatabase

or for any user:

..  code-block:: sh

    psql -d pgdatabase -U user

or to a databse on a remote server with password prompt:

..  code-block:: sh

    psql -h server.domain.tld -d pgdatabase -U user -W


*psql* meta commands.
~~~~~~~~~~~~~~~~~~~~~

*   ``\l`` - databases list.
*   ``\d`` - tables  views, sequences and foreign tables list.
*   ``\d sometable`` - *sometable* description, *sometable* may be a pattern.
*   ``\q`` - quit the *psql* shell.
*   ``\i script.sql`` - read and execute the script file.
*   ``\c database`` - connect to a database.
*   ``\h`` - command list.
*   ``\h command`` - command help.
*   ``\copy table`` - runs an SQL COPY with  file accessibility and
    privileges of the local user.

You can also use from the shell:

..  code-block:: sh

    psql -d pgdatabase -U user -f script.sql # execute the script
    psql -l # database list


and get information from special ``SELECT`` commands::

  SELECT version();
  SELECT current_database();
  SELECT current_user ;
  SELECT user ;
  SELECT usename FROM pg_user; -- user list

TABLE OPERATIONS
----------------

CREATE TABLE
~~~~~~~~~~~~

::

    CREATE TABLE enrolment (
        StudentID NUMERIC(3),
        Name VARCHAR(30),
        CourseId NUMERIC(3),
        CONSTRAINT enrolment_pk PRIMARY KEY (StudentID, CourseId),
        CONSTRAINT enrolment_fk1 FOREIGN KEY (StudentId)
            REFERENCES Student(StudentId),
        CONSTRAINT enrolment_fk2 FOREIGN KEY (CourseId)
            REFERENCES Course(CourseId)
    );

DROP TABLE
~~~~~~~~~~
::

    DROP TABLE enrolment ;

constraints
~~~~~~~~~~~

References:
    :psql:`Manual: Constraints <ddl-constraints>`

CHECK constraint
++++++++++++++++

It allow that the values in some columns of each row satisfy a boolean
condition.

When refering to a unique column they can be given after the column
definition::

  CREATE TABLE products (
      product_no integer,
      name text,
      price numeric CHECK (price > 0)
  );

The system choose a name for your constraint, but you can provide it::

  CREATE TABLE products (
      product_no integer,
      name text,
      price numeric CONSTRAINT positive_price CHECK (price > 0)
  );

You can list the constraints after the columns definition, it is
mandatory if the constraint involve many columns.
::

    CREATE TABLE products (
        product_no integer,
        name text,
        price numeric,
        CHECK (price > 0),
        discounted_price numeric,
        CHECK (discounted_price > 0),
        CONSTRAINT valid_discount CHECK (price > discounted_price)
    );

NOT NULL constraints
++++++++++++++++++++

::

    CREATE TABLE products (
        product_no integer NOT NULL,
        name text NOT NULL,
        price numeric
    );


Unique Constraints
++++++++++++++++++

Unique constraints ensure that the data contained in a column or a
group of columns is unique with respect to all the rows in the table.

::

    CREATE TABLE products (
        product_no integer UNIQUE,
        name text,
        price numeric
    );

    CREATE TABLE example (
        a integer,
        b integer,
        c integer,
        UNIQUE (a, c)
    );

Unique constraints can also be named.

::

    CREATE TABLE products (
        product_no integer CONSTRAINT unique_procuctno UNIQUE,
        name text,
        price numeric
    );

Be aware that a unique constraint does not always provide a key for
the table. Two distincts rows can have th same unique tuple if one of
its column value is NULL.

Primary Key
+++++++++++

A primary key is only a combination of of a unique constraint and a
not-null constraint.

But while you can have many unique and not null tuples of columns in
one relation (named candidate keys in relational theory); you can can
choose only one primary key.

SQL does not enforce the rule of having one primary key for each
relation.

::

    CREATE TABLE products (
        product_no integer PRIMARY KEY,
        name text,
        price numeric
    );

Foreign Keys
++++++++++++

A foreign key constraint specifies that the values in a column (or a
group of columns) must match the values appearing in some row of
another table.

::

    CREATE TABLE orders (
        order_id integer PRIMARY KEY,
        product_no integer REFERENCES products (product_no),
        quantity integer
    );


As the default reference is the primary key you can also write

::

    CREATE TABLE orders (
        order_id integer PRIMARY KEY,
        product_no integer REFERENCES products,
        quantity integer
    );

A foreign key can also reference many columns

::

    CREATE TABLE t1 (
        a integer PRIMARY KEY,
        b integer,
        c integer,
        FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
    );

You can also have more than one foreign key constraint.

The foreign key forbid to create rows with values which are not in the
foreign relation. But one can delete rows from this foreign
relation, or change the referenced value, there are many policies to
answer to such events.

For deletion the policy is introduced by the keyword ``ON DELETE``,for
updating by ``ON UPDATE``.

RESTRICT prevents deletion of a referenced row.

The default behavior NO ACTION means that if any referencing rows
still exist when the constraint is checked, an error is raised.

With RESTRICT the checking of integrity is immediate, while it is
deferred with NO ACTION.

CASCADE specifies that when a referenced row is deleted, row(s)
referencing it should be automatically deleted as well. For an updated
row, it means that the updated values of the referenced columns should be copied into the referencing rows.

SET NULL and SET DEFAULT cause the referencing columns in the
referencing rows to be set to nulls or their default values,
respectively, when the referenced row is deleted. This NULL or DEFAULT
value has still to satisfy the relation constraints including foreign keys.

::

    CREATE TABLE products (
        product_no integer PRIMARY KEY,
        name text,
        price numeric
    );

    CREATE TABLE orders (
        order_id integer PRIMARY KEY,
        shipping_address text,
        ...
    );

    CREATE TABLE order_items (
        product_no integer REFERENCES products ON DELETE RESTRICT,
        order_id integer REFERENCES orders ON DELETE CASCADE,
        quantity integer,
        PRIMARY KEY (product_no, order_id)
    );

A foreign key does not prevent referencing columns to be null. To
avoid it you must declare the referencing columns as NOT NULL, this condition
is verified if they are a primary key of the foreign relation.

If null values are allowed adding  MATCH FULL to the foreign key
declaration forbid any mix of null and not null values in the
referencing columns.

It is useful  to index the referencing columns of a foreign key, but
is not automatic.

Exclusion Constraints
+++++++++++++++++++++

Exclusion constraints ensure that if any two rows are compared on the
specified columns or expressions using the specified operators, at
least one of these operator comparisons will return false or null.

::

    CREATE TABLE circles (
        c circle,
        EXCLUDE USING gist (c WITH &&)
    );

Postgresql built-in types.
~~~~~~~~~~~~~~~~~~~~~~~~~~
There are :psql:`numerous types <datatype>` among which:

*   CHAR[(n)] : fixed-length, blank padded character string.
*   VARCHAR[(n)] : variable-length with limit string.
*   NUMERIC[(precision, scale)]: decimal number
*   DOUBLE PRECISION,  REAL, INTEGER or INT, SMALLINT, BIGINT,
    SMALL SERIAL, SERIAL, BIG SERIAL: :psql:`numeric types <datatype-numeric>`.
*   MONEY: a :psql:`monetary <datatype-money>`
    decimal number with precision determined by  ``lc_monetary``.
*   TIMESTAMP or TIMESTAMP WITHOUT TIME ZONE, TIMESTAMP WITH TIME
    ZONE, TIME [ WITH  TIME ZONE| WITHOUT TIME ZONE], INTERVAL:
    :psql:`Date/Time Types <datatype-datetime>`
*   BOOLEAN :  :psql:`boolean <datatype-boolean>` values ``TRUE``, ``FALSE``
    or *unknown*.

ALTER TABLE
~~~~~~~~~~~
User to :psql:`Modify Tables <ddl-alter>`

::

    ALTER TABLE table_name ADD COLUMN new_column_name TYPE;
    ALTER TABLE table_name DROP COLUMN column_name;
    ALTER TABLE table_name RENAME COLUMN column_name TO new_column_name;
    ALTER TABLE table_name ALTER COLUMN [SET DEFAULT value | DROP DEFAULT]
    ALTER TABLE table_name ALTER COLUMN [SET NOT NULL| DROP NOT NULL]
    ALTER TABLE table_name ADD CHECK expression;
    ALTER TABLE table_name ADD CONSTRAINT constraint_name constraint_definition
    ALTER TABLE table_name RENAME TO new_table_name;

INSERT
~~~~~~

::

    INSERT INTO table(column1, column2, …)
    VALUES
        (value1, value2, …);

    INSERT INTO table (column1, column2, …)
    VALUES
        (value1, value2, …),
        (value1, value2, …) ,...;

;;  _update_clause:

UPDATE
~~~~~~
Ref: :psql:`Manual: UPDATE <dml-update>`

``UPDATE`` is used for updating existing data. The rows updated are
selected with a :psql:`WHERE clause <where_clause>`, and the operation
applies to all rows matching the condition.

::

    UPDATE products SET price = 10 WHERE price = 5;

updates all products that have a price of 5 to have a price of 10.

::

    UPDATE products SET price = price * 1.10;

raise the price of any product by 10%

::

    UPDATE mytable SET a = 5, b = 3, c = 1 WHERE a > 0;

change three colums in the same request.

DELETE
~~~~~~
It is a command very similar to the :psql:`UPDATE command
<update_clause>`.

::

    DELETE FROM products WHERE price = 10;

delete all products having a price of 10.

::

    DELETE FROM products;

delete all rows from products.

SELECT
~~~~~~
Ref: :psql:`Manual: SELECT <sql-select>`

::

    SELECT column_1,
        column_2,
        ...
    FROM table_name

SELECT DISTINCT
+++++++++++++++
The select operator has an implicit ``ALL`` allowing to have a result
with duplicated rows. We can ask for a set with ``DISTINCT``.

::


    SELECT DISTINCT column_1,
        column_2,
        ...
    FROM table_name


SELECT ORDER BY
+++++++++++++++
::

    SELECT column_1, column_2
    FROM tbl_name
    ORDER BY column_1 ASC, column_2 DESC;


The default is ASCending.

..  _where_clause:

SELECT WHERE
++++++++++++
::

    SELECT column_1, column_2 … column_n
    FROM table_name
    WHERE conditions

To express a condition see below :ref:`conditions`.


SELECT GROUP BY, HAVING
+++++++++++++++++++++++

::

    SELECT month, sum(sales)
    FROM salesfigures
    GROUP BY month;

    SELECT name, min(sales), max(sales)
    FROM salesfigures
    GROUP BY name;

    SELECT name
    FROM salesfigures
    GROUP BY name HAVING min(sales)<20;


UNION, INTERSECT, EXCEPT
~~~~~~~~~~~~~~~~~~~~~~~~

::

    SELECT * FROM r1
    UNION
    SELECT * FROM r2;

    SELECT * FROM r1
    INTERSECT
    SELECT * FROM r2;

    SELECT * FROM r1
    EXCEPT
    SELECT * FROM r2;

JOIN
~~~~

Cross join
++++++++++
::

    T1 CROSS JOIN T2

For every possible combination of rows from T1 and T2 (i.e., a
Cartesian product), the joined table will contain a row consisting
of all columns in T1 followed by all columns in T2. If the tables
have N and M rows respectively, the joined table will have N * M
rows.

``FROM T1 CROSS JOIN T2`` is equivalent to ``FROM T1, T2`` and to
``FROM T1 INNER JOIN T2 ON TRUE``.

INNER JOIN ON
+++++++++++++
::

    T1 INNER JOIN T2 ON boolean expression

The boolean expression is like the one used in :ref:`where clauses
<where_clause>` and uses  the same :ref:`conditions`.

For each row R1 of T1, the joined table has a row for each row in
T2 that satisfies the join condition with R1.

INNER JOIN USING
++++++++++++++++
::

    T1 INNER JOIN T2 USING ( join column list )

 USING takes a comma-separated list of column names, which must be
 common in ``T1`` and ``T2``, the output is formed by rows that
 include a row for each pair of row of T1 and T2 whe these columns match.

 The result rows contain first the join colum list with the common
 values, then the remaining columns of T1 and T2.

::

    T1 NATURAL INNER JOIN T2

is like a USING with all colums that have same name in ``T1`` and ``T2``.


OUTER JOINS
+++++++++++
::

    T1 {RIGHT|LEFT|FULL} [OUTER] JOIN T2 ON boolean expressionN boolean expression
    T1 {RIGHT|LEFT|FULL} [OUTER] JOIN T2 USING ( join column list )
    T1 NATURAL {RIGHT|LEFT|FULL} [OUTER] JOIN T2

In these operations the corresponding INNER join is first performed.

Then for LEFT OUTER JOIN a row is added for each column of T1 that
does not satisfy the condition, comporting extending the values in the
T1 row with NULL in the colums of T2.

For RIGHT OUTER JOIN non matching of T2 are completed with NULL
values, and for FULL OUTER join, both non matching rows of T1 and T2
are completed with NULL on the missing columns.

WITH Queries
~~~~~~~~~~~~
WITH provides a way to write auxiliary statements for use in a larger query.
Each auxiliary statement in a WITH clause can be a SELECT, INSERT,
UPDATE, or DELETE; and the WITH clause itself is attached to a primary
statement that can also be a SELECT, INSERT, UPDATE, or DELETE.

::

    WITH regional_sales AS (
            SELECT region, SUM(amount) AS total_sales
            FROM orders
            GROUP BY region
         ), top_regions AS (
            SELECT region
            FROM regional_sales
            WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
         )
    SELECT region,
           product,
           SUM(quantity) AS product_units,
           SUM(amount) AS product_sales
    FROM orders
    WHERE region IN (SELECT region FROM top_regions)
    GROUP BY region, product;


..  _conditions:

Conditions
~~~~~~~~~~

The are used in :ref:`where clauses <where_clause>`

comparison operators.
+++++++++++++++++++++

The conditional :psql:`comparison operators <functions-comparison>`
are ``=``, ``>``, ``<``, ``>=``, ``<=``, ``<>``  or ``!=``,
``BETWEEN``, ``IS NULL``, ``IS NOT NULL`` .

Logical operators
+++++++++++++++++

The :psql:`logical operators <functions-logical>` are ``AND``, ``OR``, ``NOT``

Row and array operators.
++++++++++++++++++++++++

The :psql:`row and array comparisons <functions-comparisons>` are
``IN``, ``NOT IN``, ``SOME`` or ``ANY``, ``ALL``, ``IS DISTINCT
FROM``, ``IS NOT DISTINCT FROM``.

First to compare a single value to a row or array
`````````````````````````````````````````````````

::

    expression IN (value [, ...])

    expression NOT IN (value [, ...])

    expression operator ANY (array expression)
    expression operator SOME (array expression)

    expression operator ALL (array expression)

    SELECT column_1, column_2
    FROM tbl_name
    where column_2 IN (value1,value2,...)

The array expression can also be a subquery.

Second to compare to compare to rows.
`````````````````````````````````````

The two row values must have the same number of fields.

::

    row_constructor operator row_constructor

    row_constructor IS DISTINCT FROM row_constructor

    row_constructor IS NOT DISTINCT FROM row_constructor


For the ``<``, ``<=``, ``>`` and ``>=`` cases, the row elements are compared
left-to-right, stopping as soon as a pair of elements wher the
comparison fail or containing a NULL is found.

::

    ROW(1,2,NULL) < ROW(2,1,0) // => false

    ROW(1,NULL,2) < ROW(2,0,1) // => NULL

For ``=`` and ``<>`` the comparison is NULL if one member contain a NULL, true
if all members are not null and are equals, false if all members are
not null and one pair differ.

The ``IS DISTINCT`` and ``IS NOT DISTINCT`` are similar to ``<>`` and
``=``, except that a comparison of a pair containing a NULL yield
fale. The result of these operators is never NULL.
