Good. Now you want a **clean operational workflow**.
No chaos. No guessing. A repeatable system.

Save this in your notes.

---

# 🔷 COMPLETE WORKFLOW — PostgreSQL + pgAdmin (Docker Host Mode)

---

# 1️⃣ Daily Startup Workflow

### Step 1 — Start PostgreSQL (Host)

```bash
sudo systemctl start postgresql@17-main
```

Verify:

```bash
sudo systemctl status postgresql@17-main
```

Must say:

```
active (running)
```

---

### Step 2 — Start pgAdmin (Docker)

If container already exists:

```bash
sudo docker start pgadmin
```

If not:

```bash
sudo docker run -d \
--network host \
-e PGADMIN_DEFAULT_EMAIL=user23052036@gmail.com \
-e PGADMIN_DEFAULT_PASSWORD=pass \
--name pgadmin \
dpage/pgadmin4
```

---

### Step 3 — Open Browser

Go to:

```
http://localhost
```

Login:

* Email: your email
* Password: your pgAdmin password

---

# 2️⃣ Connecting to Database (One-Time Setup)

If server already registered → skip.

Otherwise:

**Add New Server**

General tab:

```
Name: Local PostgreSQL
```

Connection tab:

```
Host: localhost
Port: 5432
Database: postgres
Username: postgres
Password: (your DB password)
```

Save.

---

# 3️⃣ Daily Usage Pattern (Correct Way)

### Step A — Choose Database

Left panel:

```
Servers → Local PostgreSQL → Databases → test (or any)
```

### Step B — Open Query Tool

Right-click database → Query Tool

---

### Step C — Write SQL

Example:

```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    age INT
);
```

Run using ▶ button.

---

### Step D — Insert Data

```sql
INSERT INTO students (name, age)
VALUES ('Souvik', 22);
```

---

### Step E — Query Data

```sql
SELECT * FROM students;
```

---

# 4️⃣ Proper Shutdown Workflow

### Stop pgAdmin

```bash
sudo docker stop pgadmin
```

### Stop PostgreSQL

```bash
sudo systemctl stop postgresql@17-main
```

---

# 5️⃣ Emergency Debug Checklist

If pgAdmin not loading:

```
sudo docker ps
```

If DB not connecting:

```
sudo systemctl status postgresql@17-main
```

If port issue:

```
sudo ss -nltp | grep 5432
```

---

# 6️⃣ When to Use CLI vs pgAdmin

Use **psql CLI** when:

* Debugging
* Learning internals
* Interviews
* Automation scripts

Use **pgAdmin** when:

* Visual browsing
* Schema exploration
* Query testing
* Managing roles visually

---

# 7️⃣ Mental Model (Important)

pgAdmin is just a client.

Architecture now:

```
Browser
   ↓
pgAdmin (Docker container)
   ↓
PostgreSQL server (host machine)
   ↓
Data files (/var/lib/postgresql/17/main)
```

---

# 8️⃣ Critical Things You Must Remember

* Database runs even if pgAdmin is closed.
* pgAdmin is optional.
* PostgreSQL service is the real engine.
* Docker host mode removed networking barriers.

---

# 9️⃣ Now Hard Question

If you stop PostgreSQL but keep pgAdmin running,
what happens when you click a database?

Think before answering.

---

This setup is now production-style clean.
