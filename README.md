pgshelly
--------

*Pgshelly* is a CLI to SQL translator. It lets the user manipulate
PostgreSQL through typical shell-like commands. A database becomes
an application by symlinking it to the *pgshelly* script.

## Example in a nutshell

Try this on PostgreSQL:
```
CREATE DATABASE mypets
CREATE TABLE dogs (name VARCHAR PRIMARY KEY, breed VARCHAR);
CREATE FUNCTION about(dogs) RETURNS text LANGUAGE SQL AS $$
  SELECT $1.name || ' is a ' || $1.breed;
$$
```
And then try this on the command-line:
```
ln -s pgshelly mypets #Adjust this to the correct paths

./mypets dogs new --name=Pluto --breed=bloodhound
./mypets dogs new --name=Spike --breed=bulldog
./mypets dogs show
./mypets dogs new --name=Pongo --breed=dalmatian
./mypets dogs show --name=Spike
./mypets dogs new --name='Santa'\''s Little Helper' --breed=greyhound
./mypets dogs show
./mypets dogs new --name=Ren --breed=Stimpy
./mypets dogs show
./mypets dogs set --breed=chihuahua on --name=Ren
./mypets dogs show
./mypets dogs delete --name=Spike
./mypets dogs show
./mypets dogs about --name=Ren
```
## Example in detail

Let's quickly create an app called `mypets`. It will let us
store dog data at first, but we can easily expand it to cats by
adding another table.

We will symlink our new app, `./mypets` to *pgshelly*. With this,
*pgshelly* will try to connect to the database named after the
application name, in this case, `mypets`.

On my laptop, `pgshelly` lives under `$HOME/bin/`, which I already
have on my `$PATH`.

I am symlinking `bin/mypets` to `bin/pgshelly` so I don't need
the `./` prefix.

```
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

```
Right, the database `mypets` doesn't exist yet. Let's create it.

```
[  2][Wed Sep 27 21:51:35 -0500 -- alvarezp@alvarezp-samsung:~]
$ createdb mypets
```

... and retry:

```
[  0][Wed Sep 27 21:51:49 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
ERROR:  relation "dogs" does not exist
LINE 1: SELECT * FROM dogs ;
                      ^
```
The second argument to `mypets` is the "object" (table) to work with.
In this case, dogs, but we don't have a `dogs` table. Let's create it.

```
[  1][Wed Sep 27 21:51:57 -0500 -- alvarezp@alvarezp-samsung:~]
$ psql mypets
psql (9.6.3, server 9.4.10)
Type "help" for help.

mypets=# CREATE TABLE dogs (name VARCHAR PRIMARY KEY, breed VARCHAR);
CREATE TABLE
mypets=# \q
```
Showing our dogs should now work:
```
[  0][Wed Sep 27 21:52:13 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
 name | breed 
------+-------
(0 rows)

```
Yes! Let's try adding some dogs (inserting some records):

```
[  0][Wed Sep 27 21:52:19 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs new --name=Pluto --breed=bloodhound
INSERT 0 1

[  0][Wed Sep 27 21:52:28 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs new --name=Spike --breed=bulldog
INSERT 0 1

```
... and displaying them...

```
[  0][Wed Sep 27 21:52:36 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs show
 name  |   breed    
-------+------------
 Pluto | bloodhound
 Spike | bulldog
(2 rows)
```
Let's play some more:
```
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

```
Quoting is not tested, but *pgshelly* uses the double dollar sign so
it should not be so bad:

```
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

```
We can also update records:

```
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
$ mypets dogs set --breed=chihuahua on --name=Ren
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

```
... and delete:

```

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

```
And we can create custom, programmable actions from within PostgreSQL,
taking advantage of user-defined functions and field notation for
composite types. First, let's try it without having one, just to see
how it fails:
```

[  0][Wed Sep 27 21:53:53 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs about --name=Ren
ERROR:  column dogs.about does not exist
LINE 1: SELECT dogs.about FROM dogs WHERE name = $$Ren$$;
               ^
```
Let's create about():
```
[  1][Wed Sep 27 21:54:05 -0500 -- alvarezp@alvarezp-samsung:~]
$ psql mypets
psql (9.6.3, server 9.4.10)
Type "help" for help.

mypets=# create function about(dogs) RETURNS text LANGUAGE SQL AS $$
mypets$# SELECT $1.name || ' is a ' || $1.breed;
mypets$# $$;
CREATE FUNCTION
mypets=# \q
```
... and use it from `mypets`:
```
[  0][Wed Sep 27 21:54:31 -0500 -- alvarezp@alvarezp-samsung:~]
$ mypets dogs about --name=Ren
       about        
--------------------
 Ren is a chihuahua
(1 row)
```

Thank you!
