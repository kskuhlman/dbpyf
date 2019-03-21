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

###[Sqitch](http://sqitch.org) "Sane database change management" [git](https://github.com/theory/sqitch).  
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
TBD

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
