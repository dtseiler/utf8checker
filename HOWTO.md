To generate the proper commands, you first run this SQL:

```sql
select oid::Regclass, 
    reltuples, 
    relpages, 
    'select every(utf8checker.utf8chk(textsend(textin(record_out(t))),'||oid::text||')) from '||relname||' t;'
from pg_class where relkind = 'r' order by relpages ;
```

This will give you a listing of all of the tables in the database (including system tables), along with a column containing a command the will look like this:

```sql
select every(utf8checker.utf8chk(textsend(textin(record_out(t))),27171))
from ss_settings t;
```

You need to run all of these commands, which will check each table in the database. (You can probably skip system tables though, they should be ok)

When you run that sql, you will get 1 of three results:

* example 1 -> no rows in the table

```
every
-------

(1 row)
```


* example 2 -> all rows a valid utf8

```
every
-------
f
```

* example 3 -> a table with invalid utf8

```
NOTICE:  UTF8 error: table letters: row (120,"8.7 Quilting (text)",t,"Flower On Without 
<snip lots of text content>
</body>
</html>",16092,1,1,2)
CONTEXT:  SQL function "utf8chk" statement 1
every
-------
f
(1 row)
```

When the function encounters invalid utf8 data, it raises a NOTICE, which will contain the row of data with the problem. You can look at this information to find the primary key of the row, and then select the row back out.

So, the basic idea is to repeat the above process (query the table looking for bad utf8 data, fix the data, query again to look for bad utf8 data, rinse, repeat)

