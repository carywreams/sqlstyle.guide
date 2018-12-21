# SQL style guide

## Overview

You can use this set of guidelines, [fork them][fork] or make your own - the
key here is that you pick a style and stick to it. To suggest changes
or fix bugs please open an [issue][] or [pull request][pull] on GitHub.

These guidelines are designed to be compatible with Joe Celko's [SQL Programming
Style][celko] book to make adoption for teams who have already read that book
easier. This guide is a little more opinionated in some areas and in others a
little more relaxed. It is certainly more succinct where [Celko's book][celko]
contains anecdotes and reasoning behind each rule as thoughtful prose.

It is easy to include this guide in [Markdown format][dl-md] as a part of a
project's code base or reference it here for anyone on the project to freely
read—much harder with a physical book.

SQL style guide by [Simon Holywell][simon] is licensed under a [Creative Commons
Attribution-ShareAlike 4.0 International License][licence].
Based on a work at [http://www.sqlstyle.guide][self].

## General

### Do

* Use consistent and descriptive identifiers and names.
* Make judicious use of white space and indentation to make code easier to read.
* Store [ISO-8601][iso-8601] compliant time and date information
  (`YYYY-MM-DD HH:MM:SS.SSSSS`).
* Try to use only standard SQL functions instead of vendor specific functions for
  reasons of portability.
* Keep code succinct and devoid of redundant SQL—such as unnecessary quoting or
  parentheses or `WHERE` clauses that can otherwise be derived.
* Include comments in SQL code where necessary. Use the C style opening `/*` and
  closing `*/` where possible otherwise preceed comments with `--` and finish
  them with a new line.

```sql
SELECT file_hash  -- stored ssdeep hash
  FROM file_system
 WHERE file_name = '.vimrc';
```
```sql
/* Updating the file record after writing to the file */
UPDATE file_system
   SET file_modified_date = '1980-02-22 13:19:01.00000',
       file_size = 209732
 WHERE file_name = '.vimrc';
```

### Avoid

* CamelCase—it is difficult to scan quickly.
* Descriptive prefixes or Hungarian notation such as `sp_` or `tbl`.
* Plurals—use the singular ~~more natural collective term where possible instead. For example
  `staff` instead of `employees` or `people` instead of `individuals`.~~ ```-cwr```
* Quoted identifiers—if you must use them then stick to SQL92 double quotes for
  portability (you may need to configure your SQL server to support this depending
  on vendor).
* Object oriented design principles should not be applied to SQL or database
  structures.

## Naming conventions

### General

* Ensure the name is unique and does not exist as a
  [reserved keyword][reserved-keywords].
* Keep the length to a maximum of 30 bytes—in practice this is 30 characters
  unless you are using multi-byte character set.
* Names must begin with a letter and may not end with an underscore.
* Only use letters, numbers and underscores in names.
* Avoid the use of multiple consecutive underscores—these can be hard to read.
* Use underscores where you would naturally include a space in the name (first
  name becomes `first_name`).
* Avoid abbreviations and if you have to use them make sure they are commonly
  understood.

```sql
SELECT first_name
  FROM staff;
```

### Tables

* Use the singular form, employee. ~~Use a collective name or, less ideally, a plural form. For example (in order of
  preference) `staff` and `employees`.~~  ```-cwr```
* Do not prefix with `tbl` or any other such descriptive prefix or Hungarian
  notation.
* Never give a table the same name as one of its columns and vice versa.
* On Associations: ~~Avoid, where possible, concatenating two table names together to create the name
  of a relationship table. Rather than `cars_mechanics` prefer `services`.~~ ```-cwr```
    * concatenating (with an underscore) is acceptable
    * using a connector (memberof, with, etc) is acceptable if needed to add clarification

### Columns

* Always use the singular name.
* Where possible avoid simply using `id` as the primary identifier for the table.
* Do not add a column with the same name as its table and vice versa.
* Always use lowercase except where it may make sense not to such as proper nouns.

### Aliasing or correlations

* Should relate in some way to the object or expression they are aliasing.
* As a rule of thumb the correlation name should be the first letter of each word
  in the object's name.
* If there is already a correlation with the same name then append a number.
* Always include the `AS` keyword—makes it easier to read as it is explicit.
* For computed data (`SUM()` or `AVG()`) use the name you would give it were it
  a column defined in the schema.

```sql
SELECT first_name AS fn
  FROM staff AS s1
  JOIN students AS s2
    ON s2.mentor_id = s1.staff_num;
```
```sql
SELECT SUM(s.monitor_tally) AS monitor_total
  FROM staff AS s;
```

### Stored procedures

* The name must contain a verb.
* Do not prefix with `sp_` or any other such descriptive prefix or Hungarian
  notation.
* For p4, see my notes on store procedures ```-cwr```

### Uniform suffixes

The following suffixes have a universal meaning ensuring the columns can be read
and understood easily from SQL code. Use the correct suffix where appropriate.

* `_id`—a unique identifier such as a column that is a primary key.
* `_status`—flag value or some other status of any type such as
  `publication_status`.
* `_total`—the total or sum of a collection of values.
* `_num`—denotes the field contains any kind of number.
* `_name`—signifies a name such as `first_name`.
* `_seq`—contains a contiguous sequence of values.
* `_date`—denotes a column that contains the date of something.
* `_tally`—a count.
* `_size`—the size of something such as a file size or clothing.
* `_addr`—an address for the record could be physical or intangible such as
  `ip_addr`.

## Query syntax

### Reserved words

Always use uppercase for the [reserved keywords][reserved-keywords]
like `SELECT` and `WHERE`.

It is best to avoid the abbreviated keywords and use the full length ones where
available (prefer `ABSOLUTE` to `ABS`).

Do not use database server specific keywords where an ANSI SQL keyword already
exists performing the same function. This helps to make code more portable.

```sql
SELECT model_num
  FROM phones AS p
 WHERE p.release_date > '2014-09-30';
```

### White space

To make the code easier to read it is important that the correct compliment of
spacing is used. Do not crowd code or remove natural language spaces.

#### Spaces

Spaces should be used to line up the code so that the root keywords all end on
the same character boundary. This forms a river down the middle making it easy for
the readers eye to scan over the code and separate the keywords from the
implementation detail. Rivers are [bad in typography][rivers], but helpful here.

```sql
SELECT f.average_height, f.average_diameter
  FROM flora AS f
 WHERE f.species_name = 'Banksia'
    OR f.species_name = 'Sheoak'
    OR f.species_name = 'Wattle';
```

Notice that `SELECT`, `FROM`, etc. are all right aligned while the actual column
names and implementation specific details are left aligned.

Although not exhaustive always include spaces:

* before and after equals (`=`)
* after commas (`,`)
* surrounding apostrophes (`'`) where not within parentheses or with a trailing
  comma or semicolon.

```sql
SELECT a.title, a.release_date, a.recording_date
  FROM albums AS a
 WHERE a.title = 'Charcoal Lane'
    OR a.title = 'The New Danger';
```

#### Line spacing

Always include newlines/vertical space:

* before `AND` or `OR`
* after semicolons to separate queries for easier reading
* after each keyword definition
* after a comma when separating multiple columns into logical groups
* to separate code into related sections, which helps to ease the readability of
  large chunks of code.

Keeping all the keywords aligned to the righthand side and the values left aligned
creates a uniform gap down the middle of query. It makes it much easier to scan
the query definition over quickly too.

```sql
INSERT INTO albums (title, release_date, recording_date)
VALUES ('Charcoal Lane', '1990-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000'),
       ('The New Danger', '2008-01-01 01:01:01.00000', '1990-01-01 01:01:01.00000');
```

```sql
UPDATE albums
   SET release_date = '1990-01-01 01:01:01.00000'
 WHERE title = 'The New Danger';
```

```sql
SELECT a.title,
       a.release_date, a.recording_date, a.production_date -- grouped dates together
  FROM albums AS a
 WHERE a.title = 'Charcoal Lane'
    OR a.title = 'The New Danger';
```

### Indentation

To ensure that SQL is readable it is important that standards of indentation
are followed.

#### Joins

Joins should be indented to the other side of the river and grouped with a new
line where necessary.

```sql
SELECT r.last_name
  FROM riders AS r
       INNER JOIN bikes AS b
       ON r.bike_vin_num = b.vin_num
          AND b.engines > 2

       INNER JOIN crew AS c
       ON r.crew_chief_last_name = c.last_name
          AND c.chief = 'Y';
```

#### Subqueries

Subqueries should also be aligned to the right side of the river and then laid
out using the same style as any other query. Sometimes it will make sense to have
the closing parenthesis on a new line at the same character position as it's
opening partner—this is especially true where you have nested subqueries.

```sql
SELECT r.last_name,
       (SELECT MAX(YEAR(championship_date))
          FROM champions AS c
         WHERE c.last_name = r.last_name
           AND c.confirmed = 'Y') AS last_championship_year
  FROM riders AS r
 WHERE r.last_name IN
       (SELECT c.last_name
          FROM champions AS c
         WHERE YEAR(championship_date) > '2008'
           AND c.confirmed = 'Y');
```

### Preferred formalisms

* Make use of `BETWEEN` where possible instead of combining multiple statements
  with `AND`.
* Similarly use `IN()` instead of multiple `OR` clauses.
* Where a value needs to be interpreted before leaving the database use the `CASE`
  expression. `CASE` statements can be nested to form more complex logical structures.
* Avoid the use of `UNION` clauses and temporary tables where possible. If the
  schema can be optimised to remove the reliance on these features then it most
  likely should be.

```sql
SELECT CASE postcode
       WHEN 'BN1' THEN 'Brighton'
       WHEN 'EH1' THEN 'Edinburgh'
       END AS city
  FROM office_locations
 WHERE country = 'United Kingdom'
   AND opening_time BETWEEN 8 AND 9
   AND postcode IN ('EH1', 'BN1', 'NN1', 'KW1')
```

## Create syntax

When declaring schema information it is also important to maintain human
readable code. To facilitate this ensure the column definitions are ordered and
grouped where it makes sense to do so.

Indent column definitions by four (4) spaces within the `CREATE` definition.

### Choosing data types

* Where possible do not use vendor specific data types—these are not portable and
  may not be available in older versions of the same vendor's software.
* Only use `REAL` or `FLOAT` types where it is strictly necessary for floating
  point mathematics otherwise prefer `NUMERIC` and `DECIMAL` at all times. Floating
  point rounding errors are a nuisance!

### Specifying default values

* The default value must be the same type as the column—if a column is declared
  a `DECIMAL` do not provide an `INTEGER` default value.
* Default values must follow the data type declaration and come before any
  `NOT NULL` statement.

### Constraints and keys

Constraints and their subset, keys, are a very important component of any
database definition. They can quickly become very difficult to read and reason
about though so it is important that a standard set of guidelines are followed.

#### Choosing keys

Deciding the column(s) that will form the keys in the definition should be a 
carefully considered activity as it will effect performance and data integrity.

1. The key should be unique to some degree.
2. Consistency in terms of data type for the value across the schema and a lower
   likelihood of this changing in the future.
3. Can the value be validated against a standard format (such as one published by
   ISO)? Encouraging conformity to point 2.
4. Keeping the key as simple as possible whilst not being scared to use compound
   keys where necessary.

It is a reasoned and considered balancing act to be performed at the definition
of a database. Should requirements evolve in the future it is possible to make
changes to the definitions to keep them up to date.

#### Defining constraints

Once the keys are decided it is possible to define them in the system using
constraints along with field value validation.

##### General

* Tables must have at least one key to be complete and useful.
* Constraints should be given a custom name excepting `UNIQUE`, `PRIMARY KEY`
  and `FOREIGN KEY` where the database vendor will generally supply sufficiently
  intelligible names automatically.

##### Layout and order

* Specify the primary key first right after the `CREATE TABLE` statement.
* Constraints should be defined directly beneath the column they correspond to.
  Indent the constraint so that it aligns to the right of the column name.
* If it is a multi-column constraint then consider putting it as close to both
  column definitions as possible and where this is difficult as a last resort
  include them at the end of the `CREATE TABLE` definition.
* If it is a table level constraint that applies to the entire table then it
  should also appear at the end.
* Use alphabetical order where `ON DELETE` comes before `ON UPDATE`.
* If it make senses to do so align each aspect of the query on the same character
  position. For example all `NOT NULL` definitions could start at the same
  character position. This is not hard and fast, but it certainly makes the code
  much easier to scan and read.

##### Validation

* Use `LIKE` and `SIMILAR TO` constraints to ensure the integrity of strings
  where the format is known.
* Where the ultimate range of a numerical value is known it must be written as a
  range `CHECK()` to prevent incorrect values entering the database or the silent
  truncation of data too large to fit the column definition. In the least it
  should check that the value is greater than zero in most cases.
* `CHECK()` constraints should be kept in separate clauses to ease debugging.

##### Example

```sql
CREATE TABLE staff (
    PRIMARY KEY (staff_num),
    staff_num      INT(5)       NOT NULL,
    first_name     VARCHAR(100) NOT NULL,
    pens_in_drawer INT(2)       NOT NULL,
                   CONSTRAINT pens_in_drawer_range
                   CHECK(pens_in_drawer >= 1 AND pens_in_drawer < 100)
);
```

### Designs to avoid

* Object oriented design principles do not effectively translate to relational
  database designs—avoid this pitfall.
* Placing the value in one column and the units in another column. The column
  should make the units self evident to prevent the requirement to combine
  columns again later in the application. Use `CHECK()` to ensure valid data is
  inserted into the column.
* [EAV (Entity Attribute Value)][eav] tables—use a specialist product intended for
  handling such schema-less data instead.
* Splitting up data that should be in one table across many because of arbitrary
  concerns such as time-based archiving or location in a multi-national
  organisation. Later queries must then work across multiple tables with `UNION`
  rather than just simply querying one table.


## Appendix

### Reserved keyword reference

A list of ANSI SQL (92, 99 and 2003), MySQL 3 to 5.x, PostgreSQL 8.1, MS SQL Server 2000, MS ODBC and Oracle 10.2 reserved keywords.

keyword|keyword|keyword|keyword
-------|-------|-------|-------
A|DOMAIN|MINUTE|SCHEMAS
ABORT|DOUBLE|MINUTE_MICROSECOND|SCOPE
ABS|DROP|MINUTE_SECOND|SCOPE_CATALOG
ABSOLUTE|DUAL|MINVALUE|SCOPE_NAME
ACCESS|DUMMY|MLSLABEL|SCOPE_SCHEMA
ACTION|DUMP|MOD|SCROLL
ADA|DYNAMIC|MODE|SEARCH
ADD|DYNAMIC_FUNCTION|MODIFIES|SECOND
ADMIN|DYNAMIC_FUNCTION_CODE|MODIFY|SECOND_MICROSECOND
AFTER|EACH|MODULE|SECTION
AGGREGATE|ELEMENT|MONTH|SECURITY
ALIAS|ELSE|MONTHNAME|SELECT
ALL|ELSEIF|MORE|SELF
ALLOCATE|ENABLE|MOVE|SENSITIVE
ALSO|ENCLOSED|MULTISET|SEPARATOR
ALTER|ENCODING|MUMPS|SEQUENCE
ALWAYS|ENCRYPTED|MYISAM|SERIALIZABLE
ANALYSE|END|NAME|SERVER_NAME
ANALYZE|END-EXEC|NAMES|SESSION
AND|ENUM|NATIONAL|SESSION_USER
ANY|EQUALS|NATURAL|SET
ARE|ERRLVL|NCHAR|SETOF
ARRAY|ESCAPE|NCLOB|SETS
AS|ESCAPED|NESTING|SETUSER
ASC|EVERY|NEW|SHARE
ASENSITIVE|EXCEPT|NEXT|SHOW
ASSERTION|EXCEPTION|NO|SHUTDOWN
ASSIGNMENT|EXCLUDE|NO_WRITE_TO_BINLOG|SIGNAL
ASYMMETRIC|EXCLUDING|NOAUDIT|SIMILAR
AT|EXCLUSIVE|NOCHECK|SIMPLE
ATOMIC|EXEC|NOCOMPRESS|SIZE
ATTRIBUTE|EXECUTE|NOCREATEDB|SMALLINT
ATTRIBUTES|EXISTING|NOCREATEROLE|SOME
AUDIT|EXISTS|NOCREATEUSER|SONAME
AUTHORIZATION|EXIT|NOINHERIT|SOURCE
AUTO_INCREMENT|EXP|NOLOGIN|SPACE
AVG|EXPLAIN|NONCLUSTERED|SPATIAL
AVG_ROW_LENGTH|EXTERNAL|NONE|SPECIFIC
BACKUP|EXTRACT|NORMALIZE|SPECIFIC_NAME
BACKWARD|FALSE|NORMALIZED|SPECIFICTYPE
BEFORE|FETCH|NOSUPERUSER|SQL
BEGIN|FIELDS|NOT|SQL_BIG_RESULT
BERNOULLI|FILE|NOTHING|SQL_BIG_SELECTS
BETWEEN|FILLFACTOR|NOTIFY|SQL_BIG_TABLES
BIGINT|FILTER|NOTNULL|SQL_CALC_FOUND_ROWS
BINARY|FINAL|NOWAIT|SQL_LOG_OFF
BIT|FIRST|NULL|SQL_LOG_UPDATE
BIT_LENGTH|FLOAT|NULLABLE|SQL_LOW_PRIORITY_UPDATES
BITVAR|FLOAT4|NULLIF|SQL_SELECT_LIMIT
BLOB|FLOAT8|NULLS|SQL_SMALL_RESULT
BOOL|FLOOR|NUMBER|SQL_WARNINGS
BOOLEAN|FLUSH|NUMERIC|SQLCA
BOTH|FOLLOWING|OBJECT|SQLCODE
BREADTH|FOR|OCTET_LENGTH|SQLERROR
BREAK|FORCE|OCTETS|SQLEXCEPTION
BROWSE|FOREIGN|OF|SQLSTATE
BULK|FORTRAN|OFF|SQLWARNING
BY|FORWARD|OFFLINE|SQRT
C|FOUND|OFFSET|SSL
CACHE|FREE|OFFSETS|STABLE
CALL|FREETEXT|OIDS|START
CALLED|FREETEXTTABLE|OLD|STARTING
CARDINALITY|FREEZE|ON|STATE
CASCADE|FROM|ONLINE|STATEMENT
CASCADED|FULL|ONLY|STATIC
CASE|FULLTEXT|OPEN|STATISTICS
CAST|FUNCTION|OPENDATASOURCE|STATUS
CATALOG|FUSION|OPENQUERY|STDDEV_POP
CATALOG_NAME|G|OPENROWSET|STDDEV_SAMP
CEIL|GENERAL|OPENXML|STDIN
CEILING|GENERATED|OPERATION|STDOUT
CHAIN|GET|OPERATOR|STORAGE
CHANGE|GLOBAL|OPTIMIZE|STRAIGHT_JOIN
CHAR|GO|OPTION|STRICT
CHAR_LENGTH|GOTO|OPTIONALLY|STRING
CHARACTER|GRANT|OPTIONS|STRUCTURE
CHARACTER_LENGTH|GRANTED|OR|STYLE
CHARACTER_SET_CATALOG|GRANTS|ORDER|SUBCLASS_ORIGIN
CHARACTER_SET_NAME|GREATEST|ORDERING|SUBLIST
CHARACTER_SET_SCHEMA|GROUP|ORDINALITY|SUBMULTISET
CHARACTERISTICS|GROUPING|OTHERS|SUBSTRING
CHARACTERS|HANDLER|OUT|SUCCESSFUL
CHECK|HAVING|OUTER|SUM
CHECKED|HEADER|OUTFILE|SUPERUSER
CHECKPOINT|HEAP|OUTPUT|SYMMETRIC
CHECKSUM|HIERARCHY|OVER|SYNONYM
CLASS|HIGH_PRIORITY|OVERLAPS|SYSDATE
CLASS_ORIGIN|HOLD|OVERLAY|SYSID
CLOB|HOLDLOCK|OVERRIDING|SYSTEM
CLOSE|HOST|OWNER|SYSTEM_USER
CLUSTER|HOSTS|PACK_KEYS|TABLE
CLUSTERED|HOUR|PAD|TABLE_NAME
COALESCE|HOUR_MICROSECOND|PARAMETER|TABLES
COBOL|HOUR_MINUTE|PARAMETER_MODE|TABLESAMPLE
COLLATE|HOUR_SECOND|PARAMETER_NAME|TABLESPACE
COLLATION|IDENTIFIED|PARAMETER_ORDINAL_POSITION|TEMP
COLLATION_CATALOG|IDENTITY|PARAMETER_SPECIFIC_CATALOG|TEMPLATE
COLLATION_NAME|IDENTITY_INSERT|PARAMETER_SPECIFIC_NAME|TEMPORARY
COLLATION_SCHEMA|IDENTITYCOL|PARAMETER_SPECIFIC_SCHEMA|TERMINATE
COLLECT|IF|PARAMETERS|TERMINATED
COLUMN|IGNORE|PARTIAL|TEXT
COLUMN_NAME|ILIKE|PARTITION|TEXTSIZE
COLUMNS|IMMEDIATE|PASCAL|THAN
COMMAND_FUNCTION|IMMUTABLE|PASSWORD|THEN
COMMAND_FUNCTION_CODE|IMPLEMENTATION|PATH|TIES
COMMENT|IMPLICIT|PCTFREE|TIME
COMMIT|IN|PERCENT|TIMESTAMP
COMMITTED|INCLUDE|PERCENT_RANK|TIMEZONE_HOUR
COMPLETION|INCLUDING|PERCENTILE_CONT|TIMEZONE_MINUTE
COMPRESS|INCREMENT|PERCENTILE_DISC|TINYBLOB
COMPUTE|INDEX|PLACING|TINYINT
CONDITION|INDICATOR|PLAN|TINYTEXT
CONDITION_NUMBER|INFILE|PLI|TO
CONNECT|INFIX|POSITION|TOAST
CONNECTION|INHERIT|POSTFIX|TOP
CONNECTION_NAME|INHERITS|POWER|TOP_LEVEL_COUNT
CONSTRAINT|INITIAL|PRECEDING|TRAILING
CONSTRAINT_CATALOG|INITIALIZE|PRECISION|TRAN
CONSTRAINT_NAME|INITIALLY|PREFIX|TRANSACTION
CONSTRAINT_SCHEMA|INNER|PREORDER|TRANSACTION_ACTIVE
CONSTRAINTS|INOUT|PREPARE|TRANSACTIONS_COMMITTED
CONSTRUCTOR|INPUT|PREPARED|TRANSACTIONS_ROLLED_BACK
CONTAINS|INSENSITIVE|PRESERVE|TRANSFORM
CONTAINSTABLE|INSERT|PRIMARY|TRANSFORMS
CONTINUE|INSERT_ID|PRINT|TRANSLATE
CONVERSION|INSTANCE|PRIOR|TRANSLATION
CONVERT|INSTANTIABLE|PRIVILEGES|TREAT
COPY|INSTEAD|PROC|TRIGGER
CORR|INT|PROCEDURAL|TRIGGER_CATALOG
CORRESPONDING|INT1|PROCEDURE|TRIGGER_NAME
COUNT|INT2|PROCESS|TRIGGER_SCHEMA
COVAR_POP|INT3|PROCESSLIST|TRIM
COVAR_SAMP|INT4|PUBLIC|TRUE
CREATE|INT8|PURGE|TRUNCATE
CREATEDB|INTEGER|QUOTE|TRUSTED
CREATEROLE|INTERSECT|RAID0|TSEQUAL
CREATEUSER|INTERSECTION|RAISERROR|TYPE
CROSS|INTERVAL|RANGE|UESCAPE
CSV|INTO|RANK|UID
CUBE|INVOKER|RAW|UNBOUNDED
CUME_DIST|IS|READ|UNCOMMITTED
CURRENT|ISAM|READS|UNDER
CURRENT_DATE|ISNULL|READTEXT|UNDO
CURRENT_DEFAULT_TRANSFORM_GROUP|ISOLATION|REAL|UNENCRYPTED
CURRENT_PATH|ITERATE|RECHECK|UNION
CURRENT_ROLE|JOIN|RECONFIGURE|UNIQUE
CURRENT_TIME|K|RECURSIVE|UNKNOWN
CURRENT_TIMESTAMP|KEY|REF|UNLISTEN
CURRENT_TRANSFORM_GROUP_FOR_TYPE|KEY_MEMBER|REFERENCES|UNLOCK
CURRENT_USER|KEY_TYPE|REFERENCING|UNNAMED
CURSOR|KEYS|REGEXP|UNNEST
CURSOR_NAME|KILL|REGR_AVGX|UNSIGNED
CYCLE|LANCOMPILER|REGR_AVGY|UNTIL
DATA|LANGUAGE|REGR_COUNT|UPDATE
DATABASE|LARGE|REGR_INTERCEPT|UPDATETEXT
DATABASES|LAST|REGR_R2|UPPER
DATE|LAST_INSERT_ID|REGR_SLOPE|USAGE
DATETIME|LATERAL|REGR_SXX|USE
DATETIME_INTERVAL_CODE|LEADING|REGR_SXY|USER
DATETIME_INTERVAL_PRECISION|LEAST|REGR_SYY|USER_DEFINED_TYPE_CATALOG
DAY|LEAVE|REINDEX|USER_DEFINED_TYPE_CODE
DAY_HOUR|LEFT|RELATIVE|USER_DEFINED_TYPE_NAME
DAY_MICROSECOND|LENGTH|RELEASE|USER_DEFINED_TYPE_SCHEMA
DAY_MINUTE|LESS|RELOAD|USING
DAY_SECOND|LEVEL|RENAME|UTC_DATE
DAYOFMONTH|LIKE|REPEAT|UTC_TIME
DAYOFWEEK|LIMIT|REPEATABLE|UTC_TIMESTAMP
DAYOFYEAR|LINENO|REPLACE|VACUUM
DBCC|LINES|REPLICATION|VALID
DEALLOCATE|LISTEN|REQUIRE|VALIDATE
DEC|LN|RESET|VALIDATOR
DECIMAL|LOAD|RESIGNAL|VALUE
DECLARE|LOCAL|RESOURCE|VALUES
DEFAULT|LOCALTIME|RESTART|VAR_POP
DEFAULTS|LOCALTIMESTAMP|RESTORE|VAR_SAMP
DEFERRABLE|LOCATION|RESTRICT|VARBINARY
DEFERRED|LOCATOR|RESULT|VARCHAR
DEFINED|LOCK|RETURN|VARCHAR2
DEFINER|LOGIN|RETURNED_CARDINALITY|VARCHARACTER
DEGREE|LOGS|RETURNED_LENGTH|VARIABLE
DELAY_KEY_WRITE|LONG|RETURNED_OCTET_LENGTH|VARIABLES
DELAYED|LONGBLOB|RETURNED_SQLSTATE|VARYING
DELETE|LONGTEXT|RETURNS|VERBOSE
DELIMITER|LOOP|REVOKE|VIEW
DELIMITERS|LOW_PRIORITY|RIGHT|VOLATILE
DENSE_RANK|LOWER|RLIKE|WAITFOR
DENY|M|ROLE|WHEN
DEPTH|MAP|ROLLBACK|WHENEVER
DEREF|MATCH|ROLLUP|WHERE
DERIVED|MATCHED|ROUTINE|WHILE
DESC|MAX|ROUTINE_CATALOG|WIDTH_BUCKET
DESCRIBE|MAX_ROWS|ROUTINE_NAME|WINDOW
DESCRIPTOR|MAXEXTENTS|ROUTINE_SCHEMA|WITH
DESTROY|MAXVALUE|ROW|WITHIN
DESTRUCTOR|MEDIUMBLOB|ROW_COUNT|WITHOUT
DETERMINISTIC|MEDIUMINT|ROW_NUMBER|WORK
DIAGNOSTICS|MEDIUMTEXT|ROWCOUNT|WRITE
DICTIONARY|MEMBER|ROWGUIDCOL|WRITETEXT
DISABLE|MERGE|ROWID|X509
DISCONNECT|MESSAGE_LENGTH|ROWNUM|XOR
DISK|MESSAGE_OCTET_LENGTH|ROWS|YEAR
DISPATCH|MESSAGE_TEXT|RULE|YEAR_MONTH
DISTINCT|METHOD|SAVE|ZEROFILL
DISTINCTROW|MIDDLEINT|SAVEPOINT|ZONE
DISTRIBUTED|MIN|SCALE
DIV|MIN_ROWS|SCHEMA
DO|MINUS|SCHEMA_NAME

[simon]: https://www.simonholywell.com/?utm_source=sqlstyle.guide&utm_medium=link&utm_campaign=md-document
    "SimonHolywell.com"
[issue]: https://github.com/treffynnon/sqlstyle.guide/issues
    "SQL style guide issues on GitHub"
[fork]: https://github.com/treffynnon/sqlstyle.guide/fork
    "Fork SQL style guide on GitHub"
[pull]: https://github.com/treffynnon/sqlstyle.guide/pulls/
    "SQL style guide pull requests on GitHub"
[celko]: https://www.amazon.com/gp/product/0120887975/ref=as_li_ss_tl?ie=UTF8&linkCode=ll1&tag=treffynnon-20&linkId=9c88eac8cd420e979675c815771313d5
    "Joe Celko's SQL Programming Style (The Morgan Kaufmann Series in Data Management Systems)"
[dl-md]: https://raw.githubusercontent.com/treffynnon/sqlstyle.guide/gh-pages/_includes/sqlstyle.guide.md
    "Download the guide in Markdown format"
[iso-8601]: https://en.wikipedia.org/wiki/ISO_8601
    "Wikipedia: ISO 8601"
[rivers]: http://practicaltypography.com/one-space-between-sentences.html
    "Practical Typography: one space between sentences"
[reserved-keywords]: #reserved-keyword-reference
    "Reserved keyword reference" 
[eav]: https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model
    "Wikipedia: Entity–attribute–value model"
[self]: http://www.sqlstyle.guide
    "SQL style guide by Simon Holywell"
[licence]: http://creativecommons.org/licenses/by-sa/4.0/
    "Creative Commons Attribution-ShareAlike 4.0 International License"

