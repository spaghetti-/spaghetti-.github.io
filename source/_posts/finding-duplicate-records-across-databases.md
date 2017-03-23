---
title: Finding duplicate records across different databases
date: 2017-03-23 10:31:13
tags: postgresql
author: alex
---

Recently a BI database that was produced via an ELT showed duplicate records so
I needed to diff the tables. A friend suggested using redis to diff the ids so
I went with it. We need the `redis-cli` utility for this.

The fun part is doing it directly from shell.

Generating two sets[0]:

```bash
psql -h proddb.rds.amazonaws.com -U normuser -d db -c \
       'select id from table;' | tail -n +3 | ghead -n -2 \
       | xargs | sed 's/^/sadd prod /g' | redis-cli

psql -h eltdb.rds.amazonaws.com -U normuser -d db -c \
       'select id from table;' | tail -n +3 | ghead -n -2 \
       | xargs | sed 's/^/sadd elt /g' | redis-cli
```

I've found that this is reasonably fast without constructing raw redis protocol
for inserts via `redis-cli --pipe`.

Now we can generate a list of duplicate ids by doing

```bash
redis-cli --raw sdiff elt prod | xargs | sed 's/ /,/g'
```

Then we can find the records by using the output from the above command like so

```sql
SELECT data FROM table WHERE id = ANY(ARRAY[1, 2, 3, ...]);
```


[0] `ghead` is installed via `brew install coreutils` on MacOS because BSD `head`
does not support negative arguments
