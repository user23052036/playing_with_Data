
```sql
SELECT datname FROM pg_database;
```

→ Lists all databases in the PostgreSQL cluster.

Equivalent terminal command (inside psql):

```
\l or \list
```

---

```sql
CREATE DATABASE test2;
```

→ Creates a new database named `test2`.

---

`\c test`

Connects you to the database named `test`.

After running it, your prompt changes from:

```
postgres=#
```

to:

```
test=#
```
---

```sql
DROP DATABASE insta_db;
```

Deletes the entire database named `insta_db`, including all schemas, tables, data, indexes, and objects inside it.

You must:

* Not be connected to `insta_db`
* Ensure no active sessions are using it

Otherwise it will fail.

---

