
# 🔹 PostgreSQL Workflow (Linux CLI Version)

---

# 1️⃣ Understand the Layers First

There are **3 layers**:

1. **PostgreSQL Server (Database Engine)**
   Runs in background as a service.

2. **System User: `postgres`**
   Linux user that owns the database.

3. **psql Client**
   Command-line tool to talk to the database.

If you mix these up → confusion starts.

---

# 2️⃣ Check If PostgreSQL Is Running

```bash
sudo systemctl status postgresql@17-main
```

Break it:

* `postgresql@` → template service
* `17` → PostgreSQL version
* `main` → cluster name

So:

`postgresql@17-main`
means → **PostgreSQL version 17, cluster named “main”**

---

Why this exists:

PostgreSQL on Debian supports **multiple clusters on same machine**.

Example possible setup:

* `postgresql@16-main`
* `postgresql@17-test`
* `postgresql@15-analytics`

Each can:

* Run on different ports
* Have separate data directories
* Be started/stopped independently

---

If:

```
Active: active (running)
```

You are good.

If it says:

```
Active: inactive (dead)
```

That means the PostgreSQL server is **stopped**.

---

### Start it again:

```bash
sudo systemctl start postgresql@17-main
```

Then verify:

```bash
sudo systemctl status postgresql@17-main
```

You should see:

```
Active: active (running)
```

That’s it.


---

# 3️⃣ Connect to PostgreSQL (Superuser Access)

```bash
sudo -u postgres psql
```

What happens:

* Switch to postgres Linux user
* Open psql
* Connect to DB server

You’ll see:

```
postgres=#
```

Now you are **inside the database**.

---

# 4️⃣ Basic psql Commands (Inside Database)

These are NOT Linux commands.

They only work inside `psql`.

| Command              | Meaning             |
| -------------------- | ------------------- |
| `\l`                 | List databases      |
| `\c dbname`          | Connect to database |
| `\dt`                | Show tables         |
| `\du`                | Show users          |
| `\password username` | Change password     |
| `\q`                 | Quit psql           |

Example:

```sql
CREATE DATABASE test;
\l
\c test
```

---

# 5️⃣ Creating Your Own User (Important)

Right now you’re using the superuser.

Better practice:

Inside psql:

```sql
CREATE USER souvik WITH PASSWORD 'yourpassword';
ALTER USER souvik WITH SUPERUSER;
```

Then:

```sql
\q
```

---

# 6️⃣ Login Without sudo (Cleaner Way)

Now connect like this:

```bash
psql -U souvik -d test -h localhost
```

It will ask password.

This is the **real-world way**.

---

# 7️⃣ Difference Between Exit Commands

You got confused earlier. Here’s why:

| Where You Are                    | How to Exit |
| -------------------------------- | ----------- |
| Inside psql                      | `\q`        |
| Inside postgres Linux user shell | `exit`      |
| Inside root shell                | `exit`      |

You tried:

```
\q
```

inside root bash → that won't work.

Because `\q` only works inside psql.

Context matters.

---

# 8️⃣ Stop PostgreSQL Server

If needed:

```bash
sudo systemctl stop postgresql@17-main
```

Start again:

```bash
sudo systemctl start postgresql@17-main
```

---

# 🔟 Architecture Mental Model (Very Important)

When you type:

```sql
CREATE DATABASE test;
```

What actually happens?

1. psql sends query to server
2. Server parses SQL
3. Planner builds execution plan
4. Executor runs it
5. Files created inside:

   ```
   /var/lib/postgresql/17/main
   ```

PostgreSQL is not magic. It’s a process manager + storage engine.

---

# 🚨 Now Hard Question

Why does Debian use:

```
postgresql@17-main
```

instead of just `postgresql.service` like MySQL?

If you can answer that → you understand cluster design.

If not → you’re still surface level.

---

If you want next:

* Real SQL practice workflow
* PostgreSQL architecture deep dive
* Or how authentication works (peer vs md5 vs scram)

Choose one.
