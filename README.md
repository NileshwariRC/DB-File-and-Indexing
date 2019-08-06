# Database-Files-and-Indexing
Project include a relational database supporting all the DML and DDL operations in Java.

Code is written in Java and developed on Eclipse with Java 8

OVERVIEW
========
1. It is a rudimentary database engine that is based on a simplified file-per-table variation on the SQLite file format.
2. Each database table is physically stored as a separate single file and each table file is subdivided into logical sections of fixed equal size call pages
3. Page_size is a configurable attribute set to 512 Byte
4. Each page in a Table file is a node in a B+ tree, either interior or leaf
5. The location of each element in a page is referenced with a “page offset” value 

NOTE: This database is not fully ACID compliant!

FILE STORAGE FORMAT
===================
1. Two system tables created by default 'master_tables' and 'master_columns', which encode the database schema information, i.e. the “database catalog” or “meta-data” to eliminate the need to parse the SQL “CREATE TABLE” text every time you need to insert a row. 
2. All files are stored in directory named 'data' having two sub-directories, catalog and user_data

/data
|
+-/catalog
| |
| +-/master_tables.tbl
| +-/master_columns.tbl
|
+-/user_data
 |
  +-/table_name_1.tbl
  +-/table_name_2.tbl
  +-/table_name_3.tbl
  etc.
  
SUPPORTED COMMANDS
==================
SHOW TABLES;                                                                        Displays a list of all tables in the Database.
CREATE TABLE table_name(row_id INT PRIMARY KEY, name text);                        Create a new table schema if not already exist.
DROP TABLE table_name;                                                              Remove table data and its schema.
SELECT * FROM table_name;                                                           Display all records in the table.
SELECT * FROM table_name WHERE <column_name> = <value>;                             Display records whose column is <value>.
INSERT INTO <table_name> VALUES (value1, value2, ...);                              Insert a new record into the indicated table.
DELETE FROM <table_name> WHERE row_id = key_value;                                  Delete a single row/record from a table given the row_id primary key.
UPDATE <table_name> SET column_name = value WHERE row_id = value;                   Modifies one or more records in a table.
VERSION;                                                                            Show the program version.
HISTORY;                                                                            Show all recent commands.
HELP;                                                                               Show this help information
EXIT;                                                                               Exit the program

NOTE:
1. Joins on multiple tables is NOT included
2. Update operation is performed with the same size of value to be updated. (As B+Tree works Byte-by-Byte, previous bytes will be replaced)
3. row_id INT PRIMARY KEY should be specified as it is in a create query

B+TREE for TABLE FILES - INTERNAL DETAILS
=========================================
B-tree Pages
------------
1. Page Header 
--------------
All table pages have an 8-byte page header, immediately followed by a sequence of 2-byte integers that are the page offset location for each data cell

Offset from beginning of page	Content Size(bytes)		 Description
	0x00							1					The one-byte ﬂag at offset 0 indicating the b-tree page type. 
														• A value of 2 (0x02) means the page is an interior index b-tree page. 
														• A value of 5 (0x05) means the page is an interior table b-tree page. 
														• A value of 10 (0x0a) means the page is a leaf index b-tree page. 
														• A value of 13 (0x0d) means the page is a leaf table b-tree page. 
                              Any other value for the b-tree page type is an error. 
	
	0x01 							1				• The one-byte two’s complement signed integer at offset 1 gives the number of cells on the page. 
	
	0x02							2				• The two-byte two’s complement signed integer at offset 2 designates the start of the cell content area. 
														• A zero value for this	integer is interpreted as 65536
	
	0x04 							4					The four-byte signed integer page pointer at offset 4 has a different role depending on the page type: 
														• Table or Index interior page - rightmost child page number reference. 
														• Table leaf page - right sibling page number reference. 
														• Index leaf page - unused

	
	0x08 							2n			• An array of 2-byte integers that indicate the page offset location of each data cell. 
														• The array size is 2n, where n is the number of cells on the page. The array is maintained in key-sorted order—i.e. rowid order for a table ﬁle and index order for an index ﬁle.

2. Data Cells
-------------
Table data cells are located at the bottom of each page. Data cell content is different for leaf page versus interior pages. Leaf page data cells contain the table’s records. Interior page data cells will contain only keys and left child page pointers. 

3. Offsets 
----------
Offsets are the mechanism used to locate elements within a file. Offsets are expressed as non-negative 2 integers that represent the number of bytes from a given point of reference. There are two kinds of offsets used by DavisBase, file offsets and page offsets. 

File offsets indicate the location of an element expressed as the number of bytes from the beginning of a file. These are most commonly used to identify the location of a page within a file—by multiplying page number time page size.  

Page offsets indicate the location of an element expressed as the number of bytes from the beginning of a page. 

Therefore the location of a specific page within a file may be determined by the simple calculation page_number * page_size.  

For example, a file with a page size of 512 bytes, page 7 will begin at 512 * 7 = 3584 bytes from the beginning of the file.

FILE DESCRIPTION
================
1. WelcomePage.java
----------------
Takes care of the user display prompt and version.

2. Utility.java
------------
Contains all the functions accessed across different classes in the project. 
Includes the functions for parsing input string and converting it to a desired format, access paths for the files stored in the project, converting input data into a data structure used by the program for representing records and identifying input data for appropriate parsing.

3. QueryHandler.java
-----------------
Deals with display of table logic, ’show table’ query, ’drop table’ query, ’update’ query and display of the help menu.

4. ParseQuery.java
---------------
Parses the input command from the command prompt, and calls the appropriate function(query) depending on the keywords recognized in the command. 
Also extracts data from the input string that will be required to pass as arguments to the corresponding function. Individual functions have been defined for different queries. The queries dealt with from this class are Parse Create command, parse create Index command, parse Insert command, parse ‘show’ keyword, parse update command, parse delete command.

5. MyDatabase.java
---------------
Main class for our project. Takes care of building internal representations of the database and the columns data (This project uses a hashmap for the internal representation of records in the table.

6. Btree.java
----------
Deals with all operations associated with the ‘b+’ trees. It has functions for the actual working of every query defined in the system at the file level.
Creates and deletes internal nodes of the tree, creates and deletes leaf nodes of the tree , initializes the tree, checks for keys on columns of the table, searches the tree to find/ read individual records and their values,and maintains the page headers on every operation
