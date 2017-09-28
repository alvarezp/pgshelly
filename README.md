pgshelly
--------

Pgshelly lets us use a shell-like interface to manipulate PostgreSQL.

For now I only have an example/tutorial:

```
[  0][Wed Sep 27 21:50:25 -0500 -- alvarezp@alvarezp-samsung:~]
$ # Hi. This is a demo for pgshelly.

[  0][Wed Sep 27 21:50:33 -0500 -- alvarezp@alvarezp-samsung:~]
$ # I will quickly create an app called mypets. It will let me

[  0][Wed Sep 27 21:50:48 -0500 -- alvarezp@alvarezp-samsung:~]
$ # store dog data at first, but I can easily expand it to cats.

[  0][Wed Sep 27 21:51:00 -0500 -- alvarezp@alvarezp-samsung:~]
$ # Right now I have pgshelly under bin/

[  0][Wed Sep 27 21:51:17 -0500 -- alvarezp@alvarezp-samsung:~]
$ ls bin/pgshelly
bin/pgshelly

[  0][Wed Sep 27 21:51:20 -0500 -- alvarezp@alvarezp-samsung:~]
$ ln -s pgshelly bin/mypets

[  0][Wed Sep 27 21:51:28 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets
psql: FATAL:  database "mypets" does not exist

[  2][Wed Sep 27 21:51:32 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
psql: FATAL:  database "mypets" does not exist

[  2][Wed Sep 27 21:51:35 -0500 -- alvarezp@alvarezp-samsung:~]
$ createdatabase mypets
bash: createdatabase: command not found

[127][Wed Sep 27 21:51:41 -0500 -- alvarezp@alvarezp-samsung:~]
$ createdb mypets #Ooops...

[  0][Wed Sep 27 21:51:49 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
ERROR:  relation "dogs" does not exist
LINE 1: SELECT * FROM dogs ;
                      ^

[  1][Wed Sep 27 21:51:57 -0500 -- alvarezp@alvarezp-samsung:~]
$ psql mypets
psql (9.6.3, server 9.4.10)
Type "help" for help.

mypets=# create table dogs (name varchar primary key, breed varchar);
CREATE TABLE
mypets=# \q

[  0][Wed Sep 27 21:52:13 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
 name | breed 
------+-------
(0 rows)


[  0][Wed Sep 27 21:52:19 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs new --name=Pluto --breed=bloodhound
INSERT 0 1

[  0][Wed Sep 27 21:52:28 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs new --name=Spike --breed=bulldog
INSERT 0 1

[  0][Wed Sep 27 21:52:36 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
 name  |   breed    
-------+------------
 Pluto | bloodhound
 Spike | bulldog
(2 rows)


[  0][Wed Sep 27 21:52:39 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs new --name=Pongo --breed=dalmatian
INSERT 0 1

[  0][Wed Sep 27 21:52:48 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show --name=Pongo
 name  |   breed   
-------+-----------
 Pongo | dalmatian
(1 row)


[  0][Wed Sep 27 21:52:53 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show --name=Spike
 name  |  breed  
-------+---------
 Spike | bulldog
(1 row)


[  0][Wed Sep 27 21:52:56 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs new --name='Santa'\''s Little Helper' --breed=greyhound
INSERT 0 1

[  0][Wed Sep 27 21:53:16 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
         name          |   breed    
-----------------------+------------
 Pluto                 | bloodhound
 Spike                 | bulldog
 Pongo                 | dalmatian
 Santa's Little Helper | greyhound
(4 rows)


[  0][Wed Sep 27 21:53:18 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs new --name=Ren --breed=Stimpy
INSERT 0 1

[  0][Wed Sep 27 21:53:27 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
         name          |   breed    
-----------------------+------------
 Pluto                 | bloodhound
 Spike                 | bulldog
 Pongo                 | dalmatian
 Santa's Little Helper | greyhound
 Ren                   | Stimpy
(5 rows)


[  0][Wed Sep 27 21:53:30 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs set --name=Ren to --breed=chihuahua
UPDATE 1

[  0][Wed Sep 27 21:53:41 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
         name          |   breed    
-----------------------+------------
 Pluto                 | bloodhound
 Spike                 | bulldog
 Pongo                 | dalmatian
 Santa's Little Helper | greyhound
 Ren                   | chihuahua
(5 rows)


[  0][Wed Sep 27 21:53:43 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs delete --name=Spike
DELETE 1

[  0][Wed Sep 27 21:53:51 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
         name          |   breed    
-----------------------+------------
 Pluto                 | bloodhound
 Pongo                 | dalmatian
 Santa's Little Helper | greyhound
 Ren                   | chihuahua
(4 rows)


[  0][Wed Sep 27 21:53:53 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs about --name=Ren
ERROR:  function about(dogs) does not exist
LINE 1: SELECT about(dogs.*) FROM dogs WHERE name = $$Ren$$;
               ^
HINT:  No function matches the given name and argument types. You might need to add explicit type casts.

[  1][Wed Sep 27 21:54:05 -0500 -- alvarezp@alvarezp-samsung:~]
$ psql mypets
psql (9.6.3, server 9.4.10)
Type "help" for help.

mypets=# create function about(dogs) RETURNS text LANGUAGE SQL AS $$
mypets$# SELECT $1.name || ' is a ' || $1.breed;
mypets$# $$;
CREATE FUNCTION
mypets=# \q

[  0][Wed Sep 27 21:54:31 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs about --name=Ren
       about        
--------------------
 Ren is a chihuahua
(1 row)


[  0][Wed Sep 27 21:54:33 -0500 -- alvarezp@alvarezp-samsung:~]
$ # Thank you for watching! github.com/alvarezp/pgshelly

```
