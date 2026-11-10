# PHP Clinic Snippets

VS Code / VSCodium snippets for the PHP + MySQL "Clinic Consultations Logging System"
practice project, plus notes for moving the project from Linux (`php -S`) to
Windows (XAMPP).

Works in **VS Code** and **VSCodium** — same snippet engine, same file format.

---

## Install

### Linux — VSCodium (Flatpak)

```
~/.var/app/com.vscodium.codium/config/VSCodium/User/snippets/php.code-snippets
```

### Windows — VS Code / VSCodium

```
%APPDATA%\Code\User\snippets\php.code-snippets
%APPDATA%\VSCodium\User\snippets\php.code-snippets
```

`%APPDATA%` expands to `C:\Users\<you>\AppData\Roaming`. Paste it into Explorer's
address bar.

### Per project (recommended for submission)

```
<project>\.vscode\php.code-snippets
```

Travels with the folder, no global install, no collision with other snippet sets.
Explorer hides dot-folders — use `Ctrl+H` or create it from a terminal.

**Restart the editor once after copying.** The snippets folder is scanned at
startup only. `Ctrl+Shift+P` → `Developer: Reload Window` is usually enough.

### Verify

`Ctrl+Shift+P` → **Insert Snippet…** lists snippets by **name** (e.g. "Clinic
create form"). If they appear there, the file is loaded.

---

## Snippets

| Prefix | Generates |
|---|---|
| `clinhead` | config + search filter + list query + COUNT total |
| `clintable` | `<table>` + `<thead>`; cursor lands inside for `clinrows` |
| `clinrows` | `<tbody>` while-loop with UPDATE/DELETE links |
| `clincreate` | full "add" page: FK dropdown query + INSERT + form |
| `clindrop` | standalone FK `<select>` from a mysqli result |
| `dbconn` | `mysqli_connect` + charset, XAMPP defaults |
| `html5b` | HTML5 page shell with Bootstrap container |

### Tab stops

**`clinhead`**

| # | Default | Fill with | Appears |
|---|---|---|---|
| 1 | `nameCol1` | first searchable column | 1× |
| 2 | `nameCol2` | second searchable column | 1× |
| 3 | `Table` | table name | 2× (SELECT + COUNT) |
| 4 | `records` | word after "Total" | 1× |

**`clintable`** — `1` ID header, `2` column A, `3` column B, then `$0` lands on
the line between `</thead>` and `</table>` where `clinrows` goes.

**`clinrows`**

| # | Default | Fill with | Appears |
|---|---|---|---|
| 1 | `idCol` | primary key column | 3× (cell + both links) |
| 2 | `colA` | second visible column | 1× |
| 3 | `colB` | third visible column | 1× |
| 4 | `item_edit.php` | edit page filename | 1× |
| 5 | `item_delete.php` | delete page filename | 1× |

**`clincreate`**

| # | Default | Fill with |
|---|---|---|
| 1 | `fkCol` | FK column (e.g. `docID`) |
| 2 | `labelCol1` | first label column (e.g. `docFName`) |
| 3 | `labelCol2` | second label column (e.g. `docLName`) |
| 4 | `ParentTable` | table the dropdown reads from |
| 5 | `field1` | first column being saved |
| 6 | `field2` | second column being saved |
| 7 | `Table` | table being inserted into |
| 8 | `list.php` | page to redirect to after save |
| 9 | `Record` | noun in the heading (e.g. `Consultation`) |
| 10 | `Field 1` | label text |
| 11 | `Field 2` | label text |
| 12 | `Label` | dropdown label text |

**`clindrop`** — `1` FK column (4×), `2` label, `3` placeholder option text,
`4` result variable, `5`/`6` label columns.

**`dbconn`** — `1` database name.
**`html5b`** — `1` page title.

### Worked fills

**Patient list**

```
clinhead   → patFName · patLName · Patient · patients
clintable  → ID · First Name · Last Name
clinrows   → patID · patFName · patLName · pat_edit.php · pat_delete.php
```

**Doctor list**

```
clinhead   → docFName · docLName · Doctor · doctors
clintable  → ID · First Name · Last Name
clinrows   → docID · docFName · docLName · doctor_edit.php · doctor_delete.php
```

**Consultation create**

```
clincreate → docID · docFName · docLName · Doctor · patID · diagnosis ·
             Consultation · consult_mng.php · Consultation · Patient · Diagnosis · Doctor
```

---

## What the snippets do NOT do

Still typed by hand, and worth remembering because these are the marks:

- `htmlspecialchars()` on any echoed DB value
- `mysqli_real_escape_string()` on INSERT/UPDATE values
- `readonly` on the ID field of edit pages
- `mysqli_affected_rows()` check after UPDATE (0 rows changed is silent)
- `exit;` after `header()`
- Duplicate-ID check before INSERT
- Delete confirmation
- The five inquiry queries

---

## Moving from Linux (`php -S`) to Windows (XAMPP)

On Linux you ran `php -S localhost:8001` and PHP was both the server and the
interpreter. On Windows, XAMPP splits that into **Apache** (web server) and
**MySQL** (database), with **phpMyAdmin** as a browser front-end for MySQL.

### 1. Where the files go

```
C:\xampp\htdocs\<YourFamilyName>\
```

The folder name becomes the URL path. `htdocs` is the web root — the equivalent
of the directory you ran `php -S` from.

### 2. Start the services

XAMPP Control Panel → **Start Apache** and **Start MySQL**. Both must be green
before you open the page.

### 3. Credentials differ

This is the #1 thing that breaks on the move. XAMPP's MySQL root account has an
**empty** password:

```php
// Linux
mysqli_connect("localhost", "root", "password123", "clinic_database");

// Windows / XAMPP
mysqli_connect("localhost", "root", "", "clinic_database");
```

Use the `dbconn` snippet — it ships with the XAMPP defaults.

### 4. Import the database

The data does not travel with the files.

1. Open `http://localhost/phpmyadmin`
2. Log in as `root`, leave the password blank
3. **Databases** tab → create `clinic_database`
4. Select it → **Import** tab → choose your `.sql` dump → **Go**

Export the dump on Linux first:

```bash
mysqldump -u root -p clinic_database > clinic_database.sql
```

Or use phpMyAdmin's **Export** tab on the Linux machine.

### 5. Paths

`htdocs\<Folder>\` means the project is **not** at the web root. So:

```html
<link rel="stylesheet" href="/assets/bootstrap.css">   <!-- resolves to htdocs\assets\  ✗ -->
<link rel="stylesheet" href="assets/bootstrap.css">    <!-- resolves to <Folder>\assets\  ✓ -->
```

Remove leading slashes from every asset path.

### 6. Case sensitivity

Windows filenames are case-insensitive, Linux is not. `Doctor.php` and
`doctor.php` are one file on Windows, two on Linux. Keep `require` paths
matching the real filename exactly.

### 7. SQL strictness

Your Linux MySQL may be lenient where XAMPP's MariaDB is strict. Empty strings
inserted into INT columns become `0` on one and error hard on the other. If a
working page suddenly fails on Windows with *"Incorrect integer value"*, that's
this. Fix the data, don't disable strict mode.

### 8. Still want `php -S` on Windows?

Fine — XAMPP ships the CLI too:

```
C:\xampp\php\php.exe -S localhost:8001
```

Run it from inside the project folder. Apache can keep running; different ports
don't collide. Then MySQL still has to be started from the Control Panel.

---

## Diagnosing connectivity

### Checklist, in order

1. **Is MySQL running?** XAMPP Control Panel shows the port next to MySQL
   (usually 3306).
2. **Does the database exist?** phpMyAdmin's left panel should list
   `clinic_database`.
3. **Do the credentials match?** `root` + empty password on XAMPP.
4. **Is the table there?** Click the database → the table list.
5. **Is Apache running?** `http://localhost` should serve the XAMPP dashboard.
6. **Does PHP have mysqli?** In a terminal: `C:\xampp\php\php.exe -m` and look
   for `mysqli`.

### One-file connection test

Save as `test.php` in the project folder, open
`http://localhost/<Folder>/test.php`:

```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', '1');

$conn = mysqli_connect("localhost", "root", "", "clinic_database");
if (!$conn) {
    die("connect failed: " . mysqli_connect_error());
}

echo "connected: " . mysqli_get_server_info($conn) . "<br>";

$r = mysqli_query($conn, "SELECT NOW() AS now");
echo "server time: " . mysqli_fetch_assoc($r)["now"];
```

If `mysqli_connect` returns false you put the error message on screen
deliberately. If `mysqli` instead throws an exception (PHP 8.1+) you'll see an
uncaught `mysqli_sql_exception` with the same text.

### Where the logs are

| What | Path |
|---|---|
| Apache errors | `C:\xampp\apache\logs\error.log` |
| PHP errors under Apache | same file |
| MySQL errors | `C:\xampp\mysql\data\mysql_error.log` |
| php.ini | `C:\xampp\php\php.ini` |

Turn errors on while developing in `php.ini`:

```ini
display_errors = On
error_reporting = E_ALL
```

Restart Apache after editing `php.ini`.

### Port conflicts

If Apache won't start, something else owns port 80:

```
netstat -ano | findstr :80
taskkill /PID <pid> /F
```

If MySQL won't start, something owns 3306 — often a leftover MySQL Windows
service:

```
netstat -ano | findstr :3306
```

---

## Common MySQL errors

| Code | Message | Usual cause |
|---|---|---|
| 1045 | Access denied for user 'root' | wrong password — XAMPP root is blank |
| 1049 | Unknown database | DB not created / wrong name |
| 2002 | No connection could be made | MySQL not running, or wrong port |
| 2003 | Can't connect to MySQL server | MySQL stopped, or firewall |
| 1146 | Table doesn't exist | typo, or DB not imported |
| 1054 | Unknown column | wrong column name, or wrong table |
| 1062 | Duplicate entry for key PRIMARY | inserting an ID that already exists |
| 1452 | FK constraint fails | FK value not present in the parent table |
| 1364 | Field doesn't have a default value | column left out of the INSERT |
| 1048 | Column cannot be null | `NULL` into a `NOT NULL` column |

---

## phpMyAdmin in 60 seconds

| Tab | What it does |
|---|---|
| **Browse** | view rows |
| **Structure** | columns, types, keys, engine |
| **SQL** | run a query by hand |
| **Search** | filter rows |
| **Insert** | add a row manually |
| **Export** | dump as `.sql` |
| **Import** | load a `.sql` dump |
| **Operations** | rename/delete/copy the table |

**Check the engine.** If a table is **MyISAM**, `FOREIGN KEY` clauses are
accepted and silently ignored. It must be **InnoDB** for the Consultation FKs to
mean anything. SQL tab:

```sql
SHOW TABLE STATUS WHERE Name IN ('Doctor','Patient','Consultation');
```

Look at the `Engine` column.

**Get the DB version** (useful when behaviour differs between machines):

```sql
SELECT VERSION();
```

---

## Exam checklist

- [ ] project folder named after your family name
- [ ] `index.php` menu with all four modules
- [ ] Doctor / Patient / Consultation: add, search, update, delete, view
- [ ] all five inquiry queries
- [ ] **every result set shows a COUNT** — 5 points
- [ ] `patTelNo` is `type="tel"` (the data dictionary says text)
- [ ] `diagnosis` / `prescription` are `<textarea>`
- [ ] asset paths have no leading slash
- [ ] DB connection uses XAMPP credentials on the Windows machine
