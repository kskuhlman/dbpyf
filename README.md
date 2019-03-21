# dbpyf
Database version manager & (re)-formatter, in Python.

# Who
The project motivation stems from life on the IBM i (aka i5, iSeries, AS/400), but with eyes wide open about 
what the rest of the world is doing.  There are piecemeal tools around, but the general area of database 
version management is still very rough waters. (Thar be dragons!, even).  

dbpyf is an attempt to calm the storm a bit futher, in a cross-platform way. But the first priority the IBM i. ;-)


# Why
Database changes are a neglected area of change management. On many platforms, "patches" are generated for 
each alter, usually with no dependency management, testing, or recoverability built in.  And what about changes 
that don't impact the schema design?  Data should be versionable, too!

Let's identify best practices in database change management, and implement them in a plugable, cross-platform way.

# Part 1:  Schema Versioning
## Prior Work 

Here's a good high-level market review.. slanted towards MSSqlSvr a bit, but still useful:
http://www.releasemanagement.org/2016/02/database-version-control-tools/

## Market Leaders:  Flyway & Liquibase
Flyway has a solid versioning approach to database conversions, and supports multiple platforms 
(DB2 linked to here, despite it directed at DB2 UDB LUW and DB2 UDB for z instead of Db2 UDB for i):
https://flywaydb.org/documentation/database/db2


Another popular option is Liquibase.   Flyway VS Liquibase which one to use? 
- [12 minute video](https://www.youtube.com/watch?v=q0JMqKk36dU)
- Flyway doesn't natively support generating an alter script - it applies immediate. But there is a fork with this functionality. 
- Liquibase has this functionality [built-in](https://stackoverflow.com/questions/14482644/can-flyway-or-liquibase-generate-an-update-script-instead-of-updating-the-databa?rq=1)
- One team's ["lessons learned" with Flyway](http://www.jeremyjarrell.com/using-flyway-db-with-distributed-version-control/)

## Lessor Known

### [Sqitch](http://sqitch.org) "Sane database change management" [git](https://github.com/theory/sqitch).  
Has ability to run Revert and Verification scripts, but must be hand-generated.  Written in Perl. No DB2 support (yet).  

From docs:
- No opinions: 
  - Sqitch is not integrated with any framework, ORM, or platform. Rather, it is a standalone change management system 
    with no opinions about your database engine, application framework, or development environment.
- Native scripting
  - Changes are implemented as scripts native to your selected database engine. 
    Writing a PostgreSQL application? Write SQL scripts for psql. Writing a MySQL-backed app? Write SQL scripts for mysql.
- Dependency resolution
  - Database changes may declare dependencies on other changes—even on changes from other Sqitch projects. 
    This ensures proper order of execution, even when you’ve committed changes to your VCS out-of-order.
- No numbering
  - Change deployment is managed by maintaining a plan file. As such, there is no need to number your changes, 
    although you can if you want. Sqitch doesn’t much care how you name your changes.
- Iterative development
  - Up until you tag and release your application, you can modify your change deployment scripts as often as you like. 
    They’re not locked in just because they’ve been committed to your VCS. This allows you to take an iterative approach
    to developing your database schema. Or, better, you can do test-driven database development.

#### Presentations
- Sane Database Change Management with Sqitch
A one hour technical introduction to Sqitch, with detailed usage examples to help get you started.
- [Video](https://vimeo.com/50104469)
- [PDF](https://speakerd.s3.amazonaws.com/presentations/5e9bcbd0430a0130009a123139173c61/sqitch-pdxpm-2013.pdf)
- [Speaker Deck](https://speakerdeck.com/theory/sane-database-change-management-with-sqitch)
- Agile Database Development
Three hour technical tutorial originally presented at PGCon 2013 and updated in January 2014, covering source code control with Git, database change control with Sqitch, and test-driven database development with pgTAP.
- [Speaker Deck](https://speakerdeck.com/theory/agile-database-development-2ed)
- [Keynote](https://www.icloud.com/iw/#keynote/BAJN1kHfLmpMcwXIXOqByV75FvbX6AgfEbWE/agile_database_development_iovation.key)
- [PDF](https://speakerd.s3.amazonaws.com/presentations/460b5b905af60131df53620ea5c6d896/agile_database_development_iovation.pdf)

### Other less-known tools that should be researched quick:
- https://github.com/mattes/migrate
- https://bitbucket.org/liamstask/goose
- https://github.com/tanel/dbmigrate
- https://github.com/BurntSushi/migration
- https://github.com/DavidHuie/gomigrate
- https://github.com/rubenv/sql-migrate


# Part 2:  Data Changes
Database change management isn't complete without a row comparison tool to assist with testing & releasing control table changes.

A big hurdle is adding/removing/altering columns.  Also, updating FKs for columns with surrogate keys (identity/sequences),
during the application of diffs.   And how to merge & resolve conflicts? 

A few examples to get the juices flowing: 
```
drop table foo;
drop table bar;
create or replace table foo (animal char(10) not null primary key, color varchar(20));
create or replace table bar (animal char(10) not null primary key, color varchar(20));
 
insert into foo values('ant','black');
insert into bar values('ant','red');
insert into foo values('bee','yellow');
insert into bar values('cat','orange');
insert into foo values('dog','gold');
insert into bar values('dog','gold');
```

Example 2:
```
data1 = [
    ['Country','Capital'],
    ['Ireland','Dublin'],
    ['France','Paris'],
    ['Spain','Barcelona']]
``` 
```
data2 = [
    ['Country','Code','Capital'],
    ['Ireland','ie','Dublin'],
    ['France','fr','Paris'],
    ['Spain','es','Madrid'],
    ['Germany','de','Berlin']]
```    

TODO: Start of a 3rd example:
```
CUSTOMERS
| ID | NAME     | AGE | ADDRESS   | SALARY   |
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 |
|  2 | Khilan   |  25 | Delhi     |  1500.00 |
|  3 | kaushik  |  23 | Kota      |  2000.00 |
|  4 | Chaitali |  25 | Mumbai    |  6500.00 |
|  5 | Hardik   |  27 | Bhopal    |  8500.00 |
|  6 | Komal    |  22 | MP        |  4500.00 |
|  7 | Muffy    |  24 | Indore    | 10000.00 |

ORDERS
|OID  | DATE                | CUSTOMER_ID | AMOUNT |
| 102 | 2009-10-08 00:00:00 |           3 |   3000 |
| 100 | 2009-10-08 00:00:00 |           3 |   1500 |
| 101 | 2009-11-20 00:00:00 |           2 |   1560 |
| 103 | 2008-05-20 00:00:00 |           4 |   2060 |

From:  https://www.tutorialspoint.com/sql/sql-using-joins.htm
```


## Starting Simple:  Except
SQL's EXCEPT clause can identify row differences, providing a simple solution for trivial use cases.
Here's an example, taken from [jmerrell](http://jmerrell.com/2011/06/01/db2-except-sql-function/).

```
SELECT J1.*, 'TABLE1' AS SRC_TBL
FROM (
SELECT * FROM qtemp.TABLE1
EXCEPT
SELECT * FROM qtemp.TABLE2
) AS J1
UNION
SELECT J2.*, 'TABLE2' AS SRC_TBL
FROM (
SELECT * FROM qtemp.TABLE2
EXCEPT
SELECT * FROM qtemp.TABLE1
) AS J2
```


##  Getting Serious:  diffkit, daff

- [DIFFKIT](http://www.diffkit.org/quickstart.html), a data comparison & merging tool. Written in Java, config driven. 
  Seemingly robust & once popular.  Interesting features.. but is it in a state of disrepair? 
   - [Docs](http://www.diffkit.org/quickstart.html)
   - [Src](https://github.com/DiffKit/DiffKit)
  
- [DAFF](http://paulfitz.github.io/daff/), less popular, more oriented towards version control & visualization of differences. 
  Written in HAXE, an obscure (to me) cross-language tool that compiles into other languages (Ruby, Java, Python, etc):
  - [README](https://github.com/paulfitz/daff/blob/master/README.md)
  - [Source](https://github.com/frictionlessdata/specs/tree/master/sources)
  - [Binaries](https://github.com/paulfitz/daff/releases)
  - Old [Blog post on his inspiration](http://okfnlabs.org/blog/2013/08/08/diffing-and-patching-data.html).

 - [COOPY](https://github.com/paulfitz/coopy).  An earlier attempt by the author of DAFF.  A lot of overlap, but distinct features.
  - [Docs](http://share.find.coop/doc/index.html).  Excerpt:
    - The COOPY project contains programs and libraries to help you:
     - Describe the difference between two tables (see ssdiff).
     - Apply the difference between two tables to a table (see sspatch).
     - Merge tables that came from the same source but have been modified independently (see ssmerge). 
     - Two- and three-way merges supported.
     - Maintain a repository of tables with multiple active contributors (see ssfossil and the coopy program).
     - Supported table formats include CSV, sqlite, mysql, Access (read-only), Excel, OpenOffice.    
  - [Community](https://sourceforge.net/p/coopy/mailman/coopy-users/)(small).


The diffkit & daff projects have [an interesting history](https://groups.google.com/forum/#!topic/diffkit-user/vLA9GaiFsDY). 
  - That link also mentions "datomic data model", which is a new term to me.
  - The diffkit & daff authors collaborated to create an [annotated diff format for data](http://share.find.coop/doc/spec_hilite.html).
  - Complete with [tutorial](http://share.find.coop/doc/tutorial_hilite.html).
  - "hilite" doesn't address the FK problem.. that may be what the "datomic data model" reference above was getting at.  Not clear.
  
  
## Closed Source
Altova, the makers of XMLSpy, also sell [DiffDog](https://www.altova.com/blog/three-way-file-comparison-and-difference-merging/),
which claims to be able to merge tables.   That's all I know about that.



## Other Lightweight Markup Languages (LMLs)

The "hilite" format mentioned earlier could be considered just another "lightweight markup language"
([LML|https://en.wikipedia.org/wiki/Lightweight_markup_language]) like MarkDown, reStructuredText, textile, etc.

Other LMLs have more/different tooling around them... do any have a table format that's interesting to us?
Markdown itself doesn't, but variants like [MultiMarkDown](http://fletcher.github.io/MultiMarkdown-5/tables.html) do.
And there's an old proposal to fold similar functionality into the 
standard [MarkDown table RFC](http://justatheory.com/computers/markup/markdown-table-rfc.html).  
And GitHub-flavored MarkDown has some support, but it's sadly left out of [CommonMark](http://spec.commonmark.org/0.20/#html-blocks).

Despite MarkDown's increasing ubiquity, Eric Holscher makes a decent argument against using it for development documentation 
[here](http://ericholscher.com/blog/2016/mar/15/dont-use-markdown-for-technical-docs/).  
Briefly, MarkDown wasn't intended to be extensible, and GitHub supports more then just MarkDown. (e.g.,  reStructuredText and Asciidoc).
And "As a developer working with rST or Asciidoctor, I can add new markup to the language in a simple, pluggable way. 
I don’t have to change how the language is parsed, and I can share those additions with other users via a standard extension mechanism."

It would be nice to use a format that is supported by GitHub so that our tables/diffs can be displayed there.  
[GitHub-supported markup languages](https://github.com/github/markup#markups)

#### Tables in Asciidoc
The [asciidoc authors](http://asciidoctor.org/docs/asciidoc-writers-guide/) are very proud of their table support.  
You can either create tables longhand or with embedded CSV or DSV:

- Longhand:
```
|===
|Name |Group |Description

|Firefox
|Web Browser
|Mozilla Firefox is an open-source web browser.
It's designed for standards compliance,
performance, portability.

|Ruby
|Programming Language
|A programmer's best friend.
```
```
|===
```
- Embedded CSV/DSV 
```
[%header,format=csv]
|===
Artist,Track,Genre
Baauer,Harlem Shake,Hip Hop
The Lumineers,Ho Hey,Folk Rock
|===
```
- Or, shorter: 
```
,===
a,b,c
,===

Or, with colon separators:
:===
a:b:c
:===
```

###  Tables in reStructuredText
Basic table functionality in [RST](http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.html#grid-tables) is more intuitive,
but there's no direct CSV support.  I need to read up on macros & extensibility, though.

- Simple Tables
```
=====  =====  ======
   Inputs     Output
------------  ------
  A      B    A or B
=====  =====  ======
False  False  False
True   False  True
False  True   True
True   True   True
=====  =====  ======
```

- Grid Tables
```
+------------------------+------------+----------+----------+
| Header row, column 1   | Header 2   | Header 3 | Header 4 |
| (header rows optional) |            |          |          |
+========================+============+==========+==========+
| body row 1, column 1   | column 2   | column 3 | column 4 |
+------------------------+------------+----------+----------+
| body row 2             | Cells may span columns.          |
+------------------------+------------+---------------------+
| body row 3             | Cells may  | - Table cells       |
+------------------------+ span rows. | - contain           |
| body row 4             |            | - body elements.    |
+------------------------+------------+---------------------+
```

###  Other LMLs
Checked out & did not like Creole (simplistic), Org (crazy complex), MediaWiki (ugly), PerlPod (no tables), 
RDoc (no tables),  and Textile (not interesting enough).

Of these, Org & Textile are worth noting.  Org for it's calculations: 
```
| N | N^2 | N^3 | N^4 | ~sqrt(n)~ | ~sqrt[4](N)~ |
|---+-----+-----+-----+-----------+--------------|
| / |   < |     |   > |         < |            > |
| 1 |   1 |   1 |   1 |         1 |            1 |
| 2 |   4 |   8 |  16 |    1.4142 |       1.1892 |
| 3 |   9 |  27 |  81 |    1.7321 |       1.3161 |
|---+-----+-----+-----+-----------+--------------|
#+TBLFM: $2=$1^2::$3=$1^3::$4=$1^4::$5=sqrt($1)::$6=sqrt(sqrt(($1)))
```


And [Textile](https://www.promptworks.com/textile/page-layout#tables) for it's tables:
```
|_. name|_. age|
|Walter|5|
|Florence|6|
```

## Other History
You might want to just skim this section.. it feels like a tangent to me, and I'm the author.  And even more than in other sections
of this document, it's note properly source-reference quoted.

The OUSEFUL link above has some interesting additional references.

https://frictionlessdata.io/guides/validating-data/

[Data Packages](https://frictionlessdata.io/data-packages/)
Data Package is a simple container format used to describe and package a collection of data. 
The format provides a simple contract for data interoperability that supports frictionless delivery, installation 
and management of data.

Data Packages can be used to package any kind of data. 
At the same time, for specific common data types such as tabular data it has support for providing important 
additional descriptive metadata -- for example, describing the columns and data types in a CSV.

The following core principles inform our approach:
- Simplicity
- Extensibility and customisation by design
- Metadata that is human-editable and machine-usable
- Reuse of existing standard formats for data
- Language, technology and infrastructure agnostic

[goodtables](https://frictionlessdata.io/guides/validating-data/#goodtables)
goodtables is a free, open-source, hosted service for validating tabular data. 
goodtables checks your data for its structure, and, optionally, its adherence to a specified schema. 
goodtables will give quick and simple feedback on where your tabular data may not yet be quite perfect.

[Table Schema](https://frictionlessdata.io/guides/table-schema/) json format
Table Schema is a standard for providing a “schema” (similar to a database schema) for tabular data. 
This information includes the expected type of each value in a column (“string”, “number”, “date”, etc.), 
constraints on the value (“this string can only be at most 10 characters long”), and the expected format 
of the data (“this field should only contain strings that look like email addresses). 
Table Schema can also specify relations between tables.


Note that the frictionless data team invited the COOPY/DAF author to propose a standard with them, 
and he created a patch that was merged.  HOWEVER, I can't find the file in the master branch.

https://github.com/frictionlessdata/specs/pull/64#event-62379887


## Less Abmitious Efforts
If implementing in Python, and starting more gradually, [csvdiff](https://pypi.python.org/pypi/csvdiff) could be a starting point.
Then slowly add features from the other two packages.

Generates a diff between two CSV files (using it's own custom format, not HiLite or JSON-patch).

Dumping HiLite & favoring JSON-patch is [jsondiff](https://github.com/ZoomerAnalytics/jsondiff). 
[Home page](http://zoomeranalytics.com)
I haven't researched how mature it is, but it is in Python.  And we're getting further away from rows & csv files now.

This javascript library spits out standard format json patches:
https://www.npmjs.com/package/json-diff-rfc6902

Note that json-patch is not reversible by default.  
  - The [jiff](https://github.com/cujojs/jiff) package knows how to create them that way. 
    (see also [issue 9](https://github.com/cujojs/jiff/issues/9))

Or we could address reversibility by storing the whole doc plus the change instructions...


## What about the Users? 
LMLs are great for developers, but what happens when we need to pull in business users in order to resolve merge conflicts, etc?
OpenOffice / Goog Sheets / Excel?  Custom code each time?  

"OpenRefine" might be a good way to visualize changes.  Better then regular spreadsheets with it's change tracking mechanism. 

https://blog.ouseful.info/2013/08/27/diff-or-chop-github-csv-data-files-and-openrefine/


## Isn't there a Happy Medium?

Well, ndJSON (new-line delimted JSON) is basically CSV v2.  Allowing complex documents but still streamable.  

So combining the best of breeds between the CSV utilities and the JSON utilities does sound promising.  

If you have ideas on how to move forward in that direction, please reach out.


## Stepping Back..  Where does that leave us?

When all is said & done, COOPY & diffkit probably put us on the firmest fundation, despite their focus on flat csv files.  

Other links for COOPY-inspired work:
- https://blog.ouseful.info/2013/08/27/diff-or-chop-github-csv-data-files-and-openrefine/
- http://okfnlabs.org/blog/2013/08/08/diffing-and-patching-data.html
- https://theodi.org/blog/adapting-git-simple-data
- https://github.com/gitlabhq/gitlabhq/pull/4810

ndJSON is making JSON interesting again, but we need to figure out where it fits into the pipeline, and if it's the best choice.


The immediate need is to get structure & data into patch sets, 
watch out for conflicting surrogate keys (adjust child tables when found), 
and having the ability to revert bad changes. 


Git rebase makes the 2nd workable (I think), but we need to start working with more complex examples to figure out conflicts.


# Part 3:  BIG changes
There doesn't appear to be any cross-platform tools that are intended to handle schema changes for big data.

Oracle has "online table redefinition," which is a really nice hack.

For the IBM i, there is 
- [ArCad's product "Skipper-WAP"](https://arcadsoftware.com/wp-content/uploads/2014/05/PR-ARCAD-Announces-Skipper-WAP.pdf).
- And an open source project [RapidFire](https://rapid-fire.sourceforge.io/index.php) 
   [(src)](https://sourceforge.net/p/rapid-fire/activity/?page=0&limit=100#5b333045ee24ca020a916530).  
- Both create a duplicate of the object & then apply journal changes ("transaction logs") to it until 
  you're ready to rename the tables (like Oracle).  
- Rapidfire has plugins for the Rational toolset to do configuration & status checking.
